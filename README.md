# AI Passport 有线语音输入 · v1.0.0

在 FoloToy **AI Passport**（ESP32-C3）上实现**有线语音输入**：按住设备按键说话 → 松开 → 电脑端云端识别 → 文字**自动上屏**到当前输入框（Codex / DSH / Notepad 三目标，可选自动回车）。

- 设备只负责录音与选目标；识别与注入在电脑端完成（Windows 10/11）。
- 走 USB 串口，**不是蓝牙、不是 Wi-Fi**。
- 含 **AUTOSTART 总开关**（设备端决定整条链路工作/待机）、**登录自启 + 每 10 分钟自愈**、**配置助手 GUI**。

---

## 一、下载

| 文件 | 大小 | SHA-256 |
|---|---|---|
| [`ai-passport-voice-release-v1.0.0.zip`](../../releases/download/v1.0.0/ai-passport-voice-release-v1.0.0.zip) | 493,405 字节 | `ddd170198a9c0ceca42ad573965883549d3a1076ee59059b3a061e81751dee76` |

```powershell
Get-FileHash .\ai-passport-voice-release-v1.0.0.zip -Algorithm SHA256
```

> ⚠️ 只下载上方这一个 zip。**不要**下载 Release 页里自动生成的 `Source code (zip)` / `Source code (tar.gz)` —— 那两个只是仓库的源码快照，**不含固件镜像和电脑端脚本**。

**包内固件（钉死值，任何版本都不变）**

```
固件/ai-passport-voice-full.bin
SHA-256 = 719955AE196DB8CC17ADF16B50F2D27119B79BD1B80E472AA25C3DCABC1F1DF4
大小     = 695,488 字节
```

---

## 二、安装前准备

| 项 | 要求 |
|---|---|
| 系统 | Windows 10 / 11，64 位 |
| 权限 | 普通用户 + 装自启动时确认一次 UAC |
| Python | **3.11.x（必须 ≥ 3.11）**；包内**不含** Python 运行时 |
| 网络 | 装依赖需访问 PyPI；识别需访问火山引擎 ASR（云端） |
| 硬件 | AI Passport 设备 + **能传数据**的 USB 线 |
| 账号 | 你自己的**火山引擎语音识别 API key**（按量计费） |
| 路径 | 任意（含中文路径）；**不需要修改任何代码** |

---

## 三、两种装法

### A. 自己装（推荐先看这个）

解压后按顺序读包内文档：

1. `新用户使用说明_从零开始.md` —— **完整上手**：装 Python → 解压 → 刷固件 → 申请 key → 配置电脑端 → 第一次说话 → 排障表；
2. 卡住时看其中的第 9 节排障表，或 `固件/刷机教程.md`（刷机细节）。

### B. 让 AI 助手帮你装

把下面整段复制，发给你电脑上的 AI 助手（DSH / Claude Code / Codex CLI）：

