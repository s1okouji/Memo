# ScriptableObject

## 特徴

- ScriptableObjectを使うと、値のコピーを避けられる
- エディターの場合はデータをScriptableObjectとして保存することが出来る
- MonoBehaviourにアタッチすることは出来ない

## 主な使用場面

- エディターでのデータ保存 (セッションの永続化が目的)
- プロジェクトのデータを保存し、ランタイムで使用

## Editorのアセットメニューに追加する方法

```c#
[CreateAssetMenu(fileName = "Data", menuName = "ScriptableObjects/SpawnManagerScriptableObject", order = 1)]
```

- fileName: デフォルトのファイル名
- menuName: `Assets/Create`メニューでの表示名
- order: `Assets/Create`メニュー内の表示位置

## 注意点

- ScriptableObjectを継承したクラスのクラス名とファイル名を一致させる必要がある
  - `public class Item:ScriptableObject`と宣言したなら`Item.cs`というファイル名にする必要がある

## 参考

- <https://docs.unity3d.com/ja/2022.3/Manual/class-ScriptableObject.html>
- <https://docs.unity3d.com/ja/2022.3/ScriptReference/CreateAssetMenuAttribute.html>
