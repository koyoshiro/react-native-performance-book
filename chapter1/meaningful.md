# 性能优化的意义
生活在如此高速发展的时代，技术本身的更迭速度日新月异，网络基建的速度也从4G发展到了5G时代，人类在全维度对速度的追求与渴望日趋旺盛。

## 背景
尤其在移动终端，页面加载缓慢对于用户和网站本身的打击是巨大的，各项研究报告揭示了共同的趋势。
- 加载时间是导致页面放弃和忠诚度的主要因素；53％的用户报告说他们放弃了加载时间超过三秒钟的网站（来源： [SOASTA Google研究报告](https://soasta.com/blog/google-mobile-web-performance-study/)）。
- 与访问速度较慢的网站相比，用户在访问速度较快的网站上访问频率更高，停留时间更长，搜索更多并且购物频率更高；一家公司发现，转换速度提高了7％，这是因为速度提高了0.85秒（来源：[WPO Stats](https://wpostats.com/)）。
- 缓慢的加载不利于搜索引擎优化（SEO），因为它会降低您网站的排名，从而减少访问次数，阅读次数和转化次数。Google将于2018年将网站速度作为其移动搜索的排名信号（来源： [Search Engine Land](https://searchengineland.com/google-speed-update-page-speed-will-become-ranking-factor-mobile-search-289904)）。


## 性能与业务的关系
性能优化的重要意义往往体现在与业务间的关系，两者相辅相成，这是由于性能优劣的评判是由用户“投票”得出的。
- 性能可以留住用户
    - 减少可感知的等待时长，可增加用户留存；
    - 减少页面平均加载时长，可降低页面跳出率，增加页面访问率；
    - 相反，增加页面平均加载时长，将数量级得流失用户。
- 性能可以提升转化率
    - 首页加载速度每降低100毫秒，转化率就提高了1.11％；结帐页面加载速度每降低100ms，转换率增加1.55％。（[Mobify](http://resources.mobify.com/2016-Q2-mobile-insights-benchmark-report.html)）
    - 网站页面加载时间减少了20％，转换率提高了10％。（[家具村](https://www.thinkwithgoogle.com/intl/en-gb/success-stories/uk-success-stories/furniture-village-and-greenlight-slash-page-load-times-boosting-user-experience/)）
- 性能关乎用户体验

    在用户体验方面，速度至关重要。一项[消费者研究](https://www.ericsson.com/en/press-releases/2016/2/streaming-delays-mentally-taxing-for-smartphone-users-ericsson-mobility-report)表明，用户对移动端加载速度的压力类似于观看恐怖电影或解决数学问题，而不类似于在结帐台排队等待。

    下图图示了两种不同的页面加载性能，可以明显得看见两者页面渲染效果的差异。
    [![sN6Tbt.png](https://webdev.imgix.net/why-speed-matters/speed-comparison.png)](https://webdev.imgix.net/why-speed-matters/speed-comparison.png)


## 性能优化效果对比
### 动态图效果
以Ctrip App机票列表页为示例，通过动图来展现性能调优前后的对比。

[![sNTPCn.gif](https://s3.ax1x.com/2021/01/13/sNTPCn.gif)](https://imgchr.com/i/sNTPCn)

上图中，可以发现在加载新页面之前存在比较明显的白屏时间，并且在内容渲染成功之前，还存在呼吸态（骨架屏）的时间，最后再渲染出实际的页面内容，用户的使用体验经历了从打开到白屏，从白屏到呼吸态，从呼吸态到最终页面内容展示的过程，页面的体验感受大大折扣。

再看下图，经过性能调优后的相同页面，从页面被打开到展示页面内容之间实现了直出的效果，相比之下，页面打开的流畅度提升了，用户的使用体验也提升了几个档次。

[![sNTi3q.gif](https://s3.ax1x.com/2021/01/13/sNTi3q.gif)](https://imgchr.com/i/sNTi3q)

### 浏览器火焰图对比
接下来，以同一个页面调优前后的浏览器火焰图来看下性能差异。

[![sdnsIK.png](https://s3.ax1x.com/2021/01/14/sdnsIK.png)](https://imgchr.com/i/sdnsIK)

上图可见，渲染过程被分成了两个部分，两个部分之间的空白部分代表着较长的等待时间。
再来看经过性能调优后的下图，渲染过程一气呵成，没有额外非必要的等待时间，火焰图也显得紧凑。

[![sdn6PO.png](https://s3.ax1x.com/2021/01/14/sdn6PO.png)](https://imgchr.com/i/sdn6PO)