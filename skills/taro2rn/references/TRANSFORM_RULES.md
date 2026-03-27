# Taro to React Native 转换规则 — 索引

本文档为转换规则的导航入口。按需加载对应的 reference 文件，避免一次性加载全部内容。

> **项目特定配置** 请参考 `taro2rnTODO.md`。

## 按场景选择

| 转换场景 | 加载文件 | 包含内容 |
|---------|---------|---------|
| 组件转换 | [components.md](components.md) | 基础组件映射、事件处理、特殊组件（Portal/RichText/WebView）、导入汇总 |
| API / 生命周期 | [api.md](api.md) | 路由、存储、网络、UI 反馈、系统信息、页面生命周期、平台代码 |
| 样式转换 | [styles.md](styles.md) | rpx 单位、Flex 布局、文字/边框/定位、不支持的 CSS、动态宽度 |
| IM 聊天页面 | [im-layout.md](im-layout.md) | 键盘避让、FlatList inverted、滚动监听、分页加载 |
| 已知问题 | [KNOWN_ISSUES.md](KNOWN_ISSUES.md) | 扩展依赖、样式差异、API 差异、Monorepo 问题 |

## 转换速查

### 最常用映射

| Taro | RN | 备注 |
|------|-----|------|
| `<View onClick>` | `<Pressable onPress>` | 或 TouchableOpacity |
| `<Image src>` | `<Image source={{uri}}>` | 必须设宽高 |
| `<Input onInput={e => e.detail.value}>` | `<TextInput onChangeText={text}>` | |
| `className={styles.x}` | `style={styles.x}` | CSS Modules → StyleSheet |
| `rpx` 值 | `rpx(n)` / `rpx1242(n)` | 确认设计稿宽度 |
| `Taro.navigateTo` | `navigation.navigate` | React Navigation |
| `useDidShow` | `useFocusEffect` | @react-navigation |
| `useRouter().params` | `useRoute().params` | |
| `position: fixed` | `position: 'absolute'` | RN 无 fixed |
| `linear-gradient` | `<LinearGradient>` | react-native-linear-gradient |
