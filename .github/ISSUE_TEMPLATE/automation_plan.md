---
name: "Automation Plan (TOML)"
about: "Run a safe, declarative automation plan (create issues/comments) by label approval"
title: "[automation] "
labels: ["automation/request"]
---

以下に従って、Epic / Issue / Sub-issue の作成をお願いします

- 構成は必ず以下の3階層とする
    1. Epic（大きな枠組み）
    2. Issue（Epic を構成する作業単位）
    3. Sub-issue（Issue を実行するタスクレベルのもの）
- Epic の直下に Issue を配置し、Issue の直下に Sub-issue を配置すること
- ISSUE番号は各 Epic の中で閉じてよく、1 からの連番とする
- SUB-ISSUE番号は各 Issue の中で閉じてよく、1 からの連番とする
- 上記番号は GitHub の実 issue 番号ではなく、タイトル上の管理番号として扱うこと
- リリース前なので、前方互換性は不要であり、段階移行ではなく一気にベストな状態に持っていくことを念頭に記述すること
- 設計がぶれないように、これまでの会話をもとに本文を厚く記述すること
- patch例などがあれば本文に記載すること
- 粒度が荒くなりすぎないように調整しながら作成すること
- 1機能1Epicになるように出力すること
- Epic間のブロッキング情報も出力すること
- Issue / Sub-issue 間の依存関係も分析の上で記載すること
- ドキュメントの整備は必ず Issue として含め、その配下に必要な Sub-issue を置くこと
- Sub-issue は DDL / migration / API / UI / test / docs など、実行可能な粒度まで分解すること
- レポジトリを跨いだブロッキングは行わないこと
- 複数のエピックを作成する場合は、postfixで機能概要を明示してidが一意になるように出力すること
- 複数のレポジトリに関わる場合は、それぞれ以下のフォーマットに従って独立した automation-plan を出力すること
- 以下のフォーマットに従って出力すること

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
#   - blocked_by = ["issue1_model", "#123", "owner/repo#123", "{{steps.other.number}}"]
#       => この Issue は指定先が完了するまで “ブロックされる”
#   - blocks = ["issue2_api"]  # (alias: blocking)
#       => この Issue が完了するまで指定先を “ブロックする”
# 参照文字列は、(1) step の id, (2) Issue番号, (3) owner/repo#番号 を許可します。
#
# 運用上の番号規則:
#   - Epic: 機能単位
#   - Issue: 各 Epic 内で 1 からの連番（例: ISSUE-1, ISSUE-2）
#   - Sub-issue: 各 Issue 内で 1 からの連番（例: SUB-1-1, SUB-1-2, SUB-2-1）
#   ※ 上記はタイトル上の管理番号であり、GitHub の実 issue 番号そのものではありません。

[[steps]]
id = "epic_<postfix>"
type = "create_issue"
title = "Epic: <title>"
# blocked_by = ["epic_<other_postfix>"] # optional: 同一repo内の別Epicに依存する場合のみ
body = '''
# Epic: <title>

## 背景
...

## 目的
...

## スコープ
...

## 完了条件
...

## Issues
この Epic 配下の Issue は、GitHub の親子関係として紐づけます。
本文内のチェックリスト管理は不要です。
'''
labels = ["epic"]

[[steps]]
id = "issue1_<postfix>"
type = "create_issue"
title = "ISSUE-1: ..."
parent_issue = "{{steps.epic_<postfix>.number}}"
# replace_parent = true # optional: 既存親を置き換えたい場合
body = '''
# ISSUE-1: ...

## 背景
...

## 目的
...

## スコープ
...

## 実装方針
...

## patch例
```diff
...
```

## 完了条件
...

## Sub-issues
この Issue 配下の Sub-issue は、GitHub の親子関係として紐づけます。
本文内のチェックリスト管理は不要です。
'''
labels = ["issue"]

[[steps]]
id = "sub1_1_<postfix>"
type = "create_issue"
title = "SUB-1-1: ..."
parent_issue = "{{steps.issue1_<postfix>.number}}"
body = '''
# SUB-1-1: ...

## 実施内容
...

## patch例
```diff
...
```

## 完了条件
...
'''
labels = ["sub-issue"]

[[steps]]
id = "sub1_2_<postfix>"
type = "create_issue"
title = "SUB-1-2: ..."
parent_issue = "{{steps.issue1_<postfix>.number}}"
blocked_by = ["sub1_1_<postfix>"]
body = '''
# SUB-1-2: ...

## 実施内容
...

## patch例
```diff
...
```

## 完了条件
...
'''
labels = ["sub-issue"]

[[steps]]
id = "issue2_<postfix>"
type = "create_issue"
title = "ISSUE-2: Docs / Spec alignment"
parent_issue = "{{steps.epic_<postfix>.number}}"
blocked_by = ["issue1_<postfix>"] # optional: Issue単位の依存がある場合
body = '''
# ISSUE-2: Docs / Spec alignment

## 背景
...

## 目的
設計・DDL・API・運用ルールの canonical な記述を更新し、実装とのズレをなくす。

## スコープ
- 設計文書更新
- API / DDL 差分反映
- migration / 運用上の注意点整理

## 完了条件
...

## Sub-issues
この Issue 配下の Sub-issue は、GitHub の親子関係として紐づけます。
本文内のチェックリスト管理は不要です。
'''
labels = ["issue", "documentation"]

[[steps]]
id = "sub2_1_<postfix>"
type = "create_issue"
title = "SUB-2-1: Update design docs"
parent_issue = "{{steps.issue2_<postfix>.number}}"
body = '''
# SUB-2-1: Update design docs

## 実施内容
...

## 完了条件
...
'''
labels = ["sub-issue", "documentation"]

[[steps]]
id = "sub2_2_<postfix>"
type = "create_issue"
title = "SUB-2-2: Sync API / DDL docs"
parent_issue = "{{steps.issue2_<postfix>.number}}"
blocked_by = ["sub2_1_<postfix>"]
body = '''
# SUB-2-2: Sync API / DDL docs

## 実施内容
...

## 完了条件
...
'''
labels = ["sub-issue", "documentation"]

[[steps]]
id = "epic_comment_<postfix>"
type = "comment_issue"
issue = "{{steps.epic_<postfix>.number}}"
body = '''
Issues / Sub-issues created:

- ISSUE-1: {{steps.issue1_<postfix>.url}}
  - SUB-1-1: {{steps.sub1_1_<postfix>.url}}
  - SUB-1-2: {{steps.sub1_2_<postfix>.url}}

- ISSUE-2: {{steps.issue2_<postfix>.url}}
  - SUB-2-1: {{steps.sub2_1_<postfix>.url}}
  - SUB-2-2: {{steps.sub2_2_<postfix>.url}}
'''
~~~~~
<!-- /automation-plan -->