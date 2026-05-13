# Taro → React Native 已知问题 · API / 生命周期 / 键盘 / React 版本

本文档记录 Taro 转 React Native 过程中 **API 差异、滚动监听、IM 键盘处理、React 版本冲突**相关的问题与解决方案。

> 其他子域:[依赖兼容](known-issues-dependencies.md) · [样式差异](known-issues-styles.md) · [Monorepo](known-issues-monorepo.md)

---

## 1. API 差异

### 1.1 事件中心

**问题描述:** Taro 的 `eventCenter` 需要替换为 RN 版本

**原始代码(Taro):**

```tsx
import {eventCenter} from '@tarojs/taro';
eventCenter.on('event', handler);
```

**RN 代码:**

```tsx
// 使用项目封装的事件中心,或使用第三方库如 eventemitter3
import {eventCenter} from 'your-rn-core-package';
eventCenter.on('event', handler);
```

---

### 1.2 页面导航

**问题描述:** Taro 路由需要替换为 React Navigation

**原始代码(Taro):**

```tsx
import Taro from '@tarojs/taro';
Taro.navigateTo({url: '/pages/detail/index?id=1'});
```

**RN 代码:**

```tsx
import {useNavigation} from '@react-navigation/native';
const navigation = useNavigation();
navigation.navigate('Detail', {id: 1});
```

---

### 1.3 键盘收起

**问题描述:** 点击页面其他区域时需要手动收起键盘

**解决方案:**

```tsx
import {Keyboard} from 'react-native';

// 在全局触摸监听中调用
const handleGlobalTouch = () => {
    Keyboard.dismiss();
};
```

---

## 2. ScrollView 滚动监听与分页加载

### 2.1 问题描述

RN 不支持 Web/小程序的 `IntersectionObserver` API,需要替代方案实现滚动锚点监听和分页加载。

**典型场景:**

- 滚动到底部自动加载更多数据
- 监听某个元素进入/离开可视区域
- 无限滚动列表

### 2.2 解决方案

**方案 A:FlatList onEndReached(推荐用于简单列表)**

```tsx
<FlatList
    data={items}
    renderItem={({item}) => <Item {...item} />}
    onEndReached={loadMore}
    onEndReachedThreshold={0.5} // 距离底部 50% 时触发
/>
```

**优点:** 官方推荐,性能优化好 **缺点:** 仅适用于 FlatList,复杂布局不适用

---

**方案 B:ScrollView onScroll + View onLayout(推荐用于复杂布局)**

适用于需要在任意 ScrollView 布局中监听元素可见性的场景。

**实现原理:**

1. ScrollAnchor 通过 `onLayout` 获取自身位置(anchorTop)
2. ScrollView 通过 `onScroll` 获取滚动信息(scrollY, layoutHeight)
3. 通过 eventCenter 传递滚动信息(保持解耦)
4. ScrollAnchor 计算可见性并触发回调

**核心代码:**

