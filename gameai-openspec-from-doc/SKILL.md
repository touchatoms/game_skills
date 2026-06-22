---
name: gameai-openspec-from-doc
description: 输入一个或多个文档（需求/设计/接口），自动提取→分类分析→融合生成 OpenSpec change（proposal/design/tasks/specs），最后进入 explore 讨论。
license: MIT
compatibility: Requires openspec CLI. Requires python3 for docx extraction.
metadata:
  author: pimingzhen
  version: "2.0"
---

输入一个或多个文档，自动完成：

```
┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│  需求文档     │    │  设计文档     │    │  接口文档     │
│  .docx/.md   │    │  .docx/.md   │    │  .docx/.md   │
└──────┬───────┘    └──────┬───────┘    └──────┬───────┘
       │                   │                   │
       └───────────────────┼───────────────────┘
                           ▼
               ┌─────────────────────┐
               │   融合需求分析        │
               └─────────┬───────────┘
                         ▼
               ┌─────────────────────┐
               │  OpenSpec Propose   │
               │  proposal / design  │
               │  specs   / tasks    │
               └─────────┬───────────┘
                         ▼
               ┌─────────────────────┐
               │  Explore 讨论        │
               └─────────────────────┘
```

最终产出：
- `openspec/changes/<name>/proposal.md` — 做什么 & 为什么
- `openspec/changes/<name>/design.md` — 怎么做（融合设计文档内容）
- `openspec/changes/<name>/specs/*/spec.md` — 接口 & 功能规格（融合接口文档内容）
- `openspec/changes/<name>/tasks.md` — 实施步骤
- 一份融合需求分析报告
- 待确认问题清单

---

## 输入

用户通过 `/opsx:from-doc` 提供文档路径，支持以下格式：

| 格式 | 说明 |
|---|---|
| `/opsx:from-doc <需求.docx>` | 单个需求文档 |
| `/opsx:from-doc -r <需求.docx> -d <设计.docx>` | 需求 + 设计文档 |
| `/opsx:from-doc -r <需求.docx> -d <设计.docx> -i <接口.docx>` | 全量三文档 |
| `/opsx:from-doc -r <需求.docx> -i <接口.docx>` | 需求 + 接口文档 |

**参数说明：**

| 参数 | 含义 | 影响 |
|---|---|---|
| `-r` / `--requirement` | 需求文档 | 输入到 Phase 2 全维度分析 |
| `-d` / `--design` | 设计文档（技术方案/架构设计） | 融合到 design.md artifact |
| `-i` / `--interface` | 接口文档（API 协议/数据结构） | 融合到 specs + design.md |

**支持的文件格式：** `.docx`、`.md`、`.txt`

如果未提供任何路径，使用 **AskUserQuestion tool** 依次询问：
1. "请提供需求文档路径（可跳过）"
2. "请提供设计文档路径（可跳过）"
3. "请提供接口文档路径（可跳过）"

至少需要一个文档（需求文档），否则终止。

---

## 执行流程

### Phase 1: 文档提取

对每个提供的文档：

```bash
ls "<doc_path>"
```

根据文件扩展名选择提取方式：

**`.docx` — 使用 python3 解压 XML：**

```python
import zipfile
import xml.etree.ElementTree as ET

with zipfile.ZipFile('<doc_path>', 'r') as z:
    xml_content = z.read('word/document.xml')

tree = ET.fromstring(xml_content)
for p in tree.iter('{http://schemas.openxmlformats.org/wordprocessingml/2006/main}p'):
    texts = [t.text for t in p.iter('{http://schemas.openxmlformats.org/wordprocessingml/2006/main}t') if t.text]
    if texts:
        print(''.join(texts))
```

**`.md` / `.txt` — 直接 Read：**

使用 Read 工具直接读取文件内容。

**提取完成后标记每个文档：**
```
📄 需求文档: 签到-新版.docx — 75 段
📄 设计文档: 签到技术方案.md — 120 行
📄 接口文档: signin-api.docx — 45 段
```

将每个文档的内容保存为独立变量：
- `$req_content` — 需求文档内容
- `$design_content` — 设计文档内容
- `$interface_content` — 接口文档内容

---

### Phase 2: 融合需求分析

1. **对需求文档** → 15 维度深度分析（同 gameai_analyze_requirement）

2. **对设计文档** → 提取并整理：
   - 技术架构/系统分层
   - 关键设计决策
   - 模块划分
   - 数据流
   - 依赖关系

