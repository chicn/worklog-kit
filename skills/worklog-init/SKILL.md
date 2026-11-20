---
name: worklog-init
description: リポジトリにワークログの仕組み一式（docs/worklog/、CLAUDE.md の Worklog 節、書き忘れを防ぐ2つの hook）を用意する。新しいリポジトリを立ち上げたとき、または「ワークログを用意して」「worklog を入れて」「このリポジトリでも作業記録を残したい」と言われたときに使う。
---

# ワークログの仕組みをリポジトリに用意する

そのリポジトリでやったことをすべて `docs/worklog/` に残すための一式を置く。置くものは3つ。

1. `docs/worklog/` と、導入を記録した最初のワークログ
2. `CLAUDE.md` の Worklog 節（何をいつどう書くかの規則）
3. `.claude/hooks/worklog-check.py` と `.claude/settings.json` の hook 設定（書き忘れの検知）

素材はこのスキルの `assets/` にある。文面と設定は素材のまま使い、リポジトリに合わせて書き換えない。複数のリポジトリで同じ規則にそろえるため。

## 手順

### 1. 現状を確かめる

対象は現在の Git リポジトリのルート（`git rev-parse --show-toplevel`）。Git リポジトリでなければ、`git init` してよいかをユーザーに聞く。

次を確かめる。

- `docs/worklog/` や `worklog/` など、既存の作業記録のフォルダがあるか
- `CLAUDE.md` があるか。Worklog やワークログの節がすでにあるか
- `.claude/settings.json` があるか。`hooks` がすでにあるか
- `python3` が使えるか

既存の作業記録が別の場所や別の形式である場合は、作業を止めてユーザーに聞く。既存の記録を移すか、そのまま残して今後だけ新しい規則にするかは、ユーザーが決める。

### 2. フォルダを作る

`docs/worklog/` を作る。

### 3. `CLAUDE.md` に Worklog 節を入れる

`assets/claude-md-section.md` の内容を `CLAUDE.md` の末尾に足す。`CLAUDE.md` が無ければ作る。すでに Worklog 節があれば、その節を素材の内容で置き換える。置き換える前に、元の節にしかない規則が無いかを読み、あればユーザーに伝える。

### 4. hook を置く

1. `assets/worklog-check.py` を `.claude/hooks/worklog-check.py` にコピーする。
2. `assets/settings-hooks.json` の `hooks` を `.claude/settings.json` に入れる。ファイルが無ければ作る。すでに `PreToolUse` や `Stop` があれば、既存の配列に要素を足す。既存の設定は消さない。
3. `.gitignore` が `.claude/` を除外していないかを確かめる。除外されていたら、この2つのファイルがコミットできるようにする。

`.claude/settings.local.json` には入れない。コミットされず、ワークツリーや他のメンバーに届かないため。

### 5. hook が動くことを確かめる

次の2つを実行し、出力を確かめる。

```bash
# 今日のワークログがまだ無いので、permissionDecision が deny の JSON が出る
echo '{"tool_name":"Bash","tool_input":{"command":"git commit -m test"},"cwd":"'"$PWD"'"}' \
  | python3 .claude/hooks/worklog-check.py commit

# git commit 以外では何も出ない
echo '{"tool_name":"Bash","tool_input":{"command":"git status"},"cwd":"'"$PWD"'"}' \
  | python3 .claude/hooks/worklog-check.py commit
```

`python3 -m json.tool .claude/settings.json` で、設定ファイルが正しい JSON であることも確かめる。

### 6. 最初のワークログを書く

いま入れた規則に従い、`docs/worklog/<date +%Y-%m-%d-%H-%M の出力>_worklog-setup.md` に、この導入作業を記録する。書いたあとで手順5の1つめのコマンドをもう一度実行し、今度は何も出ないことを確かめる。

### 7. 報告する

次を伝える。

- 作ったファイルと変更したファイル
- hook は次のセッションから働くこと（設定はセッションの開始時に読み込まれる）
- コミットはしていないこと。ユーザーに頼まれたらコミットする

## 完了の条件

- `docs/worklog/` に最初のワークログがある
- `CLAUDE.md` に Worklog 節がある
- 手順5と6の確認がすべて期待どおりだった

## hook の調整

- 応答の終わりの確認が多すぎる、または少なすぎるときは、`worklog-check.py` の `STOP_THRESHOLD`（既定は10）を変える。環境変数 `WORKLOG_STOP_THRESHOLD` でも変えられる。
- hook をやめるときは、`.claude/settings.json` から該当の2つの要素を消す。`CLAUDE.md` の Worklog 節の末尾にある hook の説明も合わせて消す。