```tsx
// ScrollAnchor 组件
const ScrollAnchor = ({onAnchorVisibleChange}) => {
    const [anchorTop, setAnchorTop] = useState(0);
    const previousVisibleRef = useRef(false);

    // 获取锚点位置
    const handleLayout = event => {
        const {y} = event.nativeEvent.layout;
        setAnchorTop(y);
    };

    // 监听滚动信息
    useEffect(() => {
        const checkVisibility = scrollInfo => {
            const {scrollY, layoutHeight} = scrollInfo;
            const isVisible = anchorTop > 0 && anchorTop <= scrollY + layoutHeight;

            if (isVisible !== previousVisibleRef.current) {
                previousVisibleRef.current = isVisible;
                onAnchorVisibleChange?.(isVisible);
            }
        };

        const debouncedCheck = debounce(checkVisibility, 300, {
            leading: false,
            trailing: true
        });

        eventCenter.on('scrollAnchor:scroll', debouncedCheck);
        return () => {
            eventCenter.off('scrollAnchor:scroll', debouncedCheck);
            debouncedCheck.cancel();
        };
    }, [anchorTop, onAnchorVisibleChange]);

    return <View onLayout={handleLayout}>{children}</View>;
};

// 父组件使用
const MyList = () => {
    const handleScroll = useCallback(event => {
        const {contentOffset, contentSize, layoutMeasurement} = event.nativeEvent;
        eventCenter.trigger('scrollAnchor:scroll', {
            scrollY: contentOffset.y,
            contentHeight: contentSize.height,
            layoutHeight: layoutMeasurement.height
        });
    }, []);

    return (
        <ScrollView onScroll={handleScroll} scrollEventThrottle={16}>
            {items.map(item => (
                <Item key={item.id} {...item} />
            ))}
            {hasMore && (
                <ScrollAnchor
                    onAnchorVisibleChange={visible => {
                        if (visible) loadMore();
                    }}
                >
                    <Text>加载中...</Text>
                </ScrollAnchor>
            )}
        </ScrollView>
    );
};
```

**关键配置:**

- `scrollEventThrottle={16}`:必须设置,控制滚动事件频率(约 60fps)
- 防抖延迟 300ms:避免滚动过程中频繁触发
- eventCenter:实现解耦的事件驱动架构

---

## 3. IM 页面键盘处理

### 3.1 KeyboardAvoidingView 配置

**问题描述:** IM 聊天页面输入框聚焦时,键盘需要将整个输入区域(OperatingArea)推上去,而不是遮挡。

**解决方案:** 使用 RN 内置 `KeyboardAvoidingView`

```tsx
import {KeyboardAvoidingView, Platform} from 'react-native';
import {useSafeAreaInsets} from 'react-native-safe-area-context';

const insets = useSafeAreaInsets();

<KeyboardAvoidingView
    style={{flex: 1}}
    behavior={Platform.OS === 'ios' ? 'padding' : 'height'}
    keyboardVerticalOffset={Platform.OS === 'ios' ? 49 + insets.bottom : 49}
>
    <SessionContent style={{flex: 1}} />
    <OperatingArea />
</KeyboardAvoidingView>;
```

**偏移量计算:**

| 场景     | iOS offset                  | Android offset |
| -------- | --------------------------- | -------------- |
| 全屏页面 | `0`                         | `0`            |
| Tab 页面 | `49 + insets.bottom`        | `49`           |
| 有导航栏 | `navHeight + insets.bottom` | `navHeight`    |

- `49` = Tab Bar 标准高度
- `insets.bottom` = Home Indicator 高度(iPhone X+ 约 34px)
- Android 使用 `behavior="height"` 配合 `adjustResize`

---

## 4. React 版本冲突

### 4.1 问题描述

**错误信息:**

```
TypeError: Cannot read property 'ReactCurrentDispatcher' of undefined
```

**根因:** 项目中存在多个 React 实例,可能由于:

- 不同包依赖了不同版本的 React
- 原生依赖解析到了错误的 React 版本

### 4.2 解决方案

**1. package.json 添加 overrides/resolutions:**

```json
{
    "resolutions": {
        "react": "19.2.0"
    }
}
```

**2. Metro 配置强制 React 解析(metro.config.js):**

```javascript
const localReact = path.resolve(projectRoot, 'node_modules/react');

resolver: {
    resolveRequest: (context, moduleName, platform) => {
        if (moduleName === 'react' || moduleName.startsWith('react/')) {
            const subPath =
                moduleName === 'react' ? 'index.js' : moduleName.replace('react/', '') + '.js';
            return {filePath: path.resolve(localReact, subPath), type: 'sourceFile'};
        }
        return context.resolveRequest(context, moduleName, platform);
    };
}
```

> Monorepo 场景下的 React 版本冲突另见 [known-issues-monorepo.md](known-issues-monorepo.md)
