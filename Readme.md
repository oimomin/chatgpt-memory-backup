# ChatGPT メモリバックアップ用プロンプト

## 概要
プロンプトを ChatGPT に入力すると、メモリデータを可能な範囲でバックアップし、JSONファイルとしてダウンロードできます。


## 【推奨】プロンプト （10件ごとの処理）

## 🛠 使い方
負荷が高い処理が連続するため、新規チャットを立ち上げてから実行することを推奨します。
チャット入力欄にプロンプトを貼り付けてChatGPTに送信してください。
10件ごとにJSONファイルを生成して止まります。
未取得のメモリがある場合は「続きをお願い」してください。

````markdown
ChatGPTが覚えていること（メモリ）を**できる限り出力**してください。そのあとにJSONファイルに保存してください。   
**重要：ChatGPTが持っているメモリデータのみを取得し、それ以外の情報を補完・推測しないこと。**  

可能であれば、各メモリデータの「記録された日付」も一緒に出力してください。  
日付情報がない場合は、無理に推測せず「不明」として扱ってください。  

---

### **まず最初にやること**
1. ChatGPTが覚えているメモリを、チャットにできる限り表示する  
2. 最新のメモリの日付を取得（可能なら）  
3. 最古のメモリの日付を取得（可能なら）  
4. 取得したメモリの範囲を記録し、途中で止まったときのアンカーにする  

---

### **バックアップ手順**
1. 最新のメモリデータを最優先で取得し、JSON形式で保存する  
   - 1回の取得で最大10件までに制限する  
   - ファイル名は memory_backup_part_1.json, memory_backup_part_2.json など連番で管理する  
   - 取得したデータの件数と範囲をログとして出力する  

2. 最新データをすべて取得し終えたら、過去のメモリデータを取得する  
   - 取得順は「新しいものから古いものへ」  
   - **途中で止まった場合、どこまで取得できたかを記録する**  
     - **最後に取得したメモリの日付（可能なら）**  
     - **取得済みの件数**  
     - **保存したファイルの情報**  
   - **次回の再開時には、最後に取得したメモリの日付の次のデータから開始する**  

3. データ取得ができなくなった場合の対応
   - 新しいデータが取得できなくなった場合
     - 取得可能なデータがなくなったら、バックアップを終了するのではなく、完了条件の検証に移る
   - 取得できた範囲をログに記録し、次回の実行で続きから再開できるようにする  

4. **バックアップの完了条件（全て満たしたら完了）**
   - **取得したメモリの件数 = バックアップ済みの件数**
   - **バックアップデータに重複・抜けがない（ハッシュ照合で確認）**
   - **取得した「最新のメモリの日付」と、バックアップの最新データの日付が一致している（可能なら）**
   - **取得した「最古のメモリの日付」と、バックアップの最古データの日付が一致している（可能なら）**

5. **すべてのデータを取得できたことを確認し、完了報告をする**  

---

### **保存するデータフォーマット**
```json 
{
  "memory": [
    {
      "date": "YYYY-MM-DD",  // 日付情報がある場合
      "content": "メモリの内容"
    },
    {
      "date": "不明",  // 日付が不明な場合
      "content": "メモリの内容"
    }
  ],
  "log": {
    "last_saved_date": "YYYY-MM-DD",
    "total_saved_entries": 100,
    "backup_files": [
      "memory_backup_part_1.json",
      "memory_backup_part_2.json"
    ]
  }
}
```
 
````


## 【自動・非推奨】プロンプト

## 🛠 使い方
負荷が高い処理が連続するため、新規チャットを立ち上げてから実行することを推奨します。
チャット入力欄にプロンプトを貼り付けてChatGPTに送信してください。
データの抜け漏れが発生する可能性が高いです。

````markdown
ChatGPTが覚えているメモリデータを**できる限り出力**し、JSONファイルに保存してください。  
**重要：ChatGPTが持っているメモリデータのみを取得し、それ以外の情報を補完・推測しないこと。**  

可能であれば、各メモリデータの「記録された日付」も一緒に出力してください。  
日付情報がない場合は、無理に推測せず「不明」として扱ってください。  

---

## **まず最初にやること**
1. ChatGPTが覚えているメモリを、チャットにできる限り表示する  
2. 最古のメモリの日付を取得（可能なら）  
3. 最新のメモリの日付を取得（可能なら）  
4. 取得したメモリの範囲を記録し、途中で止まったときのアンカーにする  

---

## **バックアップ手順**
### **1. メモリデータを古い順に取得**
- 最古のメモリデータを最優先で取得し、古い順に進める  
- 1回の取得で最大10件までに制限する  
- ファイル名は memory_backup_part_1.json, memory_backup_part_2.json など連番で管理する  
- 取得したデータの件数と範囲をログとして出力する  

### **2. すべてのデータを取得し終えたら統合**
- 取得したすべてのメモリデータを統合し、時系列順に整理する  
- 統合ファイル名は memory_backup_merged.json とする  
- 統合後のファイル情報と件数をログに記録する  

### **3. 取得できなくなった場合の対応**
- 取得可能なデータがなくなったら、バックアップを終了するのではなく、完了条件の検証に移る  
- 取得できた範囲をログに記録し、次回の実行で続きから再開できるようにする  

### **4. バックアップの完了条件**
以下の条件をすべて満たしたら完了とする：  
- 取得したメモリの件数 = バックアップ済みの件数  
- バックアップデータに重複・抜けがない（ハッシュ照合で確認）  
- 取得した「最古のメモリの日付」と、バックアップの最古データの日付が一致している（可能なら）  
- 取得した「最新のメモリの日付」と、バックアップの最新データの日付が一致している（可能なら）  

### **5. すべてのデータを取得できたことを確認し、完了報告をする**  

---

## **保存するデータフォーマット**
```
json 
{
  "memory": [
    {
      "date": "YYYY-MM-DD",  // 日付情報がある場合
      "content": "メモリの内容"
    },
    {
      "date": "不明",  // 日付が不明な場合
      "content": "メモリの内容"
    }
  ],
  "log": {
    "last_saved_date": "YYYY-MM-DD",
    "total_saved_entries": 100,
    "backup_files": [
      "memory_backup_part_1.json",
      "memory_backup_part_2.json"
    ]
  }
}
```
````



## 注意点
- メモリの仕様が変更されると、動作しなくなる可能性があります。
- 生成されたデータが完璧にメモリを再現する保証はありません。
- リクエスト制限があるため、一度に大量のデータを取得しようとしないようにしてください。
