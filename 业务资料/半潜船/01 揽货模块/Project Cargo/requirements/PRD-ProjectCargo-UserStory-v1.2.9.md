# Project Cargo User Story

## 文档信息


| 项目   | 内容                                                       |
| ---- | -------------------------------------------------------- |
| 文档名称 | Project Cargo User Story                                 |
| 版本   | v1.2.9                                                   |
| 创建日期 | 2026-06-04                                               |
| 更新日期 | 2026-10-10                                               |
| 依据文档 | PRD-ProjectCargo-功能清单 v1.2.9；Project Cargo-需求变更清单 v1.1.0 |
| BA   | 邵艺晨                                                      |
| 评审人  | —                                                        |


> **改造范围**：本文档描述 Project Cargo 页面整体重构后的目标行为。旧版本所有 Tab 结构及其子模块（批次、COA、竞争对手、阶段、Note/Report/Attachment/RFQ/Deliverables）全部移除；页面改为「列表页 + 详情页（Basic Info + Bidding Part）」结构。

---



## 一、列表页

---



### US-L01：项目列表展示

**作为** 揽货业务人员  
**我想要** 在分页表格中查看所有有权限的项目的核心信息  
**以便于** 快速了解在手项目的状态和关键数据

**验收标准**：

- [x] 列表展示列：No.、Priority、Project Market、Project Type（多值以分号分隔）、Project Name、Project Status、End Client、Direct Client、Project Period、Project Location、PID、Status、Primary Person(s) in Charge、Last Updated、Operate
- [ ] **行粒度**：默认一行对应一个项目；仅当该项目下存在多个不同 Direct Client 时，按 Direct Client 拆分为多行；拆行时项目级列在每行完整重复展示，不合并单元格
- [ ] **未拆行时**：Direct Client 展示该项目唯一直接客户；PID、Status、Primary Person(s) in Charge 取该项目所有投标块的数据
- [ ] **拆行时**：每行展示一个 Direct Client；PID、Status、Primary Person(s) in Charge 仅取该 Direct Client 下关联投标块的数据；项目级列各行重复展示相同值
- [ ] **多投标块展示**：同一可见行包含多个投标块时，PID 与 Status 在各自单元格内按投标块顺序分行展示并一一对齐，每个 Status 对应同一内部行的 PID；PIC 按人员去重后以分号分隔
- [ ] 除 Status 外，每列支持三态排序（初始/升序/降序循环切换），排序后序号从 1 重排
- [ ] 仅展示当前用户有查看权限范围内的项目
- [ ] 列表同时展示置顶记录（置顶行优先显示）和普通记录
- [ ] 无编辑权限的用户，操作列不显示编辑入口
- [ ] 界面全部英文，不提供中英文切换

**业务规则**：

- Project Status 展示值来自 Basic Info 中用户手工选择并保存的 Project Status 字段
- Status 展示值来自对应 PID 所属投标块的 Status 字段；列表快捷编辑规则见 US-L09
- 不因投标块数量多而自动拆行，仅以 Direct Client 数量是否大于 1 决定是否拆行
- 行操作与进入详情默认定位规则见 US-L04

**前置条件**：

- 用户已登录并具备 Project Cargo 模块访问权限

**后置条件**：

- 无数据变更

**优先级**：P0

---



### US-L02：列表条件筛选

**作为** 揽货业务人员  
**我想要** 通过多维度条件筛选项目列表  
**以便于** 快速定位目标项目

**验收标准**：

- [ ] 筛选在标准列表与 View Major Projects 视图下均生效，仅列结构不同，数据范围相同
- [ ] **Float-over Only 快捷筛选**：列表页提供 Float-over Only 按钮；点击后追加条件「项目 Project Type 集合包含 Float-over」，可与筛选面板其他条件 AND 叠加；再次点击取消该快捷条件；按钮激活态与条件同步；Reset 时一并清除
- [ ] 筛选器默认收起，点击筛选按钮后展开
- [ ] 支持以下筛选条件：Priority（下拉单选）、Project Market（表格单选）、Project Type（**标签式多选**：下拉点选追加为标签，可继续追加、可逐条 × 移除；项目 Type 集合包含所选全部值即命中）、Project Name（模糊文本）、Project Status（下拉单选）、End Client（表格单选）、Direct Client（表格单选）、Project Period（年月区间双选器）、Project Location（模糊文本）、PID（文本，多个以分号分隔）、Primary Person(s) in Charge（**表格单选**）、Last Updated（**年月日选择器**）
- [ ] 点击「Search」按筛选条件刷新列表，置顶记录始终保留
- [ ] 点击「Reset」清空所有条件（含 Float-over Only 快捷筛选）后停留在筛选面板，不自动收起
- [ ] 项目编号筛选不区分大小写，模糊匹配
- [ ] **Direct Client 筛选**：项目下任一投标块 Direct Client 与所选值一致即命中；若该项目因多 Direct Client 拆行，仅展示命中客户的拆分行
- [ ] **PID 筛选**：项目下任一投标块 PID 命中即纳入该项目；拆行展示时仅保留含命中 PID 的拆分行
- [ ] **Primary Person(s) in Charge 筛选**：项目下任一投标块 PIC 命中即纳入该项目；拆行展示时仅保留含命中 PIC 的拆分行

**业务规则**：

- 筛选命中以项目为判断单位，展示行数由 US-L01 拆行规则决定
- 筛选结果受数据权限约束，仅返回用户有查看权限的项目

**前置条件**：

- 用户已进入 Project Cargo 列表页

**后置条件**：

- 列表按筛选条件刷新，置顶行保留

**优先级**：P0

---



### US-L03：列表分页

**作为** 揽货业务人员  
**我想要** 自由控制每页展示条数并在页间跳转  
**以便于** 按需浏览大量项目数据

**验收标准**：

- [ ] 每页条数支持 10/20/50/100 切换
- [ ] 支持页码跳转输入框
- [ ] 序号（No.）按列表可见行连续编排
- [ ] 切换条数、筛选或排序后，序号从 1 重新开始计算

**业务规则**：

- 无

**前置条件**：

- 无特殊前置

**后置条件**：

- 无

**优先级**：P0

---



### US-L04：行操作 — 查看 / 编辑 / 置顶

**作为** 揽货业务人员  
**我想要** 对列表中的项目执行查看、编辑或置顶操作  
**以便于** 快速进入详情维护数据或将重要项目保持置顶可见

**验收标准**：

