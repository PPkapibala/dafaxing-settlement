# 大发行结算平台 - IAA利润分成模式 PRD V1.0

**版本**：V1.0  **日期**：2026-04-23  **状态**：初稿

---

## 文档说明

本文档为大发行结算平台**IAA利润分成模式**的产品需求文档（Phase 1），即全部变现收入-投放成本的利润分成模式。

**IAA利润分成模式**：以微小/抖小实际产生的投放成本为计算基础，系统自动从微小/抖小/巨量后台按应用ID拉取收入与投放成本，无需财务手工录入，无需进行收入对账，公式简洁、数据来源清晰。

**IAA+IAP利润分账模式**（Phase 2，后续规划）：在投放成本基础上叠加云服务成本项、支付通道费用项、预留资金池等复杂扣减逻辑，适用于需要精细化成本核算的合作场景。

两个模式共用打款、BPM 审批、银企直联等基础能力，Phase 1 建设完成后可平滑扩展至 Phase 2。本文档只讨论Phase 1。

---

## 一、背景与目标

### 1.1 背景

公司在大发行模式下，需要与多个发行商和cp方进行收入分账与打款结算。后续大发行的走IAA利润分成模式的游戏会进行自动结算分账，需要通过本平台实现。

本期选择数据依赖最简的**IAA利润分成模式**作为第一阶段建设目标。

### 1.2 目标

- 以发行商在平台（微信小游戏 / 抖音小游戏 / 巨量引擎）下的应用ID为取数依据，自动拉取投放收入与投放成本，财务只需要录入分账公式，系统自动根据结算周期，计算账期内可结算金额
- 按 `发行商分账金额 = （投放收入 - 投放成本）× 分成比例` 完成自动化结算计算
- 支持发行商自主发起打款申请，走 BPM 审批流，银企直联自动打款，形成完整结算闭环

### 1.3 数据来源

1. 数据口径来源-微信小游戏

<grid cols="3">

  <column width="33">
    <image token="GnLvbouC1oZ6Mrxpo9icvQEQnbg" width="1920" height="879" align="center"/>

    <image token="ISJNbXK14o3QKhx0vhQc3CgUnVc" width="1920" height="879" align="center"/>

    R_cash = 现金总收入=买量分成收入+自然量分成收入
  </column>
  <column width="33">
    <image token="EusEbnpBoov92gxq75FcK7S4nlh" width="1920" height="879" align="center"/>

    
  </column>
  <column width="33">
    <image token="I8Njb6O92oNWvYxrVfUcIPnWnVc" width="1920" height="879" align="center"/>

    - C = 买量消耗
    
  </column>

</grid>

1. 数据口径来源-抖音小游戏

<grid cols="3">

  <column width="33">
    <image token="FSMyb0H17oL7FBxTRBfceGYYnOh" width="1920" height="879" align="center"/>

    R_cash = 现金总收入
  </column>
  <column width="33">
    <image token="ZpSob2iTXolWYIxhXUncR3LEnWe" width="1920" height="879" align="center"/>

    - R_voucher= 广告金总收入
  </column>
  <column width="33">
    <image token="EL2rbHdPboBi1hxJTS2cfDe9nxe" width="1872" height="866" align="center"/>

    - C = 买量消耗【巨量引擎】
  </column>

</grid>

---

## 二、核心概念

### 2.1 IAA分成公式

<mention-doc token="JqAtwkCeoigBWJk0zjccj64zn1e" type="wiki">混变游戏分成逻辑-C2</mention-doc>参考业务梳理分成公式

分成公式主要约束了IDS和发行商以及对应cp分得总利润的比例，先从平台拉取收入和成本，计算总利润，再根据比例计算各方可得利润即为可结算金额。

例如发行商60%，IDS20%，cp20%

#### 总利润计算公式

现金利润=现金收入-现金成本

广告金利润=广告金收入-广告金成本

- 总收入=现金收入+广告金收入
- 现金收入=买量收入+自然量收入
- 现金成本=总消耗成本*（现金收入/总收入）
- 广告金成本=总消耗成本*（广告金收入/总收入）

#### 可结算金额（利润）

发行商可结算金额：现金利润+广告金利润，其中现金利润部分可提取打款，广告金利润只能查看

CP分账利润：本期可自动计算cp现金利润

### 2.3 结算维度

结算以 **发行商 + 游戏 + 平台类型** 为独立核算单元。

同一发行商可能在微信和抖音分别推广同一款游戏，两个平台分别独立计算分账金额并出账。发行商发起打款时，需要分开提交申请。

- 总广告利润金理论上可以折算为现金，但是本期内容，**广告金只用于查看明细和汇总，不直接折算现金**
- 发行商可以查看现金和广告金利润，可以主动提取现金部分金额
- **cp只能先查看现金和广告金利润**，不能主动提款，需要等财务线下处理

---

## 三、角色与权限

### 3.1 角色定义

<lark-table rows="8" cols="2" header-row="true" column-widths="350,350">

  <lark-tr>
    <lark-td>
      角色
    </lark-td>
    <lark-td>
      描述
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      **系统管理员**
    </lark-td>
    <lark-td>
      维护游戏字典（含游戏名称，ID，上架平台信息）、分配用户权限
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      **业务负责人**
    </lark-td>
    <lark-td>
      维护发行商主体与银行账户
      配置游戏-平台与发行商关系
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      **业务运营**
    </lark-td>
    <lark-td>
      配置发行商ID与微小游戏ID映射、与抖小游戏ID映射、与巨量账户ID映射（巨量账户ID为接口同步）
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      **财务人员**
    </lark-td>
    <lark-td>
      配置分账规则、查看分账明细、处理退票/失败异常
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      **财务负责人/审批人**
    </lark-td>
    <lark-td>
      审批分成规则配置单、审批打款申请单
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      **发行商**
    </lark-td>
    <lark-td>
      查看本方结算数据与打款记录、发起打款申请
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      **cp**
    </lark-td>
    <lark-td>
      查看本方结算数据
    </lark-td>
  </lark-tr>
</lark-table>

### 3.2 权限矩阵

