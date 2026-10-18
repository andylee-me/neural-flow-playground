Neural Flow Playground 規格書

作者：學生
日期：2026-10-18
作業：Vibe Coding 互動網頁

════════════════════════════════════════

一、專案概述

名稱：Neural Flow Playground
類型：單頁互動式 Web App（單一 index.html）
定位：在瀏覽器中即時模擬神經網路訓練過程的視覺化工具。使用者可自訂資料分佈，並在左右兩側分別設定 Model A 與 Model B 的架構超參數，同時訓練兩個 MLP 並排比較決策邊界與 Loss 曲線，直觀驗證「改變一個超參數對訓練的影響」。

════════════════════════════════════════

二、目標使用者

- 對機器學習有基礎興趣、但尚未深入實作的學習者
- 想快速驗證超參數調整直覺的開發者或學生

════════════════════════════════════════

三、介面佈局

固定三欄式版面，無需切換：

  左側面板（Model A）  |  中間（共享資料集 Bar + 兩個 canvas 欄位）  |  右側面板（Model B）

  左側面板包含：
    - Model A 架構參數設定
    - 共享訓練控制按鈕

  中間區域包含（由上至下）：
    - 共享資料集 Bar（資料集選擇、Noise、Clear、Copy A→B）
    - Model A canvas（決策邊界 + 指標列 + Loss 圖）
    - Model B canvas（決策邊界 + 指標列 + Loss 圖）

  右側面板包含：
    - Model B 架構參數設定
    - Best B 按鈕、雙向 Sync 按鈕

════════════════════════════════════════

四、核心功能模組

4.1 共享資料集 Bar

位於中間欄位頂部，橫跨兩個 canvas 欄位，A 與 B 共用同一份資料。

  操作項目與行為：
  - Linear 按鈕：清空並生成線性可分資料（80 點，斜線分界加噪音）
  - Circles 按鈕：清空並生成同心圓資料（80 點，內外圈分界）
  - Spiral 按鈕：清空並生成雙螺旋資料（80 點，最難線性分離）
  - Noise 滑桿：調整噪音程度 0~50%，即時重新生成目前資料集
  - Clear 按鈕：清除所有資料點
  - Copy A→B 按鈕：將 Model A 當前所有架構超參數複製到 Model B
  - 畫布左鍵點擊：新增 Class 0（紅點）
  - 畫布右鍵點擊：新增 Class 1（藍點）

  兩個 canvas 均可點擊新增資料點，資料點為 A 與 B 共用。

────────────────────────────────────────

4.2 Model A 架構面板（左側面板）

可調整的六個超參數：

  隱藏層數量（Layers）：1~4 層，以 +/- 按鈕調整，預設 2
  每層神經元數（Neurons）：1~8，以 +/- 按鈕調整，預設 4
  激活函數（Activation）：ReLU / Sigmoid / Tanh，預設 ReLU
  學習率（LR）：0.001 / 0.005 / 0.01 / 0.1 / 1.0，預設 0.01
  優化器（Optimizer）：SGD / Momentum / Adam，預設 SGD
  批次大小（Batch Size）：1 / 8 / 16 / 32 / All，預設 All

  Auto 按鈕：根據目前資料集自動套用建議超參數（詳見 4.6）
  修改任何參數後立即重建 Model A 並重置訓練狀態。

────────────────────────────────────────

4.3 Model B 架構面板（右側面板）

與 Model A 面板結構完全對稱，包含相同六個超參數與 Auto 按鈕。

  額外控制項：
  - Load Best B 按鈕：還原 Model B 訓練中記錄的最低 loss 對應的權重
  - A→B 按鈕：將 Model A 所有超參數複製到 Model B（重建 B）
  - B→A 按鈕：將 Model B 所有超參數複製到 Model A（重建 A）

────────────────────────────────────────

4.4 訓練控制（左側面板，A 與 B 共享）

  Speed 選擇器：Slow（1 ep/frame）/ Normal（10 ep/frame）/ Fast（50 ep/frame）
  Train Both 按鈕：同時開始訓練 A 與 B（requestAnimationFrame 驅動）
  Stop 按鈕：暫停兩個模型的訓練，保留當前權重
  Step 按鈕：A 與 B 各執行一個 epoch，不啟動自動迴圈
  Reset 按鈕：重新隨機初始化 A 與 B 的所有權重
  Best A 按鈕：還原 Model A 訓練中記錄的最低 loss 對應的權重

  訓練前驗證規則：
  - 資料點數量須 >= 4
  - 兩個 class（Class 0 與 Class 1）都必須存在
  - 未滿足條件時顯示說明性提示訊息，不開始訓練

