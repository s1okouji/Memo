# HAProxyをリバースプロキシとして利用する際のconfig例

## config例

以下のようにすると2000番ポートにアクセスされたリクエストを5000番ポートで待ち受けるサーバーに送ることができる.

```
defaults
    mode tcp
    timeout connect 5000ms
    timeout client 50000ms
    timeout server 50000ms

frontend foo
    bind :::2000 v4v6
    default_backend bar

backend bar
    server server01 <<ip>>:5000
```
