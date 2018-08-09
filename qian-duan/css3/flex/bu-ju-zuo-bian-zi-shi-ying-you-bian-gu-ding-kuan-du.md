# 布局左边自适应，右边固定宽度

1. flex 左边自适应，右边固定宽度
2. 项目中踩坑集锦

> 简要介绍: 容器（flex container）和 项目（flex item）

## flex 左边自适应，右边固定宽度

```text
// html
<div class="demo">
  <div class="left">
  </div>
  <div class="right">
  </div>
</div>

// style
.demo {
  border: 3px solid red;
  display: flex;
  align-items: stretch;

  // 左右空白比为 0:1，所以右边会分配到剩余空白的所有空间
  .left {
    background: #F7E8B4;

    // 宽度自动填充
    width: auto;

    // 有多余的项目空间，分配
    flex-grow: 1;
    height: 200px;
  }

  .right {
    background: #B4D3F7;

    // 设置宽度
    width: 200px;
    min-width: 200px;
    max-width: 200px;

    // 有多余的项目空间，不分配
    flex-grow: 0;
  }
}
```

效果图如下:

![](../../../.gitbook/assets/image%20%2839%29.png)

## 项目中踩坑集锦

接着以上的做法，应用到实际的项目需求中时，发生了下面👇意料之外的情况：

> 需求:  左侧包含的内容有标题、描述\(超出并显示省略号\)，右侧是一张固定宽高的图片。

根据需求建立如下图：

![](../../../.gitbook/assets/image%20%2816%29.png)

  
现在就要解决的是描述\(**超出并显示省略号\) :**

```text
.left {
  background: #F7E8B4;

  // 宽度自动填充
  width: auto;

  // 有多余的项目空间，分配
  flex-grow: 1;
  height: 200px;
  
  // 超出并显示省略号
  p {
    text-overflow: ellipsis;
    white-space: nowrap;
    overflow: hidden;
  }
}

```

如下图：

![](../../../.gitbook/assets/image%20%2820%29.png)

可见左侧的宽度已经将右侧挤出可视宽度区域，这不是我们想要的。

**产生的原因：**

     返回我们刚刚写的css，我们将左侧宽度设置为自适应 `width: auto;` p 标签样式中（超出并显示省略号）还有个宽度限制的样式没有设置，所以它会继承父级宽度，所以才会出现上图这种情况。

**解决方法：**

    限制父级宽度并且父级宽度做到剩余空间自动分配。这句话搭配起来的意思就是left盒子 `width: 0; flex-grow: 1;`   将left盒子的宽度设置为 0 并将多余的空间全部分配给 left盒子, 这样 p标签的父级 就不会继承 left盒子样式中width宽度，继承的是 flex-grow 自动分配的宽度；这样需求中的问题也就解决了

![](../../../.gitbook/assets/image%20%2811%29.png)





