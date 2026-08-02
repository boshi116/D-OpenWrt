# luci-app-sqm-controller

## 项目简介

`luci-app-sqm-controller` 是一个面向 OpenWrt 的 LuCI Web 管理插件，后端结合 `tc`、`HTB`、`IFB`、`fq_codel` / `CAKE`、`nftables` 和 `conntrack` 实现业务分类与低时延流量控制。

与只提供静态限速的传统方案相比，本项目在基础带宽整形之外，还提供：

- 配置向导与基础设置页面
- 服务启停与运行状态查看
- 实时监控与历史监控
- 流量分类统计与命中状态查看
- 场景模板与策略中心
- 自检、日志查看与导出

## 主要特性

- 支持 `fq_codel` 与 `CAKE` 两种队列算法
- 支持上行 / 下行独立带宽整形
- 基于 `HTB + IFB` 实现上下行统一控制
- 基于 `nftables + ct mark + tc filter` 实现流量分类
- 支持游戏、流媒体、批量下载、默认流量等业务类别
- 提供自动 / 均衡 / 游戏优先 / 流媒体优先 / 批量下载等策略模式
- 提供 LuCI 可视化页面，降低命令行配置门槛
- 支持日志、自检、策略状态与分类健康度诊断

## 项目结构

```text
luci-app-sqm-controller/
├── Makefile
├── check-deps.sh
├── description.txt
├── files/
│   ├── etc/
│   │   ├── config/sqm_controller
│   │   └── init.d/sqm-controller
│   ├── share/rpcd/acl.d/luci-app-sqm-controller.json
│   └── usr/
│       ├── bin/
│       │   ├── sqm-start.sh
│       │   ├── sqm-status.sh
│       │   └── sqm-stop.sh
│       └── lib/sqm-controller/
│           ├── main.py
│           ├── tc_manager.py
│           ├── firewall_manager.py
│           ├── traffic_classifier.py
│           ├── traffic_stats.py
│           ├── monitor.py
│           ├── policy_engine.py
│           ├── config_manager.py
│           ├── self_check.py
│           ├── speedtest.py
│           └── template_manager.py
└── luasrc/
    ├── controller/sqm_controller.lua
    ├── model/cbi/sqm_controller.lua
    └── view/sqm_controller/
        ├── wizard.htm
        ├── status.htm
        ├── monitor.htm
        ├── traffic.htm
        ├── templates.htm
        ├── policy.htm
        ├── logs.htm
        └── help.htm
```

## 工作流程概述

1. 用户通过 LuCI 页面完成基础配置、分类配置和策略设置  
2. 配置统一写入 UCI：`/etc/config/sqm_controller`  
3. 后端 Python 模块读取配置并建立运行态  
4. `tc` / `HTB` / `IFB` 负责带宽整形与队列调度  
5. `nftables` 负责按协议、端口、地址等条件分类打标  
6. `ct mark` 用于连接级标记继承，减少重复匹配  
7. 监控与策略模块根据链路状态决定是否调整运行模式

## 依赖环境

项目 `Makefile` 中声明的主要依赖如下：

- `python3`
- `python3-light`
- `curl`
- `ca-bundle`
- `kmod-ifb`
- `kmod-sched-core`
- `kmod-sched-cake`
- `kmod-sched-connmark`
- `kmod-sched-ctinfo`
- `luci-base`
- `luci-compat`
- `luci-lib-ip`
- `luci-lib-nixio`

运行时还需要系统具备以下命令：

- `python3`
- `tc`
- `ip`
- `uci`
- `nft` 或 `iptables`

## 适用平台

建议运行环境：

- OpenWrt 23.05 及以上版本
- 支持 `ifb`、`htb`、`fq_codel`、`cake` 等内核模块
- 适合 x86 软路由或具有一定性能余量的 OpenWrt 设备

## 安装方式

### 方式一：作为 OpenWrt 软件包编译

将本目录放入 OpenWrt SDK / 源码树中，例如：

```bash
package/luci-app-sqm-controller
```

然后执行：

```bash
make menuconfig
make package/luci-app-sqm-controller/compile V=s
```

编译完成后可得到对应的 `.ipk` 安装包。

### 方式二：安装已编译的 ipk

从本项目release的下载编译好的ipk安装包，上传到OpenWrt系统进行安装

安装完成后，在 LuCI 后台的“服务”菜单中进入 `SQM 流量控制` 页面。

## 页面功能

- 基础设置：接口、上下行带宽、队列算法、启停控制
- 配置向导：面向普通用户的分步配置流程
- 状态监控：查看服务状态、运行态、分类后端和策略状态
- 实时监控：查看带宽、延迟、丢包、CPU、内存、温度等指标
- 分类流量统计：查看业务类别命中情况与分类健康度
- 场景模板：快速套用预设配置
- 策略中心：手动或自动切换策略模式
- 系统日志：查看运行日志和策略日志
- 帮助文档：查看机制说明与排障命令

## 已知限制

- 当前流量分类主要基于协议、端口、源地址和连接标记，面对复杂加密业务时识别粒度仍有限
- 在低性能嵌入式设备上，监控频率、分类规则数量和队列参数仍需进一步调优

