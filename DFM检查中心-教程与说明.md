# 华秋DFM 分析原理 与「DFM 检查中心」教程与实现说明

> 版本：v1.0　对应程序：`Evan_DFM.exe` / `Asher_DFM.exe`（同一套框架，品牌不同）
> 代码位置：`D:\Evan\BRD-AI\brd_ai`
> 资料来源：华秋DFM 官网（dfm.hqpcb.com）公开介绍 + 本项目源码

---

# 第一部分　华秋DFM 是如何进行分析的（对标参考）

## 1.1 产品形态

| 形态 | 说明 |
|---|---|
| 华秋DFM 免费版（客户端） | 离线桌面软件，"看图神器 + 智能检查"，主打替代 CAM350 看图 |
| 华秋DFM 企业版 / CAM 企业版 | 面向 PCB 厂 / SMT 厂，规则可定制、批量与权限 |
| 在线分析（viewer） | 网页端上传即分析 |
| 配套 | DFM 论坛、视频教程、常见问题、KiCad 华秋发行版、EDA 在线查看器 |

定位：**设计与生产之间的桥梁**——用"生产视角"提前发现 PCB/SMT 隐患。

## 1.2 输入与解析能力

- **可解析**：Allegro（.brd）、Altium Designer、PADS、**Gerber**、**ODB++** 等主流格式
- **可导出**：Gerber、BOM、坐标文件、装配图、PDF、ODB++
- **辅助输入**：BOM 表、贴片坐标文件（用于 SMT/DFA 分析）
- **元件库**：覆盖 800 万+ 元件，支持自建元件库；3D 视角 1:1 实物查看

> 关键点：它不是只读 Gerber 图形，而是尽量还原**设计语义**（网络、元件、封装、焊盘、叠层），因此能做"开短路""元件干涉""可焊性"这类需要语义的检查。

## 1.3 分析流水线（六步）

```
① 文件解析        Gerber/ODB++/BRD/AD/PADS → 层、网络、元件、焊盘、走线、钻孔、叠层
② 数据建模        建立统一内部模型（几何 + 语义 + 网络拓扑）
③ 规则匹配        规则库（120+ 项 PCB 制板 / 1000+ 项 SMT 加工）逐条比对
                  规则来源：工厂制程能力 + IPC/行业规范 + SMT 工艺经验
④ 违例定位        每条违例带坐标/区域，可在 2D/3D 视图高亮
⑤ 原因与建议      每个警示项给出"隐患原因 + 改进建议"
⑥ 报告与闭环      详尽报告（结果项可自定义输出）→ 可直接下单/拼板/阻抗计算
```

## 1.4 检查体系（官方公开口径）

**PCB 制板（DFM）120+ 项**，典型如：
- 开短路：**断头线（dead-end）**、开路、短路
- 走线：线宽/线距不足、**走线疏密不当**、锐角/酸角、孤立铜、铜到板边
- 孔/焊盘：**过孔堆叠**、**焊盘间距不足**、孔环过小、孔径超能力、纵横比
- 阻焊/丝印：阻焊桥过窄、开窗不当、丝印上焊盘、字宽字高
- 拼板/外形：异形排版、工艺边、Mark、V-Cut/邮票孔

**SMT 加工（DFA/DFT）1000+ 项**，典型如：
- 元件布局干涉（间距/高度/朝向）
- **引脚爬锡不足**、焊盘对称性（墓碑）、极性标识
- 可焊性深度检查（封装焊盘设计）
- 测试点覆盖率、探针可达性
- 钢网开窗/厚度、工艺边与传送

**工具类**：自动叠层与阻抗计算、智能推荐最优拼板、BOM 分析、**文件精准比对**（BOM/PCB/Gerber 差异性检查）、Gerber 提取坐标、可视化焊接。

## 1.5 值得我们借鉴的 5 个工程要点

1. **语义 + 几何双通道**：几何查间距/线宽，语义查开短路/干涉/网表一致性。
2. **违例必须可定位**：每条问题带 XY/区域，能"点问题→跳版图"。
3. **问题即知识**：每条附"隐患原因 + 改进建议"，让新人也能改。
4. **规则可配置、报告可裁剪**：不同工厂/客户用不同规则集与输出项。
5. **分析与生产闭环**：分析结果直接对接拼板/阻抗/下单，形成价值闭环。

---

# 第二部分　Evan_DFM「DFM 检查中心」

## 2.1 定位与差异

| 维度 | 华秋DFM | 本项目 DFM 检查中心 |
|---|---|---|
| 形态 | 商业客户端/在线，规则库庞大 | 单机 Python 桌面程序，离线可用 |
| 数据源 | Allegro/AD/PADS/Gerber/ODB++ | **.brd（Allegro extracta 无界面提取）**、SKILL 导出 JSON、Gerber/ODB++/NC Drill、参数 JSON、拼版参数 |
| 规则来源 | 内置海量规则库 | **板厂能力库 YAML**（红版 / 四会 / 嘉立创）+ 内置装配/拼版规则 |
| 检查项 | 120+ 制板 / 1000+ SMT | 6 大模块、数十条规则（可外置扩展） |
| 定位 | 2D/3D 高亮 | 2D 版图交叉定位（`fab_viewer` / `gerber_viewer`）+ 违例截图入报告 |
| 报告 | 可自定义输出 | Valor 式 Excel（Main / Issue summary / Info / 开短路网表 / Issue1..N）+ 文本 + JSON |
| 附加 | 阻抗/拼板/BOM/下单 | **拼版设计（PCS/ARRAY/PANEL 三图）**、约束生成（Allegro SKILL）、Checklist 自动填写 |

一句话：**我们用"板厂能力库 + 统一 Issue 模型 + 多模块编排"复刻了华秋DFM 的核心方法论**（解析→建模→规则→定位→建议→报告），并把拼版/约束生成/Checklist 打通。

## 2.2 架构分层