- [ ] 列表行默认对应项目；仅多 Direct Client 拆行时，每行对应一个 Direct Client 维度
- [ ] 点击查看图标：以只读锁定模式打开详情页，所有字段不可编辑，标题为该行项目名称
- [ ] 点击编辑图标：以编辑模式打开详情页，标题为该行项目名称；无编辑权限的用户该图标不显示
- [ ] **进入详情默认定位**：未拆行项目默认打开首个投标块；拆行项目默认定位到与该行 Direct Client 关联的投标块（该客户下有多块时默认第一块）
- [ ] 点击置顶图标：将该行置顶至列表最前；已置顶后图标变更，再次点击取消置顶
- [ ] 置顶按列表可见行粒度，持久化：退出登录后重新进入仍然保留，用户主动取消前不消失
- [ ] 置顶数量达 50 条上限时，提示「Pin limit reached (50).」并取消本次置顶操作

**业务规则**：

- 置顶为当前登录用户私有，其他用户不可见
- 无编辑权限的用户进入详情后为全局只读模式，Save 按钮不显示

**前置条件**：

- 列表中已有项目记录

**后置条件**：

- 置顶变更持久化保存；查看/编辑跳转至详情页

**优先级**：P0

---



### US-L05：View Major Projects 视图

**作为** 揽货业务人员  
**我想要** 将当前列表切换为 Major Project Tracker 展示模式  
**以便于** 以线下惯用的重点项目跟踪表格式查看和导出当前在跟项目

**验收标准**：

- [ ] 列表页提供「View Major Projects」按钮
- [ ] 点击后在**不改变当前筛选条件及置顶结果**的前提下，将列表切换为 Major Project Tracker 展示模式；按钮呈激活/选中态
- [ ] 再次点击退出，恢复标准列表视图，按钮恢复未激活态
- [ ] **工具栏**：筛选面板、Float-over Only 快捷筛选、New Project 等按钮始终保留；仅表格区域切换列结构与展示规则
- [ ] **列结构（23 列，A–W）**：A-Project Name、B-Priority、C-Market、D-Project Type（多值分号分隔）、E-Project Location、F-Project Description、G-Project Period、H-End Client、I-Project Status（项目级，A–I）；J-PID、K-Direct Client、L-Cargo/Scope、M-Vessel(s)、N-Laycan、O-POL、P-POD、Q-Project Developments、R-Likelihood to Award、S-Status、T-Commercial Remark（Freight/Dayrate）、U-Technical Remark（Challenge/Restrictions）、V-Expected Time to Award、W-PIC（投标块级，J–W）
- [ ] **展示规则**：同一项目多个投标块时，A–I 列纵向合并；J–W 列按投标块分行；No./查看/编辑/置顶按投标块粒度，进入详情默认定位对应投标块
- [ ] 具备编辑权限的用户可在 Major 视图对指定投标块字段执行页内编辑，规则见 US-L08

**业务规则**：

- 与标准列表视图展示同一批记录，仅列结构与行展示规则不同（标准视图见 US-L01）

**前置条件**：

- 用户已进入 Project Cargo 列表页

**后置条件**：

- 列表以 Major Project Tracker 格式展示

**优先级**：P1

---



### US-L06：导出项目清单

**作为** 揽货业务人员  
**我想要** 将项目列表数据导出为 Excel 文件  
**以便于** 用于线下汇报、统计或与外部共享

**验收标准**：

- [ ] 列表页右上角提供**同一导出按钮位**「Export Project List」（标准列表与 View Major Projects 视图下按钮名与下拉交互一致，均保留下拉箭头），不并列两个导出按钮
- [ ] 点击后以下拉菜单提供两个选项（两种视图均相同）：
  - [ ] 方式一「Export filtered records / 导出当前筛选结果」（原 Export filtered & Pinned records）：直接生成并下载 xlsx，**仅导出当前筛选命中的记录**，**不再额外汇出置顶记录**（置顶行若同时命中当前筛选，仍随筛选结果导出）
  - [ ] 方式二「选择导出」：No. 列切换为勾选框，默认全选；用户勾选后导出 xlsx
- [ ] **标准列表视图**：导出列与标准列表一致，包含 PID 后的投标块 Status 列；同一可见行含多个投标块时，导出结果中的 PID 与 Status 按相同顺序一一对应（含多 Direct Client 拆行规则）
- [ ] **View Major Projects 视图**：导出列结构与合并规则与 Major 视图一致；文件名 `Major Project Tracker_YYYYMMDD_HHmmss.xlsx`；文件顶部显示 `Last updated - YYYYMMDD`；「选择导出」在 Major 视图下同样可用

**业务规则**：

- 导出数据受数据权限约束，仅包含用户有查看权限的项目
- 方式一导出范围为**当前筛选结果**，不因置顶而追加未命中筛选的记录；Major 视图同此规则
- 两种视图仅导出文件命名与列模板不同，按钮交互保持一致

**前置条件**：

- 用户具备导出权限

**后置条件**：

- 生成 xlsx 文件并触发浏览器下载

**优先级**：P1

---



### US-L07：新建项目

**作为** 揽货业务人员  
**我想要** 点击「New Project」进入新建详情页  
**以便于** 创建一个新的 Project Cargo 项目

**验收标准**：

- [x] 点击「New Project」后跳转至详情页，页面标题固定为「Project Cargo」
- [x] 新建页面中 Project Status 默认为 Budgetary，Creator 自动填入当前登录用户（创建该项目的用户）且**不可修改**
- [x] Bidding Part 初始包含一个空投标块，PIC 默认填入当前登录用户，待用户填写
- [x] 在 View Major Projects 视图下点击 New Project，行为与标准视图一致

**业务规则**：

- 新建时项目编号（PID）不显示，保存后系统自动生成

**前置条件**：

- 用户具备新建项目的操作权限

**后置条件**：

- 跳转至详情页新建模式

**优先级**：P0

---



### US-L08：Major 视图页内编辑

**作为** 揽货业务人员  
**我想要** 在 View Major Projects 视图下直接编辑并保存指定投标块字段  
**以便于** 无需进入详情页即可快速更新重点项目跟踪信息

**验收标准**：

- [ ] 仅在 **View Major Projects** 视图下生效；具备编辑权限的用户可对下列 5 个投标块级字段直接编辑并保存，无编辑权限用户上述列只读；Project Status 在 Major 视图中不支持页内编辑，标准列表的快捷编辑规则见 US-L09
- [ ] **S-Status**（投标块 Status）：下拉选项 Open/Budgetary/Firm/Fixed/Closed/Lost，不含 Booked，与 US-C01 一致
- [ ] **Q-Project Development**：文本框
- [ ] **V-Expected Time to Award**：年月选择器，非必填
- [ ] **T-Commercial Remark（Freight/Dayrate）**：文本框
- [ ] **U-Technical Remark（Challenge/Restrictions）**：文本框
- [ ] 可编辑列以浅色内嵌控件（文本框/下拉/年月选择器）**常驻展示**，无需点击单元格再进入编辑态；控件样式与表格协调
- [ ] 编辑完成（失焦或控件确认）后自动保存；保存成功后列表即时刷新展示最新值
- [ ] A–I 项目级合并单元格垂直居中对齐
- [ ] 与详情页共用同一数据源，各字段对应当前投标块独立存储
- [ ] 投标块 Status 为 Booked、Closed 或 Lost 时，除 S-Status 外该投标块行的 Q/V/T/U 只读（与 US-C04 一致）

