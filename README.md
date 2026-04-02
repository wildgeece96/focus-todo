# Bout

macOS 向けのミニマルなポモドーロタイマーアプリ。常に最前面に表示されるフローティングウィンドウで、集中と休憩のリズムを視覚的にサポートします。

## Features

- **ポモドーロタイマー** — Work (25分) / Deep Work (50分) / Short Break (5分) / Long Break (15分)
- **フローティングウィンドウ** — 常に最前面に表示。実行中はコンパクト表示に切り替わる
- **スクリーンボーダー** — 画面端に色付きグラデーションで現在のフェーズを表示（集中=赤、休憩=青）
- **URL Scheme** — `bout://` で外部ツールから制御可能
- **CLI** — ターミナルから操作・状態確認
- **MCP Server** — Claude Code 等の AI エージェントから直接操作
- **メッセージ機能** — アプリ上にポップアップメッセージを送信

## Installation

1. [Releases](https://github.com/wildgeece96/bout/releases) から最新の zip をダウンロード
2. 解凍して `Bout.app` を `/Applications` に移動
3. 初回起動時に「開発元が未確認」と表示されたら → 右クリック → 「開く」

### アンインストール

`/Applications/Bout.app` を削除してください。ユーザーデータは `~/.bout/` に保存されています。不要な場合はこちらも削除してください。

## Requirements

- macOS 14 (Sonoma) 以降
- Apple Silicon (arm64)

## Usage

### アプリ操作

起動すると画面上にフローティングウィンドウが表示されます。

| 状態 | 表示 |
|------|------|
| 停止中 | フルサイズ (280×380) でタスク入力・フェーズ選択が可能 |
| 実行中 | コンパクト表示 (140×140) にタイマーリングと残り時間を表示 |
| 実行中 + ホバー | フルサイズに展開してコントロールを表示 |

フェーズは自動で切り替わります。4回目の Work 完了後は Long Break になります。

### CLI

```bash
# タイマー操作
bout start "レポート作成" -p work       # タスク名を指定して開始
bout start "設計作業" -p deep_work      # Deep Work (50分) で開始
bout pause                              # 一時停止
bout reset                              # リセット
bout skip                               # 次のフェーズへスキップ
bout update-task "コードレビュー"       # タスク名を変更

# 状態確認
bout status                             # 現在のタイマー状態を表示
bout messages                           # メッセージキューを表示
bout list-sessions                      # 完了したセッション一覧
bout list-sessions --date 2026-04-01    # 日付指定
bout get-summary                        # 今日のポモドーロ統計
bout get-summary --date 2026-04-01      # 日付指定

# メッセージ送信
bout send "スタンドアップの時間です" --sender system
```

### URL Scheme

外部ツール（Shortcuts、Alfred、Raycast 等）から `bout://` で制御できます。

```bash
# タイマー操作
open "bout://start?task=Write%20report&phase=work"
open "bout://pause"
open "bout://reset"
open "bout://skip"

# タスク更新
open "bout://update-task?task=Code%20review"

# メッセージ送信
open "bout://send-message?text=Hello&sender=ai"
```

| Action | Parameters | Description |
|--------|-----------|-------------|
| `start` | `task`, `phase` | タイマー開始 |
| `pause` | — | 一時停止 |
| `reset` | — | リセット |
| `skip` | — | 次のフェーズへ |
| `update-task` | `task` | タスク名変更 |
| `send-message` | `text`, `sender` | メッセージ送信 |

### MCP Server（AI エージェント連携）

Claude Code の MCP Server として動作し、AI エージェントからタイマーを直接操作できます。

**利用可能なツール:**

| Tool | Description |
|------|-------------|
| `start_timer` | タイマーを開始 |
| `pause_timer` | 一時停止 |
| `reset_timer` | リセット |
| `update_task` | タスク名変更 |
| `send_message` | メッセージ送信 |
| `get_status` | 現在のタイマー状態を取得 |
| `get_tasks` | タスク一覧を取得 |
| `get_summary` | ポモドーロ統計を取得 |
| `list_sessions` | セッション一覧を取得 |
| `list_messages` | メッセージ一覧を取得 |

## Timer Phases

| Phase | 時間 | ラベル | 説明 |
|-------|------|--------|------|
| `work` | 25分 | Focus | 通常の集中作業 |
| `deep_work` | 50分 | Deep | 長時間の深い集中作業 |
| `short_break` | 5分 | Break | 短い休憩 |
| `long_break` | 15分 | Long Break | 4ポモドーロ後の長い休憩 |

## Data & IPC

タイマーの状態はファイルベースの IPC で管理されています。

| File | Description |
|------|-------------|
| `~/.bout/status.json` | タイマーの現在状態（毎秒更新） |
| `~/.bout/command.json` | CLI → アプリへのコマンド |
| `~/.bout/messages.json` | メッセージキュー |
| `~/.bout/sessions.json` | 完了したセッション履歴 |
| `~/.bout/events.json` | タイマーイベントログ（7日間保持） |

### Event Types

`events.json` に記録されるイベント:

| Event | Description |
|-------|-------------|
| `timer_started` | タイマー開始 |
| `timer_resumed` | 一時停止から再開 |
| `timer_paused` | 一時停止 |
| `timer_reset` | リセット |
| `phase_completed` | フェーズ完了 |
| `phase_changed` | フェーズ自動切替 |

## License

MIT
