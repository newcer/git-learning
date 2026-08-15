# Git 学习笔记

> 学习环境：macOS + Ghostty + fish + GitHub  
> 当前阶段：已掌握 Git 本地基础、分支与合并、远程仓库、SSH、fetch/pull/push、merge/rebase 基础，并开始学习 PR 工作流。

---

## 1. Git 是什么

Git 可以理解为一个项目的“版本存档系统”。

每完成一个阶段，可以创建一个：

```text
commit
```

用来记录当时项目的状态。

核心流程：

```text
修改文件
   ↓
工作区
   ↓ git add
暂存区
   ↓ git commit
本地 Git 仓库
```

---

## 2. 初始化 Git 仓库

检查 Git：

```fish
git --version
```

初始化：

```fish
git init
```

查看状态：

```fish
git status
```

本次创建的仓库：

```text
/Users/hzj/git-learning/git-learning/
```

初始化后出现：

```text
On branch main
No commits yet
```

说明：

- 当前分支是 `main`
- 仓库还没有任何 commit

---

## 3. 第一次 add 和 commit

创建文件：

```fish
echo "Hello Git" > hello.txt
```

查看：

```fish
git status
```

新文件最开始属于：

```text
Untracked
```

即 Git 看到了文件，但还没有将其纳入下一次提交。

添加到暂存区：

```fish
git add hello.txt
```

提交：

```fish
git commit -m "Add hello file"
```

查看历史：

```fish
git log
git log --oneline
```

最早的提交历史：

```text
06075ba Add hello file
```

---

## 4. `.gitignore`

macOS 自动产生了：

```text
.DS_Store
```

一般不应该提交。

创建：

```fish
echo ".DS_Store" > .gitignore
```

然后：

```fish
git add .gitignore
git commit -m "Ignore .DS_Store"
```

`.gitignore` 的作用：

> 告诉 Git 哪些未跟踪文件或目录应该忽略。

---

## 5. Git 的三个区域

这是目前最重要的基础概念之一。

```text
HEAD / 最新 commit
        ↓
暂存区
        ↓
工作区
```

实验过程：

先修改：

```fish
echo "Learning Git is fun" >> hello.txt
```

然后：

```fish
git add hello.txt
```

再继续修改：

```fish
echo "Git has a staging area" >> hello.txt
```

此时同一个文件同时存在：

```text
Changes to be committed
Changes not staged for commit
```

原因：

`git add` 保存的是“文件当前这一刻的状态”，而不是永久标记整个文件。

---

## 6. `git diff` 与 `git diff --staged`

查看工作区和暂存区的差异：

```fish
git diff
```

查看暂存区和最新 commit 的差异：

```fish
git diff --staged
```

实验中：

```fish
git diff
```

显示：

```diff
+Git has a staging area
```

说明这一行只在工作区。

而：

```fish
git diff --staged
```

显示：

```diff
+Learning Git is fun
```

说明这一行已经进入暂存区。

记忆：

```text
git diff
工作区 ↔ 暂存区

git diff --staged
暂存区 ↔ HEAD
```

---

## 7. `git restore`

### 丢弃未暂存修改

```fish
git restore hello.txt
```

作用：

> 将工作区文件恢复到暂存区对应状态。

注意：

这会直接丢弃尚未暂存的修改。

---

### 取消暂存，但保留代码

```fish
git restore --staged hello.txt
```

作用：

> 把修改从暂存区撤出来，但工作区代码仍然存在。

实验后：

```text
Changes not staged for commit:
    modified: hello.txt
```

而：

```fish
cat hello.txt
```

仍能看到修改内容。

对比：

```text
git restore file
→ 丢工作区修改

git restore --staged file
→ 取消暂存，代码保留
```

---

## 8. `git revert`

已经 commit 后，如果想安全撤销某个提交：

```fish
git revert HEAD
```

它不会删除历史，而是创建一个新的 commit 来反向抵消旧 commit。

