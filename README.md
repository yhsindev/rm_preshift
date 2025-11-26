# RM Preshift Project
這是 "Architecting Racetrack Memory preshift through pattern-based prediction mechanisms" 論文的復現專案。
目標：在 Gem5 模擬器中實作 PRESHIFT 機制，並驗證其對 Racetrack Memory 存取延遲的改善。

## 📅 專案進度 (Project Roadmap)

### Phase 0: 基礎建設
- [ ] Gem5 環境架設 (Docker image)
- [ ] 取得 Benchmarks (SPEC2006, PARSEC)
- [ ] 整合 DESTINY 功耗模型工具

### Phase 1: 參數敏感度分析 (Parameter Sensitivity)
> 目標：確定 L1/L2/LLC 的最佳配置 (Experiment 1-5)
- [ ] Exp 1: 磁頭追蹤模式比較 (Fig. 5)
- [ ] Exp 2: 模式長度 (Pattern Length) 測試 (Fig. 6)
- [ ] Exp 3: 鞏固閾值 (Threshold) 測試 (Fig. 7)
- [ ] Exp 4: Pattern Table 大小測試 (Fig. 8)
- [ ] Exp 5: 硬體能耗估算 (DESTINY)

### Phase 2: 效能驗證 (Performance Evaluation)
> 目標：比較 PRESHIFT 與 EAGER/LAZY 策略 (Experiment 6-10)
- [ ] Exp 6: 各快取層級移位延遲分析 (Fig. 9)
- [ ] Exp 7: 預移位完成率 (Fig. 10)
- [ ] Exp 10: 整體記憶體延遲評估 (Fig. 13)

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
