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
    height: 100%;
    background-color: white;
    background-size: cover;
    background-position: center center;
    background-attachment: fixed;
  }
  .tanmer__text__background .tanmer__text {
    font-weight: bold;
    background-color: white;
    color: black;
    height: 100%;
    font-size: 5vw;
    
    /* display flex */
    display: flex;
    display: -webkit-flex;
    justify-content: center;
    align-items: center;
    
    /* 颜色混合，黑色透明 */
    mix-blend-mode: screen;
  }
/* end */
```

## 扩展

HTML代码块：

```text
<!-- html start -->
<div class="tanmer__text__background animated fadeIn delay-2s" style="background-image: url('./images/bg.jpg')">
    <div class="tanmmer__position__mix__blend" style="background-color: #03c9a9;"></div>
    <h1 class="tanmer__text size__10">Hello World !</h1>
  </div>
<!-- end -->
```

CSS代码块：

```text
/* css start */
  .tanmer__text__background {
    width: 100%;
    height: 100%;
    background-color: white;
    background-size: cover;
    background-position: center center;
    background-attachment: fixed;

    position: relative;
  }

  /* tanmmer__position__mix__blend */
  .tanmer__text__background .tanmmer__position__mix__blend {
    position: absolute;
    width: 100%;
    height: 100%;

    /* 颜色混合 */
    mix-blend-mode:color-burn;
  }
  
  .tanmer__text__background .tanmer__text {
    font-weight: bold;
    background-color: white;
    color: black;
    height: 100%;
    font-size: 5vw;

    /* display flex */
    display: flex;
    display: -webkit-flex;
    justify-content: center;
    align-items: center;

    /* 颜色混合，黑色透明 */
    mix-blend-mode: screen;
  }
  
/* end */
```

> ### 参考文档：
>
> [https://www.youtube.com/watch?v=vs34f9FiHps&t=3783s](https://www.youtube.com/watch?v=vs34f9FiHps&t=3783s) （mix-blend-mode youtube视屏）
>
> [https://developer.mozilla.org/en-US/docs/Web/CSS/mix-blend-mode](https://developer.mozilla.org/en-US/docs/Web/CSS/mix-blend-mode) （mix-blend-mode MDN文档）