例如：

```text
ecfecf1 Revert "..."
4dec27a ...
```

说明原 commit 仍然存在，只是多了一个反向提交。

特点：

```text
revert
→ 保留历史
→ 新建反向 commit
→ 多人协作时通常更安全
```

---

## 9. `git reset`

### `--soft`

```fish
git reset --soft HEAD~1
```

只移动 HEAD / 分支指针。

```text
HEAD       改
暂存区     不改
工作区     不改
```

实验后：

```fish
git diff --staged
```

可以看到被“撤销 commit”留下的暂存内容。

---

### `--mixed`

默认模式：

```fish
git reset HEAD
```

等价于：

```fish
git reset --mixed HEAD
```

特点：

```text
HEAD       重置
暂存区     重置
工作区     保留
```

常用于取消暂存。

---

### `--hard`

```fish
git reset --hard HEAD
```

特点：

```text
HEAD       重置
暂存区     重置
工作区     重置
```

这是最危险的一种 reset，因为工作区修改也可能直接消失。

记忆：

```text
soft  = 修改还在暂存区
mixed = 修改还在工作区
hard  = 工作区修改也丢弃
```

---

## 10. HEAD 是什么

日志中：

```text
(HEAD -> main)
```

可以先理解：

```text
HEAD
→ 我当前所在的位置
```

通常：

```text
HEAD → 当前本地分支 → 某个 commit
```

例如：

```text
HEAD
 ↓
main
 ↓
● commit
```

---

## 11. 分支 Branch

创建并切换：

```fish
git switch -c feature/greeting
```

查看：

```fish
git branch
```

输出：

```text
* feature/greeting
  main
```

`*` 表示当前分支。

分支本质上可以理解成：

> 一个可以移动的 commit 指针，而不是复制整个项目。

---

## 12. Fast-forward Merge

在：

```text
main
 ↓
●
 \
  ● feature
```

且 `main` 没有新 commit 时：

```fish
git switch main
git merge feature/greeting
```

输出：

```text
Fast-forward
```

说明 Git 只是把 `main` 指针向前移动，没有创建额外 merge commit。

实验：

```text
ca705ad (HEAD -> main, feature/greeting) Add feature greeting
```

之后删除 feature：

```fish
git branch -d feature/greeting
```

删除分支不会删除已经合并的代码。

---

## 13. 真正的 Merge Commit

当两个分支都产生新 commit：

```text
        ● feature
       /
●─────
       \
        ● main
```

执行：

```fish
git merge feature/profile
```

得到：

```text
b4c7ff0 Merge branch 'feature/profile'
```

图形历史：

```fish
git log --oneline --graph --decorate --all
```

输出类似：

```text
*   b4c7ff0 (HEAD -> main) Merge branch 'feature/profile'
|\
| * d815a7d Add profile feature
* | 90cb84e Add main branch work
|/
```

Merge commit 通常有两个父 commit。

---

## 14. Merge Conflict

故意让：

```text
main
```

和：

```text
feature/hello
```

修改 `hello.txt` 同一行。

执行：

```fish
git merge feature/hello
```

出现：

```text
CONFLICT (content): Merge conflict in hello.txt
Automatic merge failed
```

文件中出现：

```text
<<<<<<< HEAD
Hello from Main
=======
Hello from Feature
>>>>>>> feature/hello
```

解释：

```text
<<<<<<< HEAD
当前分支内容

=======
分隔线

>>>>>>> feature/hello
正在合并进来的分支内容
```

手动修改成：

```text
Hello from Main and Feature
```

然后：

```fish
git add hello.txt
git commit -m "Merge feature/hello and resolve greeting conflict"
```

最终：

```text
3aac77f Merge feature/hello and resolve greeting conflict
```

---

## 15. 冲突中的 ours / theirs

在 merge 场景下：

```text
ours
→ 当前分支

theirs
→ 正在合并进来的分支
```

