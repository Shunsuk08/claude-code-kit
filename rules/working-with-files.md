# 外部ファイル受領時のワークフロー

取引先・主催者からファイル（Excel、PDF、Word、画像等）を渡されて編集する場合の標準ワークフロー。

## 基本方針

**Desktop / Downloads / Documents で直接編集しない。** プロジェクト配下にタスク用フォルダを作り、コピーしてから編集する（誤削除・誤上書き防止、原本バックアップ、gitで履歴管理できる）。

## 標準フロー

1. **タスク用フォルダを作る**（`tasks/{カテゴリ}/{案件名}/` のような配下）
2. **元ファイルをそのフォルダにコピー**（`original.*` として原本を残す）
3. **コピーを編集**（`edited.*` → 提出版は `final.*`。Desktop の元ファイルは触らない）
4. **編集後に保存先パスを明示して案内**

## 機密情報の扱い

| 機密度 | 例 | 対処 |
|--------|---|------|
| 公開可 | 会社住所・代表者名（Web掲載済み） | コミットOK |
| 個人情報 | 生年月日・口座情報等 | `.gitignore` で除外 |
| 取引先・顧客情報 | 顧客名・契約条件等 | 案件ごとに判断、迷ったら除外 |

迷ったら `.gitignore` 除外を選ぶ（必要なら後で `git add -f`）。

## ZIP展開は `ditto` を使う（日本語ファイル名の文字化け防止）

macOS標準の `unzip` はZIP内のUTF-8ファイル名を壊すことがある。

```bash
ditto -V -x -k foo.zip dest/
```

展開後は `ls dest/ | head -3` でファイル名が日本語のまま見えるか確認する（`?????` なら失敗）。

## ファイル名・パスの罠

- シングルクォート U+2019（`'`）→ Bash・Pythonリテラル直書きで失敗する
- 全角括弧【】・全角スペース
- Excel一時ロックファイル `~$<filename>.xlsx` → find/globで除外する
- **macOSのファイル名はNFD正規化**（濁点・半濁点が分解されている）。ファイル名をコピペしたMarkdown行は、見た目が同じでもEdit/apply_patchの完全一致置換にマッチしないことがある。ファイル名を含まない近傍行をアンカーにするか、行番号ベースで編集する。

```bash
SRC=$(ls ~/Desktop/*<キーワード>*.xlsx 2>/dev/null | grep -v '~\$' | head -1)
```

## PDFはテキストレイヤー抽出を最優先

外部から受け取ったPDFは、マルチモーダル画像読み取りの前に必ずテキストレイヤー抽出を試す。Excel/Word/PowerPoint由来のPDFはほぼ確実にテキストレイヤーが残っており、画像読みは数字・固有名詞の誤読リスクが高い。

```bash
pdftotext -layout /path/to/file.pdf - | head -40
```

- poppler（`pdftotext`）/ mupdf（`mutool`）はHomebrewで導入可能。座標付き構造は `mutool draw -F stext`、画像化は `pdftoppm -png -r 150`。
- Pythonライブラリ（PyMuPDF等）が必要な場合、system python3に入っている前提で書かない。`pip install` がPEP 668で拒否される環境では `uv run --with <パッケージ> python - <<'PY' ... PY` で一時環境を作る。
- **スキャンPDF**（手書き赤入れ等、テキストレイヤーが無いもの）は `mutool draw -r 150 -o p%02d.png <file>.pdf` で全ページPNG化してReadで画像読解する。

## アンチパターン

- ❌ Desktop / Downloadsの元ファイルを直接編集（バックアップなし）
- ❌ 機密情報ありのファイルを `.gitignore` 未確認のままpush
- ❌ ファイルパスをPythonリテラルに直書き（U+2019でSyntaxError）
- ❌ Excel一時ロックファイル `~$*.xlsx` をコピー（中身は空）
- ❌ 数字・固有名詞を含むPDFを画像読み取りだけで断定