<lark-table rows="12" cols="6" header-row="true" column-widths="122,122,122,122,122,122">

  <lark-tr>
    <lark-td>
      功能模块
    </lark-td>
    <lark-td>
      发行商
    </lark-td>
    <lark-td>
      财务人员
    </lark-td>
    <lark-td>
      财务负责人/审批人
    </lark-td>
    <lark-td>
      业务负责人
    </lark-td>
    <lark-td>
      系统管理员
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      游戏管理
    </lark-td>
    <lark-td>
      查看（权限范围内）
    </lark-td>
    <lark-td>
      查看全量
    </lark-td>
    <lark-td>
      查看全量
    </lark-td>
    <lark-td>
      查看（权限范围内）
    </lark-td>
    <lark-td>
      创建/编辑/删除
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      发行商主体管理
    </lark-td>
    <lark-td>
      查看（权限范围内）
    </lark-td>
    <lark-td>
      查看全量
    </lark-td>
    <lark-td>
      查看全量
    </lark-td>
    <lark-td>
      创建/编辑（权限范围内）
    </lark-td>
    <lark-td>
      查看全量
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      发行商-银行账户管理
    </lark-td>
    <lark-td>
      查看（权限范围内）
    </lark-td>
    <lark-td>
      查看全量
    </lark-td>
    <lark-td>
      查看全量
    </lark-td>
    <lark-td>
      创建/编辑（权限范围内）
    </lark-td>
    <lark-td>
      查看全量
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      游戏-平台-发行商-微小抖小巨量后台ID关系
    </lark-td>
    <lark-td>
      查看（权限范围内）
    </lark-td>
    <lark-td>
      查看全量
    </lark-td>
    <lark-td>
      查看全量
    </lark-td>
    <lark-td>
      创建/编辑（权限范围内）
    </lark-td>
    <lark-td>
      查看全量
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      cp主体管理
    </lark-td>
    <lark-td>
      查看（权限范围内）
    </lark-td>
    <lark-td>
      查看全量
    </lark-td>
    <lark-td>
      查看全量
    </lark-td>
    <lark-td>
      创建/编辑（权限范围内）
    </lark-td>
    <lark-td>
      查看全量
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      游戏-平台-发行商分成规则配置
    </lark-td>
    <lark-td>
      查看（权限范围内）
    </lark-td>
    <lark-td>
      创建/提交审批
    </lark-td>
    <lark-td>
      审批
    </lark-td>
    <lark-td>
      审批/查看
    </lark-td>
    <lark-td>
      查看全量
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      分账明细查看
    </lark-td>
    <lark-td>
      查看（权限范围内）
    </lark-td>
    <lark-td>
      查看全量
    </lark-td>
    <lark-td>
      查看全量
    </lark-td>
    <lark-td>
      查看权限范围
    </lark-td>
    <lark-td>
      全量查看
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      打款申请发起
    </lark-td>
    <lark-td>
      发起（权限范围内）申请
    </lark-td>
    <lark-td>
      查看全量
    </lark-td>
    <lark-td>
      查看全量
    </lark-td>
    <lark-td>
      查看权限范围
    </lark-td>
    <lark-td>
      查看全量
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      打款审批
    </lark-td>
    <lark-td>
      查看（权限范围内）
    </lark-td>
    <lark-td>
      审批
    </lark-td>
    <lark-td>
      审批
    </lark-td>
    <lark-td>
      审批
    </lark-td>
    <lark-td>
      查看全量
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      打款结果查看
    </lark-td>
    <lark-td>
      查看（权限范围内）
    </lark-td>
    <lark-td>
      查看全量
    </lark-td>
    <lark-td>
      查看全量
    </lark-td>
    <lark-td>
      查看（权限范围内）
    </lark-td>
    <lark-td>
      查看全量
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      失败处理
    </lark-td>
    <lark-td>
      查看（权限范围内）
    </lark-td>
    <lark-td>
      查看全量
    </lark-td>
    <lark-td>
      查看全量
    </lark-td>
    <lark-td>
      查看（权限范围内）
    </lark-td>
    <lark-td>
      查看全量
    </lark-td>
  </lark-tr>
</lark-table>

---

## 四、功能模块详细设计

### 4.1 模块一：游戏管理

#### 4.1.1 模块定位

游戏是结算系统的基础字典，由系统管理员统一维护。每个游戏支持配置多个上架平台（微信小游戏、抖音小游戏）

#### 4.1.2 游戏基础字段

<lark-table rows="7" cols="4" header-row="true" column-widths="183,183,183,183">

  <lark-tr>
    <lark-td>
      字段
    </lark-td>
    <lark-td>
      类型
    </lark-td>
    <lark-td>
      必填
    </lark-td>
    <lark-td>
      说明
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      游戏ID
    </lark-td>
    <lark-td>
      String
    </lark-td>
    <lark-td>
      系统生成
    </lark-td>
    <lark-td>
      全局唯一
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      游戏名称
    </lark-td>
    <lark-td>
      String
    </lark-td>
    <lark-td>
      是
    </lark-td>
    <lark-td>
      游戏全称
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      游戏编码
    </lark-td>
    <lark-td>
      String
    </lark-td>
    <lark-td>
      是
    </lark-td>
    <lark-td>
      系统内唯一标识，创建后不可修改
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      备注
    </lark-td>
    <lark-td>
      String
    </lark-td>
    <lark-td>
      否
    </lark-td>
    <lark-td>
      可选说明
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      创建人
    </lark-td>
    <lark-td>
      Reference
    </lark-td>
    <lark-td>
      系统生成
    </lark-td>
    <lark-td>
      系统管理员
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      创建时间
    </lark-td>
    <lark-td>
      DateTime
    </lark-td>
    <lark-td>
      系统生成
    </lark-td>
    <lark-td>
      —
    </lark-td>
  </lark-tr>
</lark-table>

#### 4.1.3 功能需求

- **REQ-G-101**：系统应支持系统管理员新增、编辑游戏基础信息；游戏编码创建后不可修改，修改时字段禁用。
- **REQ-G-102**：系统应支持在游戏下新增、编辑上架平台信息；同一游戏下同一平台类型只允许一条记录（唯一性约束），重复时拒绝保存并提示。
- **REQ-G-103**：当游戏游戏从未被引用时（即没有绑定关系的历史记录），可以删除，已经被引用，不可以删除；
- **REQ-G-104**：游戏列表及上架平台信息由业务运营、业务负责人编辑，其他角色不显示编辑入口。

#### 4.1.4 游戏-平台-微小、抖小、巨量后台ID-发行商关系字段（子表，一个游戏一个平台可配置多条）