────────────────────────────────────────

4.5 雙模型並排視覺化

決策邊界（每個模型各自獨立的 canvas）：
  - 50x50 網格均勻採樣畫布空間
  - 每格點執行 forward pass，取得預測值（0~1）
  - 依預測值線性插值染色：紅色系（Class 0 高信心）→ 白色（決策邊界）→ 藍色系（Class 1 高信心）
  - Bilinear 插值渲染為像素級平滑熱力圖
  - 資料點（紅/藍圓點）疊加顯示於熱力圖上方
  - X/Y 軸參考線（白色半透明）

即時指標列（每個模型各自獨立）：
  - Epoch：目前 epoch 數
  - Loss：當前 epoch 的 Binary Cross-Entropy
  - Best：訓練過程中記錄的最低 loss
  - Acc：當前模型在訓練資料上的分類準確率（%）

Loss 折線圖（每個模型各自獨立）：
  - Model A：綠色折線加半透明填充
  - Model B：橘色折線加半透明填充
  - 保留最近 500 筆，超過後滾動顯示
  - Y 軸範圍 0~1，有 0.25 / 0.5 / 0.75 網格線

────────────────────────────────────────

4.6 自動最佳參數建議（Auto Best Params）

左右面板各有獨立 Auto 按鈕，根據目前所選資料集套用建議：

  資料集 Linear：1 層、4 neurons、ReLU、LR 0.01、SGD、Batch All
  資料集 Circles：2 層、6 neurons、ReLU、LR 0.01、Adam、Batch All
  資料集 Spiral：3 層、8 neurons、Tanh、LR 0.005、Adam、Batch 16
  自訂或無資料集：2 層、6 neurons、ReLU、LR 0.01、Adam、Batch All

  套用後顯示 Toast 說明建議理由，4 秒後自動消失。

════════════════════════════════════════

五、神經網路實作規格

  網路架構：多層感知機（MLP），輸入層固定 2 neurons（x, y 座標）
  輸出層：固定 1 neuron 加 Sigmoid（二元分類機率）
  損失函數：Binary Cross-Entropy
  權重初始化：ReLU 使用 He init，其他使用 Xavier init
  梯度裁切：Gradient clip ±5（每層 delta 套用）
  數值穩定：Sigmoid 輸入 clamp 至 [-500, 500]
  批次訓練：Fisher-Yates 洗牌後按 batch size 切分；一個 epoch 等於完整資料過一次

  優化器：
    SGD：w -= lr * grad
    Momentum（beta=0.9）：v = beta*v + grad；w -= lr*v
    Adam（beta1=0.9, beta2=0.999, eps=1e-8）：標準 Adam 加 bias correction

════════════════════════════════════════

六、技術規格

  技術棧：純 HTML + CSS + Vanilla JavaScript
  外部依賴：無（零 library，零 framework）
  渲染方式：HTML5 Canvas API（ImageData 像素操作）
  神經網路運算：純 JS 手刻前向傳播 + 反向傳播（Backpropagation）
  響應式尺寸：ResizeObserver 監聽視窗變化，自動調整兩個 canvas 解析度
  資料持久化：無（單次 session）
  瀏覽器相容：現代瀏覽器（Chrome / Firefox / Safari）

════════════════════════════════════════

七、限制與範圍（Out of Scope）

  - 僅支援二元分類
  - 不提供模型權重匯出或匯入
  - 不連接任何後端或外部 API
  - 訓練資料不支援 CSV 匯入
  - 不支援多輸出（多類別分類）

════════════════════════════════════════

八、成功驗收標準

1. 在任一 canvas 上自由新增兩類資料點，或使用三種預設資料集，資料在 A 與 B 間共享
2. A 與 B 可各自獨立設定超參數，點擊 Train Both 同步訓練，決策邊界與 Loss 曲線即時更新
3. 每個模型各自顯示 Epoch、Loss、Best Loss、Accuracy 指標
4. Loss 曲線在可分資料集上隨訓練持續下降；Model A 顯示綠色，Model B 顯示橘色
5. 切換單一超參數（如 A 用 SGD、B 用 Adam），可在雙螺旋資料集上觀察明顯性能差距
6. Copy A→B / A→B / B→A 同步按鈕可正確複製所有六個超參數並重建對應模型
7. Auto 為各資料集套用建議參數（包含 Spiral 的 LR 0.005）並顯示 Toast 說明
8. Best A / Best B 可獨立還原各模型的最佳權重
9. 訓練前驗證：缺少資料點或只有單一 class 時，顯示提示訊息並阻止訓練
10. 所有功能在單一 index.html 內完整運作，無任何外部依賴
