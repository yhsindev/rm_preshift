# RM Preshift Project
這是 "Architecting Racetrack Memory preshift through pattern-based prediction mechanisms" 論文的復現專案。
目標：在 Gem5 模擬器中實作 PRESHIFT 機制，並驗證其對 Racetrack Memory 存取延遲的改善。

## 📅 專案進度 (Project Roadmap)

### Phase 0: 基礎建設
- [ ] Gem5 環境架設 (Docker image)
- [ ] 取得 Benchmarks (SPEC2006, PARSEC)
- [ ] 整合 DESTINY 功耗模型工具

### Phase 1: 參數敏感度分析 (Parameter Sensitivity Analysis)
> 目標：確定 PRESHIFT 機制（L1/L2/LLC）的最佳配置並評估硬體成本。
- [ ] **Exp 1: 磁頭追蹤模式比較 (Head Tracking)**
  - 比較 Shift-based 與 Domain-based 模式，衡量正規化平均移位量。 (Ref: Fig. 5)
- [ ] **Exp 2: 模式長度測試 (Pattern Length)**
  - 測試 W=2~5 對準確度與延遲的影響，目標找出最佳長度 (W=2)。 (Ref: Fig. 6)
- [ ] **Exp 3: 鞏固閾值測試 (Consolidation Threshold)**
  - 測試觸發預移位所需的重複次數，目標鎖定閾值為 1。 (Ref: Fig. 7)
- [ ] **Exp 4: 模式表大小測試 (Pattern Table Size)**
  - 測試條目數 N=4~128 對移位延遲的影響，定案 L1/L2/LLC 的最佳大小。 (Ref: Fig. 8)
- [ ] **Exp 5: 硬體成本估算 (Hardware Cost)**
  - 使用 DESTINY 工具估算 Pattern Table 的面積、讀寫能耗與洩漏功率。 (Ref: TABLE III)

### Phase 2: 效能驗證 (Performance Evaluation)
> 目標：比較 PRESHIFT (P-P) 與現有的磁頭管理策略（EAGER、LAZY、NEXT-BLOCK, L-N 組合）在 58 個工作負載下的表現。
- [ ] **Exp 6: 各快取層級移位延遲分析 (Average Shift Latency)**
  - 比較 L1/L2/LLC 各層級減少的移位延遲 (正規化至 LAZY)。 (Ref: Fig. 9)
- [ ] **Exp 7: 預移位操作完成率 (Completion Rate)**
  - 測量預移位是否能在下一次請求到達前完成 (目標 > 95%)。 (Ref: Fig. 10)
- [ ] **Exp 8: 預測行為與準確性分析 (Prediction Accuracy)**
  - 針對 Best/Worst Case 工作負載，分析預測頻率與正確率。 (Ref: Fig. 11)
- [ ] **Exp 9: 磁疇數量敏感度測試 (Domain Count Sensitivity)**
  - 測試 64, 32, 16 磁疇數量對效能的影響，驗證 PRESHIFT 的可靠性。 (Ref: Fig. 12)
- [ ] **Exp 10: 整體記憶體延遲評估 (Overall Memory Latency)**
  - 評估對整個記憶體階層的系統級影響，目標改善幅度達 10%。 (Ref: Fig. 13)

## 🛠️ 技術細節與工具
* **Simulator:** Gem5 / rtSim
* **OS:** 建議使用 Ubuntu 20.04 LTS (Docker) 替代論文中的 Debian 8
* **Benchmarks:** SPEC2006, PARSEC, YCSB
* **Modeling:** DESTINY (Area, Energy, Power)

## 📂 資料夾說明
```text
rm_preshift/
├── README.md               # 專案目標、環境需求
├── docs/                   # 文件區
│   ├── paper_notes.md      # 論文筆記 
│   ├── architecture_spec.md # 參數表
│   └── meeting/             # 會議記錄
├── src/                    # 存放修改過的 Gem5 程式碼
│   ├── memory_ctrl/        # 記憶體控制器相關修改 (RM logic)
│   └── replacement_policy/ # 如果有改寫取代策略
├── configs/                 # Gem5 的設定檔 (se.py 或 fs.py 的修改版)
│   └── rm_config.py         # 針對 RM 參數的模擬腳本
├── scripts/                # 實驗
│   ├── run_phase1.sh       # 第一階段實驗
│   └── run_phase2.sh       # 第二階段實驗
├── analysis/               # 數據處理與繪圖
│   ├── parser.py           # 讀取 Gem5 stats.txt 的腳本
│   └── plot_fig5.py         # 繪製 Fig. 5 的腳本 (對應實驗 1)
├── references/             # 參考資料 (論文 PDF 或相關連結)
└── .gitignore             
```

## 🔗 詳細文件
更詳細的參數設定與實驗筆記，請參閱 [Docs/Paper_Notes](docs/paper_notes.md)
