# 仓储分拣移动操作机器人伺服驱动控制系统设计

本仓库为《电气传动课程综合设计》作业归档，设计对象为仓储分拣移动操作机器人的差速驱动轮伺服驱动控制系统。完整报告见 [`docs/course-design-report.pdf`](docs/course-design-report.pdf)。

## 课程信息

| 项目 | 内容 |
| --- | --- |
| 学院 | 自动化 |
| 专业 | 自动化 |
| 学生 | 倪蕴杰 |
| 学号 | 1320231081 |
| 设计题目 | 仓储分拣移动操作机器人伺服驱动控制系统设计 |
| 完成时间 | 2026 年 5 月 17 日 |

## 项目简介

本设计围绕 50 kg 仓储分拣移动操作机器人建立差速驱动轮负载模型，完成电机与减速机构选型、驱动主电路设计、控制策略设计、关键器件选型、成本估算和 Simulink 仿真验证。系统采用 48 V 低压直流母线供电，选用 48 V/150 W 永磁直流电机 PMDC 和 17:1 行星减速器，通过 MOSFET H 桥实现 PWM 调压调速，并采用转速-电流双闭环 PI 控制。

设计重点覆盖低速精确靠站、高速巡航、正反转切换、坡道下行制动、紧急停车和负载扰动等仓储机器人典型工况，验证驱动系统的动态响应、安全制动能力和工程可实施性。

## 命名约定

仓库中的仿真文件和结果图统一使用英文命名：

- `math`：数学仿真模型，基于 PMDC 电机传递函数、等效负载转矩、反电动势反馈、电流环和转速环构建。
- `physical-approx`：物理近似仿真模型，使用 Simulink/Simscape 中的 PWM、H 桥、电机和反馈测量模块近似实际驱动系统。
- `speed`：转速响应图。
- `current`：电流响应图。
- `tuned`：坡道下行制动工况中，控制器参数改进后的仿真结果。

## 系统方案

### 力学与传动设计

| 参数 | 取值 |
| --- | --- |
| 整机质量 | 50 kg |
| 驱动轮半径 | 0.08 m |
| 最大坡度 | 5° |
| 滚动阻力系数 | 0.04 |
| 机械传动效率 | 0.90 |
| 电机类型 | 48 V/150 W PMDC |
| 减速机构 | 17:1 行星减速器 |
| 电机额定转速 | 5457 r/min |
| 电机额定转矩 | 0.26 N·m |
| 峰值转矩建议 | 不低于 0.60 N·m |
| 额定轮端转矩 | 约 3.98 N·m |

### 主电路与控制策略

驱动主电路采用 48 V 直流母线、MOSFET H 桥、PMDC 电机、电流采样、母线电容和能耗制动支路组成。H 桥通过 PWM 占空比调节电机电枢平均电压，实现调速、正反转与制动控制。

控制系统采用转速外环 ASR 与电流内环 ACR 的双闭环 PI 结构。速度环负责转速跟踪和动态响应，电流环负责电磁转矩控制、限流和抗冲击。系统还设计了斜坡启动、限流加速、先减速再反向的安全换向策略，以及过流、过压、欠压、堵转、失速和过温保护。

![转速-电流双闭环控制结构](figures/speed-current-double-loop-control.png)

![MOSFET H 桥结构](figures/mosfet-h-bridge.png)

## Simulink 模型

本设计分别建立了数学仿真模型和物理近似仿真模型，用于对比双闭环 PI 控制器在不同建模精度下的响应表现。

### 数学仿真模型

模型文件：[`models/math-simulation-model.slx`](models/math-simulation-model.slx)

![数学仿真模型 Simulink 框图](simulation-results/math-simulation-model.png)

数学仿真模型用于描述 PMDC 电机电气环节、机械环节和反电动势反馈之间的关系，适合进行控制器参数设计、动态指标验证和稳态误差分析。

### 物理近似仿真模型

模型文件：[`models/physical-approx-simulation-model.slx`](models/physical-approx-simulation-model.slx)

