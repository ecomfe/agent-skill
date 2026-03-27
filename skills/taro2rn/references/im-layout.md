# IM/聊天页面布局

## 一、键盘避让布局

IM 页面典型结构：顶部导航 + 消息列表 + 底部输入框。键盘弹出时将输入框推上去。

```tsx
import {KeyboardAvoidingView, Platform} from 'react-native';
import {useSafeAreaInsets} from 'react-native-safe-area-context';

function IMScreen() {
  const insets = useSafeAreaInsets();
  return (
    <View style={{flex: 1}}>
      <NavBar />
      <KeyboardAvoidingView style={{flex: 1}}
        behavior={Platform.OS === 'ios' ? 'padding' : 'height'}
        keyboardVerticalOffset={Platform.OS === 'ios' ? 49 + insets.bottom : 49}>
        <View style={{flex: 1}}>
          <MessageList />
        </View>
        <InputArea />
      </KeyboardAvoidingView>
    </View>
  );
}
```

## 二、消息列表（FlatList inverted）

IM 消息列表使用 `inverted`，新消息在底部，滚动到顶部加载历史：

```tsx
<FlatList
  data={[...messages].reverse()}
  inverted
  renderItem={({item}) => <MessageItem {...item} />}
  onEndReached={loadHistory}
  onEndReachedThreshold={0.3}
  keyExtractor={item => item.id}
/>
```

## 三、keyboardVerticalOffset 计算

| 页面类型 | iOS | Android |
|---------|-----|---------|
| 全屏页面 | `0` | `0` |
| Tab 页面 | `49 + insets.bottom` | `49` |
| Stack（有导航栏） | `headerHeight` | `0`（adjustResize 自动处理） |

- `49` = Tab Bar 高度
- `insets.bottom` = 底部安全区（Home Indicator）
- Android 需在 `AndroidManifest.xml` 设置 `android:windowSoftInputMode="adjustResize"`

## 四、ScrollView 滚动监听与分页加载

RN 不支持 `IntersectionObserver`，替代方案：

### 方案 A：FlatList onEndReached（简单列表）

```tsx
<FlatList data={items} renderItem={({item}) => <Item {...item} />}
  onEndReached={loadMore} onEndReachedThreshold={0.5} />
```

### 方案 B：ScrollView + onLayout（复杂布局）

通过 eventCenter 解耦滚动监听：

```tsx
// ScrollAnchor 组件 - 获取位置 + 监听滚动事件
const ScrollAnchor = ({onAnchorVisibleChange, children}) => {
  const [anchorTop, setAnchorTop] = useState(0);
  const previousVisibleRef = useRef(false);

  useEffect(() => {
    const checkVisibility = ({scrollY, layoutHeight}) => {
      const isVisible = anchorTop > 0 && anchorTop <= scrollY + layoutHeight;
      if (isVisible !== previousVisibleRef.current) {
        previousVisibleRef.current = isVisible;
        onAnchorVisibleChange?.(isVisible);
      }
    };
    const debouncedCheck = debounce(checkVisibility, 300, {leading: false, trailing: true});
    eventCenter.on('scrollAnchor:scroll', debouncedCheck);
    return () => { eventCenter.off('scrollAnchor:scroll', debouncedCheck); debouncedCheck.cancel(); };
  }, [anchorTop, onAnchorVisibleChange]);

  return <View onLayout={e => setAnchorTop(e.nativeEvent.layout.y)}>{children}</View>;
};

// 父组件
<ScrollView onScroll={e => {
  const {contentOffset, contentSize, layoutMeasurement} = e.nativeEvent;
  eventCenter.trigger('scrollAnchor:scroll', {
    scrollY: contentOffset.y, contentHeight: contentSize.height,
    layoutHeight: layoutMeasurement.height
  });
}} scrollEventThrottle={16}>
  {items.map(item => <Item key={item.id} {...item} />)}
  {hasMore && <ScrollAnchor onAnchorVisibleChange={v => v && loadMore()}>
    <Text>加载中...</Text>
  </ScrollAnchor>}
</ScrollView>
```

关键配置：
- `scrollEventThrottle={16}` — 必须设置（~60fps）
- 防抖 300ms — 避免滚动中频繁触发
- eventCenter — 解耦的事件驱动架构
