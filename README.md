# HKU-BME-Bootcamp-OpenClaw

**24-Weekend Intensive Self-Study Bootcamp**
**香港大學生物醫學工程 24 週末密集自學訓練營**

Aligned with HKU School of Biomedical Engineering (SBME) BEng(BME) programme and its specializations, preparing for MSc, MPhil, or PhD pathways.

與香港大學生物醫學工程學院 (SBME) BEng(BME) 課程及其專修方向完全對齊，為 MSc、MPhil 或 PhD 研究生途徑做準備。

---

## 📖 Overview / 概覽

This repository serves as the central workspace and tracking hub for a strict, weekend-only 24-week self-study bootcamp.

本倉庫是嚴格「僅限週末」24 週自學訓練營的中央工作空間與追蹤樞紐。

Designed specifically for learners with a **Civil Engineering + Data Science background**, the bootcamp systematically builds BSc-level Biomedical Engineering foundations while maximizing leverage from existing strengths in mechanics, materials, structures, Python, statistics, and modeling.

本訓練營專為具備 **土木工程 + 數據科學** 背景的人士設計，系統性地建立學士程度生物醫學工程基礎，並最大化利用現有在力學、材料、結構、Python、統計與建模方面的優勢。

The programme compresses the core learning outcomes of the full HKU BEng(BME) curriculum (all **37 courses**) into **24 focused weekends**, leading to strong postgraduate application readiness.

本計劃將港大 BEng(BME) 全部 **37 門課程** 的核心學習成果壓縮至 **24 個專注週末**，最終達到優秀的研究生申請準備程度。

---

## 🎯 Goals & Post-Bootcamp Outcomes / 目標與訓練營後成果

- ✅ Achieve solid BSc-level BME foundations aligned with HKU SBME curriculum
- ✅ Develop strong computational, modeling, and research skills
- ✅ Build a professional portfolio (code, simulations, literature syntheses, research proposal)
- ✅ Prepare competitively for HKU SBME MSc, CUHK BME MPhil/PhD, HKUST Bioengineering / Nanoscience & Technology, and related data-driven programmes

- ✅ 達到與港大 SBME 課程對齊的穩固學士程度生物醫學工程基礎
- ✅ 培養強大的計算、建模與研究技能
- ✅ 建立專業作品集（程式碼、模擬、文獻綜合、研究提案）
- ✅ 為港大生物醫學工程碩士、中大生物醫學工程哲碩/博碩、科大生物工程／納米科技及相關數據驅動課程做好競爭性準備

---

## 📁 Repository Structure / 倉庫結構

