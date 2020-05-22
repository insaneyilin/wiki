---
title: "Autonomous Driving Levels"
date: 2019-07-09 00:52
---

[TOC]

## SAE 自动驾驶分级

SAE，美国汽车工程师协会

### L0 无自动化

- 顾名思义，由人类完全操作

### L1 提醒驾驶

- 加速、转向、制动等环节由人类自己操作
- 系统只提供提示、预警类信息

### L2 干预驾驶

- 部分自动化
- 基本驾驶操作仍由人类完成，但当系统感知到危险时会进行短暂的驾驶干预来保障驾驶安全
- e.g. 自动变道，车道偏离预警

### L3 人机共驾

- 有条件自动化
- 部分驾驶操作可由自动驾驶系统完成，如系统可在部分场景下控制加速、制动及转向操作，非完全自动驾驶

### L4 辅助自动驾驶

- 高度自动化
- 由无人驾驶系统完成全部驾驶操作
- 人类需要根据系统请求接管车辆
- 非完全自动驾驶，需要限定驾驶环境和条件

### L5 完全自动驾驶

- 完全自动化
- 由无人驾驶系统完成全部驾驶操作
- 人类可以根据需要接管
- 在所有道路和环境条件下均可进行自动驾驶

---

## 自动驾驶任务分类

一般来说，Driving Task 可以分为三个部分：

- Perceiving the enviroment
- Planning how to reach from A to B
- Controlling the vehicle

ODD, Operational Design Domain. 限定操作域。

具体驾驶任务由哪些组成？

- lateral control — steering | 横向控制，如转向
- longitudinal control — braking | 纵向控制，如刹车
- Object and Event Detection and Response (OEDR) | 物体、事件的检测与响应
- planning
  - long term (route)
  - short term
- miscellaneous

自动驾驶 5 级分类

- L0 - 无自动化
- L1 - 驾驶辅助，如 ACC, LKA
- L2 - 部分自动化，如部分情况下可以执行 Lateral Control / Longitudinal Control
- L3 - 有条件自动化，部分场景下具备 OEDR 能力
- L4 - 高度自动驾驶，具备应变（Fallback）能力
- L5 - 全自动驾驶，unlimited ODD

## 感知

```
input -> [Analyze ego motion & env (perceptin)] -> [Decide on and plan a maneuver] -> drive
```

Object and Event Detection and Response (OEDR)


Goals for perception

- static objects. e.g. road and lane markings, curbs, traffic lights
- dynamic objects (on-road). e.g. vehicles, pedestrians
- ego localization. e.g. position, velocity, orientation



Challenge to perception

- robust detection and segmentation
- sensor uncertainty
- occlusion, reflection
- illumination, lens flare
- weather, precipitation

## 规划与决策

Planning 分类：

- Reactive
- Predictive

Decision 分类：

- Long term. 从 NY 导航到 LA
- Short term. 现在能否换道到右车道？能否穿过路口？
- Immediate. 加速或刹车？

Reactive Planning

- rule-based
- we have rules that take into account the current state of ego and other objects and give decisions
- e.g. If there is a pedestrian on the road, stop


Predictive Planning

- Make predictions about other vehicles and how they are moving. Then use these predictions to inform our decisions.
- e.g. That car has been stopped for the last 10 seconds. It is going to be stopped for the next few seconds.
