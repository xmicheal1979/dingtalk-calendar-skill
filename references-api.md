钉钉日历 API 参考

基础信息
基础 URL: https://api.dingtalk.com/v1.0
认证方式: Header 中携带 x-acs-dingtalk-access-token: {access_token}

---

1. 创建日程

接口: POST /calendar/users/{unionId}/calendars/primary/events

请求头:
x-acs-dingtalk-access-token: {access_token}
Content-Type: application/json
请求体:json
{
  "summary": "会议标题",
  "description": "会议描述",
  "start": {
    "dateTime": "2026-03-30T09:00:00+08:00",
    "timeZone": "Asia/Shanghai"
  },
  "end": {
    "dateTime": "2026-03-30T10:00:00+08:00",
    "timeZone": "Asia/Shanghai"
  },
  "location": {
    "displayName": "会议室A"
  },
  "attendees": [
    {
      "id": "{unionId}",
      "isOptional": false
    }
  ],
  "reminders": [
    {
      "method": "dingtalk",
      "minutes": "15"
    }
  ]
}
字段说明:
summary: 日程标题（必填）
description: 日程描述
start/end: 开始/结束时间（必填）
dateTime: ISO 8601 格式，带时区
timeZone: 时区，默认 Asia/Shanghai
location: 地点信息
attendees: 参与者列表（使用 unionId）
reminders: 提醒设置
method: 提醒方式（dingtalk）
minutes: 提前多少分钟提醒

响应:json
{
  "id": "event_xxx",
  "summary": "会议标题",
  "start": {...},
  "end": {...},
  "createTime": "2026-03-29T12:00:00.000Z",
  "updateTime": "2026-03-29T12:00:00.000Z"
}
---

2. 查询日程列表

接口: GET /calendar/users/{unionId}/calendars/primary/events

查询参数:
timeMin: 开始时间（ISO 8601 格式，如 2026-03-30T00:00:00%2B08:00）
timeMax: 结束时间
maxResults: 最大返回数量（默认 250）
pageToken: 分页令牌

响应:json
{
  "events": [
    {
      "id": "event_xxx",
      "summary": "会议标题",
      "start": {...},
      "end": {...}
    }
  ]
}
---

3. 获取日程详情

接口: GET /calendar/users/{unionId}/calendars/primary/events/{eventId}

响应: 单个日程对象的完整信息

---

4. 更新日程

接口: PUT /calendar/users/{unionId}/calendars/primary/events/{eventId}

请求体与创建日程相同，提供需要更新的字段。

---

5. 删除日程

接口: DELETE /calendar/users/{unionId}/calendars/primary/events/{eventId}

响应: 204 No Content

---

错误码

| 错误码 | 说明 |
|--------|------|
| 400 | 请求参数错误 |
| 401 | 未授权，token 无效或过期 |
| 403 | 无权限访问 |
| 404 | 日程不存在 |
| 429 | 请求过于频繁 |
| 500 | 服务器内部错误 |

---

所需应用权限
Calendar.Calendar.Write - 日历写入
Calendar.Event.Write - 日程事件写入
Calendar.Event.Read - 日程事件读取