可以直接选择一边：

```fish
git restore --ours hello.txt
```

或：

```fish
git restore --theirs hello.txt
```

之后仍需：

```fish
git add hello.txt
git commit
```

---

## 16. fish 与 Bash 语法差异

当前 shell：

```text
fish
```

所以 Bash/Zsh 的 heredoc：

```bash
cat > file <<'EOF'
...
EOF
```

不能直接使用。

fish 可以改用：

```fish
printf '%s\n' \
'line 1' \
'line 2' \
'line 3' > file.txt
```

或者直接用编辑器：

```fish
nano file.txt
```

---

## 17. Git 与 GitHub 的区别

Git：

```text
本地版本控制工具
```

GitHub：

```text
托管 Git 仓库的平台
```

即使断网，本地仍可以：

```fish
git add
git commit
git branch
git merge
git log
```

只有与远程交互时才需要 GitHub。

---

## 18. remote / origin / origin/main

查看远程：

```fish
git remote -v
```

添加远程：

```fish
git remote add origin <URL>
```

概念：

```text
main
→ 本地分支

origin
→ 远程仓库的昵称

origin/main
→ 本地记录的远程 main 状态
```

`origin` 只是约定俗成的名字，不是 Git 强制关键字。

---

## 19. GitHub SSH 认证

最开始使用 HTTPS push：

```fish
git push -u origin main
```

出现：

```text
Password authentication is not supported for Git operations.
```

因此改用 SSH。

生成 SSH key：

```fish
ssh-keygen -t ed25519 -C "1092863528@qq.com"
```

生成：

```text
~/.ssh/id_ed25519
~/.ssh/id_ed25519.pub
```

其中：

```text
id_ed25519
→ 私钥，不能泄露

id_ed25519.pub
→ 公钥，可添加到 GitHub
```

---

## 20. GitHub SSH 走 443 端口

直接：

```fish
ssh -vT git@github.com
```

发现虽然：

```text
Connection established.
```

但 SSH 握手卡住。

改用 GitHub SSH over HTTPS 端口：

```fish
ssh -vT -p 443 git@ssh.github.com
```

成功：

```text
Authenticated to ssh.github.com using "publickey".
Hi newcer! You've successfully authenticated
```

然后配置：

```text
~/.ssh/config
```

内容：

```text
Host github.com
    HostName ssh.github.com
    Port 443
    User git
    IdentityFile ~/.ssh/id_ed25519
```

这样以后：

```fish
ssh -T git@github.com
```

也会自动走 443。

---

## 21. 把 remote 从 HTTPS 改成 SSH

原地址：

```text
https://github.com/newcer/git-learning.git
```

修改：

```fish
git remote set-url origin git@github.com:newcer/git-learning.git
```

检查：

```fish
git remote -v
```

之后 push 成功：

```text
main -> main
branch 'main' set up to track 'origin/main'
```

---

## 22. `git push`

第一次：

```fish
git push -u origin main
```

`-u`：

```text
--set-upstream
```

告诉 Git：

```text
本地 main
默认跟踪
origin/main
```

之后可以直接：

```fish
git push
```

---

## 23. ahead 状态

本地创建 commit：

```fish
echo "Learning remote repositories" > remote.txt
git add remote.txt
git commit -m "Learn Git remotes"
```

未 push 时：

```text
Your branch is ahead of 'origin/main' by 1 commit.
```

日志：

```text
bb1b2fb (HEAD -> main) Learn Git remotes
3aac77f (origin/main) ...
```

结构：

```text
origin/main
    ↓
●────●
      \
       ●
       ↑
      main
```

执行：

```fish
git push
```

后：

```text
bb1b2fb (HEAD -> main, origin/main)
```

重新同步。

---

## 24. `git fetch`

在 GitHub 网页直接修改 `remote.txt` 并提交：

```text
92afb70 Edit remote file on GitHub
```

