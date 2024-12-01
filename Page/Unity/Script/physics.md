# Phsicsについてのメモ

## RaycastNonAlloc

### メリット

- `Physics.RaycastNonAlloc`は結果を格納する配列（バッファ）を、呼び出し側が提供するので毎フレームメモリ確保を行う必要がない

### デメリット

- 指定したバッファの数よりも多くRaycastがHitしていた場合、正確な情報は得られない。なぜなら配列に格納される順序は未定義であるため

ただし戻り値がヒットした数であるため次フレーム以降でバッファサイズを修正することはできる。

そのため大きなバグ要因には成りえないと考えるが、稀に欲しい結果と異なる結果になることを留意しておくべきだと思う。

### 参考

<https://docs.unity3d.com/jp/2021.3/ScriptReference/Physics.RaycastNonAlloc.html>
