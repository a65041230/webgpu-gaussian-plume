# 進度紀錄

> 對話被壓縮或中斷後：先讀 `PROMPT.md` → `DESIGN.md` → 這份檔案。

## 環境（2026-11-15 確認）
- 本機伺服器：`python -m http.server 8765 --bind 127.0.0.1 --directory <專案資料夾>`（背景執行）。網址 `http://127.0.0.1:8765/index.html`。
- 測試瀏覽器：Playwright MCP 的 Chrome 157。`navigator.gpu` 存在；adapter vendor＝nvidia、architecture＝turing（GTX 1650），`isFallbackAdapter=false`，不是 SwiftShader。
- 可用功能：`timestamp-query`、`float32-filterable`、`shader-f16` 都有；canvas 偏好格式 `bgra8unorm`。
- 本機只有一張顯示卡（NVIDIA GeForce GTX 1650，沒有列出內顯）。
- 注意：Playwright MCP 會把自己的紀錄檔寫到 `<Playwright MCP 的啟動目錄>\.playwright-mcp\`（它啟動時的工作目錄），這不是我能改的位置；最終報告要提。

## 里程碑
- [x] M0 設計文件
- [ ] M1 骨架：WebGPU 初始化、WGSL 模組載入、HDR＋AgX、相機、物理 JS、selftest a–d
- [ ] M2 大氣 LUT、天空、時間
- [ ] M3 地形、水面
- [ ] M4 設施、陰影
- [ ] M5 體積煙流、霧、光柱
- [ ] M6 夜間燈光、火炬
- [ ] M7 分析模式、UI、selftest e
- [ ] M8 平面反射、畫質分級、手機操作、bench、shot
- [ ] 評審迴圈（最多 6 輪）
- [ ] P1、P2
- [ ] README、Artifact、最終報告

## 評審輪次
（P0 完成後開始）

## 待修問題
（無）

## 踩過的坑
（邊做邊記）
