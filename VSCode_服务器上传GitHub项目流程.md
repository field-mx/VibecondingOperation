# 在服务器上使用 VS Code 图形界面上传新项目到 GitHub

> 适用场景：使用 VS Code Remote-SSH 打开 Linux 服务器上的项目，希望通过 VS Code 的“源代码管理”界面完成 Git 初始化、提交和发布到 GitHub。

---

## 1. 用 VS Code 打开目标项目文件夹

在 VS Code 中：

```text
文件 → 打开文件夹
```

只打开**真正要上传的项目根目录**，例如：

```text
/home/maxon/workspace/VIMTS
```

不要打开更外层的：

```text
/home/maxon/workspace
```

否则可能把其他项目一起纳入 Git。

---

## 2. 创建 `.gitignore`

在项目根目录：

```text
右键 → 新建文件 → .gitignore
```

科研 / AI 项目可直接使用：

```gitignore
# 图片 / 文档
*.png
*.pdf

# 数据
*.csv
*.xlsx
*.xls
*.npy
*.npz
*.mat
*.h5
*.hdf5
*.pkl
*.pickle
*.jsonl

data/
dataset/
datasets/

# 模型权重
*.pt
*.pth
*.ckpt
*.safetensors

# 训练输出
outputs/
checkpoints/
logs/
wandb/

# Python 缓存
__pycache__/
*.pyc

# IDE
.vscode/
.idea/
```

保存：

```text
Ctrl + S
```

> `.gitignore` 最好在第一次提交前创建。

---

## 3. 初始化 Git 仓库

点击左侧：

```text
源代码管理
```

快捷键：

```text
Ctrl + Shift + G
```

如果项目还不是 Git 仓库，点击：

```text
初始化存储库 / Initialize Repository
```

---

## 4. 检查待提交文件

初始化后，在：

```text
源代码管理 → 更改 / Changes
```

检查文件列表。

确认：

- 代码文件正常出现；
- `*.png`、`*.pdf`、数据集、模型权重等被 `.gitignore` 忽略；
- 没有误加入其他项目。

常见状态：

```text
U = 未跟踪文件
M = 已修改文件
D = 已删除文件
```

---

## 5. 暂存文件

在：

```text
更改 / Changes
```

右侧点击：

```text
+
```

即可暂存全部修改。

也可以只对某个文件点击 `+`。

暂存后文件会进入：

```text
暂存的更改 / Staged Changes
```

---

## 6. 配置 Git 提交身份（首次使用时）

如果提交时出现：

```text
请确保已在 Git 中配置你的 "user.name" 和 "user.email"
```

在 VS Code 中按：

```text
Ctrl + `
```

打开集成终端。

只为当前项目配置：

```bash
git config user.name "你的GitHub用户名"
git config user.email "你的GitHub邮箱"
```

例如：

```bash
git config user.name "field-mx"
git config user.email "your_email@example.com"
```

检查：

```bash
git config user.name
git config user.email
```

> `user.name` 和 `user.email` 是 Commit 作者信息，不等于 GitHub 登录认证。

---

## 7. 创建第一次 Commit

回到：

```text
源代码管理
```

在顶部消息框输入：

```text
Initial commit
```

然后点击：

```text
提交 / Commit
```

提交成功后，可以在底部“图表 / Graph”中看到：

```text
Initial commit
```

以及当前分支：

```text
main
```

---

## 8. 登录 GitHub

如果尚未登录 GitHub：

```text
VS Code 账户图标
→ Sign in with GitHub
```

浏览器会打开 GitHub：

```text
登录 GitHub
→ 授权 VS Code
→ 返回 VS Code
```

使用这种方式时，一般无需手动配置 SSH Key。

---

## 9. 发布项目到 GitHub

### 推荐方式

按：

```text
Ctrl + Shift + P
```

搜索：

```text
GitHub: Publish to GitHub
```

选择后设置：

```text
仓库名：VIMTS
```

然后选择：

```text
Publish to GitHub public repository
```

或：

```text
Publish to GitHub private repository
```

VS Code 会自动完成：

```text
本地项目
→ 创建 GitHub 仓库
→ 建立远程 origin
→ 上传 main 分支
```

---

## 10. 不要提前创建同名空仓库

如果准备使用：

```text
GitHub: Publish to GitHub
```

推荐**不要提前在 GitHub 网页创建同名空仓库**。

否则 VS Code 可能遇到：

```text
The repository exists, but it contains no Git content.
```

或：

```text
Git: Not Found
.../repos/forks#create-a-fork
```

最简单的流程是：

```text
本地创建项目
→ VS Code 初始化 Git
→ Commit
→ Publish to GitHub
→ 让 VS Code 自动创建远程仓库
```

---

## 11. 如果本地残留了错误的远程仓库

如果之前手动配置过错误的 `origin`，可在 VS Code 中：

```text
Ctrl + Shift + P
→ Git: Remove Remote
→ origin
```

删除旧的远程连接。

然后重新：

```text
Ctrl + Shift + P
→ GitHub: Publish to GitHub
```

---

## 12. 避免“仓库套仓库”

如果 VS Code / Git 提示：

```text
warning: adding embedded git repository
```

说明项目内部还有另一个 `.git`：

```text
项目根目录/
├── .git
└── 子目录/
    └── .git
```

如果子目录只是普通代码而不是独立 Git 仓库，应移除子目录自己的 `.git` 后再提交。

操作前先确认目录层级，避免误删真正需要保留的 Git 仓库。

---

# 日常更新流程

以后修改代码后，只需要：

```text
VS Code → 源代码管理
        ↓
查看 Changes
        ↓
点击 + 暂存
        ↓
填写 Commit 信息
        ↓
Commit
        ↓
Push / Sync Changes
```

例如：

```text
修改模型代码
→ Stage
→ Commit: Add channel attention module
→ Sync Changes
```

---

# 最简流程速查

```text
1. VS Code 打开项目根目录
2. 创建 .gitignore
3. 源代码管理 → Initialize Repository
4. 检查 Changes
5. Stage Changes
6. 配置 user.name / user.email（首次）
7. Commit
8. 登录 GitHub
9. Ctrl + Shift + P
10. GitHub: Publish to GitHub
11. 选择 Public / Private
12. 上传完成
```

---

## 推荐原则

```text
一个项目 = 一个 Git 仓库
一个仓库 = 一个明确的项目根目录
先写 .gitignore，再第一次 Commit
优先让 VS Code 自动创建 GitHub 仓库
日常只用 Stage → Commit → Sync
```
