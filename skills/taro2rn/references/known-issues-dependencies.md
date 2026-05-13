# Taro → React Native 已知问题 · 依赖兼容

本文档记录 Taro 转 React Native 过程中**第三方依赖**相关的兼容性问题与解决方案,以及常用依赖清单。

> 其他子域:[样式差异](known-issues-styles.md) · [API/生命周期](known-issues-api.md) · [Monorepo](known-issues-monorepo.md)

---

## 1. 扩展依赖

### 1.1 react-native-linear-gradient

**问题描述:** React Native 不支持 CSS 渐变(`linear-gradient`、`radial-gradient` 等)

**原始代码(Taro/CSS):**

```css
.sendBtn {
    background: linear-gradient(to left, #00d3ea, #00cfa3);
}
```

**解决方案:** 使用 `react-native-linear-gradient` 库

```bash
pnpm add react-native-linear-gradient
cd ios && pod install  # iOS 需要安装原生依赖
```

**RN 代码:**

```tsx
import LinearGradient from 'react-native-linear-gradient';

<LinearGradient
    colors={['#00d3ea', '#00cfa3']}
    start={{x: 1, y: 0}}
    end={{x: 0, y: 0}}
    style={styles.sendBtn}
>
    <Text>发送</Text>
</LinearGradient>;
```

**渐变方向映射:**

| CSS 方向 | RN start | RN end |
|---------|----------|--------|
| to left | {x: 1, y: 0} | {x: 0, y: 0} |
| to right | {x: 0, y: 0} | {x: 1, y: 0} |
| to top | {x: 0, y: 1} | {x: 0, y: 0} |
| to bottom | {x: 0, y: 0} | {x: 0, y: 1} |

---

### 1.2 react-native-safe-area-context

**问题描述:** React Native 需要处理刘海屏、底部安全区域等

**解决方案:** 使用 `react-native-safe-area-context` 库

```bash
pnpm add react-native-safe-area-context
cd ios && pod install
```

**RN 代码:**

```tsx
import {SafeAreaView} from 'react-native-safe-area-context';

<SafeAreaView style={styles.container} edges={['bottom']}>
    {children}
</SafeAreaView>;
```

---

### 1.3 @quidone/react-native-wheel-picker

**问题描述:**
第三方 UI 库的 Picker 组件在 RN 中没有直接替代。需要实现支持多列联动的滚轮选择器(如省市区三级联动)。

**解决方案:** 使用 `@quidone/react-native-wheel-picker` 实现跨平台滚轮选择器

```bash
pnpm add @quidone/react-native-wheel-picker
cd ios && pod install  # iOS 需要安装原生依赖
```

**使用场景:** 地址选择器(PickerAddress)、多列滚轮选择场景

**RN 代码:**

```tsx
import {WheelPicker} from '@quidone/react-native-wheel-picker';

<WheelPicker
    value={selectedIndex}
    onValueChanging={setSelectedIndex}
    itemHeight={44}
    visibleCount={5}
>
    {data.map((item, index) => (
        <WheelPicker.Item key={item.id} value={index} label={item.name} />
    ))}
</WheelPicker>;
```

**常用属性:**

| 属性 | 类型 | 说明 |
|------|------|------|
| `value` | `number` | 当前选中索引 |
| `onValueChanging` | `(index: number) => void` | 滚动变化回调 |
| `itemHeight` | `number` | 每项高度 |
| `visibleCount` | `number` | 可见项数量 |
| `itemTextStyle` | `TextStyle` | 文本样式 |

---

### 1.4 react-native-date-picker

**问题描述:** Taro 的 `Picker mode="date"` 或第三方日期选择器在 RN 中无直接对应。官方 `@react-native-community/datetimepicker` iOS/Android 样式差异大。

**解决方案:** 使用 `react-native-date-picker` 实现跨平台一致的滚轮体验

```bash
pnpm add react-native-date-picker
cd ios && pod install  # iOS 需要安装原生依赖
```

**使用场景:** 日期选择器(PickerDate)

**RN 代码:**

```tsx
import DatePicker from 'react-native-date-picker';

<DatePicker
    date={selectedDate}
    onDateChange={setSelectedDate}
    mode='date'
    locale='zh'
    minimumDate={new Date('1900-01-01')}
    maximumDate={new Date()}
    androidVariant='iosClone' // Android 使用 iOS 风格滚轮
    fadeToColor='#ffffff'
    textColor='#333333'
/>;
```

**常用属性:**

| 属性 | 类型 | 说明 |
|------|------|------|
| `date` | `Date` | 当前选中日期 |
| `onDateChange` | `(date: Date) => void` | 日期变化回调 |
| `mode` | `'date' \| 'time' \| 'datetime'` | 选择器模式 |
| `locale` | `string` | 语言区域(如 'zh') |
| `minimumDate` | `Date` | 最小可选日期 |
| `maximumDate` | `Date` | 最大可选日期 |
| `androidVariant` | `'iosClone' \| 'nativeAndroid'` | Android 样式风格 |

---

## 2. 原生依赖安装

安装新原生依赖后必须执行:

```bash
# iOS
cd ios && pod install
cd .. && npx react-native run-ios

# Android
npx react-native run-android
```

**常见错误:**

- `View config not found for component 'XXX'` - 原生模块未链接,需要 pod install 并重新编译

---

## 3. 常用依赖清单

| 依赖包                                    | 版本    | 用途                       |
| ----------------------------------------- | ------- | -------------------------- |
| react-native-linear-gradient              | ^2.8.0  | CSS 渐变替代               |
| @react-native-community/blur              | ^4.4.0  | 模糊效果 (backdrop-filter) |
| react-native-safe-area-context            | ^5.0.0  | 安全区域处理               |
| @react-navigation/native                  | ^7.0.0  | 页面导航                   |
| @react-navigation/native-stack            | ^7.0.0  | 原生栈导航                 |
| react-native-screens                      | ^4.0.0  | 原生屏幕优化               |
| jotai                                     | ^2.0.0  | 状态管理                   |
| @react-native-async-storage/async-storage | ^2.0.0  | 本地存储                   |
| react-native-image-picker                 | ^7.0.0  | 图片选择(相机/相册)      |
| react-native-image-viewing                | ^0.2.0  | 图片预览                   |
| react-native-video                        | ^6.18.0 | 视频播放                   |
| react-native-fast-image                   | ^8.6.3  | 高性能图片/GIF             |
