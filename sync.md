这是一个非常典型的 GitHub 工作流需求。在 Git 的逻辑中，**合并（Merge）操作通常是增量的**。只要你的 `main` 分支里从来没有过同名的文件（即没有删除过这两个文件的记录），将 `main` 合并到 `docker-build` 时，这两个文件是会被保留的。

以下是完全在 GitHub 网页界面（Web UI）上完成的操作步骤：

### 第一步：同步 `main` 分支与上游
首先确保你的 `main` 分支是最新的。

1.  在 GitHub 上打开你的仓库 `darkelfzdy/pocket-tts`。
2.  确保当前分支切换在 `main`。
3.  点击中间的 **"Sync fork"** 按钮。
4.  如果有更新，点击 **"Update branch"**。

### 第二步：在 `docker-build` 分支创建 `sync.md`
1.  将分支切换到 `docker-build`。
2.  点击 **"Add file"** -> **"Create new file"**。
3.  文件名输入 `sync.md`，填写你的说明内容。
4.  点击 **"Commit changes..."**，直接提交到 `docker-build` 分支。

### 第三步：将 `main` 的更新合并到 `docker-build`
这是最关键的一步，目的是把上游的代码同步过来，同时保留你的自定义文件。

1.  在仓库页面点击 **"Pull requests"** 选项卡。
2.  点击右侧的绿色按钮 **"New pull request"**。
3.  **注意选择方向（非常重要）：**
    *   **base (左边):** 选择 `docker-build` (你要更新的目标)。
    *   **compare (右边):** 选择 `main` (源头)。
    *   *提示：GitHub 默认可能会对比上游仓库，请点击 "compare across forks"，确保左边是 `darkelfzdy/pocket-tts` 的 `docker-build`，右边是 `darkelfzdy/pocket-tts` 的 `main`。*
4.  你会看到 "Able to merge"，下方会列出上游项目更新的代码，但**不会**显示删除 `docker-image.yml` 或 `sync.md`（因为 `main` 分支里没有这些文件的删除记录）。
5.  点击 **"Create pull request"**。
6.  标题可以写 `Sync with main`，然后点击 **"Create pull request"**。
7.  在 PR 页面，点击底部的绿色按钮 **"Merge pull request"**，然后 **"Confirm merge"**。

---

### 为什么这样做是安全的？
*   **Git 增量机制：** Git 追踪的是“修改”。因为 `main` 分支里压根不存在 `.github/workflows/docker-image.yml` 和 `sync.md`，所以从 `main` 合并到 `docker-build` 时，Git 认为 `main` 对这两个文件“没有任何意见”，因此会保留 `docker-build` 分支中现有的文件。
*   **未来同步：** 以后每次上游更新，你只需要重复“第一步（同步 main）”和“第三步（PR 合并 main 到 docker-build）”即可。

### 进阶建议：使用 GitHub Actions 自动同步（可选）
如果你觉得手动点 PR 麻烦，可以在 `docker-build` 分支中再增加一个 workflow 文件（例如 `.github/workflows/sync-main.yml`），利用 `ad-m/github-push-action` 或者简单的脚本，在 `main` 分支有变动时自动合并到 `docker-build`。

但对于大多数用户，每隔一段时间手动通过网页 PR 一次是最稳妥、最直观的方式。

### 验证操作
操作完成后，你可以检查 `docker-build` 分支：
1.  代码是否已经是上游最新的？
2.  `.github/workflows/docker-image.yml` 是否还在？
3.  `sync.md` 是否还在？

如果都在，说明操作成功。由于你的 Docker 任务是基于 `push` 触发的，合并操作产生的 `Merge commit` 会自动触发你的 Docker 镜像构建。
