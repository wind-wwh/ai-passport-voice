# AI Passport 语音输入 · v1.0.0 发布清单（RELEASE）

> **本文件是本次发布的权威说明**，面向两类读者：① 人类（下载页第一眼）；② 用户电脑上的 AI 助手（先读它，再决定下载什么、读哪份文档）。
> 本文件**不在** zip 内（因为它要写 zip 自身的 SHA-256，写进去就会自指循环）。
> 版本：**v1.0.0** ｜ tag：**`v1.0.0`** ｜ 发布日期：2026-10-17

---

## 一、本版资产（Asset）

| 资产 | 大小 | SHA-256 | 用途 |
|---|---|---|---|
| `ai-passport-voice-release-v1.0.0.zip` | 493,847 字节 | `053487AA22E356327F0CA8E622DBC29A1EB34E20B0070F324E4A8D9379AB98F6` | **唯一安装包**：固件镜像 + 电脑端脚本 + 全部文档 |

包内还有一份 `SHA256SUMS.txt`（45 条），用于逐个校验**解压后**的文件。注意：

- `SHA256SUMS.txt` 只能自证「包内文件没被改」，**不能证明 zip 本身没被替换** —— 所以 zip 的哈希只出现在**本文件**和 **Release 说明**里；
- 两者必须完全一致。

**关键哈希（钉死，任何版本都不变）**

```
包内固件：固件/ai-passport-voice-full.bin   SHA-256 = 719955AE196DB8CC17ADF16B50F2D27119B79BD1B80E472AA25C3DCABC1F1DF4
大小 = 695,488 字节
```

---

## 二、包内文件说明

### 1) 根目录（8 个）

| 文件 | 用途 | 谁该读 |
|---|---|---|
| `新用户使用说明_从零开始.md` | **人类完整上手**：装 Python 3.11 → 解压 → 刷固件 → 申请火山 key → 配置电脑端 → 第一次说话 → 排障表 | 人类（必读） |
| `AI助手接管安装指南.md` | **给 AI 助手的执行版**：能力边界表、Step A~G 带校验点、禁止事项、汇报格式 | AI 助手（必读） |
| `GitHub发布_配套提示词.md` | 发布者用：Release 说明模板 + 仓库清单约定 + 提示词模板（不含 zip 哈希，避免自指） | 维护者 |
| `发布说明_v1.0.0.md` | 一页纸速查：包结构、三步上手、兼容性、验收记录 | 人类（选读） |
| `变更说明_CHANGELOG.md` | 相对上一版（固件 ③）改了什么 | 人类（选读） |
| `LICENSE` | MIT，覆盖本包脚本与文档 | 任何分发者 |
| `THIRD_PARTY_NOTICES.md` | 固件含 ESP-IDF(Apache-2.0)、LVGL(MIT)、cJSON(MIT)；借鉴来源的完整版权声明；商标独立性声明 | 任何分发者 |
| `SHA256SUMS.txt` | 45 条包内文件哈希 | 校验工具 |

### 2) `固件/`（2 个）

| 文件 | 用途 |
|---|---|
| `ai-passport-voice-full.bin` | **合并镜像**，用 `--flash_mode dio --flash_size 8MB --flash_freq 80m` 从 **`0x0`** 整片写；695,488 字节 |
| `刷机教程.md` | 刷机详细版：装 esptool、找 COM 口（`VID:PID=303A:1001`）、刷写、成功标志、报错对照 |

> ⚠️ 整片刷会清空 NVS → 设备上的目标选择 / auto-send / AUTOSTART 回到默认（Notepad、OFF、AUTOSTART **ON**）。

### 3) `电脑端/`（5 个）

