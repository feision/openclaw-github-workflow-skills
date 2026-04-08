# GitHub Pages 部署完全指南

## ❌ 常见错误：文件已上传却显示 404

**典型场景：**
```bash
# 你做了：
git clone https://github.com/feision/my-portal.git
cd my-portal
echo "<h1>Hello</h1>" > index.html
git add . && git commit -m "add index" && git push origin main

# 访问：
https://feision.github.io/my-portal/  → 404 Not Found
```

**为什么？**
因为你**只上传了文件，但没有启用 GitHub Pages 服务**。

---

## ✅ 正确的完整流程

### 步骤 1：创建/推送文件到仓库
```bash
# 确保 index.html 在仓库的根目录（或 docs/ 目录）
git add index.html
git commit -m "Add index.html"
git push origin main
```

### 步骤 2：启用 GitHub Pages（**关键！**）

#### 方法 A：通过 API（推荐用于自动化）
```bash
# 启用 Pages，指定 main 分支的根目录
curl -X POST \
  -H "Authorization: token $GITHUB_TOKEN" \
  -H "Accept: application/vnd.github.v3+json" \
  -d '{"source":{"branch":"main","path":"/"}}' \
  "https://api.github.com/repos/feision/my-portal/pages"

# 预期响应：
# {
#   "html_url": "https://feision.github.io/my-portal/",
#   "source": {"branch":"main","path":"/"},
#   "public": true
# }
```

#### 方法 B：通过网页手动启用
1. 打开仓库页面 → Settings → Pages
2. Source 选择：
   - Branch: `main`（或 `master`）
   - Folder: `/`（根目录）或 `/docs`
3. 点击 Save
4. 页面显示 "Your site is published at https://..."

### 步骤 3：等待构建完成
- **时间**：30秒 - 3分钟
- **状态检查**：
  ```bash
  # API 查看构建状态
  curl -H "Authorization: token $TOKEN" \
    "https://api.github.com/repos/feision/my-portal/pages/builds"
  ```
- **不要**立即刷新页面，稍等片刻

### 步骤 4：访问验证
```
https://feision.github.io/my-portal/
```

---

## 🔧 API 完整示例

### 创建仓库并启用 Pages（一次性脚本）
```bash
TOKEN="your_token"
REPO="my-portal"
OWNER="feision"

# 1. 创建仓库
curl -X POST -H "Authorization: token $TOKEN" \
  -d "{\"name\":\"$REPO\",\"private\":false}" \
  "https://api.github.com/user/repos"

# 2. 上传 index.html（通过 Git 或 API）
# 此处略...

# 3. 启用 GitHub Pages
curl -X POST -H "Authorization: token $TOKEN" \
  -H "Accept: application/vnd.github.v3+json" \
  -d "{\"source\":{\"branch\":\"main\",\"path\":\"/\"}}" \
  "https://api.github.com/repos/$OWNER/$REPO/pages"

# 4. 等待后验证
sleep 60
curl -I "https://$OWNER.github.io/$REPO/"
# 应返回 HTTP 200
```

---

## 🚨 必须避免的陷阱

| 陷阱 | 后果 | 如何避免 |
|------|------|----------|
| **只上传文件，不启用 Pages** | 404 Not Found | ✅ 必须调用 Pages API 或手动配置 |
| **启用 Pages 后立即访问** | 404（构建中） | ⏳ 等待 1-3 分钟 |
| **source.branch 配置错误** | 404 或空白页 | 确保分支存在且包含 `index.html` |
| **source.path 配置为 /docs 但文件在根目录** | 404 | 统一路径：文件放到 `docs/` 或改回 `/` |
| **私有仓库未开启 Pages** | 404 | 私有仓库需要额外权限配置 |
| **自定义域名未设置 CNAME** | 404 或证书错误 | 在 Pages 设置中添加 CNAME 文件 |

---

## 📊 诊断清单（遇到 404 时）

按顺序检查：

- [ ] **1. 确认文件存在**
  ```bash
  # 访问 raw 链接
  https://raw.githubusercontent.com/feision/my-portal/main/index.html
  ```
  如果 404，说明文件没上传对位置。

- [ ] **2. 确认 Pages 已启用**
  ```bash
  curl -H "Authorization: token $TOKEN" \
    "https://api.github.com/repos/feision/my-portal/pages"
  ```
  应返回 `200 OK` 并包含 `html_url`。

- [ ] **3. 确认 Pages 源配置正确**
  - `source.branch` 是文件所在分支（通常是 `main`）
  - `source.path` 是文件所在目录（`/` 或 `docs`）

- [ ] **4. 等待构建完成**
  - 检查构建状态：`GET /repos/:owner/:repo/pages/builds`
  - 最新一条 `status` 应为 `"built"`

- [ ] **5. 清除缓存测试**
  - 手机/Safari 清除缓存
  - 或使用无痕模式访问

---

## 🔄 常见问题

### Q1：为什么我手动上传了文件，Pages 还是 404？
**A：** GitHub Pages 是独立服务，默认关闭。必须通过 Settings → Pages 或 API 显式启用。

### Q2：我已经启用 Pages 了，为什么还是 404？
**A：** 可能原因：
- 正在构建中（等待 1-3 分钟）
- `index.html` 不在指定的 `source.path` 目录下
- 仓库是私有且未授权访问

### Q3：如何通过 API 关闭 Pages？
**A：**
```bash
curl -X DELETE \
  -H "Authorization: token $TOKEN" \
  "https://api.github.com/repos/feision/my-portal/pages"
```

### Q4：如何修改 Pages 的源分支/目录？
**A：**
```bash
curl -X POST -H "Authorization: token $TOKEN" \
  -d '{"source":{"branch":"main","path":"/"}}' \
  "https://api.github.com/repos/feision/my-portal/pages"
```

---

## 📝 最佳实践 SOP

**每次部署新项目到 GitHub Pages 的检查清单：**

```
□ 步骤 1: index.html 放在仓库根目录
□ 步骤 2: 推送到 main 分支
□ 步骤 3: 调用 Pages API 启用服务
□ 步骤 4: 等待 60-180 秒
□ 步骤 5: 访问 https://username.github.io/repo-name/ 验证
```

**自动化脚本模板：**
```bash
#!/bin/bash
# deploy-to-pages.sh
REPO="$1"
TOKEN="$2"

git add . && git commit -m "Deploy" && git push origin main

# 启用 GitHub Pages
curl -X POST -H "Authorization: token $TOKEN" \
  -H "Accept: application/vnd.github.v3+json" \
  -d '{"source":{"branch":"main","path":"/"}}' \
  "https://api.github.com/repos/feision/$REPO/pages"

echo "✅ Pages 已启用，等待 2 分钟后访问："
echo "https://feision.github.io/$REPO/"
```

---

## 💡 核心要点

**记住：文件上传和 Pages 启用是两个独立的操作。**

- ✅ **上传文件**：`git push` 或 GitHub API
- ✅ **启用 Pages**：Settings → Pages 或 `POST /repos/:owner/:repo/pages`
- ✅ **等待构建**：GitHub 需要时间生成静态页面
- ✅ **验证**：检查 `pages/builds` API 状态

**最常见的错误：** 以为推送到 `main` 分支就完事了，实际上 Pages 服务根本没开。

---

**下次遇到 404，先问自己：我启用 GitHub Pages 了吗？**
