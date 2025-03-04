# ScrollView

## スクロールできない問題

ScrollはContentの大きさによって出来るかが決まる。そのためContentが子GameObjectの範囲全てを覆うために、ContentSizeFitterを利用する.

ContentSizeFitterは子要素にLayoutElementコンポーネントがアタッチされている必要があるので、予めアタッチしておく。またLayoutElementのMin, Preferredのどちらかにサイズを設定し、設定した方の値をContentSizeFitterのFit項目として選択する。

<https://docs.unity3d.com/ja/2022.3/Manual/script-ContentSizeFitter.html>
