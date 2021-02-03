# 前端优化方式
回到前端领域，调优的方式除了常规的一些方法之外，主要以React Native的角度描述App中特有的一些性能优化方案。

## Bundle 瘦身
Bundle 中存在几种文件类型，针对不同类型选择不同的优化方案：
- 代码字符串
- Iconfont 字符串
- 图片文件

### 代码字符串
冗余代码是代码 Size 的主要问题，而冗余代码的产生主要源自于四个方面：
- 已下线的需求代码
- 已结项的实验代码
- NPM 冗余调用
- 缺乏抽象的重复代码

解决方案：
- 整理已下线需求，删除相应代码及库文件
- 使用组件库及方法库，减少重复代码
- 抽象可复用的组件，使用高阶组件

### 图片文件
打包压缩过程中，图片文件的压缩比极低，导致图片越大， Bundle Size 也越大。

解决方案：
- 较大的图片在保证清晰度的前提下压缩后打包
- 视业务场景使用网络图片替代
- 较小图片可以使用 IconFont 替代

## LazyRequire
编译过程中，import会被编译成 require，require 所完成的功能是读取JavaScript 模块并执行。

而大模块的执行会耗费较多时间，使得页面加载速度变慢。因此，优化的方向是当模块在需要时才开始加载。

但 React Native 提供的标准 require 目前并不支持动态加载，需要修改 React Native 源码的打包功能，使其支持动态加载功能，并提供出对应的 API 来供业务方实现。

使用示例如下：
```javascript
import {lazyRequire} from 'react';
let moduleA = lazyRequire('../src/ModuleA');
```

## 动态加载
使用 import 语句导入模块时，会自动执行所加载的模块。而当使用组件库或公共方法库的时候，往往并不希望如此。

假设 Common.js 文件为公共方法库：

```javascript
import A from './A';
import B from './B';
import C from './C';
export {
    A,
    B,
    C
}
```

此时若希望只引用 Common.js 中的A模块，即：
```javascript
import {A} from './Common.js';
```

但实际B和C模块代码也被执行了。为了使程序能如你所愿的仅执行A模块，需要使用属性 getter 动态 require 的方式来修改 Common.js 文件。

```javascript
const Common = {
    get A(){
        const module = require('./A');
        return (module && module.__esModule)? module.default:module;
    }
    get B(){
        const module = require('./B');
        return (module && module.__esModule)? module.default:module;
    }
    get C(){
        const module = require('./C');
        return (module && module.__esModule)? module.default:module;
    }
}
module.exports = Common;
```

这样在使用到A模块的时候才会执行 `require('./A').default`，并不会加载B和C。

至此，使用该方式导出模块可以减少引用模块时的无效加载数量，达到优化渲染速度的目的。

## 骨架屏/呼吸态
骨架屏是有效减少用户体感“白屏”的有效措施，通常使用骨架屏完成耗时较长的关键性任务，如核心服务请求、重要异步回调等。

以Trip App的国际航班列表页为例，来看下骨架屏的应用：

