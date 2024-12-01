# Animation Controllerによって操作されるTransformをスクリプトから動的に変更する

## スクリプトから動的に更新するには

Animatorの更新はデフォルトではNnormal、即ちUpdateの呼び出しと同期して行われる (※1のUpdateModeを参照)
しかし同期とは言いつつも、Animatorの更新はUpdateメソッド実行後に行われるのでUpdateメソッド内でAnimatorのTransformを変更しても上書されてしまう. (※2を参照)

そのためスクリプトから変更するならAnimatorの更新後に実行されるLateUpdateメソッドを使用すると良い. (※3)

以上をまとめると以下のようなスクリプトを書けば簡単にAnimationControllerで再生中のAnimatorにTransformを上書できる.

```c#
    private void LateUpdate()
    {
        var chestTransform = GetComponent<Animator>().GetBoneTransform(HumanBodyBones.Chest);
        chestTransform.rotation = Quaternion.identity;
    }
```

## 参考

1. <https://docs.unity3d.com/ja/2022.3/Manual/class-Animator.html>
2. <https://docs.unity3d.com/ja/2022.3/Manual/ExecutionOrder.html>
3. <https://docs.unity3d.com/ja/2022.3/ScriptReference/MonoBehaviour.LateUpdate.html>