此时本地最开始仍显示：

```text
Your branch is up to date with 'origin/main'.
```

因为本地还不知道 GitHub 已经变化。

执行：

```fish
git fetch
```

后：

```text
Your branch is behind 'origin/main' by 1 commit
```

日志：

```text
92afb70 (origin/main, origin/HEAD) Edit remote file on GitHub
bb1b2fb (HEAD -> main) Learn Git remotes
```

重要理解：

```text
git fetch
→ 下载远程更新
→ 更新 origin/main
→ 不直接修改本地 main 和工作区
```

---

## 25. `git pull`

执行：

```fish
git pull
```

输出：

```text
Fast-forward
```

然后：

```fish
cat remote.txt
```

出现 GitHub 上的新内容。

简单情况下可以理解：

```text
git pull
≈ git fetch + git merge
```

---

## 26. origin/HEAD

日志中：

```text
origin/main
origin/HEAD
```

区别：

```text
HEAD
→ 当前本地分支位置

origin/HEAD
→ 远程 origin 的默认分支
```

当前：

```text
origin/HEAD → origin/main
```

---

## 27. 本地和远程分叉 Diverged

故意制造：

```text
本地：
c99c8e1 Add local work

远程：
81e6146 Add GitHub work
```

共同父节点：

```text
92afb70
```

日志：

```text
* 81e6146 (origin/main) Add GitHub work
| * c99c8e1 (HEAD -> main) Add local work
|/
* 92afb70
```

状态：

```text
Your branch and 'origin/main' have diverged
```

Ghostty 提示：

```text
[⇕]
```

---

## 28. 分叉后用 merge

先：

```fish
git fetch
```

再：

```fish
git merge origin/main
```

结果：

```text
c6c7933 Merge remote-tracking branch 'origin/main'
```

日志：

```text
*   c6c7933 (HEAD -> main) Merge remote-tracking branch 'origin/main'
|\
| * 81e6146 (origin/main) Add GitHub work
* | c99c8e1 Add local work
|/
```

此时：

```text
ahead of 'origin/main' by 2 commits
```

原因：

本地独有：

```text
c99c8e1 Add local work
c6c7933 Merge...
```

随后：

```fish
git push
```

重新同步。

---

## 29. Rebase

再次制造分叉：

```text
2aae354 (origin/main) Add second GitHub work
bfb4c20 (main) Add second local work
```

日志：

```text
* 2aae354 (origin/main)
| * bfb4c20 (HEAD -> main)
|/
* c6c7933
```

执行：

```fish
git rebase origin/main
```

成功后：

```text
db2a39f (HEAD -> main) Add second local work
2aae354 (origin/main) Add second GitHub work
```

历史从：

```text
      B
     /
A ──
     \
      C
```

变成：

```text
A ── B ── C'
```

---

## 30. 为什么 rebase 会改变 commit hash

Rebase 前：

```text
bfb4c20 Add second local work
```

Rebase 后：

```text
db2a39f Add second local work
```

虽然内容和 message 类似，但 commit 的父节点改变了。

Commit 可以粗略理解为包含：

```text
文件快照
作者
时间
commit message
父 commit
```

父 commit 改变：

```text
commit hash 也改变
```

所以：

> rebase 不是移动原 commit，而是在新位置重新创建 commit。

---

## 31. Merge vs Rebase

### Merge

```text
      B
     / \
A ──●   M
     \ /
      C
```

特点：

```text
保留真实分叉
不改已有 commit
可能多一个 merge commit
```

### Rebase

```text
A ── B ── C'
```

特点：

```text
历史更线性
通常没有额外 merge commit
会重写本地 commit
hash 改变
```

记忆：

```text
merge
→ 把两条历史汇合

rebase
→ 把自己的 commit 重放到新基线之后
```

经验：

```text
自己尚未共享的 commit
→ 可以放心 rebase

已 push 且别人可能基于它开发
→ 不要随便 rebase
```

