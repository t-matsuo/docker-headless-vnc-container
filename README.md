[English](./README.en.md)

# Webブラウザ経由でLinuxデスクトップ環境が使える日本語向けコンテナイメージ

Linuxのデスクトップ環境(xfceを使用)をコンテナで起動でき、Webブラウザ経由でアクセスすることができます。
日本語向けにカスタマイズされており、デスクトップ環境はデフォルト言語が日本語に設定されています。
また、日本語入力変換も使えるようになっています。(Shift + Spaceで日本語入力ON)

デフォルトでSSLが有効になっており、PAMベースのBasic認証がかかっています。
Linuxデスクトップ以外に、ターミナルエミュレータ、ファイルブラウザアプリも搭載しています。
使っている主なコンポーネントは以下の通りです。

* [**Xfce4**] (http://www.xfce.org) - Linuxデスクトップマネージャ。
* [**noVNC**](https://github.com/novnc/noVNC) - HTML5 VNCクライアント。上記Linuxデスクトップ(xfce)へのアクセス用です。
   * http(s)://IP:Port/desktop/ というパスでアクセスできます。(IP:Portはコンテナの待ち受けIPとPort)
* [**butterfly**](https://github.com/paradoxxxzero/butterfly) - ブラウザ経由で使えるターミナルエミュレータ。
   * http(s)://IP:Port/term/ というパスでアクセスできます。(IP:Portはコンテナの待ち受けIPとPort)
* [**filebrowser**](https://github.com/filebrowser/filebrowser) - ファイルブラウザ。手元の端末からファイルをアップロード・ダウンロード可能。
   * http(s)://IP:Port/file/ というパスでアクセスできます。(IP:Portはコンテナの待ち受けIPとPort)
* [**Nginx**](https://github.com/nginx/nginx) - Webサーバ
   * 上記アプリへのリバースプロキシとして、デフォルトは8080ポートで待ち受けます。
* [**Supervisord**](http://supervisord.org/) - プロセス制御システム
   * 上記のコンポーネントは、supervisord を使って起動されます。

* 対応ブラウザ
  * Firefox
  * Chromium
  * Edge

* その他特徴
  * 通信経路にプロキシがいるような環境で、無通信のセッションが強制的に切断されないように、Linuxデスクトップおよびターミナルエミュレータは定期的にブラウザと通信を発生させます。
  * Linuxデスクトップおよびターミナルエミュレータは、ブラウザ閉じる場合に警告を出します。これによって、誤ってブラウザが閉じられるのを防ぐことができます。
  
画面例

Linuxデスクトップ

![Docker Linuxデスクトップ](.pics/screen-desktop.png)

ターミナルエミュレータ

![Docker ターミナルエミュレータ](.pics/screen-term.png)

## 使い方 (Docker)

- Docker (SSL有効)

デフォルトでは、オレオレ証明書を使ってサーバが起動します。--shm-size を指定しないと、firefoxやchromeがクラッシュしますのでご注意ください。

      docker run -d -p 8080:8080 -e PASSWORD=password --name centos-xfce-ja --shm-size=2g tmatsuo/centos-xfce-ja

- Docker (自身で用意した証明を使いたい場合)

自身で用意した証明書を使いたい場合は、/etc/pki/nginx/server.key と /etc/pki/nginx/server.crt にマウントしてください。

      docker run -d -p 8080:8080 -e PASSWORD=password -v /path/to/server.key:/etc/pki/nginx/server.key -v /path/to/server.crt:/etc/pki/nginx/server.crt  --name centos-xfce-ja --shm-size=2g tmatsuo/centos-xfce-ja

- Docker (SSL無効)

NOSSL環境変数にtrueに設定してください。

      docker run -d -p 8080:8080 -e PASSWORD=password -e NOSSL=true --name centos-xfce-ja --shm-size=2g tmatsuo/centos-xfce-ja

- アクセス方法

SSL有効・無効にかかわらず、デフォルトのポートは8080です。

* デスクトップへのアクセスは http(s)://IP:8080/desktop/ です。
* ターミナルエミュレータへのアクセスは http(s)://IP:8080/term/ です。
* ファイルブラウザへのアクセスは http(s)://IP:8080/file/ です。

Basic認証、およびターミナルエミュレータのログインユーザは root です。
パスワードは、コンテナ起動時に指定したパスワードです。ログイン後にpasswdコマンドで変更可能です。

## 使い方 (Kubernetes)

* [こちらのREADMEを参考](./kubernetes/README.md)

## デスクトップ環境(VNC)について

コンテナ起動時に以下の環境変数を指定することができます。

* `VNC_COL_DEPTH`, デフォルト: `24`
  * 色深度を指定できます。
* `VNC_RESOLUTION`, デフォルト: `1800x850`
  * デスクトップの解像度を指定できます。横解像度x縦解像度 というフォーマットで記載してください。
  * コンテナ起動後に解像度を変更したい場合は、Linuxデスクトップの設定で変更することができますが、コンテナを再起動すると元に戻ります。設定を永続化したい場合は、 /root/.bashrc ファイルに、VNC_RESOLUTION 変数を設定してください。デスクトップ環境だけを色道したい場合は、`supervisorctl restart vnc` コマンドを実行してください。

## 待ち受けポートの変更

* `PORT`, デフォルト: `8080`
  * コンテナ(Nginxリバースプロキシ)の待ち受けポートを指定できます。
  * なお、Nginx以外のコンポーネントは全て127.0.0.1でListenしているため、外部からアクセスできません。

## コピペについて

* Linuxデスクトップ内は、通常のLinuxデスクトップ環境と同じように、文字列選択でコピー、マウス中クリックでペースト可能です。
  * Linuxデスクトップとブラウザ実行端末間でのコピペは、VNCのツール(画面左)を使ってください。なお、VNCの制約により日本語はコピペできません。
* ターミナルエミュレータの文字は、文字列選択後マウス右クリックでコピー可能です。ペーストは Shift + Insert で可能です。

## その他制限

一部ショートカットは使えないことがわかっています。(以下はChrome on Windowsで試した結果)

* Ctrl + w
   * ブラウザ(タブ)が閉じてしまいます。(閉じる前に警告が出るようにしています)
* Ctrl + n
   * 新規ウィンドウが開いてしまいます。
* Ctrl + t
   * 新規タブが開いてしまいます。
* Alt + .
   * ターミナルエミュレータでは効かないようです。

