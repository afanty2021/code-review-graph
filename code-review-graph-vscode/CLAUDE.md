# CLAUDE.md - VS Code Extension Module Context

[根目录](../CLAUDE.md) > **code-review-graph-vscode/**

> **Last Updated**: 2026-04-04 22:00:51
> **Module Type**: TypeScript/VS Code Extension
> **Coverage**: 100% (15 TypeScript files scanned)

---

## 变更记录 (Changelog)

### 2026-04-04 22:00:51 - 模块文档初始化
- 📋 **创建模块级 CLAUDE.md**：详细记录 VS Code 扩展架构
- 🔗 **添加导航面包屑**：支持路径导航
- 🎨 **UI 组件清单**：记录所有视图和命令
- ✅ **集成说明**：说明与核心包的集成方式

---

## 模块职责

**code-review-graph-vscode/** 是项目的 VS Code 扩展，负责：

1. **可视化界面**：提供树视图和侧边栏界面展示代码图谱
2. **交互式图谱**：使用 D3.js 渲染交互式依赖图
3. **命令集成**：暴露常用命令到 VS Code 命令面板
4. **自动更新**：监视文件变化并自动更新图谱
5. **光标导航**：提供 "Go to Definition" 和 "Find References" 集成

---

## 入口与启动

### 扩展入口点

**主入口** (`src/extension.ts`):
```typescript
// VS Code 扩展激活入口
export function activate(context: vscode.ExtensionContext) {
  // 初始化后端、视图、命令
}

// 扩展停载
export function deactivate() {
  // 清理资源
}
```

### 激活条件

在 `package.json` 中定义的激活事件：
```json
"activationEvents": [
  "workspaceContains:.code-review-graph/graph.db",
  "onCommand:codeReviewGraph.*",
  "onView:codeReviewGraph.*"
]
```

### 启动流程

```
VS Code 激活扩展
  → extension.ts:activate()
    → 初始化 CLI 后端 (backend/cli.ts)
    → 初始化 SQLite 连接 (backend/sqlite.ts)
    → 注册树视图 (views/treeView.ts)
    → 注册图可视化 (views/graphWebview.ts)
    → 注册命令 (features/)
    → 启动文件监视器 (backend/watcher.ts)
```

---

## 对外接口

### VS Code 命令 (12 个)

通过 `package.json` 的 `contributes.commands` 暴露：

**图谱管理**:
- `codeReviewGraph.buildGraph`: 构建图谱
- `codeReviewGraph.updateGraph`: 更新图谱
- `codeReviewGraph.watchGraph`: 监视模式

**查询与导航**:
- `codeReviewGraph.showBlastRadius`: 显示爆炸半径
- `codeReviewGraph.findCallers`: 查找调用者
- `codeReviewGraph.findCallees`: 查找被调用者
- `codeReviewGraph.findTests`: 查找测试
- `codeReviewGraph.queryGraph`: 查询图谱

**可视化**:
- `codeReviewGraph.showGraph`: 显示图谱
- `codeReviewGraph.search`: 搜索节点

**代码审查**:
- `codeReviewGraph.reviewChanges`: 审查变更
- `codeReviewGraph.embedGraph`: 计算嵌入

**分析**:
- `codeReviewGraph.findLargeFunctions`: 查找大函数

### 树视图 (3 个)

通过 `package.json` 的 `contributes.views` 暴露：

1. **Code Graph** (`codeReviewGraph.codeGraph`):
   - 显示文件、类、函数、测试节点
   - 支持过滤节点类型

2. **Blast Radius** (`codeReviewGraph.blastRadius`):
   - 显示选中节点的影响半径
   - 按深度分组显示

3. **Stats** (`codeReviewGraph.stats`):
   - 显示图谱统计信息
   - 节点/边数量、语言分布

### 配置项 (11 个)

通过 `package.json` 的 `contributes.configuration` 暴露：

**CLI 配置**:
- `codeReviewGraph.cliPath`: CLI 二进制路径

**自动更新**:
- `codeReviewGraph.autoUpdate`: 自动更新图谱 (默认: true)

**可视化配置**:
- `codeReviewGraph.blastRadiusDepth`: 爆炸半径深度 (1-10, 默认: 2)
- `codeReviewGraph.graphTheme`: 图主题 (auto/light/dark)
- `codeReviewGraph.graph.defaultEdges`: 默认边类型
- `codeReviewGraph.graph.maxNodes`: 最大节点数 (10-5000, 默认: 500)

**树视图配置**:
- `codeReviewGraph.treeView.showFunctions`: 显示函数 (默认: true)
- `codeReviewGraph.treeView.showClasses`: 显示类 (默认: true)
- `codeReviewGraph.treeView.showFiles`: 显示文件 (默认: true)
- `codeReviewGraph.treeView.showTypes`: 显示类型 (默认: true)
- `codeReviewGraph.treeView.showTests`: 显示测试 (默认: true)

---

## 关键依赖与配置

### 核心依赖

**必需依赖** (package.json):
```json
{
  "dependencies": {
    "better-sqlite3": "^11.0.0",
    "d3": "^7.9.0"
  }
}
```

**开发依赖**:
```json
{
  "devDependencies": {
    "@types/better-sqlite3": "^7.6.8",
    "@types/d3": "^7.4.3",
    "@types/node": "^20.11.0",
    "@types/vscode": "^1.85.0",
    "@vscode/test-electron": "^2.3.8",
    "@vscode/vsce": "^2.22.0",
    "esbuild": "^0.20.0",
    "typescript": "^5.3.3"
  }
}
```

### 配置文件

- **`tsconfig.json`**: TypeScript 编译配置
- **`esbuild.mjs`**: 打包配置
- **`.vscodeignore`**: 打包忽略文件

### 构建配置

**esbuild 配置** (`esbuild.mjs`):
```javascript
esbuild.context({
  entryPoints: ['src/extension.ts'],
  bundle: true,
  outfile: 'dist/extension.js',
  external: ['vscode'],
  format: 'cjs',
  target: 'node20',
})
```

---

## 数据模型

### 图节点 (TreeItem)

```typescript
interface GraphNodeTreeItem extends vscode.TreeItem {
  node_id: number;
  kind: string;  // File, Class, Function, Type, Test
  qualified_name: string;
  file_path: string;
  line_start: number;
  line_end: number;
  language: string;
}
```

### 图边

```typescript
interface GraphEdge {
  id: number;
  kind: string;  // CALLS, IMPORTS_FROM, INHERITS, etc.
  source_qualified: string;
  target_qualified: string;
  file_path: string;
  line: number;
}
```

### 图谱统计

```typescript
interface GraphStats {
  nodeCount: number;
  edgeCount: number;
  languageStats: Record<string, number>;
  fileCount: number;
  functionCount: number;
  classCount: number;
  testCount: number;
}
```

---

## 测试与质量

### 测试覆盖

**测试文件**:
- `test/sqlite.test.ts`: SQLite 连接测试

**测试命令**:
```bash
# 编译
npm run compile

# 运行测试
npm test

# 打包
npm run package
```

### 代码质量工具

**类型检查**:
```bash
npm run lint  # tsc --noEmit
```

**构建验证**:
```bash
npm run compile  # esbuild
```

---

## 常见问题 (FAQ)

### Q1: 如何调试扩展？
**A**:
1. 按 F5 启动扩展开发主机
2. 在源代码中设置断点
3. 在新窗口中触发扩展功能

### Q2: 如何添加新的树视图？
**A**:
1. 在 `src/views/treeView.ts` 中实现新的 TreeDataProvider
2. 在 `package.json` 的 `contributes.views` 中注册视图
3. 在 `src/extension.ts` 中注册视图提供者

### Q3: 如何自定义图谱可视化？
**A**:
1. 修改 `src/webview/graph.ts` 中的 D3.js 代码
2. 调整节点布局、颜色、交互行为
3. 使用 `package.json` 中的配置项暴露用户设置

### Q4: 如何集成新的后端命令？
**A**:
1. 在 `src/backend/cli.ts` 中添加 CLI 调用方法
2. 在 `src/features/` 中创建新功能模块
3. 在 `package.json` 的 `contributes.commands` 中注册命令
4. 在 `src/extension.ts` 中注册命令处理器

### Q5: 如何优化大型图谱的性能？
**A**:
1. 使用 `codeReviewGraph.graph.maxNodes` 限制显示节点数
2. 实现虚拟滚动 (仅在树视图中需要)
3. 使用 Web Worker 进行 D3.js 渲染
4. 缓存 SQLite 查询结果

---

## 相关文件清单

### 核心模块 (15 个文件)

**扩展入口**:
- `src/extension.ts`: 扩展激活/停载、命令注册

**后端集成** (`src/backend/`):
- `backend/cli.ts`: CLI 进程管理
- `backend/sqlite.ts`: SQLite 数据库连接
- `backend/watcher.ts`: 文件监视器

**功能模块** (`src/features/`):
- `features/blastRadius.ts`: 爆炸半径计算
- `features/cursorResolver.ts`: 光标解析器 (Go to Definition)
- `features/navigation.ts`: 导航辅助
- `features/reviewAssistant.ts`: 审查助手
- `features/scmDecorations.ts`: SCM 装饰器
- `features/search.ts`: 搜索功能

**视图** (`src/views/`):
- `views/treeView.ts`: 树视图提供者
- `views/treeItems.ts`: 树项定义
- `views/statusBar.ts`: 状态栏集成
- `views/graphWebview.ts`: D3.js 图可视化

**入门指南** (`src/onboarding/`):
- `onboarding/installer.ts`: 安装向导
- `onboarding/welcome.ts`: 欢迎 walkthrough

**可视化** (`src/webview/`):
- `webview/graph.ts`: D3.js 图渲染逻辑

**配置**:
- `package.json`: 扩展清单、命令、视图、配置
- `tsconfig.json`: TypeScript 配置
- `esbuild.mjs`: 打包配置

**媒体资源** (`media/`):
- `media/icons/icon.png`: 扩展图标
- `media/icons/graph.svg`: 活动栏图标
- `media/walkthrough/`: 欢迎 walkthrough 资源

---

*此文档由 AI 自动生成和维护，最后更新于 2026-04-04 22:00:51*
