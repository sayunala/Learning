

# 计算机基础读书笔记

[读书笔记](https://www.cnblogs.com/haosongGame/)

## 2 数学

### 2.1 积分两大要素
+ 操作函数
+ 定义域

### 2.2 密度函数
+ 密度总是某种比率
+ 密度函数返回密度

### 2.3 曲线与表面
在计算机图形学中至关重要。虎书主要讨论3D和2D曲线和曲面的基础知识
+ 2D隐式曲线  
使用隐函数表示曲线 **f**的值相当于海拔 ***f=0***表示曲线，将空间分为***f>0***,***f<0***,***f=0***。
+ 2D梯度


### 2.4 三角形
+ 2D重心坐标  
  平面中的任意一个点***p***都能由一个已知的三角形的三个顶点***a,b,c***表示:***p=αa+βb+γc***
  问题就转化为求**α，β，γ**。

+ 3D重心坐标

  **几何方法计算α，β，γ**

  可以将参数记录为距离三角形点对边的相对距离，从下图可以看到，参数β代表点P距离bb的对边AC的相对距离。如果β=1,则点P与点b相对ac的距离是一致的。因此可以取对边的隐式函数f(x,y)=Ax+By+C,可得任意点PP到直线距离为$ f(a,b)/\sqrt{A^2+B^2} $，这样就可以算出$β=f_ac(xp,yp)/f_ac(xb,yb)$。在按此方法正确计算β,γβ,γ后，可以得出α=1−β−γ。

  ![img](https://img2020.cnblogs.com/blog/2480851/202108/2480851-20210812192104090-1380108215.png)

  


### 2.5 蒙特卡洛积分
普通积分计算不方便,蒙特卡洛积分通常是随机点乘以常数的平均值
```Csharp
float shade = average(f(),半圆);//进行积分
float sum = 0.0;
int N=10000;
float f()
{
  dosometing;
  return floatvar;
}
for(int i = 0;i < N;i++)
{
  vec X=random_Vec_on_半圆()
  sum+=f(X);
}

float Average=sun/N;
```
如何从半圆上随机选取10000个点（x,y,z）
```Csharp
do{
  x = random(-1,1);
  y = random(-1,1);
  z = random(-1,1);

}while(x^2+y^2+z^2>1)

if(z<0) z = -z;
vec v=vector(x,y,z);
average(f(),domain)=integrate(f(),domain)/integrate(1,domain);//integrate(1,domain)是domain的面积对于半圆来说是2Π
integrate(f(),domain)=average(f(),domain)*integrate(1,domain);

```
#### 2.5.1 重要性采样
已知概率密度函数，进行蒙特卡洛积分如下***p()***为domain上的概率密度函数
```Csharp
integrate = average_of_nonuniform_samples(f()/p(),
domain)
```
**采样问题策略方法**

+ 找出被积函数和积分区域

+ 找到提取采样点***xi***的方法，确保能够计算出**PDF(概率密度函数)**，***p()***

+ 对采样出的每一个 ***xi*** 计算出平均的比率
  $$
  f(x_i)/p(x_i)
  $$

  

# 3 光栅图像

光栅图形是简单的2D数组，存储每个像素的像素值，通常存RGB三个数字。存储在存储器中的光栅图像可以通过使用存储图像中的**每个像素**来控制显示器的一个像素的**颜色**来显示。

除了使用光栅阵列之外还可以使用矢量图像进行显示。存储显示图像的**指令**，不存储所需要的**像素**。通常使用文本，图表等进行显示。



## 3.1 光栅设备

+ Output

  - 显示器
    + 透视：液晶显示LCD
    + 发射型：发光二极管LED

  + 硬拷贝

    + 二进制：喷墨式打印机

    + 连续色调  

+ Input

  - 2D阵列传感器：数码相机

  - 1D阵列传感器：激光扫描仪
### 3.1.1 显示器

+ **发射型：发光二极管**

![image-20220914172303730](C:\Users\27547\AppData\Roaming\Typora\typora-user-images\image-20220914172303730.png)![image-20220914172322406](C:\Users\27547\AppData\Roaming\Typora\typora-user-images\image-20220914172322406.png)

每一个像素由一个或多个发光二极管，光强取决于通过二极管的电流强度。

+ **透视型：液晶显示器**

液晶是一种特殊的材料，它的分子结构使它能够旋转通过液晶的光的偏振（polarzation），旋转的程度是可以通过电压进行调节。

+ 水平偏振膜

+ 垂直偏振膜

  设置的电压没有改变偏振，则像素关闭，无 光透过。电压设置为偏振90°则所有光都“逃逸”。液晶显示器的效果如下

  ![image-20220914173933526](C:\Users\27547\AppData\Roaming\Typora\typora-user-images\image-20220914173933526.png)

  ![image-20220914173943032](C:\Users\27547\AppData\Roaming\Typora\typora-user-images\image-20220914173943032.png)

  

所有显示设备都有固定的分辨率（resolution），这个分辨率由网格（grid）的大小（size）确定。对显示器和图像分辨率只是像素网格的尺寸。

### 3.1.2 硬拷贝设备

在纸上永久记录图像和在显示器上直接显示图像差别很大。许多打印机只能打印二进制图像，颜料在每个网格（grid）只有两种状态要么沉积要么不沉积。

+ 喷墨式打印机：分辨率取决于墨滴(drop)的大小和纸张前进的距离，如下图：

  ![image-20220916104159133](C:\Users\27547\AppData\Roaming\Typora\typora-user-images\image-20220916104159133.png)

- 热染料转移：如下图穿过（across）纸张方向的分辨率是固定的，沿着（along）纸张方向的分辨率由加热和冷却速率与纸张速度相比得到

  ![image-20220916104223143](C:\Users\27547\AppData\Roaming\Typora\typora-user-images\image-20220916104223143.png)

**dpi用来描述二进制设备，ppi用来描述连续色调设备**

### 3.1.3 输入设备

+ 非算法计算得出的图像需要通过光栅设备（扫描仪或者相机）进行获得。在3D场景渲染中照片也经常用于纹理贴图（texture map）。像通常的输出设备一样，输入设备基于传感器阵列（arrays of sensor）要对每一个像素进行光的度量（light measurement）。

+ 数码相机的图像传感器是一个具有光敏像素网格的半导体设备。场景的图像投影到传感器上，每个像素测量光能并将其转化为数字最终进入输出图像。一些相机使用三个独立的阵列或者独立的三层阵列，来测量RGB。分辨率由行像素阵列和列像素阵列决定。

+ 彩色扫描仪由 $3*n_x$的阵列，$n_x$是有穿过纸张的方向的像素的数量。

## 3.2 图像，像素以及几何

在物理世界中图像就是二维区域上的函数（images ar functions），把图像抽象为一个函数:
$$
I(x,y):R->V
$$
$R \subseteq R^2$是一个矩形区域。V是可能的像素值的集合。

栅格图像与连续图像的抽象概念有什么关系？

像素值是图像颜色的**局部平均**，称为图像的***点采样（point sample）***

![image-20220923095359902](C:\Users\27547\AppData\Roaming\Typora\typora-user-images\image-20220923095359902.png)



如上图所示，图形的矩形区域宽$n_x$,高$n_y$，像素位于每个网格(***grid***)的中心 。坐下角为坐标原点（0,0）。

PS：有些系统使用左上角作为坐标原点，$x$轴自上而下，$y$轴自左向右。

### 3.2.1 像素值(Pixel Values)

一个像素可能记录一个灰度值或者三个RGB值，如果利用浮点数记录则需要64位储存，若仅使用0−2550−255的范围则仅需要8位，容量缩小了八倍（如果对比32位浮点则可以缩小4倍）。我们通常将利用浮点值记录的方法称为HDR(high dynamic range)而利用整数的成为LDR(low dynamic range)。通常一个像素的值有以下分类，比较重要的是1位的灰度值，8位RGB（共24位）的彩色普通图片与16位或32位的高清HDR图片。

![img](https://img2020.cnblogs.com/blog/2480851/202108/2480851-20210814004047013-681145059.png)

缩小位数通常会导致两个问题，分别是clipping和banding。其中clipping指的是真实颜色的亮度或其他值大于被设置的最大值，导致真实颜色被像素值的最大值所取代，额外的亮度或其他值被切割（clip）掉了。另外banding指的是将真实值取整至最接近的像素值时，存在连续值变换为离散值的问题，当精度较低时会存在突兀的颜色变换线或者条状物（band）。这一现象在静态时难以观察，但在动画或者视频中较为明显。

### 3.2.2 显示器光强和γ

进行显示的强度转化
$$
displayed intensity = (maximum intensity)a^γ,
$$
显示器的输入亮度$V_{in}$和输出亮度$V_{out}$不是线性关系而是指数关系：
$$
V_{out}=V_{in}^γ
$$
因此对于$V_{out}=0.5$可以有：
$$
γ=ln0.5/ln{V_{in}}
$$
因此问题变成找$V_{in}$，$V_{in}$可以通过用户调节进行获取，计算α时一般让用户或测试员调整αα值，直到灰度图与黑白相间的棋盘图从远处看拥有类似的亮度，具体对比见下图。用户通过调节$V_{in}$，当觉得两张图亮度相同时确定此时$V_{out}$对应的$V_{in}$

![img](https://img2020.cnblogs.com/blog/2480851/202108/2480851-20210814005544391-348402574.png)

通过这样可以得到当用户给定$V_{out}$时，获得响应的$V_{in}$:
$$
V_{in}=V_{out}^{1/γ}
$$
则有最终显示光强为：
$$
intensity = (M)V_{in}^γ=(M)V_{out}
$$
目的是使得输入和输出比例变动一致。其实就是当用户输入0.4想要0.4倍的光强，将这个0.4倍转化成显示器需要的真正的$V_{in}$，之后得到一个0.4的输出强度。

因为亮度值通常依然利用0至255去代表i/255，最后通常会有256个不同的亮度。在需要高度掌握最终亮度的时候可能需要一一确认加以调整，并根据角度需求添加额外的变化。

### 3.2.3 RGB

三原色构成RGB方块，所有的RGB颜色是上面的点

![image-20221004161223774](C:\Users\27547\AppData\Roaming\Typora\typora-user-images\image-20221004161223774.png)

### 3.2.4 α混合

将前景图像覆盖到背景图像上使用如下公式，通常的设备会在RGB通道之外存在α通道，每个通道8bit非常符合计算机要求
$$
c=αc_f+(1-α)c_b
$$
![image-20221004163804677](C:\Users\27547\AppData\Roaming\Typora\typora-user-images\image-20221004163804677.png)

### 3.2.5 图像存储

+ jpeg. This lossy format compresses image blocks based on thresholds in the human visual system. This format works well for natural images

+ tiff. This format is most commonly used to hold binary images or losslessly compressed 8- or 16-bit RGB although many other options exist

+ ppm. This very simple lossless, uncompressed format is most often used for 8-bit RGB images although many options exist

+ png. This is a set of lossless formats with a good set of open source man agement tools

  ## 4 光线追踪

  [一叶斋 | SmallPT —— 99 行代码光线追踪解析 (xieguanglei.github.io)](https://xieguanglei.github.io/blog/post/ray-tracing-99-lines-code.html)

  ### 介绍

  **rendering is a process that takes as its input a set of objects and produces as its output an array of pixels。**

  object-order rendering

  ```cpp
  for each object do
  ```

  

  image-order rendering

  ```cpp
  for each pixel do
      
  ```

  

  ### 4.1 基本的光追算法

  + ray genation:根据相机几何计算每个像素的观察光线的原点和方向

  + ray intersection:查找与观察光线**相交**的**最近**物体（object）

  + shading:根据光线相交（ray intersection）计算出颜色

    ```cpp
    for each pixel do:
    	compute viewing ray
        find first object hit by ray and its surface normal n
        set pixel color to value computed from hit,point and n
    ```

    

  ### 4.2 perspective 透视

  #### 4.2.1透视投影

  近大远小

  ![image-20221024142641137](C:\Users\27547\AppData\Roaming\Typora\typora-user-images\image-20221024142641137.png)

  #### 4.2.2 平行投影

  ![image-20221024142650366](C:\Users\27547\AppData\Roaming\Typora\typora-user-images\image-20221024142650366.png)

  

  

  ### 4.3 计算视线

  视线表达式，e是眼睛（相机）所在点，s是目标点

  ![image-20221024143305025](C:\Users\27547\AppData\Roaming\Typora\typora-user-images\image-20221024143305025.png)
  $$
  p(t)=e+t(s-e)
  $$
  使用右手坐标系{u,v,w}，平行正交投影w总是与视线的方向相反，或者说视线的方向默认为-w。

  ![image-20221024144957670](C:\Users\27547\AppData\Roaming\Typora\typora-user-images\image-20221024144957670.png)

  ```cpp
  struct Ray {
      Vector3 o;
      Vector3 d;
      float t;
  }
  ```

  

  #### 4.3.1 平行正交投影（Orthographic Views）

  对于所有的平行正交视图，所有的光线方向为 $-w$，尽管平行视线没有视点，但是为了能够模拟物体在相机后面，依然使用相机所在的平面作为光线开始。

  投影平面使用$u (x),v (y)$和$e$相机原点定义的平面作为投影平面，采用平面中间的点作为坐标原点e所在的位置。

  。本文使用$l<0<r,b<0<t$的横向与纵向位移为限制，位移单位为$u,v$的单位向量。我们的目的是将最终成像的大小$(n_x,n_y)$点阵图像素(i,j)转换到这个投影平面坐标(a,b)上:如下u=a,v=b。

  ![image-20221024151155763](C:\Users\27547\AppData\Roaming\Typora\typora-user-images\image-20221024151155763.png)

  ![image-20221024150654101](C:\Users\27547\AppData\Roaming\Typora\typora-user-images\image-20221024150654101.png)

  ```cpp
  Ray ray(e+au+bv,-w);
  
  ```

  $$
  ray.o<-e+au+bv
  $$

  $$
  ray.d<-w
  $$

  斜向的平行投影，将图像平面法线与w垂直，w由u，v构造，最终将**d**替换**-w**

  #### 4.3.2 透视投影

  所有的透视投影，所有的光线都有相同的起始点。

  ![image-20221024151623110](C:\Users\27547\AppData\Roaming\Typora\typora-user-images\image-20221024151623110.png)

```cpp
Ray perspectiveRay(o=e,d=-Dw+au+bv);
```

D为焦距，为e到投影平面的距离。
$$
ray.o=e
$$

$$
ray.d=-Dw+au+bv
$$

斜向透视投影同斜向平面投影一样。

### 4.4 Ray-Object Intersection(视线与物体相交)

#### 4.4.1 Ray-Sphere Intersection

球面方程
$$
f(x)=0
$$
设球心坐标为$c（x_c,y_c,z_c）$则有
$$
f(x)=(x-x_c)^2+(y-y_C)^2+(z-z_c)^2-R^2
$$
交点$p(x_p,y_p,z_p)$
$$
f(x)=(x_p-x_c)^2+(y_p-y_C)^2+(z_p-z_c)^2-R^2
$$

$$
f(\pmb{e}+t\pmb{d})=0
$$
向量表达式为：
$$
(p-c)·(p-c)=R^2
$$
带入向量之后有：
$$
(d · d)t^2 +2d · (e − c)t +(e − c) · (e − c) −R^2 =0.
$$
之后二元方程就行

#### 4.4.2 Ray-Triangle Intersection

光线采用参数方程进行表示如下
$$
\left.\begin{array}{l}
x_{e}+t x_{d}=f(u, v) \\
y_{e}+t y_{d}=g(u, v) \\
z_{e}+t z_{d}=h(u, v)
\end{array}\right\} \quad \text { or, } \quad \mathbf{e}+t \mathbf{d}=\mathbf{f}(u, v)
$$
三角形所在平面可以使用重心坐标进行如下表示：
$$
\mathbf{e}+t \mathbf{d} = \mathbf{a}+\beta(\mathbf{b}-\mathbf{a})+\gamma(\mathbf{c}-\mathbf{a})\\
$$
当且进当$\mathbf{\beta>0, \gamma>0 \text {, and } \beta+\gamma<1 \text {. }}$光线才与三角形相交，如果无解要么三角形退化(三点共线)，要么光线平行于包含三角形的平面。为为了求解 $t, \beta, \gamma$从向量形式转化为线性方程组，进而使用求解矩阵：
$$
\begin{aligned}
x_{e}+t x_{d} &=x_{a}+\beta\left(x_{b}-x_{a}\right)+\gamma\left(x_{c}-x_{a}\right) \\
y_{e}+t y_{d} &=y_{a}+\beta\left(y_{b}-y_{a}\right)+\gamma\left(y_{c}-y_{a}\right) \\
z_{e}+t z_{d} &=z_{a}+\beta\left(z_{b}-z_{a}\right)+\gamma\left(z_{c}-z_{a}\right)
\end{aligned}
$$

$$
\left[\begin{array}{lll}
x_{a}-x_{b} & x_{a}-x_{c} & x_{d} \\
y_{a}-y_{b} & y_{a}-y_{c} & y_{d} \\
z_{a}-z_{b} & z_{a}-z_{c} & z_{d}
\end{array}\right]\left[\begin{array}{l}
\beta \\
\gamma \\
t
\end{array}\right]=\left[\begin{array}{c}
x_{a}-x_{e} \\
y_{a}-y_{e} \\
z_{a}-z_{e}
\end{array}\right]
$$

求解上述矩阵使用**克拉默法则**如下：


$$
\beta=\frac{\left|\begin{array}{ccc}
x_{a}-x_{e} & x_{a}-x_{c} & x_{d} \\
y_{a}-y_{e} & y_{a}-y_{c} & y_{d} \\
z_{a}-z_{e} & z_{a}-z_{c} & z_{d}
\end{array}\right|}{|\mathbf{A}|},\\
\gamma=\frac{\left|\begin{array}{ccc}
x_{a}-x_{b} & x_{a}-x_{e} & x_{d} \\
y_{a}-y_{b} & y_{a}-y_{e} & y_{d} \\
z_{a}-z_{b} & z_{a}-z_{e} & z_{d}
\end{array}\right|}{|\mathbf{A}|},\\
t=\frac{\left|\begin{array}{ccc}
x_{a}-x_{b} & x_{a}-x_{c} & x_{a}-x_{e} \\
y_{a}-y_{b} & y_{a}-y_{c} & y_{a}-y_{e} \\
z_{a}-z_{b} & z_{a}-z_{c} & z_{a}-z_{e}
\end{array}\right|}{|\mathbf{A}|}\\
\mathbf{A}=\left[\begin{array}{lll}
x_{a}-x_{b} & x_{a}-x_{c} & x_{d} \\
y_{a}-y_{b} & y_{a}-y_{c} & y_{d} \\
z_{a}-z_{b} & z_{a}-z_{c} & z_{d}
\end{array}\right]
$$
进行光线求交的伪代码：

![image-20221024215845031](C:\Users\27547\AppData\Roaming\Typora\typora-user-images\image-20221024215845031.png)

#### 4.4.3 软件中的光追

```cpp
class HitRecord{
    Surface s// 光线命中的表面
    real t//沿着光线的t
    Vec3 n//命中点位置的面法线
    ......//纹理坐标，需要存储的切线向量
}
class Surface{
    virtual HitRecord hit(Ray r,real t0,real t1);
}
```

#### 4.4.4 光线与一组物体相交

```cpp
class Group:public Surface{
    Array<Surface> m_surfaces;//一组物体
    HitRecord hit(Ray r,real t0,real t1);
   	HitRecord closest_hit(){
        for (Surface& surf : m_surfaces ){
            HitRecord rec = surf.hit(ray,t0,t1);
            if(rec.t<max){
                HitRecord closest_hit = rec;
                t1=t;
            }
        }
        return closest_hit;
    }
}
```

### 4.5 着色（Shading）

#### 4.5.1 光源

为了着色，一个光追程序总是需要一系列光源。第五章的着色模型需要三种光源：

+ 点光源 ：从空间中的一点发射光线
+ 平行光 ：以单一的方向照明场景
+ 环境光 ：提供恒定的照明以填充阴影

更高级的系统需要其他光源：

+ area lights 区域灯光照明 : 它们基本上是发射光的场景几何体
+ environment lights ：它们使用图像来表示来自天空等遥远来源的光

需要几何信息来计算来自**点光源**或者**平行光源**的着色，在光追程序中，确定视线已经命中表面后我们需要确定四个向量：

- 着色点$x$，通过视线t计算出的交点值

+ 光线方向$l$通过计算 from the light source position or direction as part of shading。

+ 面法线$n$,取决于面的类型，每个需要能够计算视线交点的**法线**

+ 观察方向$v$与观察光线的方向相反（$v=-d/||d||$）

  从**环境光源**进行着色：

  因为光来自四面八方，因此没有$l$；着色不依赖于$v$；对于第五章的简单系统甚至不依赖于$x与n$。

  

  当一个场景中包含多个光源时，将各个光源对着色的贡献相加即可。

  #### 4.5.2 软件中的着色

  光追程序中包含**光源对象**和**材质对象**。

  需要 ***Light*** 类和 ***Material*** 类：

  + Light：负责整体照明计算（overall illumination computation）
  + Material：计算BRDF（双向面反射函数 ）

  ```cpp
  class Light{
  public:
      Color illuminate(Ray ray,HitRecord hrec);
  }
  class Material{
      virtual Color evaluate(Vec3 l,Vec3 v,Vec3 n);
  }
  ```

  每一个面应该存储**面材质的引用**

  ```cpp
  class PointLight : public Light{
      Color m_I;
      Vec3 m_p;//光源位置
      Color illuminate(Ray ray,HitRecord hrec){
  		Vec3 x = ray.evaluate(hrec.t);
  		real r = Math.abs(m_p-x);
   		Vec3 l = (m_p-x)/r;
   		Vec3 n = hrec.normal;
  		Color E = max(0,n·l)m_I/r^2;
  		Color k = hrec.surface.material.evaluate(l,v,n);
  		return KE;
      }
  }
  ```

  ```cpp
  class AmbientLight : public Light{
      Color I_a;
      Color illuminate(Ray ray, HitRecord hrec){
          Color k = hrec.surface.material.k;
          return kI_a; 
      }
  }
  ```

  #### 4.5.3 第四版Lighting

  + Lambertian Shading

  $$
  L=k_{d} I \max (0, \mathbf{n} \cdot \mathbf{l})
  $$

  1. $k_d$：是漫反射系数
  2. $I$ ：是光强
  3. $n$：着色点法线
  4. $l$：着色点到光源的向量

  + Blinn-Phong Shading

  $$
  \begin{aligned}
  \mathbf{h} &=\frac{\mathbf{v}+\mathbf{l}}{\|\mathbf{v}+\mathbf{l}\|}, \\
  L &=k_{d} I \max (0, \mathbf{n} \cdot \mathbf{l})+k_{s} I \max (0, \mathbf{n} \cdot \mathbf{h})^{p},
  \end{aligned}
  $$

  1. $k_s$：镜面反射系数

  2. $h$：如下

  3. $p$：冯式指数

     ![image-20221107143125799](F:\Markdown\Learning\CppLearning\image-20221107143125799.png)

+ 环境着色
  $$
  L=k_{a} I_{a}+k_{d} I \max (0, \mathbf{n} \cdot \mathbf{l})+k_{s} I \max (0, \mathbf{n} \cdot \mathbf{h})^{n},
  $$

  ### 4.6 光追程序

  

  ```cpp
  for each pixel do:
  	computer viewing ray;
  	if(ray hits an object with t in[0,∞]){
  		Computer n;//面法线
  		Evaluate shading model and set pixel to that color；
      }
  	else{
          set pixel color to background color;
      }
  	
  ```

  ### 4.7 阴影

  ```cpp
  color raycolor(Ray ray,real t0,real t1){
      HitRecord rec,srec;
      if(scene->hit(ray,t0,t1,rec))
      {
          Ray p=e+(rec.t).d;
          color c=rec.ka*Ia;
          if(not scene->hit(p+s*l,t0,t1,srec)){
              vec3 h=normalize(normalized(l)+normalized(v));
              c = c+rec.kd*I*Max(0,rec.n·l)+(rec.ks)·I·(rec.n,h)^p;
          }
          return c;
          
      }
      else return background_color;
  }
  ```

  ## 5 线性代数
  
  详情参照：[【官方双语/合集】线性代数的本质 - 系列合集_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1ys411472E/?spm_id_from=333.999.0.0)
  
  比较有意思的是奇异分解，这一块很有意思。

## 6 变换

### 6.1 旋转与缩放

#### 6.1.1 旋转与缩放矩阵



#### 6.1.2 任意旋转



#### 6.1.3 法线旋转

### 6.2 平移与仿射变换

#### 6.2.1 平移

#### 6.2.2 Windowing transformations

### 6.3 坐标变换





## 7 Viewing



