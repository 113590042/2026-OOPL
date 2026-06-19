# 2026 OOPL Final Report

## 組別資訊

- 組別：12
- 組員：113590042 陳佑軒
- 復刻遊戲：Syobon Action（貓里奧）

## 專案簡介

### 遊戲簡介

本專案使用 C++ 復刻經典惡搞平台遊戲 Syobon Action。遊戲外觀類似傳統橫向捲軸遊戲，但關卡中包含隱藏方塊、落下陷阱、毒蘑菇、假旗桿、尖刺及特殊管道等非直覺機關。玩家需要觀察陷阱規律、反覆嘗試並完成 1-1 至 1-4 的關卡流程。

專案以 OpenSyobon 原始碼與素材配置作為行為參考，將原本以陣列、magic number 與全域狀態為主的實作，整理成場景、實體、資源、關卡編譯與除錯工具等模組。

### 組別分工

本組由陳佑軒獨立完成，工作內容包括：

- 分析 OpenSyobon 原始碼、關卡資料與素材切片。
- 建立玩家、敵人、方塊、陷阱、管道與過關流程。
- 實作碰撞、角色物理、音效、訊息框及場景切換。
- 修正關卡物件位置、素材錯配與互動行為。
- 建立自動建置、指定關卡啟動、傳送、截圖及關卡稽核工具。

## 遊戲介紹

### 遊戲規則

- 使用 `A`、`D` 或方向鍵左右移動。
- 使用 `Z`、`Space`、`W` 或上方向鍵跳躍。
- 站在可進入的直向管道上按 `S` 或下方向鍵進入。
- 面向側向管道時按 `D` 或右方向鍵進入。
- 玩家碰到敵人、尖刺、岩漿或掉出地圖會失去生命。
- 部分問號方塊會出現金幣，部分會生成敵人或毒蘑菇。
- 抵達旗桿、終點觸發區或指定出口後進入下一段關卡。
- 按 `O` 可切換除錯用無敵模式；按 `M` 可切換聲音。
- 無敵模式下可按 `Tab` 開啟關卡選擇介面。

### 遊戲畫面

1-2A 場景與管道：

![1-2A 場景](https://raw.githubusercontent.com/113590042/CatMarioPTSD/main/docs/visual_audit/level1_2a_side_pipe_after.png)

1-4 Boss 與尖刺區域：

![1-4 Boss 區域](https://raw.githubusercontent.com/113590042/CatMarioPTSD/main/docs/visual_audit/level1_4_boss_green_spikes_interactive.png)

尖刺傷害驗證：

![尖刺傷害驗證](https://raw.githubusercontent.com/113590042/CatMarioPTSD/main/docs/visual_audit/level1_4_decoration_spike_damage.png)

## 程式設計

### 程式架構

專案主要分為以下模組：

- `App`：管理遊戲主迴圈、視窗、場景更新及場景切換。
- `Scene`：包含標題、遊戲、死亡、通關等不同畫面。
- `StageCompiler`：將舊版關卡陣列與 stage script 轉換成統一的 `LevelDefinition`。
- `Entity`：所有玩家、敵人、方塊、陷阱及裝飾物件的共同基底。
- `EntityFactory`：依舊版地圖 ID 建立對應實體，集中處理資料格式相容性。
- `ResourceManager`：載入 spritesheet，依原作座標切割及組合遊戲素材。
- `SoundManager`：管理背景音樂、音效、音量及靜音狀態。
- `MessageBubbleManager`：管理玩家與敵人的訊息框及顯示時間。
- `.codex/skills/cat-mario-game-debug`：提供關卡傳送、視窗截圖、關卡稽核及啟動測試。

核心流程為：

```text
App
 └─ Scene
     └─ GameScene
         ├─ StageCompiler -> LevelDefinition
         ├─ EntityFactory -> Entity / Block / Enemy / Hazard
         ├─ ResourceManager
         ├─ SoundManager
         └─ MessageBubbleManager
```

### 程式技術

- 使用 C++17、CMake、SDL2 與 PTSD framework 建立遊戲。
- 使用物件導向的繼承與多型統一管理不同實體。
- 使用 Factory Pattern 將地圖 ID 與具體物件建立流程分離。
- 使用 Scene Pattern 管理標題、遊戲、死亡及通關畫面。
- 使用 100 倍整數定點座標保存原作物理數值，降低浮點誤差並方便對照原始碼。
- 使用 spritesheet 切片與動態 Surface 組合管道、旗桿、尖刺及大型平台。
- 使用環境變數提供指定關卡、玩家座標、攝影機、無敵及畫面擷取等除錯功能。
- 使用 PowerShell 腳本進行關卡編譯稽核、啟動 smoke test 及遊戲視窗截圖。

## 結語

### 問題與解決方法

1. **舊版地圖 ID 無法直接對應現代物件**  
   原作將地圖、敵人與觸發器混合在多組陣列中，且大量使用 magic number。專案加入 `StageCompiler` 與 `LevelDefinition`，先將舊格式轉成具名的方塊、結構、事件與裝飾資料，再交由場景建立實體。

2. **素材切片與碰撞範圍錯配**  
   同一張 spritesheet 中包含不同尺寸素材，若只依物件 ID 取圖會出現管道缺角、尖刺位置錯誤或碰撞範圍不一致。解法是依原作 `loadg.cpp` 的切片位置建立資源索引，並為管道、旗桿與尖刺 strip 建立專用圖像產生函式。

3. **畫面看似正確但互動不正確**  
   部分綠色尖刺曾被當成無碰撞裝飾物件。修正後，可見的尖刺統一建立為 `SpikeHazard`，並讓顯示尺寸與實際碰撞尺寸一致。

4. **只看程式碼難以判斷關卡是否正確**  
   專案建立 debug workflow，可從指定關卡及座標啟動、傳送玩家、固定攝影機並擷取 SDL 視窗，再與原作畫面比較。另以 `audit-stage.ps1` 驗證各關卡的入口、出口與 progression，以 `smoke-run.ps1` 驗證程式可正常啟動。

5. **終點與自動移動流程可能穿過危險物**  
   通關動畫曾略過一般碰撞處理。修正後，即使玩家進入旗桿自動移動流程，仍會檢查死亡方塊與危險物，避免穿過尖刺後仍判定通關。

### 自評

| 項次 | 項目 | 完成 |
|------|------|------|
| 1 | 專案可使用 CMake 建置並啟動遊戲 | V |
| 2 | 完成專案權限改為 public | 待確認 |
| 3 | 具有 debug mode 的功能 | V |
| 4 | 解決專案上所有 Memory Leak 的問題 | 待進行完整工具檢測 |
| 5 | 報告中沒有任何錯字，以及沒有任何一項遺漏 | V |
| 6 | 報告至少保持基本的美感，人類可讀 | V |

### 心得

這次專案最困難的部分不是讓角色能移動，而是重現原作大量依賴全域陣列與 magic number 的關卡行為。相同的數值在不同陣列中可能代表方塊、敵人、裝飾或觸發器，只靠畫面猜測容易造成素材與碰撞錯配。

實作過程讓我更熟悉 C++ 物件導向設計、資源生命週期、場景管理、碰撞處理與舊系統重構。我也了解到遊戲復刻不能只確認程式可編譯，還需要建立可重複的測試方式。透過指定位置啟動、截圖比較及關卡稽核，除錯效率比每次從第一關手動走到問題位置高很多。

### 貢獻比例

| 組員 | 學號 | 貢獻比例 |
|------|------|----------|
| 陳佑軒 | 113590042 | 100% |
