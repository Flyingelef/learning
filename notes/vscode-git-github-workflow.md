# 使用 VSCode 进行 Git 版本控制 + GitHub 基本操作全流程

> 环境参考：Git 2.55.0 / VSCode 1.140.0 / Windows
> 目标读者：从零开始，或用过命令行但想转到 VSCode 图形界面的人

---

## 一、前置准备

### 1.1 安装

| 软件 | 说明 | 验证命令 |
|---|---|---|
| Git | 版本控制核心。Windows 装 [Git for Windows](https://git-scm.com/download/win) | `git --version` |
| VSCode | 编辑器，自带 Git 集成，无需额外装 Git 插件 | `code --version` |

如果 VSCode 左侧活动栏**看不到源代码管理图标**（三个圆点连线），说明 Git 没被识别，重装 Git 或手动在设置里指定：
`设置 → 搜索 git.path → 填入 git.exe 的完整路径`

### 1.2 首次使用必须做的身份配置

Git 的每一次提交都要记录"谁提交的"，不配置无法 commit。

```bash
git config --global user.name  "你的名字"
git config --global user.email "你的邮箱@example.com"

# 查看是否配置成功
git config --global --list
```

> ⚠️ 邮箱必须和 GitHub 账号绑定的邮箱一致，否则 GitHub 贡献图（绿点）不会统计。

### 1.3 推荐开启的配置

```bash
# 默认分支名改成 main（和 GitHub 保持一致）
git config --global init.defaultBranch main

# 解决 Windows / Linux 换行符混乱问题
git config --global core.autocrlf true

# 让 git 命令输出带颜色
git config --global color.ui auto
```

---

## 二、VSCode 界面速览

打开任意文件夹后，左边活动栏第三个图标就是 **源代码管理 (Source Control)**，快捷键 `Ctrl + Shift + G`。

![三大区域说明]
- **顶层区域**：消息输入框 + `✓` 提交按钮
- **更改 (Changes)**：已修改但未暂存的文件
- **暂存的更改 (Staged Changes)**：已 `git add` 的文件
- **文件右侧字母标记**：
  - `U` Untracked —— 新文件，Git 还没跟踪
  - `A` Added —— 新文件，已加入暂存区
  - `M` Modified —— 内容被修改过
  - `D` Deleted —— 文件被删除

**编辑器行号槽的竖条**，这是 VSCode 最实用的功能之一：
- 🔵 蓝色条 = 该行被修改
- 🟢 绿色条 = 该行为新增
- 🔴 红色三角 = 该行被删除

点击这个色条，会弹出浮动对比窗口，左下角有 `撤回此更改` 按钮，可直接还原单行。

**左下角状态栏**显示当前分支名 `main`，点击它可以切换/新建分支。

---

## 三、本地版本控制的完整流程（单人开发）

这是每天重复度最高的一段动作。

### 3.1 初始化仓库

两种方式，二选一：

**方式 A：本地新建**
```bash
git init
```
VSCode 等价操作：`源代码管理面板 → 点击"初始化仓库"`

**方式 B：克隆远程仓库（接第五节的 GitHub 流程）**
```bash
git clone git@github.com:用户名/仓库名.git
```
VSCode 等价操作：`Ctrl+Shift+P → Git: Clone`

初始化后，文件夹里会多出一个隐藏的 `.git` 目录，它就是整个仓库的数据库，**不要手动改它**。

### 3.2 写代码时被忽略的文件 —— `.gitignore`

在项目根目录新建 `.gitignore`，列出不希望被版本控制的内容：

```gitignore
# Java
target/
*.class
*.jar
!gradle/wrapper/gradle-wrapper.jar

# IDE
.idea/
*.iml
.vscode/settings.json

# 系统
.DS_Store
Thumbs.db

# 依赖与构建
node_modules/
dist/
/build/

# 本地环境变量 / 密钥 —— 极其重要
.env
application-local.yml
```

> 如果某个文件**已经被跟踪了**，再写进 `.gitignore` 是无效的，需要先移除跟踪：
> `git rm --cached 文件名`

### 3.3 日常循环：修改 → 暂存 → 提交

**第 1 步：写代码**，保存文件后，源代码管理面板会自动刷出带 `M`/`U` 标记的文件，并统计改动数量。

**第 2 步：暂存（Stage）** —— 把这次要提交的改动挑出来

VSCode 操作：鼠标悬停文件 → 点 `+` 号；或在 **更改** 标题栏点 `+` 全部暂存。

命令行等价：`git add 文件名` / `git add .`

> 为什么要有"暂存"这一步？
> 因为一次工作里往往改了多个逻辑无关的东西。暂存区让你可以把「修复 bug」和「调整 UI」拆成**两个独立的提交**，而不是揉成一坨。这是 Git 和 SVN 最大的区别，也是专业与否的分水岭。

**第 3 步：提交（Commit）** —— 生成一个版本快照

VSCode 操作：在面板顶部输入框写好提交信息，按 `Ctrl + Enter` 或点 `✓`。

命令行等价：`git commit -m "提交信息"`

**提交信息是有规范的**，推荐 [Conventional Commits](https://www.conventionalcommits.org/)：

```
feat:     新增功能
fix:      修复 bug
docs:     只改文档
style:    格式调整（不影响逻辑，如空格、缩进）
refactor: 重构（既不加功能也不修 bug）
perf:     性能优化
test:     新增或修改测试
chore:    构建/工具/依赖的变动

示例：
feat: 新增用户登录校验
fix: 修复订单金额精度丢失问题
docs: 补充 README 部署步骤
```

> 团队协作建议勾选 VSCode 提交框下方的 "始终提交并同步"，省一次 push。

**第 4 步：查看历史**

- VSCode：安装 **Git Graph** 或 **GitLens** 插件，能可视化地看提交树、分支图、每行代码的最后修改者。
- 命令行：`git log --oneline --graph --all`

---

## 四、撤销与回退（别只会往前走）

按**危险程度**从低到高排列，处理前先看清自己在哪一层。

| 场景 | 命令 | 是否可恢复 |
|---|---|---|
| 撤销某个文件的工作区改动（没 add） | `git restore 文件名`<br>或 VSCode 更改文件右侧 `↺` 撤回箭头 | ❌ 不可恢复 |
| 把文件移出暂存区（已 add 未 commit） | `git restore --staged 文件名` | ✅ 改动还在 |
| 追加到上一个提交，不改提交信息 | `git commit --amend --no-edit` | ✅ |
| 修改上一个提交的信息 | `git commit --amend -m "新信息"` | ✅ |
| 撤销某次提交，保留历史记录 | `git revert <commit-id>` | ✅ 推荐用在已 push 的提交上 |
| 回退到某次提交，**丢弃之后所有历史** | `git reset --hard <commit-id>` | ⚠️ 极危险，已 push 时禁止对公共分支使用 |

> **救命命令**：`git reflog`
> 它记录了 HEAD 走过的每一步。即使 `--hard` 回退错了，只要 commit 还在 reflog 里，就能用 `git reset --hard <那个id>` 捞回来。

VSCode 里做回退：安装 **Git Graph** 插件 → 在提交节点上右键 → 可选 `Revert Commit` / `Reset to this commit` / `Checkout`。

---

## 五、GitHub 基本操作流程

### 5.1 注册并完成 SSH 免密配置（一次配好，长期省心）

HTTPS 方式每次推送都要输密码，且 GitHub 已不支持账密。推荐 SSH。

**第 1 步：生成密钥**

```bash
ssh-keygen -t ed25519 -C "你的邮箱@example.com"
```
一路回车，会在 `C:\Users\你的用户名\.ssh\` 下生成 `id_ed25519`（私钥，绝不能给任何人）和 `id_ed25519.pub`（公钥）。

**第 2 步：把公钥加到 GitHub**

```bash
# 复制公钥内容（Windows Git Bash）
cat ~/.ssh/id_ed25519.pub
```
GitHub → 右上角头像 → **Settings** → **SSH and GPG keys** → **New SSH key** → 粘贴保存。

**第 3 步：测试连通**

```bash
ssh -T git@github.com
# 看到 "Hi 用户名! You've successfully authenticated" 即为成功
```

### 5.2 从零把本地项目推到 GitHub

这是新手最容易卡住的一段，完整顺序如下：

```
① GitHub 网页 → 右上角 + → New repository
        ↓
② 填仓库名 → 选 Public/Private
   ⚠️ 不要勾选 "Add a README file"，保持空仓库，否则后面 push 会冲突
        ↓
③ 复制仓库地址（选 SSH，形如 git@github.com:xxx/yyy.git）
        ↓
④ 本地项目根目录执行 ↓
```

```bash
git init
git add .
git commit -m "feat: 项目初始化"
git branch -M main
git remote add origin git@github.com:用户名/仓库名.git
git push -u origin main
```

**逐条解释：**

| 命令 | 作用 |
|---|---|
| `git remote add origin <地址>` | 给远程仓库起个别名叫 `origin`（业界约定俗成的名字） |
| `-u` (即 `--set-upstream`) | 把本地 main 和远程 main 绑定，之后直接 `git push` 即可，不用再带参数 |
| `git remote -v` | 查看当前绑定的远程地址 |
| `git remote set-url origin <新地址>` | 地址填错了用它改 |

> **踩坑：本地已有 README 或别的历史，push 被拒绝**
> 报 `! [rejected] main -> main (fetch first)` 时：
> ```bash
> git pull origin main --allow-unrelated-histories
> # 有冲突就解决冲突
> git push origin main
> ```

### 5.3 标准协作流程（团队 / 参与开源项目）

单人可以直接在 main 分支上干，但**多人协作必须走分支 + Pull Request**。

```bash
# 1. 保证本地 main 是最新的
git checkout main
git pull origin main

# 2. 从最新的 main 上切出功能分支
git checkout -b feature/user-login
# 等价：git switch -c feature/user-login

# 3. 在分支上开发、提交
git add .
git commit -m "feat: 实现用户登录"

# 4. 推送分支到远端
git push -u origin feature/user-login

# 5. 到 GitHub 网页发起 Pull Request (PR)
#    目标分支负责人 Code Review 通过后 Merge
# 6. 合并后清理
git checkout main
git pull origin main
git branch -d feature/user-login
```

**分支命名约定：**

```
feature/xxx    新功能
bugfix/xxx     bug 修复
hotfix/xxx     线上紧急修复
release/xxx    发布分支
docs/xxx       文档改动
refactor/xxx   重构
```

**一次典型的 PR 描述该写什么：**
- 这个 PR 做了什么（背景 Issue：`Closes #12`）
- 怎么测的
- 有没有破坏性变更

> `Closes #12` 这类关键词会自动在 PR 合并时关闭对应 Issue，非常好用。

### 5.4 同步团队其他人的代码（Fork 工作流）

参与开源项目（没有直接 push 权限）时：

```
① GitHub 网页点右上角 Fork，把仓库复制到自己账号下
        ↓
② 把自己账号下的仓库 clone 到本地
        ↓
③ 添加源仓库为 upstream（只需做一次）
   git remote add upstream git@github.com:原作者/仓库.git
        ↓
④ 每次开工前同步源仓库最新代码
   git fetch upstream
   git checkout main
   git merge upstream/main
        ↓
⑤ 在自己分支改完后 push 到自己 Fork 的仓库
        ↓
⑥ GitHub 网页发起 PR 到 源仓库
```

---

## 六、VSCode 里的图形化操作对照表

| 操作 | VSCode 图形界面 | 命令面板 (`Ctrl+Shift+P`) 命令 | 命令行 |
|---|---|---|---|
| 初始化仓库 | 源代码管理面板 → 初始化仓库 | `Git: Initialize Repository` | `git init` |
| 克隆仓库 | 欢迎页 → Clone Git Repository | `Git: Clone` | `git clone <url>` |
| 暂存 | 文件右侧 `+` | `Git: Stage All Changes` | `git add .` |
| 提交 | 顶部输入框 + `Ctrl+Enter` | `Git: Commit` | `git commit -m ""` |
| 撤回文件改动 | 文件右侧 `↺` | `Git: Discard Changes` | `git restore <file>` |
| 取消暂存 | 文件右侧 `−` | `Git: Unstage All Changes` | `git restore --staged <file>` |
| 新建分支 | 左下角分支名 → 新建分支 | `Git: Create Branch` | `git checkout -b xxx` |
| 切换分支 | 左下角分支名 → 选择分支 | `Git: Checkout to` | `git checkout xxx` |
| 推送 | `...` 菜单 → 推送 | `Git: Push` | `git push` |
| 拉取 | `...` 菜单 → 拉取 | `Git: Pull` | `git pull` |
| 对比差异 | 面板双击文件 | `Git: Open Changes` | `git diff` |
| 查看历史 | 需 Git Graph / GitLens 插件 | `Git: View History` | `git log` |
| 暂存改动稍后处理 | 文件右键 → Stash Changes | `Git: Stash` | `git stash` |
| 恢复暂存 | 右键 → Apply Stash | `Git: Apply Stash` | `git stash pop` |
| 合并分支 | `...` 菜单 → 分支 → 合并分支 | `Git: Merge Branch` | `git merge xxx` |
| 绑定远程仓库 | `...` → 远程 → 添加远程 | `Git: Add Remote` | `git remote add origin <url>` |

> **stash（储藏）** 是很实用的功能：手头改了一半要切分支时，先把改动"藏"起来，回来再恢复，避免带着脏代码切换分支。

---

## 七、合并冲突的处理

冲突不是错误，是 Git 在说"同一处两边都改了，你决定留哪个"。

发生时机：`pull`、`merge`、`rebase`、`stash pop`。

**VSCode 里的处理流程（它把这件事做得非常清爽）：**

```
1. 冲突文件出现在「合并更改」区，标记 Conflict
        ↓
2. 双击打开，VSCode 用不同颜色高亮冲突区，并在顶部给出四个按钮：
   ┌──────────────────────────────────────┐
   │  Accept Current Change   ← 保留当前分支 │
   │  Accept Incoming Change  ← 保留合入分支 │
   │  Accept Both Changes     ← 两个都保留   │
   │  Compare Changes         ← 打开对比视图 │
   └──────────────────────────────────────┘
        ↓
3. 逐个冲突块处理（也可以手动编辑为想要的最终内容）
        ↓
4. 全部解决后，把文件移入暂存区（这一步等于告诉 Git "处理完了"）
        ↓
5. 执行一次 commit，冲突解决完成
```

冲突代码块的原始样子：

```
<<<<<<< HEAD
这是当前分支的内容
=======
这是合入进来的内容
>>>>>>> feature/user-login
```

命令行下卡住时的两条退路：
```bash
git merge --abort     # 放弃合并，回到冲突前的状态
git rebase --abort    # 放弃 rebase
```

> **建议**：把 VSCode 设置里的 `Git: Merge Editor` 打开（界面会变成三栏合并视图，左侧当前分支、中间结果、右侧 incoming），处理复杂冲突时效率高很多。

---

## 八、推荐的 VSCode 插件

| 插件 | 作用 |
|---|---|
| **GitLens** | 每行代码显示最后修改者和时间；可视化 FILE HISTORY、分支对比。装它就够了 |
| **Git Graph** | 图形化的提交树，右键节点可直接 cherry-pick / revert / reset，比命令行直观得多 |
| **Git History** | 查看单个文件的完整修改历史 |
| **GitHub Pull Requests and Issues** | 在 VSCode 里直接看和 review PR，不用来回切浏览器 |
| **.gitignore Generator** | 右键一键生成对应语言的 `.gitignore` |
| **Error Lens** | 顺手加的，实时显示错误提示，写 Java 很有用 |

---

## 九、常见问题排查清单

| 症状 | 原因 | 解决 |
|---|---|---|
| VSCode 找不到 Git | git.path 未配置 | 设置里手动指定 git.exe 路径，重启 VSCode |
| push 提示 Permission denied (publickey) | SSH key 没配好或没加到 GitHub | 重走 5.1，确认 `ssh -T git@github.com` 通过 |
| push 提示 authentication failed | 用了 HTTPS 地址 | `git remote set-url origin git@...`（换成 SSH 地址） |
| push 被 reject fetch first | 远端有本地没有的新提交 | 先 `git pull --rebase` 再 push |
| 中文文件名变成 `\344\275\240` 乱码 | 路径编码 | `git config --global core.quotepath false` |
| 提交错了 author | 全局配置邮箱不对 | `git commit --amend --author="名字 <邮箱>"` |
| 误把密钥/密码推上去了 | 没有 `.gitignore` | 立刻在 GitHub 上改密钥值（提交历史里的已泄露），再到 Settings 里轮换所有凭证 |
| 大文件推不上去 | 单文件超 GitHub 100MB 限制 | 用 Git LFS |
| 仓库体积太大 | 历史里有大文件 | `git filter-repo` 清理历史（谨慎操作） |

> **最重要的安全习惯**：任何密码、API Key、私钥文件，进仓库之前先确认它在 `.gitignore` 里。GitHub 上有公开仓库被自动扫描密钥的机器人，泄露后几分钟内就会被利用。

---

## 十、每日工作流口诀

```
① 开工前：git pull                ← 拿到别人最新的代码
② 写代码前：git checkout -b xxx    ← 每个任务一个分支
③ 写完一个完整的小改进：add + commit ← 小步快跑，别攒一大坨
④ 一个任务做完：git push           ← 定时备份到远端
⑤ 任务完成：GitHub 提 PR → Review → Merge → 删分支
⑥ 收工前：git status 看一眼是不是还有没提交的东西
```

---

## 附：最常用命令速查

```bash
# 仓库
git init / git clone <url> / git status / git log --oneline

# 提交
git add . / git add <file> / git commit -m "xxx" / git commit --amend

# 撤销
git restore <file> / git restore --staged <file> / git revert <id> / git reset --hard <id> / git reflog

# 分支
git branch / git checkout -b <name> / git switch <name> / git merge <name> / git branch -d <name>

# 远程
git remote -v / git remote add origin <url> / git push -u origin main / git pull / git fetch

# 储藏
git stash / git stash list / git stash pop / git stash drop

# 查看差异
git diff / git diff --cached / git diff <branch1> <branch2>
```
