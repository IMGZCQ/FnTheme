# Fndesk 飞牛主题库接入规范

本文档说明如何制作一个符合规范的 `theme.json` 文件，在 Fndesk 主题库一键切换使用。
（目前涵盖飞牛：登录背景、登录LOGO、桌面背景、设备LOGO、Favicon等）

---
<img width="730" height="408" alt="image" src="https://github.com/user-attachments/assets/6c6735f7-255f-4f1e-853e-0c12a797a9cc" />
<img width="670" height="537" alt="image" src="https://github.com/user-attachments/assets/7a7c8474-5316-4a77-b433-d21f015061ef" />
<img width="664" height="389" alt="image" src="https://github.com/user-attachments/assets/b28f5352-81dc-48f9-acf7-c7376dff7100" />

---

## 一、`theme.json` 文件结构

### 完整示例

```json
{
  "_meta": {
    "name": "XXX 主题库",
    "desc": "高品质主题集合，涵盖自然、动漫、科技等多种风格",
    "from": "制作者 / 提供者 名字"
  },
  "themes": [
    {
      "ID": 1,
      "标题": "明日方舟终末地 - 萤石",
      "t0": "http://123.com/萤石/thumbnail.webp",
      "t1": "http://123.com/萤石/loginLogo.png",
      "t2": "http://123.com/萤石/loginBg.mp4",
      "t3": "http://123.com/萤石/deviceLogo.png",
      "t4": "http://123.com/萤石/wallpaper.mp4",
      "t5": "http://123.com/萤石/favicon.png"
    },
    {
      "ID": 2,
      "标题": "开心肥牛",
      "t0": "http://123.com/开心肥牛/thumbnail.webp",
      "t1": "http://123.com/开心肥牛/loginLogo.png",
      "t2": "http://123.com/开心肥牛/loginBg.mp4",
      "t3": "http://123.com/开心肥牛/deviceLogo.png",
      "t4": "http://123.com/开心肥牛/wallpaper.mp4",
      "t5": "http://123.com/开心肥牛/favicon.png"
    }
  ]
}
```

### 主题对象字段

| 字段 | 类型 | 必填 | 说明 |
|------|------|:----:|------|
| `ID` | 数字 | ✅ | 主题唯一标识 |
| `标题` | 字符串 | ✅ | 主题名称，显示在卡片按钮上 |
| `t0` | URL | ⭕ | 缩略图（支持常见图片格式，长边 240px） |
| `t1` | URL | ✅ | 登录 Logo（支持常见图片格式，建议小于300*130） |
| `t2` | URL | ✅ | 登录背景图（支持常见图片格式或 MP4 视频） |
| `t3` | URL | ✅ | 设备 Logo（支持常见图片格式，建议128*128） |
| `t4` | URL | ✅ | 桌面壁纸（支持常见图片格式或 MP4 视频） |
| `t5` | URL | ⭕ | 网页 Favicon （建议128*128） |

### `_meta` 字段（可选）

| 字段 | 类型 | 必填 | 说明 |
|------|------|:----:|------|
| `name` | 字符串 | ⭕ | 主题库名称，显示在工具的下拉列表中 |
| `desc` | 字符串 | ⭕ | 主题库介绍 |
| `from` | 字符串 | ⭕ | 提供者信息 |

> 💡 `_meta` 可整体省略。省略时下拉列表中显示 "未知主题库"。

---

## 二、发布方式

把 `theme.json` 放到可公开访问的 URL 后，有两种方式让用户使用：

### 方式 1：用户自行添加

用户进入工具 → **主题库来源管理 → 编辑 → 添加**，粘贴你的 URL，工具会自动从 `_meta` 读取主题库的名称、描述、作者信息。

### 方式 2：提交到 Fndesk 作者收录

把你的 `theme.json` 扔给 **米恋泥** 即可，审核通过后会加入主题库默认列表。

---

## 三、注意事项

| 项 | 说明 |
|----|------|
| 🎬 动态视频 | 建议使用 **H.264 编码的 MP4**（不支持 H.265），请自行权衡视频体积与体验 |
| 🔧 自定义项 | 目前 `t1` ~ `t5` 是固定字段，未来可能扩展更多 |
| 🌱 共创主题生态 | 欢迎大家参与制作，让飞牛更有趣！ |

---
<img width="1029" height="794" alt="image" src="https://github.com/user-attachments/assets/e0f19d6e-3650-407f-9a9b-19c46b525949" />
---

> 最后更新：2026-08-11
