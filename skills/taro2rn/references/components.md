# 组件映射 & 事件处理

## 一、基础组件

| Taro 组件 | RN 组件 | 来源 | 说明 |
|-----------|---------|------|------|
| `<View>` | `<View>` | react-native | 直接对应 |
| `<Text>` | `<Text>` | react-native | 直接对应 |
| `<Image src={url}>` | `<Image source={{uri: url}}>` | react-native | src → source |
| `<ScrollView>` | `<ScrollView>` | react-native | 属性映射见下 |
| `<Input>` | `<TextInput>` | react-native | 属性映射见下 |
| `<Textarea>` | `<TextInput multiline>` | react-native | 添加 multiline |
| `<Button>` | `<TouchableOpacity>` | react-native | 需包装 Text |
| `<Switch>` | `<Switch>` | react-native | checked → value |
| `<Block>` | `<>` 或 `<Fragment>` | react | 空标签 |

## 二、View 点击事件

```tsx
// Taro
<View onClick={handleClick}>内容</View>

// RN - TouchableOpacity 或 Pressable
<TouchableOpacity onPress={handleClick}>
  <View>内容</View>
</TouchableOpacity>

<Pressable onPress={handleClick}>
  <View>内容</View>
</Pressable>
```

## 三、Image 组件

```tsx
// Taro
<Image src={imageUrl} mode="aspectFill" />

// RN - 必须设置宽高 + resizeMode
<Image source={{uri: imageUrl}} resizeMode="cover" style={{width: 100, height: 100}} />
```

mode 映射：`aspectFit` → `contain` | `aspectFill` → `cover` | `scaleToFill` → `stretch` | `widthFix` → `contain`

## 四、Input 组件

```tsx
// Taro
<Input value={value} type="number" placeholder="请输入" maxlength={10}
  disabled={false} onInput={(e) => setValue(e.detail.value)} onConfirm={handleSubmit} />

// RN
<TextInput value={value} keyboardType="numeric" placeholder="请输入" maxLength={10}
  editable={true} onChangeText={(text) => setValue(text)} onSubmitEditing={handleSubmit} />
```

type 映射：`text` → `default` | `number` → `numeric` | `digit` → `decimal-pad` | `tel` → `phone-pad`

## 五、ScrollView 属性

```tsx
// Taro
<ScrollView scrollY onScrollToLower={loadMore} refresherEnabled onRefresherRefresh={onRefresh}>

// RN
<ScrollView horizontal={false} onEndReached={loadMore} onEndReachedThreshold={0.1}
  refreshControl={<RefreshControl refreshing={refreshing} onRefresh={onRefresh} />}>
```

## 六、列表渲染

```tsx
// Taro - ScrollView + map
<ScrollView>{list.map(item => <View key={item.id}>{item.name}</View>)}</ScrollView>

// RN - 优先 FlatList
<FlatList data={list} keyExtractor={(item) => item.id}
  renderItem={({item}) => <View><Text>{item.name}</Text></View>} />
```

## 七、Swiper 组件

```tsx
// Taro
<Swiper autoplay interval={3000} circular indicatorDots onChange={e => setCurrent(e.detail.current)}>
  <SwiperItem>...</SwiperItem>
</Swiper>

// RN - react-native-swiper 或 react-native-pager-view
import Swiper from 'react-native-swiper';
<Swiper autoplay autoplayTimeout={3} loop showsPagination onIndexChanged={index => setCurrent(index)}>
  <View>...</View>
</Swiper>
```

## 八、Picker 组件

```tsx
// Taro
<Picker mode='selector' range={options} onChange={handleChange}><View>选择</View></Picker>

// RN - @react-native-picker/picker 或自定义 Modal
import {Picker} from '@react-native-picker/picker';
<Picker selectedValue={value} onValueChange={handleChange}>
  {options.map(opt => <Picker.Item key={opt} label={opt} value={opt} />)}
</Picker>

// 日期选择器: Taro <Picker mode="date"> → RN @react-native-community/datetimepicker
```

## 九、事件处理

### 事件对象

```tsx
// Taro - 事件值在 e.detail
<Input onInput={(e) => setValue(e.detail.value)} />
<ScrollView onScroll={(e) => console.log(e.detail.scrollTop)} />

// RN - 直接传递或在 nativeEvent
<TextInput onChangeText={(text) => setValue(text)} />
<ScrollView onScroll={(e) => console.log(e.nativeEvent.contentOffset.y)} />
```

### 阻止冒泡

```tsx
// Taro
<View onClick={(e) => { e.stopPropagation(); handleInner(); }}>

// RN - Pressable + onStartShouldSetResponder
<Pressable onPress={handleInner} onStartShouldSetResponder={() => true}>
```

## 十、特殊组件

### Portal / 弹窗

```tsx
// Taro
<Portal><Popup /></Portal>

// RN
import {Modal} from 'react-native';
<Modal visible={visible} transparent animationType='fade'>
  <View style={styles.overlay}><View style={styles.content}>{children}</View></View>
</Modal>
```

### RichText

```tsx
// Taro
<RichText nodes={htmlContent} />

// RN
import RenderHtml from 'react-native-render-html';
<RenderHtml contentWidth={screenWidth} source={{html: htmlContent}} />
```

### WebView

```tsx
// Taro
<WebView src='https://example.com' />

// RN
import {WebView} from 'react-native-webview';
<WebView source={{uri: 'https://example.com'}} />
```

## 十一、导入转换汇总

```tsx
// Taro
import Taro, {useDidShow, useRouter} from '@tarojs/taro';
import {View, Text, Image, ScrollView, Input, Button} from '@tarojs/components';

// RN
import {View, Text, Image, ScrollView, TextInput, TouchableOpacity,
  Platform, Dimensions, StyleSheet} from 'react-native';
import {useNavigation, useRoute, useFocusEffect} from '@react-navigation/native';
```
