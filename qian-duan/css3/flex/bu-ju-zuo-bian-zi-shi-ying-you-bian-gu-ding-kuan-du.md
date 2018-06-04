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

![](../../../.gitbook/assets/image%20%2828%29.png)

## 项目中踩坑集锦

接着以上的做法，应用到实际的项目需求中时，发生了下面👇意料之外的情况：

> 需求:  左侧包含的内容有标题、描述\(超出并显示省略号\)，右侧是一张固定宽高的图片。

根据需求建立如下图：

![](../../../.gitbook/assets/image%20%2813%29.png)

  
现在就要解决的是描述\(**超出并显示省略号\)** 于

  


