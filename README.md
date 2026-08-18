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
│       ├── line-seed-jp-regular.woff2  # LINE Seed JP（subset・約21KB）
│       └── OFL.txt                     # 上のフォントのライセンス全文
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
# 1. index.html の表示テキストから使用文字を抜き出す
python3 - <<'PYEOF'
import re, pathlib
html = pathlib.Path('index.html').read_text(encoding='utf-8')
s = re.sub(r'<!--.*?-->', '', html, flags=re.S)
s = re.sub(r'<(style|script)\b.*?</\1>', '', s, flags=re.S)
attrs = ' '.join(re.findall(r'\b(?:alt|title)="([^"]*)"', s))
text = re.sub(r'<[^>]+>', ' ', s) + ' ' + attrs
chars = sorted({c for c in text if c.strip() and ord(c) > 31})
pathlib.Path('/tmp/chars.txt').write_text(''.join(chars), encoding='utf-8')
print('文字数:', len(chars))
PYEOF

# 2. Google Fonts API で subset CSS を取得する
#    User-Agent は必ず今の Chrome のものを渡す。短い UA だと woff2 ではなく TTF が返る
UA="Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36"
ENCODED=$(python3 -c "import urllib.parse,pathlib;print(urllib.parse.quote(pathlib.Path('/tmp/chars.txt').read_text(encoding='utf-8')))")
curl -s -H "User-Agent: $UA" \
  "https://fonts.googleapis.com/css2?family=LINE+Seed+JP:wght@400&text=${ENCODED}&display=swap" \
  -o /tmp/font.css

# 3. 全文字がカバーされたか照合してから woff2 の URL を取り出す
python3 - <<'PYEOF'
import re, pathlib
css = pathlib.Path('/tmp/font.css').read_text(encoding='utf-8')
cov = set()
for r in re.findall(r'unicode-range:\s*([^;]+);', css):
    for part in r.split(','):
        part = part.strip().lower().replace('u+','')
        if '-' in part:
            a, b = part.split('-'); cov.update(range(int(a,16), int(b,16)+1))
        else:
            cov.add(int(part,16))
want = set(map(ord, pathlib.Path('/tmp/chars.txt').read_text(encoding='utf-8')))
miss = sorted(want - cov)
assert 'woff2' in css, 'woff2 が返っていない（UA を疑う）'
assert not miss, 'カバー漏れ: ' + ''.join(map(chr, miss))
print('カバー漏れ 0')
pathlib.Path('/tmp/font-url.txt').write_text(
    re.findall(r'url\((https://fonts\.gstatic\.com/[^)]+)\)', css)[0], encoding='utf-8')
PYEOF

# 4. woff2 として妥当か確かめてから差し替える（TTF を掴んだまま上書きしないため）
curl -sL -o /tmp/new-font.woff2 "$(cat /tmp/font-url.txt)"
python3 - <<'PYEOF'
import pathlib, shutil
b = pathlib.Path('/tmp/new-font.woff2').read_bytes()
assert b[:4] == b'wOF2', 'woff2 ではない'
shutil.copyfile('/tmp/new-font.woff2', 'assets/fonts/line-seed-jp-regular.woff2')
print('差し替え完了', len(b), 'bytes')
PYEOF
```

## デザイン仕様

| 項目 | 値 |
| ---- | -- |
| 背景色 | `#f0f0f0` |
| 文字色 | `#2b2b2b` |
| 補助色 | `#6b6b6b`（フッター） |
| フォント | LINE Seed JP（SIL Open Font License 1.1 / セルフホスト） |
| 文字色（ロゴ） | `#231815`（オリジナル） |
| 最大幅 | 480px |

## プライバシー方針

- Cookie 不使用
- アクセス解析なし
- 外部トラッキング・CDN なし
- フォント・画像はすべて同一オリジン配信

## ライセンス

- LINE Seed JP — © LY Corporation、[SIL Open Font License 1.1](https://openfontlicense.org/)。全文は `assets/fonts/OFL.txt`（配布元は [google/fonts の ofl/lineseedjp](https://github.com/google/fonts/tree/main/ofl/lineseedjp)）
- ロゴ・コンテンツ — © 路上 / rojo.works
