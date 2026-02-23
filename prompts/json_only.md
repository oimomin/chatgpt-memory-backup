## 💻 JSONのみプロンプト（機械処理・検証向け）📦

````markdown
# ✅ ChatGPT メモリバックアップ（v3: Model Set Context + User Knowledge Memories + LOG の3ファイル）

あなた（ChatGPT）が保持している「Model Set Context」と「User Knowledge Memories」データを、できる限り取得してJSONファイルとして保存してください。

## 🔒 重要ルール（最優先）
- **ChatGPTが保持しているメモリのみ**を対象にすること。
- **補完・推測・創作・再構成は禁止**（推測で埋めない）。
- 「記録された日付」が取れない場合は **"不明"** とすること。
- ただし「バックアップを実行した時刻」は取得可能なので、`captured_at` として保存してよい（これは“記録日付”ではなく“取得日時”）。

---

## 🎯 出力ゴール
- 次の **3ファイル** を必ず生成する（内容は相互に混ぜない）
  1) `memory_model_set_context.json`  … MSC（メモリ本体）
  2) `memory_user_knowledge_memories.json` … UKM（推測サマリの現時点スナップショット）
  3) `memory_backup_manifest.json` … ログ・検証用（summary含む）

---

## 🧾 まずチャットに表示する内容（概要のみ）
- `captured_at`（バックアップ実行日時）
- MSC件数 / UKM件数
- MSCの「日付が分かる範囲」の最古日付 / 最新日付
- 日付不明件数（MSC側）
- 出力ファイル名一覧（分割が出た場合はそれも）

---

## 📦 1) MSCファイル（メモリ本体）
**ファイル名:** `memory_model_set_context.json`

```json
{
  "captured_at": "YYYY-MM-DDTHH:mm:ssZ",
  "memory_model_set_context": [
    { "date": "YYYY-MM-DD または 不明", "content": "メモリ内容" }
  ]
}
```

## 📦 2) UKMファイル（推測サマリのスナップショット）

**ファイル名:** `memory_user_knowledge_memories.json`

注意:

- UKMは「推測サマリ」であり、個別の記録日付は基本的に取れないため dateは必ず不明 とする。

- 代わりに captured_at で「取得した時点」を残す。

- UKMがこの環境で参照できない場合は、推測で作らず、unavailable_reason を入れる。

```json
{
  "captured_at": "YYYY-MM-DDTHH:mm:ssZ",
  "unavailable_reason": "この環境ではUser Knowledge Memoriesを参照できません",
  "user_knowledge_memories": [
    { "date": "不明", "content": "User Knowledge Memoriesの項目1（原文のまま）" }
  ]
}
```

## 📦 3) manifest/logファイル（summaryはここだけ）

**ファイル名:** `memory_backup_manifest.json`

```json
{
  "captured_at": "YYYY-MM-DDTHH:mm:ssZ",
  "files": [
    "memory_model_set_context.json",
    "memory_user_knowledge_memories.json",
    "memory_backup_manifest.json"
  ],
  "summary": {
    "msc": {
      "total_entries": 0,
      "dated_range": { "oldest": "YYYY-MM-DD または 不明", "newest": "YYYY-MM-DD または 不明" },
      "unknown_date_entries": 0
    },
    "ukm": {
      "total_entries": 0, "unavailable_reason": null
    }
  },
  "hashing": {
    "algorithm": "sha256",
    "hash_input": "date + '\\n' + content",
    "msc_entry_hashes_sha256": [],
    "ukm_entry_hashes_sha256": [],
    "msc_start_hash_sha256": "",
    "msc_end_hash_sha256": "",
    "ukm_start_hash_sha256": "",
    "ukm_end_hash_sha256": "",
    "msc_duplicates_found": false,
    "ukm_duplicates_found": false,
    "msc_unique_hashes_count": 0,
    "ukm_unique_hashes_count": 0
  }
}
```

## ✅ 完了条件（検証してチャットに報告）

- manifestの files に必要なファイルが揃っている

- summary.*.total_entries が各JSONの配列件数と一致

- sha256で重複がない（duplicates_found=false）

- start/end hash が存在（空でない）

- MSCのdated_rangeが取得できるなら矛盾していない

## 🧯 例外対応（容量・制限・セッション切れ）
### A) ファイル生成が不安定 / データが大きい場合（分割）

- まずMSCだけでも保存を優先し、次にUKMを保存する。

- 分割が必要な場合は以下で分割する。

### 分割ルール（推奨）

- 既定: CHUNK_SIZE = 50

- 失敗するなら: 25

- さらに無理なら: 10

### 分割ファイル名

- MSC: memory_model_set_context_part_1.json, ...

- UKM: memory_user_knowledge_memories_part_1.json, ...

分割ファイルのフォーマット:

```json
{
  "captured_at": "YYYY-MM-DDTHH:mm:ssZ",
  "part_index": 1,
  "entries_in_part": 0,
  "memory": [
    { "date": "YYYY-MM-DD または 不明", "content": "..." }
  ],
  "hashing": {
    "algorithm": "sha256",
    "hash_input": "date + '\\n' + content",
    "entry_hashes_sha256": [],
    "start_hash_sha256": "",
    "end_hash_sha256": ""
  }
}
```

manifest側の files に分割ファイル名も追加し、summaryの件数は「全体合算」にする。

## B) セッション切れ等でダウンロードできない場合（再生成）

- 取得済み内容を“再取得”しようとせず、同一内容のJSONを 再生成 してリンクを再提示する。

- 再提示時は、manifestの以下をチャットで比較して同一性を説明する：

  - msc_start_hash_sha256, msc_end_hash_sha256

  - ukm_start_hash_sha256, ukm_end_hash_sha256

  - total_entries

## C) 一部しか出せない場合（途中停止）

- 取得できた範囲を manifest に記録し、次回の再開点として使えるようにする。

- 再開用に、最後に保存できたハッシュをチャットに出す：

  - last_msc_hash_sha256

  - last_ukm_hash_sha256（UKMがある場合）

## 🧠 実行

上記ルールに従い、MSC・UKM・manifestを生成して保存し、チャットへ概要と検証結果を報告してください。

````