| 文件 | 用途 |
|---|---|
| `新建环境.bat` | 双击一次：建 `companion\.venv` 并按 `requirements.lock.txt` 装依赖（发布包**不含** `.venv`） |
| `换电脑部署清单.md` | 换机部署完整步骤（含 key 归属、自启动、验证） |
| `换电脑部署清单_实操版.md` | 同上，带更多命令样例 |
| `登录自动启动_部署说明.md` | 三层兜底原理、权限要求、FAQ（与 `companion\` 内同名文件内容一致） |
| `配置助手使用说明.md` | 配置助手 ①~⑥ 按钮说明（与 `companion\SETUP_TOOL.zh_CN.md` 内容一致） |

### 4) `电脑端/companion/`（31 个，电脑端程序本体）

| 文件 | 用途 |
|---|---|
| `新建`级入口：`配置助手.bat`、`SetupTool.bat` | 双击启动 GUI 配置助手（两个文件内容相同；**必须普通权限**，管理操作会自己弹 UAC） |
| `setup_tool.py` | 配置助手主体：安装/修复自启动、启动、停止、卸载、自检、状态、日志（CLI 也走它） |
| `autostart.py` | 登录自启启动器：spawn 独立的 daemon + relay 兄弟进程，单实例锁、日志轮转、每 10 分钟自愈 |
| `relay.py` | 串口收帧 → ASR 子进程 → 队列发布（`--inject` 开自动注入；`--port auto` 自动找设备） |
| `asr_stdin.py` | 火山 ASR 子进程（bigmodel 流式），只读 `volcano_api_key` 一个字段 |
| `inject_daemon.py` | 独立注入 daemon（必须独立进程：SendInput 在长串口进程里会"假成功不投递"） |
| `inject_windows.py` | Windows 校验 + 剪贴板 + `Ctrl+V`（可选 `+Enter`）注入，含焦点护栏 |
| `inject_targets.py` | 三目标规格：`notepad.exe` / `deepseek harness.exe` / `chatgpt.exe` |
| `inject_queue.py` | 原子文件队列（v2，含 target 与 auto_send） |
| `inject_client.py` | 独立注入自测客户端 |
| `inject_unicode.py`、`clipboard_text.py` | 剪贴板 / Unicode 输入辅助 |
| `serial_frame.py` | 分帧协议（与固件 `usb_link_framing.c` 逐字节一致，**改一侧必须同步另一侧**） |
| `config.example.json` | key 模板 → 复制成 `config.local.json` 再填（**包内不含真 key**） |
| `requirements.txt` / `requirements.lock.txt` | 依赖：`pyserial==3.5`、`pywin32==312`、`websockets==17.2`（需 Python ≥ 3.11） |
| `.gitignore` | 排除 `.venv/`、`config.local.json`、日志与队列目录 |
| `README.md` / `README.zh_CN.md` | 电脑端总览 |
| `DEVICE_TARGETS.md` / `DEVICE_TARGETS.zh_CN.md` | 设备端三目标 + auto-send + AUTOSTART 使用说明 |
| `TARGETS.md` / `TARGETS.zh_CN.md` | 历史「终端选目标」指南（现以设备端选择为准） |
| `INJECTION_POC.md` / `INJECTION_POC.zh_CN.md` | 注入方案 PoC 记录与取舍 |
| `SETUP_TOOL.md` / `SETUP_TOOL.zh_CN.md` | 配置助手说明原文 |
| `SETUP_REWORK.md` / `SETUP_REWORK.zh_CN.md` | 服务稳定性返工记录（`ready.json` 并发崩溃的修复） |
| `登录自动启动_部署说明.md` | 三层兜底说明原文 |

---

## 三、安装入口（按读者分流）

| 读者 | 该读 | 顺序 |
|---|---|---|
| **人类** | `新用户使用说明_从零开始.md` | 按章节 1→6 走完即可；第 9 节是排障表 |
| **AI 助手（DSH / Claude Code / Codex CLI）** | `AI助手接管安装指南.md` | 先读它拿执行边界，再按 **Step A → G** 执行；人类步骤的事实来源仍是上面那份 |

> 两条路共用同一套事实（文档名、哈希、闸门），不存在"给人和给 AI 两套说法"。

---

## 四、人类专属步骤（AI 助手必须停下）

| # | 事项 | 为什么 AI 做不了 / 不许擅自做 |
|---|---|---|
| 1 | 刷固件（`esptool write_flash`） | 会清空设备设置；刷错 COM 口代价高 → **需用户明确说"同意刷机"** |
| 2 | 任何 UAC 弹窗 | 系统安全边界，只能真人点"是" |
| 3 | 设备按键 / 录音 / 看屏幕 | 没有设备通道 |
| 4 | 火山引擎控制台注册、实名、开通、取 key | 涉及账号与付费 |
| 5 | 确认 USB 线是否能传数据 | 物理属性无法远程判断；找不到 COM 口多半是线的问题 |
| 6 | 装 Python 3.11（首次） | 安装向导需要人，或用静默参数也须人授权 |

---

## 五、校验方法

```powershell
# zip 完整性（与本文第一节的 SHA-256 比对）
Get-FileHash .\ai-passport-voice-release-v1.0.0.zip -Algorithm SHA256

