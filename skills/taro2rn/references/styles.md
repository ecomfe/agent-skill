# 样式转换

## 一、基本规则

```tsx
// Taro - CSS Modules (.module.less/.module.scss)
import styles from './index.module.less';
<View className={styles.container}>

// RN - StyleSheet
import {styles} from './styles';
<View style={styles.container}>
```

## 二、单位转换

**开始前必须确认：**
1. 视觉稿标准：750 还是 1242？
2. rpx 工具函数是否已封装？（检查 `taro2rnTODO.md` 中的路径）

| 视觉稿 | 原始值 | 转换函数 | 375 屏幕实际值 |
|--------|--------|---------|--------------|
| 750 | 24rpx | `rpx(24)` | 12pt |
| 750 | 750rpx | `rpx(750)` 或 `'100%'` | 375pt (全屏) |
| 1242 | 24rpx | `rpx1242(24)` | ~7.3pt |
| 1242 | 1242rpx | `rpx1242(1242)` 或 `'100%'` | 375pt (全屏) |

```less
/* Taro LESS */
.container { width: 750rpx; padding: 24rpx; font-size: 28rpx; margin: 20px; }
```

```tsx
// RN - 750 设计稿
import {rpx} from 'your-rn-core-package';
container: { width: '100%', padding: rpx(24), fontSize: rpx(28), margin: 20 }

// RN - 1242 设计稿
import {rpx1242} from 'your-rn-core-package';
container: { width: '100%', padding: rpx1242(40), fontSize: rpx1242(46), margin: 20 }
```

**rpx 工具函数封装（如项目未提供）：**

```tsx
import {Dimensions} from 'react-native';
const {width: screenWidth} = Dimensions.get('window');
export const rpx = (value: number): number => (value * screenWidth) / 750;
export const rpx1242 = (value: number): number => (value * screenWidth) / 1242;
export const createRpx = (designWidth: number) =>
  (value: number): number => (value * screenWidth) / designWidth;
```

## 三、Flex 布局

```less
/* Taro */
.row { display: flex; flex-direction: row; justify-content: space-between; align-items: center; }
```

```tsx
// RN - display: flex 是默认值，可省略
row: { flexDirection: 'row', justifyContent: 'space-between', alignItems: 'center' }
```

## 四、文字样式

```less
/* Taro */
.title { color: #333; font-size: 32rpx; font-weight: bold; line-height: 44rpx;
  text-align: center; text-overflow: ellipsis; white-space: nowrap; overflow: hidden; }
```

```tsx
// RN - 文字截断用 numberOfLines
title: { color: '#333', fontSize: rpx(32), fontWeight: 'bold', lineHeight: rpx(44), textAlign: 'center' }
<Text style={styles.title} numberOfLines={1}>标题</Text>
```

## 五、边框和圆角

```less
/* Taro */
.card { border: 1px solid #eee; border-radius: 16rpx; box-shadow: 0 4rpx 12rpx rgba(0,0,0,0.1); }
```

```tsx
// RN
card: {
  borderWidth: 1, borderColor: '#eee', borderRadius: rpx(16),
  shadowColor: '#000', shadowOffset: {width: 0, height: rpx(4)}, // iOS
  shadowOpacity: 0.1, shadowRadius: rpx(12),
  elevation: 4 // Android
}
```

## 六、定位

```less
/* Taro */
.overlay { position: fixed; top: 0; left: 0; right: 0; bottom: 0; z-index: 100; }
```

```tsx
// RN - 无 fixed，用 absolute
overlay: { ...StyleSheet.absoluteFillObject, zIndex: 100 }
```

## 七、不支持的 CSS 特性

| CSS 特性 | RN 替代方案 |
|---------|-----------|
| `::before` / `::after` | 添加实际的 View/Text 子元素 |
| `:hover` / `:active` | Pressable 的 style 函数 |
| `transition` | Animated API 或 Reanimated |
| `animation` | Animated API 或 Lottie |
| `background-image` | ImageBackground 组件 |
| `linear-gradient` | react-native-linear-gradient |
| `backdrop-filter` | @react-native-community/blur |
| `@media` | Dimensions API 条件判断 |
| `calc()` | 在 JS 中计算 |

## 八、动态宽度计算

```tsx
// ❌ 硬编码屏幕宽度
const itemWidth = (SCREEN_WIDTH - padding) / 3;

// ✅ 使用 onLayout 获取实际容器宽度
const [containerWidth, setContainerWidth] = useState(0);
<View onLayout={e => setContainerWidth(e.nativeEvent.layout.width)}>
  {containerWidth > 0 && items.map(item =>
    <View style={{width: Math.floor(containerWidth / 3)}} />)}
</View>
```

## 九、Flex 子元素宽度撑满

```tsx
// ❌ 按钮撑满整行
buttonStyle: { backgroundColor: '#fff', borderRadius: 20 }

// ✅ 宽度自适应内容
buttonStyle: { alignSelf: 'flex-start', backgroundColor: '#fff', borderRadius: 20 }
```