```
入口层    evan_dfm.py / asher_dfm.py / main.py(CLI)
品牌层    src/branding.py（set_brand / DFM_BRAND，图标·标题·AppUserModelID）
界面层    ui_main.MainLauncher（一级导航）
            ├─ DFM 检查中心   src/dfm_hub_ui.DfmHubWindow  ← 本文重点
            ├─ 分析BRD-DFM    src/brd_dfm_ui.BrdDFMWindow
            ├─ 分析BRD        gui.BRD_AIGUI
            ├─ 分析Gerber/ODB ui_main.FabWindow
            ├─ 制程能力 DFM   gui.DFMDialog
            ├─ 排版设计       src/panel_ui.PanelDesigner
            └─ Gerber查看器   src/gerber_viewer.GerberViewer
编排层    src/dfm_hub.run_hub()      多模块统一编排 → 单一 analysis
          src/brd_dfm.analyze_*()    BRD 专用编排
分析层    dfm_engine（能力库比对） dfm_geometry（几何DFM·定位XY）
          dfm_netlist（开短路/网表） fab_data_reader（Gerber/ODB/Drill）
          panel_engine（拼版）
数据层    brd_extracta（extracta.exe 无界面提取） skill_export（.il）
          brd_geometry（JSON → BrdDesign 模型 + 空间索引）
          dxf_reader / pdf_reader
输出层    dfm_report（Valor式 Excel/文本） dfm_snap（违例截图）
          fab_viewer / gerber_viewer（交叉定位） panel_export（PNG/PDF）
规则模型  src/dfm_rules（统一 Issue + 严重度 + 风险评分）
配置      config/dfm_capability_{hongban,sihui,jlcpcb}.yaml, checklist_rules.yaml
          samples/brd_export_sample.json
```

## 2.3 数据流（两种源 → 同一 schema → 同一报告）

```
【源A｜直接读 .brd】 brd_extracta.extract_brd()  ── extracta.exe 无界面提取
        视图: COMPONENT_PIN(网表含XY) / COMPOSITE_PAD(焊盘·过孔) / LAYER(叠层板厚)
              NET(平均线宽) / CONNECTIVITY(ratsnest=开路) / COMPONENT
              GEOMETRY(走线·板框) / DRC_ERROR(违规含XY) / PAD_DEF2(孔径·焊盘·镀通)
        单位自动 mils → mm
【源B｜SKILL 导出】  skill_export.write_export_skill() → brd_dfm_export.il
        Allegro 内: skill load "brd_dfm_export.il" → brdExportJSON → JSON
                                   │
                                   ▼
                    brd_geometry.load_brd_export() → BrdDesign（统一设计模型）
                                   │
             ┌─────────────────────┼──────────────────────┐
             ▼                     ▼                      ▼
   dfm_geometry.analyze_geometry  dfm_netlist.analyze_netlist  fab_data_reader.analyze_fab
   （几何DFM，逐处实例含XY）      （开短路/网表一致性）        （Gerber/ODB/Drill + CAD比对）
             └─────────────────────┼──────────────────────┘
                                   ▼
                     brd_dfm.aggregate_by_rule / dfm_hub.run_hub
                     （按规则汇总、统一编码、分级统计、风险评分）
                                   ▼
              dfm_report（Excel/文本，含 dfm_snap 违例截图） + fab_viewer（交叉定位）
```

## 2.4 界面层级（多级界面）

### 一级：首页导航（`ui_main.MainLauncher`）
6 张卡片（各自独立二级窗口，互不干扰）：
`DFM 检查中心`、`分析BRD-DFM`、`分析BRD`、`分析Gerber`、`分析ODB`、`制程能力 DFM`
+ 工具条：`排版设计（拼版/三图联动）`、`Gerber查看器（叠加对比）`

### 二级：DFM 检查中心（`src/dfm_hub_ui.DfmHubWindow`）

**① 数据源与检查配置**
| 控件 | 说明 |
|---|---|
| 数据源 | `BRD 文件(.brd, 需 Allegro extracta)` / `导出 JSON` / `无源文件(仅参数化)` |
| BRD / JSON | 设计数据源路径 |
| 制造数据 | Gerber / ODB++ / NC Drill 目录或文件 |
| CAD 网表 | `{"net": ["R1.1", ...]}` JSON，用于 CAD vs FAB 比对 |
| 设计参数 | 制程能力 DFM 的参数 JSON（可点"生成设计参数模板"） |
| 拼版参数 | 拼版 DFM 的参数 JSON（可点"生成拼版参数模板"） |
| 板厂 / 板型 / 能力等级 / 铜厚 | `hongban｜sihui｜jlcpcb` × `rigid｜flex` × `standard｜automotive｜extreme` × um/oz |

**② 检查模块（可组合勾选）**
`capability`、`geometry`、`netlist`、`fab`、`fabnet`、`panel`

**③ 执行分析** → 后台线程运行，进度写入日志（队列 + 主线程轮询，避免跨线程操作 Tk）

### 三级：结果与分析
| 页签/入口 | 内容 |
|---|---|
| 问题清单 | 严重度 / 代码 / **模块** / 类别 / 规则 / 数量 / **X(mm)** / **Y(mm)** / 描述+建议；按严重度着色 |
| 统计与评分 | 规则集、已执行模块、问题分级计数、**可制造性评分/风险等级/预估良率**、开短路统计、CAD-FAB 比对、制造数据统计、拼版建议、几何统计 |
| 执行日志 | 分步进度与错误提示 |
| ④ 打开交叉定位 | `fab_viewer` 版图 + 问题标记，**问题↔版图双向定位** |
| ⑤ 导出报告 | Excel（Valor 式）+ 文本 + JSON |
| 参数 DFM 对话框 / BRD-DFM 窗口 | 快速跳转专项界面 |

## 2.5 六大检查模块：检查项与判定逻辑

