# Transformに関するメモ

## 存在確認で`is not null`は使うな

Transformの存在確認で、`is not null`を使うと思ったような挙動にならないケースがある.

例）`[SerializeField]`を付けた`Transform`型変数に何も値を入れない場合 (Inspector上ではNone表記)

この際、`is not null`を使うと、`null`ではないので効果を発揮しない。
しかしながら値としては無効な値が入っているらしくエラー扱いになる。

一応`!=`と`==`に関しては演算子がoverrideされているため、`!= null`を使うと存在確認を行うこと自体はできる。

ただUnity側はbool演算子をoverrideしているので、存在確認をしたいならbool演算子を利用するべきだろう。

(要は`if(hoge)`で存在確認を行おうということ)

### 参考

<https://docs.unity3d.com/ja/2023.2/ScriptReference/Transform.html>

 