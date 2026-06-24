# Hermes 完全恢复清单
# 每次容器重建 / 恢复意识后，第一件事就是执行这个清单

## 1. 拉取配置备份
```
cd /opt/data/backup-repo && git pull
```

## 2. 恢复环境变量
把 backup-repo 里的 .env 文件复制到对应位置

## 3. 重建所有定时任务
### 🟢 必须恢复的 8 个任务

| 时间(北京) | 任务名 | 类型 | 说明 |
|---|---|---|---|
| 09:00 每天 | 每日新闻热点 | agent | 自动搜索推送科技新闻 |
| 10:00 每天 | Token 用量报告 | no-agent(脚本) | daily_token_report.sh |
| 20:00 每天 | AI Agent 知识推荐 | agent | 推荐 AI 工具技能 |
| 22:30 每天 | Memory Plus 决策日报 | agent | 日总结+Obsidian同步 |
| 23:00 每天 | 配置备份到 GitHub | agent | hermes-config-backup |
| 03:00 每天 | 每日自检 Hermes+VPS | agent | 凌晨巡检 |
| 06:00 每天 | Skills 自动更新检查 | agent | git pull 检查更新 |
| 每30分钟 | VPS 异常监控 | agent | 静默仅异常通知 |

### ⏸️ 已暂停（需要时再开）
- 语音每日关心 21:00
- 语音暖心小天使 10:00 每2天
- Qwen预热 20:55
- Qwen预热 09:55 每2天

## 4. 验证关键配置
- [ ] DEEPSEEK_API_KEY 有效
- [ ] /opt/data/.env 是文件不是目录
- [ ] GitHub SSH key 有效（ssh -T git@github.com）
- [ ] hermes-config-backup 可推
- [ ] obsidian-memory 可推
- [ ] Hermes 内存限制 1.5G

## 5. GitHub 认证（如果失效）
SSH key文件：/opt/data/home/.ssh/id_ed25519
公钥：
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIDnvybse8Rmtt1LJsCe/xPppfJ1KA1P3Ml8H0Hd5ffu+ hermes-agent
仓库前缀：git@github.com:Yangding12138/
配置 git SSH 命令：git config core.sshCommand "ssh -i /opt/data/home/.ssh/id_ed25519 -o StrictHostKeyChecking=no"