![物理近似仿真模型 Simulink 框图](simulation-results/physical-approx-simulation-model.png)

物理近似仿真模型加入 PWM Generator、H 桥、电机模块和反馈测量环节，更接近实际功率驱动系统，可用于观察 PWM 调压、功率电路和机械负载之间的耦合关系。

## 仿真工况与结果

报告第 6 章围绕 6 个典型工况进行仿真。评价指标主要包括上升时间、转速超调量、电流超调量、稳态误差和能量流向。总体上，除坡道下行制动初始参数存在超调外，其余工况均能基本满足设计要求；坡道下行制动经 ASR/ACR 参数调整后，转速和电流超调得到改善。

### 工况 1：空载快速启动响应

空载工况下，系统直接阶跃至指令转速 33.6 rad/s，用于验证双闭环控制器的基本启动响应能力。报告结果显示，数学仿真模型与物理近似仿真模型的上升时间均满足小于 0.8 s 的要求，转速超调量控制在 10% 以内，电流超调量小于 5%，稳态误差近似为 0。

| 数学仿真模型 | 物理近似仿真模型 |
| --- | --- |
| ![工况 1 数学仿真模型转速响应](simulation-results/case-01-no-load-start-math-speed.svg) | ![工况 1 物理近似仿真模型转速响应](simulation-results/case-01-no-load-start-physical-approx-speed.svg) |
| ![工况 1 数学仿真模型电流响应](simulation-results/case-01-no-load-start-math-current.svg) | ![工况 1 物理近似仿真模型电流响应](simulation-results/case-01-no-load-start-physical-approx-current.svg) |

### 工况 2：额定负载启动性能

额定负载启动工况中，系统阶跃至 33.6 rad/s，并施加额定负载转矩 0.09 N·m。该工况用于验证电机在负载作用下的起动能力和电流环限流效果。仿真结果表明，两种模型均能在较短时间内达到目标转速，转速超调、电流超调和稳态误差均满足报告设定要求。

| 数学仿真模型 | 物理近似仿真模型 |
| --- | --- |
| ![工况 2 数学仿真模型转速响应](simulation-results/case-02-rated-load-start-math-speed.svg) | ![工况 2 物理近似仿真模型转速响应](simulation-results/case-02-rated-load-start-physical-approx-speed.svg) |
| ![工况 2 数学仿真模型电流响应](simulation-results/case-02-rated-load-start-math-current.svg) | ![工况 2 物理近似仿真模型电流响应](simulation-results/case-02-rated-load-start-physical-approx-current.svg) |

### 工况 3：低速至高速动态切换

该工况模拟机器人由低速靠站状态切换到通道高速巡航状态，转速由 7.33 rad/s 加速至 33.6 rad/s。该过程对应仓储机器人从精确定位区域进入巡航通道的典型工作场景。仿真结果显示，系统能够平稳完成速度切换，上升时间、转速超调量、电流超调量和稳态误差均基本满足要求。

| 数学仿真模型 | 物理近似仿真模型 |
| --- | --- |
| ![工况 3 数学仿真模型转速响应](simulation-results/case-03-low-to-high-speed-math-speed.svg) | ![工况 3 物理近似仿真模型转速响应](simulation-results/case-03-low-to-high-speed-physical-approx-speed.svg) |
| ![工况 3 数学仿真模型电流响应](simulation-results/case-03-low-to-high-speed-math-current.svg) | ![工况 3 物理近似仿真模型电流响应](simulation-results/case-03-low-to-high-speed-physical-approx-current.svg) |

### 工况 4：正反转快速换向过程

正反转切换工况中，系统由 33.6 rad/s 切换至 -33.6 rad/s，并施加 0.09 N·m 额定负载转矩。该工况用于验证差速驱动轮原地转向、换向和反向加速时的动态性能。报告采用“先减速、再过零、后反向加速”的安全换向思想，仿真结果表明系统可完成正反转切换，稳态误差近似为 0，电流冲击受到电流环限制。