# 解压后逐文件校验（在包根目录执行；没有输出 MISMATCH 即完整）
Get-Content .\SHA256SUMS.txt | ForEach-Object {
  if ($_ -match '^([0-9A-Fa-f]{64})\s+\*(.+)$') {
    $h = (Get-FileHash $matches[2] -Algorithm SHA256).Hash
    if ($h -ne $matches[1].ToUpper()) { "MISMATCH: $($matches[2])" }
  }
}

# 固件单独校验（刷机前必做）
Get-FileHash .\固件\ai-passport-voice-full.bin -Algorithm SHA256
(Get-Item     .\固件\ai-passport-voice-full.bin).Length
```

---

## 六、环境要求

| 项 | 要求 |
|---|---|
| 系统 | Windows 10 / 11，64 位 |
| 权限 | 普通用户 + 装自启动时能确认一次 UAC |
| 运行时 | **Python 3.11.x（必须 ≥ 3.11）**；包内**不含** Python、不含 `.venv` |
| 网络 | 装依赖需访问 PyPI；识别需访问火山 ASR（云端） |
| 硬件 | AI Passport 设备 + **能传数据**的 USB 线 |
| 账号 | 你自己的火山引擎语音识别 API key（按量计费） |
| 路径 | 任意（含中文路径）；**不需要修改任何代码** |

---

## 七、兼容性

| 组合 | 结果 |
|---|---|
| 固件 v1.0.0（④）+ 本包电脑端 | ✅ 完整功能（AUTOSTART 总开关、配置助手、三层自愈） |
| 固件 v1.0.0（④）+ 旧电脑端 | ✅ 可跑（旧 relay 忽略 `system.autostart`，默认视为 ON） |
| 旧固件（③ 及更早）+ 本包电脑端 | ✅ 可跑（设备不发 `system.autostart`，新 relay 默认 ON） |

---

## 八、已知问题（本版不修）

1. **英文专名误听**：`DeepSeek` → `deep sick`（云端 ASR 质量，需在火山控制台配热词）。
2. **多段注入之间无换行**：需手动回车；后续版本会加开关。
3. **识别延迟约 4~5 秒**：`asr_stdin.py` 按 100 ms 节奏逐块上传；优化需真机验证云端接受度。
4. **注入瞬间需保持目标窗口聚焦**（约 4~5 秒内），否则被安全拒绝（日志 `foreground_is_not_dsh`）。

---

## 九、许可与归属

- 本包脚本与文档：**MIT**（见 `LICENSE`）；
- 固件镜像内含 **ESP-IDF（Apache-2.0）**、**LVGL（MIT）**、**cJSON（MIT）**；
- 借鉴来源：[`zhaohuaxiaoy/folo-ai-passport-voice`](https://github.com/zhaohuaxiaoy/folo-ai-passport-voice)（MIT © 2026 FoloToy），本项目为其裁剪重写版；
- FoloToy / AI Passport 等名称与商标归其权利人，本项目为独立衍生，无官方隶属或背书关系。

详见 `THIRD_PARTY_NOTICES.md`。

---

## 十、给用户的提示词（复制给 DSH / Claude Code / Codex CLI）

```text
帮我在 Windows 上安装「AI Passport 语音输入 v1.0.0」。设备已用 USB 数据线连好并开机。
来源只认这个仓库的 v1.0.0 正式版：https://github.com/<OWNER>/<REPO>

第一步，先读这份发布清单（资产名、大小、SHA-256、每个文件的用途都在里面）：
  https://raw.githubusercontent.com/<OWNER>/<REPO>/v1.0.0/RELEASE.md
如果 raw 地址取不到，改用 GitHub API：
  https://api.github.com/repos/<OWNER>/<REPO>/releases/tags/v1.0.0
在 assets 里找 ai-passport-voice-release-v1.0.0.zip，用它的 browser_download_url 下载。

第二步，下载并校验：
  - zip 的 SHA-256 必须等于：
    053487AA22E356327F0CA8E622DBC29A1EB34E20B0070F324E4A8D9379AB98F6
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