<lark-table rows="13" cols="4" header-row="true" column-widths="183,183,183,183">

  <lark-tr>
    <lark-td>
      字段
    </lark-td>
    <lark-td>
      类型
    </lark-td>
    <lark-td>
      必填
    </lark-td>
    <lark-td>
      说明
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      记录ID
    </lark-td>
    <lark-td>
      String
    </lark-td>
    <lark-td>
      系统生成
    </lark-td>
    <lark-td>
      —
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      所属游戏
    </lark-td>
    <lark-td>
      Reference
    </lark-td>
    <lark-td>
      系统生成
    </lark-td>
    <lark-td>
      关联游戏
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      平台类型
    </lark-td>
    <lark-td>
      Enum
    </lark-td>
    <lark-td>
      是
    </lark-td>
    <lark-td>
      微信小游戏 / 抖音小游戏（微信小游戏需要配置微信小游戏平台方应用ID；抖音小游戏需要配置抖小应用ID和巨量平台账户ID）
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      发行商ID
    </lark-td>
    <lark-td>
    </lark-td>
    <lark-td>
      是
    </lark-td>
    <lark-td>
      从发行商主体列表选择
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      发行商名称
    </lark-td>
    <lark-td>
    </lark-td>
    <lark-td>
      是
    </lark-td>
    <lark-td>
      从发行商主体列表选择
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      微信小游戏平台方应用ID
    </lark-td>
    <lark-td>
      String
    </lark-td>
    <lark-td>
      是
    </lark-td>
    <lark-td>
      与平台后台应用ID保持一致，手动填写，可以一个平台应用对应多个微小后台应用
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      抖音小游戏平台方应用ID
    </lark-td>
    <lark-td>
      String
    </lark-td>
    <lark-td>
      是
    </lark-td>
    <lark-td>
      与平台后台应用ID保持一致，手动填写，一个平台应用只允许对应一个抖小应用ID
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      巨量平台方账户ID
    </lark-td>
    <lark-td>
      String
    </lark-td>
    <lark-td>
      是
    </lark-td>
    <lark-td>
      每日从巨量同步到平台供业务选择，可多选，已经选择的不允许删除
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      巨量平台方账户名称
    </lark-td>
    <lark-td>
      String
    </lark-td>
    <lark-td>
      是
    </lark-td>
    <lark-td>
      每日从巨量同步到平台供业务选择
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      备注
    </lark-td>
    <lark-td>
      String
    </lark-td>
    <lark-td>
      否
    </lark-td>
    <lark-td>
      可选说明
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      创建人
    </lark-td>
    <lark-td>
      Reference
    </lark-td>
    <lark-td>
      系统生成
    </lark-td>
    <lark-td>
      系统管理员
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      创建时间
    </lark-td>
    <lark-td>
      DateTime
    </lark-td>
    <lark-td>
      系统生成
    </lark-td>
    <lark-td>
      —
    </lark-td>
  </lark-tr>
</lark-table>

#### 4.1.5 游戏-平台-CP关系字段（子表，一个游戏只能配置一条）

<lark-table rows="7" cols="4" header-row="true" column-widths="183,183,183,183">

  <lark-tr>
    <lark-td>
      字段
    </lark-td>
    <lark-td>
      类型
    </lark-td>
    <lark-td>
      必填
    </lark-td>
    <lark-td>
      说明
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      记录ID
    </lark-td>
    <lark-td>
      String
    </lark-td>
    <lark-td>
      系统生成
    </lark-td>
    <lark-td>
      —
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      所属游戏
    </lark-td>
    <lark-td>
      Reference
    </lark-td>
    <lark-td>
      系统生成
    </lark-td>
    <lark-td>
      关联游戏
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      CPID
    </lark-td>
    <lark-td>
    </lark-td>
    <lark-td>
      是
    </lark-td>
    <lark-td>
      从CP主体列表选择
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      CP名称
    </lark-td>
    <lark-td>
    </lark-td>
    <lark-td>
      是
    </lark-td>
    <lark-td>
      从CP主体列表选择
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      创建人
    </lark-td>
    <lark-td>
      Reference
    </lark-td>
    <lark-td>
      系统生成
    </lark-td>
    <lark-td>
      系统管理员
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      创建时间
    </lark-td>
    <lark-td>
      DateTime
    </lark-td>
    <lark-td>
      系统生成
    </lark-td>
    <lark-td>
      —
    </lark-td>
  </lark-tr>
</lark-table>

#### 4.1.6功能需求

- **REQ-R-101**：系统应支持业务负责人创建微小、抖小、巨量后台ID-发行商关系字段；根据平台类型填写对应ID字段。
- **REQ-R-102**：微信小游戏平台一个游戏可以有多个发行商，抖音平台一个游戏只可以有一个发行商
- **REQ-R-103**：关系创建后，系统将其作为分账计算引擎的取数依据；按照游戏-平台-发行商进行规则创建
- **REQ-R-104**：一个游戏只能对应一个cp，一个cp可以对应多个游戏，重复配置时需要提醒
- **REQ-R-105**：业务负责人的数据权限范围由被配置的游戏决定，只能查看和管理自己所属游戏的关系。
- **REQ-R-106**：关系与分成规则、分账明细、打款申请单均保留历史关联。

#### 

---

### 4.2 模块二：发行商主体管理

#### 4.2.1 模块定位

发行商主体由业务负责人统一维护，作为分成规则、银行账户、打款申请等模块的基础主实体。主体档案创建后即可被业务运营配置游戏关系，配置关系后不可删除。

#### 4.2.2 主体档案字段

<lark-table rows="15" cols="4" header-row="true" column-widths="183,183,183,183">

  <lark-tr>
    <lark-td>
      字段
    </lark-td>
    <lark-td>
      类型
    </lark-td>
    <lark-td>
      必填
    </lark-td>
    <lark-td>
      说明
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      主体ID
    </lark-td>
    <lark-td>
      String
    </lark-td>
    <lark-td>
      系统生成
    </lark-td>
    <lark-td>
      全局唯一
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      公司名称
    </lark-td>
    <lark-td>
      String
    </lark-td>
    <lark-td>
      是
    </lark-td>
    <lark-td>
      发行商公司全称，与营业执照一致
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      统一社会信用代码
    </lark-td>
    <lark-td>
      String
    </lark-td>
    <lark-td>
      是
    </lark-td>
    <lark-td>
      全局唯一，不可重复
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      营业执照
    </lark-td>
    <lark-td>
      File
    </lark-td>
    <lark-td>
      是
    </lark-td>
    <lark-td>
      扫描件或加盖公章复印件，支持 JPG/PNG/PDF，≤ 10MB
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      法人姓名
    </lark-td>
    <lark-td>
      String
    </lark-td>
    <lark-td>
      是
    </lark-td>
    <lark-td>
      法定代表人
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      法人证件号
    </lark-td>
    <lark-td>
      String
    </lark-td>
    <lark-td>
      是
    </lark-td>
    <lark-td>
      加密存储，展示时脱敏
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      联系人姓名
    </lark-td>
    <lark-td>
      String
    </lark-td>
    <lark-td>
      是
    </lark-td>
    <lark-td>
      日常对接人
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      联系人手机号
    </lark-td>
    <lark-td>
      String
    </lark-td>
    <lark-td>
      是
    </lark-td>
    <lark-td>
      —
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      联系人邮箱
    </lark-td>
    <lark-td>
      String
    </lark-td>
    <lark-td>
      否
    </lark-td>
    <lark-td>
      —
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      合作状态
    </lark-td>
    <lark-td>
      Enum
    </lark-td>
    <lark-td>
      系统生成
    </lark-td>
    <lark-td>
      合作中（本期只支持该状态）
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      创建人
    </lark-td>
    <lark-td>
      Reference
    </lark-td>
    <lark-td>
      系统生成
    </lark-td>
    <lark-td>
      财务人员
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      创建时间
    </lark-td>
    <lark-td>
      DateTime
    </lark-td>
    <lark-td>
      系统生成
    </lark-td>
    <lark-td>
      —
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      最后修改人
    </lark-td>
    <lark-td>
      Reference
    </lark-td>
    <lark-td>
      系统生成
    </lark-td>
    <lark-td>
      —
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      最后修改时间
    </lark-td>
    <lark-td>
      DateTime
    </lark-td>
    <lark-td>
      系统生成
    </lark-td>
    <lark-td>
      —
    </lark-td>
  </lark-tr>
