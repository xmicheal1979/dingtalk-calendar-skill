---
name: dingtalk-calendar
description: 钉钉日历管理。当用户提到"钉钉日程"、"钉钉日历"、"创建日程"、"添加会议"、"查看日程"、"删除日程"、"dingtalk calendar"、"calendar event"时使用此技能。支持：创建日程（含标题/时间/地点/参与者）、查询日程列表、获取详情、更新日程、删除日程等全部日历类操作。
---

# 钉钉日历技能

负责钉钉日历（Schedule/Calendar）的所有操作。本文件为**策略指南**，仅包含决策逻辑和工作流程。完整 API 请求格式见文末「references/api.md 查阅索引」。

---

## 工作流程（每次执行前）

1. **读取配置** → 用一条 `grep -E` 命令一次性读取配置文件`~/.dingtalk-skills/config`, 所有所需配置键值
2. **仅收集缺失配置** → 若配置文件不存在或缺少某项，**一次性询问用户**所有缺失的值，不要逐条问
3. **持久化** → 将收集到的值写入 `~/.dingtalk-skills/config` 文件
4. **获取/复用 Token** → 有效期内复用缓存（缓存 7000 秒，约 2 小时），避免重复请求；遇 401 重新获取
5. **执行操作** → 凡是包含变量替换、管道或多行逻辑的命令，`/tmp/<task>.sh` 再 `bash /tmp/<task>.sh` 执行

> 凭证禁止在输出中完整打印，确认时仅显示前 4 位 + `****`

### 所需配置

复用 dingtalk-todo 的相同配置：

| 配置键 | 说明 | 如何获取 |
|---|---|---|
| `DINGTALK_APP_KEY` | 应用 AppKey | 钉钉开放平台 → 应用管理 → 凭证信息 |
| `DINGTALK_APP_SECRET` | 应用 AppSecret | 同上 |
| `DINGTALK_MY_USER_ID` | 当前用户的企业员工 ID（userId） | 管理后台 → 通讯录 → 成员管理 |
| `DINGTALK_MY_OPERATOR_ID` | 当前用户的 unionId | 首次由脚本自动通过 userId 转换获取并写入 |

### 执行脚本模板

```bash
#!/bin/bash
set -e
CONFIG=~/.dingtalk-skills/config
APP_KEY=$(grep '^DINGTALK_APP_KEY=' "$CONFIG" | cut -d= -f2-)
APP_SECRET=$(grep '^DINGTALK_APP_SECRET=' "$CONFIG" | cut -d= -f2-)

# 新版 Token 缓存（用于日历 API）
CACHED_TOKEN=$(grep '^DINGTALK_ACCESS_TOKEN=' "$CONFIG" 2>/dev/null | cut -d= -f2-)
TOKEN_EXPIRY=$(grep '^DINGTALK_TOKEN_EXPIRY=' "$CONFIG" 2>/dev/null | cut -d= -f2-)
NOW=$(date +%s)
if [ -n "$CACHED_TOKEN" ] && [ -n "$TOKEN_EXPIRY" ] && [ "$NOW" -lt "$TOKEN_EXPIRY" ]; then
  TOKEN=$CACHED_TOKEN
else
  RESP=$(curl -s -X POST https://api.dingtalk.com/v1.0/oauth2/accessToken \
    -H 'Content-Type: application/json' \
    -d "{\"appKey\":\"$APP_KEY\",\"appSecret\":\"$APP_SECRET\"}")
  TOKEN=$(echo "$RESP" | grep -o '"accessToken":"[^"]*"' | cut -d'"' -f4)
  sed -i '/^DINGTALK_ACCESS_TOKEN=/d;/^DINGTALK_TOKEN_EXPIRY=/d' "$CONFIG"
  echo "DINGTALK_ACCESS_TOKEN=$TOKEN" >> "$CONFIG"
  echo "DINGTALK_TOKEN_EXPIRY=$((NOW + 7000))" >> "$CONFIG"
fi

# unionId：优先从配置读取
UNION_ID=$(grep '^DINGTALK_MY_OPERATOR_ID=' "$CONFIG" 2>/dev/null | cut -d= -f2-)

# 在此追加具体 API 调用
echo "Token: ${TOKEN:0:4}****"
echo "UnionId: ${UNION_ID:0:4}****"