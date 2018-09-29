---
description: html + css 实现字体+背景图片混搭效果
---

# 字体+背景混合搭配

最终效果图：

![](../../.gitbook/assets/image%20%282%29.png)

HTML代码块：

```text
<!-- html start -->
  <div class="tanmer__text__background animated fadeIn delay-2s" style="--image_url: url('./images/bg.jpg')">
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

    position: relative;

    /* variable */
    background-image: var(--image_url)
  }

  .tanmer__text__background .tanmer__text {
    font-weight: bold;
    height: 100%;

    /* display flex */
    display: flex;
    display: -webkit-flex;
    justify-content: center;
    align-items: center;

    /* 颜色混合，黑色透明 */
    mix-blend-mode: screen;

    /* variable params2 初始值 */
    font-size: var(--font_size, 5vw);
    background-color: var(--background_color, white);
    color: var(--color, black);
  }
/* end */
```

## 扩展

最终效果图：

![](../../.gitbook/assets/image%20%2845%29.png)

HTML增加的代码块：

```text
<!-- html start -->
  <div class="tanmer__text__background animated fadeIn delay-2s" style="--image_url: url('./images/bg.jpg')">
    <div class="tanmmer__position__mix__blend"></div>
    <h1 class="tanmer__text size__10">Hello World !</h1>
  </div>
<!-- end -->
```

CSS增加的代码块：

```text
 /* tanmmer__position__mix__blend */
  .tanmer__text__background .tanmmer__position__mix__blend {
    position: absolute;
    width: 100%;
    height: 100%;
  
    /* variable params2 初始值 */
    mix-blend-mode: var(--mix_blend_mode, color-burn);
    background-color: var(--background_color, #03c9a9);
  }
```

### 结构

* &lt;div  class="tanmer\_\_text\_\_background"&gt;
  * 描述：主要用于放置背景图片，使图片固定在页面中，不跟随滚动
  * 变量：在 style 属性中可以定义背景图片地址，例： --image\_url: url\('图片路径'\)
    * &lt;div class="tanmmer\_\_position\_\_mix\_\_blend" &gt;
      * 描述：层级关系 图片 &lt;  当前 &lt; 文字, 搭配 mix-blend-mode 渲染文字颜色。
      * 变量：包含--mix-blend-mode（见参考文档）、--background-color，在 style 属性中进行赋值
    * &lt;h1 class="tanmer\_\_text" &gt;
      * 描述：处理文字展示效果
      * 变量：包含 --font-size、--background-color、--color，在 style 属性中进行赋值

> ### 参考文档：
>
> [https://www.youtube.com/watch?v=vs34f9FiHps&t=3783s](https://www.youtube.com/watch?v=vs34f9FiHps&t=3783s) （mix-blend-mode youtube视屏）
>
> [https://developer.mozilla.org/en-US/docs/Web/CSS/mix-blend-mode](https://developer.mozilla.org/en-US/docs/Web/CSS/mix-blend-mode) （mix-blend-mode MDN文档）



