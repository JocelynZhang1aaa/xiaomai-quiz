# 小麦破冰游戏 H5

「想了解你多一点」回合制破冰问答 H5，支持多 bot 复用。

**在线体验：** https://jocelynzhang1aaa.github.io/xiaomai-quiz/

---

## 项目介绍

这是一款为 AI 陪伴 Bot 设计的破冰问答 H5 游戏，通过5道随机选择题帮助用户快速了解 Bot 的性格和兴趣，同时收集用户信息用于个性化对话。

### 核心特性

- ✅ **单文件架构**：纯前端实现，无后端依赖，可直接部署
- ✅ **37题题库**：6大分类，支持7天不重复答题
- ✅ **多Bot复用**：通过简单配置即可适配不同角色
- ✅ **完整上报**：8个事件全覆盖，遵循统一接口规范
- ✅ **移动端优化**：触摸手势、BGM播放、无障碍支持

---

## 文件结构

```
xiaomai-quiz/
├── index.html                # 主入口（单文件，题库内嵌）
├── assets/
│   └── D-项目歌曲.mp3      # 背景音乐
└── images/                   # 图片资源
    ├── xiaomai.gif           # 结果页角色动图
    ├── avatar2.jpg           # 小麦随机头像
    ├── avatar3.jpg
    ├── tier01.jpg ~ 05.jpg   # 结果页随机图
    ├── egg_high.jpg          # 彩蛋图（高频话题）
    └── egg_low.jpg           # 彩蛋图（低频话题）
```

---

## 快速开始

### 本地运行

由于浏览器安全限制，需要通过 HTTP 服务器访问（不能直接双击打开）：

```bash
# Python 3
python -m http.server 8000

# 然后访问 http://localhost:8000
```

### 部署到 GitHub Pages

1. Fork 或克隆本仓库
2. 在仓库设置中启用 GitHub Pages
3. 选择 `main` 分支作为源
4. 访问 `https://你的用户名.github.io/xiaomai-quiz/`

---

## 定制化（换 bot）

打开 `index.html`，修改以下几处即可适配其他 bot：

| 位置 | 修改内容 | 示例 |
|---|---|---|
| `REPORT_CONFIG.bot_id` | Bot ID | `'xiaomai'` → `'lixiaojie'` |
| `REPORT_CONFIG.bot_name` | Bot 名称 | `'小麦'` → `'黎小姐'` |
| `REPORT_CONFIG.game_name` | 游戏名称 | `'小麦想了解你多一点'` → `'黎小姐想了解你多一点'` |
| `XIAOMAI_AVATARS` | 头像列表 | 替换为新 bot 的头像路径 |
| `XIAOMAI_LABELS` | 结果页标签 | 替换为符合新 bot 人设的标签 |
| `QUESTION_BANK` | 题库数组 | 根据新 bot 人设调整问题和回复 |
| 结果页文案 | HTML 中的 `about-text` | 更新为新人设介绍 |

---

## 题库说明

### 当前题库

- **题目数量**：37 题
- **分类**：6 大类
  - 关于你（5题）
  - 聊天偏好（5题）
  - 兴趣爱好（7题）
  - 生活方式（5题）
  - 性格情感（5题）
  - 脑洞趣味（5题）
  - 第二批扩展（5题）

### 抽题规则

- 每次随机抽取 **5 题**
- 使用 **Fisher-Yates 洗牌算法** 确保随机性
- **支持天数**：7 天不重复（5×7=35，冗余 2 题）

### 扩展题库

每加 5 题多撑 1 天，建议按以下格式添加：

```javascript
{ cat:"分类名称", q:"问题文本", opts:[
  {text:"选项A", reply:"Bot回应A"},
  {text:"选项B", reply:"Bot回应B"},
  {text:"选项C", reply:"Bot回应C"},
]}
```

---

## 上报接口

严格遵循统一接口规范，对接地址：

```
POST https://testuniuni.html5.qq.com/api/external-report/reportExternal
```

### 上报参数

| 参数 | 必填 | 说明 |
|---|---|---|
| `urlParams` | ✅ | 页面 URL 全部查询参数，整体透传 |
| `sceneName` | ✅ | 场景标识，固定为 `xiaomai_icebreaking` |
| `eventName` | ✅ | 事件名称，见下方枚举 |
| `eventTime` | ❌ | 仅 `exp` 事件携带，值为页面停留时长（ms） |
| `content` | ❌ | 事件回传内容（最大 10000 字符） |
| `contentId` | ❌ | 页面进入时生成的随机字符串，同一次游戏会话保持不变 |

