---
name: "git-push"
description: "将当前项目推送到 GitHub 仓库 (h15958037493-cell/kangzhi-health-agent)。当用户要求推送到 GitHub、推送代码、上传代码时调用。"
---

# Git Push to GitHub

推送当前项目到 GitHub 仓库。**当用户说"推送到github"、"推送代码"、"push"、"上传到github"时必须调用此 skill。**

## 目标仓库

```
https://github.com/h15958037493-cell/kangzhi-health-agent.git
```

## 执行流程

### 第一步：检查 Git 仓库

```powershell
git status
```

如果报错 `fatal: not a git repository`，则初始化：

```powershell
git init
git remote add origin https://github.com/h15958037493-cell/kangzhi-health-agent.git
```

### 第二步：检查 Git 用户配置

```powershell
git config user.name
git config user.email
```

如果为空，设置：

```powershell
git config user.name "h15958037493-cell"
git config user.email "h15958037493@users.noreply.github.com"
```

### 第三步：添加并提交

```powershell
git add .
git commit -m "<简要描述本次修改>"
```

注意：PowerShell 分号用 `;` 而非 `&&`：

```powershell
git add .; git commit -m "提交信息"
```

### 第四步：推送

```powershell
git branch -M main
git push -u origin main
```

## 推送失败处理

### 报错 `Failed to connect to github.com` 或 `Connection was reset`

当前环境无法直连 GitHub，执行以下降级方案。

#### 方案一：尝试设置代理（如果用户有代理）

```powershell
git config --global http.proxy http://127.0.0.1:<端口>
git config --global https.proxy http://127.0.0.1:<端口>
```

推完后清除代理：

```powershell
git config --global --unset http.proxy
git config --global --unset https.proxy
```

#### 方案二：使用 SSH（如果已配置 SSH key）

```powershell
git remote set-url origin git@github.com:h15958037493-cell/kangzhi-health-agent.git
git push -u origin main
```

#### 方案三：使用 GitHub Personal Access Token

```powershell
$token = "<用户提供的 token>"
$encodedToken = [Convert]::ToBase64String([Text.Encoding]::ASCII.GetBytes(":$token"))
git -c http.extraHeader="Authorization: Basic $encodedToken" push -u origin main
```

或者临时设置：

```powershell
git remote set-url origin https://<token>@github.com/h15958037493-cell/kangzhi-health-agent.git
git push -u origin main
```

#### 方案四：所有方案都失败

告知用户：当前环境无法连接 GitHub，请手动在本地终端执行 `git push`。

## 分支命名规则

默认使用 `main` 分支。如果远程仓库使用 `master`，改用：

```powershell
git branch -M master
git push -u origin master
```

先用 `git ls-remote origin` 探测远程分支情况。