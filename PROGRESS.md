# 進度紀錄

> 對話被壓縮或中斷後：先讀 `PROMPT.md` → `DESIGN.md` → 這份檔案。

## 環境（2026-11-15 確認）
- 本機伺服器：`python -m http.server 8765 --bind 127.0.0.1 --directory <專案資料夾>`（背景執行）。網址 `http://127.0.0.1:8765/index.html`。
- 測試瀏覽器：Playwright MCP 的 Chrome 157。`navigator.gpu` 存在；adapter vendor＝nvidia、architecture＝turing，WebGL 字串是「NVIDIA GeForce GTX 1650 … Direct3D11」，`isFallbackAdapter=false`，不是 SwiftShader。
- 可用功能：`timestamp-query`、`float32-filterable`、`shader-f16` 都有；canvas 偏好格式 `bgra8unorm`。
- 本機只有一張顯示卡（NVIDIA GeForce GTX 1650，沒有列出內顯）。
- GitHub：公開 repo `a65041230/webgpu-gaussian-plume`（原名 harbor-plume，2026-11-17 改名、公開並開 GitHub Pages：https://a65041230.github.io/webgpu-gaussian-plume/）。作品名稱改為「煙流擴散視界」，MIT 授權。commit 作者用「沈慶隆 (Ken Shen)」＋GitHub noreply 信箱（只設在這個 repo；電腦的全域 git 設定會帶電腦名稱，不要用）。之後每次 push 前都要先問使用者。本機資料夾仍叫 harbor-plume。

