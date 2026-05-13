# Taro → React Native 已知问题 · 样式差异

本文档记录 Taro 转 React Native 过程中**样式系统**相关的兼容性问题与解决方案。

> 其他子域:[依赖兼容](known-issues-dependencies.md) · [API/生命周期](known-issues-api.md) · [Monorepo](known-issues-monorepo.md)

---

## 1. 设计稿尺寸

**问题描述:** 不同项目可能使用不同设计稿宽度(如 750、1242 等)

**重要:开始样式转换前,必须先确认以下事项:**

1. **询问确认视觉稿标准**:750 还是 1242?
2. **检查 rpx 工具函数是否已封装**

**Taro 配置示例:**

```js
// 750 设计稿(默认)
designWidth: 750

// 1242 设计稿
designWidth: 1242,
deviceRatio: {
    1242: 2 / 3.312
}
```

**解决方案:** 使用对应的 rpx 转换函数

```tsx
import {rpx, rpx1242, createRpx} from 'your-rn-core-package';

// 750 设计稿
padding: rpx(36);

// 1242 设计稿
padding: rpx1242(36);

// 自定义设计稿
const rpx640 = createRpx(640);
padding: rpx640(36);
```

**rpx 工具函数封装(如未存在则创建):**

```tsx
// src/utils/rpx.ts
import {Dimensions} from 'react-native';

const {width: screenWidth} = Dimensions.get('window');

/** 750 设计稿尺寸转换 */
export const rpx = (value: number): number => (value * screenWidth) / 750;

/** 1242 设计稿尺寸转换 */
export const rpx1242 = (value: number): number => (value * screenWidth) / 1242;

/** 自定义设计稿尺寸转换工厂函数 */
export const createRpx = (designWidth: number) => {
    return (value: number): number => (value * screenWidth) / designWidth;
};
```

---

## 2. Image 组件

**问题描述:** RN 的 Image 组件必须显式设置宽高,且需要指定 resizeMode

**原始代码(Taro):**

```tsx
<Image src={url} className={styles.icon} />
```

**RN 代码:**

```tsx
<Image
    source={{uri: url}}
    style={styles.icon} // 必须包含 width 和 height
    resizeMode='contain'
/>
```

**resizeMode 选项:**

- `cover` - 保持比例填充(可能裁剪)
- `contain` - 保持比例完整显示
- `stretch` - 拉伸填充
- `center` - 居中不缩放

**注意事项:**

- 远程图片必须设置宽高,否则不显示
- 建议添加 `onError` 回调处理加载失败

---

## 3. 阴影样式

**问题描述:** Android 和 iOS 阴影实现方式不同

**原始代码(CSS):**

```css
.card {
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}
```

**RN 代码:**

```tsx
const styles = StyleSheet.create({
    card: {
        // iOS 阴影
        shadowColor: '#000',
        shadowOffset: {width: 0, height: 2},
        shadowOpacity: 0.1,
        shadowRadius: 4,
        // Android 阴影
        elevation: 4
    }
});
```

---

## 4. Flex 布局宽度计算

### 4.1 动态宽度计算问题

**问题描述:** 使用 `Dimensions.get('window').width`
计算子元素宽度时,未考虑父容器的 padding/border,导致子元素超出容器换行。

**典型场景:** 多列网格布局(如 3 列服务项)

**错误方式:**

```tsx
const itemWidth = (SCREEN_WIDTH - containerPadding) / 3;
// 问题:未考虑外层容器的 padding/border
```

**解决方案:** 使用 `onLayout` 获取容器实际宽度

```tsx
const [containerWidth, setContainerWidth] = useState(0);

const handleLayout = (event: LayoutChangeEvent) => {
    setContainerWidth(event.nativeEvent.layout.width);
};

const itemWidth = containerWidth > 0
    ? Math.floor((containerWidth - padding * 2 - gap * 2) / 3)
    : 0;

<View style={styles.container} onLayout={handleLayout}>
    {containerWidth > 0 && items.map(...)}
</View>
```

**关键点:**

- 使用 `Math.floor()` 避免浮点精度问题
- `containerWidth > 0` 时才渲染子元素,避免首次渲染闪烁
- 配合 `justifyContent: 'space-between'` 自动分配间距

---

### 4.2 Flex 子元素宽度撑满问题

**问题描述:** Flex 布局中,子元素默认 `alignSelf: 'stretch'` 会撑满父容器宽度。

**典型场景:** 按钮/标签需要根据内容自适应宽度

**解决方案:** 添加 `alignSelf: 'flex-start'`

```tsx
// 错误:按钮撑满整行
buttonStyle: {
    backgroundColor: '#fff',
    borderRadius: 20,
}

// 正确:按钮宽度自适应内容
buttonStyle: {
    alignSelf: 'flex-start',  // 关键
    backgroundColor: '#fff',
    borderRadius: 20,
}
```

---

## 5. 模糊效果 (backdrop-filter)

### 5.1 @react-native-community/blur

**问题描述:** RN 不支持 CSS `backdrop-filter: blur()`

**解决方案:** 使用 `@react-native-community/blur`

```bash
pnpm add @react-native-community/blur
cd ios && pod install
```

**RN 代码:**

```tsx
import {BlurView} from '@react-native-community/blur';

<BlurView
    style={styles.blurContainer}
    blurType='light'
    blurAmount={30}
    reducedTransparencyFallbackColor='rgba(255,255,255,0.6)'
>
    {children}
</BlurView>;
```

**平台差异:**

| 特性 | iOS | Android |
|------|-----|---------|
| blurAmount 最大值 | 无限制 | 32 |
| VibrancyView | 支持 | 不支持 |
| 性能 | 优秀 | 良好 |

**blurType 选项:** `light`, `dark`, `xlight`, `prominent`, `regular`
