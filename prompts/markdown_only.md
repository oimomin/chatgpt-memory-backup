## 🧸🫧 Markdownのみプロンプト（読む用・アルバム向け）🌙

````markdown
# ✅ ChatGPT メモリバックアップ（v3: メモリデータ + LOG の2ファイル）

あなた（ChatGPT）が保持している「Model Set Context」と「User Knowledge Memories」データを、できる限り取得してMarkdownとして保存してください。

## 🔒 重要ルール（最優先）
- **ChatGPTが保持しているメモリのみ**を対象にすること。
- **補完・推測・創作・再構成は禁止**（推測で埋めない）。
- 「記録された日付」が取れない場合は **"不明"** とすること。
- ただし「バックアップを実行した時刻」は取得可能なので、`captured_at` として保存してよい（これは“記録日付”ではなく“取得日時”）。

---

## 🎯 出力ゴール
- 次の **2ファイル** を必ず生成する（内容は相互に混ぜない）
  1) `memory_backup.md`（本文。MSC + UKM）
  2) `memory_manifest.md`（概要と検証メモ。summaryはここだけ）

---

## 🧾 まずチャットに表示する内容（概要のみ）
- `captured_at`（バックアップ実行日時）
- MSC件数 / UKM件数
- MSCの「日付が分かる範囲」の最古日付 / 最新日付（取れる範囲で）
- 日付不明件数（MSC側）
- 出力ファイル名一覧

---

# 📘 memory_backup.md（本文フォーマット）

```markdown
## captured_at
- YYYY-MM-DDTHH:mm:ssZ

## 1) Model Set Context（MSC）
> 並びは可能なら古い順。無理なら取得順のまま。

### YYYY-MM-DD
- メモリ内容（原文のまま）

### 不明
- メモリ内容（原文のまま）

## 2) User Knowledge Memories（UKM）
> UKMに記録日付が無い場合は、見出しを固定で「不明」にする。

### 不明
- UKM項目（原文のまま）

> UKMを参照できない場合は、ここに推測で書かず、次の1行だけを書く：
「この環境ではUser Knowledge Memoriesを参照できません」
```

---

```markdown
# 📗 memory_manifest.md（summaryと検証メモ）

## captured_at
- YYYY-MM-DDTHH:mm:ssZ

## files
- memory_backup.md
- memory_manifest.md
- （分割があればそのファイル名も列挙）

## summary
- MSC total: N
- MSC dated range: oldest=YYYY-MM-DD / newest=YYYY-MM-DD（不明なら不明）
- MSC unknown date: N
- UKM total: N（参照不可なら 0）
- UKM unavailable_reason: （参照不可なら理由を書く、可能なら null）

## integrity notes（軽量）
- 本文を途中で省略していないこと
- もし途中停止したら、最後に出力できたMSCの見出し（日付）と先頭数文字をメモとして残す
```

---

## 🧯 例外（本文が長すぎる）
- `memory_backup_part_1.md`, `memory_backup_part_2.md` ... に分割してよい
- 目安は 50件ごと、無理なら 25件、さらに無理なら 10件
- manifest の files に分割ファイルを全列挙し、MSC件数は合算で記載する

## 🧯 例外（セッション切れでダウンロード不可）
- 再取得で増減させず、同一内容のmdを **再生成**してリンクを再提示する
- manifestの summary（件数とdated range）を比較して同一性を説明する

## 🧠 実行

上記ルールに従い、memory_backup・manifestを生成して保存し、チャットへ概要と検証結果を報告してください。

````