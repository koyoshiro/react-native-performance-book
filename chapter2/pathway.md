# 量化方式
了解量化角度后，我们有了量化性能的切入点，下面继续从样本采集和样本归集两个角度讲述量化方式。

## 样本采集
从`量化角度`这一小节中知道样本量是一个非常重要的统计因素，因此样本采集工作就显得尤为重要。

与其他前端页面不同的是，App作为一个具备客户端版本+前端Bundle版本的结合体，样本的采集工作会更加复杂。

### 采集指标
鉴于版本这个特殊性的存在，采集的伊始就需要确定好对应的采集指标，且并不随着版本的更迭出现指标项上的变化。
在`性能优化的定义`这一小节，介绍了React Native的加载过程，可以看出FP、FCP、FMP与TTI这些指标项都具备采集的必要性，分别对应下述场景的时间点：
- FP：白屏结束时间点
- FCP：页面开始渲染时间点
- FMP：页面主要内容出现时间点
- TTI：页面加载完毕时间点

### 采集方式
通过在客户端和前端底层框架中找到对应时机点的代码，埋点上报完成。当然，上报的信息除了指标项之外，还需要有对应的用户设备信息，客户端和前端Bundle版本信息等识别类信息，用于在样本归集阶段使用。

其中比较特殊的指标项采集方式是TTI，由于每一次的`setState`通常都会触发一次页面渲染，而每个页面的加载的内容不同，页面加载完毕的时间点也就无法从代码层面统一埋点收口处理。

这就需要使用页面像素检测来解决，页面像素检测可以理解为图像比对的一种应用方式，比较前后检测到的同一个像素点变化来确定是否存在页面重绘/刷新。

另外，由于页面像素检测的对象是整个页面，容易将页面中某块固定的部分认定为一定时间内没有出现重绘/刷新，从而造成该指标采样的错误。

针对该情况，可以依据页面的渲染内容特征来选取无需做像素检测的部分内容，忽略无需检测的内容，提高采集样本的准确率。如下图中红框所示部分作为检测区域，其余部分不属于检测范围。

<img src="https://s3.ax1x.com/2021/01/22/sI6Or8.png" style="zoom:50%" />

当然，由于页面间业务的差别，需要提供页面像素检测的API，便于接入方更精准提供判断区域。

## 样本归集
采集到一定量的样本数据后，需要对采集到的样本数据进行归类处理。目的是从多维度多指标地得出量化数据，为性能调优的迭代效果提供数据支撑。

样本归集方面主要以指标维度和平台维度这两个方面进行分析说明，了解后将对React Native的量化调优数据有一定了解，为后续调优过程提供帮助。

### 指标维度
顾名思义，以不同指标项为维度进行的样本处理。该维度的处理结果将性能调优分为不同阶段，针对不同阶段的指标项，调优的方式也不尽相同。

从渲染的总体流程来看，FP作为一类、FCP与FMP作为一类、TTI作为一类，这样切分分别对应了页面渲染的不同阶段。

- FP阶段时，页面内容一片空白，该阶段耗时主要由于App端需要加载Bundle包导致；
- FCP与FMP阶段时，页面开始渲染内容，但不可交互，该阶段耗时主要由于一些必须完成的任务导致；
- TTI阶段时，页面内容可交互，该阶段耗时是由于需要等待数据请求返回或其他前序任务完成导致。

分阶段对量化数据进行归集后，采用可视化的方式分析不同阶段的数据指标，让每一次性能调优都有数据可依。

### 平台维度
React Native支持双平台，但在不同平台上的实际性能却是截然不同的，包括各类性能调优的方式和结果会出现许多差异性。

正由于这些差异性，按平台维度去归集量化的样本数据成为了必需品。通常情况下，会将双平台的数据分开归集后进行分析，这样可以很好的去做针对性得性能调优。

例如，React Native的JS渲染引擎 Hermes只能运用在Android平台，也就意味着从调优方式开始决定了调优的不同平台和结果。

在区分双平台的情况下，每个平台的量化数据需要在归集时划定不同的性能区域，如“0-0.5s、0.5-1s”分布式得展示性能分布情况。如下图所示：

[![sI7UYV.png](https://s3.ax1x.com/2021/01/22/sI7UYV.png)](https://imgchr.com/i/sI7UYV)

采用该种方式展示的优势是可以直观得看到量化后的性能数据分布情况，其量化的性能数据往往呈现出正态分布的态势，当然随着机型分布情况的差异性容易出现长尾效应。

针对某一特定区域（如长尾部分）的性能数据，分析其形成原因，将更容易制定针对性的调优方案，这也是在性能调优的迭代过程中发现问题的重要量化指针。