[![yMo3vT.jpg](https://s3.ax1x.com/2021/02/03/yMo3vT.jpg)](https://imgchr.com/i/yMo3vT)

同时，骨架屏也是缩短FCP和FMP 指标的重要方法，主要方式：
- 减少加载骨架屏之前的非必要模块引用
- 核心服务请求参数的拼接可放在骨架屏渲染之前完成
- 骨架屏自身的渲染结构足够简单

## 分批次渲染
分批次的概念适合运用在列表型或内容型的页面中，将需要展示的内容，分成不同阶段/批次进行渲染，阶段/批次的划分数量根据业务自身情况而定，建议以屏幕的主要可视区域为宜。

该方案对提升TTI有较大作用，可数量级的减少渲染内容。

我们还是以Trip App的国际列表页为例，可以看见下图中页面仍然处在加载过程中，但红框部分已经出现了部分航班卡片，这就是采用分批次渲染实现的效果。

[![yMo3vT.jpg](https://cdn.nlark.com/yuque/0/2021/png/2487128/1611655728681-404bbc43-77d5-47de-8bb8-43f7e9a8b3f8.png?x-oss-process=image%2Fresize%2Cw_622)](https://cdn.nlark.com/yuque/0/2021/png/2487128/1611655728681-404bbc43-77d5-47de-8bb8-43f7e9a8b3f8.png?x-oss-process=image%2Fresize%2Cw_622)


## 渐进式渲染
React Native 渲染的本质是将 JSX 构建的虚拟 DOM 树通过 Native Render 的方式绘制页面内容。虚拟 DOM 树结构越复杂，Native Render 所需绘制的时间也越长。

从该特性出发，可以通过降低虚拟 DOM 树结构的复杂度来减少渲染耗时，用尽可能短的时间到达 TTI 阶段。

降低虚拟 DOM 树结构复杂度的底线是最低程度得保证业务功能的完整性，而在其渲染完成后（达到TTI阶段），通过 setState 去更新渲染完整的虚拟 DOM 树结构即可。

下面两幅图在渲染过程中采用了渐进式渲染，观察红框部分的差异可以发现，航空公司等非关键信息存在延迟加载的效果。 

[![yMo3vT.jpg](https://cdn.nlark.com/yuque/0/2021/png/2487128/1611736223282-cc18163b-deea-405e-b588-2d10f607f938.png?x-oss-process=image%2Fresize%2Cw_600)](https://cdn.nlark.com/yuque/0/2021/png/2487128/1611736223282-cc18163b-deea-405e-b588-2d10f607f938.png?x-oss-process=image%2Fresize%2Cw_600)

[![yMo3vT.jpg](https://cdn.nlark.com/yuque/0/2021/png/2487128/1611736375028-066fce28-3818-4c66-8c68-79ece0e1ffdc.png?x-oss-process=image%2Fresize%2Cw_600)](https://cdn.nlark.com/yuque/0/2021/png/2487128/1611736375028-066fce28-3818-4c66-8c68-79ece0e1ffdc.png?x-oss-process=image%2Fresize%2Cw_600)

                    
## 延迟渲染
页面在业务模块数量较为复杂的情况下，需要渲染模块也会同比增多，同时渲染耗时也会水涨船高。

延迟渲染的本质是区分对待需要渲染的模块，可以按模块的重要程度区分为核心与非核心模块，也可按模块的轻重缓急区分为重要与次要模块。优先渲染核心/重要的模块，符合界面基本交互功能（达到TTI阶段），再渲染非核心/次要模块来完成整个页面的渲染工作。

下面两张图对比红框部分，可以看到非核心模块被延迟渲染，由于在TTI阶段之后的很短时间内完成渲染，并不会影响用户的交互体验。

[![yMo3vT.jpg](https://cdn.nlark.com/yuque/0/2021/png/2487128/1611737605102-737b72be-92a0-4c95-9490-f6490772aed8.png?x-oss-process=image%2Fresize%2Cw_600)](https://cdn.nlark.com/yuque/0/2021/png/2487128/1611737605102-737b72be-92a0-4c95-9490-f6490772aed8.png?x-oss-process=image%2Fresize%2Cw_600)             

[![yMo3vT.jpg](https://cdn.nlark.com/yuque/0/2021/png/2487128/1611737680537-0ab7e7da-103c-4a39-9910-ddd444603628.png?x-oss-process=image%2Fresize%2Cw_600)](https://cdn.nlark.com/yuque/0/2021/png/2487128/1611737680537-0ab7e7da-103c-4a39-9910-ddd444603628.png?x-oss-process=image%2Fresize%2Cw_600)


## 按需渲染
页面中不可避免的会存在一些浮层或者二级界面，下面统称为次级界面。

然而次级界面在TTI阶段前，大部分是不需要进行渲染的，可以配合 LazyRequire 的方式完成。

[![yMo3vT.jpg](https://cdn.nlark.com/yuque/0/2021/png/2487128/1611734048157-394a399e-221e-46b7-a966-ac613de8e9cc.png?x-oss-process=image%2Fresize%2Cw_600)](https://cdn.nlark.com/yuque/0/2021/png/2487128/1611734048157-394a399e-221e-46b7-a966-ac613de8e9cc.png?x-oss-process=image%2Fresize%2Cw_600)

上图中红框部分的提示功能，点击后将出现下图中的内容浮层。而内容浮层本身并不需要在页面加载过程中完成加载和渲染工作，可以放在TTI阶段之后去完成，并不会影响交互的流畅程度。

[![yMo3vT.jpg](https://cdn.nlark.com/yuque/0/2021/png/2487128/1611734382248-a8bc20cc-25f7-4616-9488-fd8be1fd2efa.png?x-oss-process=image%2Fresize%2Cw_600)](https://cdn.nlark.com/yuque/0/2021/png/2487128/1611734382248-a8bc20cc-25f7-4616-9488-fd8be1fd2efa.png?x-oss-process=image%2Fresize%2Cw_600)

      
## 预渲染

空间换时间的经典方案。

假设需要在Page A 打开 Page B，若Page A 能够在打开Page B之前提前渲染Page B的话，可以大大加快Page B的渲染速度，降低性能指标。

具体方法是在Page A时，通过 Native API 热启动一个新的 React Native 容器，同时在新容器内预加载Page B的 Bundle 并执行。

[![yMo3vT.jpg](https://cdn.nlark.com/yuque/0/2021/png/2487128/1611738448390-cba15594-868a-44c2-b63b-87ad41288520.png?x-oss-process=image%2Fresize%2Cw_622)](https://cdn.nlark.com/yuque/0/2021/png/2487128/1611738448390-cba15594-868a-44c2-b63b-87ad41288520.png?x-oss-process=image%2Fresize%2Cw_622)


上图中，点击红框中的搜索按钮后，会热启动打开一个透明的React Native容器并开始执行Bundle内容，在延迟一定时间后（通常是300-500ms），展示容器中的渲染结果，如下图所示。

[![yMo3vT.jpg](https://cdn.nlark.com/yuque/0/2021/png/2487128/1611738482439-0b669bbc-66a8-47f7-bb80-6c8da2782a8a.png?x-oss-process=image%2Fresize%2Cw_622)](https://cdn.nlark.com/yuque/0/2021/png/2487128/1611738482439-0b669bbc-66a8-47f7-bb80-6c8da2782a8a.png?x-oss-process=image%2Fresize%2Cw_622)

