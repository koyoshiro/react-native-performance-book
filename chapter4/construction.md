当各种调优方式纷纷上线之后，React Native应用的性能开始令你感到满意。但随后会发现，随着需求的不断迭代，保持现有的优化成果变得困难起来，而这一切的原由可以归结为缺乏性能优化的体系。

良好的性能优化体系可以让React Native应用更顺畅地迭代下去，同时可以从源头解决低性能代码的产生。下面将通过结构和保障这两个点来描述如何建立一套性能优化体系。

# 整体结构
应用页面想要在性能量化数据上符合标准，除了进行性能调优之外，更重要的是对项目代码的整体结构有一定的前提认识。

## 频道
React Native代码在App内是以Bundle文件的形式存在的，我们把这些不同的Bundle称为频道。而Bundle（频道）的Size取决于项目的分割方式，我们来举一个例子：

> 一个App内往往分布着多个业务线，以这些业务线为维度可以切割为不同的频道。

此时App与频道的关系如下图：

<img src="https://s3.ax1x.com/2021/01/28/ypFyNV.png" style="zoom:50%" /> 


通常一个频道对应一个项目，而在实际开发过程中，某一个频道的Size随着业务迭代逐步增加，项目中也会产生更多的子项目，子项目之间的迭代周期/频率的不同促使了子频道的产生，此时App、业务线频道与业务线子频道间的关系如下图：

<img src="https://s3.ax1x.com/2021/01/28/ypVWPx.png" style="zoom:50%" /> 

形象地理解频道与子频道的关系后，倘若一个业务线至存在一个频道，可以将Bundle理解成一个SPA模式，若存在多个频道，则可以将Bundle看成是MPA模式。
- SPA模式的频道优点在于全流程的页面可以共用同一个React Native容器，以此来提升性能数据。
- MPA模式的频道优点在于方便不同子频道的迭代频道，单个Budnle文件Size较小，但无法共用同一个React Native容器。

那么该如何选择才是比较科学的方式呢？

通常我们建议通过以下几点来判断是否需要创建子频道：
- 各子频道迭代周期/频率差异性较大；
- 各子频道Bundle文件的Size足够小；
- 业务流程中首个子频道对性能的要求较高。

## 公用库
涉及到项目内部结构的维度，以整个App的角度拆分来看，各业务线的频道均会使用到React Native框架，每个频道在打包的过程中就会将React Native框架打包多次，通常使用本地指向的方式解决，修改前多个Bundle的内部情况如下图：

<img src="https://s3.ax1x.com/2021/01/28/ypMkFJ.png" style="zoom:50%" /> 

修改本地指向方式之后，多个Bundle的内部情况更新如下图，可以明显发现重复的React Native框架被统一指向了Common Bundle中，减少非必要的App Size。

<img src="https://s3.ax1x.com/2021/01/28/ypMAY9.png" style="zoom:50%" /> 

再将视角聚焦到频道内部的项目代码结构中去，往往都存在于下面几个问题：
- 公用代码不抽象
- 业务组件重复开发
- NPM包引用重复/混乱
杂乱的结构关系通过一系列的改造后，给出对应的解决方式，优化后的效果如下图：

<img src="https://s3.ax1x.com/2021/01/28/y9KQpR.png" style="zoom:50%" /> 

抽象公用代码的库来解决公用代码的重复劳动和不够抽象的问题，其中分为基础的公用方法库与业务相关的公用方法库两种大类。

提供统一的组件库，具备足够细的颗粒度，在满足通用的前提下，便于组装出适应业务功能抽象的上层的业务组件库。

使用Tree-Shaking去除非必要的NPM包引用，再通过本地重定向的方式避免重复包的问题。