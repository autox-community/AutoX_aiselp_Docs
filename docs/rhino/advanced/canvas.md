# Canvas 画布

canvas 提供了使用画布进行 2D 画图的支持，可用于简单的小游戏开发或者图片编辑。使用 canvas 可以轻松地在一张图片或一个界面上绘制各种线与图形。

canvas 的坐标系为平面直角坐标系，以屏幕左上角为原点，屏幕上边为 x 轴正方向，屏幕左边为 y 轴正方向。例如分辨率为 1920\*1080 的屏幕上，画一条从屏幕左上角到屏幕右下角的线段为:

```js
canvas.drawLine(0, 0, 1080, 1920, paint);
```

canvas 的绘制依赖于画笔 Paint, 通过设置画笔的粗细、颜色、填充等可以改变绘制出来的图形。例如绘制一个红色实心正方形为：

```js
var paint = new Paint();
//设置画笔为填充，则绘制出来的图形都是实心的
paint.setStyle(Paint.STYLE.FILL);
//设置画笔颜色为红色
paint.setColor(colors.RED);
//绘制一个从坐标(0, 0)到坐标(100, 100)的正方形
canvas.drawRect(0, 0, 100, 100, paint);
```

如果要绘制正方形的边框，则通过设置画笔的 Style 来实现：

```js
var paint = new Paint();
//设置画笔为描边，则绘制出来的图形都是轮廓
paint.setStyle(Paint.STYLE.STROKE);
//设置画笔颜色为红色
paint.setColor(colors.RED);
//绘制一个从坐标(0, 0)到坐标(100, 100)的正方形
canvas.drawRect(0, 0, 100, 100, paint);
```

结合画笔 canvas 可以绘制基本图形、图片等。

## canvas.drawARGB(a, r, g, b)

绘制 argb 颜色到画布，每个参数为`0~255`的整数，例如

```js
canvas.drawARGB(255, 255, 0, 0);
```

## 在 ui 中使用画布

我们可以使用如下代码来创建一个带有 canvas 的界面

```js
"ui";
ui.layout(
  <frame>
    <canvas id="board" w="*" h="*" />
  </frame>
);
ui.board.on("draw", function (canvas) {
  canvas.drawARGB(0xff, 0xff, 0xff, 0xff);
});
```

然后在`draw`事件中可以进行绘制图像  
需要注意的是`ui.board`并不是`canvas`，而是一个`TextureView`,由于历史设计此名称为 canvas 很容易造成混浊，在 v7.1.7 版本后可使用别名`texture`,为兼容性考虑`canvas`名称任然保留。  
**`draw`事件会在 ui 创建后会有后台线程频繁触发以更新 canvas**，请勿在该含回调函数中做任何与绘制 canvas 无关的事，**在 v7.1.6 版本取消了这一行为**，`draw`事件默认仅会触发一次，如果需要更新`canvas`则使用`ui.board.updateCanvas()`来手动触发`draw`事件，如果需要回到以前的行为只需添加以下代码

```js
"ui";
ui.layout(
  <frame>
    <canvas id="board" w="*" h="*" />
  </frame>
);
ui.board.on("draw", function (canvas) {
  canvas.drawARGB(0xff, 0xff, 0xff, 0xff);
});

threads.start(() => {
  let board = ui.board;
  setInterval(() => {
    board.updateCanvas();
  }, 1000 / 60);
});
```

在 'app 示例/画布' 中可查看更多例子