```
HKU-BME-Bootcamp-OpenClaw/
├── 00_Planning/                          # 規劃與追蹤文件
│   ├── HKU_BME_24Week_Bootcamp_Plan_Bilingual.xlsx
│   ├── HKU_BME_37_Course_Mapping_Matrix.xlsx
│   └── README_Planning.md
├── 01_Phase1_LifeSciences/              # Weekends 1-6 (BMED1207, BMED2206, BMED2301, BMED2302)
│   ├── W01_Intro_Chemistry_BMED1207/
│   ├── W02_Biomolecules_BMED1207_BMED2301/
│   ├── W03_Bioenergetics_BMED2301/
│   ├── W04_CellBiology_BMED2302/
│   ├── W05_Homeostasis_BMED2302/
│   └── W06_Phase1_Integration/
├── 02_Phase2_Quantitative/              # Weekends 7-12 (BMED2500, BMED3603)
│   ├── W07_Signals_BMED2500/
│   ├── W08_Fourier_BMED2500/
│   ├── W09_Filters_BMED2500/
│   ├── W10_Biostatistics_BMED3603/
│   ├── W11_PhysioModeling_BMED3603/
│   └── W12_Phase2_Integration/
├── 03_Phase3_Applications/              # Weekends 13-18 (BMED2600, BMED3600, BMED3501, BMED4603)
│   ├── W13_Biomechanics_BMED2600/
│   ├── W14_Kinematics_BMED2600/
│   ├── W15_Biomaterials_BMED3600/
│   ├── W16_Biotransport_BMED4603/
│   ├── W17_Imaging1_BMED3501/
│   └── W18_Imaging2_BMED3501/
├── 04_Phase4_Advanced_Research/         # Weekends 19-24 (BMED4604, BMED4601, BMED4504, BIOC3605)
│   ├── W19_TissueEng_BMED4604/
│   ├── W20_AdvBiomaterials_BMED4601/
│   ├── W21_AdvSignals_BMED4504/
│   ├── W22_Omics_BIOC3605/
│   ├── W23_Capstone_Sprint/
│   └── W24_Capstone_PGPrep/
├── Capstone_Proposal/                   # 最終研究提案 + CAES9531 模擬
├── Resources/
│   ├── textbooks/                       # 教科書 PDF / 參考書
│   ├── datasets/                        # PhysioNet, DICOM, omics
│   └── online_courses/                  # MIT OCW, NPTEL 連結
├── Agents/                              # OpenClaw agents (4 個)
│   ├── BME_Tutor.md
│   ├── Code_Lab_Assistant.md
│   ├── Paper_Scanner.md
│   └── Progress_Tracker.md
├── .gitignore
├── README.md                            # 本檔案
└── Master_Tracking.md                   # 每週進度總表
```

每個 weekend folder 內有標準 sub-folder 結構：
```
WXX_*/
├── readings/        # 預習閱讀清單 + 筆記
├── notes/           # 詳細手寫 / Obsidian notes
├── code/            # Jupyter Notebooks + Python scripts
├── deliverables/    # 最終交付物 (reports, simulations)
├── quiz/            # 自測題 + 答案
└── reflection.md    # 週末反思日誌
```

---

## ⏰ Bootcamp Rules (Strict) / 訓練營規則（嚴格執行）

- **Weekend-only study**: 8-12 focused hours per weekend (Saturday deep work + Sunday synthesis & deliverables). No weekday study.
- **Per-weekend format**: Pre-reading → Notes & synthesis → Hands-on coding/simulation lab → Exercises → Self-quiz + Reflection log + Deliverable
- **Tracking discipline**: Maintain clear notes, code, and deliverables in the designated folders. Update `Master_Tracking.md` weekly.
- **OpenClaw support**: Deploy custom agents (BME_Tutor, Code_Lab_Assistant, Paper_Scanner, Progress_Tracker) for explanations, debugging, literature, and accountability.

- **僅限週末學習**：每週末 8-12 小時專注學習（星期六深度工作 + 星期日綜合與交付物）。平日不進行學習。
- **每週末標準流程**：預習閱讀 → 筆記與綜合 → 動手編碼／模擬實驗 → 練習題 → 自測 + 反思日誌 + 交付物
- **追蹤紀律**：在指定資料夾內保持清晰的筆記、程式碼與交付物。每週更新 `Master_Tracking.md`。
- **OpenClaw 支援**：可部署自訂代理（BME_Tutor、Code_Lab_Assistant、Paper_Scanner、Progress_Tracker）協助解釋、除錯、文獻搜尋與進度追蹤。

---

## 📊 Planning Documents / 規劃文件

兩份完整的中英對照 Excel 檔案位於 `00_Planning/`：

1. **`HKU_BME_24Week_Bootcamp_Plan_Bilingual.xlsx`**
   完整 24 週詳細計劃，包含每週的學習目標、預習閱讀清單、編碼實驗、交付物、自測重點，以及你的土木工程 + 數據科學背景如何為該週帶來優勢的具體說明。

2. **`HKU_BME_37_Course_Mapping_Matrix.xlsx`**
   港大 BEng(BME) 全部 37 門課程的完整對應矩陣，顯示階段涵蓋、主要對應週末、涵蓋深度、優勢分析及優先級排序。