---

## 32. `git pull --rebase`

如果本地和远程都有新 commit，又希望历史保持线性：

```fish
git pull --rebase
```

可以粗略理解：

```text
git fetch
+
git rebase origin/main
```

而普通 pull 在 merge 策略下可以粗略理解：

```text
git fetch
+
git merge origin/main
```

---

# 33. Pull Request（PR）工作流

PR 不是 Git 原生命令，而是 GitHub 提供的协作功能。

完整流程：

```text
main
 ↓
创建 feature 分支
 ↓
开发
 ↓
commit
 ↓
push feature 分支
 ↓
创建 Pull Request
 ↓
Review
 ↓
继续修改 + commit + push
 ↓
PR 自动更新
 ↓
Merge
 ↓
本地 pull
 ↓
删除 feature 分支
```

---

## 34. PR 的 base / compare

例如：

```text
base: main
compare: feature/pr-demo
```

意思：

```text
feature/pr-demo
       ↓
      PR
       ↓
      main
```

即：

> 请求把 feature 分支的修改合并到 main。

---

## 35. 推送 feature 分支

创建：

```fish
git switch -c feature/pr-demo
```

开发：

```fish
echo "Learning Pull Requests" > pr-demo.txt
git add pr-demo.txt
git commit -m "Add PR demo"
```

第一次 push：

```fish
git push -u origin feature/pr-demo
```

之后只需要：

```fish
git push
```

---

## 36. PR 会随着分支自动更新

PR 创建后，如果 reviewer 要求修改：

```fish
echo "Pull requests update when new commits are pushed." >> pr-demo.txt

git add pr-demo.txt
git commit -m "Explain PR updates"
git push
```

不需要重新创建 PR。

原因：

> PR 比较的是 base 分支和 head 分支当前的差异。

---

## 37. PR Review

常见区域：

```text
Conversation
Commits
Files changed
```

`Files changed` 是 Code Review 核心：

> Review 的不是整个项目，而是“这个 PR 准备给 base 分支带来什么变化”。

常见 Review 结果：

```text
Comment
Approve
Request changes
```

---

## 38. PR 的合并方式

常见：

```text
Create a merge commit
Squash and merge
Rebase and merge
```

### Merge commit

保留分叉历史：

```text
      B──C
     /    \
A───●──────M
```

### Squash and merge

feature：

```text
B
C
D
```

合入 main 时压成：

```text
S
```

适合个人项目和希望 main 历史整洁的项目。

### Rebase and merge

将多个 commit 线性放到 main 后面。

---

## 39. PR Merge Requirement

在 GitHub PR 页面遇到：

```text
Merging is blocked due to failing merge requirements
```

说明仓库对 main 配置了合并规则。

可能包括：

```text
Required review
Required status checks
Branch must be up to date
Branch protection / ruleset
```

此时应该先查看具体失败 requirement，而不是直接关闭 PR 或乱改 Git 历史。

---

# 40. 当前掌握程度

目前已经掌握：

```text
✅ git init
✅ git status
✅ git add
✅ git commit
✅ git log
✅ git diff
✅ .gitignore

✅ 工作区 / 暂存区 / HEAD
✅ git restore
✅ git restore --staged

✅ git revert
✅ git reset --soft
✅ git reset --mixed
✅ git reset --hard

✅ branch
✅ git switch
✅ Fast-forward merge
✅ merge commit
✅ merge conflict
✅ ours / theirs

✅ remote
✅ origin
✅ origin/main
✅ SSH key
✅ GitHub SSH over 443

✅ git push
✅ git fetch
✅ git pull
✅ ahead / behind
✅ diverged

✅ merge vs rebase
✅ rebase 基础
✅ git pull --rebase

✅ PR 基础工作流
✅ base / compare
✅ Review 基础
✅ PR 更新
✅ Merge requirement 基础
```

---