### 模块 1　capability —— 制程能力 DFM（参数化）
- 实现：`dfm_engine.analyze_design(design, board_type, capability_level, factory)`
- 检查项：板尺寸 vs 最大拼板、板厚范围、**最小线宽/线距（按成品铜厚分档）**、不同网络孔壁间距、内/外层线路到板边、最小机械孔/最大机械孔/最小激光孔、单边孔环、**PTH 纵横比**、NPTH 孔到板边、**最小阻焊桥（按铜厚 oz 分档）**、阻焊开窗、树脂塞孔条件、丝印线宽/间隙/字符、覆盖膜（软板）、电测 PAD
- 补充项（`dfm_rules`）：激光孔环、盲孔填孔纵横比、激光孔介质厚度、树脂/油墨塞孔深度、塞孔纵横比、阻焊油墨厚度、丝印到阻焊/离成型线、表面处理是否支持
- 判定：
  - `higher_is_better=True`（越大越好，如线宽）：`设计 < 能力` → **危险(fail)**；`设计 < 能力×1.2` → **高(warn_limit)**
  - `higher_is_better=False`（越小越好，如最大孔径）：`设计 > 能力` → 危险；`设计 > 能力×0.8` → 高
  - 能力库无数据 → **低(info)**："该等级无制程数据，需与板厂评估"

### 模块 2　geometry —— 几何 DFM（定位 XY）
- 实现：`dfm_geometry.analyze_geometry(design, capability, copper_um, copper_oz, board_type)`
- 检查项（7 组，均输出**逐处实例含 XY**）：
  1. **最小线宽**（按网络/层，能力来自铜厚分档）
  2. **最小线距**（不同网络铜箔之间，扫描线算法求最近距离）
  3. **最小阻焊桥**（阻焊间距）
  4. **孔**：最小激光孔、最小机械孔、单边孔环（机械/激光）、**PTH 纵横比**
  5. **线路/铜到板边**（外层/内层分别判定）
  6. **测试点/电测焊盘尺寸**
  7. **Allegro DRC 透传**（原设计 DRC 违规，含 XY 与文本）
- 工程细节：
  - 铜箔元素统一抽象为"线段 + 半径"（`_El`），焊盘/过孔退化为中心点+半径
  - 使用 `brd_geometry.sweep_pairs()` **空间扫描**做邻近配对，避免 O(n²)
  - 上限保护：`MAX_ELEMENTS = 400000`（参与间距计算元素）、`MAX_INSTANCES_PER_RULE = 500`（每规则保留定位点）
  - extracta 走线宽度为"网络平均线宽"，因此间距/短路以**焊盘·过孔真实尺寸 + Allegro DRC** 为准，避免宽电源走线误报；SKILL 源为逐段实测线宽，更细
- 分级：`_classify()` → 超能力=危险；能力±20% 带内=高；否则合格不报

### 模块 3　netlist —— 开短路 + 网表一致性
- 实现：`dfm_netlist.analyze_netlist(design, fab_nets=None)`
- 方法：把焊盘/过孔/走线/铺铜建成节点，**并查集（DSU）**求物理连通域；与逻辑网表比对
- 输出 `netlist_summary`：`logical_net_count`、`node_count`、`element_count`、`physical_component_count`、`open_net_count`、`short_net_count`、`nodes_without_copper`、`short_events`
- 判定：
  - **开路 Open**：extracta 源以 Allegro 连通性（ratsnest）为权威；SKILL 源由几何连通域派生
  - **短路 Short**：不同网络铜箔接触（extracta 以 Allegro DRC 为准）
  - **网表一致性**：逻辑网表 vs 物理连通域差异

### 模块 4　fab —— 制造数据完整性（Gerber / ODB++ / NC Drill）
- 实现：`fab_data_reader.analyze_fab(path)`
- 解析：Gerber RS-274X（**模态 D 码/模态插补**、`%FS/%MO/%AD/%AM/%TO/%TD/%LP`、G36/G37 区域、G75 圆弧）、Excellon（工具表/坐标/PTH-NPTH）、ODB++（matrix / layers-features / cadnet-netlist）
- 检查：图形层缺失、钻孔层缺失、图层重复、**空图层**、坐标范围异常、**层间坐标偏移**、**光圈未定义**、**光圈尺寸≤0**、钻孔为空、镀通孔未标注、**X2 网络属性缺失（无法做网络比对）**
- 统计：图形数、孔数、最小线宽、最小孔径、层清单、网络表

### 模块 5　fabnet —— CAD vs FAB 网络比对
- 实现：`compare_netlists(cad_nets, fab_nets)` + `netlist_compare_issues()`
- 双向比对：
  - CAD 有连接、制造数据缺失 → **开路(Open)**（危险）
  - 制造数据出现额外连接 → **短路(Short)**（危险）
  - 网络只存在于一侧 → **网络缺失 / 多余网络**（高）
- 输出：`cad_net_count`、`fab_net_count`、`matched_nets`、`open_count`、`short_count`、`passed`

### 模块 6　panel —— 拼版 / 工艺边 DFM
- 实现：`panel_engine.design_panel(params)` → `issues`
- 检查：单板无法排入工作稿（危险）、**板材利用率偏低**（中）、仅能单拼（高）、Mark 点不足（中）、成本指数偏高（低）、工作稿放不进大板（高）
- 规则：WPNL 标准（18.4×24.4 / 20.4×24.4 / 21.4×24.4 / 24.4×28.4 / 28.4×32.4 in，**0.4in 赠送不计成本**）、默认 18×24in、工艺边、PCS 到工艺边间距、**连接筋 2.0mm**、V-Cut/邮票孔/折断边、阻抗条（数量/长/宽/间距，置于 ARRAY 之间）、Mark/定位孔、mm/mil 双单位

## 2.6 统一 Issue 模型（`src/dfm_rules.make_issue`）

