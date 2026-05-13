# Taro → React Native 已知问题 · 索引

本文档已按子域拆分为 4 个文件,按需加载对应子域可避免一次性载入 20+ KB 全量内容。

## 按子域索引

| 子域 | 加载文件 | 包含内容 |
|------|---------|---------|
| **依赖兼容** | [known-issues-dependencies.md](known-issues-dependencies.md) | react-native-linear-gradient(渐变)、safe-area-context(安全区)、wheel-picker(滚轮)、date-picker(日期)、原生依赖安装、常用依赖清单 |
| **样式差异** | [known-issues-styles.md](known-issues-styles.md) | 设计稿尺寸(rpx/rpx1242)、Image 宽高/resizeMode、阴影(iOS shadow + Android elevation)、Flex 动态宽度(onLayout)、按钮 alignSelf、@react-native-community/blur 模糊 |
| **API / 生命周期 / 键盘 / React 版本** | [known-issues-api.md](known-issues-api.md) | eventCenter、页面导航(React Navigation)、键盘收起、ScrollView 滚动监听(ScrollAnchor 方案)、IM 页面 KeyboardAvoidingView、React 版本冲突(Metro resolveRequest) |
| **Monorepo (pnpm/CocoaPods/Metro)** | [known-issues-monorepo.md](known-issues-monorepo.md) | pnpm 依赖提升、CocoaPods .pnpm 路径、Metro 端口冲突、Monorepo 特有 React 版本冲突 |

## 加载建议

先快速浏览本索引,定位到子域后**只读取对应子文件**;不要一次性读全部 4 份。

---

## 更新记录

| 日期       | 更新内容                                                        |
| ---------- | --------------------------------------------------------------- |
| 2026-05-13 | 拆分为 4 个子域文件,本文件改为索引                              |
| 2026-01-21 | 简化文档结构,合并 Monorepo 问题到附录                          |
| 2026-01-08 | 添加 IM 页面键盘处理方案(KeyboardAvoidingView 配置)           |
| 2026-01-08 | 添加 React 版本冲突解决方案                                     |
| 2026-01-07 | 添加 Flex 布局宽度计算问题(onLayout 方案、alignSelf)          |
| 2026-01-07 | 添加 @react-native-community/blur 模糊效果                      |
| 2025-12-31 | 添加 ScrollView 滚动监听与分页加载方案(ScrollAnchor)          |
| 2025-12-29 | 添加 @quidone/react-native-wheel-picker 依赖(地址选择器)      |
| 2025-12-19 | 添加 react-native-image-picker、react-native-image-viewing 依赖 |
| 2025-12-17 | 初始文档创建,记录渐变、设计稿尺寸、Image、阴影、键盘等问题     |
