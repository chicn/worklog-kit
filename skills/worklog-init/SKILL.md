---
name: worklog-init
description: リポジトリにワークログの仕組み一式（docs/worklog/、.claude/rules/worklog.md の規則、書き忘れを防ぐ2つの hook）を用意する。新しいリポジトリを立ち上げたとき、または「ワークログを用意して」「worklog を入れて」「このリポジトリでも作業記録を残したい」と言われたときに使う。
---

# ワークログの仕組みをリポジトリに用意する

そのリポジトリでやったことをすべて `docs/worklog/` に残すための一式を置く。置くものは3つ。

1. `docs/worklog/` と、導入を記録した最初のワークログ
2. `.claude/rules/worklog.md`（何をいつどう書くかの規則）
3. `.claude/hooks/worklog-check.py` と `.claude/settings.json` の hook 設定（書き忘れの検知）

素材はこのスキルの `assets/` にある。文面と設定は素材のまま使い、リポジトリに合わせて書き換えない。複数のリポジトリで同じ規則にそろえるため。

以前の版を入れたリポジトリでもう一度実行すると、新しい版に更新される。規則は `CLAUDE.md` から `.claude/rules/worklog.md` に移り、hook のスクリプトは素材で上書きされる。

`CLAUDE.md` には何も書かない。`.claude/rules/` の Markdown は、Claude Code がセッションの開始時に `CLAUDE.md` と同じ扱いで読み込むので、案内の1行も要らない。

## 手順

### 1. 現状を確かめる

対象は現在の Git リポジトリのルート（`git rev-parse --show-toplevel`）。Git リポジトリでなければ、`git init` してよいかをユーザーに聞く。

次を確かめる。

- `docs/worklog/` や `worklog/` など、既存の作業記録のフォルダがあるか
- `.claude/rules/worklog.md` がすでにあるか
- `CLAUDE.md` に Worklog やワークログの節があるか（このスキルの以前の版は、規則を `CLAUDE.md` に入れていた）
- `.claude/settings.json` があるか。`hooks` がすでにあるか
- `python3` が使えるか

既存の作業記録が別の場所にある場合と、ファイル名の形式が違う場合は、作業を止めてユーザーに聞く。本文の見出しが違うだけなら続ける。既存の記録を移すか、そのまま残して今後だけ新しい規則にするかは、ユーザーが決める。

### 2. フォルダを作る

`docs/worklog/` を作る。

### 3. 規則を `.claude/rules/worklog.md` に置く

`assets/worklog-rule.md` を `.claude/rules/worklog.md` にコピーする。すでにあれば素材で上書きする。frontmatter は足さない。`paths` を付けると、該当するファイルを触ったときにしか読み込まれなくなる。

`CLAUDE.md` に Worklog 節がある場合は、規則が二重にならないように次のとおり扱う。

- 見出しの深さを除いて `assets/worklog-rule.md` と同じ内容なら、以前の版が入れたものなので、節ごと削除する。節は、Worklog の見出しから、次の同じ深さの見出しの手前（無ければファイルの末尾）まで。削除したことは手順7で伝える。
- 内容が違うなら、削除せずにユーザーに聞く。元の節にしかない規則を伝え、消すか残すかはユーザーが決める。

`CLAUDE.md` が無いリポジトリでは、作らない。

### 4. hook を置く

1. `assets/worklog-check.py` を `.claude/hooks/worklog-check.py` にコピーする。すでにあれば素材で上書きする。
2. `assets/settings-hooks.json` の `hooks` を `.claude/settings.json` に入れる。ファイルが無ければ作る。すでに `PreToolUse` や `Stop` があれば、既存の配列に要素を足す。同じ `worklog-check.py` の要素がすでにあれば足さない。既存の設定は消さない。
3. `.gitignore` が `.claude/` を除外していないかを確かめる。除外されていたら、規則、hook のスクリプト、設定の3つのファイルがコミットできるようにする。

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

今日の日付のワークログがすでにあるリポジトリでは、1つめも何も出ない。その場合は、ワークログのフォルダが空の仮のリポジトリを `cwd` に指定して確かめる。

`python3 -m json.tool .claude/settings.json` で、設定ファイルが正しい JSON であることも確かめる。

### 6. 最初のワークログを書く

いま入れた規則に従い、`docs/worklog/<date +%Y-%m-%d-%H-%M の出力>_worklog-setup.md` に、この導入作業を記録する。書いたあとで手順5の1つめのコマンドをもう一度実行し、今度は何も出ないことを確かめる。

### 7. 報告する

次を伝える。

- 作ったファイルと変更したファイル。`CLAUDE.md` から Worklog 節を削除した場合は、そのこと
- 規則と hook は次のセッションから働くこと（どちらもセッションの開始時に読み込まれる）。読み込まれた規則は `/memory` で確かめられる
- コミットはしていないこと。ユーザーに頼まれたらコミットする

## 完了の条件

- `docs/worklog/` に最初のワークログがある
- `.claude/rules/worklog.md` があり、`CLAUDE.md` に同じ規則が残っていない
- 手順5と6の確認がすべて期待どおりだった

## hook の調整

- 応答の終わりの確認が多すぎる、または少なすぎるときは、`worklog-check.py` の `STOP_THRESHOLD`（既定は10）を変える。環境変数 `WORKLOG_STOP_THRESHOLD` でも変えられる。
- hook をやめるときは、`.claude/settings.json` から該当の2つの要素を消す。`.claude/rules/worklog.md` の末尾にある hook の説明も合わせて消す。