**业务规则**：

- 页内编辑不改变 Major 视图的列结构与合并规则
- Project Development 为投标块级字段，各投标块独立维护

**前置条件**：

- 用户已进入 View Major Projects 视图
- 用户具备编辑权限

**后置条件**：

- 对应投标块字段数据持久化，列表展示最新值

**优先级**：P1

---


### US-L09：标准列表状态快捷编辑

**作为** 揽货业务人员  
**我想要** 在标准列表中直接修改项目状态和指定 PID 的投标块状态  
**以便于** 无需进入详情页即可快速维护项目及投标进展

**验收标准**：

- [ ] 本功能仅在标准列表视图下生效；具备编辑权限的用户可通过常驻下拉框快捷修改 Project Status 和投标块 Status，无编辑权限用户只读展示
- [ ] Project Status 为项目级字段，下拉选项为 Budgetary、Firm、Proceed、Closed
- [ ] 同一项目因多个 Direct Client 拆分为多行时，在任一拆分行修改 Project Status，均更新同一项目，其他拆分行同步刷新
- [ ] Status 为投标块级字段，下拉选项为 Open/Budgetary、Firm、Fixed、Closed、Lost，不提供手工选择 Booked；当前值为 Booked 时仍展示该系统状态
- [ ] 单行仅包含一个 PID 时，PID 后展示一个 Status 下拉框
- [ ] 单行包含多个 PID 时，PID 与 Status 在各自单元格内按投标块顺序分行并一一对齐，每个 PID 对应一个独立 Status 下拉框
- [ ] 修改某个 Status 时，仅更新该 PID 对应的投标块，不影响同一项目下其他投标块
- [ ] 用户确认下拉选项后自动保存；保存成功后列表、详情页和其他视图同步展示最新值，并更新 Last Updated
- [ ] 保存失败时提示失败原因、恢复原值，不更新 Last Updated

**业务规则**：

- Project Status 与 Basic Info 的 Project Status 共用同一数据源
- Status 与对应 PID 所属 Bidding Part 的 Status 共用同一数据源
- Booked 由合同关联结果自动产生，不作为手工下拉选项
- 投标块 Status 为 Booked、Closed 或 Lost 时，Status 字段本身仍按既有状态规则处理，其他字段锁定规则见 US-C04
- Major 视图继续执行 US-L08；其中 Project Status 保持只读

**前置条件**：

- 用户已进入 Project Cargo 标准列表视图
- 用户具备编辑权限

**后置条件**：

- 对应项目或投标块状态持久化保存，所有相关页面展示一致

**优先级**：P0

---



## 二、详情页全局操作

---



### US-D01：全局保存

**作为** 揽货业务人员  
**我想要** 通过一次点击保存详情页全部内容  
**以便于** 高效提交当次所有编辑成果

**验收标准**：

- [ ] 点击「Save」后，Basic Info 和所有投标块内容一并保存
- [ ] 每个尚未生成项目编号的投标块在保存后自动生成唯一编号（PID）
- [ ] 保存成功后不自动退出详情页，用户可继续编辑
- [ ] 保存前对所有必填字段执行校验，校验不通过时高亮错误字段并阻止提交
- [ ] 无编辑权限时 Save 按钮不显示

**业务规则**：

- PID 由系统保存时自动生成，不可手工录入或修改

**前置条件**：

- 用户具备编辑/保存权限
- 处于编辑模式（非只读）

**后置条件**：

- 数据持久化，各投标块生成或保留 PID，详情页保持打开

**优先级**：P0

---



### US-D02：关闭详情页（含未保存确认）

**作为** 揽货业务人员  
**我想要** 点击「Close」安全退出详情页并返回列表  
**以便于** 完成操作后回到列表继续工作，同时避免意外丢失未保存数据

**验收标准**：

- [ ] 无未保存修改时，点击「Close」直接返回列表
- [ ] 存在未保存修改时，弹出「You have unsaved changes. Do you want to save?」弹窗
- [ ] 弹窗提供三个按钮：Cancel（留在当前页继续编辑）、No（放弃修改直接返回列表）、Yes（先保存后返回列表）

**业务规则**：

- 保存失败时（如必填项未填）不关闭详情页，等待用户处理错误后重试

**前置条件**：

- 用户处于详情页（新建、编辑或只读模式均适用）

**后置条件**：

- 返回列表页（列表刷新）或留在详情页

**优先级**：P0

---



## 三、Basic Info

---



### US-B01：维护项目基本信息

**作为** 揽货业务人员  
**我想要** 填写并保存项目的基础属性  
**以便于** 为本项目建立完整的基本信息档案，支撑后续投标工作

**验收标准**：

- [x] 以下字段为必填，保存前校验：Project Name、Priority（Normal/High）、Project Status（Budgetary/Firm/Proceed/Closed）、Project Market、Project Type（**标签式多选**，Float-over/Time/Voyage/Multiple Voy，至少选一项；下拉点选后以标签保留在输入框内，可继续追加、可逐条 × 移除）、Project Period（起止年月）、Project Location、Creator、Project Description
- [x] Project Status 提供「项目推进可能性」说明文本（带 i 图标提示：About how likely this project will process as a firm）
- [x] Project Period 使用年月粒度选择器（分起、止两个年月选择器）
- [x] Creator：必填，项目创建人，**不可修改**；新建时自动填充为当前登录用户（即创建该 Project 的用户）；已保存项目再次打开时展示已保存值，仍不可修改
- [x] End Client 为非必填，可通过表格选择，也可通过「增加」入口临时新建客户
- [x] Project Description 文本框随内容自动调整高度，展示全量文本，不截断

**业务规则**：

- Project Status 默认值为 Budgetary
- Project Status 与列表页 Project Status 列展示值保持一致（一旦保存，列表立即更新）
- Project Type 标签式多选时，列表展示多值以分号分隔
- Creator 由系统在创建时写入，全程只读，不提供 Select User 弹窗改人
- 临时新增的 End Client 在审批通过前仅本项目可引用，不对全局客户主数据生效

**输入框初始化**：

- Project Market：通过基础数据域市场类型表格组件查询，包含 No.、Name、EN Name、Code 列
- End Client：通过客商域客户主数据表格组件查询，包含 No.、Code、CN Name、EN Name 列
- Creator：系统自动填充当前登录用户，只读展示，不通过选人弹窗初始化或修改

**前置条件**：

- 用户处于详情页编辑或新建模式

**后置条件**：

- 项目基本信息保存后列表同步更新

**优先级**：P0

---



## 四、Bidding Part — 投标块

---



