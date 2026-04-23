>  最好先会 [React](React.md)

# 快速开始
创建 expo go 项目：
```bash
 pnpm create expo-app -t blank
```

两种调式方式：
+ 真机调式： 安卓系统下，下载 expo go 软件
+ 模拟器调式： [下载 Android Studio 和应用工具 - Android 开发者  \|  Android Developers](https://developer.android.com/studio?hl=zh-cn)

# 核心组件
[核心组件和API · React Native 中文网](https://reactnative.cn/docs/components-and-apis)
+ View 
+ Text
+ Image
+ ScrollView (滚动)
+ FlatList (列表)

# 布局
RN 默认是 flexbox 布局 —— flexDirection: column
RN 中使用 JS 对象（CSS IN JS 方案）
```js
const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#fff',
    alignItems: 'center',
    justifyContent: 'center',
  },
});

// 使用
style = { styles.container }
```

# 网络请求
React Native 默认使用 fetch api 
可参考 [Fetch](AJAX.md#Fetch)

# Router 跳转
[Manual installation - Expo Documentation](https://docs.expo.dev/router/installation/)

```js
npx expo install expo-router react-native-safe-area-context react-native-screens expo-linking expo-constants expo-status-bar
```

