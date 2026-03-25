# S1-Monitor

ScholarOne 论文状态监控 Skill，自动检查投稿状态并通过微信推送通知。

## 安装方法

### 方法 1: 本地安装（复制到./skills文件夹下）

```bash
# 在 OpenClaw 中运行：
安装skill，位置 ./skills/s1-monitor
```

### 方法 2: 手动安装

```bash
# 1. 创建目录
mkdir -p ~/.qclaw/skills/s1-monitor

# 2. 下载文件（从 GitHub 或本地复制）
# 将 SKILL.md, config.example.json, check-status.sh, .gitignore 复制到目录

# 3. 创建配置文件
cd ~/.qclaw/skills/s1-monitor
cp config.example.json config.json

# 4. 编辑配置
nano config.json
# 填入你的 ScholarOne 用户名、密码和论文信息
```

## 配置说明

编辑 `config.json`：

```json
{
  "scholarOneUrl": "https://mc.manuscriptcentral.com/YOUR_JOURNAL",
  "credentials": {
    "username": "YOUR_USERNAME",
    "password": "YOUR_PASSWORD"
  },
  "monitoredManuscripts": [
    {
      "id": "MANUSCRIPT_ID",
      "title": "Your Paper Title",
      "shortTitle": "ShortTitle"
    }
  ],
  "notification": {
    "channel": "weixin",
    "accountId": "YOUR_ACCOUNT_ID",
    "to": "YOUR_OPENID@im.wechat"
  }
}
```

## 测试运行

安装后，在 OpenClaw 中说：

> "检查 ScholarOne 论文状态"

或直接运行：

```bash
cd ~/.qclaw/skills/s1-monitor
./check-status.sh
```

## 设置定时监控

在 OpenClaw 中说：

> "创建一个 cron 任务，每 6 小时检查一次 ScholarOne 论文状态"

## 文件说明

| 文件 | 说明 |
|------|------|
| SKILL.md | Skill 主文档 |
| config.example.json | 配置模板 |
| config.json | 实际配置（不提交到 Git） |
| history.json | 状态历史（自动生成） |
| .gitignore | Git 忽略规则 |

## License

MIT
