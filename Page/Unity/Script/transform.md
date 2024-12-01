# Transformに関するメモ

## 存在確認の方法

Transformの存在確認で、`is not null`を使うと思ったような挙動にならないケースがある.

例）`[SerializeField]`を付けた`Transform`型変数に何も値を入れない場合 (Inspector上ではNone表記)

この際、`is not null`を使うと、`null`ではないので効果を発揮しない。
どうやら無効な値が入っているらしくエラー扱いになる。

一応`!=`と`==`に関しては演算子がoverrideされているため、`!= null`を使うと存在確認を行うこと自体はできる。

ただUnity側はbool演算子をoverrideしているので、存在確認をしたいならbool演算子を利用するべきだろう。

しかし自身でnullを代入した場合は、逆にbool演算子が利用できない。また`!= null`はoverrideされており呼び出しコストが高い。

この場合は`is not null`や`is null`を利用するべきだろう. なぜなら`is null`等はoverrideされた実装を呼び出さないことが保証されているからだ。

つまり存在確認を行う際は、`if(hoge && hoge is not null)`とするのが良いだろう.

### 参考

<https://docs.unity3d.com/ja/2023.2/ScriptReference/Transform.html>

<https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/patterns#constant-pattern>
