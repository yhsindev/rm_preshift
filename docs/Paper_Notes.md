# [Architecting Racetrack Memory preshift through  pattern-based prediction mechanisms] 
**年份/會議:** 2019 / IEEE International Parallel and Distributed Processing Symposium (IPDPS)
**連結:** [PDF Link](https://ieeexplore.ieee.org/document/8821036)

## 1. 核心概念 (一句話解釋)
> 本文提出了一種基於模式預測 (Pattern-Based Prediction) 的 Racetrack Memory (RM) 磁頭預移 (Preshift) 機制，透過學習記憶體存取的位移歷史 (Shift History) 來提前移動磁頭，顯著減少存取延遲。

## 2. 問題與動機 (Problem & Motivation)

* **2.1 背景** : 
Racetrack Memory (RM) 是一種新興的儲存技術，具有高密度（類似 DRAM）、非揮發性（類似 Flash）及低靜態功耗（類似 SRAM）的優點，被視為未來記憶體階層（Cache 或 Main Memory）的研究方向。
* **2.2 痛點**:
RM 的存取機制是序列化 (Serialized Access) 的。每一個 Cell 像是一條磁帶，要存取特定的資料位元（Domain），必須先將磁性區域 Shift (位移) 到讀寫頭（Port）的位置。這導致了變動且不可預測的存取時間 (Variable Access Time)，嚴重影響效能。
* **2.3 現有解法的不足 (Limitations)**:
1. 簡單的策略如 "Lazy" (留在原地) 或 "Eager" (回到中間) 依賴資料的空間局部性 (Locality)。
2. 當 Locality 不高時，這些策略效果很差。
3. 現有的 "Next-block preshifting" 太過簡單，無法捕捉複雜的存取模式。
* **現有的磁頭管理策略**：
1. Lazy Policy: 讀寫完後停在原地。這只對高局部性 (Locality) 的存取有效。
2. Eager Policy: 讀寫完後回到中間。這雖然減少了最差情況延遲，但增加了平均移動次數。
3. Next-block Preshifting: 簡單預測下一個是 +1。無法捕捉複雜的跳躍模式。


## 3. 核心方法 (Proposed Method: PRESHIFT)

### 3.1 核心觀察 (Key Insight)

作者發現，追蹤「位移量與方向 (Shift distance & direction)」比追蹤絕對的 Domain ID 更有效。不同記憶體地址 (Address) 的存取序列經常呈現相同的「相對位移模式」，例如「往右 1、往左 2」這種模式常跨多個地址重複出現。

### 3.2 硬體架構 (Hardware Structures)

本機制引入兩個輕量級硬體結構：

* **Shift History Register (SHR):** 小型暫存器，紀錄最近 $W$ 次位移操作 (如 3, -1, 2)，代表目前的位移情境。
* **Pattern Table:** 類似 Cache 的表格，儲存已識別模式並包含以下欄位：
  * Sequence Tag：用於比對 SHR 的歷史模式。
  * Predicted Shift：紀錄該模式下的下一次位移量。
  * Consolidation Bit：信心計數器，模式需重複出現達門檻才視為有效並啟動預測 (論文建議至少重複 1 次)。
  * Priority：支援 LRU 取代策略，Table 滿載時決定替換對象。

### 3.3 運作流程 (Workflow)

系統於每次存取進行兩階段操作：

1. **更新階段 (Update)**
	* 以當下 SHR 及本次實際位移量更新 Pattern Table，若滿載則採 LRU 替換。
	* 將最新位移寫入 SHR 以維持滑動視窗。
2. **預測階段 (Predict)**
	* 使用更新後的 SHR 查詢 Pattern Table。
	* Hit 且 Consolidated：視為可信模式，自動預移磁頭。
	* Miss 或未 Consolidated：退回 Lazy Policy，避免多餘移動。



## 4. 關鍵參數設定 (System Configuration)
**這是大家改 Code 最需要查的表，請整理在這裡：**

| Component | 參數 (Parameter) | 數值/設定 | Gem5 變數名 (Reference) |
| :--- | :--- | :--- | :--- |
| **CPU** | Cores / Freq | 4-core, 3GHz, x86-64 | `num_cpus`, `clock` |
| **L1 Cache** | Size | **32KB** (Instruction & Data Split) | `l1d_size`, `l1i_size` |
| | Assoc | 4-way | `l1d_assoc` |
| | Block Size | 64B | `cacheline_size` |
| | RM Tech | **128KB physical RM** (for 32KB logical) | *需修改 tags/data array* |
| **L2 Cache** | Size | **256KB** (Private, Unified) | `l2_size` |
| | Assoc | 8-way | `l2_assoc` |
| | Latency | 10 cycles (Data Array) | `l2_lat` |
| | RM Tech | **4MB physical RM** | |
| **LLC (L3)** | Size | **16MB** (Shared) | `l3_size` |
| | Assoc | 16-way | `l3_assoc` |
| | Latency | 24 cycles | `l3_lat` |
| **Main Mem** | Size / BW | 4GB / 32GB/s | `mem_size` |
| **RM Spec** | Domains | **64 domains** per track | *RM controller param* |
| | Port | 1 Read/Write Port | |
| | Shift Latency | 1 cycle per domain | `shift_latency` |

## 5. 實驗復現清單 (Experiments to Reproduce)
**第一階段：參數敏感度分析 (Parameter Sensitivity Analysis)**

**目標：** 確定 PRESHIFT 機制（包括 L1、L2 和 LLC 層級）的最佳配置，並評估其硬體成本。

檢查預先位移方法的有效性

| 編號 | 名稱 | 簡介 | 測量指標與目的 | 參考資料 |
| :--- | :--- | :--- | :--- | :--- |
| 1 | 磁頭追蹤模式比較 | 比較基於移位 (Shift-based) 模式與基於磁疇 (Domain-based) 模式的優劣 | 衡量正規化平均移位量 (Normalized average shift)，證明 Shift-based 較佳 | Fig. 5 |
| 2 | 模式長度 (Pattern Length) | 測試模式長度 W 從 2 到 5 對性能的影響 | 測量預移位準確度 (Accuracy)、預移位頻率 (Frequency) 與正規化平均存取延遲，以最短但仍精準的 W=2 為目標 | Fig. 6 |
| 3 | 鞏固閾值 (Consolidation Threshold) | 測試一個模式需要重複多少次（閾值）才能觸發預移位 | 觀察正規化平均移位量，選擇第一次重複後即鞏固的閾值 (值為 1) | Fig. 7 |
| 4 | 模式表大小 (Pattern Table Size) | 測試 Pattern Table 容量（N=4~128）對移位延遲的影響 | 衡量正規化平均移位量並平衡硬體成本；最終配置 L1=16、L2=32、LLC=64 條目 | Fig. 8 |
| 5 | 硬體成本估算 (能耗) | 使用 DESTINY 等工具對最終配置之 Pattern Table 建模 | 估算 Pattern Table 面積 (Area)、讀/寫能量 (Read/Write Energy) 與洩漏功率 (Leakage Power) | Table III |


**第二階段：效能驗證 (Performance Evaluation)**

**目標：** 比較 PRESHIFT (P-P) 與現有的磁頭管理策略（**EAGER、LAZY、NEXT-BLOCK, L-N 組合**）在 58 個工作負載下的表現。

和既有研究比較——記憶體階層各層分析

| 編號 | 名稱 | 簡介 | 測量指標與目的 | 參考資料 |
| :--- | :--- | :--- | :--- | :--- |
| 6 | 各快取層級的平均移位操作 | 比較不同策略在 L1 Dcache、L2 與 LLC 層級所減少的平均移位延遲 (Average Shift Latency) | 將結果正規化到 LAZY 策略，證明 PRESHIFT 於 L2/LLC 能顯著降低移位延遲 | Fig. 9 |
| 7 | 預移位操作完成率 | 衡量預移位操作是否能在下一個存取請求到達前完成 | 量測完成百分比，顯示 L2 與 LLC 幾乎所有預移位皆可及時完成 (>95%) | Fig. 10 |
| 8 | 預測行為與準確性分析 | 詳細分析 PRESHIFT 模式識別有效性 | 針對最佳與最差工作負載，統計預測頻率 (% requests provoke shift prediction) 與正確率 (% correct predictions) | Fig. 11 |
| 9 | 磁疇數量敏感度 | 測試 RM 磁疇數量變化（64、32、16 domains）對 PRESHIFT vs. LAZY 的影響 | 測量不同磁疇數量下的正規化平均移位量；磁疇越少，PRESHIFT 改善幅度可能更大 | Fig. 12 |
| 10 | 整體記憶體延遲 | 評估 PRESHIFT 對整體記憶體階層（L2、LLC、主記憶體）存取延遲的影響 | 測量正規化平均記憶體延遲並以 LAZY-LAZY (L-L) 為基準，顯示可帶來最高約 10% 改善 | Fig. 13 |
---
