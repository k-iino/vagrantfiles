# -*- coding: utf-8 -*-
# ベースとなる Vagrant Box 用 Makefile
#
# Kyota IINO <k.iino@outlook.com>

#
# Vagrant の設定
#

# Plugin
VAGRANT_PLUGINS := vagrant-vbguest

#
# Box ファイルについての設定
#

# 成果物を置くディレクトリ
BUILD_DIR := build
# 出力するときの Box ファイル名
BOX_FILE := package.box
# インストールする際の Box 名
BOX_NAME := $(shell whoami)::$(notdir $(CURDIR))
# Vagrant ファイルで利用するベース Box ファイル
BASE_BOX = `cat Vagrantfile | grep 'config.vm.box = .\+' | sed -e 's/.\+"\(.\+\)"/\1/g'`

#
# Vagrant ターゲット
#

.PHONY: clean install-plugins up install-vbguest package install-box

all: clean install-plugins up install-vbguest package install-box  ## ターゲットを順番に実行し、Box ファイルの作成とインストールを行う

clean:  ## 成果物を削除する.
	-@vagrant halt
	-@vagrant destroy -f
	-@rm -vrf $(BUILD_DIR)

install-plugins:  ## プラグインのインストール
	@vagrant plugin install $(VAGRANT_PLUGINS)

up:  ## 仮想環境を起動する
	@vagrant up

# vagrant package 実行前に再度 Guest Additions のリビルドとインストールをするために定義.
# Guest Additions をインストールする際に、X Window System が存在しない場合、
# X 関係の拡張がカーネルに組み込まないようだ.
# そのため、すでに Guest Additions が入っている環境に、
# X Window System をインストールしても X 関係の Guest Additions 環境は使えない.
# それを防ぐため、Box を作成する前には vbguest プラグインで Guest Additions を
# 再インストールしている.

install-vbguest: install-plugins  ## VirtualBox Guest Additions をインストールする. 既にインストールされている場合は再度ビルドとインストールする.
	@vagrant up
	@vagrant vbguest --do rebuild --auto-reboot
	@vagrant vbguest --do install --auto-reboot
	@vagrant halt

package:  ## 成果物を作成する.
	@mkdir -p $(BUILD_DIR)
	@vagrant package --output $(BUILD_DIR)/$(BOX_FILE)

install-box:  ## 作成した Box ファイルをローカルの Vagrant 環境にインストールする.
	@vagrant box add --name $(BOX_NAME) $(BUILD_DIR)/$(BOX_FILE)

.PHONY: uninstall uinstall-basebox

uninstall-box:  ## インストールされた成果物をローカルの Vagrant 環境から削除する.
	@vagrant box remove $(BOX_NAME)

uninstall-basebox:  ## Vagrantfile 内でベースに指定された Box ファイルを Vagrant 環境から削除する.
	@vagrant box remove $(BASE_BOX)

#
# Help
#
ifeq (, $(help_define))
.DEFAULT_GOAL := help
.PHONY: help

help:
	@for mkfile in $(MAKEFILE_LIST); do \
	    grep -E '^[ 0-9a-zA-Z_-]+:.*?## .*$$' $$mkfile \
            | sort \
	    | awk 'BEGIN {FS = ":.*?## "}; {printf "\033[36m%-30s\033[0m %s\n", $$1, $$2}'; \
	done

help_define := true
endif
