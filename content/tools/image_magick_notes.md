---
title: "ImageMagick -- 超级强大的命令行图像处理工具"
date: 2019-07-22 00:01
---

[TOC]

ImageMagick 真的太 NB 了，谁用谁知道。

[ImageMagick on Github](https://github.com/ImageMagick/ImageMagick)

![ImageMagick](https://camo.githubusercontent.com/acf344955c86740b774c626ab9e142af90397452/68747470733a2f2f696d6167656d616769636b2e6f72672f696d6167652f77697a6172642e706e67)


---

### Mac 下安装 ImageMagick

直接用 homebrew：

```
brew install imagemagick
```

---

### 常用命令

#### 图像格式转换

```
convert test.jpg test.png
```

#### 缩放

```
convert -resize 50% test.png result.png
```

#### 旋转

```
convert -rotate -90 test.png result.png
```

#### 裁剪

```
convert -crop 50x50+10+10 test.png result.png
```

#### 图片压缩

```
# 会覆盖原有图片
mogrify -quality 20 test.jpg
```

---

### 使用 Magick++ 库

Magick++ 是 ImageMagick 的 C++ API。

在我的 Mac (macOS High Sierra 10.13.6) 上，使用 homebrew 安装完 ImageMagick 就可以直接用 Magick++。

例子：

```cpp
// main.cc
#include <Magick++.h>

int main(int argc, char **argv) {
  //InitializeMagick(*argv);  // for Windows
  Magick::Image img("batman.jpg");
  img.magick("PNG");
  img.write("result.png");
  return 0;
}
```

CMakeLists.txt :

```
project(magick++_demo)
find_package(ImageMagick COMPONENTS Magick++)

include_directories(${ImageMagick_INCLUDE_DIRS})
add_executable(demo main.cc)
target_link_libraries(demo ${ImageMagick_LIBRARIES})
```
