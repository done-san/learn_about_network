# ドメインとDNSの仕組み

## 目次
1. ドメインとは
2. DNS(Domain Name System)について

### 1. ドメインとは
「IPアドレス」と対応付ける「名前(文字列)」のこと。  
この対応付けのことを「名前解決(後述)」という。  
  
ちなみに、英語では「domain」であり、日本語では「領地」や「領域」のこと。  
世界中のドメインは、ドメイン名前空間という階層構造を構成している。  
後述するがドメイン名前空間は、DNSという仕組みによって成り立っている。  
  
```
ex) 
ドメイン　: www.example.com

user@ubuntu:~$ nslookup
> www.example.com
Server:         127.0.0.53
Address:        127.0.0.53#53

Non-authoritative answer:
Name:   www.example.com
Address: 93.184.216.34 
Name:   www.example.com
Address: 2606:2800:220:1:248:1893:25c8:1946
>

結果:
DNSサーバ　　: 127.0.0.53
IPv4 Address: 93.184.216.34
IPv6 Address: 2606:2800:220:1:248:1893:25c8:1946
```  
  
#### Tips1 IPアドレスとは
コンピュータネットワークにおいて、ホストを識別する住所のようなもの。  
グローバルIPアドレスの場合、インターネット上で、一意な値を持つ識別子といえる。  
  
#### Tips2 ドメインの構造
ex) www.example.com.  
``` . ```       : ルートドメイン。ドメインの階層構造で頂点。  
``` .com ```     : トップレベルドメイン(TLD)。特に、 ``` .net ```や``` .org ```などを汎用トップレベルドメイン(gTLD) という。  
``` .example ``` : セカンドレベルドメイン。この場合、 ``` .com ```の配下にあるドメインのひとつ。  
``` www ```     : ホストを表す。このばあい、HTTPサーバを指す。  

#### Tips3 DNSロードバランシング  
1つのドメインに対して、複数(2つ以上)のIPアドレスを紐づける。  
これ(1:Nの対応付け)によって、ユーザからのリクエストを分散させることができる。  
DNSによる負荷分散の仕組みを「DNSロードバランシング」という。  

#### Tips4 中古ドメインの利用  
SEO対策の一環として、すでに他の人に利用されたドメインを買い取ることがある。  
すでにGoogleのサーチエンジンにおいて、評価付けがなされている(ドメインパワーがある)ので上位表示がされやすくなる、などのメリットがある。  

ドメイン価値チェックツール  
https://www.ispr.net/domain-value/result  

#### Tips5 ドメインの種類
``` .jp (日本) ```や``` .kr (韓国) ```などの国ごとに割り当てられるドメインをccTLD(国コードトップレベルドメイン)という。  
``` .ac.jp ```や``` co.jp ```などJPRS(日本レジストリサービス)に登録・管理されているJPドメインで、特定の組織・機関を表すドメインを属性型JPドメインという。  
その他にも、``` ○○○.tokyo.jp (○○○は任意のドメイン)```など、都道府県型JPドメインなどがある。  

### 2. DNS(Domain Name System)について
ドメイン名とIPアドレスを対応付けるためのネットワークの仕組み。

登場人物
1. DNSクライアント
2. DNSキャッシュサーバ
3. DNSコンテンツサーバ(ルートドメインサーバ、.com ネームサーバ、.example.com ネームサーバ)

``` 
ex) www.example.comにhttps接続する
URL　　　　       : https://www.example.com   
IPアドレス(+Port) : 2606:2800:220:1:248:1893:25c8:1946:443  

【 DNS通信(名前解決)のイメージ 】
1. Webブラウザ「www.example.comってどこやねん。ほな、DNSキャッシュサーバに聞こう！」

2. Webブラウザ「キャッシュサーバくん、www.example.comってどのIPアドレスを持ってるの～？」
   DNSキャッシュサーバ「うーん、ぼくもわからんわ～。ほな、ルートサーバさんに聞くか～。」
   
3. DNSキャッシュサーバ「ルートドメインサーバさん、www.example.comのIPアドレス知ってますか～？」
   ルートドメインサーバ「.com ネームサーバさんなら知ってるとおもうよ。かれのIPアドレスはxxx.xxx.xxx.xxx(*1)だよ！」

4. DNSキャッシュサーバ「.comネームサーバさん、www.example.comのIPアドレス知ってますか～？」
   .comネームサーバ「.example.com ネームサーバさんなら知ってるとおもうよ。かれのIPアドレスはyyy.yyy.yyy.yyy(*1)だよ！」

5. DNSキャッシュサーバ「.example.comネームサーバさん、www.example.comのIPアドレス知ってますか～？」
   .example.comネームサーバ「ああ、よくきたね！かれのIPならぼくが管理してるよ！かれのIPはzzz.zzz.zzz.zzz(*1)だよ！」

6. DNSキャッシュサーバ「ただいまー！なんかwww.example.comのIPアドレスはzzz.zzz.zzz.zzz(*1)らしいよ！」
   Webブラウザ「おかえりー！ありがとう！！ほんじゃ、https:zzz.zzz.zzz.zzz(*1)でリクエストしてみるよ！」

7. DNSキャッシュサーバ「TTL86400なので、1日間キャッシュしておかなくちゃね！またリクエストしてみてね～～！」

おわり

*1 : xxx.xxx.xxx.xxx, yyy.yyy.yyy.yyy, zzz.zzz.zzz.zzz はそれぞれ、任意のグローバルIPアドレスです。
```  

#### Tips6 DNSというプロトコル
DNSはUDP/53のポート番号を利用する。  
通信に利用するデータ量を自体が少なく、信頼性を担保するまでもないため、UDPを使用している。  
ただし、DNS(UDP)のセキュリティリスクを回避する必要があり、新しくDNS(TCP)の利用(= DNS SEC)が求められている。  
  
RFC9210 DNS Transport over TCP - Operational Requirements  
https://www.rfc-editor.org/rfc/rfc9210.html

#### Tips7 DNSにおけるセキュリティリスク
1. DNSキャッシュポイズニング攻撃 ： DNSキャッシュサーバにおけるDNSレコード(IPアドレスとドメイン名の紐づけ)を偽装(汚染)することで、異なったサーバに通信させる攻撃。UDPは双方向通信ではなく、一方的に送信するだけなのでキャッシュされる前に偽装したIPを仕込むことで成立する。  
2. DNSリクレクション攻撃 ： 送信元のIPアドレスを攻撃対象のクライアントのIPアドレスに設定し、何千台ものDNSサーバにクエリ(問い合わせ)をかけることでDoS攻撃を成立させる。  
  
#### Tips8 DNSレコード
DNSではBINDのようなソフトウェアを利用することになる。  
このとき、/etc/named.conf という設定ファイルをいじることになるが、そのDNSレコードには以下のようなものがある。  

Aレコード     : IPv4アドレス  
AAAAレコード  : IPv6アドレス  
CNAMEレコード : 別のドメイン  
MXレコード    : メールサーバ  
NSレコード    : DNSサーバ  
...etc  
