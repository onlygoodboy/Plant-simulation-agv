# 基于 Plant Simulation 的智能工厂多 AGV 路径规划与交通管制仿真

我分享这个项目的原因是因为我在做Plant Simulation大作业的时候，发现网上关于这部分的教程太少了，我也是一边摸索，一边借助AI工具用了很久时间才慢慢摸索出来的。所以我想着把这个我做了差不多的项目分享出来，如果有人需要的话可以直接使用。下面就是我用AI生成的项目描述，如果你需要写报告的话可以参考一下。

本项目基于 **Tecnomatix Plant Simulation 2302** 搭建了一个智能工厂多 AGV 物料搬运仿真模型。模型围绕“多 AGV 任务调度、路径运行控制、交通冲突管制、低电量自动充电和实验结果统计”展开，模拟 AGV 在仓库、多个工位、充电站和双向单车道路网中的连续搬运过程。

需要说明的是，当前模型采用的是 **规则式分段路径控制 + 方向锁/节点锁交通管制** 方法，并未完整实现 A\*、Dijkstra 或 D\* Lite 等标准图搜索算法。模型重点放在 Plant Simulation 中真实 AGV 运行、任务调度、交通管制和充电策略的搭建与验证上。

---

## 1. 作业任务要求

本项目对应课程任务为“智能工厂 AGV 路径规划与交通管制”。作业要求主要包括车间布局建模、任务生成、路径网络与交通管制设计、仿真运行、优化和报告撰写等内容。

