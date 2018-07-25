# 页面性能
当讨论页面性能的时候，人们很容易陷入误区   
比如经常会听到有人说：我刚测了下，页面加载速度是2s   
这句话本身没有错，但并不能准确的反应现实；有这么几点误区:
* 页面加载时间随用户的网络情况和设备硬件的变化而不同；单一的数字并不能反应这种区别   
    考虑到设备很多，可以根据网络情况对加载速度分组，比如wifi, 4G, 3G   
    也有人按照时间段对用户进行分组，统计每个时间段的用户量，这也是种方式（如图所示）
* 单一的指标并不能准确的表达页面的整个加载过程；加载过程中有很多时刻会影响用户对网页加载速度的感知。   
    * 比如一个页面1秒钟就有内容展示出来，但不能交互；等执行完很久的js文件后(10s)才可以交互；在这期间用户能看到链接点了却没反应；能看到输入框却不能输入，用户体验还是很差   
    * 又比如页面在加载2s后就能交互，但2s前没有任何内容展示给用户，即白屏时间长达2秒钟，很容易造成用户跳出
* 加载时间并不是性能的唯一关心点；还有很多其他因素会造成糟糕的性能，用户体验；比如滑动不顺畅，点击没有及时反馈等

为了避免犯上面的错误，谷歌员工提出了这样一个模型来描述、测量性能： RAIL (配图)   

## 感知延迟的关键指标
在深入介绍RAIL之前，先了解下用户能感知到延迟的一些关键指标（配图）   

不同的硬件设备、网络环境，用户对于性能延迟的感知会有所不同；比如PC用户和3G网络的Mobile用户，PC用户感受到页面加载延迟的值是1000ms，而3G网络的Mobile用户可以接受的延迟值为5000ms。Mobile用户相比PC用户而言，更有耐心   
上图中的指标，适用于4G,Wifi,PC用户

## RAIL

正如之前的图所示，RAIL包含了4部分：   
* R: Response   
* A: Animation   
* I: Idle   
* L: Load   

### Response
目标：100ms内对用户输入做出反应  
    如果响应超过100ms, 用户输入与反应之间的联系就会被中断，用户可以感受到延迟；这个指标适用于大部分的输入方式，比如点击按钮，toggle表单控件，或者开始动画； 对于拖拽、滚动并不适用       

测量指标&方式：   
输入事件的回调函数执行完成时间点与事件的触发时间点之差
```javascript
const subscribeBtn = document.querySelector('#subscribe');
subscribeBtn.addEventListener('click', (event) => {
  // 事件处理逻辑...
  const lag = performance.now() - event.timeStamp;
  if (lag > 100) {
    // 埋点
  }
});
```

指导方针： 
* 事件处理函数最好在50ms内完成，因为浏览器的事件处理机制，用户触发输入后，事件会进入队列中，可能需要50ms左右的时间才能执行到事件处理函数（配图）
* 可能听起来有点反常规，最好不要立刻响应用户输入。最大化的利用这100ms去完成一些耗时的工作，但要对用户无感知
* 对于超过100ms的响应，一定要提供反馈

### 动画
目标： 每10ms（内）一帧
从技术上来讲，临界值为每帧16ms(1000ms/60帧)。但是浏览器需要花费6ms的时间绘制1帧，所以要将每帧时间控制在10ms   

测量指标&方式：Long Task   
```javascript
const observer = new PerformanceObserver((list) => {
  for (const entry of list.getEntries()) {
    console.log('task time: ', Math.round(entry.startTime + entry.duration));
    console.log('task attribution: ', JSON.stringify(entry.attribution));
  }
});
observer.observe({entryTypes: ['longtask']});
```
目前entry.attribution支持显示的参数比较少，后续会更好的支持   
此外 PerformanceObserver API尚在草案阶段，最新版的chrome可以试用  

指导方针：    
* 尽可能的少做事，只处理必须要处理的   
* 可以利用100ms的输入相应时间，在动画触发之前，完成一些需要复杂计算的工作   
* 如下这些都视为动画：
   * 页面切换动画（入场动画、离场动画等）
   * 滚动 包括释放滚动后，页面的继续滑动
   * 拖拽 例如 地图的放大、缩小

### 空闲时间
目标： 最大化的利用空闲时间来保证100ms的响应时间，保证动画的流畅   

指导方针：   
* 利用空闲时间完成不紧急的工作。 比如页面的初始化加载中，尽可能的少加载资源；其余的非必要资源放到空闲时间加载
* 将工作分为耗时小于50ms的代码块执行；大于50ms的话，就会影响及时响应用户输入

### 加载
目标： 在理想时间内呈现页面给用户，并可以交互   

测量指标&方式：FCP、FP、FMP、TTI   
* FP: First Paint    
    浏览器渲染任何内容到屏幕上的时刻，比如页面背景色的变化，一个loading图标
* FCP: First Content Paint
    浏览器渲染任意DOM内容到屏幕上的时刻，比如一段文本，一张图片，SVG或者Canvas
* FMP: First Meaningful Paint
    对用户来说有用的内容信息
* TTI: Time To Interactive   
    页面可以交互的时间点
（上图）   
可以以提问的方式，来问大家各个图代表什么阶段（增加互动）   
关系为： FP >= FCP >= FMP >= TTI   

FP和FCP是可以自动测量的，浏览器可以根据渲染的节点判断出；也有对应的API来获取：
```javascript
const observer = new PerformanceObserver((list) => {
    for (const entry of list.getEntries()) {
      // `name` will be either 'first-paint' or 'first-contentful-paint'.
      const metricName = entry.name;
      const time = Math.round(entry.startTime + entry.duration);
      console.log(`${metricName}: ${time}`);
    }
  });
  observer.observe({entryTypes: ['paint']});
```
FMP不好自动化，因为不同页面，标识FMP的内容是不同的，需要开发者自己判断并统计，统计方式：   
（PS：目前有很多人在研究FMP的自动化算法，取得了一定的成果，但准确度还不是很高）
在认为完成的点，取`Performance.now()`的值作为FMP的值

TTI目前研究人员正在尝试将其计算过程标准化，并通过`PerformanceObserver`API暴露出来   
在此之前，需要开发人员自己识别TTI的时间点并做埋点统计，统计方式和FMP一样

指导方针：
* 识别影响加载性能的因素
    * 网络速度和延迟
    * 硬件
    * js的解析等
* 没必要在指定的时间内加载完所有的资源；做到渐进式加载渲染，将不必要的资源放到空闲时间去加载和执行

## 参考链接：
https://developers.google.com/web/fundamentals/performance/rail
https://developers.google.com/web/fundamentals/performance/user-centric-performance-metrics
