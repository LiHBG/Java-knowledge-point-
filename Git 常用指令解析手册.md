# Git 常用指令解析手册

## 目录
1. [配置与初始化](#配置与初始化)
2. [基础操作](#基础操作)
3. [分支管理](#分支管理)
4. [远程仓库](#远程仓库)
5. [撤销与恢复](#撤销与恢复)
6. [查看信息](#查看信息)
7. [高级操作](#高级操作)
8. [注意事项](#注意事项)

---

## 配置与初始化

### **`git config`**
```bash
git config --global user.name "Your Name"
git config --global user.email "email@example.com"
```

- **用途**：配置全局用户信息（用户名、邮箱）。
- **重点**：`--global` 表示全局配置，不加则为当前仓库配置。

### **`git init`**

```bash
git init
```

- **用途**：初始化本地仓库，生成 `.git` 目录。

------

## 基础操作

### **`git add`**

```
git add <file>       # 添加单个文件
git add .            # 添加所有修改和新文件
```

- **用途**：将工作区修改添加到暂存区（Stage）。
- **重点**：`.` 表示当前目录下所有文件。

### **`git commit`**

```
git commit -m "commit message"
```

- **用途**：将暂存区内容提交到本地仓库。
- **重点**：`-m` 直接附加提交信息，不加会进入编辑器。

------

## 分支管理

### **`git branch`**

```
git branch                  # 查看本地分支
git branch <branch-name>    # 创建新分支
git branch -d <branch-name> # 删除分支
```

- **用途**：分支的创建、查看和删除。

### **`git checkout`** / **`git switch`**

```
git checkout <branch-name>      # 切换分支
git checkout -b <new-branch>    # 创建并切换分支
git switch <branch-name>        # (Git 2.23+) 更安全的切换方式
```

- **重点**：`-b` 用于创建并切换分支。

### **`git merge`**

```
git merge <branch-name>
```

- **用途**：将指定分支合并到当前分支。
- **注意**：可能产生冲突，需手动解决。

------

## 远程仓库

### **`git clone`**

```
git clone <remote-url>
```

- **用途**：克隆远程仓库到本地。

### **`git remote`**

```
git remote -v                  # 查看远程仓库地址
git remote add origin <url>    # 添加远程仓库
```

- **重点**：`origin` 是远程仓库的默认别名。

### **`git push`**

```
git push origin main
git push -u origin main        # 首次推送时关联分支
```

- **用途**：将本地提交推送到远程仓库。
- **重点**：`-u` 简化后续推送命令。

### **`git pull`**

```
git pull origin main
```

- **用途**：拉取远程仓库最新内容并合并（相当于 `git fetch` + `git merge`）。

------

## 撤销与恢复

### **`git reset`**

```
git reset --soft HEAD^    # 撤销 commit，保留修改到暂存区
git reset --hard HEAD^    # 彻底回退到上一个版本
```

- **重点**：`--hard` 会丢弃工作区修改，慎用！

### **`git checkout -- <file>`**

```
git checkout -- README.md
```

- **用途**：撤销工作区的未暂存修改。

------

## 查看信息

### **`git status`**

```
git status
```

- **用途**：查看工作区和暂存区状态。

### **`git log`**

```
git log --oneline    # 简洁版提交历史
git log --graph      # 图形化显示分支合并
```

------

## 高级操作

### **`git stash`**

```
git stash        # 临时保存工作区修改
git stash pop    # 恢复最近一次保存的内容
```

- **用途**：处理临时任务时保存当前进度。

### **`git rebase`**

```
git rebase main
```

- **用途**：将当前分支的提交“变基”到目标分支，保持提交历史线性。
- **风险**：可能破坏提交历史，需谨慎使用。

------

## 注意事项

1. **慎用 `--force`**：强制推送可能覆盖他人提交。
2. **频繁提交**：小步提交，避免大范围代码丢失。
3. **分支保护**：主分支（如 `main`）应设置权限防止直接推送。