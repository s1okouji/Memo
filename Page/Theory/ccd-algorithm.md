# CCDアルゴリズム

## 概要

CCD法は、Cyclic Coordinate Descentと呼ばれるIK(逆運動学計算)のアルゴリズム.

末端の対象から順に回転量の最適化を行うフェーズを反復することで、最適解を求める．

シンプルかつ解の収束が早い（らしい）.

なので一般的な手法（らしい）.

## 用語

- エフェクタ
  - 変化を受ける位置? (正確な定義は見つからず..)
  - 大抵の場合エンド・エフェクタ (スケルトンの末端)を指す
- 目標
  - エンド・エフェクタを近づけさせる対象

## 計算方法

- IKはエンド・エフェクタの位置と目標位置の誤差が最小になるような各関節の回転量を求める
- CCD法は末端から各関節の回転量を最適化するフェーズを繰り返すことで目標位置との誤差を最小化する

詳しくは参考資料を参照．

## コード (in Unity)

現在は閾値による打ち切りは実装していない.

それでも解の収束が早いので計算量的には問題がない.

続きはいつか作る

```c#

    public static void SolveCcdIK(int maxIterations, Transform[] jointsTransforms, Transform effectorTransform, Vector3 targetPos)
    {
        // 閾値の設定 (未実装)

        for (var i = 0; i < maxIterations; i++)
        {
            foreach (var jointTransform in jointsTransforms)
            {
                var effectorPos = effectorTransform.position;
                var localEffectorPos = jointTransform.InverseTransformPoint(effectorPos);
                var localTargetPos = jointTransform.InverseTransformPoint(targetPos);
                
                var basis2Effector = Vector3.Normalize(localEffectorPos);
                var basis2Target = Vector3.Normalize(localTargetPos);
                
                // 回転角
                var rotationDotProduct = Vector3.Dot(basis2Effector, basis2Target);
                var angle = Mathf.Acos(rotationDotProduct);
                
                // 閾値チェック (未実装)
                
                var rotationAxis = Vector3.Cross(basis2Effector, basis2Target);
                rotationAxis.Normalize();
                jointTransform.rotation *= Quaternion.AngleAxis(Mathf.Rad2Deg * angle, rotationAxis);
            }
            // 閾値チェック（未実装）
        }
    }

```

## 参考資料

- https://mukai-lab.org/content/CcdParticleInverseKinematics.pdf
  - 東京都市大学の向井研究室の記事です. 概念の理解・コード共に参考にさせて頂きました.