### US-C01：维护投标块商务信息

**作为** 揽货业务人员  
**我想要** 在每个投标块中维护完整的商务信息  
**以便于** 记录针对某一直接客户的投标方案要素

**验收标准**：

- [x] 以下字段必填：Charterer/Direct Client、Likelihood to Award（High/Medium/Low，带 i 提示：How likely this will be awarded to us; select from the following options: Low, Medium, High）、Status（Open/Budgetary/Firm/Fixed/Closed/Lost）、PIC、POL(s)（支持多选）、POD(s)（支持多选）、Planned Laycan（年月区间）、Cargo/Scope/Updates、Proposed Vessel(s)/Shipments（弹窗多选）
- [x] **Project Development**（非必填）：文本框，自动调整高度；位于本投标块 Cargo/Scope/Updates 与 Team 之间
- [x] **Expected Time to Award**（非必填）：年月选择器，预计授标时间；用户可留空不填
- [x] **Commercial Remark（Freight/Dayrate）**（非必填）：文本框
- [x] **Technical Remark（Challenge/Restrictions）**（非必填）：文本框
- [x] **Shared Workspace**（非必填）：文本字框
- [ ] PID 仅在保存后由系统自动生成显示，初始编辑期间为空
- [x] PIC 默认取当前投标块创建人，点击字段唤起弹窗（Select User）可修改
- [x] Planned Laycan 可勾选「Same as Basic Info」，勾选后与 Basic Info 的 Project Period 起止自动同步
- [x] POL 和 POD 均支持多选，港口选项来自港口主数据
- [x] Proposed Vessel(s)/Shipments 通过「Proposed Vessel(s)」弹窗多选船舶，选项来自船舶主数据；选船并 **Save** 后，驱动 Firm/Send to Scenario 弹窗船期列表按规则同步（见 US-Q01）
- [x] 投标块操作区 Firm/Send to Scenario、Proposal Check、Technical Support、Save 按钮**始终展示**，不因 Status 值变化而隐藏
- [ ] 当 Status 为 Booked、Closed 或 Lost 且**已点击 Save 保存后**，除 Status 字段本身外其余字段全部只读；仅变更 Status 下拉、尚未 Save 时其余字段仍可编辑
- [ ] Status 下拉选项不含 Booked；存在合同关联该投标块编号时，状态由系统自动写入 Booked

**业务规则**：

- 投标块标题格式为「PID Client Name」，保存生成 PID 后自动展示
- 直接客户对应旧架构中的 Charterer 字段，迁移时由 Charterer 映射而来
- Booked 状态只由系统在存在合同关联时自动写入，用户无法手工选择 Booked
- Team 默认包含创建该投标块的用户
- Project Development 为投标块级字段，各投标块独立维护
- Proposed Vessel(s)/Shipments 变更并 Save 后，与 Scenario 弹窗船期行的增删同步规则见 US-Q01；**已成功 Send to Master Scenario 并写入船期表的记录本身不受影响**，须在 Master Scenario / 船期表侧独立修改

**输入框初始化**：

- Charterer/Direct Client：通过客商域客户主数据表格组件查询（包含 No.、Code、CN Name、EN Name）
- POL(s)/POD(s)：通过基础数据域港口主数据下拉多选查询
- Proposed Vessel(s)/Shipments：通过「Proposed Vessel(s)」弹窗从基础数据域船舶主数据多选（支持搜索；表格字段为 Vessel No.、Vessel Name、Vessel Type；勾选后点「确定」确认）
- PIC：通过系统管理域用户弹窗（Select User）查询

**前置条件**：

- 处于详情页编辑模式

**后置条件**：

- 投标块数据随块级保存或全局保存提交

**优先级**：P0

---



### US-C02：多投标块管理

**作为** 揽货业务人员  
**我想要** 为同一项目添加多个独立的投标块，针对不同客户制定投标方案  
**以便于** 在同一项目下管理与多个潜在客户的商务谈判

**验收标准**：

- [ ] Bidding Part 区域提供「+」按钮，点击后新增一个空白投标块，结构与其他块相同
- [ ] 尚未保存且未生成 PID 的投标块，标题旁显示「×」，点击后弹出确认，确认后删除该块
- [ ] 已生成 PID 的投标块，「×」不显示，不可删除
- [ ] 关闭投标通过将投标块 Status 设为「Closed」并保存来完成，无独立「Close 投标」按钮

**业务规则**：

- 每个投标块保存后生成各自独立的 PID，互不相同
- 同一项目下的各投标块相互独立，修改一个块不影响其他块
- 同一项目下多个不同 Direct Client 时，列表页按 US-L01 规则拆行展示

**前置条件**：

- 处于详情页编辑模式

**后置条件**：

- 新增块待用户填写后保存；删除块立即从页面移除

**优先级**：P0

---



### US-C03：投标块级保存

**作为** 揽货业务人员  
**我想要** 为单个投标块执行独立保存  
**以便于** 在不影响其他块的情况下为本块生成编号并锁定数据

**验收标准**：

- [ ] 每个投标块提供独立「Save」按钮
- [ ] 保存成功后，该块生成唯一 PID，并在块标题中展示
- [ ] 块级保存不影响其他投标块和 Basic Info 的状态
- [ ] 块级保存前对本块必填字段执行校验，不通过则高亮未填必填字段并阻止保存

**业务规则**：

- 块级保存粒度小于详情页全局保存；全局保存覆盖所有块

**前置条件**：

- 处于编辑模式，本投标块字段已填写

**后置条件**：

- 本块数据持久化，编号生成

**优先级**：P0

---



### US-C04：关闭投标（Status = Closed）

**作为** 揽货业务人员  
**我想要** 将投标块状态设为 Closed 来关闭某次投标  
**以便于** 标记该方向投标已结束，且保留历史数据可供查阅

**验收标准**：

- [ ] 投标状态下拉包含 Closed 选项
- [ ] 选择 Closed 并**点击 Save 保存后**，该块除 Status 字段外其余字段变为只读
- [ ] 仅将 Status 改为 Closed/Lost、尚未 Save 时，其余字段仍可编辑；Firm/Send to Scenario、Proposal Check、Technical Support 等操作按钮始终展示

**业务规则**：

- Closed 后仍可将状态改回（如重新激活）；只读锁定在**保存后**按当前 Status 生效，未保存前不锁定
- Lost 以及 Booked 状态与 Closed 一样，在保存后触发只读锁定（除 Status 外）

**前置条件**：

- 投标块已保存生成 PID

**后置条件**：

- 投标块进入只读锁定态（除 Status 外）

**优先级**：P0

---



## 五、Proposal / BAFO Review

---



### US-P01：商务方案文件管理

**作为** 揽货业务人员  
**我想要** 为投标块上传、预览、下载和删除商务方案文件  
**以便于** 统一管理报价单、预算计算等商务支撑文件

