---
name: taro2rn
description: "Taro 小程序代码转换为 React Native 代码的完整工具包。包含组件映射、API 映射、样式转换规则和已知问题解决方案。Trigger when: (1) user wants to convert Taro code to React Native, (2) user says '转 RN' or '转换 RN' or 'taro to rn' or 'taro2rn', (3) user asks about Taro/RN component mapping or style conversion, (4) user encounters issues during Taro → RN migration."
---

# Taro to React Native 转换

## 快速开始

### 第一步：处理 taro2rnTODO.md

检查项目根目录是否存在 `taro2rnTODO.md`：

| 情况 | 操作 |
|------|------|
| **存在** | 读取进度，跳至第二步 |
| **不存在** | 执行初始化流程（见下方） |

#### 初始化流程

1. **询问用户配置**：

| 配置项 | 默认值 | 说明 |
|--------|--------|------|
| 项目名称 | 当前目录名 | 标识项目 |
| 设计稿宽度 | 750 | 750 或 1242 |
| RN 应用路径 | . | React Native 应用相对路径 |

2. **基于模板生成 taro2rnTODO.md**：读取 `assets/TODO_TEMPLATE.md`，替换占位符后写入项目根目录。

### 第二步：执行转换

1. 读取 Taro 源码，分析组件结构
2. 按场景加载对应转换规则（见下方索引）
3. 遇到问题查阅 [references/KNOWN_ISSUES.md](references/KNOWN_ISSUES.md)
4. 完成后更新 `taro2rnTODO.md`

## 转换规则（按需加载）

| 转换场景 | 加载文件 | 包含内容 |
|---------|---------|---------|
| **组件转换** | [references/components.md](references/components.md) | 基础组件映射、事件处理、Portal/RichText/WebView、导入汇总 |
| **API / 生命周期** | [references/api.md](references/api.md) | 路由、存储、网络、UI 反馈、页面生命周期、平台代码 |
| **样式转换** | [references/styles.md](references/styles.md) | rpx 单位、Flex、文字/边框/定位、不支持的 CSS、动态宽度 |
| **IM 聊天页面** | [references/im-layout.md](references/im-layout.md) | 键盘避让、FlatList inverted、滚动监听、分页加载 |
| **已知问题** | [references/KNOWN_ISSUES.md](references/KNOWN_ISSUES.md) | 扩展依赖、样式差异、API 差异、Monorepo 问题 |
| **规则索引 + 速查** | [references/TRANSFORM_RULES.md](references/TRANSFORM_RULES.md) | 最常用映射速查表 |

**加载策略**：先读 TRANSFORM_RULES.md 速查表，不够用时再加载对应场景的详细文件。

## 转换工作流

```
1. 处理 taro2rnTODO.md → 初始化或读取现有进度
2. 确认设计稿标准     → 750 用 rpx()，1242 用 rpx1242()
3. 读取 Taro 源码     → 分析组件结构
4. 应用转换规则       → 按需加载对应 references
5. 处理特殊情况       → 查阅 KNOWN_ISSUES.md
6. 更新 taro2rnTODO.md → 标记完成
```

## 文档结构

```
taro2rn/
├── SKILL.md                         # 入口（本文件）
├── references/
│   ├── TRANSFORM_RULES.md           # 规则索引 + 速查表
│   ├── components.md                # 组件映射详细规则
│   ├── api.md                       # API/生命周期映射
│   ├── styles.md                    # 样式转换规则
│   ├── im-layout.md                 # IM 页面布局方案
│   └── KNOWN_ISSUES.md              # 已知问题和解决方案
└── assets/
    └── TODO_TEMPLATE.md             # 项目进度模板
```
