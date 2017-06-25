Vagrantfiles for Boxes
======================

## 概要

各種仮想環境の Vagrant Box （以下、Box）を作るための Vagrantfile を格納している.
実際に利用する仮想環境の Vagrantfile ではなく、それらのベースとなる Box の作成レシピ集である.

[Atlas](https://atlas.hashicorp.com/boxes/search) で公開されている Box は導入されてるパッケージや設定が必要最小限に留めてある.
それらに自身で利用頻度の高いパッケージや設定を追加した Box の作成やインストールを自動化し、環境の準備時間を短縮するのが目的である.

## 前提条件

以下の環境での利用を想定している.

 - Vagrant 1.9.x
 - 作業, ホストOS: Linux
 - プロバイダ: Oracle VM VirtualBox 5.x
 - その他必要なソフトウェア
   - GNU Make 4.x


## ディレクトリ構成

作成する Box によって構築可能な仮想環境ごとにディレクトリを用意している.
各ディレクリには後述する命名規則に従って、環境を内容を示す名前が付いている.

各ディレクトリには Vagrantfile の他に Box をビルド、ローカルの Vagrant 環境へインストールするための Makefile も格納されている.

```
./<仮想環境を示す名前>
./<仮想環境を示す名前>/Vagrantfile
./<仮想環境を示す名前>/Makefile  # ../mkfiles/Makefile のシンボリックリンク
./mkfiles
./mkfiles/Makefile  # 各環境の Makefile 本体
```

<仮想環境を示す名前>は以下の命名規則に従っている.

 - <仮想環境を示す名前> ::= <ディストリビューション情報> . <ロケール設定> . <キーマップ> . <X11の有無> [. <特定用途>] .<アーキテクチャ>`
 - <ディストリビューション情報> ::= <ディストリビューション名> [_ <メジャーバージョン>] [_ <コードネーム>]
 - <ロケール設定> ::= <ISO 639 言語コード> [_ <ISO 3166 国名コード（大文字）>] [. <文字コーディング名>]
 - <キーマップ> ::= <ISO 3166 国名コード（小文字）> <キー数>
 - <X11の有無> ::= no_x11 | x11 [_<ウィンドウマネージャ情報>]
 - <アーキテクチャ> ::= i386 | amd64 | arm

`<特定用途>` とは Web サーバや LDAP サーバ用途など、特定用途のための環境を示す.
例えば、Web サーバの場合は、httpd2.4 (Apache HTTP Server 2.4) などとなる.

## 利用方法

ある環境を Box ファイルとして自身の Vagrant 環境にインストールしたい場合、目的の環境のディレクトリに移動して、`make all` をすれば良い.

```
$ whoami
user
$ cd ./centos7.ja_JP.UTF-8.jp106.no_x11.amd64
$ vagrant all
Installing the 'vagrant-vbguest' plugin. This can take a few minutes...
Installed the plugin 'vagrant-vbguest (0.14.2)'!
Bringing machine 'default' up with 'virtualbox' provider...
==> default: Checking if box 'centos/7' is up to date...
... (中略) ...
==> box: Successfully added box 'user::centos7.ja_JP.UTF-8.jp106.no_x11.amd64' (v0) for 'virtualbox'!
$ vagrant box list
user::centos7.ja_JP.UTF-8.jp106.no_x11.amd64 (virtualbox, 0)
```

以上のように、Vagrant 環境に目的の環境の Box ファイルが導入された. 

Makefile では、他に以下の各種ターゲットを用意している.

| ターゲット | 処理内容 |
|---|---|
| `build` | Vagrantfile を元に成果物（Box ファイル）を作成する. |
| `install` | `build` ターゲットで作成した Box ファイルを Vagrant 環境にインストールする. |
| `uninstall` | `install` ターゲットでインストールされた Box ファイルを Vagrant 環境から削除する. |
| `uninstall-basebox` | Vagrantfile でベースに指定された Box ファイルを Vagrant 環境から削除する. |
| `clean` | 成果物を削除する. |
| `all` | `build`, `install` ターゲットの順にターゲットを実行する. |
| `help` | ヘルプを表示する.デフォルトターゲット. |

作成した Box ファイルを登録時の名前は `$(whoami)::<Vagrantfile が格納格納されているディレクトリ名>` という名前になる.

例えば、ユーザ `foo` で `centos7.ja_JP.UTF-8.jp106.no_x11.amd64` ディレクトリにて `make all` を実行したとする.
すると、build ディレクトリ以下に Box ファイルが作成され、 Vagrant 環境には `foo::centos7.ja_JP.UTF-8.jp106.no_x11.amd64` という名前でその環境の Box ファイルが登録される.