</lark-table>

#### 4.2.3 银行账户字段

<lark-table rows="13" cols="4" header-row="true" column-widths="183,183,183,183">

  <lark-tr>
    <lark-td>
      字段
    </lark-td>
    <lark-td>
      类型
    </lark-td>
    <lark-td>
      必填
    </lark-td>
    <lark-td>
      说明
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      账户ID
    </lark-td>
    <lark-td>
      String
    </lark-td>
    <lark-td>
      系统生成
    </lark-td>
    <lark-td>
      —
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      所属发行商主体
    </lark-td>
    <lark-td>
      Reference
    </lark-td>
    <lark-td>
      系统生成
    </lark-td>
    <lark-td>
      1:1 关联
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      账户名称
    </lark-td>
    <lark-td>
      String
    </lark-td>
    <lark-td>
      是
    </lark-td>
    <lark-td>
      收款账户名称
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      开户银行
    </lark-td>
    <lark-td>
      String
    </lark-td>
    <lark-td>
      是
    </lark-td>
    <lark-td>
      —
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      开户支行
    </lark-td>
    <lark-td>
      String
    </lark-td>
    <lark-td>
      是
    </lark-td>
    <lark-td>
      —
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      银行账号
    </lark-td>
    <lark-td>
      String
    </lark-td>
    <lark-td>
      是
    </lark-td>
    <lark-td>
      加密存储，展示时脱敏
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      联行号
    </lark-td>
    <lark-td>
      String
    </lark-td>
    <lark-td>
      是
    </lark-td>
    <lark-td>
      跨行转账使用
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      账户用途
    </lark-td>
    <lark-td>
      String
    </lark-td>
    <lark-td>
      否
    </lark-td>
    <lark-td>
      备注说明
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      创建人
    </lark-td>
    <lark-td>
      Reference
    </lark-td>
    <lark-td>
      系统生成
    </lark-td>
    <lark-td>
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      创建时间
    </lark-td>
    <lark-td>
      DateTime
    </lark-td>
    <lark-td>
      系统生成
    </lark-td>
    <lark-td>
      —
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      最后修改人
    </lark-td>
    <lark-td>
      Reference
    </lark-td>
    <lark-td>
      系统生成
    </lark-td>
    <lark-td>
      —
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      最后修改时间
    </lark-td>
    <lark-td>
      DateTime
    </lark-td>
    <lark-td>
      系统生成
    </lark-td>
    <lark-td>
      —
    </lark-td>
  </lark-tr>
</lark-table>

#### 4.2.4 功能需求

- **REQ-P-101**：系统应支持业务负责人创建发行商主体档案；统一社会信用代码全局唯一，重复时拒绝提交。
- **REQ-P-102**：系统应支持业务负责人为发行商主体新增/编辑银行账户，每个发行商有且仅有一个银行账户；保存后立即生效，不走审批流程；编辑操作覆盖原账户信息，变更前后完整信息记入操作日志。
- **REQ-P-103**：当发行商主体尚未配置任何游戏关系时，允许业务负责人删除该主体（同步删除关联银行账户）；已配置游戏关系时禁止删除，提示"该发行商已绑定游戏关系，无法删除"。
- **REQ-P-104**：发行商主体与银行账户创建或更新后，系统再发起打款时应自动同步至银企直联系统
- **REQ-P-105**：发起打款申请前，系统应校验收款账户信息完整性（账户名称、开户银行、开户支行、银行账号、联行号均不为空），不完整时拒绝提交并提示完善账户信息。
- **REQ-P-106**：打款申请提交时，系统应快照当时完整账户信息至申请单，后续账户变更不影响历史打款记录的展示与追溯。

---

### 4.3 模块三：CP主体管理

#### 4.3.1 模块定位

cp主体由业务负责人统一维护，作为CP利润分成规则模块的基础主实体。主体档案创建后即可被业务运营配置游戏-平台-微小、抖小、巨量后台ID-发行商-cp关系，配置关系后不可删除。

#### 4.3.2 主体档案字段

<lark-table rows="13" cols="4" header-row="true" column-widths="183,183,183,183">

  <lark-tr>
    <lark-td>
      字段
    </lark-td>
    <lark-td>
      类型
    </lark-td>
    <lark-td>
      必填
    </lark-td>
    <lark-td>
      说明
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      主体ID
    </lark-td>
    <lark-td>
      String
    </lark-td>
    <lark-td>
      系统生成
    </lark-td>
    <lark-td>
      全局唯一
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      公司名称
    </lark-td>
    <lark-td>
      String
    </lark-td>
    <lark-td>
      是
    </lark-td>
    <lark-td>
      发行商公司全称，与营业执照一致
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      法人姓名
    </lark-td>
    <lark-td>
      String
    </lark-td>
    <lark-td>
      是
    </lark-td>
    <lark-td>
      法定代表人
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      法人证件号
    </lark-td>
    <lark-td>
      String
    </lark-td>
    <lark-td>
      是
    </lark-td>
    <lark-td>
      加密存储，展示时脱敏
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      联系人姓名
    </lark-td>
    <lark-td>
      String
    </lark-td>
    <lark-td>
      是
    </lark-td>
    <lark-td>
      日常对接人
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      联系人手机号
    </lark-td>
    <lark-td>
      String
    </lark-td>
    <lark-td>
      是
    </lark-td>
    <lark-td>
      —
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      联系人邮箱
    </lark-td>
    <lark-td>
      String
    </lark-td>
    <lark-td>
      否
    </lark-td>
    <lark-td>
      —
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      合作状态
    </lark-td>
    <lark-td>
      Enum
    </lark-td>
    <lark-td>
      系统生成
    </lark-td>
    <lark-td>
      合作中（本期只支持该状态）
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      创建人
    </lark-td>
    <lark-td>
      Reference
    </lark-td>
    <lark-td>
      系统生成
    </lark-td>
    <lark-td>
      财务人员
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      创建时间
    </lark-td>
    <lark-td>
      DateTime
    </lark-td>
    <lark-td>
      系统生成
    </lark-td>
    <lark-td>
      —
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      最后修改人
    </lark-td>
    <lark-td>
      Reference
    </lark-td>
    <lark-td>
      系统生成
    </lark-td>
    <lark-td>
      —
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      最后修改时间
    </lark-td>
    <lark-td>
      DateTime
    </lark-td>
    <lark-td>
      系统生成
    </lark-td>
    <lark-td>
      —
    </lark-td>
  </lark-tr>
</lark-table>

### 4.4 模块四：分成规则配置

#### 4.4.1 规则模型

每条规则绑定「游戏 + 平台类型+发行商+CP」，审批通过后内容不可修改。如需调整，需创建新规则并设置不重叠的时间区间。

#### 4.4.2 规则字段

