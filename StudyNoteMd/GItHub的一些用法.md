# GItHub的一些用法

## 将已有项目创建并上传

### 1. 创建Git仓库

在你的项目目录下初始化 Git 仓库：

```bash
cd /path/to/your/project  # 进入你的项目目录
git init  # 初始化一个新的 Git 仓库
```

### 2. 创建`.gitignore`文件(可选)

如果你的项目中有一些文件或目录不需要上传到 GitHub（如临时文件、依赖文件等），可以在项目根目录下创建 `.gitignore` 文件，指定要忽略的文件。

例如，忽略 `node_modules`、`*.log` 文件等：

```bash
echo "node_modules/" >> .gitignore
echo "*.log" >> .gitignore
```

### 3. 添加文件到Git仓库

将项目中的所有文件添加到 Git 仓库：

```bash
git add .  # 添加所有文件
```

### 4. 提交更改

提交这些文件到本地 Git 仓库：

```bash
git commit -m "Initial commit"  # 提交并添加提交信息
```

### 5. 修改Linux的master(可选)

1. 查看当前分支名称

   ```bash
   git branch
   ```

2. 将 `master` 重命名为 `main`

   ```bash
   git branch -m master main
   ```

### 6. 连接本地仓库与GitHub仓库

在 GitHub 上创建完仓库后，会提供一个仓库的 URL。

在本地仓库中设置远程仓库并推送到 GitHub：

```bash
git remote add origin https://github.com/yourusername/my-project.git  # 设置远程仓库
git branch -M main  # 确保分支是 "main"（如果是 master 则跳过此步）
git push -u origin main  # 将本地仓库推送到 GitHub
```

### 7. 设置全局默认分支为 `main`（可选）

如果想让所有新创建的 Git 仓库默认分支为 `main`，可以修改 Git 的全局配置：

```bash
git config --global init.defaultBranch main
```

## Git证书

### **MIT License**

- **特点**: 非常宽松，允许使用、修改和分发代码，只需保留原始作者的版权声明。
- 适用场景:
  - 适合希望最大限度传播代码的项目。
  - 开发库、工具或框架，鼓励别人自由使用。
- **限制**: 不提供任何担保，作者不对代码使用后果负责。

### **Apache License 2.0**

- **特点**: 允许使用、修改和分发代码，要求保留版权声明，并且对修改后的版本需记录变更。
- 适用场景:
  - 企业级项目和开源软件，提供专利授权条款。
  - 想确保贡献者保留专利权并不被起诉。
- **限制**: 需要清楚声明所包含的文件和变更。

### **GPL (GNU General Public License)**

- **特点**: 强制开源，所有衍生项目必须使用相同的许可证发布。
- 适用场景:
  - 希望确保代码及其衍生品始终开源。
  - 社区驱动项目。
- **限制**: 对于闭源项目或商业用途限制较大。

### **BSD License (2-Clause or 3-Clause)**

- **特点**: 类似于 MIT，但有更具体的声明条款。
- 适用场景:
  - 适用于学术或研究项目。
  - 希望代码在保持宽松的同时限制滥用作者名义。
- **限制**: 与 MIT 类似，但增加了“不能用于宣传作者”的条款。

### **Unlicense**

- **特点**: 彻底放弃版权，将代码归于公共领域。
- 适用场景:
  - 希望完全开放代码使用权。
  - 不关心贡献者、衍生品或改动历史。
- **限制**: 任何人都可以随意使用、修改代码，没有任何要求。

### **如何选择**

1. **希望宽松开放**: 选择 **MIT** 或 **Apache 2.0**。

2. **希望强制开源**: 选择 **GPL**。

3. **学术或研究项目**: 选择 **BSD**。

4. **完全放弃版权**: 选择 **Unlicense**。

5. **企业项目**: 通常选择 **Apache 2.0**。

