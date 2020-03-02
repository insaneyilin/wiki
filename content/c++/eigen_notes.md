---
title: "Eigen C++ Library cheatsheet"
date: 2019-11-18 23:23
tag: c++
---

矩阵创建

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

属性、赋值等

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

子数组，子矩阵

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

矩阵、向量计算相关

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

解线性方程组

```cpp
// A x = b，求 x
x = A.ldlt().solve(b);  // A sym. p.s.d.    #include <Eigen/Cholesky>
x = A.llt() .solve(b);  // A sym. p.d.      #include <Eigen/Cholesky>
x = A.lu()  .solve(b);  // Stable and fast. #include <Eigen/LU>
x = A.qr()  .solve(b);  // No pivoting.     #include <Eigen/QR>
x = A.svd() .solve(b);  // Stable, slowest. #include <Eigen/SVD>
```

求特征值

```cpp
A.eigenvalues();
EigenSolver<Matrix3d> eig(A);
eig.eigenvalues();
eig.eigenvectors();
```

常量矩阵

```cpp
static const Eigen::Matrix3f A =
    (Eigen::Matrix3f() << 1, 2, 3,
                          4, 5, 6,
                          7, 8, 8).finished();
```

几何变换

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

