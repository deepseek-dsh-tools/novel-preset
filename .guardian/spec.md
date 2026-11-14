# novel-preset · 交付规格

## 目标

注册一个 DSH agent preset「写小说模式」（id 为 novel，order 5）：保留文件读写、搜索、终端、todo、web、present 与压缩链，**去掉**项目说明、技能目录、子智能体与计划模式。

谁在什么场景下用它：写长篇正文、需要把上下文窗口尽量留给文本的会话。

用它之后什么变了：切到这个 preset 后系统提示词更短、工具面更小，同样的窗口能放更多正文。

## 边界

- **纯声明式，零 js**：本包只有 cordis.patch.yml 与 package.json。不许加 lib 下的 js 或任何运行时逻辑。
- 不改宿主 preset 定义（dsh-agent-preset），只在自己的 patch 里 insert 一条。
- 不再依赖任何外部服务：2026-10-30 已删掉原来的 inject 行 — 该服务无人提供时 preset 挂不起来，写小说模式会整体消失且屏幕无提示。
- 压缩由 dsh-context-forge（host 层）接管，preset 内只保留 compaction 组。
- 不碰 profile 的其它 preset；id 必须是 preset-novel 且唯一。

## 验收

- [ ] 结构自检：`node -e` 读 cordis.patch.yml，断言同时出现 `- id: preset-novel`、`id: novel`、`name: 写小说模式`，且不含行首的 inject 行 → 退出码 0
- [ ] 真机：DSH 启动后 agent preset 列表里出现「写小说模式」，选中它新建会话时系统提示词不含「子智能体 / 技能目录 / 计划模式」三节
- [ ] 工具面切换：切到 novel 后 subagent 等工具不可见；切回默认 preset 后重新可见
- [ ] order 生效：preset 列表里「写小说模式」不抢到第一位（order 5，排在默认 preset 之后）
- [ ] 无 loader 告警：启动日志里没有与 preset-novel 相关的 failed entry 或 duplicate entry id

## 失败模式

- 又加回 inject 行 → preset 静默不出现，只有启动日志里一条 loader warn（屏幕无提示）
- plugins 列表里引用了不存在的插件 id → 该子项 failed，preset 本体可能仍挂上（症状：某个工具神秘消失）
- 与 profile 里同 id 条目冲突 → duplicate loader entry id，启动预检直接拦
- 回滚：cordis.patch.yml.bak-inject-20261030-201435 是上一版；改完重启 DSH

## 非目标

- 不做写作专用工具（大纲 / 角色卡 / 字数统计）
- 不做自动续写或风格策略
- 不做多语言（preset 名与描述就是中文，够用）
- 不改压缩参数（thresholdChars 8192 等沿用当前值）
