# worklog-kit

リポジトリにワークログの仕組みを入れるスキル `worklog-init` を育てるプロジェクト。

- スキルの本体は `skills/worklog-init/` にある。`~/.claude/skills/worklog-init` はここへのシンボリックリンクなので、このフォルダを編集すると、すべてのリポジトリで使うスキルがその場で変わる。
- このリポジトリ自身にも同じ仕組みを入れて試している。`.claude/hooks/worklog-check.py` は `skills/worklog-init/assets/worklog-check.py` へのシンボリックリンクで、スクリプトを直すとこのリポジトリの hook にもすぐ反映される。
- ワークログの規則は `.claude/rules/worklog.md` にあり、セッションの開始時に読み込まれる。これも `skills/worklog-init/assets/worklog-rule.md` へのシンボリックリンクなので、素材を直すとこのリポジトリの規則も変わる。
