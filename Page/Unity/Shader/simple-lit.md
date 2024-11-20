# Simple Lit 解読メモ

## 頂点シェーダー

### ソースコード

```
Varyings LitPassVertexSimple(Attributes input)
{
    Varyings output = (Varyings)0;

    UNITY_SETUP_INSTANCE_ID(input);
    UNITY_TRANSFER_INSTANCE_ID(input, output);
    UNITY_INITIALIZE_VERTEX_OUTPUT_STEREO(output);

    VertexPositionInputs vertexInput = GetVertexPositionInputs(input.positionOS.xyz);
    VertexNormalInputs normalInput = GetVertexNormalInputs(input.normalOS, input.tangentOS);

#if defined(_FOG_FRAGMENT)
        half fogFactor = 0;
#else
        half fogFactor = ComputeFogFactor(vertexInput.positionCS.z);
#endif

    output.uv = TRANSFORM_TEX(input.texcoord, _BaseMap);
    output.positionWS.xyz = vertexInput.positionWS;
    output.positionCS = vertexInput.positionCS;

#ifdef _NORMALMAP
    half3 viewDirWS = GetWorldSpaceViewDir(vertexInput.positionWS);
    output.normalWS = half4(normalInput.normalWS, viewDirWS.x);
    output.tangentWS = half4(normalInput.tangentWS, viewDirWS.y);
    output.bitangentWS = half4(normalInput.bitangentWS, viewDirWS.z);
#else
    output.normalWS = NormalizeNormalPerVertex(normalInput.normalWS);
#endif

    OUTPUT_LIGHTMAP_UV(input.staticLightmapUV, unity_LightmapST, output.staticLightmapUV);
#ifdef DYNAMICLIGHTMAP_ON
    output.dynamicLightmapUV = input.dynamicLightmapUV.xy * unity_DynamicLightmapST.xy + unity_DynamicLightmapST.zw;
#endif
    OUTPUT_SH4(vertexInput.positionWS, output.normalWS.xyz, GetWorldSpaceNormalizeViewDir(vertexInput.positionWS), output.vertexSH, output.probeOcclusion);

    #ifdef _ADDITIONAL_LIGHTS_VERTEX
        half3 vertexLight = VertexLighting(vertexInput.positionWS, normalInput.normalWS);
        output.fogFactorAndVertexLight = half4(fogFactor, vertexLight);
    #else
        output.fogFactor = fogFactor;
    #endif

    #if defined(REQUIRES_VERTEX_SHADOW_COORD_INTERPOLATOR)
        output.shadowCoord = GetShadowCoord(vertexInput);
    #endif

    return output;
}
```

### 定義

シェーダーの戻り値として使われている `Varyings`型の定義は以下

```
struct Varyings
{
    float2 uv                       : TEXCOORD0;

    float3 positionWS                  : TEXCOORD1;    // xyz: posWS

    #ifdef _NORMALMAP
        half4 normalWS                 : TEXCOORD2;    // xyz: normal, w: viewDir.x
        half4 tangentWS                : TEXCOORD3;    // xyz: tangent, w: viewDir.y
        half4 bitangentWS              : TEXCOORD4;    // xyz: bitangent, w: viewDir.z
    #else
        half3  normalWS                : TEXCOORD2;
    #endif

    #ifdef _ADDITIONAL_LIGHTS_VERTEX
        half4 fogFactorAndVertexLight  : TEXCOORD5; // x: fogFactor, yzw: vertex light
    #else
        half  fogFactor                 : TEXCOORD5;
    #endif

    #if defined(REQUIRES_VERTEX_SHADOW_COORD_INTERPOLATOR)
        float4 shadowCoord             : TEXCOORD6;
    #endif

    DECLARE_LIGHTMAP_OR_SH(staticLightmapUV, vertexSH, 7);

#ifdef DYNAMICLIGHTMAP_ON
    float2  dynamicLightmapUV : TEXCOORD8; // Dynamic lightmap UVs
#endif

#ifdef USE_APV_PROBE_OCCLUSION
    float4 probeOcclusion : TEXCOORD9;
#endif

    float4 positionCS                  : SV_POSITION;
    UNITY_VERTEX_INPUT_INSTANCE_ID
    UNITY_VERTEX_OUTPUT_STEREO
};
```

