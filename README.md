# 点云融合原理讲解  
## 点云的基本格式
![点云格式](docs/pointcloud_format.png)

	点云文件无论如何千变万化，核心数据都是x，y，z三个坐标值

## 数学基础
1.矩阵运算法则。假如有如下两个矩阵A和B，AB为A和B的矩阵相乘结果，则:  
![点云格式](docs/matA.svg) 
 
![点云格式](docs/matB.svg)  

![点云格式](docs/matC.svg)   

	
	矩阵相乘就是左矩阵的每一行和右矩阵的每一列对应相乘再求和。矩阵能相乘的条件就是左矩阵的列数要和右矩阵行数相等。
	
2.三角函数的基本性质:  
![点云格式](docs/sanjiao.png)   
几个重要的结论：

	sin(α + π/2) = cos(α)
	cos(α + π/2) = -sin(α)
	sin(α - π) = -sin(α)
	cos(α - π) = cos(α)
	sin(α+β)=sin(α)cos(β)+cos(α)sin(β)
	cos(α+β)=cos(α)cos(β)-sin(α)sin(β) 
	
## 平移和旋转理论基础  
一、仅仅有平移的时候  
假设三维空间内存在某个点P1(X1, Y1, Z1)，该点发生平移之后到了P2(X2, Y2, Z2)，在每一个方向上发生的平移分量分别为dx, dy, dz，则容易理解，有以下两个矩阵的关系成立：
A = [X1, Y1, Z1, 1],  B = [X2, Y2, Z2, 1]

![点云格式](docs/translation.png)   
二、仅仅有旋转的时候
在平面直角坐标系里面，假设有一点A(x, y) 绕原点B逆时针旋转β度，得到C点

![点云格式](docs/rotate.png)   

1.设A点旋转前的角度为δ，则旋转(逆时针)到C点后角度为δ+β  
2.求A，B两点的距离：dist1=|AB|=y/sin(δ)=x/cos(δ)  
3.求C，B两点的距离：dist2=|CB|=d/sin(δ+β)=c/cos(δ+β)  
4.显然dist1=dist2，设dist1=r所以：   

    r=x/cos(δ)=y/sin(δ)=d/sin(δ+β)=c/cos(δ+β) 

5.由三角函数两角和差公式知：

	 sin(δ+β)=sin(δ)cos(β)+cos(δ)sin(β)
	 cos(δ+β)=cos(δ)cos(β)-sin(δ)sin(β) 

所以得出：   

	c=rcos(δ+β)=rcos(δ)cos(β)-rsin(δ)sin(β)=xcos(β)-ysin(β)
   	d=rsin(δ+β)=rsin(δ)cos(β)+rcos(δ)sin(β)=ycos(β)+xsin(β) 
   	
>上述结论的β角是一个大于0度的角， 为了避免欧拉角带来万向锁死锁的问题，需要将旋转角度做约束(β限定在(-pi, pi]区间内)，具体关于万向锁的问题可以参考：https://zhuanlan.zhihu.com/p/346718090

将β限定在(-pi, pi]区间内，根据sin(),cos()的奇偶性，sin(-β)=-sin(β),cos(-β)=cos(β)，则：	

	c=xcos(β)+ysin(β)
	d=ycos(β)-xsin(β)

即：

![点云格式](docs/rotateMat.png)   

为了计算方便，我们将上述结果做变形，并将二维平面推广到3维，以及考虑三个轴的旋转问题，得到如下的结论：  
a)平移

![点云格式](docs/transMat.png)  
b) 旋转

![点云格式](docs/roateMat2.png)   

R矩阵即为旋转矩阵，具体为三个方向的旋转矩阵的乘积：  

![点云格式](docs/R.png)   

三、综合来看，某个点在三维世界里的平移旋转可以拆分为这样一个过程：该3D点先进行旋转操作，再进行平移操作(这里说的旋转，指的是绕原点的旋转，所以并不能做先平移再旋转的操作，旋转分量会改变)。如果将平移矩阵记做T，将旋转矩阵记做R，则这个过程可以表示为：

	P2 = T * R * P1
	T * R的结果就是我们日常所说的外部参数
	
