# Let's Encrypt 証明書の更新

## 手順

1. rootにログインする (または以下のコマンドをsudoで実行)
1. `docker compose run --rm certbot renew`を実行する

証明書取得のやり方の方が大事だとは思うのだが、忘れてしまった．
更新の自動化はcron使ったり、systemd使ったり色々やり方はある.
3ヶ月後にやろうかな.