| 数学仿真模型 | 物理近似仿真模型 |
| --- | --- |
| ![工况 4 数学仿真模型转速响应](simulation-results/case-04-forward-reverse-math-speed.svg) | ![工况 4 物理近似仿真模型转速响应](simulation-results/case-04-forward-reverse-physical-approx-speed.svg) |
| ![工况 4 数学仿真模型电流响应](simulation-results/case-04-forward-reverse-math-current.svg) | ![工况 4 物理近似仿真模型电流响应](simulation-results/case-04-forward-reverse-physical-approx-current.svg) |

### 工况 5：坡道下行制动特性

坡道下行制动工况模拟机器人在 5° 下坡重载条件下，由巡航速度 33.6 rad/s 紧急制动至 0。报告中对应负载转矩为 -0.28 N·m。该工况中，驱动轮在重力和惯性作用下反拖电机，电机进入发电制动状态，机械能和重力势能经 H 桥与制动支路转化为制动电阻热能。

初始控制器参数下，坡道下行制动的转速超调量和电流超调量不完全满足要求。报告随后调整 ASR/ACR 参数，改进后转速超调和电流超调满足设计要求。

**初始参数仿真结果**

| 数学仿真模型 | 物理近似仿真模型 |
| --- | --- |
| ![工况 5 初始参数数学仿真模型转速响应](simulation-results/case-05-downhill-braking-math-speed.svg) | ![工况 5 初始参数物理近似仿真模型转速响应](simulation-results/case-05-downhill-braking-physical-approx-speed.svg) |
| ![工况 5 初始参数数学仿真模型电流响应](simulation-results/case-05-downhill-braking-math-current.svg) | ![工况 5 初始参数物理近似仿真模型电流响应](simulation-results/case-05-downhill-braking-physical-approx-current.svg) |

**参数改进后仿真结果**

| 数学仿真模型 | 物理近似仿真模型 |
| --- | --- |
| ![工况 5 改进参数数学仿真模型转速响应](simulation-results/case-05-downhill-braking-math-speed-tuned.svg) | ![工况 5 改进参数物理近似仿真模型转速响应](simulation-results/case-05-downhill-braking-physical-approx-speed-tuned.svg) |
| ![工况 5 改进参数数学仿真模型电流响应](simulation-results/case-05-downhill-braking-math-current-tuned.svg) | ![工况 5 改进参数物理近似仿真模型电流响应](simulation-results/case-05-downhill-braking-physical-approx-current-tuned.svg) |

### 工况 6：20% 负载突变抗扰性能

该工况模拟机器人以 33.6 rad/s 巡航时负载转矩突然增加 20%，即由 0.09 N·m 增加至 0.108 N·m。该工况用于验证系统在外部负载扰动下的抗扰性能。仿真结果表明，负载突变后电流环能够快速补偿负载转矩变化，速度环将转速拉回给定值，系统稳态误差仍接近 0。

| 数学仿真模型 | 物理近似仿真模型 |
| --- | --- |
| ![工况 6 数学仿真模型转速响应](simulation-results/case-06-load-disturbance-math-speed.svg) | ![工况 6 物理近似仿真模型转速响应](simulation-results/case-06-load-disturbance-physical-approx-speed.svg) |
| ![工况 6 数学仿真模型电流响应](simulation-results/case-06-load-disturbance-math-current.svg) | ![工况 6 物理近似仿真模型电流响应](simulation-results/case-06-load-disturbance-physical-approx-current.svg) |

## 关键器件与成本

| 器件 | 推荐规格 | 数量 | 作用 |
| --- | --- | --- | --- |
| 驱动电机 | 48 V 150 W PMDC | 2 台 | 提供左右驱动轮动力 |
| 减速器 | 17:1 行星减速器 | 2 台 | 降速增矩，匹配最高轮速 |
| H 桥驱动模块 | BTS7960 或同等级 48 V MOSFET H 桥 | 2 块 | PWM 调速、正反转、制动 |
| 电流传感器 | ACS712-20A | 2 个 | 电流内环反馈与过流保护 |
| 速度反馈传感器 | MT6701 磁编码器 | 2 个 | 速度闭环反馈 |
| 制动电阻 | 5 Ω / 50 W 铝壳电阻 | 4 只 | 坡道下行和紧急停车能耗制动 |
| 母线电容 | 4700 μF / 63 V | 2 个 | 稳定 48 V 直流母线 |

