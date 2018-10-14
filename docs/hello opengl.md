# Hello OpenGL

## 什么是OpenGL？
- OpenGL本身并不是一个API，它仅仅是一个由Khronos组织制定并维护的规范(Specification)
- OpenGL规范严格规定了每个函数该如何执行，以及它们的输出值
- OpenGL的大多数实现都是由显卡厂商编写；Apple系统OpenGL库是由Apple自身维护的；Linux下有显卡生产商提供的OpenGL库，也有一些爱好者改编的版本
    - 当产生一个bug时通常可以通过升级显卡驱动是一个不错的选择。。。

### OpenGL学习资料
- OpenGL3.x学习卡片：https://www.slideshare.net/Khronos_Group/opengl-es-32-reference-guide
- OpenGL3.x API查询：https://www.khronos.org/registry/OpenGL-Refpages/es3.0/
- 所有OpenGL版本接口查询：http://docs.gl/
- LearnOpenGL网站工程：https://github.com/LearnOpenGL-CN/learnopengl-cn.github.io
    - 本地部署，方便阅读

## 核心模型与立即渲染模式
- 早期的OpenGL使用立即渲染模式（Immediate mode，也就是固定渲染管线）
- OpenGL3.2开始，规范文档开始废弃立即渲染模式，并鼓励开发者在OpenGL的核心模式(Core-profile)下进行开发，这个分支的规范完全移除了旧的特性
    - 当使用OpenGL的核心模式时，OpenGL迫使我们使用现代的函数。当我们试图使用一个已废弃的函数时，OpenGL会抛出一个错误并终止绘图
- 我们的教程面向OpenGL3.3的核心模式
    - 所有OpenGL的更高的版本都是在3.3的基础上，引入了额外的功能，并没有改动核心架构。新版本只是引入了一些更有效率或更有用的方式去完成同样的功能

## 扩展支持
- 当一个显卡公司提出一个新特性或者渲染上的大优化，通常会以扩展的方式在驱动中实现
    - 开发者不必等待一个新的OpenGL规范面世，就可以使用这些新的渲染特性了，只需要简单地检查一下显卡是否支持此扩展
    - 例如苹果的opengl驱动中，就把很多gles3.x支持的内容都在gles2.0中通过ARB扩展接口实现
    - ==找一下引擎实例渲染相关的接口==


## 状态机
- OpenGL自身是一个巨大的状态机(State Machine)
    - OpenGL的状态通常被称为OpenGL上下文(Context)
    - 假设当我们想告诉OpenGL去画线段而不是三角形的时候，我们通过改变一些上下文变量来改变OpenGL状态，从而告诉OpenGL如何去绘图。一旦我们改变了OpenGL的状态为绘制线段，下一个绘制命令就会画出线段而不是三角形


## 对象
- OpenGL库是用C语言写的，C的一些语言结构不易被翻译到其它的高级语言，因此OpenGL开发的时候引入了一些抽象层。“对象(Object)”就是其中一个

- 把OpenGL上下文看作一个大的结构体，我们使用的对象就像其中一个成员
```
// OpenGL的状态
struct OpenGL_Context {
    ...
    object* object_Window_Target;
    ...     
};
```

- 这一小段代码展现以后使用OpenGL时常见的工作流
```
// 创建对象
unsigned int objectId = 0;
glGenObject(1, &objectId);
// 绑定对象至上下文
glBindObject(GL_WINDOW_TARGET, objectId);
// 设置当前绑定到 GL_WINDOW_TARGET 的对象的一些选项
glSetObjectOption(GL_WINDOW_TARGET, GL_OPTION_WINDOW_WIDTH, 800);
glSetObjectOption(GL_WINDOW_TARGET, GL_OPTION_WINDOW_HEIGHT, 600);
// 将上下文对象设回默认
glBindObject(GL_WINDOW_TARGET, 0);
```