| 字段 | 含义 | 示例 |
|---|---|---|
| `code` | 问题代码（合并后统一重编） | `FAB-TR-01`、`NET-03`、`DRC-01`、`PNL-02` |
| `category` | 类别 | 线路/钻孔/阻焊/丝印/塞孔/测试/拼板/组装/网络比对/制造数据/DRC |
| `rule` | 规则名 | 最小线宽、单边孔环、开路(Open) |
| `severity` | 严重度 | **危险 / 高 / 中 / 低 / 关闭** |
| `status` | 状态 | 打开 / 关闭 |
| `measured` / `required` / `unit` | 实测值 / 规则值 / 单位 | 0.08 / 0.10 / mm |
| `x` / `y` | 定位坐标（mm） | 12.73 / 14.87 |
| `layer` / `from` / `to` | 涉及层 / 起 / 止 | top / R1 / 1 |
| `count` | 数量（同规则汇总） | 47 |
| `description` / `suggestion` | 问题描述 / **专家建议** | — |
| `note` / `net` | 备注 / 网络 | 成品铜厚 35um / GND |
| `locations` | 全部定位点列表（汇总项） | `[[x,y], ...]` |
| `module` | 来源模块（检查中心） | capability/geometry/netlist/fab/fabnet/panel |

**严重度权重与评分**（`dfm_rules.compute_risk_score`）：
```
权重: 危险=10  高=5  中=2  低=0.5  关闭=0
penalty = Σ(权重 × 数量)
score   = max(0, 100 − min(100, penalty))
良率区间 = [score−2, score+2] %
风险等级: 有危险→危险；否则有高→高；有中→中；仅低→低
```

## 2.7 报告结构（`src/dfm_report.generate_dfm_excel`）

| 工作表 | 内容 |
|---|---|
| `Main` | 封面：单板信息（客户/板名/版本/规则集/EDA/时间）、问题总数与分级汇总、**评分/风险/良率/结论**、制造与装配统计 |
| `Issue summary` | 问题清单：序号、问题点(代码)、描述、**专家建议**、数量、优先级（配色）、状态、涉及层、**X/Y**、备注；每行超链接跳转到明细页 |
| `Info` | 制造工艺（线宽/线距/孔径/层数/板厚/纵横比/表面处理/孔数/尺寸…）+ 装配工艺（器件/表贴/插件/引脚/钢网/DFT）+ 制程能力备注 |
| `开短路网表` | 开路/短路/网表一致性明细（含网络与节点） |
| `Issue1..N` | 每条问题一页：规则名称、涉及层、从/到、测量值、规则值、X/Y、严重度、状态、首次版本、反馈；**嵌入违例截图** |

截图由 `src/dfm_snap.py` 生成：`render_overview_png()`（整板总览 + 问题标记）与 `render_crop_png()`（按 XY 局部放大），在 `_prepare_images()` 中批量生成后由 `_embed()` 写入 Excel。

同时输出 `generate_dfm_text()` 文本报告；检查中心额外导出 `dfm_hub_analysis.json`（剔除 fab_data/instances 等大字段）。

## 2.8 板厂能力库（规则数据源）

`config/dfm_capability_*.yaml`，结构：
```yaml
board_types:
  rigid:
    label: 硬板
    levels: [standard, automotive, extreme]
    level_labels: {...}
    standard:
      panel:   { max_working_panel, max_array, min_pcs, min_breakaway_rail, ... }
      trace:   { min_line_spacing_by_copper: [{copper_min_um, copper_max_um, line_um, space_um}],
                 hole_to_hole_spacing: {pth_to_pth, ...}, trace_to_board_edge: {inner, outer}, ... }
      lamination: { board_thickness: {min_mm, max_mm}, ... }
      drill:   { min_mech_drill, max_mech_drill, min_laser_via, annular_ring_mech, npth_to_edge, ... }
      plating: { pth_aspect_ratio, blind_via_fill_aspect, ... }
      plug_hole / solder_mask / silkscreen / surface_finish / test / coverlay(软板)
```
现有：`hongban`（红版）、`sihui`（四会）、`jlcpcb`（嘉立创，来自其公开能力表：按铜厚分档线宽线距、PTH/NPTH 孔环、各类间距、阻焊桥按铜厚、丝印 0.15mm/字高 1.0mm、拼板 V-Cut 25°/mouse bite/工艺边/Mark、HDI 盲埋孔、背钻、castellated、盲槽、板厚公差、阻抗 ±10%/±5%）。

单位约定：每项带 `unit`（mm/um/mil/percent/ratio），`1 mil = 0.0254 mm`，代码统一 `_to_mm()` 换算。

---

# 第三部分　使用教程

## 3.1 GUI 教程（推荐）

### A. 有 Allegro 环境：直接分析 .brd
1. 双击 `Evan_DFM.exe` → 首页点 **DFM 检查中心**
2. 数据源选 `BRD 文件 (.brd, 需 Allegro extracta)`，选择 `.brd`
3. 选板厂（红版/四会/嘉立创）、板型（硬板/软板）、能力等级、铜厚 um（oz 可留空自动换算）
4. 勾选模块：`geometry` + `netlist`（+ 可选 `capability` 需设计参数 JSON）
5. 点 **③ 执行分析**（大板约 1–2 分钟，extracta 后台无界面提取）
6. 三级页签查看：**问题清单**（含 X/Y）、**统计与评分**、**执行日志**
7. 点 **④ 打开交叉定位** → 版图高亮每个 warning/fail 的 XY，可问题↔版图双向定位
8. 点 **⑤ 导出报告** → Excel（含违例截图）+ 文本 + JSON

### B. 无 Allegro：用 SKILL 导出 JSON
1. 首页 → **分析BRD-DFM** → **① 导出SKILL脚本(.il)**（或 CLI `python main.py --skill-export`）
2. Allegro 打开 .brd，Command 窗口执行：
   ```
   skill load "brd_dfm_export.il"
   brdExportJSON                     ; 导出到 .brd 同目录
   ```