<lark-table rows="19" cols="4" header-row="true" column-widths="183,183,183,183">

  <lark-tr>
    <lark-td>
      字段
    </lark-td>
    <lark-td>
      类型
    </lark-td>
    <lark-td>
      必填
    </lark-td>
    <lark-td>
      说明
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      规则ID
    </lark-td>
    <lark-td>
      String
    </lark-td>
    <lark-td>
      系统生成
    </lark-td>
    <lark-td>
      —
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      游戏
    </lark-td>
    <lark-td>
      Reference
    </lark-td>
    <lark-td>
      是
    </lark-td>
    <lark-td>
      —
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      平台类型
    </lark-td>
    <lark-td>
      Enum
    </lark-td>
    <lark-td>
      是
    </lark-td>
    <lark-td>
      微信小游戏 / 抖音小游戏
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      发行商主体
    </lark-td>
    <lark-td>
      Reference
    </lark-td>
    <lark-td>
      是
    </lark-td>
    <lark-td>
      —
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      CP主体
    </lark-td>
    <lark-td>
    </lark-td>
    <lark-td>
      是
    </lark-td>
    <lark-td>
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      生效开始时间
    </lark-td>
    <lark-td>
      Date
    </lark-td>
    <lark-td>
      是
    </lark-td>
    <lark-td>
      财务手动填写
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      生效结束时间
    </lark-td>
    <lark-td>
      Date
    </lark-td>
    <lark-td>
      是
    </lark-td>
    <lark-td>
      财务手动填写
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      结算类型
    </lark-td>
    <lark-td>
      Enum
    </lark-td>
    <lark-td>
      是
    </lark-td>
    <lark-td>
      IAA
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      结算公式
    </lark-td>
    <lark-td>
    </lark-td>
    <lark-td>
      是
    </lark-td>
    <lark-td>
      看下子表
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      发行方分成比例
    </lark-td>
    <lark-td>
      Decimal
    </lark-td>
    <lark-td>
      是
    </lark-td>
    <lark-td>
      发行商分成占总利润百分比，如 60%
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      CP方分成比例
    </lark-td>
    <lark-td>
    </lark-td>
    <lark-td>
      是
    </lark-td>
    <lark-td>
      发行商分成占总利润百分比，如 20%
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      结算周期
    </lark-td>
    <lark-td>
      Enum
    </lark-td>
    <lark-td>
      是
    </lark-td>
    <lark-td>
      按自然月
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      备注
    </lark-td>
    <lark-td>
      String
    </lark-td>
    <lark-td>
      否
    </lark-td>
    <lark-td>
      可选说明
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      规则状态
    </lark-td>
    <lark-td>
      Enum
    </lark-td>
    <lark-td>
      系统生成
    </lark-td>
    <lark-td>
      草稿 / 审批中 / 生效中 / 已到期 / 已驳回
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      BPM 实例号
    </lark-td>
    <lark-td>
      String
    </lark-td>
    <lark-td>
      系统生成
    </lark-td>
    <lark-td>
      BPM 发起审批后回写
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      审批通过时间
    </lark-td>
    <lark-td>
      DateTime
    </lark-td>
    <lark-td>
      系统生成
    </lark-td>
    <lark-td>
      BPM 通过时自动写入
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      创建人
    </lark-td>
    <lark-td>
      Reference
    </lark-td>
    <lark-td>
      系统生成
    </lark-td>
    <lark-td>
      财务人员
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      创建时间
    </lark-td>
    <lark-td>
      DateTime
    </lark-td>
    <lark-td>
      系统生成
    </lark-td>
    <lark-td>
      —
    </lark-td>
  </lark-tr>
</lark-table>

#### 4.4.3 结算公式子表

##### 4.4.3.1 IAA分成公式

从分账规则获取发行商分账占总利润百分比

从分账规则获取CP分账占总利润百分比

###### 总利润计算公式

- 总利润=现金利润+广告金利润
- 现金利润=现金收入-现金成本
- 广告金利润=广告金收入-广告金成本
- 总收入=现金收入+广告金收入
- 现金收入=买量收入+自然量收入
- 现金成本=总消耗成本*（现金收入/总收入）
- 广告金成本=总消耗成本*（广告金收入/总收入）

###### 可结算金额（利润）

发行商可结算金额：现金利润*发行商分成比例+广告金利润*发行商分成比例，其中现金利润部分可提取打款，广告金利润只能查看

<text bgcolor="light-gray">发行商可结算金额为正数时，才可以提款，负数不可提款，如前期有负数，需要将负数冲正以后才可以提款，即有账户余额概念，和每账期可结算金额概念</text>

CP可结算金额：现金利润*CP分成比例

##### 4.4.3.1 IAA分成公式取数逻辑

###### 微信小游戏各指标计算公式

<sheet token="MWi6semo6hXY4ot7gLuc4pt5n1f_pI8ZRO"/>

###### 抖音小游戏各指标计算公式

<sheet token="MWi6semo6hXY4ot7gLuc4pt5n1f_5Gzz1s"/>

#### 4.4.3 功能需求

- **REQ-RU-101**：系统应仅允许财务人员创建分成规则；创建前需确认已存在对应的发行商-游戏-平台-CP关系，否则提示先配置关系。
- **REQ-RU-102**：规则审批通过后，所有字段不可修改，无编辑入口；如需调整，须重新创建新规则。
- **REQ-RU-103**：提交时系统自动校验（同时满足以下三项方允许提交）：
	- 结束时间 > 开始时间
	- 所填时间区间内，该「发行商 + 游戏 + 平台类型+cp」维度无已出账记录
	- 所填时间区间与该维度已审批通过的其他规则无重叠
- **REQ-RU-104**：当财务提交规则时，系统应触发 BPM 审批流程；审批通过后规则状态变更为"生效中"，所有字段锁定。
- **REQ-RU-105**：规则被驳回后标记"已驳回"，不生效；财务可重新创建新规则提交审批。
- **REQ-RU-106**：规则无手动停用操作，以生效结束时间自然到期，状态自动变更为"已到期"。
- **REQ-RU-107**：系统应支持从已有规则复制创建新规则，复制后各字段可编辑。
- **REQ-RU-108**：系统应在规则详情中展示审批状态、审批通过时间、生效开始/结束时间及完整分成公式说明。

---

### 4.5 模块五：分账计算

#### 4.5.1 发行商账期可结算池