**验收标准**：

- [ ] 每个投标块内有独立的「Upload Commercial Proposal（Freight & Precal）」上传区
- [ ] 点击「Upload」打开本地文件选择器，支持多文件同时选择；上传成功后文件写入下方表格
- [ ] 文件表格展示：No.、File Name、Uploaded by、Upload time、Operate
- [ ] 操作列每行提供预览（图标）、下载（图标）、删除（图标）三个按钮
- [ ] 删除前弹出二次确认；确认删除后该行从表格移除，不再参与 Proposal Check 邮件附件
- [ ] 上传失败时页面提示失败信息，不写入表格

**业务规则**：

- 同一上传区可多次上传追加，不限制为单文件
- 商务方案与技术方案两个上传区各自独立，互不影响
- Proposal Check 弹窗（US-Q03）中对应 Remark 字段标题展示为 **Commercial Remark（Freight/Dayrate）**

**前置条件**：

- 投标块已存在（新建或编辑模式均可）

**后置条件**：

- 文件存储至文件服务，表格更新

**优先级**：P1

---



### US-P02：技术方案文件管理

**作为** 揽货业务人员  
**我想要** 为投标块上传、预览、下载和删除技术方案文件（积载图等）  
**以便于** 集中管理配载技术支撑文件

**验收标准**：

- [ ] 每个投标块内有独立的「Upload Technical Proposal（Stowage Plan）」上传区
- [ ] 交互规则和文件表格结构与商务方案文件管理完全一致
- [ ] 两个上传区各自独立，商务文件和技术文件分区显示

**业务规则**：

- 同商务方案文件管理规则
- Proposal Check 弹窗（US-Q03）中对应 Remark 字段标题展示为 **Technical Remark（Challenge/Restrictions）**

**前置条件**：

- 投标块已存在

**后置条件**：

- 文件存储至文件服务，表格更新

**优先级**：P1

---



## 六、投标块快捷操作

---



### US-Q01：Scenario 弹窗 — 管理 Shipment 方案

**作为** 揽货业务人员  
**我想要** 通过 Scenario 弹窗为投标块管理具体的拟议航次方案  
**以便于** 规划每批货物的运输安排并最终发送至船期表

**验收标准**：

- [ ] 点击投标块中「Firm/Send to Scenario」按钮打开 Scenario 弹窗
- [ ] 弹窗头部展示当前投标块 PID 和 Project Name（只读）
- [ ] **与 Proposed Vessel(s)/Shipments 联动填充**：在投标块 Proposed Vessel(s)/Shipments 中选择船舶并 **Save** 后，所选船舶自动填充到本弹窗船期列表行（多船多行）；打开弹窗时展示当前已同步的船期行
- [ ] 打开弹窗或点击「ADD」添加行时，系统按规则自动填充：PID、Project Name、Vessel、Start/End time（日期默认取当月 1 日）、Start Port、End Port、Status（默认 Firm）、PIC
- [ ] **Vessel 可选范围**：修改 Vessel 或通过 + ADD 新增行时，下拉可选船舶**仅限于**当前 Proposed Vessel(s)/Shipments 已选船舶，不可从全量船舶主数据任选
- [ ] 未锁定行的各字段可在弹窗内修改。其中，Start time 保持不变；修改 Days 时，系统根据 Start time 自动计算 End time；修改 End time 时，系统根据 Start time 自动反算 Days
- [ ] Window span(day) 可通过现有 Project Window 弹窗选择配置
- [ ] 状态字段可选：Firm（默认）、Fixed、Booked
- [ ] **弹窗不提供行级删除按钮**；船期行减少仅通过 Proposed Vessel(s)/Shipments 变更并 Save 后的同步规则实现（见下），或通过 Master Scenario 侧对已绑定记录的操作回传
- [ ] **Proposed Vessel 变更后再 Save 的同步规则**：
  - [ ] 已成功 **Send to Master Scenario** 的船期行**保留**，不被删除
  - [ ] 船舶已不在当前 Proposed Vessel(s) 中、且**尚未**发送至 Master Scenario（未置灰锁定）的船期行，由系统**自动移除**
  - [ ] 新增的 Proposed Vessel 自动补入本弹窗船期行

**业务规则**：

- Scenario 弹窗中 Shipment 行的 Status（Firm/Fixed/Booked）与项目状态、投标块状态无联动关系，独立维护
- 船期行初始值可来自投标块（Proposed Vessel、Planned Laycan、POL/POD、PIC 等），填充后未锁定行可改；但 Proposed Vessel 变更并 Save 时，按上述增删规则与投标块保持同步，**不**视为「与投标块完全脱钩」
- 上述同步仅影响弹窗内船期行列表；**已写入 Master Scenario 的正式记录本身不受 Proposed Vessel 变更影响**，须在 Master Scenario 侧独立操作
- Scenario 弹窗内**无**行级 Delete / 删除入口；不可通过弹窗手工删行

**输入框初始化**：

- Vessel：选项仅来自当前投标块 Proposed Vessel(s)/Shipments 已选船舶（船舶主数据经 Proposed Vessel 弹窗选定后的子集）
- PIC：通过系统管理域用户弹窗（Select User）查询，自动填充后可修改

**前置条件**：

- 投标块已存在（含未保存新块，但建议保存后操作以确保编号已生成）

**后置条件**：

- Shipment 方案数据保存在弹窗中；发送至 Master Scenario 后与船期表建立绑定并锁定对应行

**优先级**：P0

---



### US-Q02：发送至船期表

**作为** 揽货业务人员  
**我想要** 将 Scenario 弹窗中已规划好的 Shipment 方案分别发送至 Personal Scenario 或 Master Scenario  
**以便于** 在个人船期表中试排航次，或将方案正式纳入 Master Scenario 并接收后续变更回传

**验收标准**：

- [ ] Scenario 弹窗提供两个独立发送按钮：**Send to Personal Scenario**、**Send to Master Scenario**
- [ ] 选中 Shipment 行后方可操作发送按钮
- [ ] **Send to Personal Scenario**：
  - [ ] 发送成功后，在当前用户的 Personal Scenario 中生成对应船期记录，命名格式为「PID-Project Name-Shipment#1」（同投标块多条依序 #2、#3…递增）
  - [ ] 发送成功后，弹窗内对应行**不置灰、不锁定**，仍可继续编辑及再次发送
  - [ ] Personal Scenario 侧对该记录的修改**不回传**至弹窗对应行