## 測試流程的注意事項
- **Playwright 的視窗是最小化的**：`requestAnimationFrame` 每秒只有 1–2 次。程式裡有計時器備援（`LOOP.starved`），所以畫面照樣會跑，但 rAF 量到的幀率沒有意義，效能要看 `?bench=1` 的「等 GPU 做完」量法和 timestamp-query。
- **瀏覽器會快取 index.html**：每次載入都要加 `&_=時間戳`，否則會拿到舊檔。
- **Playwright MCP 只允許把截圖存在 `<Playwright MCP 的啟動目錄>\.playwright-mcp\`**（它啟動時的工作目錄）。截圖先存到那裡的 `hp\` 子資料夾，再搬到 `shots\`。這個 `.playwright-mcp` 資料夾是這次工作才產生的，最後要清掉並在報告說明。
- 截圖用 `browser_run_code_unsafe`：`page.goto` → `waitForFunction(__shotReady)` → `page.screenshot`。多張圖用 scratchpad 的 `montage.ps1` 拼成一張再看，省 context。

## 里程碑
- [x] M0 設計文件
- [x] M1 骨架：WebGPU 初始化、WGSL 模組載入與 binding 反射、HDR＋AgX、相機、物理 JS、selftest a–d（全部 PASS）
- [x] M2 大氣 LUT（transmittance／multi-scatter／sky-view）、環境光、雲、星、時間
- [x] M3 地形（GPU 產生高度圖 → 回讀 → clipmap＋morph）、水面
- [x] M4 設施（程序網格＋instancing＋程序材質）、cascaded shadow maps
- [x] M5 體積煙流（密度函式、光學深度體積、raymarch、時間累積、合成）、大氣霧、bloom
- [x] M6 夜間燈光 sprite、地面光池、火炬火焰
- [x] M7 分析模式（compute 熱圖、等值線、測站、Cmax、剖面簾幕）、UI 面板、selftest e
- [ ] M8 平面反射、畫質分級＋動態解析度、手機操作、bench、FXAA（都做了；bench 還沒實際跑過）
- [ ] 評審迴圈（最多 6 輪）
- [ ] P1、P2
- [ ] README、Artifact、最終報告

## 目前的預設值與數字（之後調整要同步改這裡）
- 預設：D 級、都市型、u10=3.5 m/s、風向 225°、煙囪 220／250／200 m、Q=150 g/s（每支）、Ts=55 °C、vs=20 m/s、d=8 m、Ta=26 °C、RH=95%。
- 算出來：煙囪口風速約 7.8 m/s、最終浮升約 145–153 m（三支煙囪口風速不同；2026-11-17 實測更正，之前寫的 115–125 m 是錯的）、白煙可見長度約 330 m、xmax 約 2.5 km、Cmax 約 93 μg/m³。
- RH 與白煙長度：60% → 約 85 m；85% → 約 145 m；95% → 約 265–330 m；98% → 約 440 m。
- 開場鏡頭：相機 (2750, 地形+22, −150)、yaw 270°、pitch −1°、FOV 24°、時間 −16.7 分（太陽高度 3.76°、方位 267.5°）。

## 評審輪次
（P0 完成後開始）

## 待修問題
- 前景鋪面太空、太平；地形的草地／樹林有規則的斑點（雜訊貼圖重複）。
- 市鎮建築偏白、太方。
- 煙流近看偏軟（像棉花），邊緣細節可以再銳利一點；迎光面顏色偏土黃。
- 整體太橘，天空上方的藍紫色在開場鏡頭裡看不到（望遠鏡頭只看到低仰角）。

## 踩過的坑
1. **WGSL：`fwidth`／`textureSample` 不能放在依 varying 分支的 `if` 裡**（uniform control flow）。先在分支外算好，或改用 `textureSampleLevel`。
2. **uniform struct 的解析**：我的 `parseStruct` 原本一行抓一個欄位，`struct LVU { o: vec4f, u: vec4f, … }` 寫在同一行就只抓到第一個。已改成去掉註解後用全域 regex。
3. **group 0 的成員要最早建立**：`common` 裡宣告的 uniform（Frame、Atmo、PlumeU）每個 pipeline 都會綁，包含啟動時產生地形的 pipeline。
4. **同一張貼圖不能在同一個 pass 又當輸出又被綁定**：產生某張貼圖的 pass 不能 include `scene_tex`（裡面有那張貼圖）。光學深度體積的 compute pass 因此另外宣告自己的 group 1。
5. **raymarch 的步長估計**（三個都造成過畫面上的直線或弧線）：
   - 用「到上風平面的距離」當步長：視線貼著平面走時一直走小步，步數耗盡。改用到煙囪口的距離。
   - 視線擦過煙流外框時估計距離趨近 0：加下限（局部粗細的 1/4）。
   - 高斯尾巴在外框處硬切：加一個平滑收到 0 的窗函數。
6. **雜訊八度交叉淡化要完全連續**：兩層的取樣座標只能差一個 2 倍縮放。多加了不同的時間位移，就會在「八度切換」的位置切出斷面。
7. **光學深度體積的外框要抓緊**：原本用 4.2σ＋蜿蜒的保守外框，一格體素大到 8–24 m，靠近煙囪口的細煙流取樣不到。改成只包住混合比例高於凝結臨界值的範圍。另外，體積存的是到體素中心的累積值，查的時候要扣掉取樣點自己那格的一半。
8. **霧裡的建築陰影（CSM）只在近處用**：遠處細長的影子在少量取樣下會變成雜線。
9. **靠近煙囪的氣膠霾不能太濃**：為了讓遠處的霾看得到而把係數調大，結果近處的霾把白煙染髒、遮暗。已限制 `min(f, 0.03)`。
11. **開場天空裡那道淡淡的圓弧（2026-11-17，改用 Opus 5.5 後修好）**。排除法：只看霧（除錯模式 2）沒有 → 問題在煙流；步數拉到 512 還在 → 不是步數用完；關掉蜿蜒 → 「布簾」紋路消失但邊線還在。根因有三個，要一起修：
    - 淡霾裡允許一步走 500 m（`0.03/σ` 的規則），比煙流截面（σ 約 100–300 m）還寬，輪廓附近的積分被高估 → 輪廓變成硬線。改成在煙流範圍內步長最多 0.8 倍粗細。
    - 蜿蜒振幅隨 σ 一直長、波長固定 → 遠處中心線斜率高達 5–8，煙流被扭成薄片，側看是一道道銳利的線。改成振幅上限 ＝ 波長／18（被截掉的變異數加回瞬時寬度，時間平均仍是同一個高斯包絡），高頻分量的係數也調小。振幅改存在 `plumeLUT` 第 4 列。
    - 霾用 `min(f, 0.008)` 硬截 → 煙囪附近變成均勻的實心管，均勻介質從側面看邊緣是銳利的。改成平滑飽和 `f/(1+f/0.006)`，往下風最後 40% 淡出，最遠 5 km。
12. **重建 GPU 資源時（切換畫質、device lost）會往已釋放的燈光 buffer 寫資料**：`buildSceneGPU` → `updateStacks` 在新的燈光 buffer 建立前就呼叫了。已在 `buildSceneGPU` 開頭把 `SCENE.lbuf` 清掉。
13. **Playwright MCP 偶爾會重啟**，分頁變回 about:blank，頁面裡的全域變數也就不見了。看到 `XXX is not defined` 先檢查分頁網址。
14. **除錯模式 `S.dbgVol`**（在頁面 console 設定）：1＝只看煙流不透明度（放大約 7 倍）、2＝只看大氣霧、3＝不合成體積。排查體積瑕疵時先用它分出是哪一層。
10. **物理上的重點**：27 °C 的夏季空氣裡，55 °C 飽和的煙氣稀釋 10–50 倍就不再過飽和，所以白煙只有幾百公尺長。要更長只能靠更高的相對濕度。遠處看得到的是氣膠霾，不是水氣。