<lark-table rows="17" cols="3" header-row="true" column-widths="244,244,244">

  <lark-tr>
    <lark-td>
      字段
    </lark-td>
    <lark-td>
      类型
    </lark-td>
    <lark-td>
      说明
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      记录ID
    </lark-td>
    <lark-td>
      String
    </lark-td>
    <lark-td>
      系统自动生成
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      发行商主体
    </lark-td>
    <lark-td>
      Reference
    </lark-td>
    <lark-td>
      —
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      游戏
    </lark-td>
    <lark-td>
      Reference
    </lark-td>
    <lark-td>
      —
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      平台类型
    </lark-td>
    <lark-td>
      Enum
    </lark-td>
    <lark-td>
      微信小游戏 / 抖音小游戏
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      规则ID
    </lark-td>
    <lark-td>
      String
    </lark-td>
    <lark-td>
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      结算账期
    </lark-td>
    <lark-td>
      Date Range
    </lark-td>
    <lark-td>
      结算开始日期-结算结束日期
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      账户余额
    </lark-td>
    <lark-td>
    </lark-td>
    <lark-td>
      历次结算累计转入，扣除已结算后余额
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      历史累计可结算金额
    </lark-td>
    <lark-td>
    </lark-td>
    <lark-td>
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      历史累计已结算金额
    </lark-td>
    <lark-td>
    </lark-td>
    <lark-td>
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      现金余额
    </lark-td>
    <lark-td>
    </lark-td>
    <lark-td>
      历次结算累计转入，扣除已结算后余额
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      广告金余额
    </lark-td>
    <lark-td>
    </lark-td>
    <lark-td>
      历次结算累计转入
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      当期现金可结算金额（利润）
    </lark-td>
    <lark-td>
      Decimal
    </lark-td>
    <lark-td>
      当期发行商可结算现金利润，打款失败的金额也会重新回流到可结算金额资金池
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      当期现金已结算金额
    </lark-td>
    <lark-td>
    </lark-td>
    <lark-td>
      打款成功金额
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      当期现金打款中金额
    </lark-td>
    <lark-td>
    </lark-td>
    <lark-td>
      申请打款但是，银行打款流水还未返回打款成功或者失败的金额
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      当期广告金可结算金额（利润）
    </lark-td>
    <lark-td>
    </lark-td>
    <lark-td>
      当期发行商可结算广告金利润
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      同步时间
    </lark-td>
    <lark-td>
      DateTime
    </lark-td>
    <lark-td>
      系统从后台拉取数据的最后时间
    </lark-td>
  </lark-tr>
</lark-table>

#### 4.5.2 发行商每账期结算明细

- 总利润=现金利润+广告金利润
- 现金利润=现金收入-现金成本
- 广告金利润=广告金收入-广告金成本
- 总收入=现金收入+广告金收入
- 现金收入=买量收入+自然量收入
- 现金成本=总消耗成本*（现金收入/总收入）
- 广告金成本=总消耗成本*（广告金收入/总收入）
- 发行商现金利润=现金利润*发行商分账占比
- 发行商广告金利润=广告机利润*发行商分账占比

#### 4.5.3 游戏-平台-发行商每日收入池与成本池数据流水（分账前）

<lark-table rows="13" cols="3" header-row="true" column-widths="244,244,244">

  <lark-tr>
    <lark-td>
      字段
    </lark-td>
    <lark-td>
      类型
    </lark-td>
    <lark-td>
      说明
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      发行商主体
    </lark-td>
    <lark-td>
      Reference
    </lark-td>
    <lark-td>
      —
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      游戏
    </lark-td>
    <lark-td>
      Reference
    </lark-td>
    <lark-td>
      —
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      平台类型
    </lark-td>
    <lark-td>
      Enum
    </lark-td>
    <lark-td>
      微信小游戏 / 抖音小游戏
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      规则ID
    </lark-td>
    <lark-td>
      String
    </lark-td>
    <lark-td>
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      结算账期
    </lark-td>
    <lark-td>
      Date Range
    </lark-td>
    <lark-td>
      结算开始日期-结算结束日期
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      每日收入金额
    </lark-td>
    <lark-td>
    </lark-td>
    <lark-td>
      历次结算累计转入，扣除已结算后余额
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      现金金额
    </lark-td>
    <lark-td>
    </lark-td>
    <lark-td>
      历次结算累计转入，扣除已结算后余额
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      广告金金额
    </lark-td>
    <lark-td>
    </lark-td>
    <lark-td>
      历次结算累计转入
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      成本金额
    </lark-td>
    <lark-td>
    </lark-td>
    <lark-td>
      打款成功金额
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      现金成本金额
    </lark-td>
    <lark-td>
    </lark-td>
    <lark-td>
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      广告金成本金额
    </lark-td>
    <lark-td>
    </lark-td>
    <lark-td>
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      同步时间
    </lark-td>
    <lark-td>
      DateTime
    </lark-td>
    <lark-td>
      系统从后台拉取数据的最后时间
    </lark-td>
  </lark-tr>
</lark-table>

#### 4.5.4 CP虚拟账户可结算池

以「CP + 游戏 + 平台类型 + 规则」为维度独立核算：

<lark-table rows="11" cols="3" header-row="true" column-widths="244,244,244">

  <lark-tr>
    <lark-td>
      字段
    </lark-td>
    <lark-td>
      类型
    </lark-td>
    <lark-td>
      说明
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      记录ID
    </lark-td>
    <lark-td>
      String
    </lark-td>
    <lark-td>
      系统自动生成
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      CP主体
    </lark-td>
    <lark-td>
      Reference
    </lark-td>
    <lark-td>
      —
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      游戏
    </lark-td>
    <lark-td>
      Reference
    </lark-td>
    <lark-td>
      —
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      平台类型
    </lark-td>
    <lark-td>
      Enum
    </lark-td>
    <lark-td>
      微信小游戏 / 抖音小游戏
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      规则ID
    </lark-td>
    <lark-td>
      String
    </lark-td>
    <lark-td>
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      结算账期
    </lark-td>
    <lark-td>
      Date Range
    </lark-td>
    <lark-td>
      结算开始日期-结算结束日期
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      历史累计可结算金额
    </lark-td>
    <lark-td>
    </lark-td>
    <lark-td>
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      当期现金可结算金额（利润）
    </lark-td>
    <lark-td>
      Decimal
    </lark-td>
    <lark-td>
      当期CP可结算现金利润
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      当期广告金可结算金额（利润）
    </lark-td>
    <lark-td>
    </lark-td>
    <lark-td>
      当期CP可结算广告金利润
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      同步时间
    </lark-td>
    <lark-td>
      DateTime
    </lark-td>
    <lark-td>
      系统从后台拉取数据的最后时间
    </lark-td>
  </lark-tr>
</lark-table>

#### 4.5.2 CP每账期可结算明细

- 总利润=现金利润+广告金利润
- 现金利润=现金收入-现金成本
- 广告金利润=广告金收入-广告金成本
- 总收入=现金收入+广告金收入
- 现金收入=买量收入+自然量收入
- 现金成本=总消耗成本*（现金收入/总收入）
- 广告金成本=总消耗成本*（广告金收入/总收入）
- CP现金利润=现金利润*发行商分账占比
- CP广告金利润=广告机利润*发行商分账占比

#### 4.5.4 游戏-平台-cp每日收入池与成本池

