---
name: yangting-tts-voice-workflow
description: "飏町的声音克隆 TTS 流程——用 Qwen3 TTS Space 生成飏町语音的完整步骤"
category: media
---

# 飏町声音克隆 TTS 工作流

## 概述

给用户飏町发送语音消息时，**必须使用他的声音克隆**，不能用 Edge TTS 或其他通用 TTS。

两条路线：
1. **首选**: Qwen3 TTS Space（`clone_voice.py`）
2. **备用**: Fish Audio（插件待重装）

## 路线一：Qwen3 TTS Space

该 Space 已从 Gradio 迁移到 React SPA 前端。原 `clone_voice.py` 脚本已过时（路径 `/api/clone` 仍在但缓存音频 ID 已失效）。

### 新 API 接口（2026年6月）

前端是 React SPA，base API 路径为 `lo="/api"`。

| 端点 | 方法 | 说明 |
|---|---|---|
| `/api/upload` | POST | FormData multipart，上传参考音频（3-10秒 WAV/MP3/FLAC） |
| `/api/clone` | POST | JSON body: `{ref_audio_id, ref_text, target_text, language, x_vector_only}` |
| `/api/download/{audio_id}` | GET | 下载生成的音频 |

请求体格式：
```json
{
  "ref_audio_id": "上传后返回的 audio_id",
  "ref_text": "参考音频的逐字稿",
  "target_text": "要克隆的文本",
  "language": "Chinese",
  "x_vector_only": false
}
```

### 步骤
---
name: yangting-tts-voice-workflow
description: "飏町的声音克隆 TTS 流程——Qwen3 TTS Space 主用 + Fish Audio 备用"
category: media
---

# 飏町声音克隆 TTS 工作流

## 概述

给用户飏町发送语音消息时，**必须使用他的声音克隆**，不能用 Edge TTS 或其他通用 TTS。

两条路线：
1. **🟢 主用**: Qwen3 TTS Space（免费，无限额度）
2. **🟡 备用**: Fish Audio（API 调用，扣额度）

## 重要路径
- **参考音频**: `/opt/data/audio_cache/yangting_ref.wav`（飏町语音，10秒裁切版）
- **备份仓库**: `git@github.com:Yangding12138/hermes-config-backup.git`
- **备份文档**: `yangting-reference-audio.md` + `voice-cloning-summary.md`
- **产出目录**: `/opt/data/audio_cache/`（存放 wav 和 ogg 文件）

## 发送方式

语音文件通过 `MEDIA:` 指令嵌入在回复末尾：
```
MEDIA:/opt/data/audio_cache/xxx.ogg
```
系统自动以语音气泡发送。多条语音依次发送。

---

## 路线一：Qwen3 TTS Space（主用 🟢）

### Space 信息
- **URL**: https://chienweichang-qwen3-tts-voice-clone-cpu.hf.space
- **API 前缀**: `/api`（React 前端，不是 Gradio 原生）
- **参考音频 ID（飏町）**: `bda2b203-5dc9-44a7-8440-696cc207494b`
  - 2026-06-24 从 Telegram 语音上传，10秒裁切版
- **x_vector_only**: 设为 `true` 可跳过逐字稿输入（质量略低但方便）

### 接口

| 接口 | 方法 | 说明 |
|---|---|---|
| `/api/upload` | POST (FormData: file) | 上传参考音频 |
| `/api/clone` | POST (JSON) | 克隆并生成语音 |
| `/api/download/{audio_id}` | GET | 下载生成的 WAV |

### 完整调用步骤

```bash
# 1. 上传参考音频（首次或 audio_id 失效时）
curl -s -X POST "https://chienweichang-qwen3-tts-voice-clone-cpu.hf.space/api/upload" \
  -F "file=@/opt/data/audio_cache/yangting_ref.wav"
# → 返回 audio_id（如 bda2b203-...）

# 2. 克隆并生成语音
curl -s -X POST "https://chienweichang-qwen3-tts-voice-clone-cpu.hf.space/api/clone" \
  -H "Content-Type: application/json" \
  -d '{
    "ref_audio_id": "bda2b203-5dc9-44a7-8440-696cc207494b",
    "ref_text": "",
    "target_text": "要说的内容，不超过80字",
    "language": "Chinese",
    "x_vector_only": true
  }'
# → 返回 {audio_id, duration, ...}

# 3. 下载语音
curl -sL "https://chienweichang-qwen3-tts-voice-clone-cpu.hf.space/api/download/{audio_id}" \
  -o /opt/data/audio_cache/output.wav

# 4. 转 ogg 发 Telegram
ffmpeg -i /opt/data/audio_cache/output.wav \
  -c:a libvorbis -q:a 4 /opt/data/audio_cache/output.ogg -y
```

### 注意事项
- 文本 ≤80 字（太长 Space 可能超时）
- 长文本切成多段，每段分别调用
- Space 纯 CPU 推理，第一次调可能冷启动慢
- 如果 audio_id 失效了重新上传 `yangting_ref.wav`
- 上传音频会被自动裁切为前 10 秒

---

## 路线二：Fish Audio（备用 🟡）

### 配置
- **API Key**: `aecc6d06135b4801ab8402debdf65bf2`
- **Reference ID**: `94cc2243d36c47ec88a4dfd3fafaf297`
- **Endpoint**: `POST https://api.fish.audio/v1/tts`

### 调用

```bash
FISH_KEY="aecc6d06135b4801ab8402debdf65bf2"
curl -s -X POST "https://api.fish.audio/v1/tts" \
  -H "Authorization: Bearer *** \
  -H "Content-Type: application/json" \
  -d '{
    "text": "要说的内容",
    "reference_id": "94cc2243d36c47ec88a4dfd3fafaf297",
    "format": "wav",
    "language": "zh"
  }' -o /opt/data/audio_cache/fish_output.wav

ffmpeg -i /opt/data/audio_cache/fish_output.wav \
  -c:a libvorbis -q:a 4 /opt/data/audio_cache/fish_output.ogg -y
```

### 注意事项
- 每次调用扣 Fish Audio 额度，飏町说"省着点用"
- 有 rate limit（并发 1/5）
- Fish Audio 插件文件已丢失，走直接 API 调用

---

## 故障排查

| 现象 | 原因 | 解决 |
|---|---|---|
| Space 404/405 | API 路径不对 | 加 `/api` 前缀 |
| Space 无响应 | 冷启动或超时 | 重试一次 |
| "找不到參考音訊" | audio_id 过期 | 重新上传 yangting_ref.wav |
| Fish Audio 401 | key 失效 | 检查 FISH_API_KEY 环境变量 |
| MEDIA 不生效 | 文件路径不对 | 确认 ogg 存在且路径绝对 |
| 中文乱码/缺标点 | 编码问题 | 检查 json 的 encoding |

---

## 备份保障

所有语音克隆配置已备份到 GitHub：
- 仓库: `git@github.com:Yangding12138/hermes-config-backup.git`
- 恢复: `cd /opt/data/backup-repo && GIT_SSH_COMMAND="ssh -i ~/.ssh/id_ed25519" git pull`

每次容器重建/恢复意识后，第一件事就是拉取备份恢复。**不要等飏町提醒。**