### 使用されているマクロ

#### UNITY_SETUP_INSTANCE_ID(input)

```
// - UNITY_SETUP_INSTANCE_ID        Should be used at the very beginning of the vertex shader / fragment shader / ray tracing hit shaders,
//                                  so that succeeding code can have access to the global unity_InstanceID.
//                                  Also procedural function is called to setup instance data.
```

`unity_InstanceID`というグローバルキーワードへインスタンスIDを設定する処理と
インスタンス生成を行うマクロ`UnityInstancing.cginc`内に定義されている.

恐らくGPUインスタンシングを有効化にすることで使用可能になる

#### UNITY_TRANSFER_INSTANCE_ID(input, output)

```
- UNITY_TRANSFER_INSTANCE_ID     Copy instance ID from input struct to output struct. Used in vertex shader.
```
`UnityInstancing.cginc`内で定義されているマクロ. 
一つ目の引数に指定した構造体からInstanceIDをコピーし、二つ目の引数に値を入れる処理を行う

以下が実際の定義
```
#define UNITY_TRANSFER_INSTANCE_ID(input, output)   output.instanceID = UNITY_GET_INSTANCE_ID(input)
```

#### UNITY_INITIALIZE_VERTEX_OUTPUT_STEREO(output)

```
basic stereo instancing setups
- UNITY_INITIALIZE_VERTEX_OUTPUT_STEREO  Assign the stereo target eye.
```

VRゴーグルなどの二つのレンダーターゲットが必要な場合に使用される.

- 参考
  - https://docs.unity3d.com/ja/2018.4/Manual/SinglePassInstancing.html

#### TRANSFORM_TEX(tex,name)

`UnityCG.cginc`内で定義されているマクロ
Textureを入力した際のタイリングとオフセットを機能させる
内部では、`_ST`というサフィックスを持つテクスチャプロパティを宣言する必要がある.
SimpleLitShaderの場合は、`SimpleLitInput.hlsl`内に定義されている.

```
CBUFFER_START(UnityPerMaterial)
    float4 _BaseMap_ST;
    half4 _BaseColor;
    half4 _SpecColor;
    half4 _EmissionColor;
    half _Cutoff;
    half _Surface;
    UNITY_TEXTURE_STREAMING_DEBUG_VARS;
CBUFFER_END
```

- 参考
  - https://docs.unity3d.com/ja/Packages/com.unity.render-pipelines.universal@14.0/manual/writing-shaders-urp-unlit-texture.html

#### OUTPUT_LIGHTMAP_UV

```
#define OUTPUT_LIGHTMAP_UV(lightmapUV, lightmapScaleOffset, OUT) OUT.xy = lightmapUV.xy * lightmapScaleOffset.xy + lightmapScaleOffset.zw;
```

LightMapのScaleとOffsetを適用させるマクロ.
`Lighting.hlsl`内に定義されている

#### OUTPUT_SH4

```
OUTPUT_SH4(absolutePositionWS, normalWS, viewDir, OUT, OUT_OCCLUSION) OUT.xyz = SampleProbeSHVertex(absolutePositionWS, normalWS, viewDir, OUT_OCCLUSION)
```

`Lighting.hlsl`内に定義されているマクロ. LightMapがOFFのとき(すなわちRealTimeRenderingのとき)にのみ展開される.
SampleProbeSHVertexはGlobalIlluminationのLightProbeやSkyboxの環境光を保存するAmbientProbeなどの計算結果を3次元の32bit浮動小数点のベクトルで返す関数
このマクロではOUTに入る3次元のテクスチャの座標として格納される. (これはDECLARE_LIGHTMAP_OR_SHマクロで宣言される)

### 関数

#### VertexPositionInputs GetVertexPositionInputs(float3 positionOS)

- 実装箇所: `ShaderVariablesFunctions.hlsl`
- 処理内容: ワールド空間、ビュー空間、クリップ空間に変換をする

