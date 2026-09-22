# auto-filler（秒填鸭）

Chrome 扩展：用 LLM 语义匹配把用户本地存储的个人信息自动填入网页表单。数据全部留在本地（IndexedDB + chrome.storage），兼容任意 OpenAI-compatible API。

## Commands

```bash
npm run dev        # Chrome 开发模式 (HMR)
npm run build      # 生产构建
npm run zip        # 打包 .zip 上架 Chrome Web Store
npm run compile    # 仅类型检查 (tsc --noEmit)
```

无测试框架，无 linter。

## Architecture

基于 WXT（Vite 之上的扩展框架，`entrypoints/` 自动生成 manifest 入口，`@` alias 指向项目根）。

- `entrypoints/background.ts` — service worker：消息路由，编排 scan/fill 流程，调用 LLM 匹配
- `entrypoints/content.ts` — content script：扫描 DOM 表单字段（label/placeholder/name 等线索），用 native setter 填值（兼容 React）
- `entrypoints/popup/` — 用户 UI：扫描 → 确认匹配 → 填充，视图状态机 idle→scanning→result→filling→filled
- `entrypoints/options/` — 扩展内置设置页：管理个人信息和 API 配置，无外部网页

数据流：Options 录入个人信息（`utils/db.ts`，IndexedDB）和 API 配置（`utils/storage.ts`，chrome.storage）→ Popup 触发 scan → background 让 content 提取字段 → `utils/matcher.ts` 组 prompt 调 `/chat/completions` 语义匹配 → Popup 展示确认 → content 填值。

消息协议（`chrome.runtime.sendMessage`）：`startScan` / `startFill`（popup → background → content），`scan` / `fill`（background → content）。

## Conventions

- Vanilla HTML/CSS/TS，不用框架；popup 用命令式 DOM 渲染。主色 `#257FFD`。
