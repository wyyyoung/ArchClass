# ArchClass - Multi-Architecture Code Classification System

## 项目简介

**ArchClass** 是一个面向多指令集架构的 C/C++ 项目智能代码分类系统。该系统通过**静态分析**和**大语言模型（LLM）**的深度协作，自动识别和标记代码中与特定指令集架构相关的代码片段，帮助开发者快速定位架构相关代码，提升跨平台开发效率。

## 核心功能

- **多架构支持**: 支持 x86、ARM、RISC-V、MIPS、PowerPC、LoongISA 等主流指令集架构
- **智能代码分析**: 自动识别架构相关的 intrinsics、汇编代码、宏定义和编译器属性
- **符号依赖分析**: 构建符号依赖图，通过图传播算法自动标记架构信息
- **LLM 驱动的修复**: 结合静态分析和大语言模型进行智能代码修复
- **迭代优化机制**: 持续迭代修复直到错误显著减少
- **可视化报告**: 生成详细的分析报告和代码标记结果

## 技术栈

| 分类 | 技术 |
|------|------|
| 语言 | Python 3.8+ |
| 框架 | Flask、LangChain |
| 代码分析 | Tree-Sitter |
| 大语言模型 | OpenAI API、LangChain OpenAI |
| 向量数据库 | FAISS |
| 数据库 | MySQL |

---

## 大语言模型（LLM）集成

### 1. LLM 在系统中的核心作用

ArchClass 创新性地将大语言模型融入代码分析流程，主要体现在以下几个方面：

| 应用场景 | LLM 角色 | 具体功能 |
|----------|----------|----------|
| **代码修复** | 智能修复引擎 | 根据静态分析结果修复架构相关代码问题 |
| **报告生成** | 报告撰写助手 | 基于分析结果生成结构化报告 |
| **错误聚类** | 智能分析器 | 对错误行进行语义聚类，筛选代表性错误 |
| **代码重写** | 代码优化器 | 提供架构迁移建议和代码优化方案 |

### 2. Agent 模块架构

系统包含两个核心的 LLM Agent：

#### 2.1 report_Agent（报告生成代理）

负责生成详细的分析报告：

```python
# 核心流程
report_Agent_instance = report_Agent(system_prompts_for_reporter)
report = report_Agent_instance.get_response(
    f"请基于以下建议和分析结果，根据用户的要求{user_prompt}，生成一份报告：\n\nllm提出的修改建议：{advice}\n\n分析结果：{file_analysis_results}"
)
```

**设计要点**：
- 使用 `ConversationBufferWindowMemory` 维护对话上下文
- 支持自定义用户提示，生成定制化报告
- 整合静态分析结果和 LLM 建议

#### 2.2 rewrite_Agent（代码重写代理）

负责代码修复和优化建议：

```python
# 核心流程
agent = rewrite_Agent(system_prompts_for_ask)
response = agent.get_response(question)
```

**设计要点**：
- 处理用户的自然语言查询
- 提供代码修改建议
- 支持交互式代码优化

### 3. 迭代修复机制

系统采用**迭代式修复策略**，结合静态分析和 LLM 进行持续优化：

```
┌─────────────────────────────────────────────────────────────┐
│                    迭代修复流程                              │
├─────────────────────────────────────────────────────────────┤
│  静态分析 ──▶ 错误聚类 ──▶ LLM修复 ──▶ 验证反馈 ──▶ 静态分析  │
│     │                              │                       │
│     ▼                              ▼                       │
│  检测错误行                      检查修复效果              │
│     │                              │                       │
│     └──────────────────────────────┴───────────────────────┘
│                              │
│                              ▼
│                       达到停止条件？
│                              │
│              ┌───────────────┴───────────────┐
│              ▼                               ▼
│           是 → 输出报告                  否 → 继续迭代
└─────────────────────────────────────────────────────────────┘
```

#### 3.1 错误筛选与聚类

为提高修复效率，系统首先对错误行进行筛选和聚类：

```python
# 使用 K-means 聚类选择代表性错误行
if len(embeddings) > 1:
    kmeans = KMeans(n_clusters=1, random_state=0)
    kmeans.fit(embeddings)
    
    # 获取最具代表性的错误行
    closest_idx = np.argmin(np.linalg.norm(embeddings - kmeans.cluster_centers_, axis=1))
    representative_error_line = error_lines[closest_idx]
```

**聚类优势**：
- **节省 Token 消耗**：只修复代表性错误，减少 LLM 调用次数
- **避免重复修复**：相似错误只修复一次
- **提高修复效率**：聚焦高价值修复点

#### 3.2 修复流程

1. **静态分析阶段**：使用 `Static_CodeAnalyzer` 检测代码错误
2. **错误筛选阶段**：通过 K-means 聚类筛选代表性错误
3. **LLM 修复阶段**：调用大语言模型修复选中的错误行
4. **验证阶段**：重新执行静态分析，检查修复效果
5. **迭代决策**：根据错误减少情况决定是否继续迭代

### 4. 向量嵌入与相似度计算

系统使用 **Sentence Transformers** 进行错误行的语义嵌入：

```python
from sentence_transformers import SentenceTransformer

# 加载预训练模型
model = SentenceTransformer('all-MiniLM-L6-v2')

# 生成错误行嵌入
embeddings = model.encode(error_lines)
```

