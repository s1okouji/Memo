# docker compose downするタイミングはしっかり見極めよう.

RSSフィーダーとしてtiny tiny rss（以降tt-rssと呼称する）をVPS上で動かして動かしている.

tt-rssは現在docker版が推奨されているため、dockerを利用.

SSL化にはLet's encryptを使用している.

Let's encryptの更新をした後Webサーバーを再起動させる必要があったのだが、その際に`docker compose down`コマンドを使用. この時にdbサーバーも一緒にダウンしてしまったため、全て吹っ飛んだという話. 

少しの痛手で済んだのが幸い.

## docker compose downをすると基本的に全て消える

当たり前だがdbサーバーをdownさせると、中のデータは基本消える.

### 解決策

1. そもそも`down - up`ではなく、`stop - start`を使う.
2. どうしてもダウンしないといけない場合は、downしないといけない物だけdownをする.
   - 参考: https://aton-kish.github.io/blog/post/2020/10/04/docker-compose-rm/
   - ```docker-compose up {{ service name }}```
   - ```docker-compose down {{ service name }}```を使えば良い.
