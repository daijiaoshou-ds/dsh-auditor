# dsh-auditor 安装

## 一、装插件

在 DSH 里 **插件 → 添加插件**，输入：

```
dsh-auditor
```

这是 npm 上的包（`npmjs.com/package/dsh-auditor`），pnpm 直接从 registry 拉。装不上时
依次退这两条：

- **GitHub**：`github:daijiaoshou-ds/dsh-auditor`（同一份装配产物，另一个仓库）；
- **离线**：把 zip 解压到**一个固定目录**（例如 `D:\dsh-auditor`，别放下载目录——以后
  升级要覆盖同一目录），在 **插件 → 添加插件** 里填这个目录的绝对路径。升级时：插件页
  卸载 → 覆盖解压 → 重新添加 → 刷新/重启 DSH。

命令行等价（不常用）：

```
plugin_manager install_bundle  target=dsh-auditor
```

装完在 DSH 里打开「审计工作台」页签。

## 二、配环境（第一次，约 10–20 分钟）

页签左上角「环境配置」里六步，从上到下点：

| 步骤 | 装什么 | 国内网络 |
|---|---|---|
| Python 解释器 | 3.13.5（约 28MB） | python.org 慢就走华为云镜像（已内置） |
| 虚拟环境 | `%LOCALAPPDATA%\dsh-auditor\venv`（**不在引擎目录里**，见第六节） | 本地，几秒 |
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

## 四、装在哪些目录（升级时这份东西不会被抹掉）

| 东西 | 位置 | 说明 |
|---|---|---|
| 插件代码 | `<解压目录>` 或 pnpm 的包目录 | 升级时整个替换 |
| 引擎代码 | `%LOCALAPPDATA%\dsh-auditor\engine` | 随插件版本同步，`.engine-version` 变了才覆盖 |
| **venv** | `%LOCALAPPDATA%\dsh-auditor\venv` | 引擎的**兄弟目录**，不随插件升级 |
| 模型 / BGE 缓存 | `engine\core\nn\_storage`、`engine\core\nn\models` | 大文件，不随升级重下 |
| 任务与产物 | `engine\_storage_jobs` | 一份账一个目录 |

升级（换 zip）：插件页卸载 → 覆盖解压到同一目录 → 重新添加 → 重启 DSH。
**环境不用重装**：venv 和模型都不在插件目录里。

## 五、装完发现环境页有红项，先看它说的那句话

六步每一步都自带说明；红项分两种，处理方式完全不同：

- **"没装"** —— 真缺，点对应按钮装。
- **"探测失败（退出码 N）：……"** —— 解释器在，但跑不起来。括号里是它的原话，
  照那句话去查；别急着重装，重装解决不了这一类。

## 六、`failed to locate pyvenv.cfg`：venv 的位置被挡了

症状：六步里第 1、2 步红，细节写着解释器读不到同目录的 `pyvenv.cfg`，可文件明明在。

原因：venv 的启动器（`venv\Scripts\python.exe`）要读同目录的 `pyvenv.cfg`；当它躺在
DSH 沙箱经手过的目录里（旧版本把 venv 建在 `engine\` 下），这一步会被挡住，而报错读起来
像"没装 Python"。DSH 默认的"工作区内修改"权限、以及给目录打的 ACL，是这类症状的来源。

处理：环境页第 2 步会变成 **「迁移虚拟环境」**，点一下即可——只把目录改名到
`%LOCALAPPDATA%\dsh-auditor\venv`，并在老位置留一个目录联接，**依赖一个都不重装**。
新装的插件本来就把 venv 建在数据目录，不会再遇到。

手工等价操作（迁移按钮不可用时，先退出 DSH）：先把 venv 改名到数据目录，再在老位置
留一个目录联接——配置里写死的 python 路径、旧任务记录照样能用：

```powershell
Move-Item "$env:LOCALAPPDATA\dsh-auditor\engine\venv" "$env:LOCALAPPDATA\dsh-auditor\venv"
cmd /c mklink /J "$env:LOCALAPPDATA\dsh-auditor\engine\venv" "$env:LOCALAPPDATA\dsh-auditor\venv"
```

如果哪天环境页又变红、而 `engine\venv` 已经是个联接（插件/引擎重装把它顶掉了），
按上面第二条重做联接即可：`cmd /c rmdir "...\engine\venv"` 再 `mklink /J`。
