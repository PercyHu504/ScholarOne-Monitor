# S1-Monitor: ScholarOne 论文状态监控

自动登录 ScholarOne，监控指定论文投稿状态，每次检查后通过微信推送通知。

## ⚙️ Setup（首次使用）

1. 将 `config.example.json` 复制一份，重命名为 `config.json`
2. 编辑 `config.json`，填入你自己的信息：
   - ScholarOne 的期刊网址、账号、密码
   - 推送方式：参考 `config.example.json` 中的 notification 字段格式，
     accountId 和 to 参数从 OpenClaw 配置中获取}
3. 告诉 Claw："使用 scholarone-monitor skill 初始化配置"
4. Claw 会登录并让你选择需要监测的论文

## 🔒 隐私说明

本 skill 不会上传任何个人信息。
config.json 和 state.json 已被 .gitignore 排除在版本控制之外。
## 功能特点

- 🔐 自动登录 ScholarOne 系统
- 📊 抓取指定论文的投稿状态
- 🔍 大小写敏感的状态比对（精确匹配）
- 📱 每次检查后都推送微信通知（无论状态是否变化）
- 📝 完整的状态变化历史记录
- 🎯 **交互式配置向导**（自动列出论文供选择）

---

## 快速开始（交互式配置）

⚠️ **重要：当用户说"设置 ScholarOne 监控"或类似意图时，必须执行以下交互流程！**

### 第一步：收集 ScholarOne 登录信息

向用户询问：
```
请提供你的 ScholarOne 登录信息：

1. 期刊站点 URL（如：https://mc.manuscriptcentral.com/xxx）
2. 用户名
3. 密码

这些信息仅保存在本地，不会上传到任何服务器。
```

### 第二步：登录并获取论文列表

1. 使用浏览器打开用户提供的 URL
2. 使用用户提供的用户名和密码登录
3. 导航到 Author Dashboard
4. 抓取所有论文列表，格式化输出给用户：

```
📚 你在 ScholarOne 上有 X 篇论文：

#  | ID               | 标题                              | 状态
---|------------------|-----------------------------------|------------------
1  | IoT-12345-2024   | Your Paper Title Here...          | Under Review
2  | IoT-67890-2024   | Another Paper Title...            | Reject (revise)
3  | IoT-11111-2023   | Published Paper...                | Accept

请输入要监控的论文序号（可多选，如 "1" 或 "1,3"）：
```

### 第三步：选择监控频率

```
请选择监控频率：

1. 每 1 小时
2. 每 3 小时
3. 每 6 小时（推荐）
4. 每 12 小时
5. 每 24 小时
6. 自定义（输入小时数）

请输入选项编号：
```

### 第四步：确认并生成配置

```
📋 配置确认：

━━━━━━━━━━━━━━━━━━

🔐 ScholarOne 站点：https://mc.manuscriptcentral.com/iot
👤 用户名：your_username

📄 监控论文：
  • IoT-12345-2024 - Your Paper Title...

⏰ 监控频率：每 6 小时

📱 推送方式：微信

━━━━━━━━━━━━━━━━━━

确认创建配置？(y/n)
```

### 第五步：生成配置文件

用户确认后：
1. 创建 `config.json`
2. 创建 Cron 定时任务
3. 发送测试通知验证配置正确

---

## 手动配置（高级用户）

如果用户已有配置信息，可以直接手动配置：

### 1. 复制配置模板

```bash
cd ~/.qclaw/config/skills/s1-monitor
cp config.example.json config.json
```