```text
帮我在 Windows 上安装「AI Passport 语音输入 v1.0.0」。设备已用 USB 数据线连好并开机。
安装包只认这个仓库的 v1.0.0 正式版：https://github.com/wind-wwh/ai-passport-voice

第一步，先读这两处信息（不要凭记忆，也不要跑去装别的同名仓库）：
  1) 发布清单（每个文件的用途、安装入口、人类专属步骤）：
     https://raw.githubusercontent.com/wind-wwh/ai-passport-voice/main/RELEASE.md
  2) 发布元数据（不可变，含附件 sha256 digest 与下载地址）：
     https://api.github.com/repos/wind-wwh/ai-passport-voice/releases/tags/v1.0.0
     在 assets 里找 ai-passport-voice-release-v1.0.0.zip，用 browser_download_url 下载。
  若这两处写的大小/哈希与我下面给的不一致，立刻停下来告诉我，不要自己判断哪个对。
  注意：不要下载自动生成的 Source code (zip)/(tar.gz) —— 那里面没有固件和脚本。

第二步，下载并校验：
  - zip 的 SHA-256 必须等于：
    ddd170198a9c0ceca42ad573965883549d3a1076ee59059b3a061e81751dee76
  - 大小应为 493,405 字节。
  - 对不上就立刻停止：不要解压、不要运行其中任何文件、不要换地址重试。
  - 校验通过后解压到 D:\AIPassport。

第三步，按包内文档执行（不要凭记忆装）：
  - 先读 `AI助手接管安装指南.md`（给你看的执行边界与校验点），
  - 人类步骤的事实来源是 `新用户使用说明_从零开始.md`，
  - 按其 Step A → G 执行：确认 Python ≥ 3.11、跑 `电脑端\新建环境.bat`、
    把 key 写进 `companion\config.local.json`、跑 `setup_tool.py --install` 与 `--run`。

以下必须停下来问我、等我回话，不要自己绕过：
  ① 刷固件（没我说"同意刷机"不许执行，执行前先报 COM 口与"会清空设备设置"）；
  ② 任何 UAC 弹窗；③ 设备按键/录音/看屏幕；④ 火山控制台注册与取 key；⑤ 确认 USB 线能否传数据。

纪律：不要打印我的火山 API key；不要以管理员身份运行电脑端服务；不要修改包内任何 .py 文件；
不要跳过校验、不要"先试试看"。

结束条件：配置助手显示「服务：运行中（心跳正常）」+「设备：在线」，并且我在记事本按住 UP
说话后文字成功上屏。最后按【已完成】/【需要你操作】/【未验证或风险】三段汇报。
```

---

## 四、装完之后

| 场景 | 怎么做 |
|---|---|
| 每天用 | 登录 Windows 后服务自动起，设备插上即可；服务死掉最多 10 分钟自愈 |
| 改了 `config.local.json` / 任何 `.py` | 配置助手 **③ 停止 → ② 恢复**（Python 不热重载） |
| 临时停用 | 配置助手 **③ 停止**（10 分钟内可能被自愈拉起） |
| 长期停用 / 卸载 | 先 **④ 卸载自启动**，再 ③ 停止，最后删目录 |
| 换电脑 | 见包内 `电脑端/换电脑部署清单.md` |

**设备按键**：选择页 4 行（Codex → DSH → Notepad → AUTOSTART ON/OFF），`OK` 进录音页 / 翻转开关；录音页**按住 UP 说话、松开识别**（松开后 4~5 秒内保持目标窗口聚焦）。
> ⚠️ 三键共用一路 ADC：**按住 UP 时 DOWN/OK 识别不稳**，要先松开 UP 再按。

---

## 五、已知问题（v1.0.0 不修）

1. **英文专名误听**：`DeepSeek` → `deep sick`（云端 ASR 质量，需在火山控制台配热词）。
2. **多段注入之间没有换行**：需手动回车。
3. **识别延迟约 4~5 秒**：受上传节奏与云端返回影响。
4. **注入瞬间需保持目标窗口聚焦**，否则被安全拒绝（日志 `foreground_is_not_dsh`）。

---

## 六、仓库文件说明

| 文件 | 用途 |
|---|---|
| [`RELEASE.md`](RELEASE.md) | **本次发布的权威清单**：资产哈希、46 个包内文件的逐个说明、安装入口、人类专属步骤 |
| [`LICENSE`](LICENSE) | MIT（覆盖本包脚本与文档） |

固件镜像内含 **ESP-IDF（Apache-2.0）**、**LVGL（MIT）**、**cJSON（MIT）**；协议与音频管线设计借鉴 [`zhaohuaxiaoy/folo-ai-passport-voice`](https://github.com/zhaohuaxiaoy/folo-ai-passport-voice)（MIT © 2026 FoloToy）。完整声明见包内 `THIRD_PARTY_NOTICES.md`。

FoloToy / AI Passport 等名称与商标归其权利人；本项目为**独立衍生的第三方项目**，与 FoloToy 官方无隶属或背书关系。

---

## 七、安全提醒

- `config.local.json` 里是**明文 API key**，只留在本机：不进仓库、不发群、不贴聊天记录；
- 火山 ASR 按账号计费，**换人使用必须换成对方自己的 key**；
- 任何索要你的火山账号密码的行为都应视为可疑——本项目只需要一把 API key；
- 若怀疑泄露 → 火山控制台轮换 key，再更新本机 `config.local.json`。
