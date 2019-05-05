---
title: 实现一个仿win7的字块碰撞动画
date: 2019-04-30 11:58:44
tags:
    - flutter
---

## 效果图GIF

<div>![效果图](/images/crash.gif)</div>

代码已提交至[github](https://github.com/sakurahe/flutter_animation/blob/master/lib/example/animate_demo.dart)

## 实现

### 我们先定义一个移动盒子的class

```
class MoveBox {
  double x; // 初始x轴位置
  double y; // 初始y轴位置
  double speedX; // x速度
  double speedY; // y速度
  double block; // 移动盒子大小
  Color color; // 盒子颜色

  MoveBox({this.x, this.y, this.speedX, this.speedY, this.block, this.color});
}
```

### 我们还需要定义盒子所移动的范围

```
 double _width = 0.0; // 设备宽
 double _height = 0.0; // 设备高
 Widget build(BuildContext context) {
   ...
   setState(() {
      _width = MediaQuery.of(context).size.width;
      _height = MediaQuery.of(context).size.height -
          appBar.preferredSize.height -
          MediaQuery.of(context).padding.top;
    });
   ...
 }
```

这里需要在生成widget树重新对宽和高进行赋值

### 更新盒子位置

```
moveBox.x += moveBox.speedX;
moveBox.y += moveBox.speedY;
// 限定下边界
if (moveBox.y > _height - moveBox.block) {
  moveBox.y = _height - moveBox.block;
  moveBox.speedY = -moveBox.speedY;
  randomColor();
}
// 限定上边界
if (moveBox.y < 0) {
  moveBox.y = 0;
  moveBox.speedY = -moveBox.speedY;
  randomColor();
}
// 限定左边界
if (moveBox.x < 0) {
  moveBox.x = 0;
  moveBox.speedX = -moveBox.speedX;
  randomColor();
}
// 限定右边界
if (moveBox.x > _width - moveBox.block) {
  moveBox.x = _width - moveBox.block;
  moveBox.speedX = -moveBox.speedX;
  randomColor();
}
```

在这里，每次碰撞到边界就会重新随机一种颜色

这样，碰撞盒子就完成了。
