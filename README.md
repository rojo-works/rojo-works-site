# rojo-works-site

路 R O J O 上 (rojo.works) の仮置きサイト。

- **公開URL**: <https://rojo.works/>
- **構成**: 静的HTML（ビルド不要）
- **ホスティング**: [Cloudflare Pages](https://pages.cloudflare.com/)

## ファイル構成

```text
.
├── index.html                       # シングルページ本体
├── assets/
│   ├── logo.svg                     # 跡 ROJO ロゴ（viewBox最適化済み）
│   └── fonts/
│       └── line-seed-jp-regular.woff2  # LINE Seed JP（subset・約30KB）
├── .gitignore
└── README.md
```

## 編集と公開

```bash
git clone https://github.com/rojo-works/rojo-works-site.git
cd rojo-works-site
# index.html を編集
git add .
git commit -m "更新内容"
git push
```

push後、Cloudflare Pages が自動でビルドして数十秒で本番反映される。

## フォントの再生成（文言変更時）

`assets/fonts/line-seed-jp-regular.woff2` は **使用文字だけを含むsubset**。
index.html で新しい日本語・記号を追加したら、subsetを再生成する必要がある。

```bash
# 必要文字を全部含めたテキストを作る
TEXT="路上ROJOrojoworks ... (使用する全文字を列挙) ..."

# Google Fonts API で subset CSS を取得
ENCODED=$(python3 -c "import urllib.parse, sys; print(urllib.parse.quote(sys.argv[1]))" "$TEXT")
curl -s -H "User-Agent: Mozilla/5.0" \
  "https://fonts.googleapis.com/css2?family=LINE+Seed+JP:wght@400&text=${ENCODED}&display=swap" \
  > /tmp/font.css

# CSS から woff2 URL を抽出してダウンロード
WOFF2_URL=$(grep -oE 'https://fonts.gstatic.com/[^)]+' /tmp/font.css)
curl -sL -o assets/fonts/line-seed-jp-regular.woff2 "$WOFF2_URL"
```

## デザイン仕様

| 項目 | 値 |
| ---- | -- |
| 背景色 | `#f0f0f0` |
| 文字色 | `#2b2b2b` |
| 補助色 | `#6b6b6b`（フッター） |
| フォント | LINE Seed JP（Apache License 2.0 / セルフホスト） |
| 文字色（ロゴ） | `#231815`（オリジナル） |
| 最大幅 | 480px |

## プライバシー方針

- Cookie 不使用
- アクセス解析なし
- 外部トラッキング・CDN なし
- フォント・画像はすべて同一オリジン配信

## ライセンス

- LINE Seed JP — [Apache License 2.0](https://github.com/line/LINESeedJP)
- ロゴ・コンテンツ — © 路上 / rojo.works