# 41. 个人开发是否够用

当前水平已经足够应付绝大多数个人项目。

推荐个人开发基础流程：

```fish
git switch main
git pull

git switch -c feature/xxx

# 开发

git status
git diff

git add ...
git commit -m "..."

git push -u origin feature/xxx
```

然后：

```text
GitHub PR
→ Review
→ Merge
```

最后：

```fish
git switch main
git pull
git branch -d feature/xxx
```

---

# 42. 推荐继续学习

下一阶段：

```text
1. rebase conflict
2. git stash
3. git commit --amend
4. git cherry-pick
5. interactive rebase
6. git reflog
7. 远程分支
8. 完整 PR / GitHub 协作
9. tag / release
```

更进阶：

```text
git bisect
git worktree
git hooks
submodule
commit signing
```

---

# 43. 最重要的 Git 心智模型

## 提交链

```text
commit ← commit ← commit
```

每个 commit 指向父 commit。

---

## 分支

```text
branch
  ↓
commit
```

分支本质是可移动指针。

---

## HEAD

```text
HEAD
 ↓
当前分支
 ↓
commit
```

---

## 本地与远程

```text
main
→ 本地分支

origin/main
→ 本地记录的远程 main 状态

origin
→ 远程仓库昵称
```

---

## push

```text
本地 commit
    ↓
GitHub
```

---

## fetch

```text
GitHub
  ↓
更新 origin/main
但不动当前 main
```

---

## pull

```text
fetch
+
整合到当前分支
```

---

## merge

```text
把两个历史汇合
```

---

## rebase

```text
把自己的 commit
重新应用到新的基线之后
```

---

# 44. 常用命令速查

```fish
# 状态
git status

# 查看差异
git diff
git diff --staged

# 暂存与提交
git add <file>
git commit -m "message"

# 历史
git log --oneline
git log --oneline --graph --decorate --all

# 分支
git branch
git switch main
git switch -c feature/xxx

# 撤销
git restore <file>
git restore --staged <file>
git revert HEAD

git reset --soft HEAD~1
git reset --mixed HEAD~1
git reset --hard HEAD~1

# 合并
git merge <branch>

# rebase
git rebase origin/main
git pull --rebase

# 远程
git remote -v
git fetch
git pull
git push

# 新分支第一次 push
git push -u origin feature/xxx

# 删除分支
git branch -d feature/xxx
git push origin --delete feature/xxx
```

---

# 45. 当前实际 Git 历史中的关键节点

学习过程中出现过的部分 commit：

```text
06075ba Add hello file
af15569 Ignore .DS_Store
b5a766f Learn git staging

ca705ad Add feature greeting

90cb84e Add main branch work
d815a7d Add profile feature
b4c7ff0 Merge branch 'feature/profile'

38b9336 Change greeting on main
815cd86 Change greeting on feature
3aac77f Merge feature/hello and resolve greeting conflict

bb1b2fb Learn Git remotes
92afb70 Edit remote file on GitHub

c99c8e1 Add local work
81e6146 Add GitHub work
c6c7933 Merge remote-tracking branch 'origin/main'

bfb4c20 Add second local work
2aae354 Add second GitHub work

# rebase 后：
db2a39f Add second local work
```

其中：

```text
bfb4c20 → db2a39f
```

是理解 rebase 改写 commit hash 的重要实际案例。

---

## 最后的学习总结

目前最重要的进步不是记住了多少命令，而是已经建立了这套理解：

```text
工作区
  ↓ add
暂存区
  ↓ commit
本地仓库
  ↓ push
远程仓库
```

以及：

```text
HEAD
main
origin/main
```

分别是什么。

遇到问题时，优先使用：

```fish
git status
git log --oneline --graph --decorate --all
git diff
git diff --staged
```

先判断“Git 当前到底处于什么状态”，再决定下一步，而不是靠猜命令。

这会比单纯背 Git 命令可靠得多。