- [ ] **Send to Master Scenario**：
  - [ ] 发送成功后，在 Master Scenario 中生成对应船期记录，命名格式同上
  - [ ] 发送成功后，弹窗内对应行**置灰锁定**：禁止再次 **Send to Master Scenario**、禁止行内字段**编辑**；后续编辑或删除须在 Master Scenario 中操作（弹窗本身无行级删除按钮）
  - [ ] 锁定行右侧出现**向右箭头**，点击可跳转至 Master Scenario 页面查看/操作对应记录
  - [ ] 已因 Master 发送置灰锁定的行**仍可勾选**，可继续执行 **Send to Personal Scenario**
  - [ ] 弹窗行与 Master Scenario 记录建立绑定关系；Master Scenario 侧对该记录的删改**同步回传**至弹窗对应行
- [ ] **重复发送校验（点击时批量校验）**：点击 **Send to Personal Scenario** 或 **Send to Master Scenario** 时，系统对当前**已勾选**的全部 Shipment 行调用船期表域接口执行重复判定：
  - [ ] **Send to Personal Scenario**：逐行校验当前用户在 Personal Scenario 中是否已存在与该 Shipment 待发送内容**完全相同**的船期记录；若存在则该行不可发送（校验不通过）
  - [ ] **Send to Master Scenario**：已发送至 Master 并置灰锁定的行视为已存在，校验不通过
  - [ ] 若勾选行中存在任一不满足重复校验的行，则**整批发送失败**，不写入任何船期记录；不满足重复校验的行，参与判定的 Shipment 字段文本框/控件**标红**提示
  - [ ] 重复判定字段范围：Vessel、Start Time、Days、End Time、Start Port、End Port、Window span(day)、Status、PIC（与船期表域「完全相同」判定口径一致）
  - [ ] 发送按钮**不因重复预判而置灰**；重复问题在点击发送时以校验失败 + 字段标红方式反馈
  - [ ] 用户修正标红字段或取消勾选问题行后，可再次点击发送
- [ ] **已成功写入船期表（尤其是 Master Scenario）的记录**：后续修改投标块其他字段**不影响**该船期表记录本身；如需修改须在船期表 / Master Scenario 页面独立操作
- [ ] **例外（弹窗未锁定行）**：修改 Proposed Vessel(s)/Shipments 并 Save 时，按 US-Q01 规则增删弹窗内**未** Master 锁定的船期行；已 Master 锁定行保留；不因此删除或改写已写入船期表的正式记录

**业务规则**：

- 「发送至船期表」为后台数据同步操作，发送成功后提示成功；**发送动作本身不跳转**至船期表菜单页面
- 仅 **Send to Master Scenario** 成功后的行进入置灰锁定态，并展示向右箭头供跳转 Master Scenario；Personal 发送不触发锁定与跳转箭头
- Master 置灰锁定限制：禁止再次 Send to Master、禁止行内编辑；不限制勾选与 Personal 发送；可通过行右侧箭头跳转 Master Scenario；弹窗无行级删除按钮，锁定行的删除仅能在 Master Scenario 侧操作
- 重复校验在点击发送时执行；任一勾选行不通过则整批失败，标红重复项字段

**前置条件**：

- Scenario 弹窗中存在已填写的 Shipment 行
- 对应投标块已生成编号（PID）
- 至少勾选一条待发送的 Shipment 行

**后置条件**：

- Personal 发送成功：Personal Scenario 生成记录；弹窗行保持可编辑（除已成功发送至 Master Scenario 的记录）
- Master 发送成功：Master Scenario 生成绑定记录；弹窗行置灰锁定（仍可勾选发送 Personal；行右侧出现跳转箭头）
- 重复校验失败：不写入船期表；问题行重复判定字段标红

**优先级**：P1

---



### US-Q03：Proposal Check — 方案审阅与团队邮件

**作为** 揽货业务人员  
**我想要** 通过 Proposal Check 弹窗整理审阅内容并向团队发送审核邮件  
**以便于** 确保方案在提交客户前经过团队内部评审

**验收标准**：

- [ ] 点击「Proposal Check」按钮打开审阅弹窗
- [ ] 弹窗顶部提供邮件开头文本编辑区，用户可自由编辑；提供「Set as Template」按钮，将当前内容保存为个人模板，下次打开弹窗时自动载入
- [ ] 弹窗中可编辑字段标题与投标块保持一致：**Commercial Remark（Freight/Dayrate）**、**Technical Remark（Challenge/Restrictions）**、Team、Shared Workspace
- [ ] 点击「Request Team Review」：向当前投标块 Team 成员发送邮件；邮件正文含用户编辑后的开头文本及项目基本信息；邮件附件为两类上传区当前文件表格中仍存在的全部文件（已删除的不含）
- [ ] 发送成功后弹窗关闭，返回投标块
- [ ] 发送失败时弹窗提示失败，提供「Resend」和「Cancel」两个选项
- [ ] 点击「Cancel」关闭弹窗，不发送邮件

**业务规则**：

- 邮件附件范围以当前页面两个文件表格中仍可见的文件为准，历史已删除文件不含
- 个人邮件模板仅对当前用户生效，不影响其他用户

**前置条件**：

- 投标块已有至少一名团队成员

**后置条件**：

- 邮件发出后团队成员收到审核请求邮件

**优先级**：P1

---



### US-Q04：Technical Support（配载申请）

**作为** 揽货业务人员  
**我想要** 为投标块发起配载技术支持申请  
**以便于** 获得技术团队的配载方案支持

**验收标准**：

- [ ] 点击「Technical Support」按钮，触发配载申请流程
- [ ] 交互逻辑与原有 Stowage Apply 页面完全一致

**业务规则**：

- Technical Support 为原 Stowage Apply 功能的更名，业务逻辑不变

**前置条件**：

- 投标块已存在

**后置条件**：

- 配载申请流程启动

**优先级**：P1

---



## 七、团队与客户管理

---



### US-T01：项目团队管理（Team 弹窗）

**作为** 揽货业务人员  
**我想要** 通过弹窗管理当前投标块的团队成员  
**以便于** 明确参与该投标的各角色人员及联系方式

**验收标准**：

- [ ] 在 Commercial Bid 区域点击「Team」打开团队管理弹窗
- [ ] 弹窗内功能与原有 Teams 页面完全一致：字段、角色类型、增删成员等均沿用现有逻辑
- [ ] 团队默认包含创建该投标块的用户
- [ ] 改造内容仅为将 Teams Tab 改为弹窗呈现，功能能力不变

**业务规则**：

- 参见现有 Teams 页面业务规则（1:1 一致）

**输入框初始化**：

- 角色列表：通过系统管理域查询项目类角色
- 用户选择：通过系统管理域用户数据查询（按角色过滤）

**前置条件**：

- 处于编辑模式

**后置条件**：

- 团队成员变更随投标块保存时一并提交

**优先级**：P0

---



### US-T02：客户信息临时新增

**作为** 揽货业务人员  
**我想要** 在选择 End Client 或 Charterer/Direct Client 时快速新建当前系统中尚未登记的客户  
**以便于** 不中断当前项目填写流程即可完成客户信息的创建

**验收标准**：

