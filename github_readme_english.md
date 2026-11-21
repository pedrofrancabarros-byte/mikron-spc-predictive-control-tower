# Mikron MBO-XT200 — Predictive SPC Control Tower

![Power BI](https://img.shields.io/badge/Power%20BI-Desktop-F2C811?style=flat&logo=powerbi&logoColor=black)
![DAX](https://img.shields.io/badge/Language-DAX-0078D4?style=flat)
![Methodology](https://img.shields.io/badge/Methodology-Six%20Sigma%20%7C%20SPC-2E7D32?style=flat)
![Domain](https://img.shields.io/badge/Domain-Predictive%20Maintenance-1E293B?style=flat)

An end-to-end **Industrial Analytics & Predictive Maintenance** solution built in **Power BI** for high-precision CNC machining (contextualized around the **Mikron MBO-XT200** platform). The Control Tower monitors part-by-part dimensional accuracy across three independent spindle stations, combines **Statistical Process Control (SPC)** with **Remaining Useful Life (RUL)** forecasting, and translates statistical alerts into real-time financial ROI via an interactive **What-If Simulator**.

---

## 1. Business Problem & Engineering Context

In Swiss-type / high-precision CNC machining—where internal diameter tolerances are held to $\pm 2.5\ \mu\text{m}$ around a nominal target of `4.0000 mm`—relying on reactive maintenance and periodic sampling creates four major operational bottlenecks:

1. **Silent Scrap Generation:** Detecting a defect only after parts cross specification limits (`USL = 4.0025 mm`, `LSL = 3.9975 mm`) means scrap has already occurred. Statistical control limits ($\pm 3\sigma$: `UCL = 4.00135 mm`, `LCL = 3.99865 mm`) expose instability before parts become non-conforming.
2. **High-Cost Unplanned Downtime:** Emergency tool failures incur heavy unplanned stop costs (`450 CHF` baseline) compared to scheduled preventive tool changes (`120 CHF` baseline).
3. **Ambiguous Root-Cause Diagnosis:** Scrap counts alone cannot distinguish between **gradual tool wear** (mean shift) and **mechanical chatter / vibration** (sudden variance explosion). Tracking $C_p$ alongside $C_{pk}$ isolates the exact failure mechanism.
4. **Disconnect Between Shopfloor Metrics and Financial Impact:** Statistical indices rarely communicate business value to operations leadership. This project links every statistical intervention directly to net cost avoidance in Swiss Francs (`CHF`).

---

## 2. Dashboard Architecture

### Page 1: `SPC Control Tower` (Shopfloor Operations & Diagnostics)
Designed using **Shopfloor UI/UX** principles (5-second readability rule):
* **HMI-Style Spindle Selector:** Single-click tile slicer (`MBO-XT200-S1`, `S2`, `S3`) for instant station isolation.
* **Dynamic Andon Status Banner:** Color-coded alert system (**Green** = Optimal, **Amber** = Predictive Alarm, **Red** = Critical Scrap) with worst-case escalation across the line.
* **Executive KPI Strip:** `Total Scrap Parts`, `Net Savings (CHF)`, `Current Cpk` (with Six Sigma traffic-light rules), and `Current RUL (min)`.
* **Shewhart Control Chart:** Part-by-part diameter trace with metrological Y-axis zoom (`3.9965 mm` to `4.0035 mm`) and semantic line hierarchy (solid trace vs. dashed/dotted limits).
* **Process Capability Degradation ($C_p$ vs. $C_{pk}$):** Dual-line diagnostic chart benchmarked against the Six Sigma minimum target ($C_{pk} \ge 1.33$).
* **Remaining Useful Life (RUL) Area Chart:** Fixed-scale (`0–160 min`) countdown acting as a tool-life fuel gauge.

| Spindle 1 — Optimal (`0_OPTIMAL`) | Spindle 2 — Tool Wear (`3_CRITICAL_SCRAP`) |
| :---: | :---: |
| ![Spindle 1 Optimal](images/01_spindle1_optimal.png) | ![Spindle 2 Tool Wear](images/02_spindle2_toolwear.png) |

| Spindle 3 — Spindle Chatter (`2_PREDICTIVE_ALARM`) |
| :---: |
| ![Spindle 3 Chatter](images/03_spindle3_chatter.png) |

---

### Page 2: `Executive Overview & ROI Simulator` (Management & Financial Sensitivity)
Provides a multi-spindle helicopter view paired with real-time financial modeling:
* **Multi-Spindle Comparison Matrix:** Side-by-side health, scrap, $C_{pk}$, RUL, and simulated savings across all stations, with integrated Andon and $C_{pk}$ conditional formatting.
* **Interactive What-If Parameters:** Dynamic sliders for **Unplanned Downtime Cost** (`200–1,000 CHF`, default `450 CHF`) and **Preventive Tool Change Cost** (`50–300 CHF`, default `120 CHF`).
* **Reactive vs. Predictive Cost Chart:** Clustered column chart updating in real time as cost assumptions change.
* **Scrap Cost Pareto by Failure Mode:** Quantifies the financial weight of `TOOL_WEAR` (`1,530 CHF`) vs. `SPINDLE_CHATTER` (`595 CHF`).

![Executive Overview and ROI Simulator](images/04_executive_roi_simulator.png)

---

## 3. Failure Modes & Baseline Results

The dataset (`SPC_Data`) contains **3,000 sequential part measurements** (1,000 parts per spindle) modeling three distinct industrial behaviors:

| Station | Failure Mode | Statistical Signature | Scrap Parts | Current $C_{pk}$ | Current RUL (min) | Reactive Scrap Cost | Baseline Net Savings |
| :--- | :--- | :--- | :---: | :---: | :---: | :---: | :---: |
| **MBO-XT200-S1** | `STABLE` | Centered at `4.0000 mm`; $C_p$ and $C_{pk}$ overlapping above `1.33` | `0` | `1.75` | `150.0` | `0 CHF` | `0 CHF` |
| **MBO-XT200-S2** | `TOOL_WEAR` | Progressive downward drift breaching `LSL`; $C_p$ stays high while $C_{pk}$ collapses | `18` | `0.22` | `0.0` | `1,530 CHF` | `1,860 CHF` |
| **MBO-XT200-S3** | `SPINDLE_CHATTER` | Sudden dispersion spike around Part 700 breaching `USL` & `LSL`; $C_p$ and $C_{pk}$ drop together | `7` | `0.69` | `150.0` | `595 CHF` | `925 CHF` |
| **Line Total** | **All Stations** | **Worst-Case Escalation: `3_CRITICAL_SCRAP`** | **`25`** | **`0.89`** | **`0.0`** | **`2,125 CHF`** | **`2,785 CHF`** |

> **Baseline ROI Summary:** Without predictive intervention, the line incurs **`3,025 CHF`** in reactive costs (`2,125 CHF` scrap + `900 CHF` across 2 unplanned stops). With predictive tool changes (`240 CHF`), the Control Tower achieves **`2,785 CHF`** in net savings.

---

## 4. DAX Engineering Highlights

### Spindle-Level Iterative Financial Model (`Net_Savings_CHF`)
Using `SUMX` over `VALUES('SPC_Data'[Spindle_ID])` ensures healthy stations (`S1`) are never penalized with false preventive maintenance costs while preventing double-counting in consolidated line totals:

```dax
Net_Savings_CHF = 
SUMX(
    VALUES('SPC_Data'[Spindle_ID]),
    VAR SpindleScrapCost = CALCULATE(SUM('SPC_Data'[Reactive_Scrap_Cost_CHF]))
    VAR UnplannedStop = IF(SpindleScrapCost > 0, 450, 0)
    VAR PreventiveChange = IF(SpindleScrapCost > 0, 120, 0)
    RETURN
    (SpindleScrapCost + UnplannedStop) - PreventiveChange
)
```

### Dynamic What-If Simulation Measures (Page 2)
Connects Power BI Numeric Range Parameters directly to the financial engine using `SELECTEDVALUE` with baseline fallbacks:

```dax
Sim_Custo_Reativo_CHF = 
SUMX(
    VALUES('SPC_Data'[Spindle_ID]),
    VAR ScrapCost = CALCULATE(SUM('SPC_Data'[Reactive_Scrap_Cost_CHF]))
    VAR StopCost = SELECTEDVALUE('Param_Custo_Paragem'[Param_Custo_Paragem], 450)
    RETURN
    ScrapCost + IF(ScrapCost > 0, StopCost, 0)
)
```

```dax
Sim_Custo_Preditivo_CHF = 
SUMX(
    VALUES('SPC_Data'[Spindle_ID]),
    VAR ScrapCost = CALCULATE(SUM('SPC_Data'[Reactive_Scrap_Cost_CHF]))
    VAR ChangeCost = SELECTEDVALUE('Param_Custo_Troca'[Param_Custo_Troca], 120)
    RETURN
    IF(ScrapCost > 0, ChangeCost, 0)
)
```

```dax
Sim_Poupanca_Liquida_CHF = [Sim_Custo_Reativo_CHF] - [Sim_Custo_Preditivo_CHF]
```

### Dynamic Andon Color Engine (`Andon_Color`)
Maps operational machine states to standardized industrial hex codes for conditional formatting across banners and tables:

```dax
Andon_Color = 
SWITCH(
    TRUE(),
    CONTAINSSTRING([Current_Machine_Status], "CRITICAL"), "#DC2626",
    CONTAINSSTRING([Current_Machine_Status], "PREDICTIVE") || CONTAINSSTRING([Current_Machine_Status], "WARNING"), "#D97706",
    "#16A34A"
)
```

---

## 5. Repository Structure

```text
mikron-spc-predictive-control-tower/
├── README.md                              # Project overview, visual gallery & DAX documentation
├── data/
│   └── spc_data_3000.csv                  # 3,000-row simulated SPC dataset (S1, S2, S3)
├── powerbi/
│   └── PowerBI_Micron_CNC.pbix            # Complete Power BI Desktop report (Pages 1 & 2)
├── docs/
│   └── Documentacao_Tecnica_SPC.pdf       # Full technical & executive documentation
└── images/
    ├── 01_spindle1_optimal.png            # Page 1 — S1 (Stable)
    ├── 02_spindle2_toolwear.png           # Page 1 — S2 (Tool Wear)
    ├── 03_spindle3_chatter.png            # Page 1 — S3 (Chatter)
    └── 04_executive_roi_simulator.png     # Page 2 — Multi-Spindle Executive & What-If Simulator
```

---

## 6. How to Run Locally

1. Clone or download this repository.
2. Open `powerbi/PowerBI_Micron_CNC.pbix` using **Microsoft Power BI Desktop**.
3. If prompted to update the data source path, navigate to **Transform data** $\rightarrow$ **Data source settings** $\rightarrow$ **Change source**, and point to `data/spc_data_3000.csv`.
4. Click **Apply changes** and explore both the `SPC Control Tower` and `Executive Overview & ROI` pages.