## 点云融合原理
自动驾驶点云采集设备：

![点云格式](docs/car.png)
原理基本：

![点云格式](docs/touying.png)

点云融合(投影)，基本原理是将3D世界坐标系下的点投影到2D图片上，具体拆分为以下几个步骤：  
a)因为雷达和相机在物理摆放的时候不能做到位置上和方向上的精确重叠，所以要将雷达采集到的数据做一个平移旋转(刚体)变换，这个用于描述冲雷达到相机坐标系的变换的矩阵称作外部参数，外部参数描述了相机将以何种方式去看雷达采集到的3D点，也体现了相机和雷达之间的相对位置关系。具体推到原理在上一节已经说明。  
b)相机看到的3D点通过镜头由小孔成像原理投影到图片上。这个小孔成像的变化过程，也可以由一个矩阵表明，这个矩阵是由相机本身决定的，相机的焦距，成像的图片尺寸等等均会影响这个变化过程。具体可以百度"张氏标定法"，里面详细介绍了这个过程。

[张氏标定法](https://www.cnblogs.com/wangguchangqing/p/8335131.html)

>这里提到的小孔成像的变换矩阵即是内部参数

K矩阵之和相机自身的特性相关

![点云格式](docs/K.png)  
fx，fy 表示将相机的焦距𝑓变换为在x,y方向上像素度量表示；cx, cy 表示成像光心坐标点，和成像图像大小有关系。  

c)由于相机的制造工艺不能完全符合理论值的预期，所以投影到图片上的点会发生偏移，为了修正这个问题，引入了畸变参数，来将投影点修正到理论范围内。畸变参数同样是由相机本身决定的，在计算内参的时候一般会附带给出。去除畸变的实质就是对像素点坐标做加加减减的线性计算。

## 总结：  

![点云格式](docs/Res.png)

## 点云融合实例代码讲解  
1.云融合示例  
2.3D框投影示例    

## 项目运行指南

### 环境要求

#### 系统要求
- **操作系统**: macOS, Linux, Windows
- **Python版本**: Python 3.6+
- **C++编译器**: GCC 7+, Clang 5+, MSVC 2017+

#### 依赖库

**Python依赖:**
```bash
pip install numpy
pip install Pillow
```

**C++依赖:**
- PCL (Point Cloud Library) 1.8+
- Eigen3
- CMake 3.5+
- OpenGL (用于3D可视化)

### 环境安装

#### macOS 环境安装

1. **安装Homebrew** (如果未安装):
```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

2. **安装Python依赖**:
```bash
pip3 install numpy Pillow
```

3. **安装C++依赖**:
```bash
# 安装PCL库
brew install pcl

# 安装CMake
brew install cmake

