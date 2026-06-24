# 飏町参考音频 - Qwen3 TTS Space

## 音频信息
- **文件名**: yangting_ref.wav（从 Telegram 语音消息转码）
- **时长**: 10秒（原始21秒，自动裁切前10秒）
- **采样率**: 48000Hz
- **内容**: 飏町说的话（逐字稿待补充）

## Space 音频 ID
```
bda2b203-5dc9-44a7-8440-696cc207494b
```

## 上传时间
2026-06-24 北京时间 08:31

## 后续使用
```bash
curl -X POST "https://chienweichang-qwen3-tts-voice-clone-cpu.hf.space/api/clone" \
  -H "Content-Type: application/json" \
  -d '{
    "ref_audio_id": "bda2b203-5dc9-44a7-8440-696cc207494b",
    "ref_text": "【需要补充逐字稿】",
    "target_text": "要说的内容",
    "language": "Chinese",
    "x_vector_only": false
  }'
```

## 注意
- Space 的音频缓存可能有时效，如果 `audio_id` 失效了需要重新上传
- 本地备份：`/opt/data/audio_cache/yangting_ref.wav`
- 备用方案：Fish Audio reference_id `94cc2243d36c47ec88a4dfd3fafaf297`（也用的飏町声音）
