# カスタムコントロールの作成手順

## 手順 (Unity 6)

1. `VisualElement`クラスを継承したクラスを作成する
2. 1.で継承したクラスに`partial`モディファイアを付与する
3. 1で継承したクラスに`UxmlElement`属性を付与する

### カスタム属性を付けたい場合

プロパティを定義し、`[UxmlAttribute]`属性を付与する.

この際値の実態はフィールドで持つ. (どこかのUnity JapanのTutorial動画で説明してくれていたはず)

またプロパティのsetterには、値変更時に更新を行うため`MarkDirtyRepaint()`メソッドを呼び出す

まとめると以下のようになる

```C#:Sample.cs
    private int _value = 100;

    [UxmlAttribute]
    public int Value
    {
        get => _value;
        set
        {
            _value = value;
            MarkDirtyRepaint();
        }
    }
```

### カスタム属性に範囲制限を設けたいとき

プロパティのsetterで値を加工してやることで実現ができる

```c#
[UxmlAttribute]
public int Value
{
    get => _value;
    set
    {
        var tmp = value >= 0 ? value : 0;
        _value = value > _max ? _max : tmp;
        MarkDirtyRepaint();
    }
}
```

### Binding可能にするには

`[CreateProperty]`を付与すると、プロパティバッグが作られbindできるようになる.

```c#
[UxmlAttribute]
[CreateProperty]
public int Value
{
    get => _value;
    set
    {
        var tmp = value >= 0 ? value : 0;
        _value = value > _max ? _max : tmp;
        MarkDirtyRepaint();
    }
}
```

## 手順 (Unity 2022以前)

1. `VisualElement`クラスを継承したクラスを作成する.
2. `VisualElement.UxmlTraits`クラスを継承したクラス(`UxmlTraits`)を内部クラスとして作成する
3. `UxmlFactory`クラスを作成する
