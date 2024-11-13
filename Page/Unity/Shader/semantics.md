# セマンティクス

- セマンティクスは、シェーダーの入力または出力のパラメータの使用目的を示す文字列.
- シェーダーステージ間で渡される全ての変数で必要.
- 基本的にパイプライン間で渡されるデータはシステムによって解釈されることはない.
  - しかし一部のセマンティクスはシステムによって特殊な解釈をされるものがある.
  - このようなセマンティクスは、SystemValueセマンティクスと呼ばれる. （Direct3D 10以降）

## 頂点シェーダーのセマンティクス

| 入力 | 説明 | Type |
| --- | --- | --- |
| BINORMAL[n] | 従法線 | float4 |
| BLENDINDICES[n] | Blend Index | uint |
| BLENDWEIGHT[n] | Blend Weight | float |
| COLOR[n] | 拡散と反射色 | float4 |
| NORMAL[n] | 法線ベクトル | float4 |
| POSITION[n] | オブジェクト空間内の頂点の位置 | float4 |
| POSITIONT | 変換された頂点の位置 | float4 |
| PSIZE[n] | ポイントのサイズ | float |
| TANGENT[n] | タンジェント | float4 |
| TEXCOORD[n] | テクスチャの座標 | float4 |
| SV_InstanceID | ランタイムによって自動生成されるインスタンスごとの識別子 | uint |
| SV_VertexID | ランタイムによって自動生成される頂点ごとの識別子 | uint |


| 出力 | 説明 | Type |
| --- | --- | --- |
| COLOR[n] | 拡散または反射色 | float4 |
| FOG | 頂点のフォグ | float |
| POSITIOn[n] | スクリーンスペースでの位置, Direct3D 9で使用可能 | float4 |
| PSIZE | ポイントのサイズ | float |
| TESSFACTOR[n] | テッセレーション係数 | float |
| SV_ClipDistance[n] | クリップ距離データ | float|
| SV_CullDistance[n] | カリング距離データ | float |
| SV_Position[n] | 同種空間における頂点の位置. (x,y,z)をwで除算した値がスクリーンスペースにおける位置になる. 頂点シェーダーで必ず書き出す必要がある (**Direct3D 10以降で使用可**) | float4 |

※ nは、0からサポートされるリソース数までの省略可能な整数

## ピクセルシェーダーのセマンティクス

## 参考

- https://learn.microsoft.com/en-us/windows/win32/direct3dhlsl/dx-graphics-hlsl-semantics
- https://learn.microsoft.com/ja-jp/windows/win32/direct3d11/d3d10-graphics-programming-guide-input-assembler-stage-using#instanceid
