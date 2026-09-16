# GitHub 代码推送标准操作指南

## 1. 凭证准备 (Personal Access Token)
GitHub 已全面禁用密码推送，必须使用 Token (PAT) 进行身份验证。
1. **获取路径**：网页端依次点击 `Settings` -> `Developer settings` -> `Personal access tokens` -> `Tokens (classic)` -> `Generate new token (classic)`。
2. **关键权限**：务必勾选 **`repo`** 选项（赋予仓库完整的读写权限）。
3. **保存凭证**：点击生成后，务必立即复制以 `ghp_` 开头的字符串备用。

---

## 2. 首次上传全新工程
进入你的代码工程根目录，依次执行以下命令：

```bash
# 进入工程文件夹
cd /home/maxonskyler/MAC-Net/
# 1. 初始化本地 Git 仓库
git init

# 2. 将所有文件添加到暂存区（注意末尾的点号）
git add .

# 3. 提交并添加描述信息
git commit -m "Initial commit"

# 4. 关联远程仓库（采用 Token 内嵌 URL 的方式，实现免密验证）
# 替换下方命令中的 Token 字符串
git remote add origin https://<你的Token>@[github.com/field-mx/MAC-NET.git](https://github.com/field-mx/MAC-NET.git)

# 5. 将本地默认分支重命名为 main（匹配 GitHub 标准）
git branch -M main

# 6. 推送本地代码到远程仓库，并建立追踪关系
git push -u origin main
```

---

## 3.更新了代码，再次提交

```bash
# 1. 将所有修改、新增或删除的文件添加到暂存区
git add .

# 2. 提交本次改动，并在引号内填入简明的修改说明
git commit -m "描述你具体修改了什么内容"

# 3. 直接推送到 GitHub 远程仓库
git push
```
