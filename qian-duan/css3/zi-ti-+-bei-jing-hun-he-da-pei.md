---
description: html + css 实现字体+背景图片混搭效果
---

# 字体+背景混合搭配

最终效果图：

![](../../.gitbook/assets/image%20%282%29.png)

HTML代码块：

```text
<!-- html start -->
  <div class="tanmer__text__background animated fadeIn delay-2s" style="background-image: url('./images/bg.jpg')">
    <h1 class="tanmer__text size__10">Hello World !</h1>
  </div>
<!-- end -->
```

CSS代码块：

```text
<!-- animation css -->
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/animate.css/3.5.2/animate.min.css">

/* 可要可不要 */
html, body {
  height: 1000px;
  width: 100%;
} 
 
/* css start */
  .tanmer__text__background {
    width: 100%;
    background-color: white;
    background-size: cover;
    background-position: center center;
    background-attachment: fixed;
  }
  .tanmer__text__background .tanmer__text {
    font-weight: bold;
    background-color: white;
    color: black;
    height: 100vh;
    width: 100vw;

    font-size: 14vw;
    
    /* flex */
    display: flex;
    display: -webkit-flex;
    justify-content: center;
    align-items: center;
    
    /* 颜色混合，黑色透明 */
    mix-blend-mode: screen;
  }
/* end */
```



CSS中用到了:

* background-attachment: fixed;  // 背景图片定位
* background-size: cover; // 覆盖在盒子宽度以内
* font-size: 14vm; // vw: 相对窗口宽度大小，窗口宽度为 100% ，字体大小为窗口大小的14%；
* display: flex; // 弹性盒子布局
* mix-blend-mode: screen; // screen；黑色部分透明，multiply; 白色部分透明
  * 样式视屏：[https://www.youtube.com/watch?v=vs34f9FiHps&t=3783s](https://www.youtube.com/watch?v=vs34f9FiHps&t=3783s) （此视屏中包含了其他css 秘密）



