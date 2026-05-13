---
name: taro2rn
description: "Taro 小程序代码转 React Native 完整工具包。提供组件映射、API 映射、样式转换规则和已知问题解决方案。兼容 Taro 3.x 与 React Native 0.72+。"
when_to_use: "用户需要将 Taro 项目代码迁移至 React Native 时触发。Trigger phrases: '转 RN' / '转换 RN' / 'taro to rn' / 'taro2rn' / '迁移成原生' / 'port Taro to RN' / 'Taro 转 native' / '从 Taro 迁出'。也包括询问 Taro/RN 组件映射、样式映射、迁移问题。Do NOT use for: uni-app 或 Vue 小程序迁移、纯新建 RN 项目(无 Taro 来源)、单文件实验性转换、纯 RN 项目调试。"
allowed-tools: Read Write Edit Glob Grep
---

# Taro to React Native 转换

## 快速开始

### 第一步:处理 taro2rnTODO.md

检查项目根目录是否存在 `taro2rnTODO.md`:

| 情况 | 操作 |
|------|------|
| **存在** | 读取进度,跳至第二步 |
| **不存在** | 执行初始化流程(见下方) |

#### 初始化流程

1. **询问用户配置**:

| 配置项 | 默认值 | 说明 |
|--------|--------|------|
| 项目名称 | 当前目录名 | 标识项目 |
| 设计稿宽度 | 750 | 750 或 1242 |
| RN 应用路径 | . | React Native 应用相对路径 |

2. **基于模板生成 taro2rnTODO.md**:读取 `assets/TODO_TEMPLATE.md`,替换占位符后写入项目根目录。

### 第二步:执行转换

进入实际转换前 `ultrathink`:先确认设计稿宽度、检查 rpx 工具函数是否已封装、识别项目是否为 Monorepo(`pnpm-workspace.yaml` 或 workspaces 字段)。这三项判断错了后续每次转换都要返工。

1. 读取 Taro 源码,分析组件结构
2. 按场景加载对应转换规则(见下方索引)
3. 遇到问题查阅对应子域 known-issues 文件
4. 完成后更新 `taro2rnTODO.md`

## 转换规则(按需加载)

### 主要转换规则

| 转换场景 | 加载文件 | 包含内容 |
|---------|---------|---------|
| **规则索引 + 速查** | [references/TRANSFORM_RULES.md](references/TRANSFORM_RULES.md) | 最常用映射速查表(先读这个) |
| **组件转换** | [references/components.md](references/components.md) | 基础组件映射、事件处理、Portal/RichText/WebView、导入汇总 |
| **API / 生命周期** | [references/api.md](references/api.md) | 路由、存储、网络、UI 反馈、页面生命周期、平台代码 |
| **样式转换** | [references/styles.md](references/styles.md) | rpx 单位、Flex、文字/边框/定位、不支持的 CSS、动态宽度 |
| **IM 聊天页面** | [references/im-layout.md](references/im-layout.md) | 键盘避让、FlatList inverted、滚动监听、分页加载 |

### 已知问题(按子域加载,不要一次性读全部)

| 子域 | 加载文件 | 适用场景 |
|---------|---------|---------|
| **依赖兼容** | [references/known-issues-dependencies.md](references/known-issues-dependencies.md) | 渐变、安全区、Picker、DatePicker、常用依赖清单 |
| **样式差异** | [references/known-issues-styles.md](references/known-issues-styles.md) | rpx 设计稿、Image、阴影、Flex 宽度、模糊效果 |
| **API / 键盘 / React 版本** | [references/known-issues-api.md](references/known-issues-api.md) | API 差异、ScrollView 监听、IM 键盘、React 版本冲突 |
| **Monorepo** | [references/known-issues-monorepo.md](references/known-issues-monorepo.md) | pnpm 依赖提升、CocoaPods、Metro、Monorepo React 冲突 |
| **(索引/兼容)** | [references/KNOWN_ISSUES.md](references/KNOWN_ISSUES.md) | 子域路由索引(旧链接仍可用) |

**加载策略**:先读 TRANSFORM_RULES.md 速查表 → 不够用时按场景加载详细 references → 遇到具体问题只加载对应子域的 known-issues-*.md。

## 转换工作流

```
1. 处理 taro2rnTODO.md → 初始化或读取现有进度
2. 确认设计稿标准     → 750 用 rpx(),1242 用 rpx1242()
3. 读取 Taro 源码     → 分析组件结构
4. 应用转换规则       → 按需加载对应 references
5. 处理特殊情况       → 按子域查阅 known-issues-*.md
6. 更新 taro2rnTODO.md → 标记完成
```

## 文档结构

```
taro2rn/
├── SKILL.md                              # 入口(本文件)
├── references/
│   ├── TRANSFORM_RULES.md                # 规则索引 + 速查表
│   ├── components.md                     # 组件映射详细规则
│   ├── api.md                            # API/生命周期映射
│   ├── styles.md                         # 样式转换规则
│   ├── im-layout.md                      # IM 页面布局方案
│   ├── KNOWN_ISSUES.md                   # 已知问题索引(旧链接兼容)
│   ├── known-issues-dependencies.md      # 依赖兼容
│   ├── known-issues-styles.md            # 样式差异
│   ├── known-issues-api.md               # API/键盘/React 版本
│   └── known-issues-monorepo.md          # Monorepo 特有
└── assets/
    └── TODO_TEMPLATE.md                  # 项目进度模板
```