- [ ] End Client 和 Charterer/Direct Client 选择器旁提供「增加」按钮
- [ ] 点击后弹出客户信息管理弹窗，可填写必要信息完成临时新增
- [ ] 临时新增的客户可立即被当前项目字段引用
- [ ] 提交审批通过后，该客户方正式写入全局客户主数据；审批通过前不对全局主数据生效，其他项目无法选用该客户

**业务规则**：

- 客户审批流程由客商域和工作流域共同处理，具体审批链路待业务方提供

**前置条件**：

- 处于编辑模式，需选择一个系统中尚未登记的客户

**后置条件**：

- 临时客户可在当前项目中使用；审批后写入全局

**优先级**：P2

---



## 八、待确认事项


| 序号  | 待确认问题                                                                                      | 影响功能                               |
| --- | ------------------------------------------------------------------------------------------ | ---------------------------------- |
| 1   | 权限矩阵待业务提供：查看范围划分维度（按团队/角色/负责人等）；编辑权限控制规则；导出、新建、保存、状态变更、标准列表状态快捷编辑、Major 视图页内编辑、发送 Scenario 等操作权限的角色清单 | US-L01、US-L04、US-D01、US-L08、US-L09、US-Q02 |
| 2   | 客户临时新增的审批链路：具体审批节点、审批角色、审批通过条件                                                             | US-T02                             |


---



## 附录一：字段映射表



### A. 列表页字段（标准列表视图）


| 页面字段名                              | 代码字段名                                      | 字段来源说明                            |
| ---------------------------------- | ------------------------------------------ | --------------------------------- |
| Priority（优先级）                      | PRIORITY（1=High，0=Normal）                  | 保留旧字段                             |
| Project Market（市场类型）               | MARKET / MARKET_EN_NM                      | 保留旧字段，迁移数据                        |
| Project Type（项目类型）                 | PROJECT_TYPE / PROJECT_TYPE_LIST           | **标签式多选**，多值分号分隔展示                |
| Project Name（项目名称）                 | PROJECT_ENM                                | 保留旧字段                             |
| Project Status（项目状态）               | STATUS（新值体系：Budgetary/Firm/Proceed/Closed） | 旧字段，值体系重建                         |
| End Client（终端客户）                   | 新字段（待命名）                                   | 全新字段                              |
| Direct Client（直接客户）                | PARTNER_FNM2 / PARTNER_CD                  | 未拆行：唯一客户；拆行：每行一个客户                |
| Project Period（项目时段）               | PLAN_START_DT / PLAN_END_DT（精度由日改为年月）      | 旧字段，精度调整                          |
| Project Location（项目地点）             | 新字段（待命名）                                   | 全新字段                              |
| PID（项目编号）                          | PROJECT_CD                                 | 未拆行：所有块合集；拆行：该 Direct Client 下块合集 |
| Status（投标块状态）                     | STATUS（投标块层，新值体系）                       | 与 PID 按投标块顺序分行并一一对应               |
| Primary Person(s) in Charge（主要负责人） | PROJECT_MGT_ENM（各投标块 PIC 合集）               | 未拆行：所有块 PIC 合集；拆行：该客户下 PIC 合集     |
| Last Updated（最后更新）                 | MODIFIED_DT                                | 保留旧字段                             |




### B. 列表筛选条件字段


| 页面字段名                           | 代码字段名                            | 备注                      |
| ------------------------------- | -------------------------------- | ----------------------- |
| Float-over Only 快捷筛选            | PROJECT_TYPE_LIST                | 项目 Type 集合包含 Float-over |
| Priority（筛选）                    | PRIORITY                         |                         |
| Project Market（筛选）              | MARKET / MARKET_EN_NM            |                         |
| Project Type（筛选）                | PROJECT_TYPE_LIST                | **标签式多选**               |
| Project Name（筛选）                | PROJECT_ENM                      | 模糊匹配                    |
| Project Status（筛选）              | STATUS                           | 新值体系                    |
| End Client（筛选）                  | 新字段                              |                         |
| Direct Client（筛选）               | PARTNER_CD / PARTNER_FNM2        |                         |
| Project Period（筛选）              | PLAN_START_DT / PLAN_END_DT      | 年月粒度                    |
| Project Location（筛选）            | 新字段                              | 模糊匹配                    |
| PID（筛选）                         | PROJECT_CD                       | 多值分号分隔                  |
| Primary Person(s) in Charge（筛选） | PROJECT_MGT_CD / PROJECT_MGT_ENM | 表格单选                    |
| Last Updated（筛选）                | MODIFIED_DT                      | 年月日选择器                  |




### C. Basic Info 字段


| 页面字段名               | 代码字段名                        | 备注                 |
| ------------------- | ---------------------------- | ------------------ |
| Project Name        | PROJECT_NM / PROJECT_ENM     |                    |
| Priority            | PRIORITY                     |                    |
| Project Status      | STATUS                       | 新值体系               |
| Project Market      | MARKET / MARKET_EN_NM        |                    |
| End Client          | 新字段（待命名）                     | 全新字段               |
| Project Type        | PROJECT_TYPE_LIST            | **标签式多选**          |
| Project Period（起）   | PLAN_START_DT（年月精度）          |                    |
| Project Period（止）   | PLAN_END_DT（年月精度）            |                    |
| Project Location    | 新字段（待命名）                     | 全新字段               |
| Creator（创建人）        | 新字段（待命名，区别于旧 PROJECT_MGT_CD） | 创建时自动填充，不可修改       |
| Project Description | 新字段 或 CARGO_DESC（待确认）        | 旧字段曾为货物描述，需确认复用或新建 |




### D. 投标块（Commercial Bid）字段


| 页面字段名                                    | 代码字段名                                 | 备注                                           |
| ---------------------------------------- | ------------------------------------- | -------------------------------------------- |
| Charterer / Direct Client                | PARTNER_EN_NM / PARTNER_CD            | 旧 Charterer 改名                               |
| Likelihood to Award（中标概率）                | 新字段（待命名）                              | 全新字段                                         |
| PID（投标块编号）                               | PROJECT_CD（各块独立）                      |                                              |
| Status（投标块状态）                            | STATUS（投标块层，新值体系）                     |                                              |
| PIC（负责人）                                 | 新字段，区别于 PROJECT_MGT_CD                | 各投标块独立 PIC                                   |
| POL(s)（装货港）                              | POL_CD / POL_EN_NM                    | 多选                                           |
| POD(s)（卸货港）                              | POD_CD / POD_EN_NM                    | 多选                                           |
| Planned Laycan（计划受载期）                    | LAYCAN_START_DT / LAYCAN_END_DT（年月精度） |                                              |
| Cargo/Scope/Updates                      | 新字段                                   |                                              |
| Project Development                      | 新字段（待命名）                              | **投标块级**，位于 Cargo/Scope 与 Team 之间            |
| Proposed Vessel(s)                       | PLAN_VESSEL_CD / PLAN_VESSEL_EN_NM    | 弹窗多选（Vessel No./Name/Type）；Save 后联动 Scenario |
| Expected Time to Award                   | 新字段（待命名）                              | 非必填，**年月选择器**                                |
| Commercial Remark（Freight/Dayrate）       | 新字段                                   | 投标块级                                         |
| Technical Remark（Challenge/Restrictions） | 新字段                                   | 投标块级                                         |
| Team                                     | 团队相关字段（PROJECTTEAM_ID 等）              | 功能不变，弹窗呈现                                    |
| Shared Workspace                         | 新字段（待命名）                              | 全新字段                                         |




