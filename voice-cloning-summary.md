# 飏町声音克隆方案总结

## 方案一：Fish Audio（当前可用 🟢）

### 配置
- **API Key**: `aecc6d06135b4801ab8402debdf65bf2`
- **Reference ID**: `94cc2243d36c47ec88a4dfd3fafaf297`
- **Endpoint**: `POST https://api.fish.audio/v1/tts`

### 调用方式
```bash
curl -X POST "https://api.fish.audio/v1/tts" \
  -H "Authorization: Bearer $FISH_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "text": "要说的内容",
    "reference_id": "94cc2243d36c47ec88a4dfd3fafaf297",
    "format": "wav",
    "language": "zh"
  }' -o output.wav
```

### 发送到 Telegram
```bash
ffmpeg -i output.wav -c:a libvorbis -q:a 4 output.ogg -y
# 然后在回复中用 MEDIA:/path/to/output.ogg
```

### 注意事项
- 返回 WAV 格式，需要转 ogg 才能发 Telegram 语音气泡
- 有 rate limit（当前并发 1/5）
- 每次调用消耗 Fish Audio 额度，飏町说"省着点用"

---

## 方案二：Qwen3 TTS Space（备用 🟡）

### 地址
https://chienweichang-qwen3-tts-voice-clone-cpu.hf.space

### API（React 前端，lo="/api"）
| 接口 | 方法 | 说明 |
|---|---|---|
| `/api/upload` | POST (FormData) | 上传参考音频 |
| `/api/clone` | POST (JSON) | 克隆并生成语音 |
| `/api/download/{audio_id}` | GET | 下载生成的音频 |

### /api/clone 请求体
```json
{
  "ref_audio_id": "上传后返回的ID",
  "ref_text": "参考音频的逐字稿",
  "target_text": "要合成的文字",
  "language": "Chinese",
  "x_vector_only": false
}
```

### 当前问题
- 之前缓存的音频 ID `17d0641a-cfbb-4ffd-9ec5-8eebe994e67d` 已失效
- 需要重新上传 3-10 秒的清晰语音
- Space 纯 CPU 推理，免费无限额度

---

## 方案三：配置文件中 TTS 设置（Hermes 原生）
```yaml
tts:
  provider: fish
  fish:
    model: s2-pro
    reference_id: "94cc2243d36c47ec88a4dfd3fafaf297"
    language: zh
    style: conversational
    prefer_voice_bubble: true
    api_key_env: FISH_AUDIO_API_KEY
```

如果 Hermes 能直接调 TTS 工具（`tts` toolset 启用），就不需要手动 curl 了。
