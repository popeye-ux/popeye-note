---
title: 'Git 工作流程最佳實踐：團隊協作的關鍵'
description: '介紹 Git Flow 與 GitHub Flow 的差異，以及如何建立適合你團隊的工作流程'
pubDate: '2024-02-10'
heroImage: '../../assets/blog-placeholder-4.jpg'
tags: ['Git', '協作', '工作流程']
---

良好的 Git 工作流程是高效團隊協作的基礎。本文將介紹幾種常見的 Git 工作流程，以及各種情境下的最佳實踐。

## 常見的 Git 工作流程

### Git Flow

Git Flow 是一種功能分支模型，適合有固定發布週期的專案：

```
main (生產環境)
  └── develop (開發主線)
        ├── feature/user-auth (功能開發)
        ├── feature/payment (功能開發)
        └── release/1.2.0 (發布準備)

hotfix/security-patch (緊急修復)
```

**優點：** 清晰的版本管理，適合複雜的發布流程
**缺點：** 分支管理複雜，不適合持續部署

### GitHub Flow

更簡單的工作流程，適合持續部署的專案：

```
main (生產環境)
  ├── feature/add-dark-mode
  ├── fix/login-bug
  └── docs/update-readme
```

**流程：**
1. 從 `main` 建立功能分支
2. 開發完成後開啟 Pull Request
3. 通過 Code Review 後合併到 `main`
4. 自動部署到生產環境

## Git 提交訊息規範

好的提交訊息非常重要，推薦使用 [Conventional Commits](https://www.conventionalcommits.org/) 規範：

```
<type>(<scope>): <description>

[optional body]

[optional footer]
```

**常用的 type：**

| Type | 說明 |
|------|------|
| `feat` | 新功能 |
| `fix` | Bug 修復 |
| `docs` | 文件更新 |
| `style` | 格式調整（不影響程式邏輯） |
| `refactor` | 重構 |
| `test` | 測試相關 |
| `chore` | 建置工具、依賴更新 |

**範例：**

```bash
git commit -m "feat(auth): 新增 Google OAuth 登入功能"
git commit -m "fix(cart): 修復購物車數量計算錯誤"
git commit -m "docs(readme): 更新安裝說明"
```

## 實用的 Git 指令

### 互動式 Rebase

整理提交記錄，讓 PR 更易讀：

```bash
# 整理最近 3 個提交
git rebase -i HEAD~3

# 在編輯器中：
# pick abc123 feat: 新增登入頁面
# squash def456 fix: 修復打字錯誤
# squash ghi789 style: 調整樣式
```

### 暫存工作（Stash）

切換任務時暫存目前的工作：

```bash
# 暫存目前修改
git stash push -m "WIP: 登入頁面開發中"

# 查看暫存列表
git stash list

# 恢復暫存
git stash pop
```

### Cherry-pick

將特定提交應用到其他分支：

```bash
# 將 commit abc123 應用到目前分支
git cherry-pick abc123

# 應用多個提交
git cherry-pick abc123 def456
```

## 保護主分支

在 GitHub 上設定分支保護規則：

1. 要求 PR 審查（至少 1 人）
2. 要求所有 CI 測試通過
3. 禁止直接推送到 `main`
4. 要求分支與 `main` 同步

## 結語

沒有一種工作流程適合所有團隊，關鍵是選擇一種適合你團隊規模和發布頻率的方式，並且**持續執行**。

最重要的原則是：**保持 main 分支永遠是可部署的狀態**。
