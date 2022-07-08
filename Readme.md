## 設定モードへ移行
```
en
```

#### 強制ログイン実行 (設定モードへ移行できなかったとき)
```
svintr-config
```

## ホスト名の設定
```
hostname  IX2215
```

## タイムゾーンの設定
```
timezone +09 00
```

## IPv4 UFSキャッシュの設定
最大数の調整と有効化
```
ip ufs-cache max-entries 50000
ip ufs-cache enable
```

## デフォルトルートの設定
普段の通信はTunnel0.0を使用するように設定します。
Tunnel0.0が何かしらの理由で使用不可になった時にGigaEternet0.1を使用するように設定します。
```
ip route default GigaEthernet0.1 distance 200
ip route default Tunnel0.0 distance 20
```

## DHCPの有効化
```
ip dhcp enable
```

## IPv4フィルタの設定
```
ip access-list ipv4_udp_range permit udp src any sport range 1024 65535 dest any dport range 1024 65535
```

## IPv6 UFSキャッシュの設定
最大数の調整と有効化
```
ipv6 ufs-cache max-entries 50000
ipv6 ufs-cache enable
```

## IPv6 DHCPサーバーの設定
```
ipv6 dhcp enable
```

## IPv6フィルタの設定
```
ipv6 access-list ipv6_udp_range permit udp src any sport range 1024 65535 dest any dport range 1024 65535
```

## ブリッジの設定
IPv6のみブリッジしたいのでIPv4のブリッジを無効にします。
```
bridge irb enable
no bridge 1 bridge ip
```

## DNSサーバーの設定
DNSサーバーとキャッシュDNSサーバーを設定。
DNSサーバーは好きなものを使ってください。ここではCLoduflareの`1.1.1.1`と`1.0.0.1`を使用しています。
```
ip name-server 1.1.1.1
ip name-server 1.0.0.1
dns cache enable
dns cache max-records 256
```

## Proxy DNSの設定
```
proxy-dns ip enable
proxy-dns ip request both
proxy-dns interface GigaEthernet0.0 priority 200
proxy-dns ipv6 enable
proxy-dns ipv6 request both
```

## SSHサーバーの有効化
SSHで触れるように、SSHサーバーを有効化します。
```
ssh-server ip enable
```

## Web-GUIの有効化
ウェブブラウザで状態を見ることができるように、httpサーバーを有効化します。
```
http-server authentication-method digest
http-server username admin secret-password xxxxxxxxxxxxxxxxx
http-server ip enable
```

## NetMeisterの設定
NECのNetMeisterとサービスを使って管理できるように設定します。
事前の登録が必要です。
[クラウド型統合管理サービス NetMeister（ネットマイスター）](https://www.necplatforms.co.jp/product/netmeister/index.html)
```
nm ip enable
nm account toyohashi password secret xxxxxxxxxxxxxxxxx
nm sitename toyohashi
nm ddns hostname ix2215
nm ddns notify interface GigaEthernet0.1 protocol ip
nm ddns notify interface GigaEthernet0.0 protocol ipv6
```

## PPPoEのルーティングの設定
```
route-map use-pppoe permit 10
  match ip address access-list server_services
  set interface GigaEthernet0.1
```

## PPP認証の設定
プロバイダーから通知されたユーザー名とパスワードを入力します。
```
ppp profile ISP1
  authentication myname hogehoge.hogehogeppp.net
  authentication password hogehoge@hoge.hogehogeppp.net hogehoge
```

## IPv4 DHCPプロファイルの作成
IP範囲、デフォルトゲートウェイ、DNSサーバー、ドメイン名、リース時間を設定します。
```
ip dhcp profile private
  assignable-range 172.16.1.1 172.168.1.254
  default-gateway 172.16.0.1
  dns-server 172.16.0.1
  domain-name hogehoge
  lease-time 3600
```

## IPv6 DHCPプロファイルの作成
NGNからRAでIPv6アドレスを引っ張ってくるように設定します。
```
ipv6 dhcp client-profile dhcpv6-cl
  information-request
  option-request dns-servers

ipv6 dhcp server-profile dhcp6-sv
  dns-server dhcp
```

## QoSの設定
音声通話などのパケットを優先的に通すように設定します。
```
class-map match-any voice
  match ip access-list ipv4_udp_range normal
  match ipv6 access-list ipv6_udp_range normal
!
policy-map output-policy
  class voice
    set ip dscp 48
    set ipv6 dscp 48
  class class-local
  class class-default
```

## 使わないインターフェースのシャットダウン
```
device USB0
  shutdown
```


## インターフェースの設定
インターフェースにIPを振ったりいろいろします。
```
interface GigaEthernet0.0
  no ip address
  ipv6 enable
  ipv6 address autoconfig receive-default
  ipv6 dhcp client dhcpv6-cl
  bridge-group 1
  no shutdown

interface GigaEthernet1.0
  ip address 172.16.0.1/23
  ip dhcp binding private
  ip ufs-cache timeout tcp 60
  ip ufs-cache timeout udp 300
  ipv6 enable
  ipv6 dhcp server dhcpv6-sv
  bridge-group 1
  service-policy enable
  service-policy input output-policy
  service-policy output output-policy
  no shutdown

interface GigaEthernet2.0
  ip address 192.168.1.1/24
  ip policy route-map use-pppoe
  bridge-group 1
  no shutdown

interface GigaEthernet0.1
  encapsulation pppoe
  auto-connect
  ppp binding ISP1
  ip address ipcp
  ip tcp adjust-mss auto
  ip napt enable
  ip napt hairpinning
  ip napt translation tcp-timeout 3600
  ip napt translation udp-timeout 1800
  ip napt translation dns-timeout 30
  ip ufs-cache timeout tcp 60
  ip ufs-cache timeout udp 60
  no shutdown
```

## IPv4 over IPv6 トンネル (transix, DS-Lite)の設定
インターネットマルチフィードとトンネルを張ってIPv4 over IPv6で通信をします。
宛先アドレスはインターネットマルチフィードのアドレスを指定します。
NTT東日本とNTT西日本でアドレスが違うので、ターミナルで`nslookup gw.transix.jp`と入力して出てきたIPv6アドレスを入力するか、FQDNで`gw.transix.jp`と指定します。
```
interface Tunnel0.0
  tunnel mode 4-over-6
  tunnel destination 2404:8e01::feed:102
  ip unnumbered GigaEthernet1.0
  ip tcp adjust-mss auto
  ip ufs-cache timeout tcp 60
  ip ufs-cache timeout udp 60
  no shutdown
```