报告估算系统硬件 BOM 总成本约 2770 元，主要成本集中在电机、减速器和速度反馈传感器。整体方案采用成熟工业模块，便于原型搭建、调试和后期维护。

## 文件结构

```text
.
├── .gitignore
├── README.md
├── docs/
│   ├── course-design-report.pdf
│   └── course-design-report-source.docx
├── models/
│   ├── math-simulation-model.slx
│   └── physical-approx-simulation-model.slx
├── figures/
│   ├── drive-main-circuit-source.agx
│   ├── drive-main-circuit.svg
│   ├── speed-current-double-loop-control.png
│   ├── mosfet-h-bridge.png
│   ├── pmdc-motor.jpg
│   └── planetary-gearbox.jpg
└── simulation-results/
    ├── simulation-cases.txt
    ├── math-simulation-model.png
    ├── physical-approx-simulation-model.png
    ├── case-01-no-load-start-math-speed.svg
    ├── case-01-no-load-start-math-current.svg
    ├── case-01-no-load-start-physical-approx-speed.svg
    ├── case-01-no-load-start-physical-approx-current.svg
    ├── case-02-rated-load-start-math-speed.svg
    ├── case-02-rated-load-start-math-current.svg
    ├── case-02-rated-load-start-physical-approx-speed.svg
    ├── case-02-rated-load-start-physical-approx-current.svg
    ├── case-03-low-to-high-speed-math-speed.svg
    ├── case-03-low-to-high-speed-math-current.svg
    ├── case-03-low-to-high-speed-physical-approx-speed.svg
    ├── case-03-low-to-high-speed-physical-approx-current.svg
    ├── case-04-forward-reverse-math-speed.svg
    ├── case-04-forward-reverse-math-current.svg
    ├── case-04-forward-reverse-physical-approx-speed.svg
    ├── case-04-forward-reverse-physical-approx-current.svg
    ├── case-05-downhill-braking-math-speed.svg
    ├── case-05-downhill-braking-math-current.svg
    ├── case-05-downhill-braking-math-speed-tuned.svg
    ├── case-05-downhill-braking-math-current-tuned.svg
    ├── case-05-downhill-braking-physical-approx-speed.svg
    ├── case-05-downhill-braking-physical-approx-current.svg
    ├── case-05-downhill-braking-physical-approx-speed-tuned.svg
    ├── case-05-downhill-braking-physical-approx-current-tuned.svg
    ├── case-06-load-disturbance-math-speed.svg
    ├── case-06-load-disturbance-math-current.svg
    ├── case-06-load-disturbance-physical-approx-speed.svg
    └── case-06-load-disturbance-physical-approx-current.svg
```

## 使用说明

- 阅读完整设计内容：打开 [`docs/course-design-report.pdf`](docs/course-design-report.pdf)。
- 修改报告源文件：打开 [`docs/course-design-report-source.docx`](docs/course-design-report-source.docx)。
- 复现数学仿真：使用 MATLAB/Simulink 打开 [`models/math-simulation-model.slx`](models/math-simulation-model.slx)。
- 复现物理近似仿真：使用 MATLAB/Simulink 打开 [`models/physical-approx-simulation-model.slx`](models/physical-approx-simulation-model.slx)。
- 查看仿真结果：进入 `simulation-results/`，按 [`simulation-results/simulation-cases.txt`](simulation-results/simulation-cases.txt) 中的编号对应查看各工况的转速和电流响应图。

## 说明

本仓库用于课程设计作业归档、展示和复现实验结果。报告中的模型参数、控制器参数、器件选型和成本估算均以课程设计分析为依据。
