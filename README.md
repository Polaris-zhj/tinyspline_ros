tinyspline_ros库中提供B样条功能，可以对多个离散点进行平滑。其使用涉及到几个函数，下面一一介绍，并举例说明如何使用tinyspline_ros库进行离散点平滑。

tinyspline_ros库的开源地址：**[tinyspline](https://github.com/msteinbeck/tinyspline)**

1、BSpline的构造函数

```C++
BSpline(size_t nCtrlp, size_t dim = 2, size_t deg = 3,
		tinyspline::BSpline::type type = TS_CLAMPED);
```

BSpline 是一个表示 B 样条曲线的类。B 样条曲线在计算机图形学和几何建模中非常有用。以下是您给出的 BSpline构造函数参数的解释：

- **nCtrlp**: 这是控制点的数量。控制点是定义 B 样条曲线的关键点。
- **dim**: 这是空间维度。对于二维图形，它的值通常是2；对于三维图形，它的值通常是3。
- **deg**: 这是 B 样条曲线的度或阶数。常见的 B 样条曲线度为3，但可以根据需要选择其他值。度决定了曲线在各个方向上的弯曲程度。
- **type**: 这是 B 样条曲线的类型。`TS_CLAMPED` 通常表示 "clamped" B 样条，其中第一个和最后一个控制点是曲线的一部分。还有其他类型的 B 样条，如 `TS_periodic`（周期性）或 `TS_open`（开放）。

这个构造函数用于初始化一个 B 样条对象，该对象使用给定的控制点、维度、阶数和类型来定义曲线。

举例：

```C++
// b_spline.cpp
#include "matplotlibcpp.h"
#include "tinyspline_ros/tinysplinecpp.h"
#include <cmath>
#include <iostream>
#include <vector>
namespace plt = matplotlibcpp;

int main() {
  std::vector<double> x_list = {1, 3, 5, 7, 9};
  std::vector<double> y_list = {5, 19, 15, 19, 24};
  double accumulate_length = 0.0;
  for (int i = 0; i < x_list.size(); ++i) {
    double dx = x_list[i + 1] - x_list[i];
    double dy = y_list[i + 1] - y_list[i];
    accumulate_length += std::hypot(dx, dy);
  }

  tinyspline::BSpline b_spline_raw(x_list.size(), 2, 3);
  std::vector<tinyspline::real> ctrlp_raw = b_spline_raw.controlPoints();
  for (size_t i = 0; i != x_list.size(); ++i) {
    ctrlp_raw[2 * i] = x_list[i];
    ctrlp_raw[2 * i + 1] = y_list[i];
  }
  b_spline_raw.setControlPoints(ctrlp_raw);
  double delta_t = 1.0 / (2 * accumulate_length);
  double tem_t = 0.0;
  std::vector<double> result_x_list, result_y_list;
  while (tem_t <= 1) {
    auto result = b_spline_raw.eval(tem_t).result();
    result_x_list.emplace_back(result[0]);
    result_y_list.emplace_back(result[1]);
    tem_t += delta_t;
  }
  plt::figure(1);
  plt::plot(x_list, y_list, ".");
  plt::plot(result_x_list, result_y_list);
  plt::show();

  return 0;
}
```

```cmake
cmake_minimum_required(VERSION 3.0.2)
project(Bspline)

set(CMAKE_EXPORT_COMPILE_COMMANDS ON)
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# add_compile_options(-std=c++11)

file(GLOB_RECURSE PYTHON2.7_LIB "/usr/lib/python2.7/config-x86_64-linux-gnu/*.so") 
set(PYTHON2.7_INLCUDE_DIRS "/usr/include/python2.7") #指定头文件路径
include_directories(include
        ${PYTHON2.7_INLCUDE_DIRS}       #添加头文件到工程
)
find_package(catkin REQUIRED COMPONENTS
  roscpp
  rospy
  std_msgs
)
find_package(tinyspline_ros REQUIRED)
include_directories(SYSTEM ${EIGEN3_INCLUDE_DIR})

include_directories(
# include
  ${catkin_INCLUDE_DIRS}
  ${PROJECT_SOURCE_DIR}/../tinyspline_ros/include
)
#MPCExample
add_executable(b_spline src/b_spline.cpp)
target_link_libraries(b_spline ${PYTHON2.7_LIB}  tinyspline_ros_cpp)

```

package.xml内追加下面命令，因为上面代码编译依赖tinyspline_ros功能包

```xml
 <build_depend>tinyspline_ros</build_depend>
```

图像结果：

![image-20231219230425285](https://image-1314148267.cos.ap-nanjing.myqcloud.com/typora-img/image-20231219230425285.png)

目录结构：

![image-20231219230723901](https://image-1314148267.cos.ap-nanjing.myqcloud.com/typora-img/image-20231219230723901.png)

