---
title: "Baidu Apollo 感知红绿灯代码串讲"
date: 2019-11-01 00:52
---

[TOC]

Apollo 3.5 版本，代码地址：

[apollo/modules/perception/camera/lib/traffic_light/](https://github.com/ApolloAuto/apollo/tree/r3.5.0/modules/perception/camera/lib/traffic_light/)。

交通信号灯感知，使用摄像头获取当前车道对应红绿灯的状态。

支持的灯形：

![](/wiki/attach/images/baidu_apollo_trafficlights_codes_reading/detect_box_types.png)

支持的颜色：

- 红
- 绿
- 黄
- 黑

使用长焦、短焦两个相机：

- 长焦相机(12mm焦距)：观察前方远距离的信号灯，视野较小；
- 短焦相机(6mm焦距)：视野较大，观察近处信号灯。

输入：

- 相机图像 channel
- 相机参数（内外参）
- HDMap
- 无人车定位
- V2X 红绿灯结果

输出：

- { Timestamp, {Signal ID, Detected BBox, Signal Color} }

## 1 Onboard 部分

```
modules/perception/onboard/component/trafficlights_perception_component.h
modules/perception/onboard/component/trafficlights_perception_component.cc
```

`TrafficLightsPerceptionComponent`: 红绿灯模块对应的 CyberRT 组件

对应 dag 文件：

modules/perception/production/dag/dag_streaming_perception_trafficlights.dag

```
module_config {
  module_library : "/apollo/bazel-bin/modules/perception/onboard/component/libperception_component_camera.so"
  components {
    class_name : "TrafficLightsPerceptionComponent"
    config {
      name: "TrafficLightsComponent"
      config_file_path: "/apollo/modules/perception/production/conf/perception/camera/trafficlights_perception_component.config"
      flag_file_path: "/apollo/modules/perception/production/conf/perception/perception_common.flag"
      readers {
          channel: "/apollo/perception/traffic_light_status"
      }
    }
  }
}
```

处理入口函数：

```cpp
void TrafficLightsPerceptionComponent::OnReceiveImage(
    const std::shared_ptr<apollo::drivers::Image> msg,  // 图像消息
    const std::string& camera_name  // 相机名
    );
```

这是一个回调(callback)函数，当接收到对应相机的图像消息时自动调用。

`OnReceiveImage()` 主要处理流程：

```
1. 投影选相机

查询定位和地图信号灯信息，根据信号灯在图像平面上的投影情况选择用于检测红绿灯的相机，保存相机选择结果。

2. 判断当前图像时间戳和上一次处理时间戳间隔，若间隔过小，跳过

这一步是为了降低实际检测的处理频率，红绿灯检测不需要很高的帧率

3. 同步当前图像和相机选择结果

判断当前图像是否来自于要用于检测红绿灯的相机。如果不是则直接返回，不用继续处理当前图像；如果是则继续进行下一步。

4. 调用红绿灯检测算法模块，给出红绿灯识别结果

- Detect: 检测，根据地图红绿灯投影在 Crop ROI 中检测红绿灯 2D 框
- Recognize: 识别，对检测出的红绿灯 2D 框内容进行分类（红、绿、黑、黄）
- Revise: 修正，处理可能出现的漏检(Unknown)、误识别情况

5. 同步感知红绿灯识别结果和 V2X 红绿灯结果，输出。

```

相关代码：

```cpp
void TrafficLightsPerceptionComponent::OnReceiveImage(
    const std::shared_ptr<apollo::drivers::Image> msg,
    const std::string& camera_name) {
  std::lock_guard<std::mutex> lck(mutex_);

  // ...

  camera::TLPreprocessorOption preprocess_option;
  preprocess_option.image_borders_size = &image_border_sizes_;

  // 1. 投影选相机
  // UpdateCameraSelection() 实现了根据信号灯投影选择相机的功能
  // query pose and signals, add cached camera selection by lights' projections
  if (!UpdateCameraSelection(image_msg_ts, preprocess_option, &frame_)) {
    AWARN << "add_cached_camera_selection failed, ts: "
          << std::to_string(image_msg_ts);
  }
  const auto update_camera_selection_time =
      PERCEPTION_PERF_BLOCK_END_WITH_INDICATOR(perf_indicator,
                                               "UpdateCameraSelection");

  // 2. 判断当前图像时间戳和上一次处理时间戳间隔，若间隔过小，跳过
  // skipping frame according to last proc image timestamp
  if (last_proc_image_ts_ > 0.0 &&
      receive_img_timestamp - last_proc_image_ts_ < proc_interval_seconds_) {
    // ...
    return;
  }

  // 3. 同步当前图像和相机选择结果
  // sync image with cached projections
  bool sync_image_ok =
      preprocessor_->SyncInformation(image_msg_ts, camera_name);
  const auto sync_information_time = PERCEPTION_PERF_BLOCK_END_WITH_INDICATOR(
      perf_indicator, "SyncInformation");
  // 当前图像不是来自于要选用的相机，直接返回，不需要进行后续的检测、识别
  if (!sync_image_ok) {
    // ...
    return;
  }

  // ...

  // 由于存在跳帧策略，选相机时查 pose 的时间和当前图像的时间可能不同步，再查一次 pose 进行投影
  // 确保得到的投影准确
  if (!VerifyLightsProjection(image_msg_ts, preprocess_option, camera_name,
                              &frame_)) {
    AINFO << "VerifyLightsProjection on image failed, ts: "
          << std::to_string(image_msg_ts) << ", camera_name: " << camera_name
          << " last_query_tf_ts_: " << std::to_string(last_query_tf_ts_)
          << " need update_camera_selection immediately,"
          << " reset last_query_tf_ts_ to -1";
    last_query_tf_ts_ = -1.0;
  }

  // 4. 调用红绿灯检测算法模块，给出红绿灯识别结果
  traffic_light_pipeline_->Perception(camera_perception_options_, &frame_);

  // 5. 同步感知红绿灯识别结果和 V2X 红绿灯结果
  SyncV2XTrafficLights(&frame_);

  // 发送结果消息
  // ...
}

// V2X 红绿灯消息回调函数
void TrafficLightsPerceptionComponent::OnReceiveV2XMsg(
    const std::shared_ptr<apollo::v2x::IntersectionTrafficLightData> v2x_msg) {
  std::lock_guard<std::mutex> lck(mutex_);
  v2x_msg_buffer_.push_back(*v2x_msg);
}

void TrafficLightsPerceptionComponent::SyncV2XTrafficLights(
    camera::CameraFrame* frame) {
  const double camera_frame_timestamp = frame->timestamp;
  auto sync_single_light = [&](base::TrafficLightPtr light) {
    for (auto itr = v2x_msg_buffer_.rbegin(); itr != v2x_msg_buffer_.rend();
         ++itr) {
      double v2x_timestamp = (*itr).header().timestamp_sec();
      // 从 V2X 消息缓存中找到时间戳足够接近的一帧
      if (std::fabs(camera_frame_timestamp - v2x_timestamp) <
          v2x_sync_interval_seconds_) {
        const int v2x_lights_num =
            (*itr).current_lane_trafficlight().single_traffic_light_size();
        const auto& v2x_lights = (*itr).current_lane_trafficlight();
        for (int i = 0; i < v2x_lights_num; ++i) {
          const auto& v2x_light = v2x_lights.single_traffic_light(i);
          // 利用 Signal ID 进行对应
          if (light->id != v2x_light.id()) {
            continue;
          }
          base::TLColor v2x_color = base::TLColor::TL_UNKNOWN_COLOR;
          bool blink = false;
          switch (v2x_light.color()) {
            default:
            case apollo::v2x::SingleTrafficLight::UNKNOWN:
              v2x_color = base::TLColor::TL_UNKNOWN_COLOR;
              break;
            case apollo::v2x::SingleTrafficLight::RED:
              v2x_color = base::TLColor::TL_RED;
              break;
            case apollo::v2x::SingleTrafficLight::YELLOW:
              v2x_color = base::TLColor::TL_YELLOW;
              break;
            case apollo::v2x::SingleTrafficLight::GREEN:
              v2x_color = base::TLColor::TL_GREEN;
              break;
            case apollo::v2x::SingleTrafficLight::BLACK:
              v2x_color = base::TLColor::TL_BLACK;
              break;
            case apollo::v2x::SingleTrafficLight::FLASH_GREEN:
              v2x_color = base::TLColor::TL_GREEN;
              blink = true;
              break;
          }
          // 完全相信 V2X 的结果
          AINFO << "Sync V2X success. update color from "
                << static_cast<int>(light->status.color) << " to "
                << static_cast<int>(v2x_color) << "; signal id: " << light->id;
          light->status.color = v2x_color;
          light->status.blink = blink;
        }
        break;
      }
    }
  };
  for (auto& light : frame->traffic_lights) {
    sync_single_light(light);
  }
}

```

---

## 2 算法部分

```
modules/perception/camera/lib/traffic_light
  ├── preprocessor
  └── detector
        └── detection
        └── recognition
  └── tracker
```

### 2.1 preprocessor

预处理，主要完成的任务是通过投影地图信号灯到图像平面来选择相机。

```
preprocessor/
  ├── tl_preprocess.proto          # 配置定义
  ├── tl_preprocessor.cc
  ├── tl_preprocessor.h            # 投影选相机预处理 (*)
  ├── pose.cc
  ├── pose.h                       # 主车和相机 Pose 封装
  ├── multi_camera_projection.cc
  └── multi_camera_projection.h    # 多相机投影

```

`tl_preprocessor.cc` 主要函数介绍：

```cpp
// 投影选相机
bool TLPreprocessor::UpdateCameraSelection(
    const CarPose &pose, const TLPreprocessorOption &option,
    std::vector<base::TrafficLightPtr> *lights) {
  const double &timestamp = pose.getTimestamp();
  selected_camera_name_.first = timestamp;
  selected_camera_name_.second = GetMaxFocalLenWorkingCameraName();
  AINFO << "TLPreprocessor Got signal number: " << lights->size()
        << ", ts: " << std::to_string(timestamp);
  // 大部分情况下是没有信号灯的，此时我们默认选择焦距最大的相机，只是为了让红绿灯模块有输出
  if (lights->empty()) {
    AINFO << "No signals, select camera with max focal length: "
          << selected_camera_name_.second;
    return true;
  }
  // ProjectLightsAndSelectCamera() 为具体投影选相机的方法
  if (!ProjectLightsAndSelectCamera(pose, option,
                                    &(selected_camera_name_.second), lights)) {
    AERROR << "project_lights_and_select_camera failed, ts: "
           << std::to_string(timestamp);
  }
  return true;
}

bool TLPreprocessor::ProjectLightsAndSelectCamera(
    const CarPose &pose, const TLPreprocessorOption &option,
    std::string *selected_camera_name,
    std::vector<base::TrafficLightPtr> *lights) {
  // 两个数组，分别保存投影在/不在图像上的灯
  for (auto &light_ptrs : lights_on_image_array_) {
    light_ptrs.clear();
  }
  for (auto &light_ptrs : lights_outside_image_array_) {
    light_ptrs.clear();
  }
  // project light region on each camera's image plane
  const auto &camera_names = projection_.getCameraNamesByDescendingFocalLen();
  for (size_t cam_id = 0; cam_id < num_cameras_; ++cam_id) {
    const std::string &camera_name = camera_names[cam_id];
    if (!ProjectLights(pose, camera_name, lights,
                       &(lights_on_image_array_[cam_id]),
                       &(lights_outside_image_array_[cam_id]))) {
      AERROR << "select_camera_by_lights_projection project lights on "
             << camera_name << " image failed";
      return false;
    }
  }

  // ...
  // 根据投影结果选择相机
  SelectCamera(&lights_on_image_array_, &lights_outside_image_array_, option,
               selected_camera_name);
  return true;
}

void TLPreprocessor::SelectCamera(
    std::vector<base::TrafficLightPtrs> *lights_on_image_array,
    std::vector<base::TrafficLightPtrs> *lights_outside_image_array,
    const TLPreprocessorOption &option, std::string *selected_camera_name) {
  // do not check boundary if this is min focal camera
  auto min_focal_len_working_camera = GetMinFocalLenWorkingCameraName();
  AINFO << "working camera with minimum focal length: "
        << min_focal_len_working_camera;

  // 按照焦距从大到小遍历相机，检查是否有灯的投影
  const auto &camera_names = projection_.getCameraNamesByDescendingFocalLen();
  for (size_t cam_id = 0; cam_id < lights_on_image_array->size(); ++cam_id) {
    const auto &camera_name = camera_names[cam_id];
    bool ok = true;
    if (camera_name != min_focal_len_working_camera) {
      // 有灯无法投影到该相机上，跳过
      if (lights_outside_image_array->at(cam_id).size() > 0) {
        AINFO << "light project out of image, "
              << "camera_name: " << camera_name
              << " lights_outside_image_array->at(cam_id).size(): "
              << lights_outside_image_array->at(cam_id).size();
        continue;
      }
      // 投影边界检查
      auto lights = lights_on_image_array->at(cam_id);
      for (const auto light : lights) {
        // check boundary
        if (OutOfValidRegion(light->region.projection_roi,
                             projection_.getImageWidth(camera_name),
                             projection_.getImageHeight(camera_name),
                             option.image_borders_size->at(camera_name))) {
          ok = false;
          AINFO << "light project out of image region, "
                << "camera_name: " << camera_name << " border_size: "
                << option.image_borders_size->at(camera_name);
          break;
        }
      }
    } else {
      // do not checkout the boundary if this is min focal camera
      ok = (lights_on_image_array->at(cam_id).size() > 0);
    }
    if (ok) {
      *selected_camera_name = camera_name;
      break;
    }
  }
}
```

`multi_camera_projection.cc` ：

```cpp
bool MultiCamerasProjection::BoundaryBasedProject(
    const base::BrownCameraDistortionModelPtr camera_model,
    const Eigen::Matrix4d& c2w_pose,
    const std::vector<base::PointXYZID>& points,
    base::TrafficLight* light) const {
  CHECK_NOTNULL(camera_model.get());
  int width = static_cast<int>(camera_model->get_width());
  int height = static_cast<int>(camera_model->get_height());
  int bound_size = static_cast<int>(points.size());
  AINFO << "bound size " << bound_size;
  if (bound_size < 4) {
    AERROR << "invalid bound_size";
    return false;
  }
  std::vector<Eigen::Vector2i> pts2d(bound_size);
  auto c2w_pose_inverse = c2w_pose.inverse();

  // 将 HDMap 中的 signal 的 boundary points 投影到图像上
  for (int i = 0; i < bound_size; ++i) {
    const auto& pt3d_world = points.at(i);
    Eigen::Vector3d pt3d_cam =
        (c2w_pose_inverse *
         Eigen::Vector4d(pt3d_world.x, pt3d_world.y, pt3d_world.z, 1.0))
            .head(3);
    if (std::islessequal(pt3d_cam[2], 0.0)) {
      AWARN << "light bound point behind the car: " << pt3d_cam;
      return false;
    }
    pts2d[i] = camera_model->Project(pt3d_cam.cast<float>()).cast<int>();
  }

  // 取投影点的 bounding box
  int min_x = std::numeric_limits<int>::max();
  int max_x = std::numeric_limits<int>::min();
  int min_y = std::numeric_limits<int>::max();
  int max_y = std::numeric_limits<int>::min();
  for (const auto& pt : pts2d) {
    min_x = std::min(pt[0], min_x);
    max_x = std::max(pt[0], max_x);
    min_y = std::min(pt[1], min_y);
    max_y = std::max(pt[1], max_y);
  }

  base::BBox2DI roi(min_x, min_y, max_x, max_y);
  if (OutOfValidRegion(roi, width, height) || roi.Area() == 0) {
    AWARN << "Projection get ROI outside the image. ";
    return false;
  }
  light->region.projection_roi = base::RectI(roi);
  return true;
}
```

### 2.2 detector/detection

检测，根据灯的投影在 Crop ROI 中给出红绿灯 2D 框

![](/wiki/attach/images/baidu_apollo_trafficlights_codes_reading/tl_detect.png)

```
detection/
  ├── detection.proto    # 配置定义
  ├── detection.cc
  ├── detection.h        # TrafficLightDetection (*)
  ├── cropbox.cc
  ├── cropbox.h          # 用于根据红绿灯投影框获取 Crop ROI
  ├── select.cc
  └── select.h           # 候选检测框与红绿灯投影框匹配

```

`detection.cc` 主要函数介绍:

```cpp
bool TrafficLightDetection::Detect(const TrafficLightDetectorOptions &options,
                                   CameraFrame *frame) {
  // ...
  const auto &data_provider = frame->data_provider;
  auto input_blob = rt_net_->get_blob(net_inputs_[0]);
  // ...
  std::vector<base::TrafficLightPtr> &lights_ref = frame->traffic_lights;

  AINFO << "detection input " << lights_ref.size() << " lights";
  // detected_bboxes_ 是模型给出的所有候选检测框
  detected_bboxes_.clear();
  // selected_bboxes_ 是匹配后的检测框
  selected_bboxes_.clear();

  // ...
  // 检测模型 inference
  Inference(&lights_ref, data_provider);
  for (size_t j = 0; j < detected_bboxes_.size(); ++j) {
    base::RectI &region = detected_bboxes_[j]->region.detection_roi;
    float score = detected_bboxes_[j]->region.detect_score;
    lights_ref[0]->region.debug_roi.push_back(region);
    lights_ref[0]->region.debug_roi_detect_scores.push_back(score);
  }

  // 匹配，为每个投影框选择一个检测框
  select_.SelectTrafficLights(detected_bboxes_, &lights_ref);
  return true;
}

bool TrafficLightDetection::Inference(
    std::vector<base::TrafficLightPtr> *lights, DataProvider *data_provider) {
  // ...
  crop_box_list_.clear();        // 保存 Crop ROI
  resize_scale_list_.clear();    // 保存 2D 检测框缩放比例
  int img_width = data_provider->src_width();
  int img_height = data_provider->src_height();
  int resize_index = 0;
  auto batch_num = lights->size();
  auto input_img_blob = rt_net_->get_blob(net_inputs_[0]);
  auto input_param = rt_net_->get_blob(net_inputs_[1]);

  // 每个 Crop ROI 都 resize 到 [min_crop_size x min_crop_size] 再进行检测
  input_img_blob->Reshape(static_cast<int>(batch_num),
                          static_cast<int>(detection_param_.min_crop_size()),
                          static_cast<int>(detection_param_.min_crop_size()),
                          3);
  param_blob_->Reshape(static_cast<int>(batch_num), 6, 1, 1);
  float *param_data = param_blob_->mutable_cpu_data();
  for (size_t i = 0; i < batch_num; ++i) {
    auto offset = i * param_blob_length_;
    param_data[offset + 0] =
        static_cast<float>(detection_param_.min_crop_size());
    param_data[offset + 1] =
        static_cast<float>(detection_param_.min_crop_size());
    param_data[offset + 2] = 1;
    param_data[offset + 3] = 1;
    param_data[offset + 4] = 0;
    param_data[offset + 5] = 0;
  }

  for (size_t i = 0; i < batch_num; ++i) {
    base::TrafficLightPtr light = lights->at(i);
    base::RectI cbox;
    // 计算 Crop ROI
    crop_->getCropBox(img_width, img_height, light, &cbox);

    if (!OutOfValidRegion(cbox, img_width, img_height) && cbox.Area() > 0) {
      crop_box_list_.push_back(cbox);
      light->region.debug_roi[0] = cbox;
      light->region.crop_roi = cbox;

      data_provider_image_option_.do_crop = true;
      data_provider_image_option_.crop_roi = cbox;
      data_provider_image_option_.target_color = base::Color::BGR;
      data_provider->GetImage(data_provider_image_option_, image_.get());

      // 保存缩放比例，用于还原原图像中的检测框
      float resize_scale =
          static_cast<float>(detection_param_.min_crop_size()) /
          static_cast<float>(std::min(cbox.width, cbox.height));
      resize_scale_list_.push_back(resize_scale);

      // 将所有 Crop ROI 都 resize 到相同大小
      inference::ResizeGPU(*image_, input_img_blob, img_width, resize_index,
                           mean_[0], mean_[1], mean_[2], true, 1.0);
      resize_index++;
    }
  }
  // Inference
  cudaDeviceSynchronize();
  rt_net_->Infer();
  cudaDeviceSynchronize();

  // 还原原图像中的检测框
  SelectOutputBoxes(crop_box_list_, resize_scale_list_, resize_scale_list_,
                    &detected_bboxes_);

  // NMS，消除有重叠区域的 Crop ROIs 中重复的检测框
  ApplyNMS(&detected_bboxes_);

  return true;
}

```

`cropbox.cc` :

```cpp
void CropBox::Init(float crop_scale, int min_crop_size) {
  crop_scale_ = crop_scale;          // 缩放倍数
  min_crop_size_ = min_crop_size;    // 最小 crop size
}

void CropBox::getCropBox(const int width, const int height,
                         const base::TrafficLightPtr &light,
                         base::RectI *crop_box) {
  int rows = height;
  int cols = width;
  if (OutOfValidRegion(light->region.projection_roi, width, height) ||
      light->region.projection_roi.Area() <= 0) {
    crop_box->x = 0;
    crop_box->y = 0;
    crop_box->width = 0;
    crop_box->height = 0;
    return;
  }
  int xl = light->region.projection_roi.x;
  int yt = light->region.projection_roi.y;
  int xr = xl + light->region.projection_roi.width - 1;
  int yb = yt + light->region.projection_roi.height - 1;

  // 取 projection roi 的长边，乘以 crop_scale 作为 Crop ROI 的边长
  int center_x = (xr + xl) / 2;
  int center_y = (yb + yt) / 2;
  int resize =
      static_cast<int>(crop_scale_ * static_cast<float>(std::max(
                                         light->region.projection_roi.width,
                                         light->region.projection_roi.height)));
  // 限制 Crop ROI 最小边长
  resize = std::max(resize, min_crop_size_);
  resize = std::min(resize, width);
  resize = std::min(resize, height);

  // 计算新的 left top 和 right bottom 点
  xl = center_x - resize / 2 + 1;
  xl = (xl < 0) ? 0 : xl;
  yt = center_y - resize / 2 + 1;
  yt = (yt < 0) ? 0 : yt;
  xr = xl + resize - 1;
  yb = yt + resize - 1;
  if (xr >= cols - 1) {
    xl -= xr - cols + 1;
    xr = cols - 1;
  }
  if (yb >= rows - 1) {
    yt -= yb - rows + 1;
    yb = rows - 1;
  }

  crop_box->x = xl;
  crop_box->y = yt;
  crop_box->width = xr - xl + 1;
  crop_box->height = yb - yt + 1;
}

```

`select.cc` :

```cpp
void Select::SelectTrafficLights(
    const std::vector<base::TrafficLightPtr> &refined_bboxes,
    std::vector<base::TrafficLightPtr> *hdmap_bboxes) {
  // 将投影框和候选检测框进行匈牙利匹配
  std::vector<std::pair<size_t, size_t> > assignments;
  munkres_.costs()->Resize(hdmap_bboxes->size(), refined_bboxes.size());

  for (size_t row = 0; row < hdmap_bboxes->size(); ++row) {
    auto center_hd = (*hdmap_bboxes)[row]->region.detection_roi.Center();
    // 地图灯的投影在图像外，对应 score 置 0
    if ((*hdmap_bboxes)[row]->region.outside_image) {
      AINFO << "projection_roi outside image, set score to 0.";
      for (size_t col = 0; col < refined_bboxes.size(); ++col) {
        (*munkres_.costs())(row, col) = 0.0;
      }
      continue;
    }
    // 计算 score 矩阵
    for (size_t col = 0; col < refined_bboxes.size(); ++col) {
      float gaussian_score = 100.0f;
      auto center_refine = refined_bboxes[col]->region.detection_roi.Center();
      // use gaussian score as metrics of distance and width
      double distance_score = Calc2dGaussianScore(
          center_hd, center_refine, gaussian_score, gaussian_score);

      double max_score = 0.9;
      auto detect_score = refined_bboxes[col]->region.detect_score;
      double detection_score =
          detect_score > max_score ? max_score : detect_score;

      double distance_weight = 0.7;
      double detection_weight = 1 - distance_weight;
      // 图像上 2D 距离分数和检测模型 confidence 的加权平均
      // 分数越大表示匹配程度越高
      (*munkres_.costs())(row, col) =
          static_cast<float>(detection_weight * detection_score +
                             distance_weight * distance_score);
      const auto &crop_roi = (*hdmap_bboxes)[row]->region.crop_roi;
      const auto &detection_roi = refined_bboxes[col]->region.detection_roi;
      // 如果候选检测框不是完全在当前 Crop ROI 之内， score 置 0
      if ((detection_roi & crop_roi) != detection_roi) {
        AINFO << "detection_roi outside crop_roi, set score to 0."
              << " detection_roi: " << detection_roi.x << " " << detection_roi.y
              << " " << detection_roi.width << " " << detection_roi.height
              << " crop_roi: " << crop_roi.x << " " << crop_roi.y << " "
              << crop_roi.width << " " << crop_roi.height;
        (*munkres_.costs())(row, col) = 0.0;
      }
      AINFO << "score " << (*munkres_.costs())(row, col);
    }
  }
  // 求解匹配
  munkres_.Maximize(&assignments);

  // 为每个灯分配检测框
  for (size_t i = 0; i < hdmap_bboxes->size(); ++i) {
    (*hdmap_bboxes)[i]->region.is_selected = false;
    (*hdmap_bboxes)[i]->region.is_detected = false;
  }
  for (size_t i = 0; i < assignments.size(); ++i) {
    if (static_cast<size_t>(assignments[i].first) >= hdmap_bboxes->size() ||
        static_cast<size_t>(
            assignments[i].second >= refined_bboxes.size() ||
            (*hdmap_bboxes)[assignments[i].first]->region.is_selected ||
            refined_bboxes[assignments[i].second]->region.is_selected)) {
    } else {
      auto &refined_bbox_region = refined_bboxes[assignments[i].second]->region;
      auto &hdmap_bbox_region = (*hdmap_bboxes)[assignments[i].first]->region;
      refined_bbox_region.is_selected = true;
      hdmap_bbox_region.is_selected = true;

      const auto &crop_roi = hdmap_bbox_region.crop_roi;
      const auto &detection_roi = refined_bbox_region.detection_roi;
      bool outside_crop_roi = ((crop_roi & detection_roi) != detection_roi);
      if (hdmap_bbox_region.outside_image || outside_crop_roi) {
        hdmap_bbox_region.is_detected = false;
      } else {
        hdmap_bbox_region.detection_roi = refined_bbox_region.detection_roi;
        hdmap_bbox_region.detect_class_id = refined_bbox_region.detect_class_id;
        hdmap_bbox_region.detect_score = refined_bbox_region.detect_score;
        hdmap_bbox_region.is_detected = refined_bbox_region.is_detected;
        hdmap_bbox_region.is_selected = refined_bbox_region.is_selected;
      }
    }
  }
}

```

### 2.3 detector/recognition

识别，对检测结果进行分类，给出颜色识别结果

```
recognition/
  ├── recognition.proto  # 配置定义
  ├── recognition.cc
  ├── recognition.h      # TrafficLightRecognition (*)
  ├── classify.cc
  └── classify.h         # classification inference
```

主要函数：

```cpp
bool TrafficLightRecognition::Detect(const TrafficLightDetectorOptions& options,
                                     CameraFrame* frame) {
  std::vector<base::TrafficLightPtr> candidate(1);

  for (base::TrafficLightPtr light : frame->traffic_lights) {
    if (light->region.is_detected) {
      candidate[0] = light;
      // 检测模型输出的检测框有三个类别，分别对应方形、竖直和水平形状
      // 每种形状都有一个识别模型
      if (light->region.detect_class_id ==
          base::TLDetectionClass::TL_QUADRATE_CLASS) {
        AINFO << "Recognize Use Quadrate Model!";
        classify_quadrate_->Perform(frame, &candidate);
      } else if (light->region.detect_class_id ==
                 base::TLDetectionClass::TL_VERTICAL_CLASS) {
        AINFO << "Recognize Use Vertical Model!";
        classify_vertical_->Perform(frame, &candidate);
      } else if (light->region.detect_class_id ==
                 base::TLDetectionClass::TL_HORIZONTAL_CLASS) {
        AINFO << "Recognize Use Horizonal Model!";
        classify_horizontal_->Perform(frame, &candidate);
      } else {
        return false;
      }
    } else {
      // 未检测到，输出 Unknown color
      light->status.color = base::TLColor::TL_UNKNOWN_COLOR;
      light->status.confidence = 0;
    }
  }

  return true;
}

```

### 2.4 tracker

修正，红绿灯语义、时序校正

```
tracker/
  ├── semantic.proto         # 配置定义
  ├── semantic_decision.cc
  └── semantic_decision.h    # SemanticReviser (*)

```

语义校正：

当前时刻，同一语义的红绿灯中，输出置信度最高的颜色。如果置信度最高的颜色有多种，输出数量最多的。如果数量相同，输出unknown（比如一红一绿）。

时序校正：

基于一些规则对模型结果进行后处理。例如，某帧漏检出现 unknown color，根据前 1.5 秒内有颜色的输出将当前帧结果修正为该颜色，保证输出结果稳定；黄灯误识别为红灯，将之后的颜色均校正为红色(硬策略，避免黄灯出现在红灯之后给 PnC 造成困扰)。

主要函数：

```cpp
bool SemanticReviser::Track(const TrafficLightTrackerOptions &options,
                            CameraFrame *frame) {
  double time_stamp = frame->timestamp;
  std::vector<base::TrafficLightPtr> &lights_ref = frame->traffic_lights;
  std::vector<SemanticTable> semantic_table;
  if (lights_ref.size() <= 0) {
    history_semantic_.clear();
    ADEBUG << "no lights to revise, return";
    return true;
  }

  // 根据信号灯语义进行分组
  for (size_t i = 0; i < lights_ref.size(); i++) {
    base::TrafficLightPtr light = lights_ref.at(i);
    int cur_semantic = light->semantic;
    ADEBUG << "light " << light->id << " semantic " << cur_semantic;

    SemanticTable tmp;
    std::stringstream ss;

    if (cur_semantic > 0) {
      ss << "Semantic_" << cur_semantic;
    } else {
      ss << "No_semantic_light_" << light->id;
    }

    tmp.semantic = ss.str();
    tmp.light_ids.push_back(static_cast<int>(i));
    tmp.color = light->status.color;
    tmp.time_stamp = time_stamp;
    tmp.blink = false;
    auto iter =
        std::find_if(std::begin(semantic_table), std::end(semantic_table),
                     boost::bind(compare, _1, tmp));

    if (iter != semantic_table.end()) {
      iter->light_ids.push_back(static_cast<int>(i));
    } else {
      semantic_table.push_back(tmp);
    }
  }

  // 时序校正
  for (size_t i = 0; i < semantic_table.size(); ++i) {
    SemanticTable cur_semantic_table = semantic_table.at(i);
    ReviseByTimeSeries(time_stamp, cur_semantic_table, &lights_ref);
  }

  return true;
}
```

```cpp
// 保存同语义红绿灯信息的数据结构
// 假设同语义的红绿灯状态是一致的
struct SemanticTable {
  double time_stamp = 0.0;
  double last_bright_time_stamp = 0.0;
  double last_dark_time_stamp = 0.0;
  bool blink = false;  // 是否闪烁
  std::string semantic;  // 语义
  // semantic 是一个 4 位的 01 字符串，每一位分别标识 NO_TURN、U_TURN、LEFT_TURN、RIGHT_TURN
  // 如 `0010` 表示左转灯，`1010` 表示既控制左转也控制直行
  std::vector<int> light_ids;  // 红绿灯 ID
  base::TLColor color;
};

```

同语义灯校正：

```cpp
base::TLColor SemanticReviser::ReviseBySemantic(
    SemanticTable semantic_table, std::vector<base::TrafficLightPtr> *lights) {
  std::vector<int> vote(static_cast<int>(base::TLColor::TL_TOTAL_COLOR_NUM), 0);
  std::vector<base::TrafficLightPtr> &lights_ref = *lights;
  base::TLColor max_color = base::TLColor::TL_UNKNOWN_COLOR;

  // 相同语义的灯，记录每种颜色出现的次数
  for (size_t i = 0; i < semantic_table.light_ids.size(); ++i) {
    int index = semantic_table.light_ids.at(i);
    base::TrafficLightPtr light = lights_ref[index];
    auto color = light->status.color;
    vote.at(static_cast<int>(color))++;
  }

  // 只有黑色或者 unknown 的情况
  if ((vote.at(static_cast<size_t>(base::TLColor::TL_RED)) == 0) &&
      (vote.at(static_cast<size_t>(base::TLColor::TL_GREEN)) == 0) &&
      (vote.at(static_cast<size_t>(base::TLColor::TL_YELLOW)) == 0)) {
    if (vote.at(static_cast<size_t>(base::TLColor::TL_BLACK)) > 0) {
      return base::TLColor::TL_BLACK;
    } else {
      return base::TLColor::TL_UNKNOWN_COLOR;
    }
  }

  vote.at(static_cast<size_t>(base::TLColor::TL_BLACK)) = 0;
  vote.at(static_cast<size_t>(base::TLColor::TL_UNKNOWN_COLOR)) = 0;

  // 出现数量最多的颜色
  auto biggest = std::max_element(std::begin(vote), std::end(vote));
  int max_color_num = *biggest;
  max_color = base::TLColor(std::distance(std::begin(vote), biggest));
  vote.erase(biggest);
  auto second_biggest = std::max_element(std::begin(vote), std::end(vote));

  // 如果数量最多的前两种颜色数量相同，返回 unknown
  if (max_color_num == *second_biggest) {
    return base::TLColor::TL_UNKNOWN_COLOR;
  } else {
    return max_color;
  }
}
```

时序校正：

```cpp
void SemanticReviser::ReviseByTimeSeries(
    double time_stamp, SemanticTable semantic_table,
    std::vector<base::TrafficLightPtr> *lights) {
  ADEBUG << "revise " << semantic_table.semantic
         << ", lights number:" << semantic_table.light_ids.size();

  std::vector<base::TrafficLightPtr> &lights_ref = *lights;
  base::TLColor cur_color = ReviseBySemantic(semantic_table, lights);
  base::TLColor pre_color = base::TLColor::TL_UNKNOWN_COLOR;
  semantic_table.color = cur_color;
  semantic_table.time_stamp = time_stamp;
  ADEBUG << "revise same semantic lights";
  ReviseLights(lights, semantic_table.light_ids, cur_color);

  std::vector<SemanticTable>::iterator iter =
      std::find_if(std::begin(history_semantic_), std::end(history_semantic_),
                   boost::bind(compare, _1, semantic_table));

  // 找相同 ID 的灯历史 revise_time_s_ （1.5 秒）内的颜色
  // 如果当前颜色是黑灯或者 unknown，直接用历史颜色赋值
  if (iter != history_semantic_.end()) {
    pre_color = iter->color;
    if (time_stamp - iter->time_stamp < revise_time_s_) {
      ADEBUG << "revise by time series";
      switch (cur_color) {
        case base::TLColor::TL_YELLOW:
          if (iter->color == base::TLColor::TL_RED) {
            ReviseLights(lights, semantic_table.light_ids, iter->color);
            iter->time_stamp = time_stamp;
            iter->hystertic_window.hysteretic_count = 0;
          } else {
            UpdateHistoryAndLights(semantic_table, lights, &iter);
            ADEBUG << "High confidence color " << s_color_strs[cur_color];
          }
          break;
        case base::TLColor::TL_RED:
        case base::TLColor::TL_GREEN:
          UpdateHistoryAndLights(semantic_table, lights, &iter);
          if (time_stamp - iter->last_bright_time_stamp > blink_threshold_s_ &&
              iter->last_dark_time_stamp > iter->last_bright_time_stamp) {
            iter->blink = true;
          }
          iter->last_bright_time_stamp = time_stamp;
          ADEBUG << "High confidence color " << s_color_strs[cur_color];
          break;
        case base::TLColor::TL_BLACK:
          iter->last_dark_time_stamp = time_stamp;
          iter->hystertic_window.hysteretic_count = 0;
          if (iter->color == base::TLColor::TL_UNKNOWN_COLOR ||
              iter->color == base::TLColor::TL_BLACK) {
            iter->time_stamp = time_stamp;
            UpdateHistoryAndLights(semantic_table, lights, &iter);
          } else {
            ReviseLights(lights, semantic_table.light_ids, iter->color);
          }
          break;
        case base::TLColor::TL_UNKNOWN_COLOR:
        default:
          ReviseLights(lights, semantic_table.light_ids, iter->color);
          break;
      }
    } else {
      iter->time_stamp = time_stamp;
      iter->color = cur_color;
    }

    // set blink status
    if (pre_color != iter->color ||
        fabs(iter->last_dark_time_stamp - iter->last_bright_time_stamp) >
            non_blink_threshold_s_) {
      iter->blink = false;
    }

    for (auto index : semantic_table.light_ids) {
      lights_ref[index]->status.blink =
          (iter->blink && iter->color == base::TLColor::TL_GREEN);
    }
    ADEBUG << "semantic " << semantic_table.semantic << " color "
           << s_color_strs[iter->color] << " blink " << iter->blink << " cur "
           << s_color_strs[cur_color];
    ADEBUG << "cur ts " << std::to_string(time_stamp);
    ADEBUG << "bri ts " << std::to_string(iter->last_bright_time_stamp);
    ADEBUG << "dar ts " << std::to_string(iter->last_dark_time_stamp);
  } else {
    semantic_table.last_dark_time_stamp = semantic_table.time_stamp;
    semantic_table.last_bright_time_stamp = semantic_table.time_stamp;
    history_semantic_.push_back(semantic_table);
  }
}
```
