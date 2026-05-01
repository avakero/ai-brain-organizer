# 🧠 AI Brain Organizer

Notion の「🧠 AI Brain」データベースに溜まった未整理メモを、
毎朝 Claude Opus 4.7 が読み解いて Type / Tags / Priority / Due / Status を自動で整理し、
整理結果を豪華 HTML メールで通知します。

**完全無料・サーバー不要・GitHub Actions で動作。**

---

## できること

- 毎朝 7:00 JST に自動実行
- Notion の `Status=Inbox` ページを全件取得
- Claude API でメタデータを推論（Type, Tags, Priority, Due, Status）
- 推論結果を Notion に書き戻す
- 整理レポートを Gmail SMTP で送信（受信ボックスに直届き）

---

## アーキテクチャ

```
GitHub Actions（cron: 毎朝22:00 UTC = 7:00 JST）
    ↓
1. Notion API で Status=Inbox を全件取得
    ↓
2. Claude Opus 4.7 で各ページを推論
    ↓
3. Notion API でプロパティ更新
    ↓
4. SMTP（Gmail アプリパスワード）でレポートメール送信
```

---

## セットアップ手順

### STEP 1: リポジトリを作成

GitHub で新規リポジトリ `ai-brain-organizer` を作成し、このコードを Push します。

### STEP 2: Anthropic API キーを取得

1. <https://console.anthropic.com> にアクセス
2. アカウント作成 → 「API Keys」→「Create Key」
3. `sk-ant-...` から始まる文字列をコピー

### STEP 3: Notion インテグレーションを作成

1. <https://www.notion.so/profile/integrations> を開く
2. 「+ New integration」→ ワークスペースを選択
3. Type: 「Internal」を選んで作成
4. 「Configure integration」→「Internal Integration Secret」をコピー（`secret_...` または `ntn_...`）
5. **重要**: AI Brain データベースを開き、右上「...」→「Connections」→「Connect to」で作成したインテグレーションを選択

### STEP 4: Notion データベース ID を取得

AI Brain の URL から ID を抜き出します。

例: `https://www.notion.so/be95c08e7a9d43a291dcbfb99126ae10?v=...`
→ ID は `be95c08e7a9d43a291dcbfb99126ae10`

### STEP 5: Gmail アプリパスワードを取得

1. <https://myaccount.google.com/security>
2. 「2 段階認証プロセス」を有効化
3. 検索欄で「アプリパスワード」→ アプリ名「AI Brain Organizer」で作成
4. 表示された 16 文字をコピー

### STEP 6: GitHub Secrets を登録

リポジトリの「Settings」→「Secrets and variables」→「Actions」で以下を登録：

| Name | 値 |
|:---|:---|
| `ANTHROPIC_API_KEY` | `sk-ant-...` |
| `NOTION_API_KEY` | `secret_...` または `ntn_...` |
| `NOTION_DATABASE_ID` | `be95c08e7a9d43a291dcbfb99126ae10` |
| `GMAIL_USER` | `あなた@gmail.com` |
| `GMAIL_APP_PASSWORD` | 16 文字のアプリパスワード |
| `RECIPIENT_EMAIL` | 送信先（`GMAIL_USER` と同じでOK） |

オプション（「Variables」タブ）:

| Name | 値 |
|:---|:---|
| `CLAUDE_MODEL` | `claude-opus-4-7`（デフォルト） |

### STEP 7: 動作確認

1. リポジトリの「Actions」タブを開く
2. 「🧠 AI Brain Daily Organize」ワークフローを選択
3. 「Run workflow」→「Run workflow」で手動実行
4. 緑のチェック ✅ が出ればOK、Gmail を確認

翌朝 7:00 JST から自動実行されます。

---

## ローカル動作確認

```bash
# 環境変数を設定（.env ファイルに書いて source するなど）
export ANTHROPIC_API_KEY="sk-ant-..."
export NOTION_API_KEY="secret_..."
export NOTION_DATABASE_ID="..."

# 依存ライブラリをインストール
pip install -r requirements.txt

# 推論のみテスト（メール送信なし）
python scripts/test_local.py

# フル実行（メール送信あり）
export GMAIL_USER="..."
export GMAIL_APP_PASSWORD="..."
export RECIPIENT_EMAIL="..."
python src/main.py
```

---

## ファイル構成

```
ai-brain-organizer/
├── .github/workflows/
│   └── daily-organize.yml       # 毎朝の自動実行
├── scripts/
│   └── test_local.py            # ローカルテスト用
├── src/
│   ├── main.py                  # エントリポイント
│   ├── notion_brain.py          # Notion API ラッパー
│   ├── ai_organizer.py          # Claude API で推論
│   ├── email_sender.py          # Gmail SMTP 送信
│   └── html_template.py         # 豪華 HTML メール
├── requirements.txt
└── README.md
```

---

## カスタマイズ

### 実行時刻を変更する

`.github/workflows/daily-organize.yml` の cron を編集：

```yaml
schedule:
  - cron: '0 22 * * *'  # 22:00 UTC = 7:00 JST
```

JST = UTC + 9 なので、JST 8:00 にしたければ `0 23 * * *`、JST 6:00 にしたければ `0 21 * * *`。

### Type の選択肢を追加・変更する

`src/ai_organizer.py` の `SYSTEM_PROMPT` を編集して Type 選択肢を更新し、
Notion 側の Type プロパティの選択肢も合わせて変更してください。

---

## トラブルシューティング

### `Notion 取得エラー: object_not_found`

→ STEP 3 の「インテグレーションをデータベースに接続」を忘れている可能性大。
   AI Brain ページ右上「...」→「Connections」で確認。

### `推論エラー: ...`

→ Claude API キーや quota を確認。`max_tokens` を上げる必要がある場合は
   `src/ai_organizer.py` の値を調整。

### メールが届かない

→ Gmail のアプリパスワードを再発行。または「迷惑メール」フォルダを確認。

---

## ライセンス

MIT License

---

*Powered by [Claude API](https://anthropic.com) / [Notion API](https://developers.notion.com)*