中ではbuild-inのシェーダー関数が使われている. 詳細はURP内の`built-in-shader-methods.md`を参照

#### VertexNormalInputs GetVertexNormalInputs(float3 normalOS, float4 tangentOS)

- 実装箇所: `ShaderVariablesFunctions.hlsl`
- 処理内容: オブジェクト空間の法線情報からワールド空間の法線情報を計算する関数

※ mikkitsのTangent Space Normal Mapsに準拠した実装になっている.

参考: http://www.mikktspace.com/


#### real ComputeFogFactor(float z)

- 実装箇所: `ShaderVariablesFunctions.hlsl`
- 処理内容: Fog項を計算する

#### float3 GetWorldSpaceViewDir(float3 positionWS)

- 実装箇所: `ShaderVariablesFunctions.hlsl`
- 処理内容: 引数に入力されたワールド空間上のposition**から**ビューまでの向きを計算する (正規化はされない)

#### real3 NormalizeNormalPerVertex(real3 normalWS)

- 実装箇所: `ShaderVariablesFunctions.hlsl`
- 処理内容: ワールド空間の法線の正規化を行う

skinningやshapesのblendによって、法線の長さが著しく変化することがあるため正規化を行う

#### float4 GetShadowCoord(VertexPositionInputs vertexInput)

- 実装箇所: `Shadows.hlsl`
- 処理内容: Main Lightの場合は、スクリーンにおける座標を返す。それ以外の場合はShadowCoordを返す

### 処理内容

```text

Vertexシェーダー(input)
{
    <<初期化>>
    <<InsntaceIdの設定>>
    <<頂点ステレオ化の初期化>>

    <<ワールド空間・ビュー空間・クリップ空間上の頂点座標を計算>>
    <<ワールド空間のNormalを計算>>

    if(Fogが有効である)
    {
        <<FogFactorを計算>>
    }

    <<BaseMapに対しスケールとオフセットを適用する(uv座標を計算する)>>
    <<頂点座標をOutput用構造体へコピー>>

    if(NormlMapが存在している)
    {
        <<頂点からビューへの向きを計算する>>
    }else{
        <<ワールド空間の法線を正規化する>>
    }

    <<LightMapのスケールとオフセットを適用する(uv座標を計算する)>>

    if(動的ライトマップが有効)
    {
        <<DynamicLightMapのスケールとオフセットを適用させる (uv座標を計算する)>>
    }

    <<GIの計算を行う>>

    if(頂点ライティングが有効である)
    {
        <<頂点ライトの計算>>
    }
}

```

## フラグメントシェーダー [WIP]

### マクロ

#### UNITY_SETUP_INSTANCE_ID(input)

// TODO: アンカーで飛べるようにする
Vertex Shaderでも説明したため省略

#### UNITY_SETUP_STEREO_EYE_INDEX_POST_VERTEX

- 配置場所: `UnityInstancing.cginc`
- 処理内容: ステレオマルチビューが有効な時に`unity_StereoEyeIndex`キーワードを設定する

#### SETUP_DEBUG_TEXTURE_DATA(inputData, uv) 

```
#define SETUP_DEBUG_TEXTURE_DATA(inputData, uv)                   SetupDebugDataTexture(inputData, TRANSFORM_TEX(uv.xy, unity_MipmapStreaming_DebugTex), unity_MipmapStreaming_DebugTex_TexelSize, unity_MipmapStreaming_DebugTex_MipInfo, unity_MipmapStreaming_DebugTex_StreamInfo, unity_MipmapStreaming_DebugTex)
```

- 配置場所: `Debugging3D.hlsl`
- 処理内容: 

#### void ApplyDecalToSurfaceData(float4 positionCS, inout SurfaceData surfaceData, inout InputData inputData)



### 関数

#### inline void InitializeSimpleLitSurfaceData(float2 uv, out SurfaceData outSurfaceData)

#### void LODFadeCrossFade(float4 positionCS)

- 配置場所: `LODCrossFade.hlsl`
- 処理内容: 距離に応じてディザリング画像と混ぜ合わせる

参考: https://docs.unity3d.com/ja/2022.3/Manual/class-LODGroup.html