<lark-table rows="13" cols="3" header-row="true" column-widths="244,244,244">

  <lark-tr>
    <lark-td>
      字段
    </lark-td>
    <lark-td>
      类型
    </lark-td>
    <lark-td>
      说明
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      cp主体
    </lark-td>
    <lark-td>
      Reference
    </lark-td>
    <lark-td>
      —
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      游戏
    </lark-td>
    <lark-td>
      Reference
    </lark-td>
    <lark-td>
      —
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      平台类型
    </lark-td>
    <lark-td>
      Enum
    </lark-td>
    <lark-td>
      微信小游戏 / 抖音小游戏
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      规则ID
    </lark-td>
    <lark-td>
      String
    </lark-td>
    <lark-td>
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      日期
    </lark-td>
    <lark-td>
      Date Range
    </lark-td>
    <lark-td>
      结算开始日期-结算结束日期
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      每日收入金额
    </lark-td>
    <lark-td>
    </lark-td>
    <lark-td>
      历次结算累计转入，扣除已结算后余额
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      现金金额
    </lark-td>
    <lark-td>
    </lark-td>
    <lark-td>
      历次结算累计转入，扣除已结算后余额
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      广告金金额
    </lark-td>
    <lark-td>
    </lark-td>
    <lark-td>
      历次结算累计转入
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      成本金额
    </lark-td>
    <lark-td>
    </lark-td>
    <lark-td>
      打款成功金额
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      现金成本金额
    </lark-td>
    <lark-td>
    </lark-td>
    <lark-td>
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      广告金成本金额
    </lark-td>
    <lark-td>
    </lark-td>
    <lark-td>
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      同步时间
    </lark-td>
    <lark-td>
      DateTime
    </lark-td>
    <lark-td>
      系统从后台拉取数据的最后时间
    </lark-td>
  </lark-tr>
</lark-table>

#### 4.5.4 功能需求

- **REQ-E-101**：系统应每日 T+1 从微小、抖小、巨量同步投放收入与成本数据，按游戏-平台-发行商关系、游戏-平台-cp分别写入对应维度的收入池与成本池；同步失败时在每日收入池与成本池数据流水（分账前）页面进行提醒，不触发当日相关结算。
- **REQ-E-102**：结算账期到期时，系统应自动触发分账计算，无需财务手动操作；系统对同一结算账期保证幂等性，不允许重复结算。
- **REQ-E-103**：当账期内未匹配到生效中规则时，系统应阻断结算并告警通知财务，不写入待结算；所有阻断情形均不自动重试。
- **REQ-E-104**：当账期内无对应发行商-游戏-平台关系时，系统应阻断结算并告警通知业务负责人补充配置。
- **REQ-E-105**：当 `分账金额 < 0`（成本超过收入）时，系统不阻断结算，允许负值写入待结算。

#### 4.5.5 异常处理策略

<lark-table rows="5" cols="3" header-row="true" column-widths="244,244,244">

  <lark-tr>
    <lark-td>
      异常情形
    </lark-td>
    <lark-td>
      处理策略
    </lark-td>
    <lark-td>
      操作对象
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      数据同步失败
    </lark-td>
    <lark-td>
      允许手动同步
    </lark-td>
    <lark-td>
      财务手动同步
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      账期内无匹配发行商关系
    </lark-td>
    <lark-td>
      阻断结算
    </lark-td>
    <lark-td>
      业务负责人手动配置
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      账期内无生效中规则
    </lark-td>
    <lark-td>
      阻断结算
    </lark-td>
    <lark-td>
      财务手动配置规则
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      分账金额 < 0
    </lark-td>
    <lark-td>
      负值写入
    </lark-td>
    <lark-td>
      不操作
    </lark-td>
  </lark-tr>
</lark-table>

### 

#### 4.5.6功能需求

- **REQ-D-101**：系统应支持按「发行商 + 游戏 + 平台类型 + 账期」维度查看分账明细，支持按发行商、游戏、平台类型、时间范围筛选。
- **REQ-D-102**：分账明细数据只读，不可修改。
- **REQ-D-103**：当分账金额为负值时，系统应在明细中以红色标注并展示提示说明。
- **REQ-D-104**：系统应支持导出分账明细数据（Excel 格式）。
- **REQ-D-105**：分账明细应支持按发行商、游戏、平台汇总展示，财务可查看各维度待结算金额汇总。
- **REQ-D-106**：系统应支持按「CP + 游戏 + 平台类型 + 账期」维度查看分账明细，支持按CP、游戏、平台类型、时间范围筛选。

---

### 4.7 模块七：发行商自主发起打款

#### 4.7.1 流程说明
```plaintext
发行商登录系统，选择游戏 + 平台类型
系统自动关联发行商主体及收款银行账户（只读展示），
查看账户可结算余额，填写打款金额与备注
  ↓
系统校验：
  ① 打款金额 ≤ 该维度下的可结算余额
  ② 发行商对应分账规则"生效中"
  ③ 收款账户信息完整（账户名称/银行/支行/账号/联行号均不为空）
  ↓
校验通过 → 把对应金额从可结算现金池中转移到打款中现金池
  ↓
生成打款申请单（快照完整账户信息）（发起人可感知）
  → 发起 BPM 审批 （发起人无感知）
  ↓
bpm审批通过→ 打款申请单审批通过，当前打款状态为打款中
  ↓
  调用银企直联系统发起转账
  ↓
银行回调银企直联 → 银企直联回调大发行系统
  ↓
成功：打款中现金池→已结算现金池，归档电子回单→ 打款申请单当前打款状态为打款成功，发起人自行到银行卡查询是否到账
失败：打款中现金池→可结算现金池，归档电子回单→ 打款申请单当前打款状态为打款失败，发起人自行查询原因，可线下联系财务

```

#### 4.7.2 申请单字段

