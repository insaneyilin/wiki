---
title: "Eigen C++ Library cheatsheet"
date: 2019-11-18 23:23
tag: c++
---

[TOC]

---

## 旋转表示的转换

[使用eigen实现四元数、欧拉角、旋转矩阵、旋转向量间的转换](https://mp.weixin.qq.com/s?__biz=MzAwNTMzODc4OA==&mid=2456142984&idx=1&sn=8ddf7fe8f9761ff515cde43acdbd2211&chksm=8c8f4743bbf8ce55515fc53981f2adbfaec2aa76c264a120d81eebb94893be5cecd05e9e90f7&mpshare=1&scene=1&srcid=&sharer_sharetime=1588902724110&sharer_shareid=c0d8d647194ca8d8e74a6f38874f41b2&exportkey=AZ7XWRb0W1y7qwPcoKbiRrw%3D&pass_ticket=%2FfsLdT%2BkaMm1dxYkA01aDwUAfR0baNIRPg00%2BpWK1wdzHbiL7L3bW7BxAKQH4LQV#rd)

### 旋转向量

初始化旋转向量：旋转角为 `alpha`，旋转轴为 `(x,y,z)`：

```cpp
Eigen::AngleAxisd rotation_vector(alpha,Vector3d(x,y,z));
```

旋转向量转旋转矩阵：

```cpp
Eigen::Matrix3d rotation_matrix;
rotation_matrix=rotation_vector.matrix();

Eigen::Matrix3d rotation_matrix;
rotation_matrix=rotation_vector.toRotationMatrix();
```

旋转向量转欧拉角(Z-Y-X，即RPY)：

```cpp
Eigen::Vector3d eulerAngle=rotation_vector.matrix().eulerAngles(2,1,0);
```

旋转向量转四元数

```cpp
Eigen::Quaterniond quaternion(rotation_vector);
Eigen::Quaterniond quaternion;
quaternion=rotation_vector;
```

### 旋转矩阵

初始化旋转矩阵

```cpp
Eigen::Matrix3d rotation_matrix;
rotation_matrix<<x_00,x_01,x_02,x_10,x_11,x_12,x_20,x_21,x_22;
```

旋转矩阵转旋转向量

```cpp
Eigen::AngleAxisd rotation_vector(rotation_matrix);
Eigen::AngleAxisd rotation_vector;
rotation_vector=rotation_matrix;
Eigen::AngleAxisd rotation_vector;
rotation_vector.fromRotationMatrix(rotation_matrix);
```

旋转矩阵转欧拉角(Z-Y-X，即RPY)

```cpp
Eigen::Vector3d eulerAngle=rotation_matrix.eulerAngles(2,1,0);
```

旋转矩阵转四元数

```cpp
Eigen::Quaterniond quaternion(rotation_matrix);
Eigen::Quaterniond quaternion;
quaternion=rotation_matrix;
```

### 欧拉角

初始化欧拉角(Z-Y-X，即RPY)

```cpp
Eigen::Vector3d eulerAngle(yaw,pitch,roll);
```

欧拉角转旋转向量

```cpp
Eigen::AngleAxisd rollAngle(AngleAxisd(eulerAngle(2),Vector3d::UnitX()));
Eigen::AngleAxisd pitchAngle(AngleAxisd(eulerAngle(1),Vector3d::UnitY()));
Eigen::AngleAxisd yawAngle(AngleAxisd(eulerAngle(0),Vector3d::UnitZ())); 
Eigen::AngleAxisd rotation_vector;
rotation_vector=yawAngle*pitchAngle*rollAngle;
```

欧拉角转旋转矩阵

```cpp
Eigen::AngleAxisd rollAngle(AngleAxisd(eulerAngle(2),Vector3d::UnitX()));
Eigen::AngleAxisd pitchAngle(AngleAxisd(eulerAngle(1),Vector3d::UnitY()));
Eigen::AngleAxisd yawAngle(AngleAxisd(eulerAngle(0),Vector3d::UnitZ())); 
Eigen::Matrix3d rotation_matrix;
rotation_matrix=yawAngle*pitchAngle*rollAngle;
```

欧拉角转四元数

```cpp
Eigen::AngleAxisd rollAngle(AngleAxisd(eulerAngle(2),Vector3d::UnitX()));
Eigen::AngleAxisd pitchAngle(AngleAxisd(eulerAngle(1),Vector3d::UnitY()));
Eigen::AngleAxisd yawAngle(AngleAxisd(eulerAngle(0),Vector3d::UnitZ())); 
Eigen::Quaterniond quaternion;
quaternion=yawAngle*pitchAngle*rollAngle;
```

### 四元数

初始化四元数

```cpp
Eigen::Quaterniond quaternion(w,x,y,z);
```

四元数转旋转向量

```cpp
Eigen::AngleAxisd rotation_vector(quaternion);
Eigen::AngleAxisd rotation_vector;
rotation_vector=quaternion;
```

四元数转旋转矩阵

```cpp
Eigen::Matrix3d rotation_matrix;
rotation_matrix=quaternion.matrix();
Eigen::Matrix3d rotation_matrix;
rotation_matrix=quaternion.toRotationMatrix();
```

四元数转欧拉角(Z-Y-X，即RPY)

```cpp
Eigen::Vector3d eulerAngle=quaternion.matrix().eulerAngles(2,1,0);
```

---

## 矩阵创建

```cpp
#include <Eigen/Dense>

Eigen::Matrix<double, 3, 3> M1;  // 固定size
Eigen::Matrix<double, 3, Eigen::Dynamic> M2;  // 列长度不定
Eigen::Matrix<double, Eigen::Dynamic, Eigen::Dynamic> M3;  // 动态size
Eigen::Matrix<double, 3, 3, Eigen::RowMajor> M4;  // 行优先存储（默认是列优先）
Eigen::Matrix3f M5;
Vector3f x;  // 3x1
RowVector3f a;  // 1x3
VectorXd v;  // dynamic column vector
```

## 属性、赋值等

```cpp
#include <Eigen/Dense>

Eigen::Matrix3d A;
A.fill(5.0);   // A 中元素全部设置为 5.0

// 按照行列下标访问
A(0, 0);
A(0, 2);

// 行、列数
A.rows();
A.cols();

A << 1, 2, 3,
     4, 5, 6,
     7, 8, 9;  // 按照 row-major 顺序赋值（注意和存储方式不同）

Eigen::Matrix<double, 3, Eigen::Dynamic> B;
B << A, A, A;  // horizontally stacked
B.resize(3, 2);

// 单位阵
MatrixXd::Identity(5, 5);
Matrix3d::Identity();
B.setIdentity(3, 3);

// 类似的有全零矩阵 Zero()，随机矩阵 Random()
Matrix3d::Zero();
A.setZero();

// 线性增长数组
VectorXd::LinSpaced(size, low, high);
v.setLinSpaced(size, low, high);
VectorXi::LinSpaced(((hi - low) / step) + 1,
                    low, low + step * (size - 1));  // low:step:hi

Eigen::Vector3f v(1.f, 2.f, 3.f);
```

## 子数组，子矩阵

```cpp
Eigen::VectorXd v;
v.resize(10);

v.head(2);
v.head<2>();
v.tail(3);
v.tail<3>();
v.segment(1, 3);  // [v[1], v[2]]

MatrixXd A;
A.reize(10, 10);
A.block(i, j, rows, cols);  // 以 (i, j) 为左上角元素取一个 size 为 (rows, cols) 的子矩阵
A.row(i);  // 第 i 行
A.col(j);  // 第 j 列

// 转置
A.transpose();

// 逆矩阵
A.inverse();

// 求行列式值
A.determinant();
```

## 矩阵、向量计算相关

```cpp
// 矩阵乘法，矩阵向量乘法可以直接进行
Eigen::Matrix3f A;
Eigen::Matrix3f B;
Eigen::Matrix3f C;
C = A * B;

Eigen::Vector3f x;
A * x;

// 逐元素计算
C = A.cwiseProduct(B);

A.array().sin();
A.array().cos();

A.cwiseAbs();  // A 中所有元素取绝对值

A.minCoeff();  // A 中最小元素
A.maxCoeff();  // A 中最大元素

// 向量计算
Eigen::Vector3f x;
x.norm();  // 模长

Eigen::Vector3f y;
x.dot(y);
x.cross(y);

// 类型转换
A.cast<double>();
```

## 解线性方程组

```cpp
// A x = b，求 x
x = A.ldlt().solve(b);  // A sym. p.s.d.    #include <Eigen/Cholesky>
x = A.llt() .solve(b);  // A sym. p.d.      #include <Eigen/Cholesky>
x = A.lu()  .solve(b);  // Stable and fast. #include <Eigen/LU>
x = A.qr()  .solve(b);  // No pivoting.     #include <Eigen/QR>
x = A.svd() .solve(b);  // Stable, slowest. #include <Eigen/SVD>
```

## 求特征值

```cpp
A.eigenvalues();
EigenSolver<Matrix3d> eig(A);
eig.eigenvalues();
eig.eigenvectors();
```

## 常量矩阵

```cpp
static const Eigen::Matrix3f A =
    (Eigen::Matrix3f() << 1, 2, 3,
                          4, 5, 6,
                          7, 8, 8).finished();
```

## 几何变换

```cpp
// 四元数
Eigen::Quaterniond quatd;
quatd.setFromTwoVectors(Eigen::Vector3d(0, 1, 0), vec);

Eigen::Matrix3d rot_mat = quatd.toRotationMatrix();

Eigen::Quaterniond quatd(w, x, y, z);
quatd.normalize();
quatd.norm();

// 仿射变换
Eigen::Affine3d pose = Eigen::Affine3d::Identity();
pose.translation() = t;  // 3x1
pose.linear() = quatd.toRotationMatrix();  // 4x4

t = pose.translation();
rot_mat = pose.linear();

// 从四元数、平移向量构造 Affine3d
Eigen::Quaterniond q(1.0, 0.0, 0.0, 0.0);
Eigen::Vector3d t(1.0, 2.0, 3.0);
Eigen::Affine3d pose;
pose = Eigen::Translation3d(t) * q;  // 这里把 * 看成变换的复合
std::cout << "pose: " << pose.matrix();
```

