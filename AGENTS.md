# AGENTS.md

本项目是**个人学习记录仓库**（Jetson AGX Orin Industrial + 自研载板），不是生产代码库。
Pi 在本目录及子目录运行时自动加载本文件；修改后执行 `/reload`。

## 项目定位

- 目标：以“动手记录”的方式，掌握 Jetson AGX Orin Industrial 的 Bring-up、算力使用与性能调优，最终服务安防雷达（脉冲相控阵）算法落地。
- 产出物是**可复现的笔记**，不是产品代码。笔记的可信度 > 篇幅。
- 读者是我自己几个月后的回顾，所以“命令原文 + 版本号 + 实际输出”比结论更重要。

## 设备事实基线

- 模块：Jetson AGX Orin Industrial 64GB —— 248 TOPS(INT8)、15–75 W 可配置、2048-core Ampere GPU + 64 Tensor Core、12-core Cortex-A78AE 2.0 GHz、64 GB 256-bit LPDDR5（带 ECC）、64 GB eMMC、-40 ~ +85 °C。
  来源：NVIDIA 官方博客与数据手册（https://developer.nvidia.com/blog/step-into-the-future-of-industrial-grade-edge-ai-with-nvidia-jetson-agx-orin-industrial/）
- 注意：工业版的 TOPS 低于商用版 AGX Orin 64GB（275 TOPS），主频/功耗包络不同，**不要照抄商用版的调优数据**。
- 载板：公司自研，**板上无 EEPROM**。这意味着不能按 devkit 流程直接刷机，需要在 BSP/board config 与设备树侧处理（NVIDIA L4T Developer Guide「Jetson Module Adaptation and Bring-Up」，Orin 系无 CVB EEPROM 时需将 `cvb_eeprom_read_size` 置为 `<0x0>`）。具体步骤未在本仓库验证前，一律标注“待实测确认”。
- 以上参数若与实际设备不符，以设备实测（`/proc/device-tree/model`、`nvpmodel -q`、`jetson_release -v`）为准，并回来修订本节。

## 硬性约束

1. **不编造**。硬件参数、工具用法、寄存器/DT 字段、性能数字，必须能指到 NVIDIA 官方文档（docs.nvidia.com 的 Jetson Linux Developer Guide / JetPack 文档）或本地实测输出。查不到就说“查不到”。
2. **标注事实等级**。每条关键结论后必须能看出它属于哪一类：
   - `[官方]` 附文档链接与章节名
   - `[实测]` 附版本、命令与输出片段
   - `[推测]` / `[论坛]` 必须显式标注，不得写成定论
3. **记录版本**。任何实验记录开头必须写明：JetPack/L4T 版本（`cat /etc/nv_tegra_release`）、`jetson_release -v`、`/proc/device-tree/model`、功耗模式（`nvpmodel -q`）。不同版本结论不通用。
4. **命令可复现**。粘贴命令用原文，不省略参数；失败的命令也要保留报错全文（截断处标 `...`）。
5. **区分“开发主机”和“设备端”**。刷机/交叉编译在 x86 Ubuntu 主机上做，标清哪条命令在哪个机器上执行。
6. 不提交密钥、token、序列号、公司内部原理图/PCB 文件；截图里的 SN 要打码。
7. 不执行对设备或主机的破坏性操作（`rm -rf`、对 eMMC/NVMe 覆写、`dd` 写块设备、`apt upgrade` 跨版本升级 L4T）而不先确认；刷机前确认可回滚。

## 笔记与目录约定

建议结构（按需调整）：

```
docs/            # 分阶段学习笔记，01-xx 02-xx 编号
labs/            # 每个实验一个目录：脚本、配置、结果数据
assets/          # 图片、日志、profiling 输出
sessions/        # Pi 会话导出归档（.html），便于公司侧回看对话过程
AGENTS.md
```

- `sessions/` 下只放已脱敏的会话导出；`.gitignore` 默认忽略 `pi-session-*.html`，仅放行 `sessions/` 目录内的文件。

每个实验记录使用固定结构：

```
## 目标
## 前置条件（硬件连接、版本、依赖）
## 步骤（命令原文 + 关键输出）
## 结果与数据（表格；性能要给单位与测量方法）
## 结论（标 [官方]/[实测]/[推测]）
## 踩坑与排查
## 待办
```

## 性能数据规范

- 报性能必须同时给出：精度（FP32/FP16/INT8）、batch、功耗模式（如 MAXN/15W）、clocks 状态（是否 `jetson_clocks`）、测量工具（tegrastats / nsight-systems / trtexec / 自研计时）。
- 延迟类数据说明是否含前后处理与传输（D2H/H2D），雷达管线尤其要区分「算法耗时」和「端到端耗时」。
- 不比较来路不明的第三方跑分。

## 回复风格

- 中文回答，简洁，先给结论再给细节。
- 提到文件给相对路径 + 行号；提到设备上路径给绝对路径。
- 声称“已验证/已跑通”必须有实际输出为依据。
- 涉及未验证的 Jetson 细节时，明确说“需要查官方文档”而不是给出看似合理的猜测。

## 禁止事项

- 不把商用 AGX Orin / Orin NX / Xavier 的资料当成工业版结论直接套用。
- 不在未确认载板 EEPROM 处理方案前，按 devkit 流程刷机。
- 不修改公司提供的 BSP 原始文件而不留备份与改动说明。