3. 回到 **DFM 检查中心**，数据源选 `导出 JSON`，选择该 JSON → **③ 执行分析**

### C. 无 BRD：试跑样例
```powershell
python main.py --dfm-hub --brd-dfm samples\brd_export_sample.json
```
或在 GUI 中选择 `samples\brd_export_sample.json` 作为数据源。

### D. 叠加制造数据与网络比对
1. `制造数据` 选 Gerber/ODB++/Drill 目录 → 勾选 `fab`（完整性）
2. `CAD 网表` 选 `{"net":["R1.1",...]}` JSON → 勾选 `fabnet`（Open/Short）
3. 执行后可在报告中看到"开短路网表"页与比对统计

### E. 参数化制程能力 + 拼版检查
1. 点 **生成设计参数模板** → 填线宽/线距/孔径/孔环/板厚/阻焊桥/丝印等 → 保存
2. 点 **生成拼版参数模板** → 填单板尺寸/WPNL 选项/工艺边/阻抗条 → 保存
3. 勾选 `capability` + `panel` → 执行 → 得到能力比对问题 + 拼版利用率/成本指数

## 3.2 CLI 教程

```powershell
# DFM 检查中心（统一编排）
python main.py --dfm-hub --brd-dfm board.brd --factory jlcpcb --board-type rigid --capability standard
python main.py --dfm-hub --brd-dfm samples\brd_export_sample.json
python main.py --dfm-hub --design-params design.json --panel-params panel.json --factory jlcpcb
python main.py --dfm-hub --brd-dfm board.brd --fab .\gerbers --cad-nets cad_nets.json
python main.py --dfm-hub --brd-dfm board.brd --hub-modules geometry,netlist --view

# 专项
python main.py --skill-export                       # 生成 SKILL 导出脚本
python main.py --brd-dfm board.brd                  # BRD DFM（几何+开短路+网表）
python main.py --dfm dfm_design_sample.json         # 参数化制程能力 DFM
python main.py --fab .\gerbers --cad-nets cad.json  # 制造数据 DFM + 网络比对
python main.py --panelize panel_params.json         # 自动拼板
python main.py --brd board.brd --checklist x.xlsx    # 约束生成 + Checklist 自动填写
```

| 参数 | 说明 |
|---|---|
| `--dfm-hub` | 启用 DFM 检查中心（多模块统一编排） |
| `--hub-modules` | 逗号分隔：`capability,geometry,netlist,fab,fabnet,panel`（默认全部） |
| `--design-params` | 制程能力 DFM 的设计参数 JSON |
| `--panel-params` | 拼版 DFM 参数 JSON |
| `--brd-dfm` | `.brd`（extracta）或导出 JSON |
| `--skill-export` | 写出 `brd_dfm_export.il` |
| `--fab` / `--cad-nets` | 制造数据目录 / CAD 网表 JSON |
| `--factory` | `hongban｜sihui｜jlcpcb` |
| `--board-type` / `--capability` | `rigid｜flex` / `standard｜automotive｜extreme` |
| `--copper` / `--copper-oz` | 成品铜厚 um / oz |
| `--view` | 分析后打开图形化交叉定位 |
| `--output` | 输出目录（默认 `output/`） |

输出：`dfm_hub_report.xlsx`、`dfm_hub_report.txt`、`dfm_hub_analysis.json`

## 3.3 实现说明（代码索引）

| 功能 | 文件 | 关键函数 |
|---|---|---|
| 多模块编排 | `src/dfm_hub.py` | `run_hub(cfg, progress)`、`_recodes()`、`CHECK_MODULES` |
| 检查中心界面 | `src/dfm_hub_ui.py` | `DfmHubWindow._build/_collect_cfg/_run/_poll/_fill_issues/_fill_stats/_open_viewer/_export` |
| BRD 提取（无界面） | `src/brd_extracta.py` | `find_extracta()`、`run_extracta()`、`build_design_dict()`、`extract_brd()` |
| SKILL 导出 | `src/skill_export.py` | `export_skill_text()`、`write_export_skill()` |
| 设计模型 + 空间索引 | `src/brd_geometry.py` | `BrdDesign`、`load_brd_export()`、`SpatialGrid`、`sweep_pairs()`、`to_fab_data()` |
| 几何 DFM | `src/dfm_geometry.py` | `analyze_geometry()`、`_build_elements()`、`_classify()` |
| 开短路/网表 | `src/dfm_netlist.py` | `analyze_netlist()`、`DSU`、`_touching()` |
| BRD 编排 | `src/brd_dfm.py` | `analyze_brd_file()`、`analyze_brd_dfm()`、`aggregate_by_rule()` |
| 能力库比对 | `src/dfm_engine.py` | `analyze_design()`、`validate_constraints()`、`load_capability()` |
| 规则/问题模型 | `src/dfm_rules.py` | `make_issue()`、`build_issue_report()`、`summarize_issues()`、`compute_risk_score()` |
| 制造数据 | `src/fab_data_reader.py` | `parse_gerber()`、`parse_excellon()`、`parse_odbpp()`、`integrity_check()`、`compare_netlists()`、`build_fab_report()` |
| 报告 | `src/dfm_report.py` | `generate_dfm_excel()`、`generate_dfm_text()`、`_prepare_images()` |
| 违例截图 | `src/dfm_snap.py` | `render_overview_png()`、`render_crop_png()` |
| 交叉定位 | `src/fab_viewer.py` / `src/gerber_viewer.py` | `FabViewer`、`GerberViewer` |
| 拼版 | `src/panel_engine.py` / `panel_ui.py` / `panel_export.py` | `design_panel()`、`PanelDesigner`、`export_png()/export_pdf()` |

