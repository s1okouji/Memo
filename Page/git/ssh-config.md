# gitのsshのためにconfigを書く

## やること

`C:\Users\<UserName>\.ssh`フォルダ以下に`config`ファイルを生成し、中に以下を追加する．

```
Host github.com
    HostName github.com
    User git
    IdentityFile <gitに登録した公開鍵に対応する秘密鍵の絶対パス>
```

これで鍵・ユーザー名を指定せずともcloneができる.
鍵のパスワードに関してはssh-agentを利用することで、入力を省略できる.

ただsubmoduleのcloneの際は別途パスワードが聞かれてしまうので、その場合はssh-agentの設定とgit for windowsで使用するssh-clientをwindowsに搭載されているOpenSSHバイナリに指定する必要があった気がする。

git submoduleコマンドのssh実行プロセスが通常のgitコマンドと異なるのかという疑問はいまだ解決できていない.