### 2. 编辑 config.json

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
  "checkInterval": "6h",
  "notification": {
    "channel": "weixin",
    "accountId": "YOUR_WEIXIN_ACCOUNT_ID",
    "to": "YOUR_WEIXIN_OPENID@im.wechat"
  }
}
```

### 配置字段说明

| 字段 | 说明 | 示例 |
|------|------|------|
| `scholarOneUrl` | ScholarOne 期刊站点 URL | `https://mc.manuscriptcentral.com/iot` |
| `credentials.username` | ScholarOne 用户名 | `your_username` |
| `credentials.password` | ScholarOne 密码 | `your_password` |
| `monitoredManuscripts[].id` | 论文 ID | `IoT-12345-2024` |
| `monitoredManuscripts[].title` | 论文完整标题 | `Your Paper Title` |
| `monitoredManuscripts[].shortTitle` | 论文简称（用于通知） | `MyPaper` |
| `checkInterval` | 检查间隔 | `6h`（6小时） |
| `notification.channel` | 推送渠道 | `weixin` |
| `notification.accountId` | 微信账号 ID | 从 OpenClaw 配置获取 |
| `notification.to` | 微信 OpenID | `your_openid@im.wechat` |

---

## 工作流程

### 1. 登录 ScholarOne
- 使用浏览器打开配置的 `scholarOneUrl`
- 输入 `credentials.username` 和 `credentials.password` 登录
- 等待页面加载完成

### 2. 抓取论文状态
- 导航到 Author Dashboard
- 在论文列表中找到目标论文（按 ID 匹配）
- 提取当前状态文本

### 3. 状态比对（大小写敏感）

⚠️ **重要：状态比对区分大小写**

| 原状态 | 新状态 | 是否变化 |
|--------|--------|----------|
| `Under Review` | `Under Review` | ❌ 无变化 |
| `Under Review` | `Under review` | ✅ 有变化 |
| `Under Review` | `Pending Revision` | ✅ 有变化 |

使用 JavaScript 的严格相等比较：`currentStatus === historyStatus`

### 4. 更新历史记录
- 更新 `lastChecked` 时间戳
- **状态变化时**：追加新记录到 `history` 数组

### 5. 微信推送通知

⚠️ **每次检查后都推送通知，无论状态是否变化**

#### 状态有变化时：
```
🔔 ScholarOne 论文状态更新！

━━━━━━━━━━━━━━━━━━

📄 论文：[shortTitle]
🆔 ID：[manuscriptId]

📤 原状态：[原状态]
📥 新状态：[新状态]

━━━━━━━━━━━━━━━━━━

⏰ 检查时间：2026-03-25 18:32
```

#### 状态无变化时：
```
✅ ScholarOne 论文状态检查

━━━━━━━━━━━━━━━━━━

📄 论文：[shortTitle]
🆔 ID：[manuscriptId]

📊 当前状态：[当前状态]

━━━━━━━━━━━━━━━━━━

⏰ 检查时间：2026-03-25 18:32
📈 状态稳定，暂无变化
🔄 下次检查：6小时后
```

---

## 微信推送方法

```javascript
message({
  action: "send",
  channel: "weixin",
  accountId: config.notification.accountId,
  target: config.notification.to,
  message: "通知内容"
})
```

---

## 常见状态说明

| 状态 | 含义 |
|------|------|
| Under Review | 审稿中 |
| Pending Revision | 等待修改 |
| Reject (revise and resubmit) | 拒稿但可重投 |
| Accept | 已录用 |
| Published | 已发表 |

---

## 文件结构

```
s1-monitor/
├── SKILL.md              # 本文件
├── config.example.json   # 配置模板（提交到 Git）
├── config.json           # 实际配置（敏感信息，不提交）
├── history.json          # 状态历史记录（不提交）
├── check-status.sh       # 检查脚本
└── .gitignore            # Git 忽略文件
```

---

## 触发关键词

当用户说以下内容时，启动交互式配置：
- "设置 ScholarOne 监控"
- "配置论文状态监控"
- "帮我监控 ScholarOne 论文"
- "添加论文监控"

---

## 注意事项

1. **大小写敏感**：ScholarOne 的状态可能有大小写差异，需要精确匹配
2. **每次都推送**：保持用户知情，即使状态未变化也推送确认
3. **保留历史**：所有状态变化都记录在 history 数组中，便于追溯
4. **保护隐私**：`config.json` 已加入 `.gitignore`，不会被提交
5. **交互式配置**：优先使用交互流程，让用户选择论文和频率

---

## License

MIT
