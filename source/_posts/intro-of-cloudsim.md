---
date: 2015/12/16 12:34:50
title: Cloudsim 云计算仿真系统介绍
categories:
- simulation
tags:
- cloudsim
- simulation
---

bw: bandwidth
cloudlet
CIS: cloud information service,执行资源的分配与注册， CloudSim 初始化时会生成该类的实例，不需要自己处理
PE: Processing Element, CPU unit
vm: virtual machine, runs inside a Host, sharing hostList with other VMs. process cloudlets.process according to CloudletScheduler's(timeShared, spaceShared) defined policy. VmSchedulerTimeShared is property of Host.
datacenterBroker: 代理仁，代表用户执行操作，对用户隐藏虚拟机管理（创建，提交任务到虚拟机，虚拟机销毁）。
Event: 


可以有多个datacenter， 一个datacenter有多个host，一个host有多个PE，


一个仿真流程

1. 设置用户数量（broker count)
2. 设定通用的变量
3. 创建CIS（自动创建）
4. 创建Datacenter：包括 host + characteristics(Pe, Ram, Bw, price)
5. 创建DatacenterBroker
6. 创建Vm instances
7. 提交Vm　list 到 broker
8. 创建 Cloudlet(tasks) List
9. 提交(submit) Cloudlet lists到broker
9. start simulation
10. stop simulation
11. 输出仿真的状态


## CloudSim Events the heartbeat of simulation 

非常重要的类： SimEvent

