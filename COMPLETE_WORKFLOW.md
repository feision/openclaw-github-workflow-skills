---

## 📝 实战记录（2025-04-05）

> 记录一次真实的 openclaw 仓库搭建过程，包括踩坑和解决方案。

### 场景

需要创建 `feision/openclaw` 仓库，采用 `main`（默认）+ `openclaw`（开发）双分支策略。本地已有完整 workspace 文件，目标是上传到 GitHub。

### 我的最初错误

1. **先创建 `openclaw` 再创建 `main`** → PR 失败（"No commits between main and openclaw"）
2. **分支名不一致**（本地 `master` vs 远程 `main`）
3. **使用孤儿分支** → 无共同历史

### 正确的最终流程（测试成功）

#### 步骤 1: 创建空仓库（GitHub UI）

- 访问 https://github.com/new
- 仓库名: `openclaw`
- **不要勾选** "Initialize this repository with a README"
- 创建 → 远程默认分支为 `main`

#### 步骤 2: 本地创建 `main` 占位并推送

```bash
mkdir openclaw-upload && cd openclaw-upload
git init
git branch -m main          # 关键：确保本地分支名与远程一致
echo "# OpenClaw" > README.md
git add . && git commit -m "Initial main - placeholder"
git remote add origin https://github.com/feision/openclaw.git
git push -u origin main     # ✅ 先推 main
```

#### 步骤 3: 从 `main` 创建 `openclaw` 分支

```bash
git checkout -b openclaw
```

#### 步骤 4: 上传完整文件（关键：保护 .git 目录）

```bash
# 安全方法：使用临时目录过滤 .git
TMPDIR=$(mktemp -d)
cp -r /home/node/.openclaw/workspace/* "$TMPDIR/"
find "$TMPDIR" -name ".git" -type d -exec rm -rf {} +
cp -r "$TMPDIR"/* .
rm -rf "$TMPDIR"

# 或者：克隆后直接替换（推荐）
git clone https://github.com/feision/openclaw.git workspace
cd workspace
git checkout -b openclaw
# 删除除 .git 外的所有文件
find . -mindepth 1 -maxdepth 1 ! -name '.git' -exec rm -rf {} \;
# 复制完整 workspace 文件过来
cp -r /home/node/.openclaw/workspace/* .
find . -name ".git" -type d -exec rm -rf {} + 2>/dev/null  # 删除子目录的 .git
git add . && git commit -m "feat: 上传完整工作空间"
git push origin openclaw
```

#### 步骤 5: 创建 PR 并合并

- GitHub UI: https://github.com/feision/openclaw 会自动提示 "Compare & pull request"
- base: `main`, compare: `openclaw`
- 填写描述，创建 PR
- 审查后点击 "Merge pull request"

#### 关键命令验证

```bash
# 验证分支关系
git log --oneline main..openclaw   # 应有输出
git diff main...openclaw --stat    # 应显示新增文件

# 验证远程分支
git ls-remote origin
```

### 结果

✅ 仓库地址: https://github.com/feision/openclaw  
✅ 默认分支: `main`  
✅ 开发分支: `openclaw`（小写）  
✅ PR 成功创建并合并

### 注意事项

1. **分支顺序不能颠倒**：必须先有 `main`，才能创建 `openclaw` 并 PR
2. **分支名一致**：本地如果是 `master` 必须改成 `main`（或配置 GitHub 默认分支）
3. **避免嵌套 .git**：复制文件后务必删除子目录的 `.git`，否则会变成子模块
4. **保持 `.git` 完整**：操作时只删除**子目录**的 `.git`，顶层 `.git` 不能动
5. **验证后再 PR**：用 `git log main..openclaw` 确认有差异再创建 PR

---

## 📋 版本控制与上传规范

> ⚠️ **重要原则**：上传代码时，先检查主分支（main）和子分支（openclaw）是否一致，用 OpenClaw 子分支形式上传，再自动合并到 main 主仓库，上传者使用**龙虾 (Lobster)** identity 标记，便于版本历史追溯。
>
> 适用于所有 OpenClaw 教程仓库的维护工作。

### ✅ 分支策略标准

- **主分支**: `main` - 稳定、可发布的版本（默认分支）
- **开发分支**: `openclaw` - 日常修改和新增内容
- **工作流**: 在 `openclaw` 分支工作 → PR → 合并到 `main`

> ⚠️ **注意**：所有仓库必须使用 `main` 作为主分支名称，不要使用 `master`。上传前确保本地和远程的默认分支都是 `main`。

### 🔍 上传前检查清单

1. **主分支与子分支内容一致性检查**
   ```bash
   # 确保 openclaw 分支包含 main 的所有提交
   git checkout main
   git pull origin main
   git checkout openclaw
   git merge main --ff-only  # 应该成功，无冲突
   ```

2. **工作区洁净度检查**
   ```bash
   git status  # 应显示 'nothing to commit'
   git diff HEAD~1..HEAD --stat  # 确认本次提交内容
   ```

3. **移除嵌套 .git 目录**（如果复制了工作空间）
   ```bash
   find . -name ".git" -type d -not -path "./.git/*" -exec rm -rf {} + 2>/dev/null || true
   ```

### 🚀 OpenClaw 子分支上传流程

> 使用 OpenClaw 工具链自动化上传（避免手动错误）

1. **准备空仓库**（GitHub UI）
   - 不初始化 README
   - 默认分支为 `main`

2. **本地初始化并推送 main**
   ```bash
   git init
   git branch -m main
   git remote add origin <repo-url>
   git add . && git commit -m "Initial commit - placeholder"
   git push -u origin main
   ```

3. **创建 openclaw 分支**
   ```bash
   git checkout -b openclaw
   ```

4. **上传实际内容**
   - 复制完整工作空间文件
   - **必须**删除子目录的 `.git`（防止嵌套）
   - 提交并推送

5. **创建 PR 并合并**
   - GitHub UI 自动提示 Compare & pull request
   - base: `main`, compare: `openclaw`
   - 审查后合并

### 🏷️ 提交者标识规范

所有自动生成的提交**必须使用以下身份**，便于版本历史追溯：

```bash
git config user.name "龙虾 (Lobster)"
git config user.email "lobster@openclaw.ai"
```

若使用 OpenClaw 工具上传，会在内部自动应用此标识。

### 📝 提交消息模板

```
type: short description

optional longer description

- 关键变更点
- 关联的教程或问题

Co-authored-by: 龙虾 (Lobster) <lobster@openclaw.ai>
```

常用 `type`: `feat`（新功能）、`docs`（文档）、`fix`（修复）、`chore`（维护）

---

**本次实战验证了 COMPLETE_WORKFLOW.md 中的流程，确保可一次成功。** 🎯