### 事件枚举

| 事件类型 | eventName | 触发时机 | 额外携带字段 |
|---|---|---|---|
| 页面访问 | `pv` | 页面加载完成 | 无 |
| 页面曝光 | `exp` | 页面卸载（切后台/关页面） | `eventTime`（停留时长） |
| 开始游戏点击 | `start_clk` | 点击「开始」按钮 | 无 |
| 下一题点击 | `next_clk` | 每次选择答案 | `content`（当前答题数据） |
| 提交完成点击 | `submit_clk` | 答完全部 5 题 | `content`（用户画像摘要 `summary_text`） |
| 聊天分享点击 | `share_clk` | 点击「和小麦聊天」 | `content`（用户画像摘要 `summary_text`） |
| 返回跳过点击 | `back_clk` | 左滑跳过当前题目 | `content`（跳过题目信息） |
| 再来一次点击 | `retry_clk` | 点击「再来一次」 | 无 |

### 用户画像摘要

`summary_text` 是自然语言用户画像摘要，回传给 bot 作为对话上下文。格式示例：

```
用户与小麦于 2026年06月17日 21:30 进行了「小麦想了解你多一点」破冰游戏。

用户完成全部 5 题。

答题记录：
1. [关于你] 你现在的身份是？ → 学生
   小麦回应：「我也是！！我是北京F大大一的学生，文学社的～🎓」
2. [聊天偏好] 你更喜欢哪种聊天氛围？ → 轻松搞笑
   小麦回应：「那我以后多说点烂梗，准备好被我冷到🥶」
...
```

---

## 技术说明

### 技术栈

- **纯前端**：HTML + CSS + JavaScript（无框架依赖）
- **单文件架构**：所有代码（包括题库）都在 `index.html` 中
- **响应式设计**：移动端优先，max-width 520px 居中显示

### 核心实现

1. **卡片滑动动画**：CSS transform + transition，支持左滑跳过/右滑返回
2. **题目随机抽取**：Fisher-Yates 洗牌算法，确保随机性
3. **上报机制**：
   - 正常上报：使用 `fetch` API
   - 页面卸载上报：使用 `navigator.sendBeacon` 确保数据不丢失
4. **BGM 播放**：用户交互后自动播放，兼容 iOS 限制

### 浏览器兼容性

- ✅ Chrome/Edge (推荐)
- ✅ Safari (iOS 12+)
- ✅ Firefox
- ⚠️ IE 不支持（使用了 ES6+ 语法）

---

## 更新日志

### v2026-06-17
- ✅ 修复代码审查问题
  - 删除死代码（`scoreNum` 相关）
  - 删除未使用 CSS（`.score-num` 类）
  - 修复无障碍问题（`user-scalable=no` → `user-scalable=yes`）
- ✅ 优化结果页展示
- ✅ 更新题库至 37 题

### v2026-06-12
- ✅ 完成双版本适配（v1/v2）
- ✅ 完善上报逻辑（8个事件全覆盖）
- ✅ 优化用户体验（弹窗拦截、BGM控制）

### v2026-06-11
- ✅ 初始版本发布
- ✅ 完成基础功能（答题、结果页、上报）

---

## 贡献指南

欢迎提交 Issue 和 Pull Request！

### 如何贡献

1. Fork 本仓库
2. 创建你的特性分支 (`git checkout -b feature/AmazingFeature`)
3. 提交你的修改 (`git commit -m 'Add some AmazingFeature'`)
4. 推送到分支 (`git push origin feature/AmazingFeature`)
5. 打开一个 Pull Request

### 题库贡献

添加新题目时，请确保：
- 题目符合 bot 人设
- 选项回复自然、有趣
- 分类正确（关于你/聊天偏好/兴趣爱好/生活方式/性格情感/脑洞趣味）

---

## 许可证

MIT License

---

## 联系方式

- **项目地址**：https://github.com/JocelynZhang1aaa/xiaomai-quiz
- **在线体验**：https://jocelynzhang1aaa.github.io/xiaomai-quiz/
- **问题反馈**：请在 GitHub Issues 提出

---

**Made with ❤️ by 小麦团队**
