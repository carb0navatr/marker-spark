# Marker Fork 同步工作流程

## 仓库配置

当前仓库有两个 remote:
- `upstream`: 上游开源项目 https://github.com/datalab-to/marker.git
- `origin`: 您的 fork (需要手动添加)

## 初始设置

### 1. 添加您的 fork 作为 origin
```bash
git remote add origin https://github.com/YOUR_USERNAME/marker.git
```

### 2. 首次推送到您的 fork
```bash
git push -u origin master
```

### 3. 验证 remotes 配置
```bash
git remote -v
# 应该看到:
# origin    https://github.com/YOUR_USERNAME/marker.git (fetch)
# origin    https://github.com/YOUR_USERNAME/marker.git (push)
# upstream  https://github.com/datalab-to/marker.git (fetch)
# upstream  https://github.com/datalab-to/marker.git (push)
```

## 日常同步工作流程

### 从上游同步最新更新

每当上游项目有更新时,按以下步骤同步:

#### 方法 1: 直接合并到 master (推荐用于简单场景)

```bash
# 1. 切换到 master 分支
git checkout master

# 2. 从上游获取最新更改
git fetch upstream

# 3. 合并上游的 master 分支
git merge upstream/master

# 4. 解决可能的冲突(如果有)
# 编辑冲突文件,然后:
# git add <冲突文件>
# git commit

# 5. 推送到您的 fork
git push origin master
```

#### 方法 2: 使用 rebase (保持提交历史更清晰)

```bash
# 1. 切换到 master 分支
git checkout master

# 2. 从上游获取最新更改
git fetch upstream

# 3. rebase 到上游的 master 分支
git rebase upstream/master

# 4. 解决可能的冲突(如果有)
# 编辑冲突文件,然后:
# git add <冲突文件>
# git rebase --continue

# 5. 强制推送到您的 fork (注意: rebase 会改变历史)
git push origin master --force-with-lease
```

#### 方法 3: 使用功能分支 (推荐用于有自定义修改的场景)

```bash
# 1. 从 master 创建同步分支
git checkout master
git checkout -b sync-upstream-YYYYMMDD

# 2. 获取上游更新
git fetch upstream

# 3. 合并上游更改
git merge upstream/master

# 4. 解决冲突并测试

# 5. 推送同步分支
git push origin sync-upstream-YYYYMMDD

# 6. 在 GitHub 上创建 Pull Request 将同步分支合并到 master
# 这样可以在合并前进行代码审查

# 7. PR 合并后,更新本地 master
git checkout master
git pull origin master
```

## 开发自己的功能

### 创建功能分支
```bash
# 基于最新的 master 创建功能分支
git checkout master
git pull origin master
git checkout -b feature/your-feature-name
```

### 开发和提交
```bash
# 进行修改
git add .
git commit -m "描述您的更改"

# 推送到您的 fork
git push origin feature/your-feature-name
```

### 合并功能分支
```bash
# 在 GitHub 上创建 Pull Request 合并到您 fork 的 master 分支
# 或者本地合并:
git checkout master
git merge feature/your-feature-name
git push origin master
```

## 保持您的修改同时同步上游更新

如果您有自己的修改,同时想保持与上游同步:

```bash
# 1. 确保您的修改在功能分支上
git checkout feature/your-feature

# 2. 更新 master 到最新的上游版本
git checkout master
git fetch upstream
git merge upstream/master
git push origin master

# 3. 将上游更新 rebase 到您的功能分支
git checkout feature/your-feature
git rebase master

# 4. 解决冲突并推送
git push origin feature/your-feature --force-with-lease
```

## 查看上游和 fork 的差异

```bash
# 查看上游有哪些新提交
git fetch upstream
git log HEAD..upstream/master --oneline

# 查看详细差异
git diff HEAD..upstream/master
```

## 常见问题

### 如何检查是否需要同步?
```bash
git fetch upstream
git status
# 或者
git log HEAD..upstream/master --oneline
```

### 如果同步时出现冲突怎么办?
1. 查看冲突文件: `git status`
2. 编辑冲突文件,解决标记为 `<<<<<<<`, `=======`, `>>>>>>>` 的部分
3. 标记为已解决: `git add <文件>`
4. 完成合并: `git commit` (merge) 或 `git rebase --continue` (rebase)

### 如何撤销错误的合并?
```bash
# 如果还没有推送
git reset --hard HEAD~1

# 如果已经推送,需要创建反向提交
git revert -m 1 HEAD
```

## 自动化脚本

### 快速同步脚本
可以创建一个脚本文件 `sync.sh`:

```bash
#!/bin/bash
echo "Fetching upstream changes..."
git fetch upstream

echo "Merging upstream/master..."
git merge upstream/master

if [ $? -eq 0 ]; then
    echo "Merge successful! Pushing to origin..."
    git push origin master
    echo "Sync completed!"
else
    echo "Merge conflicts detected. Please resolve them manually."
    exit 1
fi
```

使用方法:
```bash
chmod +x sync.sh
./sync.sh
```

## 最佳实践

1. **定期同步**: 建议每周或每次开发新功能前同步一次上游
2. **使用分支**: 所有自定义开发都在功能分支上进行,保持 master 分支干净
3. **测试后合并**: 同步上游更新后,先运行测试确保没有破坏性更改
4. **记录修改**: 维护一个 CHANGELOG 或文档记录您的自定义修改
5. **备份**: 在进行重大同步前,创建备份分支
