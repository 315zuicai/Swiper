# 一个轮播图Vue组件
大家需要的话可以自取
由Swiper组件和SwiperItem组件组成。Swiper组件中有主要逻辑，SwiperItem设置插槽用于图片展示。

#### 1. 启动流程

1. 挂载完毕后，首先操作DOM，在前后分别插入一个Slide，即实现图片由`1-2-3-4`到`4copy-1-2-3-4-1copy`的布局。同时进行轮播图的样式对象声明`CSSStyleDeclaration`，便于其他操作进行时对样式的调整。最后滚动轮播图由当前的 4至1。

   ```javascript
   handleDom: function() {
         // 1.获取要操作的元素
         let swiperEl = document.querySelector(".swiper");
         let slidesEls = swiperEl.getElementsByClassName("slide");
   
         // 2.保存个数
         this.slideCount = slidesEls.length;
   
         // 3.如果大于1个, 那么在前后分别添加一个slide
         if (this.slideCount > 1) {
           // 深克隆
           let cloneFirst = slidesEls[0].cloneNode(true);
           let cloneLast = slidesEls[this.slideCount - 1].cloneNode(true);
           swiperEl.insertBefore(cloneLast, slidesEls[0]);
           swiperEl.appendChild(cloneFirst);
           // swiper的宽度
           this.totalWidth = swiperEl.offsetWidth;
           // swiper的样式声明对象
           this.swiperStyle = swiperEl.style;
         }
   
         // 4.让swiper元素, 显示第一个(目前是显示前面添加的最后一个元素)
         this.setTransform(-this.totalWidth);
       },
   ```

2. 开启间隔定时器，每隔`interval`进行滚动

#### 2. 滚动和校验

1. transition + transform实现滚动
2. 每次滚动完毕后，校验正确的位置，并且是在上一个动画等待animDuration后进行，也就是当4经过animDuration后，变为`1copy`的时候进行判断，如果不等待animDuration，那么一开始滚动一点点就直接瞬间移动到1，没有4到`1copy`的动画过渡。
   - 当处于`1copy`后，瞬间移动到1
   - 当处于`4copy`后，瞬间移动到4

```javascript
    /**
     * 定时器操作
     */
    startTimer: function() {
      this.playTimer = window.setInterval(() => {
        this.currentIndex++;
        this.scrollContent(-this.currentIndex * this.totalWidth);
      }, this.interval);
    },
    stopTimer: function() {
      window.clearInterval(this.playTimer);
    },

    /**
     * 滚动到正确的位置
     */
    scrollContent: function(currentPosition) {
      // 0.设置正在滚动
      this.scrolling = true;

      // 1.开始滚动动画
      this.swiperStyle.transition = "transform " + this.animDuration + "ms";
      this.setTransform(currentPosition);

      // 2.判断滚动到的位置
      this.checkPosition();

      // 4.滚动完成
      this.scrolling = false;
    },

    /**
     * 校验正确的位置
     */
    checkPosition: function() {
      window.setTimeout(() => {
        // 1.校验正确的位置
        this.swiperStyle.transition = "0ms";
        if (this.currentIndex >= this.slideCount + 1) {
          this.currentIndex = 1;
          this.setTransform(-this.currentIndex * this.totalWidth);
        } else if (this.currentIndex <= 0) {
          this.currentIndex = this.slideCount;
          this.setTransform(-this.currentIndex * this.totalWidth);
        }

      }, this.animDuration);
    },

    /**
     * 设置滚动的位置
     */
    setTransform: function(position) {
      this.swiperStyle.transform = `translate3d(${position}px, 0, 0)`;
      this.swiperStyle[
        "-webkit-transform"
      ] = `translate3d(${position}px), 0, 0`;
      this.swiperStyle["-ms-transform"] = `translate3d(${position}px), 0, 0`;
    },
```

#### 3. 触摸事件

利用TouchEvent事件进行监听https://developer.mozilla.org/zh-CN/docs/Web/API/TouchEvent

> `TouchEvent.touches`：一个 [`TouchList`](https://developer.mozilla.org/zh-CN/docs/Web/API/TouchList)，其会列出所有当前在与触摸表面接触的  [`Touch`](https://developer.mozilla.org/zh-CN/docs/Web/API/Touch) 对象，不管触摸点是否已经改变或其目标元素是在处于 `touchstart `阶段
>
> `Touch.pageX`：触点相对于HTML文档左边沿的的X坐标

三种方法：

1. touchStart
   - 如果正在滚动，不可以拖动
   - 停止间隔定时器，不进行轮播
   - 保存开始拖动的位置，触点相对于HTML文档左边沿的的X坐标
2. touchMove
   - 计算用户拖动距离
   - `setTransform`移动位置，拖多少移多少
3. touchEnd
   - 获取移动距离
   - 判断移动距离，根据移动距离，决定当前`currentIndex`+1还是-1
   - `scrollContent`移动位置

#### 4. 指示器

根据`slideCount = 4`遍历设置div标签，并设置样式。同时动态绑定class，`:class="{active: index === currentIndex-1}"`当激活的时候，设置背景颜色。
