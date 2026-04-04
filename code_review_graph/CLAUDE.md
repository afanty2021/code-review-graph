# CLAUDE.md - Core Package Module Context

[根目录](../CLAUDE.md) > **code_review_graph/**

> **Last Updated**: 2026-04-04 22:00:51
> **Module Type**: Python Package
> **Coverage**: 100% (62 Python files scanned)

---

## 变更记录 (Changelog)

### 2026-04-04 22:00:51 - 模块文档初始化
- 📋 **创建模块级 CLAUDE.md**：详细记录核心包架构
- 🔗 **添加导航面包屑**：支持路径导航
- 📊 **接口清单**：记录所有对外接口和数据模型
- ✅ **测试覆盖**：映射测试文件到源文件

---

## 模块职责

**code_review_graph/** 是项目的核心 Python 包，负责：

1. **代码解析**：使用 Tree-sitter 解析 19 种编程语言的 AST
2. **图存储**：SQLite 持久化代码结构图谱
3. **MCP 服务**：通过 FastMCP 暴露 22 个工具和 5 个提示模板
4. **增量更新**：基于 Git 的变化检测和增量索引
5. **高级分析**：执行流检测、社区检测、向量搜索、重构建议
6. **CLI 接口**：命令行工具入口点

---

## 入口与启动

### 主要入口点

**1. CLI 入口** (`cli.py`)
```python
# 安装后注册的命令行工具
code-review-graph install|build|update|watch|status|serve|visualize|wiki|detect-changes|register|repos|eval
```

**2. MCP 服务器入口** (`main.py`)
```python
# FastMCP 服务器，通过 stdio 通信
from code_review_graph.main import mcp
mcp.run()  # 启动 MCP 服务器
```

**3. 包入口** (`__init__.py`)
```python
from code_review_graph import __version__  # 版本号导出
```

### 启动流程

```
CLI 命令
  → cli.py:main()
    → 调用相应模块函数
      → graph.py:CodeGraph (图操作)
      → parser.py:parse_file() (解析)
      → incremental.py:update_graph() (增量更新)
      → main.py:mcp.run() (MCP 服务)
```

---

## 对外接口

### MCP 工具 (22 个)

通过 `main.py` 注册到 FastMCP 服务器：

**图谱管理**:
- `build_or_update_graph`: 构建或更新图谱
- `list_graph_stats`: 列出图谱统计信息
- `list_repos_func`: 列出多仓库注册表

**查询工具**:
- `get_review_context`: 获取代码审查上下文
- `get_impact_radius`: 获取影响半径 (BFS)
- `query_graph`: 查询节点和边
- `find_large_functions`: 查找大函数
- `semantic_search_nodes`: 语义搜索节点

**执行流分析**:
- `get_flow`: 获取执行流
- `list_flows`: 列出所有执行流
- `get_affected_flows_func`: 获取受影响的流

**社区检测**:
- `get_community_func`: 获取社区信息
- `list_communities_func`: 列出所有社区
- `get_architecture_overview_func`: 获取架构概览

**变更分析**:
- `detect_changes_func`: 检测变更 (风险评分)
- `get_docs_section`: 获取文档节

**重构支持**:
- `refactor_func`: 重构预览
- `apply_refactor_func`: 应用重构

**Wiki 生成**:
- `generate_wiki_func`: 生成 wiki
- `get_wiki_page_func`: 获取 wiki 页面

**搜索**:
- `cross_repo_search_func`: 跨仓库搜索

**嵌入**:
- `embed_graph`: 计算向量嵌入

### MCP 提示模板 (5 个)

通过 `prompts.py` 定义：

1. `review_changes_prompt`: 变更审查
2. `architecture_map_prompt`: 架构图
3. `debug_issue_prompt`: 调试问题
4. `onboard_developer_prompt`: 开发者入门
5. `pre_merge_check_prompt`: 合并前检查

### CLI 命令

通过 `cli.py` 暴露：

- `install`: 安装 MCP 配置
- `init`: install 的别名
- `build`: 构建图谱
- `update`: 增量更新
- `watch`: 监视模式
- `status`: 显示统计
- `visualize`: 生成可视化
- `wiki`: 生成 wiki
- `detect-changes`: 检测变更
- `register`: 注册仓库
- `unregister`: 注销仓库
- `repos`: 列出仓库
- `eval`: 运行评估
- `serve`: 启动 MCP 服务

---

## 关键依赖与配置

### 核心依赖

**必需依赖** (pyproject.toml):
```toml
[project.dependencies]
mcp = ">=1.0.0,<2"
fastmcp = ">=0.1.0,<2"
tree-sitter = ">=0.23.0,<1"
tree-sitter-language-pack = ">=0.3.0,<1"
networkx = ">=3.2,<4"
watchdog = ">=4.0.0,<6"
```

**可选依赖**:
```toml
[project.optional-dependencies]
embeddings = ["sentence-transformers>=3.0.0,<4", "numpy>=1.26,<3"]
google-embeddings = ["google-generativeai>=0.8.0,<1"]
communities = ["igraph>=0.11.0"]
eval = ["matplotlib>=3.7.0", "pyyaml>=6.0"]
wiki = ["ollama>=0.1.0"]
```

### 配置文件

- **`.code-review-graph/graph.db`**: SQLite 图数据库 (自动生成)
- **`.mcp.json`**: MCP 服务器配置 (自动生成)
- **`tsconfig.json`**: TypeScript 路径别名配置 (项目根目录)

### 环境变量

**向量嵌入**:
- `EMBEDDINGS_MODEL`: sentence-transformers 模型路径
- `GOOGLE_API_KEY`: Google Gemini API 密钥
- `MINIMAX_API_KEY`: MiniMax API 密钥

**Wiki 生成**:
- `OLLAMA_BASE_URL`: Ollama 服务地址
- `OLLAMA_MODEL`: Ollama 模型名称

---

## 数据模型

### 核心数据类

**节点信息** (`parser.py`):
```python
@dataclass
class NodeInfo:
    kind: str  # File, Class, Function, Type, Test
    name: str
    file_path: str
    line_start: int
    line_end: int
    language: str = ""
    parent_name: Optional[str] = None
    params: Optional[str] = None
    return_type: Optional[str] = None
    modifiers: Optional[str] = None
    is_test: bool = False
    extra: dict = field(default_factory=dict)
```

**边信息** (`parser.py`):
```python
@dataclass
class EdgeInfo:
    kind: str  # CALLS, IMPORTS_FROM, INHERITS, IMPLEMENTS, CONTAINS, TESTED_BY, DEPENDS_ON
    source: str
    target: str
    file_path: str
    line: int = 0
    extra: dict = field(default_factory=dict)
```

**图节点** (`graph.py`):
```python
@dataclass
class GraphNode:
    id: int
    kind: str
    name: str
    qualified_name: str
    file_path: str
    line_start: int
    line_end: int
    language: Optional[str]
    parent_name: Optional[str]
    params: Optional[str]
    return_type: Optional[str]
    modifiers: Optional[str]
    is_test: bool
    file_hash: Optional[str]
    extra: dict
    updated_at: float
```

**图边** (`graph.py`):
```python
@dataclass
class GraphEdge:
    id: int
    kind: str
    source_qualified: str
    target_qualified: str
    file_path: str
    line: int
    extra: dict
    updated_at: float
```

### 数据库模式

**节点表** (`nodes`):
- `id`: 主键
- `kind`: 节点类型 (File, Class, Function, Type, Test)
- `name`: 名称
- `qualified_name`: 限定名称 (唯一)
- `file_path`: 文件路径
- `line_start`, `line_end`: 行号范围
- `language`: 编程语言
- `parent_name`: 父节点名称
- `params`, `return_type`, `modifiers`: 函数/类型信息
- `is_test`: 是否为测试
- `file_hash`: SHA-256 哈希
- `extra`: JSON 额外数据
- `updated_at`: 更新时间戳

**边表** (`edges`):
- `id`: 主键
- `kind`: 边类型 (CALLS, IMPORTS_FROM, INHERITS, IMPLEMENTS, CONTAINS, TESTED_BY, DEPENDS_ON)
- `source_qualified`: 源节点限定名称
- `target_qualified`: 目标节点限定名称
- `file_path`: 文件路径
- `line`: 行号
- `extra`: JSON 额外数据
- `updated_at`: 更新时间戳

**元数据表** (`metadata`):
- `key`: 键
- `value`: 值

---

## 测试与质量

### 测试覆盖

**单元测试** (映射到源文件):

| 测试文件 | 覆盖源文件 | 测试内容 |
|---------|-----------|---------|
| `test_parser.py` | `parser.py` | 多语言解析、交叉文件解析 |
| `test_graph.py` | `graph.py` | 图 CRUD、统计、影响半径 |
| `test_incremental.py` | `incremental.py` | 构建、更新、迁移、Git 操作 |
| `test_flows.py` | `flows.py` | 执行流检测、关键性评分 |
| `test_communities.py` | `communities.py` | 社区检测、架构概览 |
| `test_search.py` | `search.py` | FTS5 混合搜索 |
| `test_changes.py` | `changes.py` | 风险评分变更分析 |
| `test_refactor.py` | `refactor.py` | 重命名预览、死代码检测 |
| `test_embeddings.py` | `embeddings.py` | 向量编码/解码、相似度 |
| `test_visualization.py` | `visualization.py` | 导出、HTML 生成 |
| `test_hints.py` | `hints.py` | 审查提示生成 |
| `test_prompts.py` | `prompts.py` | MCP 提示模板 |
| `test_wiki.py` | `wiki.py` | Wiki 生成 |
| `test_skills.py` | `skills.py` | 技能定义 |
| `test_registry.py` | `registry.py` | 多仓库注册表 |
| `test_migrations.py` | `migrations.py` | 数据库迁移 |
| `test_tools.py` | `tools/*.py` | MCP 工具集成 |
| `test_tsconfig_resolver.py` | `tsconfig_resolver.py` | TypeScript 路径解析 |
| `test_multilang.py` | `parser.py` | 19 种语言解析 |
| `test_integration_v2.py` | 全部 | v2 管道集成测试 |

**测试命令**:
```bash
# 运行所有测试
pytest tests/ --tb=short -q

# 运行特定测试
pytest tests/test_parser.py -v
pytest tests/test_integration_v2.py -v

# 测试覆盖率
pytest tests/ --cov=code_review_graph --cov-report=html
```

### 代码质量工具

**Lint** (ruff):
```bash
ruff check code_review_graph/
```

**类型检查** (mypy):
```bash
mypy code_review_graph/ --ignore-missing-imports
```

**安全扫描** (bandit):
```bash
bandit -r code_review_graph/
```

---

## 常见问题 (FAQ)

### Q1: 如何添加对新语言的支持？
**A**: 在 `parser.py` 的 `EXTENSION_TO_LANGUAGE` 字典中添加扩展名映射，并在 `get_parser()` 函数中确保语言包可用。

### Q2: 如何自定义 MCP 工具？
**A**: 在 `tools/` 目录下创建新模块，实现工具函数，然后在 `main.py` 中注册到 FastMCP 服务器。

### Q3: 如何调试增量更新问题？
**A**: 使用 `code-review-graph status` 查看图谱统计，检查 `incremental.py` 中的哈希比对逻辑，启用 DEBUG 日志级别。

### Q4: 如何优化大型 monorepo 的性能？
**A**:
1. 使用 `register` 命令分别管理子仓库
2. 调整 SQLite 缓存大小 (`PRAGMA cache_size`)
3. 启用 WAL 模式 (默认启用)
4. 考虑使用 `embed_graph` 预计算向量索引

### Q5: 如何扩展社区检测算法？
**A**: 在 `communities.py` 中的 `detect_communities()` 函数中添加新算法，支持 igraph 的 Leiden 算法或基于文件分组的自定义方法。

---

## 相关文件清单

### 核心模块 (19 个文件)

**解析与存储**:
- `parser.py` (80+ 行): 多语言 AST 解析器
- `graph.py` (80+ 行): SQLite 图数据库引擎
- `migrations.py`: 数据库迁移 (v1-v5)

**增量更新**:
- `incremental.py`: Git 变化检测、文件监视

**MCP 服务**:
- `main.py` (50+ 行): FastMCP 服务器入口
- `prompts.py`: 5 个 MCP 提示模板
- `cli.py` (100+ 行): CLI 命令处理

**MCP 工具** (`tools/` 目录):
- `tools/__init__.py`: 工具注册
- `tools/_common.py`: 共享工具函数
- `tools/build.py`: 图谱构建工具
- `tools/query.py`: 图查询工具
- `tools/review.py`: 审查上下文工具
- `tools/flows_tools.py`: 执行流工具
- `tools/community_tools.py`: 社区检测工具
- `tools/registry_tools.py`: 多仓库工具
- `tools/refactor_tools.py`: 重构工具
- `tools/docs.py`: 文档工具

**高级分析**:
- `flows.py`: 执行流检测
- `communities.py`: 社区检测
- `search.py`: 混合搜索
- `changes.py`: 变更影响分析
- `refactor.py`: 重构建议
- `embeddings.py`: 向量嵌入
- `visualization.py`: D3.js 可视化
- `wiki.py`: Markdown wiki 生成

**辅助模块**:
- `hints.py`: 审查提示生成
- `skills.py`: Claude Code 技能定义
- `registry.py`: 多仓库注册表
- `tsconfig_resolver.py`: TypeScript 路径解析
- `constants.py`: 常量定义

**评估** (`eval/` 目录):
- `eval/runner.py`: 评估运行器
- `eval/scorer.py`: 评分器
- `eval/reporter.py`: 报告生成器
- `eval/benchmarks/`: 基准测试套件

---

*此文档由 AI 自动生成和维护，最后更新于 2026-04-04 22:00:51*
