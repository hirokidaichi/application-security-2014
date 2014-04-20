


Webシステムの脆弱性とセキュリティ
====


この研修資料は、新人技術者とWebディレクタなどの総合職を対象としたものです。理解のための単純化している部分があります。適宜、出典先などを確認し理解を深めてください。


## 副読本 

* [安全なウェブサイトの作り方](http://www.ipa.go.jp/security/vuln/websecurity.html)
* [体系的に学ぶ 安全なWebアプリケーションの作り方](http://www.amazon.co.jp/dp/4797361190)

## 課題について
課題は、完全に技術者向けは[エ]と記載。それ以外は、非エンジニアでもトライしてみよう。
また、[高]と記載されている問題については、それ以外を完了してから余裕があれば挑戦してみよう。

## 事前準備
Google Chromeをインストールしておこう。
後述するXSSフィルターという機能がデフォルトでついているので、
それをつけない形で起動する。

http://meme.efcl.info/2011/05/google-chromexss.html

macの人はterminalで以下：
```open -a Google\ Chrome --args --disable-xss-auditor```

# ディレクタ/プランナが把握してほしいこと
どんな種類の攻撃があって、「なにができてしまうか」を把握すること。そして、悪意あるユーザーに利用されやすい「仕様」とはどういうものかを理解すること。

```
エンジニア「XSSが可能な脆弱性が見つかりました」
あなた「CookieにHTTP Onlyはつけていましたか？」
エンジニア「ほう・・」
```

# エンジニアが把握してほしいこと
よく知られた攻撃について適切な対策を知ること。多くのフレームワークでそれを防ぐためにどのようにしているかを理解できるようになること。脆弱なシステムを作らないための原理原則を知ること。


# 脆弱性って？

システムやそれに関わるひとが持っている弱点。その弱点を攻撃することで「本来できないはずのこと」がシステムのバグや欠陥でできてしまうこと。

## 例：
http://d.hatena.ne.jp/keyword/%A4%DC%A4%AF%A4%CF%A4%DE%A4%C1%A4%C1%A4%E3%A4%F3%A1%AA


# どんなところに脆弱性が潜むのか

人間の弱点(脆弱性）と同じで、異なるパーツをつないでいるところがもろいことが多い。たとえば、Webアプリケーションとデータベースとか、Webアプリケーションとブラウザとか。メールサービスとWebアプリケーションとか。

伝言ゲームをイメージするといいかもしれない。お互いの認識がずれていては、別の人に想定しない行動をとらせてしまうのと似ている。


# 基本的なWebアプリケーションの構成

基本的なWebシステムの構成要素としては

* ブラウザ(ネイティブアプリ)
* ユーザ
* ロードバランサ
* アプリケーションサーバ
	- アプリケーションサーバの動くOS
* データベースサーバ
* キャッシュサーバ
* メール(Push)配信サーバ

![hoge.png](https://qiita-image-store.s3.amazonaws.com/0/35671/de230bb8-71ba-4192-5a2d-1083c2b9ca77.png "hoge.png")


etc,

などがくみ合わさってできている。
それぞれをつなぐ部分で、Aという構成要素が想定しているやり取りの方法とBという構成要素が想定しているやり取りの方法の不一致が、脆弱性を生んでしまう。

# 意味を持っている文字列
システムとシステムを橋渡しするためにコンピュータが解釈するための規則を持った文字列をやり取りする。それは言語と呼ばれたりプロトコルと呼ばれたりする。


## ファイル名/シェル
``C:¥Program Files¥hoge¥fuga.png ``
``/usr/local/bin``
ファイルを階層的に管理するためのパスには、``/``や``:``のような区切りをもっている。この区切りはOSが解釈して、どこにデータがあるのかを探す。

## HTML

```
<div id="1"><script>/*javascript*/</script></div>
```
HTMLはタグという``<XXX>中身</XXX>``で囲まれた階層構造を持っていて、``<div id="2">``のように開きタグに属性(attribute)を持つ構造になっている。
## HTTP

```
telnet yahoo.co.jp 80
GET / HTTP/1.1

HTTP/1.1 302 Moved Temporarily
Date: Sun, 20 Apr 2014 06:30:50 GMT
Location: http://www.yahoo.co.jp
Vary: Accept-Encoding
Content-Type: text/html; charset=utf-8
Cache-Control: private
Transfer-Encoding: chunked

0
```

## Cookie / Session情報
http://ja.wikipedia.org/wiki/HTTP_cookie

```
Set-Cookie: PREF=ID=b5f8dc2e4f1032fa:FF=0:TM=1296885240:LM=1296885240:S=zh1x1wrDNqySDQ8P; expires=Mon, 04-Feb-2013 05:54:00 GMT; path=/; domain=.google.co.jp
Set-Cookie: NID=43=q425g0gazPJFqGsQsxiWpoY5kC_244zg0CnJV4t_vsbE3O7x-4GXcOLzXqtmKc_MfQu4JsM0viGAI5Y6QxgkzmBvXEhNBS_nB_MBhqdAWrTTSKhMmwJ78ppclNblivPz; expires=Sun, 07-Aug-2011 05:54:00 GMT; path=/; domain=.google.co.jp; HttpOnly

```
HTTPプロトコルで、パスワードなどの情報を送るとSet-Cookieというメッセージが帰ってくる（ように作る）。一度セットされると自分で消したり、期限が切れるまでそのドメインには送り続ける。広告のターゲティングなどにも同じくクッキーが用いられている。

Session用のCookieは、たいていの場合一定時間で失効する。同じブラウザでも１週間や１月で再度ログイン画面を要求されたりするのは、悪意あるひとがずっとなりすませないようにするため。

では、なりすましができてしまった場合でも

* パスワードを変更する
* クレジットカード番号を盗む
* ポイントや物品を購入する

のは難しいようにたいていのサービスではなっている。
（そのように作らないと行けない。）
これはどういう仕様によるものか、見てみよう。



## SQL

http://ja.wikipedia.org/wiki/SQL

## iOS Custom URLSheme / Android intent
スマートフォンアプリのOSには、他のアプリとコミュニケーションを行うための機能が提供されている場合が多い。
それらを収集したり、提供することでどのような問題がおこるか。
たとえば、lineボタンでクリックするとタイムラインに自動ポストするURLがあったらどうなるか。

## 壊れた入力をどのように扱うか？
悪意あるユーザーは、簡単にブラウザ上で展開されたアプリケーションの中身を書き換えることができます。多少難しさに違いがあってもAndroidアプリ、iPhoneアプリも同様です。

なので、サーバーに対して「壊れた入力」をすることができます。たとえば、twitterでは150文字制限されていますが、10000文字送ることもできます。その場合、サーバー側で150字までかどうか検査して、それを超えた場合エラーを返すように作られています。

このように「壊れた入力」であるかどうかを最初に検査することをvalidationといいます。

## 悪意ある入力をどのようにあつかうか？
しっかりとしたvalidationをしていても、悪意ある入力による被害を防ぐことは難しいです。悪意あるユーザはWebアプリではなくて、Webアプリとつながっている別のシステムに対して操作をしようとします。たとえば、他の人のブラウザ上で、データベースサーバで、アプリケーションの動くOSで。

なので、セキュリティのためには、他のシステムに対してユーザの入力を反映させる出口のところで、それぞれのシステムに対して無害なように書き換えを行います。これをescpape処理と言います。

![hoge.png](https://qiita-image-store.s3.amazonaws.com/0/35671/58d00b35-4fbf-6526-f237-3250d1748e4a.png "hoge.png")




# Webアプリケーションとブラウザ
ブラウザはHTMLとJavaScriptを解釈して、ユーザにインターフェースを提供する。
## 文字参照について
> 文字参照（もじさんしょう、英: character reference）とはHTMLなどのSGML文書においては、直接記述できない文字や記号（マークアップで使われる、半角の不等号「<」や「>」など）を表記する際に用いられる方法である。SGML構成素のひとつとして定義されており、文書文字集合中の文字を参照する為の手段を提供する。HTMLにおける文字参照には、表記方法により数値文字参照[1]と文字実体参照[2]の二種が存在する。XMLにおいては、HTMLにおける「数値文字参照」を「文字参照」と呼ぶ。なおHTMLにおける「文字実体参照」は、XMLでは実体参照[3]と呼び区別する。

http://ja.wikipedia.org/wiki/文字参照


```参照の例

 	&nbsp;	ノーブレークスペース - 折り返しを起こさない（ホワイトスペースではない）空白
<	&lt;	小なり記号（半角）
>	&gt;	大なり記号（半角）
&	&amp;	アンパサンド（半角）
"	&quot;	二重引用符（半角）
```

## JavaScriptについて
JavaScriptはブラウザ上で動く、プログラム言語です。たとえば、HTML中にこのように埋め込まれたときにも動作します。

```
<div onmouseover="alert('hello')"></div>
<script>alert('hello')</script>
```

## XSS
### 概要

> ウェブアプリケーションの中には、検索のキーワードの表示画面や個人情報登録時の確認画面、掲示板、ウェブのログ統計画面等、利用者からの入力内容や HTTP ヘッダの情報を処理し、ウェブページとし て出力するものがあります。ここで、ウェブページへの出力処理に問題がある場合、そのウェブページに スクリプト等を埋め込まれてしまいます。この問題を「クロスサイト・スクリプティングの脆弱性」と呼び、こ の問題を悪用した攻撃手法を、「クロスサイト・スクリプティング攻撃」と呼びます。クロスサイト・スクリプ ティング攻撃の影響は、ウェブサイト自体に対してではなく、そのウェブサイトのページを閲覧している利用者に及びます。
(安全なウェブサイトの作り方 改訂第6版 p.22)


![cdraw.png](https://qiita-image-store.s3.amazonaws.com/0/35671/4523b1d8-8c7d-1ddc-3e7a-5b60f1c77471.png "cdraw.png")

### Same Origin Policy

http://d.hatena.ne.jp/hasegawayosuke/20130330/p1
簡単に言うと、アクセスできるリソースは基本的には同じサイトのものだけにしようというもの。同じサイトかどうかはURLの
``http://`` ``domain.org`` ``:8080``のようなサーバーを指定する部分で決まる。
### CORS

* クロスサイト方式での XMLHttpRequest API 実行
* Web フォント 
* WebGL テクスチャ
* drawImage を用いて Canvas に描かれた画像

などのリソースをクロスサイトに利用するための仕様。

https://developer.mozilla.org/ja/docs/HTTP_access_control

### XSS体験
http://xss-quiz.int21h.jp/

### 対策
1. ユーザーからの入力を適切な形で変換する。
1. cookieにHTTP only属性をつける（xssがおこらない訳ではない）

### クライアント側でのXSS
* URLのhashを$(location.hash)して、エレメントを作成した。問題は何か?
* Intentに含まれる内容をWebView上にHTMLとして展開した。何が問題か？ 


### 課題
1. xssクイズのステージ２をやってみよう。
1. [エ]xssクイズのステージ３をやってみよう。
1. [エ]Text::Xslateというtemplateライブラリでは自動escape機能があるが、どのようにescapeされているか調べてみよう。
1. [エ]&lt;script&gt;タグの中に入力文字列を含める場合どのようにescapeするべきか。
1. あとで[エ]utf7を入力されて引き起こされるxssを調べてみよう。
1. あとで[エ]ブラウザのxssフィルター実装状況について調べよう。
1. あとで[エ]xssクイズのステージ４をやってみよう。
1. あとで[エ]xssクイズのステージ５をやってみよう。
1. あとで[高][エ][XSS Filter Ecation Cheat Sheet](https://www.owasp.org/index.php/XSS_Filter_Evasion_Cheat_Sheet)を読んで、いくつかの挙動を確かめてみよう。

## CSRF
### 概要
CSRF(クロスサイト・リクエスト・フォージェリ) とは：


> ウェブサイトの中には、サービスの提供に際しログイン機能を設けているものがあります。ここで、ログインした利用者からのリクエストについて、その利用者が意図したリクエストであるかどうかを識別する仕 組みを持たないウェブサイトは、外部サイトを経由した悪意のあるリクエストを受け入れてしまう場合があ ります。このようなウェブサイトにログインした利用者は、悪意のある人が用意した罠により、利用者が予 期しない処理を実行させられてしまう可能性があります。このような問題を「CSRF(Cross-Site Request Forgeries/クロスサイト・リクエスト・フォージェリ)の脆弱性」と呼び、これを悪用した攻撃を、「CSRF 攻 撃」と呼びます。
(安全なウェブサイトの作り方 改訂第6版 p.29)

http://www.ipa.go.jp/security/vuln/vuln_contents/csrf.html

### 対策
* セッションIDやユーザIDとソースコード中のsalt値から導かれるハッシュ値 や
* ページごとに生成される、毎回異なる値（ページトークン）

などをページ中に含み、副作用のあるリクエストのときに同時に送るようにして、その中身を検証する。


### 課題
1. 情報を見るだけでなく、記録するような仕様をひとつ考えてみよう。
1. POST KEYをメンバーのIDにした。何が問題か？
1. ソースコードが漏洩した。post keyの生成の仕方をどのように変更するか？
1. なにか一つサイトを調べ、post keyやtokenなどがリクエストと同時に渡されているか確かめてみよう。
1. [エ]クロスドメインにgetをさせる方法を３つ考えよう
1. [エ]クロスドメインにpostをさせる方法を考えよう
1. [エ]Rails,CakePHPなどでpostkeyをどのように作成しているか調べよう

## その他
### 課題
1. [エ]JSONハイジャッキングについて調べて概要と対策をまとめよう
1. [エ]HTTPヘッダインジェクションについて調べて概要と対策をまとめよう
1. [エ]メールヘッダインジェクションについて調べて概要と対策をまとめよう


# Webアプリケーション/ネイティブアプリとOS
## コマンドインジェクション
### 概要

> 閲覧者からのデータの入力や操作を受け付けるようなWebサイトで、プログラムに与えるパラメータにOSに対する命令文を紛れ込ませて不正に操作する攻撃。また、そのような攻撃を許してしまうプログラムの脆弱性のこと。
「インジェクション」(injection)は「注入」の意味で、Webアプリケーションのプログラムに与えるパラメータの一部に、そのWebサーバで動作しているOSのコマンドとして解釈できる文字列を紛れ込ませ、遠隔からサーバを乗っ取って不正に操作する攻撃である。

http://e-words.jp/w/OSE382B3E3839EE383B3E38389E382A4E383B3E382B8E382A7E382AFE382B7E383A7E383B3.html

できてしまうと、ソースコードが盗めてしまう。
DBの中身も盗めてしまう。もしかしたら、sshの鍵も盗めてしまう。

### 対策

* そもそもOSコマンド（shellが起動する）を発行するプログラムを書かない。（そういうAPIを使わない）
* OSコマンドを発行しても、ユーザー入力を含めない。
* どうしても含める場合は適切にエスケープする枯れたライブラリを採用する。

### 課題
1. [エ]　perl/ruby（どちらか得意な方）でファイル開く際にユーザ入力を適切に処理する方法を調べる。
1. [エ]　Android/iOS（どちらか得意な方）でファイル開く際にユーザー入力を適切に処理する方法を調べる。また、適切に処理しなかったとき、どのような場合に問題になるか。


## ディレクトリトラバーサル
### 概要

> ディレクトリトラバーサル (英語: directory traversal) とは、利用者が供給した入力ファイル名のセキュリティ検証/無害化が不十分であるため、ファイルAPIに対して「親ディレクトリへの横断 (traverse)」を示すような文字がすり抜けて渡されてしまうような攻略方法のことである。

http://ja.wikipedia.org/wiki/ディレクトリトラバーサル

ソースコードが盗めてしまうかもしれない。
場合によっては、サーバーにログインするための情報が盗まれてしまうかもしれない。

### 対策
* ファイルを開く際に、ユーザー入力を含めない。
* どうしても含める場合は適切にエスケープする枯れたライブラリを採用するか、決まった入力のみ許可するようにする。

### 課題

# WebアプリケーションとWebアプリケーション
### 課題
1. [エ]urlにmode=hogeという入力がある場合にrubyのmethod missing(またはperlのAUTOLOAD)を使い、処理を切り分けるアプリケーションを作成した。どのような点に注意しないと行けないか。snippetを書いてみよう。
1. [エ]http://codepad.org のようなサイトを作成しようと考えた。どのように実装するべきか。
1. [エ]ユーザーからの入力を正規表現として受け入れて、検索するサービスを考えた。どのように作るか。


# WebアプリケーションとDBサーバー
## SQL Injection
### 概要
http://www.ipa.go.jp/security/vuln/vuln_contents/sql.html
http://ja.wikipedia.org/wiki/SQLインジェクション
### 対策
* ユーザー入力を用いてSQLを作成しない
* 作成する場合は、プレースホルダーを使う
* プレースホルダーを使わない場合でもライブラリが提供するquote関数を通してescapeする。

http://www.ipa.go.jp/files/000017320.pdf


### 課題
1. [エ]/^¥d+$/という正規表現をrubyで書き、入力検査をした。ユーザの入力は数値しか存在しないはずなので、"SELECT * FROM table WHERE id=$id"というSQLを発行した。何が問題か。

# Webアプリケーションとその他システム
### 課題 
1. memcached injectionについて調べ、どのような対策がされているかまとめなさい
1. メールヘッダインジェクションについて調べ、どのような対策がなされているかまとめなさい


# あなたをだます脆弱性
## フィッシング
### 概要
http://ja.wikipedia.org/wiki/フィッシング_(詐欺)

![スクリーンショット 2014-04-20 20.01.00.png](https://qiita-image-store.s3.amazonaws.com/0/35671/f86e0327-a149-7b9e-e274-3038a3c248de.png "スクリーンショット 2014-04-20 20.01.00.png")

http://www.bk.mufg.jp/info/phishing/20131118.html

手段は、メールでも手紙でも電話でもLINEでも。
http://did2memo.net/2013/06/18/naver-line-phishing-mail-sample/

http://did2memo.net/2014/04/01/naver-line-stamp-present-form/

### 対策
* ユーザーとしては、ブラウザのURL欄を確認し、httpsになっているか、ちゃんと発行元が今アクセスしているサービスの会社かを確認する
* サービスとしてはパスワード情報を表示させる際にURLが確認できるようにする。

### 課題
1.  mixiやfacebookの情報を使うアプリで、認証フェーズがどのようになっているかいくつか確認し、分類してみよう。
1. ソーシャルブックマークサイトを作成した。
ブックマークレットを使うとiframeを展開してUIを表示する仕様にした。このとき、ログインしていない場合はログインフォームを表示したのだが、何が問題か？
1. [エ]ソーシャルブックマークサイトを作成した。location.href、window.titleとコメントを送って、ユーザーごとに記録したのだが、あとからtitleやサマリーはサーバーサイドからリクエストを送って作成するように仕様変更した。どんな問題に対処するためか。




## クリックジャッキング
### 概要

>クリックジャッキングとは、外見上は無害に見えるウェブページをクリックしている間にウェブ利用者をだまして秘密情報を露呈させる、あるいはウェブ利用者のコンピュータの支配を獲得する悪意の技術である。クリックジャック攻撃などとも呼ばれる。各種のウェブブラウザとプラットホームに渡る脆弱さを利用するクリックジャッキングは、例えば他の機能を実行する為のボタンをクリックするように埋め込まれたコードあるいはスクリプトの形態をしているため、利用者に気付かれることなく実行され得る

http://thehackernews.com/2013/07/hacking-linkedin-account-vulnerability_13.html

### 対策

* X-FRAME-OPTIONをつける
* frame killer/bustingのコードを入れる

```
<style> html{display:none;} </style>
<script>
   if(self == top) {
       document.documentElement.style.display = 'block'; 
   } else {
       top.location = self.location; 
   }
</script>
```

### 課題
1. Click Jackingの対策がむずかしい仕様はどういうものか。


## オープンリダイレクタ
### 概要
*意図せず*に任意のサイトへのリダイレクト機能を提供してしまう脆弱性。
たとえば、信頼しているサイト```http://trust.com```がある。ユーザーはこのドメインのサイトであれば、安全であろうと思いclickをする。あるいは、trust.comのパスワードを求められてもURLの確認をしないで、入力してしまうかもしれない。

このとき、trust.comに任意のサイトへのリダイレクト機能を提供してしまった場合、```http://trust.com/redirect/evil.com```のようなURLによって、ユーザーには安全なURLだと錯覚させながら、悪意あるサイトへの誘導をすることができてしまう。

twitter.comではt.coという短縮URLを提供しているが、WebUI上では、中身を展開して表示している。

広告サービスや短縮URLサービスのようにもとからOpen Redirectorを提供しているサービスであれば、もとより錯覚させてしまう恐れがないので、その限りではない。場合によっては危険なサイトのDBを持っており、遷移させないようにするなどの機能を持っていることもある。

### 対策
* オープンリダイレクタがつく場合、錯覚させないようなドメインでサービスに展開する
* 限られたドメインのみに遷移できるようにする。
* Locationヘッダやmetaタグ内にユーザー入力が入らないようにする。
* 入る場合には不適切な入力をescapeする




#暗号技術
## 共通鍵暗号
http://ja.wikipedia.org/wiki/共通鍵暗号
お互いに共通の鍵を持っていることで暗号をかけたり、元に戻したりすることができる暗号。

## 公開鍵暗号
http://ja.wikipedia.org/wiki/公開鍵暗号

鍵のペアA,Bがあって、Aでかけた鍵はBでしか開けない。
Bで掛けた鍵はAでしか解けないという風にしかけがしてある。

どちらかを誰にでも見えるところにおいて、それを公開鍵といい、
自分だけがもっているものを秘密鍵という。

たとえば、
Aliceの公開鍵を使ってBobが暗号化をしたら、Aliceの秘密鍵でしか読めないため、二人しか知らない秘密を作ることができる。

Aliceが自分の公開鍵で合い言葉を暗号化した場合、
Bobや他の人はAliceが合い言葉を送ったのはAliceだとわかる。

公開鍵：(e=3,n=22)
秘密鍵：(d=7,n=22)
のとき、
暗号：E = M^e mod n
復号：M = E^d mod n
という風に変換できる。Excelで1~21までのメッセージを暗号化、復号化してみよう。

## 一方向関数
http://ja.wikipedia.org/wiki/暗号学的ハッシュ関数

### Study:SDカード
AndroidアプリでSDカードにユーザーの個人情報をデータとして保存したい。どんな脅威があり、どのように保存するのが安全だろうか。

### Study:パスワード保存
Webサービスのパスワード検証する仕組みを作りたい。
パスワードを生で保存したら危険だという話を聞いたことがある。
どのような脅威に対して、どのように保存するのがよいか。




### 課題[エ]
次のサイトを見て、様々な脆弱性を試しながら、実行してみよう。
http://works.surgo.jp/translation/gruyere/


### 課題[エ]
以下はとある換字式暗号で暗号化された文章です。
これを解読しみよう。

>
FQNW R FJB HXDWP, CQNAN FJB JW JVJIRWP YDKURLJCRXW LJUUNM CQN FQXUN NJACQ LJCJUXP, FQRLQ FJB XWN XO CQN KRKUNB XO VH PNWNAJCRXW. RC FJB LANJCNM KH J ONUUXF WJVNM BCNFJAC KAJWM WXC OJA OAXV QNAN RW VNWUX YJAT, JWM QN KAXDPQC RC CX URON FRCQ QRB YXNCRL CXDLQ. CQRB FJB RW CQN UJCN 1960'B, KNOXAN YNABXWJU LXVYDCNAB JWM MNBTCXY YDKURBQRWP, BX RC FJB JUU VJMN FRCQ CHYNFARCNAB, BLRBBXAB, JWM YXUJAXRM LJVNAJB. RC FJB BXAC XO URTN PXXPUN RW YJYNAKJLT OXAV, 35 HNJAB KNOXAN PXXPUN LJVN JUXWP: RC FJB RMNJURBCRL, JWM XENAOUXFRWP FRCQ WNJC CXXUB JWM PANJC WXCRXWB.
BCNFJAC JWM QRB CNJV YDC XDC BNENAJU RBBDNB XO CQN FQXUN NJACQ LJCJUXP, JWM CQNW FQNW RC QJM ADW RCB LXDABN, CQNH YDC XDC J ORWJU RBBDN. RC FJB CQN VRM-1970B, JWM R FJB HXDA JPN. XW CQN KJLT LXENA XO CQNRA ORWJU RBBDN FJB J YQXCXPAJYQ XO JW NJAUH VXAWRWP LXDWCAH AXJM, CQN TRWM HXD VRPQC ORWM HXDABNUO QRCLQQRTRWP XW RO HXD FNAN BX JMENWCDAXDB. KNWNJCQ RC FNAN CQN FXAMB: "BCJH QDWPAH. BCJH OXXURBQ." RC FJB CQNRA OJANFNUU VNBBJPN JB CQNH BRPWNM XOO. BCJH QDWPAH. BCJH OXXURBQ. JWM R QJEN JUFJHB FRBQNM CQJC OXA VHBNUO. JWM WXF, JB HXD PAJMDJCN CX KNPRW JWNF, R FRBQ CQJC OXA HXD.
BCJH QDWPAH. BCJH OXXURBQ.