![作业任务要求](https://github.com/onlygoodboy/Plant-simulation-agv-/blob/main/%E4%BD%9C%E4%B8%9A%E6%88%AA%E5%9B%BE.jpg)

---

## 2. 模型截图

下图为最终整理后的 Plant Simulation 模型界面。模型界面按照功能分为核心数据表区、扩展表区、模型主体区、主流程控制方法区、路径与锁控制方法区、辅助函数区、实验辅助区和结果可视化区。

<img width="1458" height="672" alt="image" src="https://github.com/user-attachments/assets/931336e9-5b73-4764-8799-a7a3852034f6" />

---

## 3. 项目背景

在智能工厂和柔性物流系统中，AGV 常用于仓库与生产工位之间的物料搬运。当多台 AGV 在同一路网中运行时，容易出现以下问题：

- 多台 AGV 同时抢占同一路段；
- 双向单车道上出现对向会车；
- 多台 AGV 在交叉节点处重合；
- 急单任务响应不及时；
- 简单先来先服务策略导致等待时间较长；
- AGV 电量不足影响连续运行。

因此，本项目在 Plant Simulation 中构建多 AGV 搬运模型，并设计任务调度、分段路径运行、交通锁控制和自动充电策略，用于分析不同调度策略对系统运行效率的影响。

---

## 4. 软件环境

| 项目 | 说明 |
|---|---|
| 仿真软件 | Tecnomatix Plant Simulation 2302 |
| 编程语言 | SimTalk |
| 操作系统 | Windows 11 |
| 仿真类型 | 离散事件仿真 |
| 应用场景 | 智能工厂多 AGV 物流搬运 |

---

## 5. 模型组成

### 5.1 物理对象

模型中主要包括以下对象：

- `AGV池`：生成并管理 5 台 AGV；
- `MaterialSource`：生成仓库物料；
- `B_W`：仓库/主供料缓冲区；
- `B_S1 ~ B_S10`：各工位缓冲区；
- `M_CS`：AGV 充电站；
- `M_W、M_E、M_F、M_G、M_H、M_S1 ~ M_S10`：Marker 路网节点；
- `物料终结`：用于接收完成搬运后的物料。

### 5.2 数据表

| 表名 | 作用 |
|---|---|
| `ConfigTable` | 存放模型参数，如 AGV 数量、速度、装卸时间、低电量阈值、调度策略编号等 |
| `TaskTable` | 记录搬运任务，包括任务编号、起点、终点、状态、分配 AGV、起终缓冲区等 |
| `AGVTable` | 记录 AGV 状态，包括当前任务、当前位置、电量、总行驶距离、充电次数等 |
| `LockTable` | 记录路段锁和节点锁，用于减少对向会车和节点重合 |
| `ResultTable` | 记录不同策略下的仿真结果 |
| `ChartTable` | 整理图表展示所需数据，用于结果可视化 |

### 5.3 方法模块

| 方法名 | 作用 |
|---|---|
| `ResetRealState` | 重置仿真状态，包括任务、AGV、电量、锁表和统计数据 |
| `StartRealSimulation` | 启动真实 AGV 搬运仿真 |
| `GenerateDynamicTask` | 动态生成新的搬运任务 |
| `DispatchRealTasks` | 根据调度策略为空闲 AGV 分配任务 |
| `ExecuteRealTask` | 控制 AGV 完成取货、装货、送货、卸货和状态更新 |
| `DriveByPath` | 控制 AGV 按节点进行分段路径运行 |
| `DriveOneSegment` | 控制 AGV 运行单个路段，并检查方向锁和节点锁 |
| `ChargeAGV` | 控制低电量 AGV 前往充电站并充电 |
| `UpdateResultTable` | 统计仿真结果并写入结果表 |
| `UpdateChartTable` | 将结果表中的数据整理为图表展示数据 |

---

## 6. 模型运行逻辑

模型整体运行流程如下：

```text
ResetRealState
        ↓
StartRealSimulation
        ↓
GenerateDynamicTask
        ↓
DispatchRealTasks
        ↓
ExecuteRealTask
        ↓
DriveByPath / DriveOneSegment
        ↓
判断是否需要充电
        ↓
ChargeAGV
        ↓
UpdateResultTable / UpdateChartTable
```

具体过程为：首先通过 `ResetRealState` 清空历史状态并初始化 AGV、电量、任务和锁表；随后 `StartRealSimulation` 启动仿真，定时调用任务生成和任务分配方法；`GenerateDynamicTask` 按照设定规则生成搬运任务并写入 `TaskTable`；`DispatchRealTasks` 根据当前调度策略扫描等待任务和空闲 AGV，并完成任务分配。

任务被分配后，`ExecuteRealTask` 控制 AGV 前往取货点、装货、送货、卸货并更新任务状态。AGV 在运行过程中不是直接一次性前往终点，而是通过 `DriveByPath` 和 `DriveOneSegment` 按节点逐段运行。每进入一个路段或节点前，系统会检查方向锁和节点锁，避免对向会车和节点重合。任务完成后，如果 AGV 电量低于阈值，则调用 `ChargeAGV` 前往充电站充电。仿真结束后，`UpdateResultTable` 和 `UpdateChartTable` 统计并展示结果。

---

## 7. 调度策略设计

本项目中的 `StrategyMode` 表示任务调度策略，而不是路径规划算法编号。路径运行部分统一采用规则式分段路径控制方法；在此基础上，通过改变 `StrategyMode` 对比不同任务分配策略对系统运行效率的影响。

| StrategyMode | 调度策略 | 说明 |
|---:|---|---|
| 1 | 先来先服务策略 | 按任务生成顺序分配任务 |
| 2 | 紧急任务优先策略 | 优先分配 `Urgent` 急单任务 |
| 3 | 综合调度策略 | 综合考虑任务紧急程度、AGV 到取货点距离和任务等待时间 |

其中，策略 1 作为基准策略；策略 2 强调急单响应；策略 3 在急单优先的基础上进一步考虑 AGV 位置和任务等待时间，用于减少空驶和长时间等待。

---

## 8. 路径控制与交通管制

### 8.1 分段路径控制

当前模型没有直接实现 A\*、Dijkstra 或 D\* Lite 算法，而是采用适合固定 Marker 路网的规则式分段路径控制方法。

例如，AGV 从仓库 `W` 前往工位 `S3` 时，路径可拆分为：

```text
W → E → F → G → K → S3
```

AGV 从工位 `S8` 返回仓库 `W` 时，路径可拆分为：

```text
S8 → B → F → E → W
```

该方法通过 `DriveByPath`、`DriveToMain`、`MoveMain`、`DriveFromMainToNode` 和 `DriveOneSegment` 等方法控制 AGV 逐段运行，使路径过程更清晰，也便于加入交通管制。

### 8.2 方向锁

方向锁用于减少双向单车道上的对向会车。AGV 进入某一路段前，会检查该路段反方向是否已被占用。如果反方向被占用，AGV 等待；如果未被占用，则 AGV 占用当前方向路段并继续运行，到达下一节点后释放锁。

### 8.3 节点锁

节点锁用于减少多个 AGV 同时进入同一个 Marker 节点造成的重合。AGV 进入目标节点前，会先判断目标节点是否空闲。如果节点已被占用，则等待；如果节点空闲，则锁定目标节点并继续前进。

通过方向锁和节点锁结合，模型能够减少对向会车和节点重合，提高多 AGV 连续运行的稳定性。

---

## 9. 充电策略

每台 AGV 在 `AGVTable` 中记录当前电量。AGV 初始电量为 100%，执行任务后根据运行距离扣减电量。

当 AGV 电量低于设定阈值时，系统不再为其分配新的搬运任务，而是调用 `ChargeAGV` 方法，使其前往充电站 `M_CS`。到达充电站后，AGV 进入充电状态，等待设定时间后电量恢复至 100%，随后重新进入空闲状态并参与后续任务调度。

```text
电量低于阈值
        ↓
AGV 前往充电站 M_CS
        ↓
开始充电
        ↓
电量恢复至 100%
        ↓
AGV 重新进入 Idle 状态
        ↓
继续参与任务分配
```

---

## 10. 实验结果

仿真时间设置为 7200 秒。三种调度策略的实验结果如下：

| 实验编号 | 策略 | 完成任务数 | 平均完成时间/s | 平均等待时间/s | 未完成任务数 | 总行驶距离 | AGV利用率/% | 平均电量/% | 充电次数 |
|---|---|---:|---:|---:|---:|---:|---:|---:|---:|
| E12 | 策略1：先来先服务 | 30 | 402.15 | 245.45 | 0 | 16320 | 45.33 | 59.04 | 11 |
| E13 | 策略2：紧急任务优先 | 30 | 391.54 | 235.28 | 0 | 16880 | 46.89 | 78.24 | 12 |
| E14 | 策略3：紧急优先 + 最近 AGV + 等待补偿 | 30 | 371.12 | 220.33 | 0 | 14800 | 41.11 | 80.80 | 12 |

结果表明，三种策略均在 7200 秒内完成了 30 个任务，未完成任务数均为 0，说明模型整体运行稳定。

其中，策略 3 的综合效果最好。相比策略 1，策略 3 的平均完成时间由 402.15 s 降低到 371.12 s，平均等待时间由 245.45 s 降低到 220.33 s，总行驶距离由 16320 降低到 14800。说明综合考虑任务紧急程度、AGV 当前位置和任务等待时间后，系统能够更合理地分配 AGV，减少无效行驶和等待时间。

---

## 11. 项目特点

- 基于真实 Plant Simulation 模型搭建，而不是单纯程序计算；
- 使用 AGV 池生成和管理多台 AGV；
- 设置了动态任务生成机制；
- 实现了多 AGV 连续搬运；
- 采用规则式分段路径控制方法；
- 通过方向锁减少对向会车；
- 通过节点锁减少 Marker 节点重合；
- 引入低电量自动充电策略；
- 使用图表对象展示实验结果；
- 对三种调度策略进行了对比分析。

---

## 12. 说明

本项目主要用于课程学习和仿真实验展示，模型文件需要使用 Tecnomatix Plant Simulation 打开模型文件.spp。当前模型重点展示多 AGV 任务调度、分段路径控制、交通冲突处理、自动充电和结果可视化逻辑，尚未达到工业级调度系统的完整程度。


## 13. 运行流程

如果只是想查看 AGV 仿真动画效果，可以直接点击 Plant Simulation 的开始仿真按钮运行模型。
如果需要复现实验结果，并生成三种调度策略的对比数据与图表，可以按照以下步骤操作。

### 13.1 清空上一次实验结果

首先右键点击 `ClearFinalResults` 方法，并选择运行，用于清除上一次正式实验留下的数据，避免新旧实验结果混在一起。

<p align="center">
  <img width="288" height="204" alt="运行 ClearFinalResults" src="https://github.com/user-attachments/assets/4a99c771-c92a-4f19-a456-00366b392c16" />
</p>

---

### 13.2 设置第一组调度策略

打开 `ConfigTable`，将 `StrategyMode` 设置为 `1`。
其中，三种策略含义如下：

| StrategyMode | 调度策略     | 含义                           |
| -----------: | -------- | ---------------------------- |
|            1 | 先来先服务策略  | 按任务生成顺序依次分配任务                |
|            2 | 紧急任务优先策略 | 优先处理 `Urgent` 急单任务           |
|            3 | 综合调度策略   | 综合考虑任务紧急程度、AGV 到取货点距离和任务等待时间 |

<p align="center">
  <img width="260" height="126" alt="设置 StrategyMode" src="https://github.com/user-attachments/assets/bb5efca5-ae53-45c0-89df-cb5c4a672e89" />
</p>

<p align="center">
  <img width="612" height="345" alt="ConfigTable 设置 StrategyMode" src="https://github.com/user-attachments/assets/be9a1883-6eb2-4ff9-9b67-2a17afa9077b" />
</p>

---

### 13.3 设置仿真时间并运行模型

打开事件控制器，设置仿真运行时间。本项目实验中统一设置为 **2 小时**，即 `7200 s`。
设置完成后，点击开始仿真，并根据需要使用快进功能加速仿真过程。

<p align="center">
  <img width="276" height="153" alt="打开事件控制器" src="https://github.com/user-attachments/assets/26c1f706-e04e-45d0-9eb6-6bf53ba064e9" />
</p>

<p align="center">
  <img width="276" height="282" alt="设置仿真时间" src="https://github.com/user-attachments/assets/07510b1c-32d5-4e9c-9606-9f29506b7433" />
</p>

<p align="center">
  <img width="275" height="296" alt="开始并快进仿真" src="https://github.com/user-attachments/assets/d0c85bae-1dbd-4749-aa26-368ae1bd0db0" />
</p>

---

### 13.4 记录本次仿真结果

仿真运行到设定时间后，右键点击 `UpdateResultTable` 方法并运行，将当前策略下的实验结果写入 `ResultTable`。

<p align="center">
  <img width="237" height="192" alt="运行 UpdateResultTable" src="https://github.com/user-attachments/assets/6582d5fb-3b5c-498b-a0c9-17e7017d593b" />
</p>

运行完成后，可以打开 `ResultTable` 查看本次实验数据。

<p align="center">
  <img width="279" height="135" alt="打开 ResultTable" src="https://github.com/user-attachments/assets/bbfbd67d-2a42-416b-a6c5-aa22b2bac2d8" />
</p>

<p align="center">
  <img width="1655" height="708" alt="ResultTable 实验结果" src="https://github.com/user-attachments/assets/fabc4ded-b118-4544-8ebb-9b89b082135e" />
</p>

---

### 13.5 重复运行策略 2 和策略 3

完成 `StrategyMode = 1` 的实验后，分别将 `StrategyMode` 设置为 `2` 和 `3`，并重复以下步骤：

```text
设置 StrategyMode
↓
ResetRealState
↓
StartRealSimulation
↓
运行到 7200 s
↓
UpdateResultTable
```

最终，`ResultTable` 中会分别记录三种调度策略的实验结果。

---

### 13.6 更新图表数据

三组实验全部完成后，右键点击 `UpdateChartTable` 方法并运行。
该方法会从 `ResultTable` 中提取最新的三组策略结果，并整理到 `ChartTable` 中，供图表对象读取。

<p align="center">
  <img width="219" height="189" alt="运行 UpdateChartTable" src="https://github.com/user-attachments/assets/22350dc5-f293-4a4c-bd53-b9d850ba13dc" />
</p>

<p align="center">
  <img width="225" height="108" alt="打开 ChartTable" src="https://github.com/user-attachments/assets/4ca62492-f031-45ba-911f-8b7f90c2934b" />
</p>

<p align="center">
  <img width="1626" height="261" alt="ChartTable 数据" src="https://github.com/user-attachments/assets/edc16d6d-d867-4a82-81f0-bec7017a9e38" />
</p>

---

### 13.7 查看对比图表

最后，分别点击结果可视化区中的三个图表对象：

```text
Chart_Time
Chart_Distance
Chart_Battery
```

打开图表对象后，点击“显示图表”，即可查看三种调度策略的对比结果。

<p align="center">
  <img width="207" height="132" alt="结果可视化区图表对象" src="https://github.com/user-attachments/assets/a10e1344-e6af-4a8e-8201-e9b9aa61360e" />
</p>

<p align="center">
  <img width="930" height="503" alt="显示图表" src="https://github.com/user-attachments/assets/3af4a5d3-87ba-4c90-bbcb-8ad8555c7036" />
</p>

---

## 14. 图表含义

为了更直观地展示三种调度策略的实验结果，模型中设置了三个图表对象，分别用于展示时间指标、总行驶距离和平均剩余电量。相比只查看 `ResultTable`，图表能够更直接地反映不同策略之间的性能差异。

### 14.1 Chart_Time：时间指标对比图

`Chart_Time` 用于展示三种调度策略下的时间指标对比，主要包括 **平均完成时间** 和 **平均等待时间**。通过该图可以直观看出，不同任务分配策略会影响 AGV 系统的任务响应效率。其中，策略 3 的平均完成时间和平均等待时间均低于策略 1 和策略 2，说明综合考虑任务优先级、AGV 当前位置和任务等待时间后，系统能够更合理地分配车辆，减少任务排队等待，提高整体搬运效率。

<p align="center">
  <img width="420" height="380" alt="Chart_Time 时间指标对比图" src="https://github.com/user-attachments/assets/893c35c3-0582-490e-af2d-92317c3497a2" />
</p>

---

### 14.2 Chart_Distance：总行驶距离对比图

`Chart_Distance` 用于展示三种调度策略下 AGV 的 **总行驶距离** 对比。该图主要反映不同调度策略是否会造成 AGV 过多空驶或绕行。从结果来看，策略 3 的总行驶距离最短，说明“紧急优先 + 最近 AGV + 等待补偿”的综合调度策略能够减少不必要的跨区域调度，使 AGV 尽量选择距离任务起点较近的车辆执行任务，从而降低系统总运行距离。

<p align="center">
  <img width="420" height="378" alt="Chart_Distance 总行驶距离对比图" src="https://github.com/user-attachments/assets/849d75e2-9c7d-40a3-8305-5e8a9bec41ae" />
</p>

---

### 14.3 Chart_Battery：平均剩余电量对比图

`Chart_Battery` 用于展示三种调度策略下 AGV 的 **平均剩余电量** 情况。该图可以辅助分析不同调度策略对车辆电量状态的影响。结果显示，策略 3 的平均剩余电量最高，说明在当前任务分配和充电策略共同作用下，AGV 能够保持较好的电量水平。虽然平均电量还会受到充电次数和充电时机影响，但该结果可以说明策略 3 在减少无效行驶和维持车辆持续运行方面表现更稳定。

<p align="center">
  <img width="420" height="378" alt="Chart_Battery 平均剩余电量对比图" src="https://github.com/user-attachments/assets/f535342c-e38a-4d45-a036-55ccd6bf8a0e" />
</p>

---

### 14.4 图表指标汇总

| 图表对象             | 展示指标          | 主要作用                |
| ---------------- | ------------- | ------------------- |
| `Chart_Time`     | 平均完成时间、平均等待时间 | 分析任务响应效率和完成效率       |
| `Chart_Distance` | 总行驶距离         | 分析 AGV 空驶、绕行和整体运行距离 |
| `Chart_Battery`  | 平均剩余电量        | 分析车辆电量状态和充电策略效果     |
