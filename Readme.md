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
```
ip ufs-cache max-entries 50000
ip ufs-cache enable
```

## デフォルトルートの設定
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
```
bridge irb enable
no bridge 1 bridge ip
```

## DNSサーバーの設定
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
```
ssh-server ip enable
```

## Web-GUIの有効化
```
http-server authentication-method digest
http-server username admin secret-password xxxxxxxxxxxxxxxxx
http-server ip enable
```

## NetMeisterの設定
```
nm ip enable
nm account toyohashi password secret xxxxxxxxxxxxxxxxx
nm sitename toyohashi
nm ddns hostname ix2215
nm ddns notify interface GigaEthernet0.1 protocol ip
nm ddns notify interface GigaEthernet0.0 protocol ipv6
```

## PPPoEへのルーティングの設定
```
route-map use-pppoe permit 10
  match ip address access-list server_services
  set interface GigaEthernet0.1
```

## PPP認証の設定
```
ppp profile ISP1
  authentication myname hogehoge.hogehogeppp.net
  authentication password hogehoge@hoge.hogehogeppp.net hogehoge
```

## IPv4 DHCPプロファイルの作成
```
ip dhcp profile private
  assignable-range 172.16.1.1 172.168.1.254
  default-gateway 172.16.0.1
  dns-server 172.16.0.1
  domain-name TYH-Network
  lease-time 3600
```

## IPv6 DHCPプロファイルの作成
```
ipv6 dhcp client-profile dhcpv6-cl
  information-request
  option-request dns-servers

ipv6 dhcp server-profile dhcp6-sv
  dns-server dhcp
```

## QoSの設定
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