---

## 🚀 Getting Started / 開始使用

1. **Clone or fork** this repository.
2. Open the two Excel files in `00_Planning/` and review the overall plan and course mapping.
3. Set up your tracking system (Git + Obsidian or Notion recommended).
4. Choose your starting weekend and begin Phase 1.
5. Deploy OpenClaw agents (`Agents/` folder) whenever you need explanations, code assistance, or accountability.

1. **複製或 Fork** 本倉庫。
2. 開啟 `00_Planning/` 中的兩個 Excel 檔案，檢視整體計劃與課程對應。
3. 設置你的追蹤系統（推薦 Git + Obsidian 或 Notion）。
4. 選擇開始週末，從 Phase 1 開始執行。
5. 需要解釋、程式碼協助或進度追蹤時，隨時部署 OpenClaw 代理。

---

## 📚 Core Resources / 核心資源

### Textbooks / 教科書
- **Y.C. Fung** – Biomechanics / 《生物力學》 (BMED2600 基礎)
- **Ratner et al.** – Biomaterials Science / 《生物材料科學》 (BMED3600 核心)
- **Prince & Links** – Medical Imaging Signals and Systems / 《醫學影像訊號與系統》 (BMED2500/3501 核心)
- **Alberts et al.** – Molecular Biology of the Cell / 《細胞的分子生物學》 (Phase 1 關鍵)
- **Guyton & Hall** – Textbook of Medical Physiology / 《醫學生理學教科書》 (Phase 1-3 生理學選讀)

### Free / Online Resources / 免費資源
- **MIT OCW** Bioengineering courses (BE.010J 等)
- **NPTEL** Biomedical Engineering lectures
- **PhysioNet** physiological signal datasets (ECG, EEG)
- **Public DICOM** / medical image datasets
- **PubMed** / **Google Scholar**

### Tools / 工具
- **Python stack**: numpy, scipy, matplotlib, pandas, scikit-learn, PyTorch, scikit-image, nibabel
- **Jupyter Notebook** + **Git**
- **FEniCS** (optional, for PDE solving in transport phenomena)

---

## 🤖 OpenClaw Agents / OpenClaw 代理

詳見 `Agents/` 資料夾：

| Agent | 用途 | 觸發時機 |
|-------|------|----------|
| **BME_Tutor** | 解釋生物醫學概念 (chemistry, biology, physiology) | 預習時遇到不懂的 concept |
| **Code_Lab_Assistant** | 協助 coding (Python, signal processing, simulation) | 編碼實驗室 / 除錯 |
| **Paper_Scanner** | 搜尋 + 摘要 BME 文獻 (PubMed, arXiv) | 文獻回顧 / Capstone 提案 |
| **Progress_Tracker** | 追蹤週末進度 + 提醒未完成 deliverables | 每週日晚上更新 Master_Tracking.md |

---

## 📝 License / 授權

This project is licensed under the [MIT License](LICENSE).
本專案採用 [MIT 授權條款](LICENSE)。

---

## 🗓️ Project Timeline / 項目時間表

- **2026-06-10**: Repo initialized (Setup + structure)
- **2026-06-13**: PhysicsSelfStudy Wk 1 Theory Block (parallel project)
- **2026-06-20**: **BME Bootcamp Weekend 1 launch** (BMED1207 + BMED2206)
- **2026-12-12**: BME Bootcamp Weekend 24 completion + Capstone
- **2027-01+**: HKU SBME MSc / CUHK BME MPhil applications

---

**Single-handed turnaround through disciplined self-study.**
**以嚴謹自學實現單打獨鬥的翻身。**

*Civil Engineer → Biomedical Engineer*
*土木工程師 → 生物醫學工程師*

Built with persistence, curiosity, and strategic leverage of existing strengths.
以堅持、好奇心，以及對現有優勢的策略性運用打造。

---

*Maintainer: Saba (葉) | Last updated: 2026-06-10*