`run_hub(cfg)` 返回的关键字段：
```python
{
  "factory"/"factory_label"/"board_type"/"capability_level"/"copper_um",
  "board_meta",                      # 报告封面信息
  "issues",                          # 统一编码后的问题列表（含 module/x/y/count）
  "instances",                       # 逐处实例（供交叉定位）
  "fab_data",                        # 版图几何（供 fab_viewer 渲染）
  "issue_summary", "risk",           # 分级统计 + 评分/良率/结论
  "info_stats",                      # Info 页统计
  "modules_run", "modules_requested",
  "netlist_summary", "geometry_stats", "fab_stats", "fabnet_summary", "panel",
  "errors"/"warnings"/"error_count"/"warning_count"/"passed",
  "source",                          # {type, path, fab_path}
}
```

## 3.4 扩展指南

### 新增一个板厂能力库（3 步）
1. 复制 `config/dfm_capability_hongban.yaml` → `dfm_capability_myfab.yaml`，按实测能力填值
2. `src/dfm_engine.py` 注册：
   ```python
   FACTORY_FILES["myfab"] = "dfm_capability_myfab.yaml"
   FACTORY_LABELS["myfab"] = "某某板厂"
   FACTORY_ALIASES["某某"] = "myfab"
   ```
3. 打包 spec 的 `datas` 增加该 yaml → 重新打包（界面/CLI 自动出现该板厂）

### 新增一个检查模块（4 步）
1. 写分析函数，返回 **Issue 列表**（用 `dfm_rules.make_issue(...)`，务必带 `x/y`）
2. `src/dfm_hub.py`：`CHECK_MODULES` 增加 `("mymod", "描述")`，在 `run_hub()` 内加分支并把问题 `append` 到 `issues`（记得 `it["module"]="mymod"`）
3. `src/dfm_hub_ui.py`：模块勾选自动来自 `CHECK_MODULES`；如需新输入项，在 `_build()` 加控件并在 `_collect_cfg()` 传出
4. 需要定位就在结果里提供 `fab_data`（或复用 BRD/制造数据的），`_open_viewer()` 自动可用

### 新增一条规则
- 能力库类：在 yaml 增加字段 + `dfm_engine.analyze_design` 或 `dfm_geometry.analyze_geometry` 增加一次 `add_check/emit`
- 固定阈值类（装配/拼版）：改 `dfm_rules.ASSEMBLY_RULES` / `PANEL_RULES` 或 `panel_engine.DEFAULT_PARAMS`
- 记得在 `dfm_rules.SUGGESTIONS` 补"专家建议"文案

---

# 第四部分　与华秋DFM 的差距与路线图

| 能力 | 华秋DFM | 本项目现状 | 建议路线 |
|---|---|---|---|
| 规则数量 | 120+ 制板 / 1000+ SMT | 数十条（能力库驱动） | 按 JLC/红版/四会能力表逐条补齐；规则 DSL 化 |
| 元件库/3D | 800 万+ 元件、3D 1:1 | 无 | 接 BOM + 封装库；3D 用 trimesh/导出 STEP |
| SMT/DFA | 布局干涉、爬锡、可焊性 | 基础装配项（间距/基准点/测试点/对称性/极性/BGA） | 增加干涉矩阵、钢网释放率、波峰焊阴影 |
| DFT | 测试覆盖率 | 测试点尺寸/到板边 | 覆盖率计算、探针间距、ICT/飞针 |
| 文件比对 | BOM/PCB/Gerber 差异 | CAD vs FAB 网表、Gerber 叠加对比 | 增加 BOM↔坐标↔Gerber 三方差异 |
| 阻抗/叠层 | 自动叠层阻抗计算 | 能力库含阻抗容差 | 加叠层求解器与阻抗计算 |
| 异形排料 | 智能推荐拼板 | 矩形/旋转/错位/镜像 + 凑表值 | NFP 临界靠接、真实轮廓嵌套、模拟退火 |
| 报告 | 可自定义输出项 | Valor 式固定模板 + 截图 | 模板化/可配置字段、批量导出 |
| 解析格式 | Allegro/AD/PADS/Gerber/ODB++ | BRD(extracta/SKILL)/Gerber/ODB++/Drill/DXF/PDF | AD/PADS 导入、zip/rar 包、%AM 宏与 %SR |

**近期最有价值的三步**
1. 规则库扩容：把嘉立创/红版/四会能力表全字段落库并接入 `geometry` 实测（酸角、孤立铜、过孔堆叠、焊盘间距）
2. 报告增强：违例局部放大截图逐条嵌入（已具备 `dfm_snap`）+ 结果项可勾选输出
3. SMT/DFA 扩展：元件干涉 + 钢网 + 测试覆盖率

---

# 第五部分　Gerber DFM 专题（如何像 BRD 一样分析 Gerber）

## 5.1 原理：Gerber → 同一设计模型

`.brd` 是二进制专有格式，我们用 Allegro `extracta` 提取；**Gerber 是文本图形格式**，本身只有"画什么"没有"为什么画"。
做法是写一个**适配器** `src/gerber_design.py`，把 Gerber/ODB++ 解析结果转换成与 BRD **完全相同**的设计模型（`brd_geometry.BrdDesign`），于是几何 DFM、开短路、报告、交叉定位**全部直接复用**：

```
Gerber/ODB++/Drill ─parse→ 图元(draw/arc/region/flash/holes)
        │  identify_layer(): X2 %TF.FileFunction → 文件名约定 → ODB matrix
        ▼
   build_design_dict()  →  clines / pads / vias / shapes / outline / nets / stackup
        │  无 X2 时: _derive_nets() 用网格加速的并查集按几何连通性派生网络
        ▼
   BrdDesign ─→ dfm_geometry.analyze_geometry + dfm_netlist.analyze_netlist
        ▼
   brd_dfm.analyze_design_model → Valor 式报告 + fab_viewer 交叉定位
```

**图元 → 模型映射**

