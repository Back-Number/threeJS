# webGL前言
>author:hqx

three.js的底层是基于webGL技术的，webGL使得浏览器可以直接通过调用GPU的功能创建三维图形，但其底层细节复杂，且需要学习复杂的着色器语言，Three.js则提供了一个很简单的关于WebGL特性的JavaScript API，帮助用户快速创建3D场景
## 图形学相关概念
webGL实际上是利用了GPU的可编程渲染管线来渲染图形到我们的浏览器，利用GLSL着色器语言，来自定义渲染内容。

### 点，线段和三角形
wenGL可以绘制`点，线段，三角形`三种类型的图像，点和线段就不多说了，最主要的是三角形，因为三角形是一个面，而面是构成3D图像的基本要素，可以想象一下一个球体，实际上可以看成很多个精细的面构成的多面体，近似一个球体。而为什么使用三角形呢？**因为三个点构成面会使用到最少的顶点坐标，减少了计算机的运算量**。同样地模拟一个球体，使用矩形会产生多得多的点坐标。  
且由点坐标形成图形的绘制顺序也是很重要的，比如三个点是按照顺时针还是逆时针来绘制三角形，计算机将利用这一点来判断我们看到的3D图像的正反面。  

tips：面的定义是三角形还是矩形历史上存在争议，在建模领域，矩形比三角形更容易增强和平滑，在计算机渲染器和游戏引擎领域，三角形渲染效率更高。

### 矩阵变换
webGL进行3D渲染的时候会准备三个坐标变化矩阵，分别对3D模型做对应的操作，他们分别是：
- 模型转换：  模型变换矩阵影响的是所绘制的模型，模型的位置，模型的旋转，模型的放大和缩小等相关的情况
- 视图转换：  简单来说，就是定义拍摄3D空间的镜头（摄像机），决定了镜头的位置，镜头的参考点，镜头的方向等
- 投影转换：  这个坐标变换定义了屏幕的横竖比例，剪切的领域等，另外获取远近法则的效果也需要用这个变换矩阵

### 矩阵
对于线性代数中的矩阵，主要运用在计算机图形学中，webGL就是利用GPU的渲染管线来进行渲染的一种技术，而GPU不同于CPU，CPU接收队列数据依序执行，而GPU一次性接受大量数据，再反应到我们的监视器像素之中。这里的大量数据，都是靠矩阵存储图形空间坐标信息传递给GPU，且矩阵的一些运算特性，可以很方便地处理图像的视角，纹理等信息


## 基本使用
一下内容为webGL的基本使用方式，其会调用GPU在页面上简单地绘制一个点，仅介绍基本工作流程
### 1.创建canvas画布
~~~
let canvas = document.getElementById('webgl'); 
let gl = canvas.getContext('webgl');
~~~

### 2.声明顶点着色器和片元着色器
~~~
    //顶点着色器
    let vertexShaderSource = `
            void main(){
                gl_PointSize=40.0;
                gl_Position =vec4(0.5,0.5,0.0,1.0);
            }
           `;

    //片元着色器源码
    var fragShaderSource = `
            void main(){
                gl_FragColor = vec4(0.0,0.0,1.0,1.0);
            }
            `;

~~~

>由于着色器都是GLSL着色器语言，JS无法直接运行他们，需要写在``内部

### 3.初始化着色器
两个着色器都是运行在GPU上的程序，在JS中需要我们手动编译执行他们到GPU
~~~
function initShader(gl, vertexShaderSource, fragmentShaderSource) {
      //创建顶点着色器和片元着色器对象
      let vertexShader = gl.createShader(gl.VERTEX_SHADER);
      let fragmentShader = gl.createShader(gl.FRAGMENT_SHADER);

      //引入顶点、片元着色器源代码
      gl.shaderSource(vertexShader, vertexShaderSource);
      gl.shaderSource(fragmentShader, fragmentShaderSource);

      //编译顶点、片元着色器
      gl.compileShader(vertexShader);
      gl.compileShader(fragmentShader);

      //创建程序对象program
      let program = gl.createProgram();

      //附着顶点着色器和片元着色器到program
      gl.attachShader(program, vertexShader);
      gl.attachShader(program, fragmentShader);

      //链接program
      gl.linkProgram(program);

      //使用program
      gl.useProgram(program);

      //返回程序program对象
      return program;
    }
~~~

初始化步骤如下：
- 初始传参，需要（WebGL上下文对象，顶点着色器源码，片元着色器源码）
- 由createShader()分别创建两个着色器对象
- 由shaderSource()为两个对象引入源码
- 由compileShader()编译两个着色器对象
- 由createProgram()创建程序
- 由attachShader()附着编译后的两个对象到程序
- 由linkProgram()链接程序
- 由useProgram()使用程序
- 将程序program返回

### 4.绘制
做完以上工作以后就可以通过drawArrays()方法进行绘制了，通过以上代码会简单地绘制一个点到浏览器页面上

## 顶点着色器和片元着色器
`顶点着色器`和`片元着色器`是GLSL最重要的部分，我们的webGL正是利用他们将我们想要渲染的图形信息传递到GPU。他们两者都缺一不可，顶点着色器负责描绘图形的空间点坐标信息，而片元着色器负责描绘图形的颜色信息

### 顶点着色器
其通常形式如下：
~~~
const vertexShaderSource = `
    attribute vec3 position; 
    void main() {
        gl_Position = vec4(position,1); 
    }
`
~~~
这是通过GLSL着色器语言编写的一个顶点着色器，和C语言很类似，一个void main()定义的主函数，声明了`gl_Position`的变量，其接收了一个`attribute`修饰的`vec`的类型参数
#### gl_Position变量
这个变量即为返回给GPU的顶点坐标数据，矩阵格式
#### 坐标变换
在webGL中首先会生成模型，视图，投影的各个矩阵，然后进行合并后返回给gl_Position，这样一个结合多种附加条件返回最终坐标的过程就是坐标变换
#### 修饰符
- `attribute`: 表示要传递的数据是各个顶点不同的数据
- `uniform`: 表示要传递的数据对于所有的顶点都是一样的，比如一些变换操作
#### GLSL类型
- `vec*`: 表示向量，*指代向量的维数，vec4即四维向量，内部元素为浮点型
- `mat*`: 表示方阵，*指代方阵的维数，内部元素为浮点型
#### 向片元着色器传递数据
使用`varying`关键字可以向片元着色器传递数据

### 顶点缓存VBO
webGL会将顶点坐标进行坐标变换最后返回给顶点着色器，则就需要对多个顶点坐标进行缓存，这样的一个存储空间就称之为顶点缓存(VBO),由此可见VBO中的数据都是独特的attribute类型，VBO的工作流程如下：
- 顶点的各种信息保存到数组里（限定一维数组）
- 使用WebGL的方法生成VBO
- 使用WebGL的方法将数组中的信息传给VBO
- 顶点着色器中的attribute函数和VBO结合
