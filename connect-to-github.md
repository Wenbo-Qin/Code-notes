# 🧭 Mac SSH 连接与 GitHub 仓库操作笔记（简明版）

---

## 一、准备工作

    ```bash
    # 查看是否安装 Git
    git --version

    # 配置全局用户名和邮箱
    git config --global user.name "你的GitHub用户名"
    git config --global user.email "你的邮箱地址"
    ```

---

## 二、配置 SSH 连接 GitHub

    ```bash
    # 生成 SSH key
    ssh-keygen -t ed25519 -C "你的GitHub邮箱"

    # 启动 agent 并添加私钥
    eval "$(ssh-agent -s)"
    ssh-add --apple-use-keychain ~/.ssh/id_ed25519

    # 复制公钥
    pbcopy < ~/.ssh/id_ed25519.pub
    ```

    > 打开 GitHub → Settings → SSH and GPG keys → New SSH key → 粘贴保存

    ```bash
    # 测试连接
    ssh -T git@github.com
    ```

    输出：
    ```
    Hi your-username! You've successfully authenticated, but GitHub does not provide shell access.
    ```
    代表 SSH 连接成功。

---

## 三、关联远程仓库

    ```bash
    # 查看当前远程地址
    git remote -v

    # 如果是 https，改成 ssh
    git remote set-url origin git@github.com:你的用户名/仓库名.git

    # 再次确认
    git remote -v
    ```

    输出示例：
    ```
    origin  git@github.com:your-username/your-repo.git (fetch)
    origin  git@github.com:your-username/your-repo.git (push)
    ```

---

## 四、本地项目上传到 GitHub（空仓库）

    ```bash
    # 进入项目目录
    cd ~/Documents/your-project

    # 初始化仓库
    git init

    # 添加忽略文件
    echo ".idea/" >> .gitignore

    # 添加并提交
    git add .
    git commit -m "first commit"

    # 关联远程仓库（SSH 地址）
    git branch -M main
    git remote add origin git@github.com:你的用户名/仓库名.git

    # 推送
    git push -u origin main
    ```

---

## 五、远程已有项目（需要 pull）

    ```bash
    # 克隆远程项目
    git clone git@github.com:你的用户名/仓库名.git
    cd 仓库名

    # 或在已有项目中拉取更新
    git fetch origin
    git pull --rebase origin main
    ```

    > 如果冲突：
    > 1. 修改冲突文件  
    > 2. 执行 `git add .`  
    > 3. 执行 `git rebase --continue`

---

## 六、忽略已上传的文件（例如 .idea）

    ```bash
    echo ".idea/" >> .gitignore
    git add .gitignore
    git rm -r --cached .idea
    git commit -m "remove .idea and add to .gitignore"
    git push origin main
    ```

---

## 七、常用命令速查

| 功能 | 命令 |
|------|------|
| 查看状态 | `git status` |
| 添加文件 | `git add .` |
| 提交 | `git commit -m "说明"` |
| 推送 | `git push origin main` |
| 拉取 | `git pull origin main` |
| 查看远程 | `git remote -v` |
| 修改远程为 SSH | `git remote set-url origin git@github.com:user/repo.git` |
| 撤销修改 | `git restore <file>` |

---

## 八、常见问题

| 问题 | 原因 | 解决方法 |
|------|------|-----------|
| 要输入用户名密码 | 用的是 HTTPS | 改成 SSH 地址 |
| Permission denied | SSH key 未添加 | `ssh-add ~/.ssh/id_ed25519` |
| pull 报错 divergent | 分支不同步 | `git pull --rebase origin main` |
| .idea 上传了 | 未在 .gitignore | 按上面步骤移除 |

---

## 九、建议设置（一次配置）

    ```bash
    # 默认使用 rebase 拉取
    git config --global pull.rebase true

    # 优先使用 SSH
    git config --global url."git@github.com:".insteadOf "https://github.com/"
    ```

---

## 十、总结

- 配置 SSH：生成密钥 → 添加到 GitHub → 测试连接  
- 上传本地项目：`git init → add → commit → remote → push`  
- 拉取远程项目：`git clone` 或 `git pull --rebase`  
- 忽略文件：`.gitignore + git rm --cached`  

保存文件名推荐：`GitHub_SSH_Notes.md`