**应用场景**：
- 错误行语义相似度计算
- 错误聚类和分类
- 重复错误检测

### 5. LLM 工具封装

系统提供了专门的 LLM 工具封装模块 (`llm_tools/`)：

| 工具 | 功能 | 说明 |
|------|------|------|
| `task.py` | 任务状态管理 | 检查和更新分析任务进度 |
| `graph.py` | 图数据处理 | 处理架构依赖图数据 |

### 6. 性能优化策略

#### 6.1 Token 消耗优化

| 策略 | 实现方式 | 效果 |
|------|----------|------|
| 错误筛选 | 聚类选择代表性错误 | 减少 80%+ 的 LLM 调用 |
| 上下文裁剪 | 只传递相关代码片段 | 降低单次调用 Token 数 |
| 批处理 | 合并相似请求 | 提高处理效率 |

#### 6.2 时间消耗分析（基于 Eigen 科学计算库测试）

| 阶段 | 耗时 |
|------|------|
| 静态分析 | ~2.15 秒 |
| 错误编码（Sentence Transformer） | ~55.05 秒 |
| 相似度计算（FAISS） | ~0.31 秒 |
| **总计** | **~59.74 秒** |

---

## 核心模块说明（分步识别）

### 1. PreScreener（预筛选器）
- 扫描项目中的 C/C++ 源文件
- 过滤无关文件，保留架构相关代码

### 2. static_classify_tool（静态分类工具）
- 使用 Tree-Sitter 进行语法解析
- 识别标记目标、内部符号和外部符号

### 3. symbol_tagger（符号标记器）
- 通过 intrinsics、汇编、宏定义等识别架构特征
- 支持多种指令集架构的自动标记

### 4. CodeAnalyzer（代码分析器）
- 构建符号依赖图
- 通过拓扑排序传播架构信息

---

## 项目结构

```
code-annotation-backend-master/
├── algo_framework/          # 核心算法框架
│   ├── module/              # 功能模块
│   │   ├── prescreener.py   # 预筛选模块
│   │   ├── code_analyzer.py # 代码分析模块
│   │   ├── code_rewriter.py # 代码重写模块
│   │   └── agent.py         # LLM 代理模块
│   ├── utils/               # 工具函数
│   ├── data/                # 配置数据
│   └── models/              # 机器学习模型
│       └── all-MiniLM-L6-v2/ # Sentence Transformer 模型
├── app/                     # Flask 应用
│   ├── User/                # 用户管理
│   ├── Project/             # 项目管理
│   ├── Analysis/            # 分析服务
│   └── Graph/               # 图数据服务
├── db/                      # 数据库模块
├── llm_tools/               # LLM 工具封装
│   ├── task.py              # 任务状态管理
│   └── graph.py             # 图数据处理
├── utils/                   # 通用工具
└── run.py                   # 启动脚本
```

---

## 支持的指令集架构

| 架构 | 识别方式 | 示例 |
|------|----------|------|
| **x86** | SSE/AVX intrinsics | `_mm_add_ps`, `__m128` |
| **ARM** | NEON intrinsics | `vaddvq_s32`, `uint8x16_t` |
| **RISC-V** | RISC-V intrinsics | `__riscv_vadd`, `vint32m1_t` |
| **MIPS** | MIPS 汇编指令 | `addi`, `lw` |
| **PowerPC** | AltiVec intrinsics | `vec_add`, `vector float` |
| **LoongISA** | LoongArch intrinsics | `__lsx_vadd`, `__lasx_xvadd` |

---

## 使用方法

### API 接口

#### 1. 项目分析

```bash
POST /api/analysis/execute
Content-Type: application/json

{
"project_id": "1",
"project_path": "/path/to/project",
"user_prompt": "请分析代码的架构依赖关系并生成报告"
}
```

**响应示例**：

```json
{
"report": "基于分析，您的项目包含以下架构相关代码...",
"file_tag_data": [
    {
    "file_path": "src/PacketMath.h",
    "tag_data": [
        {"start": 45, "end": 52, "arch_names": ["x86", "arm"]}
    ]
    }
]
}
```

#### 2. 查询分析结果

```bash
GET /api/analysis/result?task_id=<task_id>
```

#### 3. 交互式查询

```bash
POST /api/analysis/ask
Content-Type: application/json

{
"question": "如何将这段 x86 代码迁移到 ARM 架构？"
}
```

---

## 工作流程

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│  1. 预筛选阶段  │ ──▶ │  2. 静态分析阶段 │ ──▶ │  3. 符号标记阶段 │
└─────────────────┘     └─────────────────┘     └─────────────────┘
        │                       │                       │
        ▼                       ▼                       ▼
    加载源文件              构建符号依赖图            识别架构特征
    过滤无关文件            分析调用关系              标记代码位置
                            │                       │
                            ▼                       ▼
                   ┌─────────────────┐     ┌─────────────────┐
                   │  4. 图传播阶段   │ ──▶ │  5. LLM 处理阶段 │
                   └─────────────────┘     └─────────────────┘
                                                   │
                                                   ▼
                                         ┌─────────────────┐
                                         │  6. 报告生成    │
                                         └─────────────────┘
```

---
