+++
title = "My Blog"
author = ["Yuichi Tsunoda"]
draft = false
+++

## Hugo + ox-hugo の導入メモ {#setup-ox-hugo}

この記事は、=ox-hugo= を使って Org から Hugo の記事を生成する手順のメモです。


### 目的 {#目的}

-   Org で原稿を書く
-   1 見出し = 1 記事として Markdown を生成する
-   Hugo の `content/posts/` に自動出力する


### 構成（推奨） {#構成-推奨}

```text
~/sites/
├── blog-org/
│   └── blog.org
└── username.github.io/
    ├── content/
    ├── static/
    ├── themes/
    └── hugo.toml
```


### エクスポート {#エクスポート}

-   ファイル全体: `C-c C-e H H`
-   見出し単位: `C-c C-e H h`


### コード例 {#コード例}

```bash
cd ~/sites/username.github.io
hugo server -D
```


### 画像 {#画像}

画像は `static/images/` に置くと管理が簡単です。

例：

-   画像ファイル: `static/images/sample.png`
-   Org 本文: `[[/images/sample.png]]`

---


## サンプル記事：技術メモの書き方 {#writing-style}

これは下書き（draft）です。=hugo server -D= で表示されます。


### 見出しの粒度 {#見出しの粒度}

-   大見出しは「結論」
-   中見出しは「理由」「手順」「注意点」
-   小見出しは「具体例」


### よくある注意 {#よくある注意}

-   `#+hugo_base_dir` は Hugo ルートを指す（=content/= ではない）
-   `EXPORT_FILE_NAME` が重複すると上書きされる
