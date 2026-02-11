---
name: "Automation Plan (TOML)"
about: "Run a safe, declarative automation plan (create issues/comments) by label approval"
title: "[automation] "
labels: ["automation/request"]
---

以下に従って、epicとissueの作成をお願いします

- ISSUE番号はそのepicの中で閉じていいので1からの連番
- 設計がぶれないようにこれまでの会話をもとに本文を厚く記述
- patch例などあればそれも本文に記載
- 粒度が荒くならない様に調整しながら作成
- 1機能1epicになるように出力
- 依存関係を分析の上、フォーマットに従って記載
- 以下のフォーマットに従って出力
- レポジトリを跨いだブロッキングなどは行わないように注意
- 複数のレポジトリに関わる場合は、それぞれ以下のフォーマットに従って出力する

---

<!-- automation-plan -->
~~~~~toml
version = 1
# target_repo = "OWNER/REPO" # 省略時はこの repo

# 変数:
#   {{request.repo}} {{request.issue}} {{request.url}} {{request.actor}} {{request.author}}
#   {{steps.<id>.number}} {{steps.<id>.url}} {{steps.<id>.id}}
#
# 依存関係（Issue Dependencies / Relationships）:
#   - blocked_by = ["sub1", "#123", "owner/repo#123", "{{steps.other.number}}"]
#       => この Issue は指定先が完了するまで “ブロックされる”
#   - blocks = ["sub2"]  # (alias: blocking)
#       => この Issue が完了するまで 指定先を “ブロックする”
# 参照文字列は、(1) step の id, (2) Issue番号, (3) owner/repo#番号 を許可します。

[[steps]]
id = "epic"
type = "create_issue"
title = "Epic: <title>"
body = '''
# Epic: <title>

## 背景
...

## Sub-issues
この Epic の Sub-issues は、GitHub の Sub-issue 機能として紐づけられます（本文のチェックリスト管理は不要）。
'''
# labels = ["epic"] # optional

[[steps]]
id = "sub1"
type = "create_issue"
title = "WF-1: ..."
parent_issue = "{{steps.epic.number}}"
# replace_parent = true # optional: 既存親を置き換えたい場合
body = '''
...
'''

[[steps]]
id = "sub2"
type = "create_issue"
title = "WF-2: ..."
parent_issue = "{{steps.epic.number}}"
blocked_by = ["sub1"] # sub1 が終わるまで sub2 は動かせない
body = '''
...
'''

[[steps]]
id = "epic_comment"
type = "comment_issue"
issue = "{{steps.epic.number}}"
body = '''
Sub-issues created:
- {{steps.sub1.url}}
'''
~~~~~
<!-- /automation-plan -->
