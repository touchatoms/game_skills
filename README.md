# game-ai-skills

游戏公司 AI Skills 仓库模板。

这个仓库用于沉淀团队常用的 AI 工作流，让 Codex、Claude Code、Cursor Agent 等 AI 编程工具在分析需求、生成测试用例、检查配置、审查代码、发版检查时，使用统一标准。

## 目录结构

```text
game-ai-skills/
├── game_requirement_analyzer
├── game_testcase_generator
├── game_activity_analyzer
├── game_config_checker
├── code_review_cocos2d_lua
├── code_review_unity
├── code_review_cocos_creator
├── release_check
├── game_performance_check
└── README.md
```

## 当前 Skills

| Skill | 作用 |
|---|---|
| game_requirement_analyzer | 游戏需求分析，拆解客户端、服务端、配置、接口、风险点 |
| game_testcase_generator | 根据需求生成测试用例和开发自测清单 |
| game_activity_analyzer | 捕鱼活动专项需求分析，重点关注强发库、库存、掉落、房间和炮倍 |
| game_config_checker | 捕鱼配置检查，检查奖励、权重、库存、强发库参数 |
| code_review_cocos2d_lua | Lua / Cocos2d-lua 代码审查 |
| code_review_unity | Unity C# 代码审查 |
| code_review_cocos_creator | Cocos Creator / Cocos2d 代码审查 |
| release_check | 游戏发版前检查 |
| game_performance_check | 游戏性能检查 |

## 推荐用法

### 需求分析

```text
使用 game_requirement_analyzer 分析 docs/活动需求.md
```

### 生成测试用例

```text
使用 game_testcase_generator 根据上面的需求分析结果生成测试用例
```

### 捕鱼活动分析

```text
使用 game_activity_analyzer 分析这个捕鱼活动需求，重点看强发库和奖励库存风险
```

### 配置检查

```text
使用 game_config_checker 检查这个活动配置是否有上线风险
```

### Lua 代码 Review

```text
使用 code_review_cocos2d_lua 审查这个领奖逻辑，重点看 nil、重复点击、网络回调和重复发奖风险
```

### Unity 代码 Review

```text
使用 code_review_unity 审查这个活动界面代码，重点看生命周期、资源加载、按钮重复点击
```

### Cocos 代码 Review

```text
使用 code_review_cocos_creator 审查这个 Cocos Creator 代码，重点看 Bundle、SpriteAtlas、小游戏平台兼容
```

### 发版检查

```text
使用 release_check 生成微信小游戏热更新发版检查清单
```

### 性能检查

```text
使用 game_performance_check 分析这个活动界面卡顿问题
```

## Claude Code 使用方式

把本仓库放到 Claude Code 的 Skills 目录。

### 全局安装

```bash
git clone https://github.com/touchatoms/game_skills.git ~/.claude/skills/game-ai-skills
```

也可以把每个 Skill 目录拷贝到：

```bash
~/.claude/skills/
```

最终结构示例：

```text
~/.claude/skills/
├── game_requirement_analyzer/
│   └── SKILL.md
├── game_testcase_generator/
│   └── SKILL.md
└── game_activity_analyzer/
    └── SKILL.md
```

### 项目内安装

在项目根目录放：

```text
.claude/
└── skills/
    ├── game_requirement_analyzer/
    ├── game_testcase_generator/
    └── game_activity_analyzer/
```

## Codex 使用方式

推荐放到项目目录：

```text
.codex/
└── skills/
    ├── game_requirement_analyzer/
    ├── game_testcase_generator/
    └── game_activity_analyzer/
```

也可以在项目根目录增加 `AGENTS.md`，让 Codex 更稳定地按团队规范执行。

示例：

```md
# AGENTS.md

当用户要求需求分析时，优先使用 game_requirement_analyzer 的结构。

当用户要求生成测试用例时，优先使用 game_testcase_generator 的结构。

当用户要求分析捕鱼活动、强发库、库存、掉落配置时，优先使用 game_activity_analyzer 或 game_config_checker。

代码审查时：
- Lua 使用 code_review_cocos2d_lua
- Unity 使用 code_review_unity
- Cocos 使用 code_review_cocos_creator

发版前检查使用 release_check。
性能问题分析使用 game_performance_check。
```

## 团队协作建议

建议团队按下面流程使用：

```text
策划需求文档
   ↓
game_requirement_analyzer
   ↓
开发任务拆解
   ↓
game_testcase_generator
   ↓
开发自测
   ↓
code_review_cocos2d_lua / code_review_unity / code_review_cocos_creator
   ↓
game_config_checker
   ↓
release_check
   ↓
上线
```

## GitHub 推送方式

如果你本地还没有初始化仓库：

```bash
git init
git add .
git commit -m "init game ai skills"
git branch -M main
git remote add origin https://github.com/touchatoms/game_skills.git
git push -u origin main
```

如果仓库已经存在：

```bash
git add .
git commit -m "add game ai skills"
git push
```

## 后续可扩展 Skills

建议后续继续增加：

```text
game_ui_asset_checker
game_audio_checker
game_i18n_checker
android_release_check
ios_release_check
wechat_minigame_check
yooassets_checker
spriteatlas_checker
payment_check
ads_check
analytics_check
bug_root_cause_analyzer
```

## 维护规范

每个 Skill 必须包含：

```text
skill_name/
└── SKILL.md
```

`SKILL.md` 顶部必须包含：

```yaml
---
name: skill_name
description: 这个 Skill 的用途说明
---
```

命名建议：

- 使用英文小写
- 单词之间用下划线
- 名称要稳定，不要频繁修改
- description 要写清楚触发场景