### E. Proposal/BAFO Review 文件字段


| 页面字段名             | 代码字段名           | 备注  |
| ----------------- | --------------- | --- |
| No.（序号）           | 展示用，系统生成        |     |
| File Name（文件名）    | ATTACH_NM       |     |
| Uploaded by（上传人）  | MAKER_ENM       |     |
| Upload time（上传时间） | UPDATELOAD_TIME |     |




### F. Scenario — Shipment 表字段


| 页面字段名                | 代码字段名                      | 备注                                                          |
| -------------------- | -------------------------- | ----------------------------------------------------------- |
| PID                  | PROJECT_CD                 | 自动填充                                                        |
| Project Name         | PROJECT_ENM                | 自动填充                                                        |
| Vessel（船舶）           | VESSEL_CD / VESSEL_EN_NM   | 可改；选项仅限 Proposed Vessel(s) 已选船；随 Proposed Vessel Save 联动增删行 |
| Start Time（开始时间）     | LAYCAN_START_DT（日级精度）      | 自动填充后可改（Master 锁定行不可改）                                      |
| Days（天数）             | STAGE_DAYS                 | 与 Start/End 互算                                              |
| End Time（结束时间）       | LAYCAN_END_DT（日级精度）        | 自动填充后可改（Master 锁定行不可改）                                      |
| Start Port（起始港）      | POL_CD / POL_EN_NM         | 自动填充后可改                                                     |
| End Port（结束港）        | POD_CD / POD_EN_NM         | 自动填充后可改                                                     |
| Window span(day)     | 现有窗口期字段                    |                                                             |
| Status（Shipment 行状态） | 新字段（待命名，Firm/Fixed/Booked） | 独立于投标块/项目状态                                                 |
| PIC（Shipment 行负责人）   | 与投标块 PIC 对应字段              | 可改                                                          |
| Master 锁定态 / 跳转箭头    | —                          | Send to Master 成功后置灰锁定禁编辑；弹窗无行级删除按钮；行右侧箭头跳转 Master Scenario |


---



## 变更记录


| 版本     | 日期         | 变更内容                                                                                                                                                                                             | 变更人 |
| ------ | ---------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | --- |
| v1.0.0 | 2026-06-04 | 基于逆向工程版本及需求分析文档编写                                                                                                                                                                                | 邵艺晨 |
| v1.1.0 | 2026-06-08 | 对齐功能清单 v1.1.0：多 Direct Client 拆行、Float-over Only 快捷筛选、View Major Projects 视图、导出按钮随视图切换、Project Type 多选、Project Development 移至投标块、Expected Time to Award（年月选择器）、Remark 副标题、列表/筛选/行操作/Major 视图全量修订 | 邵艺晨 |
| v1.1.1 | 2026-06-08 | Project Development 修正为投标块级（各块独立），非 Tab 级共享                                                                                                                                                      | 邵艺晨 |
| v1.2.0 | 2026-06-08 | 新增 US-L08：Major 视图页内编辑（投标块 Status、Project Development、Expected Time to Award、Commercial/Technical Remark）                                                                                        | 邵艺晨 |
| v1.2.1 | 2026-06-08 | US-L08 修正：Status 指投标块 Status（列 S），非 Project Status（列 I）                                                                                                                                          | 邵艺晨 |
| v1.2.2 | 2026-06-08 | 对齐功能清单 v1.2.2：US-L05 补充 A–W 列字母映射及 US-L08 交叉引用；US-C01 补充 Shared Workspace；US-P01/P02 补充 Proposal Check 字段标题；US-B01/US-T01 措辞修订                                                                   | 邵艺晨 |
| v1.2.3 | 2026-06-09 | 对齐功能清单 v1.2.3 及原型确认：L02 标签式多选/ PIC 单选/ Last Updated 日期；L04 置顶提示；L06 导出双选项一致；L08 常驻控件；B01/C01/C04 状态锁定与按钮展示规则                                                                                     | 邵艺晨 |
| v1.2.4 | 2026-06-09 | US-Q02 澄清：Personal/Master 发送行为分离——Personal 不锁定、不回传；Master 锁定并回传 Master 侧变更；Personal 发送前校验 Personal Scenario 是否已存在完全相同记录，存在则按钮置灰；Master 不做重复校验，以发送后行锁定作为已发送反馈                                     | 邵艺晨 |
| v1.2.5 | 2026-06-11 | US-Q02 修订：Master 发送后置灰锁定行仍可勾选并 Send to Personal；重复校验改为点击 Personal/Master 发送时批量执行，任一勾选行不通过则整批失败并对重复判定字段标红，取消发送前按钮置灰预判                                                                             | 邵艺晨 |
| v1.2.6 | 2026-08-07 | 船期联动修订：US-C01 Proposed Vessel 改为弹窗多选（Vessel No./Name/Type）并 Save 驱动 Scenario；US-Q01 Vessel 仅限 Proposed Vessel、增删同步与 Master 锁定行保留；US-Q02 Master 锁定禁删+箭头跳转、区分发送不跳转与箭头跳转、澄清已发船期表记录不受投标块改动影响（未锁定行除外） | 邵艺晨 |
| v1.2.7 | 2026-08-07 | US-L06：Export filtered & Pinned records 改为仅导出当前筛选结果（不再额外汇出置顶）；US-B01/US-L07：Creator 创建时自动填充且全程不可修改，取消 Select User 改人                                                                             | 邵艺晨 |
| v1.2.8 | 2026-08-07 | US-Q01/Q02：明确 Scenario 弹窗不提供行级删除按钮；行减少仅靠 Proposed Vessel 同步或 Master Scenario 侧操作；Master 锁定口径改为禁编辑（去掉暗示弹窗内可删的「禁删」措辞）                                                                              | 邵艺晨 |
| v1.2.9 | 2026-10-10 | US-L01/L06：标准列表在 PID 后新增投标块 Status，多 PID 时按投标块分行对齐并同步导出；新增 US-L09：标准列表支持 Project Status 与投标块 Status 下拉快捷编辑；US-L08 进一步区分标准列表与 Major 视图的 Project Status 编辑规则 | 邵艺晨 |
