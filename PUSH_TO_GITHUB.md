# 推送到 GitHub 指南

## 方法 1: 使用 GitHub CLI (推荐)

### 安装 GitHub CLI
```bash
# Ubuntu/Debian
curl -fsSL https://cli.github.com/packages/githubcli-archive-keyring.gpg | sudo dd of=/usr/share/keyrings/githubcli-archive-keyring.gpg
sudo chmod go+r /usr/share/keyrings/githubcli-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" | sudo tee /etc/apt/sources.list.d/github-cli.list > /dev/null
sudo apt update
sudo apt install gh

# 登录
gh auth login
```

### 创建仓库并推送
```bash
cd /home/admin/RuoYi-Quartz-RCE-Disclosure

# 创建公开仓库
gh repo create RuoYi-Quartz-RCE --public --source=. --remote=origin --push

# 或者创建私有仓库
# gh repo create RuoYi-Quartz-RCE --private --source=. --remote=origin --push
```

## 方法 2: 使用 Git + Personal Access Token

### 步骤 1: 创建 Personal Access Token
1. 访问 https://github.com/settings/tokens
2. 点击 "Generate new token (classic)"
3. 选择权限: `repo` (完整仓库访问)
4. 生成并复制 token

### 步骤 2: 推送代码
```bash
cd /home/admin/RuoYi-Quartz-RCE-Disclosure

# 添加远程仓库 (使用你的 token)
git remote add origin https://M0onc:YOUR_TOKEN@github.com/M0onc/RuoYi-Quartz-RCE.git

# 重命名分支为 main
git branch -M main

# 推送
git push -u origin main
```

## 方法 3: 使用 SSH 密钥

### 步骤 1: 生成 SSH 密钥 (如果没有)
```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
cat ~/.ssh/id_ed25519.pub
```

### 步骤 2: 添加 SSH 密钥到 GitHub
1. 访问 https://github.com/settings/keys
2. 点击 "New SSH key"
3. 粘贴你的公钥

### 步骤 3: 推送代码
```bash
cd /home/admin/RuoYi-Quartz-RCE-Disclosure

git remote add origin git@github.com:M0onc/RuoYi-Quartz-RCE.git
git branch -M main
git push -u origin main
```

## 仓库信息

- **仓库名称:** RuoYi-Quartz-RCE
- **用户名:** M0onc
- **作者署名:** moonsec

## 文件清单

- `README.md` - 漏洞披露报告
- `ruoyi-quartz-rce.yaml` - Nuclei POC 模板
- `LICENSE` - MIT 许可证
- `.gitignore` - Git 忽略文件