| Gerber 图元 | 设计模型 | 支撑的检查 |
|---|---|---|
| `D01` draw | `clines`（线段 + **实测线宽**） | 最小线宽、线距、铜到板边 |
| `G02/G03` arc | 折线化后并入 `clines` | 同上 |
| `G36/G37` region | 边界折线（线宽 0，只按轮廓算间距）+ `shapes`(bbox) | 铺铜轮廓参与间距 |
| `D03` flash | `pads`（未匹配钻孔 = SMD） | 焊盘/测试点尺寸 |
| flash + 钻孔匹配（≤0.05mm） | `vias`（`drill_mm` + `pad_mm`） | **单边孔环 = (pad−drill)/2**、孔径、纵横比 |
| NC Drill 孔（无匹配焊盘） | `vias`（`pad_mm=None`，type=npth） | 最小/最大孔径（不误报孔环） |
| Profile/外形层 | `board.outline` | 铜到板边、板尺寸 |
| `%TO.N` / `%TO.P` | `nets` / 节点(refdes.pin) | 开路、短路、网表一致性 |
| 无 X2 属性 | 连通性派生网络 `N0001…` | 间距/孔环/板边（无法判开路） |

## 5.2 "Gerber 有很多种格式"——支持矩阵

| 格式 / 变体 | 说明 | 支持 |
|---|---|---|
| **RS-274X**（扩展 Gerber） | 主流格式，光圈内嵌 | ✅ 模态 D 码(D01/02/03 可省略)、模态插补(G01/02/03)、`%FS/%MO/%AD/%LP/%TO/%TD/%TF/%TA`、`G36/G37` 区域、`G74/G75` 单/多象限圆弧、`G70/G71`、`G04` 注释 |
| **Gerber X2** | 带 `TF/TA/TO/TD` 属性（文件功能、网络、元件、引脚） | ✅ 且用于**层自动识别**与网络/节点语义（可判开短路） |
| Gerber X3 | X2 之上的属性扩展 | ⚠️ 按 X2 处理 |
| 坐标格式 `%FSLA/%FSTA` | 前导/后随零省略、绝对/增量 | ✅ LA（绝对+前导省略）；⚠️ 增量(G91)/后随零省略(T) 未特殊处理 |
| 单位 `%MOMM/%MOIN` | mm / inch | ✅ 自动换算（inch×25.4） |
| **光圈宏 `%AM`** | 1圆/2·20矢量线/21中心线/22左下线/4轮廓/5多边形/6摩尔纹/7热焊盘 | ⚠️ 安全跳过（不渲染、不参与几何），不会污染解析 |
| **步进重复 `%SR`** | 阵列复制 | ⚠️ 未复制（跳过，不报错） |
| 极性 `%LPD/%LPC` | 暗/亮（负相） | ✅ 记录 `clear` 标记，查看器以背景色擦除 |
| `%MI/%RO/%OF/%IO/%IR/%AS` | 镜像/旋转/偏移/轴交换 | ⚠️ 未应用（实际很少用） |
| **RS-274D**（旧式） | 光圈表外置、D 码≤9 | ❌ 需外置光圈表，建议转 RS-274X |
| **Excellon NC Drill** | `M48` 头、`T#C` 工具表、METRIC/INCH、FMAT2 | ✅ 工具表/坐标/PTH-NPTH 判别；⚠️ 铣槽路由(G00/G01)、`R` 重复钻孔未展开 |
| **ODB++** | `matrix` + `layers/*/features` + `netlists/cadnet/netlist` | ✅ 层类型来自 matrix，features 支持 L/P/A，netlist 支持分号与 `+node` 两种写法 |
| IPC-2581 | XML 单一文件 | ❌ 未支持 |
| ZIP/RAR 打包 | 常见交付形式 | ⚠️ 请先解压后用"目录导入" |
| 命名约定 | AD/Protel `.GTL/.GBL/.GTS/.GBS/.GTO/.GBO/.GKO/.GM1`；KiCad `-F.Cu/-B.Cu/-F.Mask/-Edge.Cuts`；Allegro `.art`；Eagle `.cmp/.sol/.plc` | ✅ 正则启发式识别 |

## 5.3 层类型自动识别（三级策略）

1. **X2 `%TF.FileFunction`（最权威）**：`Copper,L1,Top`、`Copper,L2,Bot`、`SolderMask,Top`、`Legend,Top`、`SolderPaste,Top`、`Profile`、`Drill,PTH/NPTH`
2. **文件名启发式**（`NAME_RULES` 正则表）：`npth/drill/pth` → 钻孔；`profile/outline/edge.cuts/.gko/.gm1` → 外形；`.gts/-f.mask/smt` → 阻焊；`silkscreen/silk/.gto/.plc` → 丝印；`.gtp/paste` → 锡膏；`.gtl/-f.cu/top` → 顶层铜；`.gbl/-b.cu/bottom` → 底层铜；`.g2/.gp1/inner/inr` → 内层铜
3. **ODB++**：`matrix` 的层类型（signal/power_ground/mixed→铜，solder_mask、silk_screen、paste_mask、rout、drill）

识别结果在 **统计与评分 → "Gerber 层识别"** 中逐层可见，并标注来源（`X2` / `name` / `odb-matrix`）。铜层顺序决定"外层/内层"判定（stackup 第一层=TOP，最后一层=BOTTOM）。

## 5.4 有 X2 与无 X2 的能力差异

| 检查项 | 有 X2 属性 | 无 X2（连通性派生） |
|---|---|---|
| 最小线宽（实测） | ✅ | ✅ |
| 最小线距（不同网络铜箔） | ✅ | ✅（按派生网络） |
| 单边孔环 / 孔径 / 最大孔 | ✅ | ✅ |
| PTH 纵横比 | ✅（**需填板厚**） | ✅（需填板厚） |
| 铜/线路到板边 | ✅ | ✅ |
| 阻焊桥（需阻焊层文件） | ✅ | ✅ |
| 测试点/焊盘尺寸 | ✅ | ✅ |
| **开路 Open** | ✅（逻辑网表 vs 物理连通域） | ❌ 无逻辑网表，无法判定 |
| **短路 Short** | ✅（不同网络铜箔接触） | ⚠️ 需另提供 CAD 网表 |
| 元件/引脚级定位（refdes.pin） | ✅ | ❌ |