<lark-table rows="22" cols="3" header-row="true" column-widths="244,244,244">

  <lark-tr>
    <lark-td>
      字段
    </lark-td>
    <lark-td>
      类型
    </lark-td>
    <lark-td>
      说明
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      申请单号
    </lark-td>
    <lark-td>
      String
    </lark-td>
    <lark-td>
      系统自动生成
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      游戏
    </lark-td>
    <lark-td>
      Reference
    </lark-td>
    <lark-td>
      发行商选择
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      平台类型
    </lark-td>
    <lark-td>
      Enum
    </lark-td>
    <lark-td>
      发行商选择（微信小游戏 / 抖音小游戏）
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      发行商主体
    </lark-td>
    <lark-td>
      Reference
    </lark-td>
    <lark-td>
      系统自动关联，只读
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      规则ID
    </lark-td>
    <lark-td>
    </lark-td>
    <lark-td>
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      收款账户名称
    </lark-td>
    <lark-td>
      String
    </lark-td>
    <lark-td>
      快照：提交时从银行账户自动抓取
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      开户银行
    </lark-td>
    <lark-td>
      String
    </lark-td>
    <lark-td>
      快照
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      开户支行
    </lark-td>
    <lark-td>
      String
    </lark-td>
    <lark-td>
      快照
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      银行账号
    </lark-td>
    <lark-td>
      String
    </lark-td>
    <lark-td>
      快照，加密存储
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      联行号
    </lark-td>
    <lark-td>
      String
    </lark-td>
    <lark-td>
      快照
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      打款金额
    </lark-td>
    <lark-td>
      Decimal
    </lark-td>
    <lark-td>
      发行商手动填写，≤ 可打款金额
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      申请备注
    </lark-td>
    <lark-td>
      String
    </lark-td>
    <lark-td>
      业务说明，可选
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      发起人
    </lark-td>
    <lark-td>
      Reference
    </lark-td>
    <lark-td>
      发行商用户（代发时记录实际操作财务及代发原因）
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      发起时间
    </lark-td>
    <lark-td>
      DateTime
    </lark-td>
    <lark-td>
      —
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      审批状态
    </lark-td>
    <lark-td>
      Enum
    </lark-td>
    <lark-td>
      待审批 / 已通过 / 已驳回
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      执行状态
    </lark-td>
    <lark-td>
      Enum
    </lark-td>
    <lark-td>
      待执行 /打款中 / 打款成功/ 打款失败
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      BPM 实例号
    </lark-td>
    <lark-td>
      String
    </lark-td>
    <lark-td>
      BPM 系统返回
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      发起打款流水号
    </lark-td>
    <lark-td>
      String
    </lark-td>
    <lark-td>
      结算系统在发起打款时自动生成
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      银企资金流水号
    </lark-td>
    <lark-td>
      String
    </lark-td>
    <lark-td>
      银企直联执行完成后返回并关联
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      银行交易流水号
    </lark-td>
    <lark-td>
      String
    </lark-td>
    <lark-td>
      银行返回
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      电子回单
    </lark-td>
    <lark-td>
      File
    </lark-td>
    <lark-td>
      成功后自动下载归档
    </lark-td>
  </lark-tr>
</lark-table>

---

### 4.8 模块八：审批流

- 分成规则配置审批是涉及到IDS内部成员，所以流程使用BPM系统
- 打款申请流程分为两个端，一个是发起人端，需要在结算平台内查看发起单，查看审批结果和打款结果；一个端是审批端，只涉及到IDS内部成员，所以审批端流程使用BPM系统
- 系统自动发起 BPM 流程，回写 BPM 实例号、当前审批节点与审批状态

---

## 五、审批通知机制

### 5.1 通知渠道

飞书bpm、平台内通知

### 5.2 通知场景

<lark-table rows="4" cols="3" header-row="true" column-widths="244,244,244">

  <lark-tr>
    <lark-td>
      通知场景
    </lark-td>
    <lark-td>
      接收角色
    </lark-td>
    <lark-td>
      通知内容
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      审批成功失败
    </lark-td>
    <lark-td>
      发起人
    </lark-td>
    <lark-td>
      申请单号、审批状态
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      打款成功
    </lark-td>
    <lark-td>
      发起人
    </lark-td>
    <lark-td>
      申请单号、执行状态
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      打款失败
    </lark-td>
    <lark-td>
      发起人
    </lark-td>
    <lark-td>
      申请单号、执行状态，失败原因，提示手动处理
    </lark-td>
  </lark-tr>
</lark-table>

---

## 六、非功能需求

<lark-table rows="7" cols="2" header-row="true" column-widths="350,350">

  <lark-tr>
    <lark-td>
      类型
    </lark-td>
    <lark-td>
      要求
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      性能
    </lark-td>
    <lark-td>
      核心列表页首屏加载 ≤ 2s；分账明细查询（百万级数据）≤ 3s
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      安全
    </lark-td>
    <lark-td>
      基于 RBAC 模型进行权限控制；银行账号、法人证件号加密存储，展示时脱敏
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      可用性
    </lark-td>
    <lark-td>
      年度可用性 ≥ 99.9%；RPO ≤ 1h，RTO ≤ 4h
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      审计
    </lark-td>
    <lark-td>
      规则变更、审批操作、打款执行等关键动作保留完整操作日志，含操作人、时间、变更前后数据
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      幂等
    </lark-td>
    <lark-td>
      同一结算账期不允许重复结算；金额冻结/释放/变更采用并发控制机制保障一致性
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      主数据同步
    </lark-td>
    <lark-td>
      发行商主体与银行账户从大发行系统单向同步至银企直联系统，银企直联系统不得反向修改
    </lark-td>
  </lark-tr>
</lark-table>

---

## 七、两个系统协同说明

### 7.1 系统职责边界

**大发行结算平台**（本系统）：

- 发行商主体管理、银行账户管理
- 游戏与上架平台信息管理
- 发行商-游戏-平台关系管理
- 分成规则配置与审批
- T+1 分账计算
- 可结算金额、冻结金额、已结算金额管理
- 打款申请单管理与状态流转
- BPM 审批流转
- 手动解冻操作
- 业务审计与数据看板

**银企直联系统**：

- 对接银行银企直联接口
- 接收发行商主体与银行账户同步数据（只读使用）
- 生成资金流水单号、发起银行转账
- 接收银行回调，标准化结果回传至大发行系统
- 提供余额查询、流水查询、转账日志查询能力

### 7.2 打款账户约定

用于IDS向发行商打款的账户，前提是必须要在银企直联系统账户信息中维护，并且在前期约定使用银企直联中某一账户进行打款，如余额不足则会打款失败，由资金组负责将资金转入打款账户

---

## 八、已确认事项

1. **结算维度粒度**：当前设计为「发行商 + 游戏 + 平台类型」分别独立出账。若同一发行商在微信和抖音均有合作，需要分平台独立发起
1. **取数接口规范**：待确认，需要用公司洞洞乐账号权限获取尝试一次
1. **负值结算处理**：分账金额为负时允许写入。下一期自动冲抵

---

## 九、术语表

<lark-table rows="8" cols="2" header-row="true" column-widths="350,350">

  <lark-tr>
    <lark-td>
      术语
    </lark-td>
    <lark-td>
      说明
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      IAA利润分成
    </lark-td>
    <lark-td>
      Phase 1 分成模式：以平台投放成本和全部变现收入为计算基础，数据自动取数
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      IAP利润分账
    </lark-td>
    <lark-td>
      Phase 2 分成模式：在流水基础上叠加成本项、费用项等复杂扣减逻辑
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      平台方应用ID
    </lark-td>
    <lark-td>
      游戏在微信/抖音平台的唯一标识，在结算平台后台维护
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      银企直联
    </lark-td>
    <lark-td>
      银行与企业之间的直连系统，提供转账、余额查询、流水查询等底层资金能力
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      可结算金额
    </lark-td>
    <lark-td>
      已完成分账计算但尚未打款的累计金额
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      打款中金额
    </lark-td>
    <lark-td>
      打款申请提交后至完成前的金额
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      T+1
    </lark-td>
    <lark-td>
      数据产生后第二个工作日同步，如 4月22日数据于4月23日同步入库
    </lark-td>
  </lark-tr>
</lark-table>

## 十、待确认事项

广告金余额怎么转入到其他游戏并进行消耗，目前还未有取数逻辑
