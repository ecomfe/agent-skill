# Taro → React Native 已知问题 · Monorepo (pnpm / CocoaPods / Metro)

本文档记录 **pnpm/yarn workspaces 等 Monorepo 环境**下,Taro 转 React Native 特有的问题与解决方案。

> 其他子域:[依赖兼容](known-issues-dependencies.md) · [样式差异](known-issues-styles.md) · [API/生命周期](known-issues-api.md)

---

## 1. pnpm 依赖提升问题

**问题描述:** pnpm 将依赖提升到根目录 `node_modules`,但 RN 原生构建工具期望依赖在项目本地。

**解决方案:** 在 `postinstall` 脚本中创建符号链接

```javascript
// scripts/setup-symlinks.js
const SYMLINKS = [
    {local: 'apps/rn-app/node_modules/react-native', target: 'node_modules/react-native'},
    {
        local: 'apps/rn-app/node_modules/@react-native/gradle-plugin',
        target: 'node_modules/@react-native/gradle-plugin'
    }
];
```

---

## 2. CocoaPods .pnpm 路径问题

**问题描述:** CocoaPods 返回 `.pnpm` 内部虚拟路径,该路径不存在。

**解决方案:** 创建 `.pnpm` 目录符号链接

```javascript
{
    local: 'node_modules/.pnpm/hermes-compiler@0.14.0/node_modules/hermes-compiler',
    target: 'node_modules/hermes-compiler'
}
```

---

## 3. Metro 端口冲突

**解决方案:** 使用 `--no-packager` 参数分离 Metro

```json
{
    "scripts": {
        "rn:android": "pnpm --filter rn-app android -- --no-packager"
    }
}
```

---

## 4. React 版本冲突(Monorepo 特有)

**根因:** Monorepo 中多个包依赖不同 React 版本

**解决方案:** 根 package.json 添加 pnpm overrides

```json
{
    "pnpm": {
        "overrides": {
            "react-native-reanimated>react": "19.2.0"
        }
    }
}
```

> 非 Monorepo 通用的 React 版本冲突方案另见 [known-issues-api.md §4](known-issues-api.md#4-react-版本冲突)
