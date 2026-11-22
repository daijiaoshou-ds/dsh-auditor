# dsh-auditor 安装

## 一、装插件

在 DSH 里 **插件 → 添加插件**，输入这一行（公开仓库，pnpm 直接拉）：

```
github:daijiaoshou-ds/dsh-auditor
```

离线装（GitHub 连不上时）：把 zip 解压到**一个固定目录**（例如 `D:\dsh-auditor`，
别放下载目录——以后升级要覆盖同一目录），然后在 DSH 的 **插件 → 添加插件** 里填这个
目录的绝对路径。升级时：插件页卸载 → 覆盖解压 → 重新添加 → 刷新/重启 DSH。

命令行等价（不常用）：

```
plugin_manager install_bundle  target=<本目录的绝对路径>
```

装完在 DSH 里打开「审计工作台」页签。

## 二、配环境（第一次，约 10–20 分钟）

页签左上角「环境配置」里六步，从上到下点：

| 步骤 | 装什么 | 国内网络 |
|---|---|---|
| Python 解释器 | 3.13.5（约 28MB） | python.org 慢就走华为云镜像（已内置） |
| 虚拟环境 | 引擎目录下的 `venv/` | 本地，几秒 |
| 基础依赖 | pandas / numpy / pyarrow / calamine / openpyxl | 默认阿里云源 |
| 模型依赖（CPU） | torch CPU 轮子（约 122MB）+ transformers / modelscope | torch 官方源 + 交大镜像兜底 |
| 模型文件 | hajishou-V2.0（约 2GB） | ModelScope 国内直下 |
| 基座模型（BGE） | bge-large-zh（约 1.6GB） | ModelScope 优先（不预下的话，第一本账要静默等十几分钟） |

GPU 版不装也能用全部功能（推理是 CPU 跑得动的规模）；要显卡再在「模型依赖」里
切成 CUDA 版（约 2.4GB）。

## 三、用

1. 右栏「＋」新建任务：选层 → 上传序时账 → 确认字段映射；
2. 如果出现「先对一下科目」：给认不出的写法挑标准科目（写进全局别名表），
   或者「先不管，直接开始」；
3. 主区点功能开跑；停到复核位可以交给这个任务的 AI 复核会话，也可以不复核。
