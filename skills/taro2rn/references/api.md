# API 映射 & 生命周期

## 一、路由导航

```tsx
// Taro
Taro.navigateTo({url: '/pages/detail/index?id=1'});
Taro.redirectTo({url: '/pages/home/index'});
Taro.navigateBack({delta: 1});
Taro.switchTab({url: '/pages/tab/index'});

// RN - React Navigation
import {useNavigation} from '@react-navigation/native';
const navigation = useNavigation();
navigation.navigate('Detail', {id: '1'});
navigation.replace('Home');
navigation.goBack();
navigation.navigate('Tab');
```

## 二、存储

```tsx
// Taro - 同步
Taro.setStorageSync('key', value);
const data = Taro.getStorageSync('key');
Taro.removeStorageSync('key');

// RN - AsyncStorage（异步）
import AsyncStorage from '@react-native-async-storage/async-storage';
await AsyncStorage.setItem('key', JSON.stringify(value));
const data = JSON.parse(await AsyncStorage.getItem('key'));
await AsyncStorage.removeItem('key');
```

## 三、网络请求

```tsx
// Taro
Taro.request({url, method: 'POST', data: {id: 1}, header: {'Content-Type': 'application/json'},
  success: res => console.log(res.data)});

// RN - fetch 或项目封装的 httpRequest
const response = await fetch(url, {
  method: 'POST', headers: {'Content-Type': 'application/json'},
  body: JSON.stringify({id: 1})});
const data = await response.json();
```

## 四、UI 反馈

```tsx
// Taro
Taro.showToast({title: '成功', icon: 'success', duration: 2000});
Taro.showLoading({title: '加载中'});
Taro.showModal({title: '提示', content: '确定删除？', showCancel: true});

// RN - react-native-toast-message 或项目封装
import Toast from 'react-native-toast-message';
Toast.show({type: 'success', text1: '成功'});
```

## 五、系统信息

```tsx
// Taro
const {screenWidth, screenHeight, platform} = Taro.getSystemInfoSync();

// RN
import {Dimensions, Platform, PixelRatio} from 'react-native';
const {width: screenWidth, height: screenHeight} = Dimensions.get('window');
const platform = Platform.OS; // 'ios' | 'android'
```

## 六、设备能力

```tsx
// Taro
Taro.vibrateShort();
Taro.setClipboardData({data: 'text'});

// RN
import {Vibration} from 'react-native';
import Clipboard from '@react-native-clipboard/clipboard';
Vibration.vibrate(50);
Clipboard.setString('text');
```

## 七、图片操作

```tsx
// Taro
const res = await Taro.chooseImage({count: 9, sourceType: ['album', 'camera']});
Taro.previewImage({urls: imageList, current: imageList[0]});

// RN
import {launchImageLibrary} from 'react-native-image-picker';
const result = await launchImageLibrary({mediaType: 'photo', selectionLimit: 9});
// 图片预览: react-native-image-viewing
```

## 八、页面生命周期

```tsx
// Taro
import {useDidShow, useDidHide} from '@tarojs/taro';
useDidShow(() => console.log('页面显示'));
useDidHide(() => console.log('页面隐藏'));

// RN - React Navigation hooks
import {useFocusEffect} from '@react-navigation/native';
useFocusEffect(useCallback(() => {
  console.log('页面显示');
  return () => console.log('页面隐藏');
}, []));
```

## 九、路由参数

```tsx
// Taro
import {useRouter} from '@tarojs/taro';
const {id, type} = useRouter().params;

// RN
import {useRoute} from '@react-navigation/native';
const {id, type} = useRoute().params;
```

## 十、下拉刷新

```tsx
// Taro
usePullDownRefresh(() => { fetchData().then(() => Taro.stopPullDownRefresh()); });

// RN - RefreshControl
const [refreshing, setRefreshing] = useState(false);
const onRefresh = async () => { setRefreshing(true); await fetchData(); setRefreshing(false); };
<ScrollView refreshControl={<RefreshControl refreshing={refreshing} onRefresh={onRefresh} />}>
```

## 十一、触底加载

```tsx
// Taro
useReachBottom(() => loadMore());

// RN - FlatList
<FlatList onEndReached={loadMore} onEndReachedThreshold={0.1} />

// 或 ScrollView
const handleScroll = (event) => {
  const {layoutMeasurement, contentOffset, contentSize} = event.nativeEvent;
  if (layoutMeasurement.height + contentOffset.y >= contentSize.height - 20) loadMore();
};
<ScrollView onScroll={handleScroll} scrollEventThrottle={400}>
```

## 十二、条件平台代码

```tsx
// Taro
if (process.env.TARO_ENV === 'weapp') { /* 微信 */ }

// RN
import {Platform} from 'react-native';
if (Platform.OS === 'ios') { /* iOS */ }

// 平台特定样式
container: { ...Platform.select({ ios: {paddingTop: 44}, android: {paddingTop: 0} }) }

// 平台特定文件: index.ios.tsx / index.android.tsx / index.native.tsx
```
