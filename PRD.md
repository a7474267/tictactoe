# Wordle 遊戲 PRD（產品需求文件）

## 1. 產品概述

### 1.1 產品描述

一個基於 Vue.js 開發的中文版 Wordle 文字猜謎遊戲，玩家需要在有限次數內猜出正確的五個字母單字。

### 1.2 目標用戶

- 喜歡文字遊戲的玩家
- 學習英文的學生
- 尋找休閒娛樂的一般用戶

## 2. 功能需求

### 2.1 核心遊戲玩法

- 玩家需在 6 次機會內猜出 5 個字母的單字
- 每次猜測後，系統會給出顏色提示：
  - 綠色：字母正確且位置正確
  - 黃色：字母正確但位置錯誤
  - 灰色：字母不在答案中

### 2.2 單字庫管理

- 維護一個包含 100 個五字母英文單字的題庫
- 確保所有單字都是有效的英文單字
- 避免重複出現相同的單字，直到題庫中所有單字都被使用過

### 2.3 遊戲介面

- 5x6 的遊戲格子，顯示玩家的猜測
- 虛擬鍵盤，支援滑鼠點擊輸入
- 實體鍵盤支援
- 即時顯示字母狀態的視覺反饋

### 2.4 遊戲狀態

- 遊戲進行中（"playing"）
- 獲勝（"won"）：顯示煙火特效，2 秒後自動重置
- 失敗（"lost"）：顯示正確答案，需手動重置
- 重置機制：
  - 自動重置：每日更換、勝利後
  - 手動重置：失敗後通過 Toast 組件

### 2.5 特效與回饋

- Toast 提示系統：
  - 無效單字提示（2 秒後自動消失）
  - 遊戲結束提示（持續顯示直到重置）
- 勝利時顯示煙火特效
- 字母狀態顏色反饋：
  - 正確（綠色）
  - 位置錯誤（黃色）
  - 不存在（灰色）

## 3. 技術需求

### 3.1 前端技術

- Vue 3 Composition API
- CSS3 動畫效果
- 響應式設計

### 3.2 本地存儲

- 使用 localStorage 儲存：
  - 已使用的單字列表（usedWords）
  - 上次遊戲日期（lastPlayedDate）
  - 每小時自動檢查日期變更
  - 當所有單字用完時自動重置已使用列表

### 3.3 性能要求

- 頁面載入時間 < 2 秒
- 動畫效果流暢（60fps）
- 支援主流瀏覽器

### 3.4 定時任務

- 每小時檢查日期變更
- 跨日自動重置遊戲
- 使用 setInterval 實現定時檢查
- 組件卸載時自動清理定時器

## 4. 使用者體驗

### 4.1 介面設計

- 深色主題
- 清晰的視覺反饋
- 適配不同螢幕尺寸（手機、平板、桌面）

### 4.2 操作流程

1. 進入遊戲自動開始新局
2. 輸入猜測的單字
3. 提交後立即顯示結果
4. 遊戲結束時：
   - 獲勝：顯示煙火特效，2 秒後自動重置
   - 失敗：顯示正確答案，等待用戶手動重置

### 4.3 錯誤處理

- 非法單字提示
- 輸入超出長度限制提示
- 遊戲狀態異常自動重置

## 5. 未來擴展

### 5.1 潛在功能

- 遊戲統計數據
- 分享功能
- 多語言支援
- 自定義主題
- 競賽模式

### 5.2 維護計劃

- 定期更新單字庫
- 優化遊戲算法
- 收集用戶反饋
- 修復已知問題

## 6. 驗收標準

### 6.1 功能驗收

- 單字猜測邏輯正確性
- 顏色提示準確性
- 鍵盤狀態同步
- 遊戲狀態正確切換

### 6.2 性能驗收

- 響應式設計適配
- 動畫效果流暢度
- 本地存儲可靠性
