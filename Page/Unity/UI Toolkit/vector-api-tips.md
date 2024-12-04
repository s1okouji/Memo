# Vector APIのTips

## Stroke, Fillを別メソッドに分ける

Vector APIではPainter2Dインスタンスを使用し、線や塗りつぶしを行う。

この時お作法的なものが必要なので、別メソッドにまとめた方が楽。

以下は直線を引く際の処理を拡張メソッドとして切り出したもの。

```c#
public static void StrokeLine(this Painter2D painter, Vector2 startPoint, Vector2 endPoint)
{
    painter.BeginPath();
    painter.MoveTo(startPoint);
    painter.LineTo(endPoint);
    painter.Stroke();
    painter.ClosePath();
}
```

同様にFillでも同じことが行える。

以下の一つ目のメソッドは2点で生成される長方形を塗りつぶすメソッド。

二つ目は時計回りに配置した4点で囲われる長方形を塗りつぶすメソッド。今の実装では時計回りに頂点を指定する必要があるが、少し処理を変えれば任意の4点に対応できそう。

```c#
public static void FillArea(this Painter2D painter, Vector2 startPoint, Vector2 endPoint)
{
    var x1 = startPoint.x;
    var y1 = startPoint.y;
    var x2 = endPoint.x;
    var y2 = endPoint.y;
    painter.FillArea(startPoint, new Vector2(x2,y1), endPoint, new Vector2(x1,y2));
}

public static void FillArea(this Painter2D painter, Vector2 point1, Vector2 point2, Vector2 point3, Vector2 point4)
{
    painter.BeginPath();
    painter.MoveTo(point1);
    painter.LineTo(point2);
    painter.LineTo(point3);
    painter.LineTo(point4);
    painter.ClosePath();
    painter.Fill();
}
```
