# tranform-origin + transform

自己可以先简单的实现 hover 默认的两种状态实现下滑线滑动

使用元素的 before 和 after , 定位到当前操作的为父级元素上, 效果前后利用 hover 和 transition 实现过度动画, 方便往下进阶。

### 达到的效果

采用 transition\(过度效果\)、transform: scaleX \(缩放\)、transform-origin\(改变原点位置\)，实现下滑线从左边近，右边出

#### 首先理解 hover 选择器的作用, 使用它时元素被分解成三个步骤

* hover 前\(默认样式\)
* hover 中\(:hover 设置样式\)
* hover 后\(默认样式\)

由此可以看出 hover 正常状态下只有两种状态

所以，必须要有一种方法，能够使得 hover 动画的进入与离开产生两种不一样的效果, 实现:

* hover 前设置原点位置 transform-orgin: right 0\)、缩放大小 transform: scaleX\(0\), 隐藏 before 选择器
* hover 中\(设置原点位置 transform-orgin: 0 0\)、缩放大小 transform: scaleX\(1\), 显示 before 选择器
* hover 前\(恢复 transform: scaleX\(0\)、transform-orgin: right 0\), 向右隐藏 before 选择器

### 代码如下

```text
<div class="box">Hover default</div>

.box {
  position: relative;
  left: 25%;
  width: 200px;
  height: 60px;
  line-height: 60px;
  color: #333;
  text-align: center;
  transition: color .5s;
  border-top: 2px solid #e2e2e2;
  border-bottom: 2px solid #e2e2e2;
}
.box::before {
  content: '';
  position: absolute;
  left: 0px;
  bottom: 0px;
  width: 200px;
  height: 2px;
  background-color: #000;
  transition: transform .5s;
  transform: scaleX(0);
  transform-origin: right 0;
}
.box::after {
  content: '';
  position: absolute;
  right: 0px;
  top: -2px;
  width: 200px;
  height: 2px;
  background-color: #000;
  transition: transform .5s;
  transform: scaleX(0);
  transform-origin: left 0;
}
.box:hover {
  color: red;
}
.box:hover::before {
  transform: scaleX(1);
  transform-origin: 0 0;
}
.box:hover::after {
 transform: scaleX(1);
 transform-origin: right 0;
}
```

### 参考文档

transform-origin 取值:

* x-axis 默认以元素框上边框为 x正半轴, 上边框与左边框的焦点为顶点; 水平偏移量有以下几种取值方式
  * left 等于 0 或 0%
  * center 等于 50%
  * right 等于 100%
* y-axis 以元素框左边框为 y正半轴, 左边框与上边框的焦点为顶点; 水平偏移量有以下几种取值方式
  * top 等于 0 或 0%
  * center 等于 50%
  * right 等于 100%
* z-axis 3D变形中会用到 详解见文档

transform-origin 实例解析:  [https://developer.mozilla.org/zh-CN/docs/Web/CSS/transform-origin](https://developer.mozilla.org/zh-CN/docs/Web/CSS/transform-origin)

transform-origin 坐标系图析:  [https://www.jianshu.com/p/fd44d21287ea](https://www.jianshu.com/p/fd44d21287ea)







