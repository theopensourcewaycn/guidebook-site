# mdBook 网站和 PDF 构建

本仓库使用 mdBook 构建开源之道指南的网站和 PDF 文档。

## 文件说明

- `book.toml` - mdBook 配置文件
- `.github/workflows/build.yml` - 主库的构建工作流
- `content/` - 内容子仓库（来自 guidebook 仓库）
- `content/.github/workflows/trigger-parent.yml` - 子仓库的触发工作流

## 自动构建

### 主库构建触发条件

- 推送到 main 分支
- 创建 Pull Request
- 子仓库内容更新（通过 repository_dispatch）
- 手动触发

### 构建输出

- **网站**: 自动部署到 GitHub Pages
- **PDF**: 作为 Release 附件发布

## 配置步骤

### 1. 主库配置

主库（guidebook-site）已配置完成，包括：
- ✅ `book.toml` - mdBook 配置
- ✅ `.github/workflows/build.yml` - 构建工作流

### 2. 子仓库配置

在 guidebook 仓库需要：

1. 添加 `.github/workflows/trigger-parent.yml`（已创建）
2. 在 guidebook 仓库设置中添加 Secret：
   - 名称: `PARENT_REPO_TOKEN`
   - 值: 具有 `repo` 权限的 GitHub Personal Access Token

### 3. 主库 GitHub Pages 配置

在 guidebook-site 仓库设置中：
1. 进入 Settings → Pages
2. Source 选择 "Deploy from a branch"
3. Branch 选择 `gh-pages` 分支，目录选择 `/ (root)`

### 4. 创建 GitHub Token

为子仓库触发主库构建，需要创建 Personal Access Token：

1. 访问 GitHub Settings → Developer settings → Personal access tokens → Tokens (classic)
2. 点击 "Generate new token (classic)"
3. 设置权限:
   - 勾选 `repo` (完整仓库访问权限)
4. 生成后复制 token
5. 在 guidebook 仓库添加 Secret:
   - Settings → Secrets and variables → Actions → New repository secret
   - Name: `PARENT_REPO_TOKEN`
   - Value: 粘贴刚才的 token

## 工作流程

1. 当 guidebook 仓库（内容仓库）有新提交时
2. 触发 `trigger-parent.yml` 工作流
3. 通过 API 触发 guidebook-site 仓库的 `repository_dispatch` 事件
4. guidebook-site 执行构建：
   - 检出代码（包含子仓库）
   - 安装 mdBook 和 mdbook-pdf
   - 构建 HTML 网站
   - 构建 PDF 文档
   - 部署网站到 GitHub Pages
   - 发布 PDF 到 Release

## 本地测试

```bash
# 安装 mdBook
cargo install mdbook mdbook-pdf

# 克隆仓库（包含子模块）
git clone --recursive https://github.com/theopensourcewaycn/guidebook-site.git
cd guidebook-site

# 构建并预览
mdbook serve

# 构建 PDF
mdbook-pdf --standalone true
```

## 故障排查

- 如果子仓库触发失败，检查 `PARENT_REPO_TOKEN` 是否正确配置
- 如果 PDF 构建失败，检查 mdbook-pdf 的依赖是否安装完整
- 如果网站部署失败，检查 GitHub Pages 设置是否正确
