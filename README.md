# astrbot_plugin_qq_call

AstrBot 的 **Windows QQ 原生语音通话** 插件：主人在聊天里明确说“给我打电话”时，插件启动通话 agent，并模拟点击 QQ 聊天窗口的电话按钮，发起 QQ 语音通话。

> 这是 Windows UI 自动化方案，不是 QQ 官方 API。QQ 更新界面后按钮坐标可能失效。

## 功能

- 主人说“给我打电话 / 打给我”时直接触发，不依赖模型是否选择工具；
- 只允许配置的 `owner_qq` 触发；
- 自动启动 `call_agent.py`，在接通后用 ASR + LLM + TTS 对话；
- 可选自动接听（默认关闭，必须配置来电白名单）。

## 环境

- Windows 10/11；
- 桌面版 QQ NT，且机器人小号已登录；
- Python 3.10+；
- 虚拟声卡（如 VB-Audio Cable）用于把 TTS 送进 QQ 麦克风；
- FFmpeg（TTS 转码）。

```powershell
python -m pip install -r requirements.txt
```

## 配置

AstrBot 的插件数据目录是 `data/plugin_data/astrbot_plugin_qq_call`。不要把配置、录音或 pid 放进插件代码目录。

1. 建数据目录并复制模板：

```powershell
$data = "data/plugin_data/astrbot_plugin_qq_call"
New-Item -ItemType Directory -Force $data | Out-Null
Copy-Item config.example.json "$data/config.json"
```

2. 编辑 `data/plugin_data/astrbot_plugin_qq_call/config.json`：
   - `llm`：OpenAI 兼容接口地址、key、模型；
   - `tts.provider`：`edge_tts` / `breeze_online` / `breeze_local`；
   - `audio`：扬声器、虚拟声卡、麦克风设备名；
   - `outgoing_call.contact_title`：对方聊天窗口的标题（QQ 备注）；
   - `outgoing_call.button_offset_from_right` / `button_offset_y`：电话按钮相对窗口右上角的位置，默认值适配常见 QQ NT；
   - `data`：录音/TTS 输出目录和滚动清理（默认 `retention_hours=24`、`max_files=200`）；
   - 自动接听默认关闭；开启时 `auto_answer.allowed_callers` 必须非空。

3. 在 AstrBot 插件配置里填：
   - `owner_qq`：唯一允许触发的主人 QQ；
   - `python_exe`：运行本插件的 Python；
   - `config_file`：默认 `config.json`，相对于 `data/plugin_data/astrbot_plugin_qq_call`；
   - `qq_call_dir` 留空即使用插件自身目录；
   - `agent_args` 留空；配置文件由插件自动用 `--config` 传入。

4. 在 QQ 设置里把麦克风明确选成虚拟声卡（如 `VB-Audio Cable A`），否则对方听不到 TTS。

## 使用

主人在 QQ 私聊里说：

> 给我打个电话

插件会：

```text
检查 owner_qq
→ 启动 call_agent.py
→ place_call.py 临时前置 QQ 聊天窗口并点电话
→ 手机响，接听
→ call_agent.py 在通话里回复
```

先 dry-run 确认坐标（在数据目录里执行）：

```powershell
$cfg = "data/plugin_data/astrbot_plugin_qq_call/config.json"
python place_call.py --config $cfg
```

确认后再真的拨：

```powershell
python place_call.py --config $cfg --call
```

## 三种 TTS

- `edge_tts`：默认，通用中文女声，无需 key；
- `breeze_online`：填自己的 Breeze key / voice_id，按字消耗积分；
- `breeze_local`：本地 Breeze HTTP 适配器，需自行安装模型。

仓库不附带任何 voice_id、声音样本、人格、QQ 号或 key。

## 隐私

- 配置、录音、日志都在本地；
- 录音和 TTS 输出写在 `data/plugin_data/astrbot_plugin_qq_call`，按 `data.retention_hours` 和 `data.max_files` 自动清理；
- 不要把自己的 `config.json`、QQ 号、key 提交到仓库；
- 插件不包含任何特定角色的人格或声音。

## 限制

- 仅 Windows，依赖桌面 QQ 和 UI 自动化；
- 只支持私聊语音，不支持群语音、视频、主动群呼；
- 轮次式对话，暂不支持随时打断；
- 第三方协议/UI 自动化有账号风控风险，请只用于自己的小号；
- 只对明确同意的人拨号，不要骚扰或群呼。

## License

MIT
