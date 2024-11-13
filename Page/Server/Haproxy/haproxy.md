# HAProxyの使い方

## インストール

Ubuntuの場合

`apt -y install haproxy`

## Config

インストールされると自動的に生成される.

- 場所: `/etc/haproxy/haproxy.cfg`

## Configの再読み込み

systemdのreloadでconfigが再度読み込まれる

`systemctl reload haproxy.service`

restartでは再読み込みされないので注意
