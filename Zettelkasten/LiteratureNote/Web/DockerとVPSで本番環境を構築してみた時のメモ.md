---
title: DockerとVPSで本番環境を構築してみた時のメモ
source: https://qiita.com/ta-ke-no-bu/items/fac00841855750d2669f
author:
  - "[[ta-ke-no-bu]]"
published: 2025-01-23
created: 2025-05-22
description: はじめにローカル環境でStrapiを試している中で、 サーバー上でStrapiを利用するためには、サーバーにStrapiやらデータベースをインストールする必要があるから、「せっかくならデータベース…
tags:
  - web
  - Docker
---
![](https://relay-dsp.ad-m.asia/dmp/sync/bizmatrix?pid=c3ed207b574cf11376&d=x18o8hduaj&uid=3516551)

## はじめに

ローカル環境でStrapiを試している中で、 サーバー上でStrapiを利用するためには、サーバーにStrapiやらデータベースをインストールする必要があるから、「せっかくならデータベースごとDockerに入れたままサーバーで動かしたい」と考え、サーバー周りとセキュリティの勉強を兼ねてやってみた時のわたしのやってみた系メモの供養。  
VPSサーバーで作業したけどEC2でもほぼ同じ手順だと思う。

補足: [Strapi](https://strapi.io/) は、Node.js環境で動作するセルフホスト型のオープンソースヘッドレスCMSで、APIを通じてコンテンツを提供する。

※ちなみに説明にちょいちょい名前でるけどStrapi自体は本題的にそこまで重要じゃない。

## 想定する全体の流れ

サーバー上で動かす及び別途フロントをwebサーバーで表示するまでの全体の流れは下記

1. ローカルでDockerプロジェクトの作成
2. ローカル環境Dockerでのプロジェクトの表示・動作確認
3. VPSサーバー契約
4. sshでログインしてみてシステムをアップデートしてからサーバーにDockerインストール
5. 本番環境用プロジェクトのDockfile作成、.env作成、Build、表示・動作確認など
6. CIツール設定と本番環境Dockerでの表示・動作確認
7. 本番環境サーバーでフロント表示用にNginxの設定
8. 本番環境独自ドメインの利用とHTTPS化
9. HTTPS化（Let's encrypt）証明書の自動更新設定など

この投稿では3,4,6,7,8,9などVPSサーバー上のメモが中心

## 前提条件

- バックエンド: Strapi + maria db、本体もデータベースもDocker内で設定
- サーバーは: XserverのVPS（サーバーのOSはUbuntu）
- フロントエンド環境はAstro、サーバー上のDockerは使用せずにローカルだけDockerで作業してCIでデプロイする想定
- 書いた人は普段は主にフロントエンドエンジニア。バックエンドも少々。VPSサーバーは初めて。Dokerでの開発環境は割とやる。
- 便宜上、VPSサーバー上での環境を「本番環境」、ローカルでの環境を「開発環境」と記載する。VPSでとかローカルでとか言ったりもしてて表記揺れがたまにある

## ディレクトリ構成

```text
.
├── .github
│   └── workflows
│       ├── backend.yml
│       └── frontend.yml
├── strapi（backendディレクトリ）
│   ├── Dockerfile（本番及び開発環境用）
│   └── ... (other files)
├── astro（frontendディレクトリ）
│   ├── Dockerfile（開発環境用）
│   └── ... (other files)
├── docker-compose.prod.yml（本番環境用）
└── docker-compose.yml（開発環境用）
```

## Dockerプロジェクトについて

ここでは詳細は割愛してVPSサーバー上で動かすために必要なメモだけ

### バックエンドディレクトリでやったこと

・マルチステージビルドでコンテナイメージを軽量化  
・PID1問題の回避  
・pm2対応（バックエンドがNode.js環境だから）

[Node.jsのDockerfile作成のベストプラクティス](https://zenn.dev/kouchanne/articles/6485193823ecec5735d4)

[Docker のセキュリティに関するベストプラクティス 10 項目](https://snyk.io/jp/blog/10-docker-image-security-best-practices/)

#### PID1問題の回避

PID1問題とは  
Dockerコンテナ内で実行されるプロセスが、本来システムの初期化プロセスが担うべき重要な役割（シグナル管理、子プロセスの管理）を適切に果たせない問題

[Node.jsをDockerで動かすときのPID 1問題について動作確認する](https://mzqvis6akmakplpmcjx3.hatenablog.com/entry/2020/11/10/231047)

#### PM2について

PM2とは  
Node.jsアプリケーション用のプロセスマネージャー  
バックエンド側が止まったら困るので永続的に実行させるためpm2をいれる  
アプリケーションがクラッシュしたり、予期せず終了した場合は自動的に再起動するようにする

流れとしては、バックエンドのディレクトリ内で  
① server.jsファイルをプロジェクトディレクトリに作る  
② PM2のエコシステムファイル（設定ファイル）を作る  
③ PM2で起動する（pm2 start ecosystem.config.js）

[Process Manager - Strapi Developer Documentationopen\_in\_new](https://docs.strapi.io/dev-docs/deployment/process-manager#install-pm2)

[本番環境でStrapiをpm2で動かす](https://yasuda.cloud/entry/2022/06/28/233949)

[What are Process Managers and How to use them with Strapiopen\_in\_new](https://strapi.io/blog/what-are-process-managers-and-how-to-use-them-with-strapi)

## VPSサーバー契約後〜Dockerのインストール

### SSHでログインする

サーバーのコンパネでssh接続用の鍵を作成して適当な場所に保存（/Users/ユーザー名/.ssh/とか）  
[SSH Key | 圧倒的な性能・圧倒的なコスパVPS【XServer VPS】サポートサイト](https://vps.xserver.ne.jp/support/manual/man_server_ssh.php)

```bash
ssh -i sshの証明書.pem root@サーバーから提供されたIPアドレス
```

sshログインでパーミッションエラー（WARNING: UNPROTECTED PRIVATE KEY FILE!）が出たら  
証明書の親フォルダのパーミッションを700

証明書のパーミッションを600に変更

```bash
chmod 700 .ssh/
chmod 600 .ssh/証明書.pem
```

### システム＆セキュリティアップデート

```bash
sudo apt update
sudo apt upgrade
```

### サーバーにDockerをインストール

[VPSでDockerを使ってPythonとFastAPIでAPI環境構築](https://sora.fukui.jp/vps-docker-fastapi/)

[Install Docker Engine on Ubuntu](https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository)

確認

```bash
docker run hello-world
```

Hello from Docker!のメッセージが表示されたらインストールされてる

### サーバーにDockerパッケージとしてdocker-compose-pluginをインストール

```bash
sudo apt-get install docker-compose-plugin
```

[Install Docker Engine on Ubuntu](https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository) の2.Install the Docker packages.の手順通りにやる

下記の方法は非推奨だとコメントで教えていだだきましたけど戒めに折り畳んで置いとく  
[Scenario three: Install the Docker Compose standalone](https://docs.docker.com/compose/install/#scenario-three-install-the-docker-compose-standalone)

\[非推奨\]サーバーにdocker-composeをインストール

```bash
# バージョン確認
docker compose -v

# docker-composeにパスが通ってなかったら下記
docker compose version

# 最新版かどうか公式で確認
# https://github.com/docker/compose

# 場所の確認
which docker-compose

# バージョンが古かった場合はアンインストールして入れ直す
# 1.削除
sudo rm -rf 上で確認したパス

# 2.latestで入れなおす
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

# 3.権限付与
sudo chmod +x /usr/local/bin/docker-compose
```

docker-composeの確認

```bash
docker compose -v
```

メモ: 実際にソースを入れて表示確認などは下記のSTEP2〜3がわかりやすい

[Ubuntu 20.04へのDocker Composeのインストールおよび使用方法](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-compose-on-ubuntu-20-04-ja#2-docker-compose-yml)

### 一般ユーザーでDockerを実行できるようにする

[Ubuntu 22.04にdockerをインストールする](https://qiita.com/yoshiyasu1111/items/17d9d928ceebb1f1d26d#%E4%B8%80%E8%88%AC%E3%83%A6%E3%83%BC%E3%82%B6%E3%81%A7docker%E3%82%92%E5%AE%9F%E8%A1%8C%E3%81%A7%E3%81%8D%E3%82%8B%E3%82%88%E3%81%86%E3%81%AB%E3%81%99%E3%82%8B)

[Ubuntuでユーザーを追加、およびsudo権限を付与する](https://hibiki-press.tech/dev-env/ubuntu/adduser-gpasswd/5086)

一般ユーザーでDockerを実行できるようにする理由  
一般ユーザーでDockerを実行することで、root権限を持つユーザーと通常のユーザーを分けることができるからセキュリティが向上する。あと、一般ユーザーがDockerグループに追加されることで、sudoコマンドなしでDockerコマンドを実行できるため、開発や運用作業が効率化される

**ユーザー追加**

パスワードを設定するのでメモする。

[パスワード生成（パスワード作成）ツール](https://www.luft.co.jp/cgi/randam.php)

ここでは追加するユーザー名は「dockeruser」とする。

```bash
sudo adduser dockeruser
```

アクセスできるユーザの確認

```bash
getent group | grep docker

# 「docker」という文字列を含む行をフィルタリングしてるから多分こんな感じで返ってくる
docker:x:999:
dockeruser:x:1001:
```

戻り値の説明

```bash
docker:x:999:は、
グループ名: docker（Dockerをインストールした際に自動的に作成される「docker」グループ）
グループパスワード: x (シャドウパスワードを使用)
グループID (GID): 999
グループメンバー: なし (空)　　　←　ここ

dockeruser:x:1001:は、
グループ名: dockeruser（adduserで追加したユーザー専用のグループ）
グループパスワード: x (シャドウパスワードを使用)
グループID (GID): 1001
グループメンバー: なし (空)
```

**さっき追加したdockeruserもDocker を使えるようにする**  
dockerグループは、Dockerデーモンへのアクセスを制御する。  
このグループにユーザーを追加することで、そのユーザーはsudoなしでDockerコマンドを実行できるようになる。

```bash
sudo usermod -aG docker dockeruser
```

アクセスできるユーザの確認

```bash
getent group | grep docker

# 多分こんな感じで返ってくる
docker:x:999:dockeruser
dockeruser:x:1001:
```

戻り値の説明

```bash
docker:x:999:dockeruser
グループ名: docker
グループパスワード: x (シャドウパスワードを使用)
グループID (GID): 999
グループメンバー: dockeruser　　　←　ここ　グループに追加することでdockerを使えるようになった
dockeruser:x:1001:
グループ名: dockeruser
グループパスワード: x (シャドウパスワードを使用)
グループID (GID): 1001
グループメンバー: なし (空)
```

ユーザーにsudo権限を付与する

```bash
sudo gpasswd -a dockeruser sudo

or

sudo usermod -aG sudo dockeruser
```

再ログインしたら反映されているはず。

グループ追加のコマンドについて

- `usermod -aG` は複数のグループに同時にユーザーを追加することができる
- `gpasswd -a` は単一のグループへの追加に特化、複数のグループに追加する場合は、コマンドを複数回実行

例：一人のユーザーを複数のグループに追加する場合

```bash
sudo usermod -aG docker,sudo username
```

切り替え

```bash
su dockeruser
```

### 一般ユーザーでログインできるようにする

```bash
# 公開鍵をコピー
cp -r /root/.ssh/ /home/dockeruser/.ssh

# 権限追加
chown -hR dockeruser:dockeruser /home/dockeruser/.ssh
```

ローカルからSSH接続ができるか確認

```bash
ssh -i ~/.ssh/ssh_key_name.pem dockeruser@サーバーから提供されたIPアドレス
```

ここでパスワードを聞かれずパーミッションエラーがでたら、パスワードログインが可能かどうか確認するためにSSHサーバーの設定ファイルの確認

```bash
nano /etc/ssh/sshd_config
```

PasswordAuthenticationがnoだったらyesに変更

再起動

```bash
service ssh restart
or
systemctl restart sshd
```

### サーバ起動時にDockerも起動するように設定

```bash
sudo systemctl enable docker.service
sudo systemctl enable containerd.service
```

## 公開鍵と秘密鍵を作る

1\. ローカルの.sshフォルダにターミナルで移動して鍵を作成

※セキュリティ上鍵生成時にパスフレーズを設定する

[パスワード生成（パスワード作成）ツール](https://www.luft.co.jp/cgi/randam.php)

2\. 鍵の生成:

```bash
ssh-keygen -t ed25519 -a 100 -f id_rsa
```

※ `-t ed25519`: セキュアでパフォーマンスがいいので鍵のタイプはed25519を指定

※ **`-a`** オプションはキーのストレッチング（アルゴリズムによる鍵を派手にする処理）に関連しており、より強力な鍵を生成する

※ **`100`** 回のストレッチングが適用

※ファイル名はわかりやすいのにしとく、ここでは便宜上id\_rsaのまま

上記によりid\_rsa.pub（ **公開鍵** ）とid\_rsa（ **秘密鍵** ）が作成される。

3\. 公開鍵の配置について  
公開鍵はサーバーに配置する。この鍵を使用してサーバーへのSSH接続を認証する。

\[準備\]公開鍵の内容を表示してクリップボードにコピーをする

```bash
pbcopy < ~/.ssh/id_rsa.pub
```

### サーバー側の公開鍵設定

#### A. ローカルから公開鍵をサーバーに簡単にコピーする場合

```bash
ssh-copy-id -i 追加したい公開鍵のパス -o IdentityFile=秘密鍵のパス 一般ユーザー名@サーバーのIPアドレス
```

一般ユーザーのパスワードを聞かれるので入力。もしすでにauthorized\_keysがあれば上書きされてるはず。

ログインしてみる

```bash
ssh 一般ユーザー名@サーバーのポート

or

ssh -i .ssh/id_rsa 一般ユーザー名@サーバーのポート
```

で、ログイン  
秘密鍵のパスフレーズを聞かれたら入力するとログインできる

#### B. サーバー上でauthorized\_keysを作成する場合

VPSサーバーに一般ユーザーでログインしてホームディレクトリでパーミッション700の.sshディレクトリが無かったら作成

1\. ホームディレクトリの中身を見る

```bash
ls -a
```

2..sshがない場合は作成する。あるなら3へ

```bash
mkdir .ssh
chmod 700 .ssh
cd .ssh
```

3..sshに移動して、 **authorized\_keysを作成**

```bash
nano authorized_keys
```

nanoが立ち上がるので、先ほど\[準備\]でコピーした公開鍵を貼り付ける。

保存して終了したら、自分のみ読み書き可能にする

```bash
chmod 600 authorized_keys
```

4\. ログインしてみる

```bash
ssh -i .ssh/id_rsa 一般ユーザー名@サーバーのポート
```

で、ログイン  
秘密鍵のパスフレーズを聞かれたら入力するとログインできる

## 通常ログインでの接続を禁止する

### パスワードログインを禁止

パスワードログインが可能かどうか確認

```bash
nano /etc/ssh/sshd_config
```

**PasswordAuthentication** がyesだったらnoに変更

再起動

```bash
service ssh restart
or
systemctl restart sshd
```

### rootユーザーのSSHログインを禁止

【流れ】

1. rootじゃないユーザーでSSHログインできることを確認
2. rootじゃないユーザーでsudo でrootと同じコマンドを実行できる or sudo su - でrootになれるか、sudo ls /rootでrootディレクトリにアクセスできるか
3. rootユーザーのSSHを制限する

一般ユーザーでログイン。設定を変更するためにrootになる

```bash
sudo su -
```

念のためバックアップ

```bash
cp /etc/ssh/sshd_config /etc/ssh/sshd_config.org
```

sshd\_configを開く

```bash
sudo nano /etc/ssh/sshd_config
```

PermitRootLoginをオフにしてファイルを保存してエディタを閉じる

sshの再起動

```bash
sudo systemctl restart sshd
```

## ファイアウォールの設定

### 現状確認

```bash
sudo ufw status

# 稼働中 Status: active
# 稼働してない Status: inactive
```

### デフォルト設定

```bash
# 外部からの入力トラフィックはデフォルトで拒否
sudo ufw default deny incoming
# 内部から外部への出力トラフィックはデフォルトで許可
sudo ufw default allow outgoing
```

接続を維持するためsshのデフォルトの22 と切り替え予定の「SSHだと推測されにくい」ランダムなポートを決めて許可

※22番ポートは攻撃されやすいので、22とかけ離れた数値がいい

※他のサーバーとのバッティングを避けるため、 `/etc/services` に記載 **されていない** 番号を選ぶ

```bash
cat /etc/services
```

### ポートの許可

22番ポートは狙われやすいのでsshのポートを変更する準備

```bash
# sshの標準のポートを許可。後で変更するけど一時的に許可
ufw allow 22

# 使おうと思ってるランダムなポートが本当に使用されていないか確認
sudo lsof -i:ランダムなポート
# 上でなにも表示されなければ許可する
ufw allow ランダムなポート
```

[**ランダムポートジェネレーター**](https://ja.proxyscrape.com/tools/random-port-generator)

### パケットフィルターの設定

※Xserverはこの手順が必要

1. サーバーのコントロールパネルにアクセスしてパケットフィルター設定を開く
2. パケットフィルターをオンにして **フィルタールール** 設定を追加する
3. フィルター「カスタム」プロトコル「TCP」ポート番号「先程のランダムなポート」許可する送信元IPアドレス「すべて許可」

### 使用ポートを変更

sshd\_configを開く

```bash
sudo nano /etc/ssh/sshd_config
```

/etc/ssh/sshd\_configを開いたら#Port 22という記述を探す  
コメントアウトを外して、22を先程設定したランダムなポートに書き換えて保存して終了

再起動

```bash
systemctl restart sshd
```

### 変更後のポートでログインできるか試す

```bash
ssh -p ランダムなポート ユーザー名@サーバーのIP
```

### ファイアウォールを有効にする

```bash
# ファイアウォールを有効
# 既存の ssh 接続が切断されるかもしれないけどいい？って聞かれたらyesとお返事
sudo ufw enable

# ステータス確認
sudo ufw status

# ここまでやったら下記のようになってるはず
Status: active

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW       Anywhere
ランダムなポート              ALLOW       Anywhere
22/tcp (v6)                ALLOW       Anywhere (v6)
ランダムなポート (v6)         ALLOW       Anywhere (v6)
```

### 特定のIPのアクセスを制限

```bash
# 特定のIPのアクセスを許可
sudo ufw allow from 特定のIP to any port ランダムなポート

# 参考
# 特定のIPからのアクセスを拒否
sudo ufw deny from 特定のIP  
# 特定のIPの拒否設定を解除
sudo ufw delete deny from 特定のIP
```

### ログの有効化

ファイアウォールの動作やブロックされたトラフィックに関する情報を **`/var/log/ufw.log`** ファイルに記録する

```bash
# ログを有効にする
sudo ufw logging on

# リロード
sudo ufw reload
```

ログファイルの種類

```bash
# ログイン試行の記録
# 誰がログインを試みたか、失敗したか
# このログは通常、/var/log/btmp ファイルに保存
sudo lastb

# grepで自分のIPアドレス以外
sudo lastb | grep -v "自分のIPアドレス"

# 認証関連のログ情報
# SSHのログイン試行、sudoの使用などがこのログに記録
sudo cat /var/log/auth.log

# grepでFailed passwordのみ抽出
sudo cat /var/log/auth.log | grep "Failed password"
# grepでdockeruserじゃないもののみ抽出
sudo cat /var/log/auth.log | grep -v "dockeruser"

# Uncomplicated Firewall (UFW) によって発生するログ
# ファイアウォールに関する情報がここに記録され、不正な接続試行や
# ファイアウォールによってドロップされたトラフィックの表示
sudo cat /var/log/ufw.log
```

### 補足:確認するべきログファイルとコマンド

セキュリティやシステムのトラブルシューティングのために確認するべきログファイルと、それを表示するためのコマンド

1. **`last` コマンド:**
	- `/var/log/wtmp` ファイルを利用して、ユーザーのログインとログアウトの履歴を表示する
	- `last` コマンドを使用して、直近のログイン履歴を表示できる
	```bash
	sudo last
	```
2. **`journalctl` コマンド:**
	- `systemd` イベントログを表示するために使用され、システムイベントやサービスのログ情報が含まれる
	```bash
	sudo journalctl
	```
3. **`dmesg` コマンド:**
	- カーネルがブートおよび実行中に発生したメッセージを表示
	```bash
	dmesg
	```
4. **`/var/log/syslog` ファイル:**
	- システムの一般的なログ情報が含まれる
	```bash
	cat /var/log/syslog
	```
5. **`/var/log/kern.log` ファイル:**
	- カーネルに関するログ情報が含まれる
	```bash
	cat /var/log/kern.log
	```

これらのコマンドはシステムの状態や異常を把握するのに役立ち、ログの情報から問題の特定やセキュリティの監視をする

## セキュリティ強化

### FWで攻撃の対策

30秒間の間に6回以上接続を試みた IP アドレスを拒否

```bash
sudo ufw limit ランダムなポート/tcp
```

使わなくなったポート22を拒否

```bash
sudo ufw deny 22
```

※Xserverはコンパネのパケットフィルターからも削除

sshd\_configを開く

保存して再起動

```bash
sudo systemctl restart sshd
```

### SYNCookieが有効かを確認

[【まとめ】SYNフラッド (Flood) 攻撃とSYN Cookie](https://zenn.dev/nosehana/articles/98517087ca0ded)

変更を適用

```bash
sudo sysctl -p
```

### libpam-google-authenticatorの導入（2段階認証）

libpam-google-authenticator  
Linuxシステムにおいて二段階認証（2FA）を実装するためのPAM（Pluggable Authentication Module）モジュール

[SSHログインにGoogle Authenticatorを使用する](https://security.sios.jp/security/ssh_google_authenticator/)

```bash
sudo apt update

# インストール
sudo apt install -y libpam-google-authenticator
```

/etc/pam.d/sshdを開く

```bash
sudo nano /etc/pam.d/sshd
```

メモ:編集権限がなかったら一回閉じて権限付与

```bash
sudo chown ユーザー名: /etc/pam.d/sshd
```

再起動

```bash
sudo systemctl restart sshd
```

sshd\_configを開く

```bash
sudo nano /etc/ssh/sshd_config

# KbdInteractiveAuthenticationを探してnoからyesに変更
# キーボードインタラクティブ認証を有効にすることにより、ユーザーはパスワード入力後にGoogle Authenticatorから生成されたコードを入力する必要がある
KbdInteractiveAuthentication yes
#チャレンジレスポンスの設定をyesに変更
# 多要素認証の一部として機能し、セキュリティを向上
ChallengeResponseAuthentication yes

# 少しスクロールしてUsePAMがyesであることを確認
#PAMはPluggable Authentication Modules
#PAMを使用して認証を管理することにより、追加の認証方法が利用可能になる
UsePAM yes
```

保存して再起動

```bash
sudo systemctl restart sshd
```

google-authenticatorを実行

```bash
google-authenticator
```

対話の翻訳

```bash
Do you want authentication tokens to be time-based
# 認証トークンを時間ベースにするか
# Google Authenticatorは時間ベースなのでyes

# QRコードが表示されるので、
# Authyなど2段階認証アプリで読み取るとコード生成されるのでコードを入力してEnter

Your new secret key is: xxxxxxxxxxxxxxxxxxxxxxx
Enter code from app (-1 to skip): xxxxxxxx
Code confirmed
Your emergency scratch codes are:
  XXXXXXXX
  XXXXXXXX
  XXXXXXXX
  XXXXXXXX
  XXXXXXXX

# 緊急コードが5個出てくるのでメモしておく

Do you want me to update your "/home/ユーザー名/.google_authenticator" file? (y/n) 
# ホームディレクトリのgoogle_authenticatorを更新するかどうかなのでyes

Do you want to disallow multiple uses of the same authentication
token? This restricts you to one login about every 30s, but it increases
your chances to notice or even prevent man-in-the-middle attacks (y/n) y
# 同じ認証トークンを複数回使用することを許可しますか？
# この場合、ログインは30秒に1回に制限されますが、マン・イン・ザ・ミドル・アタックに気づく可能性が高まります。
# 中間者攻撃に気づいたり、防いだりする可能性が高まります。

By default, a new token is generated every 30 seconds by the mobile app.
In order to compensate for possible time-skew between the client and the server,
we allow an extra token before and after the current time. This allows for a
time skew of up to 30 seconds between authentication server and client. If you
experience problems with poor time synchronization, you can increase the window
from its default size of 3 permitted codes (one previous code, the current
code, the next code) to 17 permitted codes (the 8 previous codes, the current
code, and the 8 next codes). This will permit for a time skew of up to 4 minutes
between client and server.
Do you want to do so? (y/n) y
# デフォルトでは、モバイルアプリによって30秒ごとに新しいトークンが生成されます。
# クライアントとサーバー間で起こりうる時間のズレを補正するために
# 現在時刻の前後の追加トークンを許可します。これにより
# 認証サーバーとクライアントの間で最大30秒のタイムスキューが発生します。もし
# 時刻の同期がうまくいかない場合は、デフォルトの3個の許可コード（1つ前のコード、現在のコード、次のコード)から、
# 17個の許可コード(8個前のコード、現在のコード、および次の8コード)までウィンドウを大きくすることができます。
# これにより、クライアントとサーバーの間で最大4分のタイムスキューを許容することができます。
# そうしますか？

If the computer that you are logging into isn't hardened against brute-force
login attempts, you can enable rate-limiting for the authentication module.
By default, this limits attackers to no more than 3 login attempts every 30s.
Do you want to enable rate-limiting? (y/n) y
# ログインしているコンピューターがログインの試行のブルートフォースに対して強化されていない場合
# 認証モジュールのレート制限を有効にすることができます。
# デフォルトでは、攻撃者は30秒に3回までしかログインを試行できないように制限されています。
# レート制限を有効化しますか？
```

### fail2banの導入

fail2ban  
ファイルを監視して特定の条件（例えば、同一IPからの認証失敗が一定回数を超えた場合とか）に基づいて、そのIPアドレスを一時的にブロックするなどしてブルートフォース攻撃やDoS攻撃からサーバーを保護する

```bash
sudo apt update
sudo apt install fail2ban
```

**設定ファイルのバックアップ**

```bash
sudo cp -p /etc/fail2ban/jail.{conf,local}
```

設定ファイルの編集

```bash
sudo nano /etc/fail2ban/jail.local
```

sshdの設定

例は「30分間で3回アクセス失敗したら1時間ban」

```bash
[sshd]
・・・
# 以下を追加
enabled = true
maxretry = 3
findtime = 1800  # 30分
bantime = 3600  # 1時間
```

recidiveの設定

例は「6時間で３回banされたIPは永久に接続拒否」

```bash
[recidive]
・・・
# 以下を追加
enabled = true
bantime = 21600  # 6時間
findtime = 21600  # 6時間
maxretry = 3
```

有効にして開始

```bash
# 有効にする
sudo systemctl enable fail2ban
# スタート
sudo systemctl start fail2ban

# ステータスの確認
sudo systemctl status fail2ban
```

メモ: ちょっとした設定の変更はFail2ban クライアントでやってもいい

```bash
# -hでコマンド確認
fail2ban-client -h

# 例
# IPを拒否:
sudo fail2ban-client set sshd banip BANしたいIP
```

メモ: もしメールサーバーをいれたらメールサーバーでも設定しないといけない

jail.localにpostfixとかnginxとかmysqld-authとか設定があったのでログを見て都度確認

[Ubuntu20.04 fail2ban](https://rohhie.net/ubuntu20-04-fail2ban/)

### おまけ:~/.ssh/configファイルにエントリを追加する

毎回パラメータつけたりしてログインするのがめんどかったら以下を編集

```bash
nano ~/.ssh/config
```

```bash
Host myserver（ホストネーム）
    HostName IPアドレス
    User ユーザー名
    Port ポート
    IdentityFile ~/.ssh/id_rsa
    IdentitiesOnly yes
    AddKeysToAgent yes
```

上記の設定をしておけばホストネームのみでssh接続が可能

```bash
ssh myserver
```

※IdentitiesOnly yesはこのホストへの接続に指定された鍵以外を使用しない

※ AddKeysToAgent yesは `ssh` によって自動的に `ssh-agent` に鍵が追加されるので、初回のみパスワードが必要だけど次回から `ssh-agent` から呼び出すので入力不要

[基礎から学ぶ！最初にやるべきSSHのセキュリティ設定【全コマンド例付き】](https://hackers-high.com/linux/ssh-config-for-beginners/)

### セキュリティチェックリスト

[サーバー作成直後に設定しておくべき初期セキュリティ設定](https://manual.sakura.ad.jp/vps/support/security/firstsecurity.html)

- **root ユーザーに SSH ログインできないこと**
- **一般ユーザーに公開鍵認証を用いないと SSH ログインできないこと**
- **sudo コマンドを実行して、 root 権限としてのコマンドが実行できる**
- **アップデート計画やバックアップ計画を立てたか確認する**
- **ファイアウォールが機能しているか確認する**
	- 対象となるポートがファイアウォールによってブロックされているかどうかを確認
	- **telnet コマンドを使用する場合:**
		```bash
		telnet your_server_ip your_port
		```
	- **nmap コマンドを使用する場合:**
		```bash
		nmap -p your_port your_server_ip
		```

## デプロイ

github Actionsを使用  
ここではバックエンドのワークフローのみ、フロントエンドは本題と関係ないので割愛

### 準備

vpsのgitをアップデート

```bash
# バージョン確認
git version

# 公式のバージョン確認
# https://git-scm.com/

# 古かったらアップデートする
# ppa:git-core/ppaはUbuntu システムで Git を提供するための個別のパッケージアーカイブ（PPA）
sudo add-apt-repository ppa:git-core/ppa
sudo apt update
sudo apt upgrade

# バージョン確認
git version
```

### VPSサーバー上でgithub接続用の鍵を作成

サーバーからgithubに接続する用の鍵

```bash
ssh-keygen -t ed25519 -a 100 -f ~/.ssh/github
```

できたらコピー

```bash
# 中身を開いてコピーしておく
cat ~/.ssh/github.pub
```

パーミッション変更

```bash
chmod 600 ~/.ssh/github
```

config作成

```bash
nano ~/.ssh/config
```

以下を貼り付ける

```yaml
Host github.com
        User git
        IdentityFile ~/.ssh/github
```

保存してパーミッションを600に変更

```bash
chmod 600 ~/.ssh/config
```

### githubに登録

Githubの [SSH and GPG keys](https://github.com/settings/keys) へアクセス、\[New SSH Key\]をクリックして公開鍵を登録する

**SSH keys / Add new** ページが開いたら、 **Title** に鍵識別のための適当な文字列を入力。  
**Key** に公開鍵(文字列)を貼り付け

VPS操作に戻って、gitに接続してみる

```bash
ssh -T git@github.com
# 初回は信頼性を確立できません。本当に接続を続けますか？って聞かれるのでyes
# パスコードを設定していたらパスコードを入力

# 以下のメッセージが表示されたら成功
Hi リポジトリ名! You've successfully authenticated, but GitHub does not provide shell access.
```

クローンする

```bash
git clone git@github.com:gitのユーザー名/リポジトリ名.git /home/sshのユーザー名/ディレクトリ名

# パスワードを聞かれるので.ssh/github.pubのpassを入力
```

### Githubでシークレットキーの設定

1\. GitHubリポジトリの設定から「Secrets and variables」→「Actions」に移動し、新しいリポジトリシークレットを作成

- SSH\_◯◯はgithubからサーバーに接続する情報
	- SSH\_HOST: サーバーのホスト名またはIPアドレス
	- SSH\_USERNAME: SSH接続に使用するユーザー名
	- SSH\_PORT: SSH接続ポート（通常は22）
	- SSH\_PRIVATE\_KEY: サーバーへのアクセスに使用する秘密鍵
	- SSH\_PASSPHRASE: 秘密鍵作成時のパスフレーズ（必要な場合）
	- DEPLOY\_DIR\_PATH: サーバー上のデプロイ用ディレクトリ

2\. 環境変数の設定

- .envファイルで設定してる環境変数もここで設定した。（注意:後述）
	- ENV\_FILE:.envの中身まるごと（コメントアウトなどは消しておく。コメントアウト内の（）でエラーが出た）

### Github Actionsのワークフロー作成

プロジェクトにワークフロー用ディレクトリ作成

```bash
mkdir .github/workflows
```

【想定する流れ】  
GitHub Actions内でDockerイメージをビルドし、ghcr.ioにプッシュした後、SSH接続によってサーバーにデプロイする流れをイメージ

1\. Dockerイメージのビルドとプッシュ

- ここではdocker/build-push-actionでDockerイメージをビルドしてghcr.ioにプッシュする方法をとる

2\. 環境設定ファイルの準備

- .envファイルは、Dockerイメージのビルド前に用意する

3\. SSH接続とデプロイ

- appleboy/ssh-action@masterを利用してSSH接続してから
	- git pullで最新のコードを取得
	- .envファイルをsecretsから生成（注意:後述））
	- ghcr.ioからプッシュしたDockerイメージをpull
	- docker compose upコマンドでコンテナを起動

そうしてできたサンプル

注意  
最後のdocker-compose.prod.ymlを起動する時に.envの値が呼び出せなかったのでいろいろ試して先に進めるのを優先した結果「.envファイルの中身をsecretsにぶち込んで新たに.envファイルを生成する」という方法を取ってるけど、セキュリティ的によろしくないと思うのであくまでサンプル。

backend.yml

```yaml
# workflowの名前
name: CI Backend Docker

# トリガーの設定
on:
  push:
    branches:
        - 'main'
    paths:
        - 'strapi/**' # strapiディレクトリの中の変更がプッシュされたら実行

# 環境変数の設定
env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}
  DEPLOY_DIR_PATH: ${{ secrets.DEPLOY_DIR_PATH }}

jobs:
  build-and-push-image:
    runs-on: ubuntu-latest # 実行環境の設定
    permissions: # パッケージの読み取り権限と書き込み権限の設定
      contents: read
      packages: write
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4 # リポジトリのコードをチェックアウト

      - name: Log in to the Container registry
        uses: docker/login-action@v3 # Dockerレジストリにログイン
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.MY_GITHUB_TOKEN }}

      - name: Extract metadata (tags, labels) for Docker
        id: meta
        uses: docker/metadata-action@v5 # Dockerイメージに関するメタデータを抽出
        with:
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}

      - name: "Create env file" # secretsから.envを作成
        run: |
          echo "${{ secrets.ENV_FILE }}" > .env

      - name: Build and push Docker image
        uses: docker/build-push-action@v5 # Dockerイメージをビルドしてghcr.ioにプッシュ
        with:
          context: .
          file: ./strapi/Dockerfile
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
  deploy:
    runs-on: ubuntu-latest
    needs: build-and-push-image
    permissions: # パッケージの読み取り権限と書き込み権限の設定
      contents: read
      packages: write
    steps:
      - name: executing remote ssh commands using ssh key
        uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.SSH_HOST }}
          username: ${{ secrets.SSH_USERNAME }}
          key: ${{ secrets.SSH_PRIVATE_KEY }}
          passphrase: ${{ secrets.SSH_PASSPHRASE }}
          port: ${{ secrets.SSH_PORT }}
          # サーバーのソースとgitのソースの差分が出てくると思うのでgitpullしてから
          # secretsに放り込んだenvを作ってイメージをプルしてprodでビルドするようにした
          script: |
            cd ~/${{ env.DEPLOY_DIR_PATH }}
            git pull
            echo "${{ secrets.ENV_FILE }}" > .env
            docker pull ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:latest
            docker compose -f docker-compose.prod.yml up -d
```

ここまでで、一旦バックエンドの本番環境の構築が完了

## Nginx設定・ドメイン割当・SSL対応

フロントの表示用の作業

[【Ubuntu編】エックスサーバーVPSにNginxをインストールしてWEB環境を構築する方法](https://sabameijin.com/ubuntu-nginx-web/)

[UbuntuにNginxをインストールしてWebサーバー構築【マルチドメイン】](https://self-development.info/ubuntu%E3%81%ABnginx%E3%82%92%E3%82%A4%E3%83%B3%E3%82%B9%E3%83%88%E3%83%BC%E3%83%AB%E3%81%97%E3%81%A6web%E3%82%B5%E3%83%BC%E3%83%90%E3%83%BC%E6%A7%8B%E7%AF%89%E3%80%90%E3%83%9E%E3%83%AB%E3%83%81/)

[UbuntuにNginxを設定しよう｜複数ドメイン・HTTPS化](https://itc.tokyo/nginx/ubuntu-nginx/)

事前準備: ドメインを取得

### Nginxのインストール

ウェブサーバーとしてHTTPS通信を実現するためにNginxをいれる

パッケージを最新状態にアップデート

```bash
sudo apt update
```

nginxの状況確認

```bash
sudo apt info nginx
```

インストールされていなかったらnginxのインストール

```bash
sudo apt install nginx -y
# -yは途中で聞かれる質問すべてyesにするオプション
```

Nginxを起動

```bash
sudo systemctl start nginx
```

VPS再起動後も自動で起動させる

```bash
sudo systemctl enable nginx
```

Nginxのルートディレクトリの確認

```bash
grep "root /" -r /etc/nginx/
```

**備考**: nginxのよく使うコマンド

```bash
# 起動
sudo systemctl start nginx
# 停止
sudo systemctl stop nginx
# 再起動
sudo systemctl restart nginx
# 状態確認
sudo systemctl status nginx
# 再読み込み（設定変更後などに使う）
sudo systemctl reload nginx
# 自動起動設定を有効
sudo systemctl enable nginx
# 自動起動設定を無効
sudo systemctl disable nginx
```

### Nginxのインストール後のファイアウォール設定

ufwで利用可能なアプリケーションかどうかを確認

```bash
sudo ufw app list
```

Nginxがインストールされていたら次の3つがあるはず

- Nginx Full: ポート80（http）とポート443（https）の両方が対象
- Nginx HTTP: ポート80（http）が対象
- Nginx HTTPS: ポート443（https）が対象

両方使いたいからNginx Fullを有効にする

```bash
sudo ufw allow 'Nginx Full'

# もちろんポート番号で管理してもいい
sudo ufw allow 80
sudo ufw allow 443
```

アクセスが許可されたか確認

```bash
sudo ufw status
```

#### パケットフィルターの設定

※Xserverはこの手順が必要

1.サーバーのコントロールパネルにアクセスしてパケットフィルター設定を開く

2.パケットフィルターをオンにして **フィルタールール** 設定を追加する

3.フィルター「web（20 / 21 / 80 / 443）」を追加

#### webで確認

nginxが起動しているかどうか確認

```bash
systemctl is-active nginx

# activeだったら起動してる
```

ブラウザでサーバーのIPアドレスでアクセスしてみる

「 **Welcome to nginx!**」と書いたページが表示されたらOK

※ちなみにそのhtmlの場所は/var/www/html（grep "root /" -r /etc/nginx/でrootって言われた場所）/index.nginx-debian.html

### DNSレコードの設定

今回はxserver ドメインで取得したドメインをxserver VPSで使えるようにする（と、思ったら自動で設定されてた）

[ネームサーバーの設定 | 圧倒的な性能・圧倒的なコスパVPS【XServer VPS】サポートサイト](https://vps.xserver.ne.jp/support/manual/man_domain_namesever_setting.php)

ドメイン取得とサーバーが別々の会社の場合は以下

| **1.ネームサーバーの設定** | ドメインを取得したサービスのコンパネ |
| --- | --- |
| **2.ドメインの設定** | サーバーのコンパネ |

#### 手順

1. サーバーにドメイン設定しておく（ドメイン設定とかあるはず）
2. サーバーで提供しているネームサーバーを確認
3. ドメインを取得した（お名前.comとかxserver Domainとか）サービスの管理画面のメニューでネームサーバー設定みたいなメニューを選択
4. **プライマリネームサーバー、セカンダリネームサーバーなど必要なネームサーバーを入力**
5. ネームサーバーの設定が完了したら、次はDNSレコードの設定で、サーバーのコンパネを開く
6. DNS設定みたいな項目を探す
7. ドメインを追加してDNSレコードの追加画面を開く（Xserver同士だったらすでに設定されてる）
8. WEBサイトの設定を行う場合には、通常は、 **エイリアス（www）のあり・なしの両方** へアクセスできるように設定。DNSレコード設定の追加でwww.ドメイン名の設定もする。xserverだと内容っていう項目にはサーバーのIPアドレスを入力。種別やTTLはデフォルトのまま

ブラウザで、　 [http://www](http://www/) + ドメイン名でアクセスしてみる。

「 **Welcome to nginx!**」と書いたページが表示されたらOK

### バーチャルホストの設定

フロントエンド用ディレクトリを作成

```bash
# 今いる場所の確認
pwd
# root（/home/ユーザー名）にいたらwwwディレクトリを作る
mkdir www
# wwwディレクトリ内のすべてのファイル・サブディレクトリの所有者をユーザーにする
sudo chown -R ユーザー名:www-data /home/ユーザー名/www
# Ubuntuの場合、Nginxのデフォルトの実行ユーザーは通常www-data

# パーミッションを755にする
sudo chmod 755 /home/ユーザー名/www
```

### Nginxの設定

設定ファイルはubuntuの場合、 **`/etc/nginx/nginx.conf`** にある。サイトごとの設定は **`/etc/nginx/sites-available/`** ディレクトリに個別の設定ファイルとして配置され、 **`/etc/nginx/sites-enabled/`** にシンボリックリンクを作成する

**※confファイル名には、管理しやすいようにドメイン名を使用する**

#### sites-available/defaultを見てみる

```yaml
# of Nginx configuration files in order to fully unleash the power of Nginx.
# https://www.nginx.com/resources/wiki/start/
# https://www.nginx.com/resources/wiki/start/topics/tutorials/config_pitfalls/
# https://wiki.debian.org/Nginx/DirectoryStructure
#
# In most cases, administrators will remove this file from sites-enabled/ and
# leave it as reference inside of sites-available where it will continue to be
# updated by the nginx packaging team.
#
# This file will automatically load configuration files provided by other
# applications, such as Drupal or Wordpress. These applications will be made
# available underneath a path with that package name, such as /drupal8.
#
# Please see /usr/share/doc/nginx-doc/examples/ for more detailed examples.
##

# Default server configuration
#
server {
        listen 80 default_server;
        listen [::]:80 default_server;

        # SSL configuration
        #
        # listen 443 ssl default_server;
        # listen [::]:443 ssl default_server;
        #
        # Note: You should disable gzip for SSL traffic.
        # See: https://bugs.debian.org/773332
        #
        # Read up on ssl_ciphers to ensure a secure configuration.
        # See: https://bugs.debian.org/765782
        #
        # Self signed certs generated by the ssl-cert package
        # Don't use them in a production server!
        #
        # include snippets/snakeoil.conf;
        root /var/www/html;

        # Add index.php to the list if you are using PHP
        index index.html index.htm index.nginx-debian.html;

        server_name _;

        location / {
                # First attempt to serve request as file, then
                # as directory, then fall back to displaying a 404.
                try_files $uri $uri/ =404;
        }

        # pass PHP scripts to FastCGI server
        #
        #location ~ \.php$ {
        #       include snippets/fastcgi-php.conf;
        #
        #       # With php-fpm (or other unix sockets):
        #       fastcgi_pass unix:/run/php/php7.4-fpm.sock;
        #       # With php-cgi (or other tcp sockets):
        #       fastcgi_pass 127.0.0.1:9000;
        #}

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #       deny all;
        #}
}
# Virtual Host configuration for example.com
#
# You can move that to a different file under sites-available/ and symlink that
# to sites-enabled/ to enable it.
#
#server {
#       listen 80;
#       listen [::]:80;
#
#       server_name example.com;
#
#       root /var/www/example.com;
#       index index.html;
#
#       location / {
#               try_files $uri $uri/ =404;
#       }
#}
```

翻訳してみる

VPSは下部の「Virtual Host configuration for example.com」のお手本を使うっぽい

#### confを作成

defaultの下のサンプル部分をコピーして適宜書き換える

```bash
server {
       listen 80;
       listen [::]:80;

       server_name example.com;

       root /var/www/example.com;
       index index.html;

       location / {
               try_files $uri $uri/ =404;
       }
}
```

```bash
# confファイル作成
sudo nano /etc/nginx/sites-available/ドメイン名.conf
# 書き換えたサンプルを貼り付ける

# シンボリックリンク作成
sudo ln -s /etc/nginx/sites-available/ドメイン.conf /etc/nginx/sites-enabled/

# リンクが作成されてるか確認
ll /etc/nginx/sites-enabled/
# ls -alF /etc/nginx/sites-enabled/でもいい　、ls -l /etc/nginx/sites-enabled/でもいい
# ドメイン名.conf -> /etc/nginx/sites-available/ドメイン名.confの表示が出たらOK

# エラーチェック
sudo nginx -t

# nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
# nginx: configuration file /etc/nginx/nginx.conf test is successful
# と返ってくれば問題ない。エラーが出てたら修正して再度チェック

# Nginxの再起動
sudo systemctl restart nginx
```

これでドメインにアクセスした時の表示するファイルはwww/index.htmlになったはずなので

```bash
nano www/index.html
```

適当なhtmlを置いてみる

index.html

```html
<!doctype html>
<html lang="js">
<head>
    <meta charset="utf-8">
    <title>Nginxのデモ</title>
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/gh/kognise/water.css@latest/dist/dark.min.css">
</head>
<body>

    <h1>Nginxの設定をした</h1>
    <p>ちゃんと出来ていれば表示するはず</p>

</body>
</html>
```

ブラウザでアクセスしてみて表示されたらOK

※エラーの場合

```bash
# ログを見る
sudo cat /var/log/nginx/error.log
```

パーミッションの数値の出し方がいまいちわかっていなくて「13: Permission denied」で困った。

```bash
# パーミッションを755にする
sudo chmod 755 /home/ユーザー名/www
# 上の階層のパーミッションを調べる
ls -ld /home/ユーザー名/
# ls -ld /home/ユーザー名/はディレクトリ自体の情報
# ls -l /home/ユーザー名/はそのディレクトリ内のファイルやサブディレクトリも含めた情報

# 上の階層のパーミッションが755じゃないから上の階層のパーミッションも変える
sudo chmod 755 /home/ユーザー名
```

ちなみに・・・パーミッションの数値の出し方

- **`r`** （読み取り可能）: 4
- **`w`** （書き込み可能）: 2
- **`x`** （実行可能）: 1

例として "drwxrwxr-x" の場合：  
d/rwx/rwx/r-x ・・・1文字目を除いて3つずつ区切る

- d: ディレクトリを示すフラグ（計算には含まない、ファイルである場合は **`-`** が表示される）
- 所有者 (rwx): 4 + 2 + 1 = 7
- グループ (rwx): 4 + 2 + 1 = 7
- その他 (r-x): 4 + 0 + 1 = 5  
	したがって、数値表現は 775 で「所有者とグループは書き込み可能だけどその他は書き込み不可」  
	755は「所有者以外書き込み不可」

#### サブドメインについて

htmlの格納ファイルを/home/ユーザー名/wwwとする。  
例えばサブドメインで/home/ユーザー名/www/subだけsub.ドメイン.comと設定し、その他はwww.ドメイン.comと設定する場合

```bash
server {
    listen 80;
    server_name www.ドメイン.com;

    root /home/ユーザー名/www;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}

server {
    listen 80;
    server_name sub.ドメイン.com;

    root /home/ユーザー名/www/sub;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

サブドメインで/home/ユーザー名/www/subだけsub.ドメイン.comと設定し、その他はwww.ドメイン.comと設定する場合のnginxのconfファイル

/home/ユーザー名/www/はwww.ドメイン.comでもドメイン.comでもアクセス可能  
/home/ユーザー名/www/も/home/ユーザー名/www/subも別途SSL証明書を発行して使用

```bash
server {
    listen 80;
    server_name www.ドメイン.com ドメイン.com;

    root /home/ユーザー名/www;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
        # その他の設定
    }

    # HTTPからHTTPSへのリダイレクト
    location ~ /.well-known/acme-challenge/ {
        allow all;
        root /var/www/html;
    }

    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name www.ドメイン.com ドメイン.com;

    ssl_certificate /etc/nginx/ssl/ドメイン.com.crt;
    ssl_certificate_key /etc/nginx/ssl/ドメイン.com.key;

    root /home/ユーザー名/www;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
        # その他の設定
    }
}

server {
    listen 443 ssl;
    server_name sub.ドメイン.com;

    ssl_certificate /etc/nginx/ssl/サブドメイン.com.crt;
    ssl_certificate_key /etc/nginx/ssl/サブドメイン.com.key;

    root /home/ユーザー名/www/sub;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
        # その他の設定
    }
}
```

### SSLの設定

[【Ubuntu編】エックスサーバーVPSにNginxをインストールしてWEB環境を構築する方法](https://sabameijin.com/ubuntu-nginx-web/)

[UbuntuにNginxを設定しよう｜複数ドメイン・HTTPS化](https://itc.tokyo/nginx/ubuntu-nginx/)

[Ubuntu 20.04でLet’s Encryptを使用してNginxを保護する方法](https://www.digitalocean.com/community/tutorials/how-to-secure-nginx-with-let-s-encrypt-on-ubuntu-20-04-ja)

【流れ】  
Let's Encryptを使用

1. certbotとpython3-certbot-nginxをインストール
2. 証明書の取得
3. 自動更新の検証

#### certbotとpython3-certbot-nginxをインストール

Certbotは  
Let's Encryptが提供する自動化ツールで、SSL/TLS証明書を簡単に取得し、管理するためのクライアント

python3-certbot-nginxは  
Certbotのプラグインの一つで、Nginxウェブサーバーと連携してSSL/TLS証明書を取得および管理するためのツール。Let's Encryptからの証明書取得プロセスが簡素化され、Nginxの設定ファイルも自動的に更新される。

```bash
sudo apt install certbot python3-certbot-nginx
```

Certbotのプラグインを介してSSL証明書を設定する

```bash
sudo certbot --nginx -d example.com -d www.example.com
```

`-nginx` プラグインで `certbot` を実行し、 `-d` を使用して証明書を有効にするドメイン名を指定

初回はメールアドレスとか聞かれる。途中に「お知らせをメールで送る」みたいな問いがあるから受け取りたくなかったらちゃんと読んだほうがいい

```bash
Saving debug log to /var/log/letsencrypt/letsencrypt.log
Enter email address (used for urgent renewal and security notices)
 (Enter 'c' to cancel):

デバッグ・ログを/var/log/letsencrypt/letsencrypt.logに保存する。
メールアドレスを入力（緊急の更新やセキュリティの通知に使用します）
 (キャンセルするには'c'を入力）： **メールアドレス**

--------------------------------------------------------------

Please read the Terms of Service at
https://letsencrypt.org/documents/LE-SA-v1.3-September-21-2022.pdf. You must
agree in order to register with the ACME server. Do you agree?
****

以下の利用規約をお読みください。
https://letsencrypt.org/documents/LE-SA-v1.3-September-21-2022.pdf。ACMEサーバーに登録するには
ACMEサーバーに登録するために同意する必要があります。同意しますか？ **y**
--------------------------------------------------------------

Would you be willing, once your first certificate is successfully issued, to
share your email address with the Electronic Frontier Foundation, a founding
partner of the Let's Encrypt project and the non-profit organization that
develops Certbot? We'd like to send you email about our work encrypting the web,
EFF news, campaigns, and ways to support digital freedom.

最初の証明書の発行に成功したら、次のことを行っていただけますか？
あなたのメールアドレスを、Let's Encryptプロジェクトの創設パートナーであり
Let's Encryptプロジェクトの創設パートナーであり、Certbotを開発する非営利団体である
Certbotを開発する非営利団体です。電子フロンティア財団は、Let's Encrypt プロジェクトの創立パートナーであり、Certbot を開発する非営利団体です、
EFF のニュース、キャンペーン、およびデジタル フリーダムを支援する方法についてお知らせします。 **n**
--------------------------------------------------------------

Select the appropriate numbers separated by commas and/or spaces, or leave input
blank to select all options shown (Enter 'c' to cancel):

カンマまたはスペースで区切られた適切な数字を選択するか、または入力を空白のままにする。
を空白にして、表示されているすべてのオプションを選択する（キャンセルするには「c」を入力する）： **空白**

--------------------------------------------------------------
```

※途中で失敗したけどsudo certbot -vで再開できた

[**メモ\_XserverVPSで無料SSL(Let's Encrypt)を設定する方法**](https://kiras7.hatenablog.com/entry/2023/12/08/135352)

#### 自動更新の検証

Let’s Encryptの証明書は90日間のみ有効なので証明書の更新プロセスを自動化してるか検証

```bash
# ステータス確認
sudo systemctl status certbot.timer

# Active: active (waiting)とかって出たたらOK

# ドライラン
sudo certbot renew --dry-run

# エラーがなければ設定完了
```

※sshの設定ってnginxのconfに追加とかしなくてもいいの？って思ったらcertbotが勝手にいい感じに書き換えてくれてた

鍵は `/etc/letsencrypt/live/ドメイン/fullchain.pem` とかで生成されてる

完了したらhttpsでアクセスできるか確認してみる。

フロントの受け皿ができたので、あとはStrapiの更新をwebhookで受け取ってビルドするworkflowの設定などをしたらOK

## その他参考にした記事とか

### 各社のVPSサーバー設定

#### xserver

[エックスサーバーVPSでDockerをインストールする](https://vps-one.site/docker-xserver-vps/)

#### カゴヤ

[KAGOYA VPSでDockerを使う方法を紹介します。【docker-composeも】](https://vps-one.site/docker-kagoya-cloud-vps/)

#### さくら

[さくらのVPSでDockerを使う方法を紹介します。](https://vps-one.site/docker-sakura-vps/)

[さくらのVPS 初めてでも分かる使い方 docker環境準備](https://techwalk.net/blog/setup-sakurano-docker/)

[さくらのVPSにsshで接続 & 初期設定](https://zenn.dev/techstart/articles/932b59ffddb8ef)

### 各社共通

[【簡単な4つの方法】UbuntuにDockerをインストールするには](https://kinsta.com/jp/blog/install-docker-ubuntu/)

[VPS契約後の初期設定](https://zenn.dev/naco/articles/95d729089804b9)

[SSHの説明とVPSでのsshの接続方法について](https://zenn.dev/yuukis234/articles/7b8bee9a799520)

### VPSサーバーのセキュリティ

[【チュートリアル】VPSを借りたらやるべき最低限のセキュリティ初期設定](https://nishinatoshiharu.com/initial-vps-security-settings/)

### Nginx

[NginxでVirtual Hostsを設定してみた](https://www.kagoya.jp/howto/cloud/webserver/vps-configuration/)

[自前のブログ作り③ ー 環境準備：Nginxをインストールして、Let's Encryptで常時SSL化する](https://beamaker.jp/article/3)

[Ubuntu✖️Nginx✖️Next.js(SSG)✖️StrapiでWEBサイトの環境を作る](https://sterfield.co.jp/blog/ubuntu-nginx-next-js-ssg-strapi-web/)

[2](https://qiita.com/ta-ke-no-bu/items/#comments)

[2](https://qiita.com/ta-ke-no-bu/items/fac00841855750d2669f/likers)

4