# 安装Eigen3
brew install eigen
```

#### Linux 环境安装 (Ubuntu/Debian)

1. **安装Python依赖**:
```bash
sudo apt update
sudo apt install python3-pip
pip3 install numpy Pillow
```

2. **安装C++依赖**:
```bash
sudo apt install libpcl-dev
sudo apt install cmake
sudo apt install libeigen3-dev
```

#### Windows 环境安装

1. **安装Python依赖**:
```bash
pip install numpy Pillow
```

2. **安装C++依赖**:
- 下载并安装 [PCL All-in-One](https://github.com/PointCloudLibrary/pcl/releases)
- 下载并安装 [CMake](https://cmake.org/download/)
- 下载并安装 [Visual Studio](https://visualstudio.microsoft.com/) (包含MSVC编译器)

### 模块运行指南

#### 1. 旋转和平移模块 (`旋转和平移/`)

**功能**: 演示3D点的旋转和平移变换，计算外部参数矩阵

**运行方式**:
```bash
cd 旋转和平移
python3 demo.py
```

**输出**: 显示计算得到的变换矩阵

#### 2. 点云投影过程模块 (`点云投影过程/`)

**功能**: 将3D点云投影到2D图像上，可视化3D边界框在图像上的投影

**运行方式**:
```bash
cd 点云投影过程
python3 demo.py 0_1595574171.166545.jpeg label.json
```

**参数说明**:
- `0_1595574171.166545.jpeg`: 输入图像文件
- `label.json`: 包含3D标注信息的JSON文件

**输出**: 显示带有3D边界框投影的图像

#### 3. 融合参数计算过程模块 (`融合参数计算过程/`)

**功能**: 计算点云融合的变换矩阵，将点云数据投影到图像上并显示

**运行方式**:
```bash
cd 融合参数计算过程
python3 demo.py
```

**输出**: 
- 控制台输出变换矩阵
- 显示带有投影点云点的图像

#### 4. 辅助演示使用模块 (`辅助演示使用/`)

**功能**: 使用PCL库进行3D可视化，展示坐标系和3D框

**运行方式**:

1. **编译项目**:
```bash
cd 辅助演示使用
mkdir -p build && cd build
cmake ..
make
```

2. **运行程序**:
```bash
./main
```

**输出**: 打开3D可视化窗口，显示:
- 三个坐标轴箭头 (红色X轴、绿色Y轴、蓝色Z轴)
- 3D立方体框 (线框模式)
- 交互式3D界面

**操作说明**:
- 鼠标左键: 旋转视角
- 鼠标右键: 平移视角
- 鼠标滚轮: 缩放
- 按 `q` 或关闭窗口退出

### 故障排除

#### 常见问题

1. **PCL库找不到**
   - 确保正确安装了PCL库
   - 检查CMakeLists.txt中的PCL版本要求

2. **编译错误**
   - 确保CMake版本 >= 3.5
   - 检查所有依赖库是否正确安装

3. **Python模块导入错误**
   - 确保安装了所需的Python包: `pip install numpy Pillow`

4. **3D窗口无法显示**
   - 确保系统支持OpenGL
   - 检查显卡驱动是否最新

#### 调试技巧

1. **查看详细编译信息**:
```bash
make VERBOSE=1
```

2. **检查PCL安装**:
```bash
pkg-config --modversion pcl_common
```

3. **检查Python包版本**:
```bash
python3 -c "import numpy; print(numpy.__version__)"
python3 -c "import PIL; print(PIL.__version__)"
```

### 项目结构说明

```
fusion/
├── README.md                    # 项目说明文档
├── 旋转和平移/                  # 旋转平移变换演示
│   ├── demo.py                 # Python演示代码
│   ├── main.cpp                # C++演示代码
│   └── CMakeLists.txt          # CMake构建文件
├── 点云投影过程/                # 点云投影演示
│   ├── demo.py                 # 投影演示代码
│   ├── label.json              # 3D标注数据
│   └── 0_1595574171.166545.jpeg # 示例图像
├── 融合参数计算过程/            # 融合参数计算
│   ├── demo.py                 # 参数计算代码
│   ├── readme.txt              # 参数说明
│   └── 0_1595574171.166545.txt # 点云数据
├── 辅助演示使用/                # 3D可视化演示
│   ├── main.cpp                # 可视化代码
│   └── CMakeLists.txt          # CMake构建文件
└── docs/                       # 文档图片
    ├── pointcloud_format.png   # 点云格式说明
    ├── car.png                 # 设备示意图
    └── ...                     # 其他说明图片
```

### 学习建议

1. **按顺序学习**: 建议按照模块顺序学习，从基础理论到实际应用
2. **理论结合实践**: 先理解README中的理论部分，再运行对应的演示代码
3. **参数调优**: 尝试修改代码中的参数，观察结果变化
4. **扩展应用**: 基于现有代码，尝试应用到自己的项目中

### 参考资料

- [PCL官方文档](https://pointclouds.org/documentation/)
- [张氏标定法详解](https://www.cnblogs.com/wangguchangqing/p/8335131.html)
- [欧拉角万向锁问题](https://zhuanlan.zhihu.com/p/346718090)
- [CMake官方文档](https://cmake.org/documentation/)
