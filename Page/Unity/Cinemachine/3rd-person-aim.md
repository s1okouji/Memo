# 3rd Person Aimの使い方

## 概要

3人称視点用のカメラリグの一部.

意図的に回転ノイズを打ち消す役割を持つ.
また`Aim Target Reticle`を指定することで、照準を画面上に自動で配置してくれる.

内部ではRayを一定の距離照射しており、ヒットした位置のグローバル座標を常に`AimTarget`というVector3型の変数に保持している.

## 参考

<https://docs.unity3d.com/ja/Packages/com.unity.cinemachine@2.6/manual/Cinemachine3rdPersonAim.html>
