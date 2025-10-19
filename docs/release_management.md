# Release Management

当前自动化的发布流程通过 GitHub Actions 在 GitHub Releases 上生成预发布包，覆盖以下平台：
- Windows x86_64 (`x86_64-pc-windows-msvc`)
- Windows ARM64 (`aarch64-pc-windows-msvc`)
- Android ARM64 (`aarch64-linux-android`)

其他分发渠道仍保留，但不会在该流水线中自动更新，需要按需手动维护：
- GitHub Releases https://github.com/openai/codex/releases/
- `@openai/codex` on npm: https://www.npmjs.com/package/@openai/codex
- `codex` on Homebrew: https://formulae.brew.sh/cask/codex
  - Homebrew cask 仓库位置：https://github.com/Homebrew/homebrew-cask/blob/main/Casks/c/codex.rb

# Cutting a Release

> 当前流程针对 `old/local-release-*` 备份分支运行，触发条件已由 tag 调整为分支推送；脚本 `codex-branch-maintenance` 会自动创建并推送该分支。

1. 在 `codex-branch-maintenance` 项目中执行：
   ```
   bun run index.ts --repo <codex 仓库路径> --release
   ```
   该命令会：
   - 从 `local-release` HEAD 创建 `old/local-release-YYYYMMDD` 备份分支（若不存在）
   - 在 `local-release` 上执行 `cargo test` 与必要构建
   - 将备份分支推送至远端，触发 GitHub Actions。

2. GitHub Actions (`rust-release.yml`) 检测到备份分支推送后会运行：
   - 构建并打包上述三个平台的二进制
   - 以备份分支命名创建预发布版 GitHub Release（不会同步 npm / Homebrew）

3. 构建完成后，可在项目 Release 页面下载产物，或通过 workflow artifact 获取。

## 手动更新 npm / Homebrew

当前流水线不会自动发布 npm 包或 Homebrew cask。如需更新，可参考历史脚本 `codex-rs/scripts/create_github_release` 的实现或按仓库要求手动提交：
- npm 通过 Trusted Publishing 或本地 `npm publish`。
- Homebrew 需在 `homebrew-cask` 仓库提交 PR 更新 [`Casks/c/codex.rb`](https://github.com/Homebrew/homebrew-cask/blob/main/Casks/c/codex.rb)。

> 旧版本的 `codex-rs/scripts/create_github_release` 依赖 tag 并会发布 npm/Homebrew，此流程已停用。
