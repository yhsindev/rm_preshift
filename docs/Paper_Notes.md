# [論文名稱] Architecting Racetrack Memory Preshift...
**年份/會議:** 20xx / MICRO (舉例)
**連結:** [PDF Link](這裡貼連結)

## 1. 核心概念 (一句話解釋)
> *用一句話告訴組員這篇論文在幹嘛，例如：*
> 利用 Pattern-based 的預測機制，提前移動 Racetrack Memory 的磁頭，解決 Shift 延遲過高的問題。

## 2. 關鍵參數設定 (System Configuration)
**這是大家改 Code 最需要查的表，請整理在這裡：**

| 參數 (Parameter) | 數值/設定 (Value) | 對應 Gem5 變數名 (若已知) |
| :--- | :--- | :--- |
| **System Architecture** | 4-core, x86-64 | `num_cpus` |
| **L1 Cache** | 32KB, 8-way, Private | `l1_size`, `l1_assoc` |
| **L2 Cache** | 256KB, 8-way, Private | `l2_size` |
| **LLC (L3)** | 8MB, 16-way, Shared | `l3_size` |
| **Racetrack Memory** | Latency: ? ns / Shift: ? ns | *需修改 memory_ctrl* |
| **Preshift Table** | L1: 16 entries / L2: 32 / LLC: 64 | *需新增參數* |

## 3. 關鍵機制實作細節 (Implementation Details)
### 3.1 預測邏輯 (Prediction Logic)
* **何時觸發？** (例如：每次 Cache Access 時)
* **如何預測？** (例如：根據 PC address 查表)
* **公式/演算法：**
  > (這裡可以貼論文裡的公式截圖，或者用文字描述：Next_Shift = Current_Shift + Delta)

### 3.2 模式表結構 (Pattern Table Structure)
* **Table Entry 包含什麼欄位？** (例如：Tag, Last_Offset, Confidence_Counter)
* **取代策略 (Replacement Policy):** (例如：LRU?)

## 4. 實驗復現清單 (Experiments to Reproduce)
*(對應 README 的 Roadmap，這裡記錄詳細設定)*

* **[Exp 1] Head Tracking Comparison**
    * **目標:** 重現 Fig. 5
    * **變數:** Shift-based vs Domain-based
    * **Y軸:** Normalized Average Shift
    * **注意:** 記得把數據正規化 (Normalize) 到 Baseline。

* **[Exp 2] Pattern Length Sensitivity**
    * **目標:** 重現 Fig. 6
    * **變數:** Pattern Length $W = 2, 3, 4, 5$

## 5. 遇到的問題與解法 (Q&A / Troubleshooting)
* *(組員 A)*: Gem5 編譯時在 memory_ctrl 報錯？
    * *(組員 B)*: 記得在 SConscript 加入新的 .cc 檔。

---