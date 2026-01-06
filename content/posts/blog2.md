+++
title = "Arch Linux と Sway WM を使ったデスクトップ環境構築ガイド（書きかけ）"
author = ["Yuichi Tsunoda"]
date = 2025-12-29T00:00:00+09:00
draft = false
+++

## はじめに {#はじめに}


### Sway WM について {#sway-wm-について}

[Swayウィンドウマネージャー](https://ja.wikipedia.org/wiki/Sway_(%E3%82%A6%E3%82%A3%E3%83%B3%E3%83%89%E3%82%A6%E3%83%9E%E3%83%8D%E3%83%BC%E3%82%B8%E3%83%A3))（以下、Sway）は、[Wayland](https://ja.wikipedia.org/wiki/Wayland)で動作する[タイル型ウィンドウマネージャー](https://ja.wikipedia.org/wiki/%E3%82%BF%E3%82%A4%E3%83%AB%E5%9E%8B%E3%82%A6%E3%82%A3%E3%83%B3%E3%83%89%E3%82%A6%E3%83%9E%E3%83%8D%E3%83%BC%E3%82%B8%E3%83%A3)です。
僕が Linux を使うモチベーションの一つが「タイル型ウィンドウマネージャーが使えること」なのですが、Wayland 環境で使えるタイル型ウィンドウマネージャーは結構限られています（Sway 以外だと Hyprland くらい？）。
僕はずーっと Sway を使っているのですが、Sway はかなり動作が安定しているし、ドキュメントも割と充実しているし、Arch Linux であれば日本語入力も問題なく使えるので、タイル型ウィンドウマネージャー初心者の方にも非常にオススメです。
ここでは、Arch Linux 上での Sway を使ったデスクトップ環境の構築方法を紹介します。


#### 僕のデスクトップ環境 {#僕のデスクトップ環境}

![](/assets/img/sway-desktop.gif)
見た目よりも視認性・操作性・メンテナンス性を重視して、シンプルなデザインにしています。ビジュアル重視の方は、[Hyprland](https://hypr.land/)を試してみると良いかもしれません。


## Sway のインストール {#sway-のインストール}

> Arch Linux のインストール方法については[こちらの動画（英語）](https://www.youtube.com/watch?v=FxeriGuJKTM)や、[Arch Wiki（日本語）](https://wiki.archlinux.jp/index.php/%E3%82%A4%E3%83%B3%E3%82%B9%E3%83%88%E3%83%BC%E3%83%AB%E3%82%AC%E3%82%A4%E3%83%89)、[こちらのドキュメント（日本語）](https://zenn.dev/ytjvdcm/articles/0efb9112468de3)を参照して下さい。
> 初心者の方は[archinstall](https://wiki.archlinux.jp/index.php/Archinstall)というヘルパーライブラリを使うことをおすすめします。


### archinstall でインストールする場合 {#archinstall-でインストールする場合}

[archinstall](https://wiki.archlinux.org/title/Archinstall)を使うと、Arch Linux のインストールと、Sway を含むデスクトップ環境を簡単にインストールできます。その際、Desktop Environment の項目で「Sway」を選択してください。
その他の項目のオススメの設定も一応載せておきます。

-   Mirror: Japan
-   Disk configuration → Partitioning
    -   Disk configuration type: Use a best-effort default partition layout
    -   File system: ext4
    -   Create a separate partition for /home: Yes
-   Profile
    -   Type
        -   Type: Desktop
        -   Desktop environment: GNOME, Sway
        -   Seat access: polkit
    -   Greeter: GDM
-   Audio: pipewire
-   Kernel: linux &amp; linux-lts
-   Network configuration: Use NetworkManager
-   Time zone: Japan


### 手動でインストールする場合 {#手動でインストールする場合}

```bash
sudo pacman -S sway swaybg swaylock swayidle
```

-   waybar
-   wofi
-   mako
-   brightnessctl
-   wl-clipboard


## 日本語入力とフォントの設定 {#日本語入力とフォントの設定}


### 日本語ロケールの設定 {#日本語ロケールの設定}

Sway 環境で日本語を使うためには、まずシステム全体で **日本語ロケール（ja_JP.UTF-8）** を有効にする必要があります。
ロケールが定義されているファイルは `/etc/locale.gen` です。このファイルを開いて、=ja_JP.UTF-8 UTF-8= の行の先頭にある `#` を取り除いて日本語ロケールを有効にします。
次に、以下のコマンドを実行して `/etc/locale.conf` で有効になっているロケールをシステムに反映します。

```bash
sudo locale-gen
```


### 日本語の入力メソッドの設定 {#日本語の入力メソッドの設定}

日本語入力には[fcitx5](https://wiki.archlinux.jp/index.php/Fcitx5)という入力メソッドフレームワークを使います。以下のコマンドで必要なパッケージをインストールします。

```bash
sudo pacman -S fcitx5-im fcitx5-configtool fcitx5-mozc
```

次に、=/etc/environment=ファイルを編集して、fcitx5 を有効にします。以下の内容を追加してください。

```sh
GTK_IM_MODULE=fcitx
QT_IM_MODULE=fcitx
XMODIFIERS=@im=fcitx
```

そして、=fcitx5-configtool=を起動して、Available Input Method から Mozc を選択し、Current Input Method に追加されているのを確認し、Apply をクリックして下さい。
最後に、ログアウトして再ログインすることで、fcitx5 が有効になります。


### フォントの設定 {#フォントの設定}

フォントは[Noto Sans CJK JP](https://fonts.google.com/noto/specimen/Noto+Sans+JP)を使うのがオススメです。以下のコマンドでインストールできます。

```bash
sudo pacman -S noto noto-fonts-cjk noto-fonts-emoji noto-fonts-extra
```


## Sway の設定 {#sway-の設定}

Sway の設定ファイルは、=~/.config/sway/config= にあります。
