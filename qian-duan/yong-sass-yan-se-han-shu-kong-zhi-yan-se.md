---
description: 用Sass颜色函数控制颜色，它们可以大大加快开发速度
---

# 用Sass颜色函数控制颜色

将要处理的颜色创建变量

```css
$base-color: #AD141E;
```

### 变黑 & 变淡 （Darken & Lighten）

原文：

> These two adjust the Lightness of the color’s HSL values. Sass will parse our hex color variable to hsl, then make the adjustments. You call them on the color with a percentage value between 0 and 100. I usually stay in the range of 3-20%.

翻译

> 这两个调整了颜色的HSL值的亮度。Sass将把我们的十六进制颜色变量解析为hsl，然后进行调整。你可以用0到100之间的百分数来称呼它们。通常停留在3-20%的范围内。

```text
darken( $base-color, 10% )
lighten( $base-color, 10% )
```

![Darken &amp; Lighten](../.gitbook/assets/dark_light.png)

### 饱和 & 冲淡（Saturate & Desaturate）

原文

> These will will adjust the Saturation of the colors HSL values, much like Darken and Lighten adjusted the Lightness. Again, you need to give a percentage value to saturate and desaturate.

翻译

> 这些将会调整HSL值颜色的饱和度，就像变暗一样，调节亮度。同样，你需要给出一个百分比值来饱和和饱和度。

```text
saturate( $base-color, 20% )
desaturate( $base-color, 20% )
```

![Saturate &amp; Desaturate](../.gitbook/assets/tumblr_luv5ij6hvy1qb5ozt.png)

### 调整颜色（Adjust-hue）

原文

> This adjusts the hue value of HSL the same way all of the others do. Again, it takes a percentage value for the change.

翻译

> 通过百分比值的变化，调整HSL的色调值。

```text
adjust-hue( $base-color, 20% )
```

![Adjust-hue](../.gitbook/assets/adjust.png)

### 添加Alpha透明度（Adding Alpha Transparency）

原文

> Using our hex color we can do a few things to get it to be a little transparent. We can call hsla, rgba, opacify, and transparentize. All of them accomplish the same thing, just in different ways. I stick to rgba as it comes most naturally to me which takes a color and a value from 0 to 1 for the alpha.

翻译

> 使用我们的十六进制颜色，我们可以做一些事情使它变得有点透明。我们可以调用hsla、rgba、opacify和透明。所有的人都以不同的方式完成同样的事情。我坚持rgba，因为它对我来说是最自然的，它的颜色和值从0到1。

```text
rgba( $base-color, .7 )
```

### 色彩和阴影（Tint & Shade）

原文

> Our very own Phil LaPier has added to those base color functions. Both of these are accessible in Bourbon. They mix your color with a value of white \(tint\) and black \(shade\) and are similar to Darken and Lighten. They take the color and a % value for the change.

翻译

> 把你的颜色和白色\(色调\)和黑色\(阴影\)的值混合，类似于变暗和变亮。它们以颜色和%值进行更改。

```text
tint( $base-color, 10% )
shade( $base-color, 10% )
```

![Tint &amp; Shade](../.gitbook/assets/tint.png)

### 更多优势（Getting more advanced）

原文

> Once you have those down and you are looking for more control you can look into some more advanced color control with adjust-color, scale-color, change-color. These are for multiple changes in one color function. You can easily lighten the color and add some transparency all in one.

翻译

> 一旦你有了这些，你想要更多的控制，你可以看到一些更高级的颜色控制和调整颜色，标色，改变颜色。这些是一个颜色函数的多个变化。你可以轻松地减轻颜色，并在其中添加一些透明度。

原文

> Some of the best places to use these color functions are for gradients, borders and shadows. When you need a slightly darker border and a slightly lighter inset shadow just adjust a color variable and let Sass do the rest for you. Buttons provide the perfect place to test out the functions. Check out some of the functions used on the thoughtbot buttons:

翻译

> 使用这些颜色函数的一些最好的地方是渐变、边框和阴影。当你需要一个稍微深一点的边框和一个稍微亮一点的嵌入阴影时，只需调整一个颜色变量，让Sass为你做其他的事情。按钮提供了测试函数的最佳位置。查看一些在thoughtbot按钮上使用的功能:

```text
border: 1px solid darken($base-color, 20%);
text-shadow: 0 -1px 0 darken($base-color, 10%);
@include box-shadow(inset 0 1px 0 lighten($base-color, 20%));
```

![Getting more advanced](../.gitbook/assets/advan.png)

原文出处 [https://robots.thoughtbot.com/controlling-color-with-sass-color-functions](https://robots.thoughtbot.com/controlling-color-with-sass-color-functions)