3. **对接口文档** → 提取并整理：
   - 所有 API 端点（method + path）
   - 请求参数（字段名、类型、必填、说明）
   - 响应字段（字段名、类型、说明）
   - 错误码定义
   - 数据结构/枚举定义
   - 调用时机和频率

4. **融合输出**：
   将三份分析合并为一份完整的分析报告，明确标注信息来源：

```markdown
## 接口协议（来源：接口文档）
| 接口 | Method | 请求参数 | 响应字段 | 说明 |
|---|---|---|---|---|
| get_signin_info | GET | activity_id | consecutive_days, ... | 拉取签到状态 |
```

将融合分析写入：`/tmp/openspec_merged_analysis.md`

---

### Phase 3: 创建 OpenSpec Change

1. **确定 change name**（同 v1，从需求文档标题提取 kebab-case，用户确认）

2. **创建 change 目录**
   ```bash
   openspec new change "<name>"
   ```

3. **获取 artifact 依赖**
   ```bash
   openspec status --change "<name>" --json
   ```

4. **按依赖顺序生成 artifacts，融合多文档内容：**

   | Artifact | 内容来源 |
   |---|---|
   | **proposal.md** | 需求文档分析（15维度概括） |
   | **design.md** | 需求分析的技术部分 + **设计文档提取的架构/决策** + 接口文档的数据结构 |
   | **specs/\*.md** | 需求功能点 + **接口文档的 API 定义**（每个接口的 request/response 写入对应 spec） |
   | **tasks.md** | 需求分析任务 + 设计文档的模块拆分 + 接口文档的联调任务 |

   对每个 ready artifact：
   ```bash
   openspec instructions <artifact-id> --change "<name>" --json
   ```
   - 用 `template` 作为文件结构
   - 将对应来源内容融合填充（不要只是拼接，要整合）
   - `context`、`rules` 是约束，不写入文件

   **特别处理 — 接口文档 → specs：**
   如果提供了接口文档，在 `specs/` 目录下额外创建 `api-contract/spec.md`：
   ```
   specs/api-contract/spec.md
   ```
   内容包含：
   - 每个 API 的 Requirement + Scenario
   - Request/Response schema
   - 错误码表

5. **验证完成**
   ```bash
   openspec status --change "<name>"
   ```

---

### Phase 4: Explore 讨论

1. 展示融合方案摘要
2. 逐条"待确认问题"（标注每个问题来自哪个文档）
3. 如果提供了设计文档和接口文档，重点讨论：
   - 设计方案是否有缺口？
   - 接口定义是否完整？
   - 与需求是否一致？
4. 询问下一步

---

## 输出示例

```
## 📄 Phase 1: 文档提取

📄 需求文档: 签到-新版.docx — 75 段
📄 设计文档: 签到技术设计.md — 120 行
📄 接口文档: signin-api.docx — 45 段

## 🔍 Phase 2: 融合分析

✅ 需求分析: 15/15 维度
✅ 设计提取: 系统分层 + 5 个关键决策
✅ 接口提取: 4 个 API + 3 个数据结构 + 8 个错误码
融合报告: /tmp/openspec_merged_analysis.md

## 📋 Phase 3: OpenSpec 方案

**Change:** daily-signin-v2

✅ proposal.md — 融合需求概述
✅ design.md — 融合技术设计决策 + 接口架构
✅ specs/daily-signin/spec.md
✅ specs/cumulative-signin/spec.md
✅ specs/signin-coin-limit/spec.md
✅ specs/api-contract/spec.md — 接口协议规格
✅ tasks.md — 42 个实施任务（含联调任务）

## 💬 Phase 4: 待讨论

### 来自需求文档 (5)
1. 连续签到最大天数？
...

### 来自设计文档 (2)
6. 缓存策略是 Redis 单层还是 Redis+本地？
...

### 来自接口文档 (3)
8. get_signin_info 是否需要分页？
...

### 跨文档一致性问题 (2)
11. 需求说"支持最多2个道具"，接口设计支持3个 — 以哪个为准？
...
```

---

## Guardrails

- **至少需要一个需求文档**，设计文档和接口文档可选
- **Phase 1→4 不可跳过**
- **跨文档一致性检查**：如果需求、设计、接口之间存在矛盾，必须在"待确认问题"中高亮标记
- **信息来源标注**：artifact 中融合多文档内容时，不要混淆来源
- **文档提取失败**：提示文件格式问题，不影响其他已成功提取的文档
- **change name 已存在**：询问覆盖/新名称/继续已有
- **openspec CLI 不可用**：手动创建文件作为降级
- **分析不明确处列为待确认**，不猜测
- 使用 **AskUserQuestion tool** 让用户做关键决策
