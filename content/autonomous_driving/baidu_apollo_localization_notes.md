---
title: "Baidu Apollo 点云融合定位"
date: 2019-11-09 00:52
---

参考 [Apollo2.0多传感器融合定位模块解析](http://www.sohu.com/a/228425367_391994) 及 Github 上 Apollo 5.0 版本代码。

[TOC]

---

## 无人车定位技术概览

无人车需要精确的定位，精确是指厘米级，也就是说 **10厘米** 以内。

无人车定位指标要求

- 精度：平均误差 < 10cm
- 鲁棒性：最大误差 < 30cm
- 场景：全天候场景

定位的作用

- 定位系统可以与高精地图配合提供 **静态场景感知**
- 可将感知得到的动态物体正确放入静态场景
- **位置和姿态用于路径规划和车辆控制**

无人车定位常用坐标系

- 地心惯性坐标系（ECI）
- 地心地固坐标系（ECEF）
- 当地水平坐标系（e.g. ENU，导航坐标系，N系）
- 通用横轴墨卡托投影坐标系（UTM 坐标系，坐标 x,y 加上投影带编号标识一个点）
- 车体坐标系，RFU（右前上）
- IMU 坐标系，基本和车体坐标系一致（考虑安装误差，标定）
- 相机坐标系
- 激光雷达坐标系

定位的挑战

- 高精度、高鲁棒性需求
- GPS信号遇到阻隔会引起信号丢失
- 在复杂的城市环境中，由于建筑物和植物的存在，引起多镜效应导致定位不准
- 由于天气情况，或者人为修缮，会导致定位精度不高

定位技术分类

- 基于信号(GNSS 等)
- 航迹推算(IMU, Odometry 等)
- 环境特征匹配(Lidar, Camera 等)

## 不同定位技术简介

### 基于信号的定位(GNSS/RTK)

基于信号的定位，它的代表就是GNSS，其实就是全球导航卫星系统。

所谓 RTK ，指的是载波相位差分技术（Real time kinematic），实时处理两个测量站载波相位观测量的差分方法，将基准站采集的载波相位发给用户接收机，进行求差解算坐标。

- 差分技术，是在一个精确的已知位置（基站）上安装GNSS监测接收机，计算得到基站与GNSS卫星的距离校正
- RTK差分可以达到厘米级定位
- 缺点：基站铺设成本较高；非常依赖卫星数量，比如在一些桥洞和高楼大厦的环境下，可视的卫星数量会急剧下降；容易受到电磁环境干扰。在受到遮挡时，信号丢失，没有办法做定位。

百度为什么要自己实现 GNSS/RTK：

> 现在惯导芯片一般都会配一个板卡（NovAtel也卖这样的板卡），直接集成了RTK的定位结果。为什么我们还需要自己开发GNSS-RTK呢？ **从系统的角度考虑，需要每个子模块都是可控的** ，举一个简单的例子，当给出一个定位结果偏了，但给出的方差很小，也就是置信度很高。我们是没办法知道原因的。

### 航迹推算（IMU，Odometry 等）

航迹推算，依靠IMU等，根据上一时刻的位置和方位推断现在的位置和方位。

捷联惯性导航（ SINS, strapdown inertial navigation system ）

- 利用惯性测量元件（陀螺仪、加速度计）测量得到的载体相对于惯性空间角运动和线运动参数，经过惯性导航解算得到载体的速度、位置、姿态
- 惯性导航（简称惯导）的基本工作原理是以牛顿力学定律为基础。惯导系统的价格很高。惯导系统通过陀螺仪获知航向和姿态角，通过加速度得到速度，进而形成导航坐标系；通过速度得到位移，可以得到六自由度的信息。
- 优点是输出频率非常高，短时精度高
- 缺点是误差随着时间累积

### 环境特征匹配（Lidar点云定位）

基于Lidar的定位，用我们观测到的特征和数据库里的特征和存储的特征进行匹配，得到现在车的位置和姿态。

- 预先制作定位地图，然后用车上的实时点云和地图进行匹配，来计算激光雷达的位置和姿态，再通过激光雷达与IMU之间的外参，得到IMU的位置和姿态。
- 匹配有很多种方法，可以是基于3D点云匹配的ICP方法，也可以是我们这里给出的基于2D概率地图的直方图滤波器匹配定位。
- 激光定位的优点是在没有GPS情况下可以工作，鲁棒性比较好
- 缺点就是需要预先制作地图，同时要定期更新地图（因为环境会发生变化），雨雪天气也会受到影响（因为Lidar被折射的比较多，收到的点云数据变少）

## Baidu MSF(多传感器融合) 定位

Multi-Sensor Fusion.

定位实现了多传感器融合定位，既做到优势互补，也提高了稳定性，增强了定位精度。

### 点云定位

输入

- Lidar 点云
- 融合定位提供的预测位置和姿态
- 点云定位地图

输出

- [x, y, z, yaw]

点云定位主要包括 "XY 计算" 和 "Yaw 优化"，Z 是通过定位地图来获取的。

Yaw优化用到了图像对齐（LK光流图像配准 forward addtive 方法），XY计算用了直方图滤波器（2D Histogram Filter Localization）。

#### XY 计算

XY 计算使用 SSD-HF(Sum of Squared Difference Histogram Filter)。

SSD 把每个激光点的反射值或者高度值和定位地图对应值相减，然后把差的平方加起来，每个度量值越小说明位置匹配的越好。

怎么匹配多个位置？我们用二维直方图滤波器，将其中心放在预测姿态(x0, y0)，滤波器一般选择21*21，那么搜索的范围就是2.6米* 2.6米，需要计算441个位置的SSD值。

#### Yaw 优化

在直方图滤波器计算前，我们会对yaw做一次优化。

Yaw的优化，即对航向角的优化。因为融合定位的时候很容易产生角度的误差，如果用低端的IMU更容易产生，尤其是最开始的时候误差非常大。在航向角有1.5度误差的时候，明显的看出来已经有较大偏差，为什么会有这么大的偏差？这个时候我们要做航向角的优化，如果不做，直接做SSD直方图分布，可以看到它会集中在一个区域，经过平均以后，位置大概在另一区域，和真值差了一米多，那就偏了。处理方法用的是图像处理Lucas-Kanade算法，主要是做图像的对齐。

#### 加权使用点云反射值和高度值

> 对于反射值和高度值，做匹配的时候怎么联合起来？比如：左边有一个路段，反射值和匹配值经过平均以后，得到的位置会偏的很多，这时高度值定位非常好。用固定的方法加在一块，仍然得不到很好的分布，所以我们采用了自适应融合的方式，也就是说 **反射值和高度值分别计算直方图分布，由直方图分布的优劣来决定其权重** 。

> 我再举另一个例子，这是真实的路测例子，在扫描地图的时候路是这样的，后来他们重新铺设了，定位的时候只用反射值就会有误差。红色的是真值，蓝色是定位的结果。如下图左边所示，红色蓝色偏离的很厉害，这是非常危险的。调成固定权重的模式会好一点，如果采用自适应权重，则非常吻合。

### 惯性导航及融合定位

接下来我们介绍惯性导航及融合定位。

我们来解释一下导航解算的原理，它分下面几步：

1. 姿态更新，对陀螺仪输出的角速度进行积分得到姿态增量，叠加到上次的姿态上；
2. 比力坐标转换，是从IMU载体坐标系到位置、速度求解坐标系（惯性坐标系）；
3. 速度更新，速度这一步需要考虑重力加速度的去除，得到惯性系下的加速度，通过积分得到速度；
4. 位置更新，通过积分得到位置。

在惯性导航中，提到了导航方程的每一次迭代都需要利用上一次的导航结果作为初始值，因此在 **使用惯导之前必须进行初始化** 。初始位置和初始速度信息需要外部提供，一般是GNSS。

姿态对准是指得到IMU的roll, pitch, yaw；roll, pitch的对准过程一般称为调平：当车静止时，加速度计测量的比力仅由重力导致，可以通过`f=C*g`来求解；对于非常高精度的IMU可通过罗经对准的方式，车静止，通过测量载体系中的地球自转来确定载体的方位(yaw)。

对于车上使用的IMU，没那么高等级，一般采用两种方式：车直线跑起来，用从GNSS获取的速度方向来估计航向；第二种方式是 **采用双天线GNSS，通过两个位置连线来计算航向** 。

介绍完GNSS定位、点云定位以及惯性导航解算以后，我们看这几个模块怎么融合到一起。这里使用了Kalman滤波器的 **松耦合** ，意思就是我们只用了位置、姿态和速度做融合。

我们使用松耦合的方式把惯性导航解算、GNSS定位、点云定位三个子模块融合在一起。松耦合的数据只有 **位置、速度、姿态** ，紧耦合会包括GNSS的导航参数、定位中的伪距、距离变化等。

我们使用了一个 **误差卡尔曼滤波器**(error state KF) ，惯性导航解算的结果用于kalman滤波器的时间更新，也就是预测；而GNSS、点云定位结果用于kalman滤波器的测量更新。

Kalman滤波会输出位置、速度、姿态的误差用来修正惯导模块，IMU期间误差用来补偿IMU原始数据。

这里单独把我们所做的Kalman滤波拿出来，如果做GNSS定位输出和点云定位的时候，我们会发现GNSS定位非常快，只要有原始消息就可以快速解算，但是点云定位的时间比较长，需要几十毫秒甚至上百毫秒，会出问题。也就是我们先收到的消息解算时间很长，才会有结果， **会造成量测更新的乱序** 。我们解决这个问题的思路是用了 **两个滤波器** ，一个滤波器是Filter1，主要是做 **时间更新** ，实时的对外提供服务；量测更新是Filter2，我们会把Filter1的状态拷贝过来做 **量测更新** ， **还会补齐最新时刻，用的是时间更新的方式** ，最后用Filter2代替Filter1，这样就解决了时间戳乱序的问题。

相关论文：

> [Robust and Precise Vehicle Localization based on Multi-sensor Fusion in Diverse City Scenes](https://arxiv.org/abs/1711.05805)

---

## Baidu Apollo MSF 定位论文笔记

Robust and Precise Vehicle Localization based on Multi-sensor Fusion in Diverse City Scenes

high localization accuracy and resilience(意思是能够快速恢复，有韧性、弹性)

centimeter-level (平均 10cm 误差)

GNSS Lidar IMU 融合

对于 Lidar，使用反射值(intensity)和高度(altitude)线索。

An error-state Kalman filter is applied to fuse the localization measurements from different sources with novel uncertainty estimation.

### Intro

单点 GNSS 误差会达到 10 米，难以提供精确定位。

The carrier-phase based differential GNSS technique, known as Real Time Kinematic (RTK), can provide centimeter positioning accuracy.

RTK，误差为厘米级别。

**LiDAR** works well when the environment is full of 3D or **texture features**, while **RTK** performs excellently in **open space** .

An inertial measurement unit (IMU), including the gyroscopes and the accelerometers, continuously calculate the position, orientation, and velocity via the technology that is commonly referred to the dead reckoning. But it suffers badly from **integration drift**.

### Lidar Map Generation

obtain a grid-cell representation of the environment. Each cell stores the **statistics** of **laser reflection intensity and altitude**.

use single Gaussian distribution to model the environment but involve both the intensity and the altitude.

combining both the intensity and the altitude measurements in the cost function through an **adaptive weighting method** can significantly improve the localization accuracy and robustness.

Lidar based localization

(x, y, a, h), a: altitude from HDMap; h: heading(yaw)

#### A. Heading Angle Estimation

project the online point cloud onto the ground plane, and an intensity image is generate, but the pixels filled with the intensity values in the image are thinly dispersed due to the **sparsity** of the online point cloud.

During the heading angle h estimation step, Lucas-Kanade algorithm is applied, which is a technique that uses the image gradient to search for the best match between two images. use the forwards additive algorithm.

Although the Lucas-Kanade algorithm can produce the horizontal translation estimation, we found that it is not accurate and robust enough in practice.

#### B. Horizontal Localization

Histogram filter is one of nonparametric approaches. It approximates the posteriors by decomposing the state space into finitely many regions, and representing the cumulative posterior for each region by a single probability value.

关于直方图滤波，最大化后验概率，通过取负对数技巧，可以转为求最小 SSD。

公式 (6) 里对于反射值的 SSD_{r} ，可以看成用两个标准差进行加权。The impact of the environment change is implicitly diminished by the variance term.

### GNSS based localization

这一块内容不懂。

### Sensor Fusion

an error-state Kalman filter is applied to fuse the localization measurements(Lidar, RTK) with IMU.

The fusion framework can optimally combine the orientation rate and accelerometer information from IMU, for improved accuracy. IMU is sufficiently accurate to provide robust state estimates between LiDAR and RTK measurements.

这里涉及到一系列 IMU 相关的方程。

#### A. SINS Kinematics Equation and Error Equation

A Strap-down Inertial Navigation System (SINS) estimates the position, velocity and attitude by integrating the IMU data.

#### B. Filter State Equation

Because the SINS error grows over time, to get a precise PVA, we use an error-state Kalman filter to estimate the error of the SINS and use the estimated error-state to correct the SINS.

#### C. Filter Measurement Update Equation

The measurement update comprises the LiDAR and GNSS parts.

#### D. Delay Handling

Due to the transmission and computation delay, the measurement delay and disorder must be taken into consideration.

用两个 kf，kf1 只用来做 IMU 的预测和校正，kf2 用来处理观测到达时间和实际观测时间乱序的情况。

### Experimental result

数据集构建：开放场景用 NovAtel Inertial Explorer 等后处理提升精度；复杂场景，综合使用 NovAtel IE post-processing, LiDAR SLAM, loop closure, and the global pose-graph optimization

两种 modes:

- 2-Systems:LiDAR + IMU
- 3-Systems: LiDAR + GNSS + IMU.

3-systems 精度最高，2-systems 稍微差一点，定位误差 0.3m 内能达到 99%

平均定位误差 5~10cm

Lidar: 10Hz
GNSS: 5Hz
SINS: 200Hz

Single CPU core 版本实现里，通过用更小的 histogram filter size 和对图像数据降采样来加速 yaw 估计。

### Conclusion and future work

未来工作考虑更低成本的传感器（如 low-cost, low-end MEMS IMU）。
