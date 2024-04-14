# 目的 purpose
コードレビューにおいて、修正ファイル数が多ければ多いほどレビュワーにとっての負荷が高まる。<br>結果的に、レビュー時の見落としが発生し、最悪の場合障害につながるケースも考えられる。
<br>こちらのhooksは、修正ファイル数の上限を事前に設定し、push時に親ブランチと子ブランチのファイル数の差分が上限値を超えていないかどうかチェックするためのものである。
<br>もし上限値を超えていた場合はエラーが発生し、pushが中止される。
<br>このタイミングでタスクの細分化や余分なコードがないかどうかの見直しを行うきっかけにしてもらえると嬉しい。


In code reviews, the greater the number of modified files, the greater the burden on reviewers. <br>As a result, oversights may occur during reviews, which in the worst case may lead to failures.
<br>This hook is used to set an upper limit on the number of modified files in advance and check whether the difference in the number of files between the parent branch and child branch does not exceed the upper limit when pushing.
<br>If the upper limit is exceeded, an error will occur and the push will be aborted.
<br>I would be happy if this timing could be used as an opportunity to subdivide tasks and review whether there are any unnecessary codes.

# 導入手順 Installation steps
.git/hooks/pre-pushを作成していない場合<br>Create .git/hooks/pre-push if you haven't done so already.
```
$ touch .git/hooks/pre-push
```
.git/hooks/pre-pushを開く<br>let's open .git/hooks/pre-push
```
vi .git/hooks/pre-push
```

以下コードをペースト<br>Complete by pasting the code below
```
#!/bin/bash

current_branch=$(git branch --show-current)

parent_branch=$(git show-branch | grep '*' | grep -v "$(git rev-parse --abbrev-ref HEAD)" | head -1 | awk -F'[]~^[]' '{print $2}')

count_changed_lines() {
    git diff --shortstat $parent_branch..$current_branch | sed -rn 's/^.* ([0-9]+) insertions.*$/\1/p'
}

# 上限値を任意の値で設定
MAX_LINES=100

CHANGED_LINES=$(count_changed_lines)

if [ "$CHANGED_LINES" -gt "$MAX_LINES" ]; then
    echo "Aborting push: The number of changed lines exceeds the upper limit ($MAX_LINES) ($CHANGED_LINES lines)"
    exit 1
fi

exit 0
```