> 无 X2 时报告会给出提示：*"建议导出 X2 或提供 CAD 网表"*。

## 5.5 操作步骤（GUI）

1. 首页 → **DFM 检查中心**
2. 数据源选 **`Gerber 文件/目录 (RS-274X + NC Drill)`** 或 **`ODB++ 工程目录`**
3. 选择目录（含全部 Gerber + 钻孔）或单个文件
4. 选板厂（红版/四会/嘉立创）、板型、能力等级、铜厚；**填「板厚mm」**（否则纵横比不检查）
5. 勾选模块：`geometry` + `netlist`（+ `fab` 完整性；+ `fabnet` 需 CAD 网表 JSON）
6. **③ 执行分析** → 问题清单（每条含 X/Y 与数量）
7. **④ 打开交叉定位** → 版图高亮每处违例，问题↔版图双向跳转
8. **⑤ 导出报告** → Excel（Main / Issue summary / Info / 开短路网表 / Issue1..N，含违例截图）+ 文本 + JSON

## 5.6 操作步骤（CLI）

```powershell
# Gerber 几何 DFM + 开短路 + 网表（等价于以制造数据为源的 --dfm-hub）
python main.py --gerber-dfm .\gerbers --factory jlcpcb --board-thickness 1.6

# 检查中心方式：只跑几何+网表+完整性
python main.py --dfm-hub --fab .\gerbers --hub-modules geometry,netlist,fab

# 叠加 CAD 网表做 CAD vs Gerber 比对（Open/Short）
python main.py --gerber-dfm .\gerbers --cad-nets cad_nets.json --board-thickness 1.6

# 分析后直接打开交叉定位
python main.py --gerber-dfm .\gerbers --view
```

输出：`output/dfm_hub_report.xlsx` / `.txt` / `dfm_hub_analysis.json`

## 5.7 实测示例（本项目验证用例）

| 场景 | 结果 |
|---|---|
| 2 层 + 外形 + 阻焊 + PTH 钻孔（含 X2） | 层识别全部 `X2`；派生 4 clines / 4 pads / 2 vias；检出 孔环 0.05<0.18、线宽 0.08<0.10(3处)、线距 0.0<0.1(5处)、短路 6 处、网表一致性；16 个定位实例 |
| 同样图形但**去掉 X2** | 连通性派生出 `N0001/N0002` 两个网络；检出 孔环 + 线宽(2处)，无误报短路；报告提示"建议导出 X2" |

## 5.8 已知限制与建议

- **`%AM` 宏光圈 / `%SR` 步进重复未展开**：若板子大量使用（如自定义散热焊盘、阵列马克点），几何会缺失 → 建议改用标准光圈、或提供 ODB++/`.brd`
- **Gerber 无板厚与叠层**：纵横比需手填板厚；层数由铜层文件数推断
- **阻焊桥**依赖阻焊层文件（开窗间距推算）
- 丝印/阻焊层作为独立层元素，不参与"不同网络铜箔间距"
- 增量坐标(G91)、后随零省略(T)、RS-274D、IPC-2581 暂不支持
- 大文件（>10 万图元）建议先看 `统计与评分 → 几何统计`，必要时分模块执行

---

# 附录　常见问题（FAQ）

| 问题 | 处理 |
|---|---|
| 点"执行分析"报 extracta 未找到 | 需本机安装 Cadence Allegro（含 `extracta.exe`）；或改用 SKILL 导出 JSON；或先用 `samples/brd_export_sample.json` 试跑 |
| .brd 分析很慢 | 大板 1–2 分钟属正常（后台线程，界面不卡）；可只勾 `geometry` 或限制模块 |
| 没有 X/Y 定位 | 需 BRD/JSON 或制造数据源；纯参数化（capability）为参数级判定，无坐标 |
| 交叉定位打不开/空白 | 说明本次无 `fab_data`（未提供 BRD/JSON 或制造数据） |
| 网络比对提示"无 X2 网络属性" | Gerber 未导出 `%TO.N/%TO.P`，请在 EDA 导出时启用 X2 属性，或改用 ODB++ netlist |
| 换板厂后结果不变 | 确认能力等级存在（如嘉立创 rigid 只有 standard/extreme）；等级下拉会随板厂/板型联动 |
| exe 双击无反应 | 必须整目录分发（含 `_internal`）；不要单独拷贝 exe |
| 界面/图标模糊 | 已内嵌 DPI 感知 manifest（PerMonitorV2）；若任务栏仍旧图标，重启 explorer 或重新固定 |
| 打包缺模块 | 检查 `Evan_DFM.spec` / `Asher_DFM.spec` 的 `hiddenimports` 与 `datas`（能力库 yaml、samples、图标） |
| Gerber 分析只有完整性问题、没有线宽/孔环 | 未勾选 `geometry` 模块，或数据源选成"无源文件"；请在数据源选 Gerber/ODB 目录并勾选 geometry |
| Gerber 纵横比没有检查 | Gerber 不含板厚信息，请在"板厚mm"填写（或 CLI `--board-thickness`） |
| Gerber 层识别错误 | 优先用 X2（导出时启用 `%TF.FileFunction`）；或按常见命名（.gtl/.gbl/.gts/.gko/-F.Cu/-Edge.Cuts）；识别结果见"统计与评分 → Gerber 层识别" |
| Gerber 报不出开路 | 无 X2 网络属性时无法判定开路（网络由连通性派生）；请导出 X2 或提供 CAD 网表做 `fabnet` 比对 |
| Gerber 图形缺失/焊盘不显示 | 可能使用了 `%AM` 宏光圈或 `%SR` 步进重复（当前跳过）；改用标准光圈或提供 ODB++ |