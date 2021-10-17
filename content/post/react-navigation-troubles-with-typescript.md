---
title: "React NavigationをTypescriptと一緒に使う際につまづいたところ"
date: 2021-10-09T19:31:57+09:00
draft: false
tags: ['react native', 'react navigation']
summary: "記述量が多くて少し辛かった"
---

最近業務でReact Nativeを書いているのですが、ナビゲーションのライブラリとして
React NavigationをTypescriptと一緒に使う際に色々大変だったので（主にドキュメントを読んでいないせい）
まとめておこうと思いこの記事を書いています。質問や気になる点があれば[@kmgk21444557](https://twitter.com/kmgk21444557)までご連絡ください。

アプリの作成、実行にはExpoを使っています。

サンプルアプリのリポジトリ：[https://github.com/kmgk/ReactNavigationExample](https://github.com/kmgk/ReactNavigationExample)

## 目次
- [環境](#%E7%92%B0%E5%A2%83)
- [テスト用アプリの作成](#%E3%83%86%E3%82%B9%E3%83%88%E7%94%A8%E3%82%A2%E3%83%97%E3%83%AA%E3%81%AE%E4%BD%9C%E6%88%90)
- [navigate('Hoge')でエラーが出る](#navigatehoge%E3%81%A7%E3%82%A8%E3%83%A9%E3%83%BC%E3%81%8C%E5%87%BA%E3%82%8B)
- [Bottom Tabs Navigator でヘッダーが二重に出る問題](#bottom-tabs-navigator-%E3%81%A7%E3%83%98%E3%83%83%E3%83%80%E3%83%BC%E3%81%8C%E4%BA%8C%E9%87%8D%E3%81%AB%E5%87%BA%E3%82%8B%E5%95%8F%E9%A1%8C)
  - [下のヘッダー(Tab.Screen)を消す](#%E4%B8%8B%E3%81%AE%E3%83%98%E3%83%83%E3%83%80%E3%83%BCtabscreen%E3%82%92%E6%B6%88%E3%81%99)
  - [上のヘッダー(TabNavigator)を消す](#%E4%B8%8A%E3%81%AE%E3%83%98%E3%83%83%E3%83%80%E3%83%BCtabnavigator%E3%82%92%E6%B6%88%E3%81%99)
- [参考文献](#%E5%8F%82%E8%80%83%E6%96%87%E7%8C%AE)

## 環境
```
node: v16.10.0
yarn: 1.22.4
expo-cli: 4.12.1
OS: android (Pixel 3a XL API R)
```
`package.json`
```package.json
{
  "dependencies": {
    "@react-navigation/bottom-tabs": "^6.0.7",
    "@react-navigation/native-stack": "^6.2.2",
    "expo": "~42.0.1",
    "expo-status-bar": "~1.0.4",
    "react": "16.13.1",
    "react-dom": "16.13.1",
    "react-native": "https://github.com/expo/react-native/archive/sdk-42.0.0.tar.gz",
    "react-native-safe-area-context": "3.2.0",
    "react-native-screens": "~3.4.0",
    "react-native-web": "~0.13.12"
  },
  "devDependencies": {
    "@babel/core": "^7.9.0",
    "@react-navigation/native": "^6.0.4",
    "@types/react": "~16.9.35",
    "@types/react-native": "~0.63.2",
    "typescript": "~4.0.0"
  },
}
```

## テスト用アプリの作成
expo-cliを用いてアプリを作成します。テンプレートは`blank (Typescript)`を選択しました。
```
$ expo init ReactNavigationExample
```
必要なパッケージをインストール
```
yarn add @react-navigation/native @react-navigation/native-stack @react-navigation/bottom-tabs
expo install react-native-screens react-native-safe-area-context react-native-pager-view
```

[Hello React Navigation](https://reactnavigation.org/docs/hello-react-navigation)を参考に、
`HomeScreen`と`HogeScreen`の2画面作成し、`createNativeStackNavigator`でナビゲーションを作成します。

`App.tsx`
```App.tsx
import { NavigationContainer, useNavigation } from '@react-navigation/native';
import { createNativeStackNavigator } from '@react-navigation/native-stack';
import React from 'react';
import { Button, StyleSheet, Text, View } from 'react-native';

const HomeStack = createNativeStackNavigator();

export default function App() {
  return (
    <NavigationContainer>
      <HomeStack.Navigator initialRouteName="Home">
        <HomeStack.Screen 
          name="Home" 
          component={HomeScreen} 
        />
        <HomeStack.Screen
          name="Hoge"
          component={HogeScreen}
        />
      </HomeStack.Navigator>
    </NavigationContainer>
  );
}

const HomeScreen: React.FC = () => {
  const navigation = useNavigation()

  return (
    <View style={styles.container}>
      <Text>HomeScreen</Text>
      <Button 
        title="Go To HogeScreen" 
        onPress={() => navigation.navigate('Hoge')} 
      />
    </View>
  )
}

const HogeScreen: React.FC = () => {
  return (
    <View style={styles.container}>
      <Text>HogeScreen</Text>
    </View>
  )
}

const styles = StyleSheet.create({
  ...
});
```

## `navigate('Hoge')`でエラーが出る
まずここでエラーが出ます。`onPress={() => navigation.navigate('Hoge')}`のnavigateの引数である文字列Hogeの型が正しくないようです。
```
型 'string' の引数を型 '{ key: string; params?: undefined; merge?: boolean | undefined; } | 
{ name: never; key?: string | undefined; params: never; merge?: boolean | undefined; }' 
のパラメーターに割り当てることはできません
```
'Hoge'が割り当てられる予定のnameパラメータの型がneverになっています。なんで。

ここで公式に[Type checking with TypeScript](https://reactnavigation.org/docs/typescript)というページがあることに気が付きます。
>To type check our route name and params, the first thing we need to do is to create an object type with mappings for route name to the params of the route. For example, say we have a route called Profile in our root navigator which should have a param userId:

とあるのでとりあえずルートをまとめたオブジェクト`RootStackParamList`を作成します。
```App.tsx
type RootStackParamList = {
  Home: undefined;
  Hoge: undefined;
}
```
型にundefinedを指定するとナビゲーションからパラメータを与えられません。逆にパラメータが欲しい場合はそのパラメータのマッピングを指定する必要があります。

これを`createNativeStackNavigator`に指定します。
```
const HomeStack = createNativeStackNavigator<RootStackParamList>();
```
こうすることでNavigator内部のScreenでnameが制限されます。
定義したルート名しか使えないのでtypoの心配もなくなりました。

![Navigator内部のScreenでnameが制限](/images/stack-screen-strict-name.png)

ただ、まだ`navigate('Hoge')`のエラーは出たままです。先程の記事の[Type checking screens](https://reactnavigation.org/docs/typescript#type-checking-screens)を見ると
>To type check our screens, we need to annotate the navigation prop and the route prop received by a screen. The navigator packages in React Navigation export a generic types to define types for both the navigation and route props from the corresponding navigator.

>The type takes 2 generics, the param list object we defined earlier, and the name of the current route. This allows us to type check route names and params which you're navigating using navigate, push etc. The name of the current route is necessary to type check the params in route.params and when you call setParams.

また、`useNavigation`をよく見てみると
```tsx
useNavigation<NavigationProp<ReactNavigation.RootParamList, ...
```
とあるので、いい感じにジェネリクスを指定します。今回は`createNativeStackNavigator`で
StackNavigatorを作成したので`NativeStackNavigationProp`を使用します。
```tsx
const navigation = useNavigation<NativeStackNavigationProp<RootStackParamList, 'Home'>>();
```
こうすることでエラーは消え、定義したルート名のみが指定できるようになりました。問題解決！

## Bottom Tabs Navigator でヘッダーが二重に出る問題
まずは[Bottom Tabs Navigator](https://reactnavigation.org/docs/bottom-tab-navigator)を
参考にBottom Tabsを用意します。
```tsx
type TabParamList = {
  Tab1: undefined;
  Tab2: undefined;
  Tab3: undefined;
}

const TabNavigator: React.FC = () => {
  return (
    <Tab.Navigator initialRouteName="Tab1">
      <Tab.Screen name="Tab1" component={Tab1Screen} />
      <Tab.Screen name="Tab2" component={Tab2Screen} />
      <Tab.Screen name="Tab3" component={Tab3Screen} />
    </Tab.Navigator>
  )
}

// Tab1Screenなどは省略・・・
```

先程のホーム画面から`TabNavigator`へ遷移するよう設定し、いざ遷移！

![ヘッダーが二重になっている](/images/double-header.png)

なぜかヘッダーが二重になっています。まずは下のヘッダー（タイトルがTab1の方）を消してみます。

### 下のヘッダー(Tab.Screen)を消す

これは`Tab.Navigator`のオプションで`headerShown`というパラメータがあるのでそれにfalseを渡してあげるだけです。

```tsx
<Tab.Navigator initialRouteName="Tab1" screenOptions={{headerShown: false}}>
```

### 上のヘッダー(TabNavigator)を消す

次に上のヘッダー（タイトルがTabの方）を消します。
先程のTab.Navigatorに渡したheaderShownは一度削除して、再びヘッダーが二重になるようにします。
ここで、TabNavigationを呼び出しているNavigationContainerの方を見てみます。

```tsx
<NavigationContainer>
  <HomeStack.Navigator initialRouteName="Home">
    ...
    <HomeStack.Screen
      name="Tab"
      component={TabNavigator}
    />
  </HomeStack.Navigator>
</NavigationContainer>
```

`HomeStack.Screen`のヘッダーが表示されているので、それを消せば解決します。
先程と同様headerShownを渡します。Screenは`options`でオプションを受け取るので注意。(Navigatorは`screenOptions`)

```tsx
<HomeStack.Screen
  name="Tab"
  component={TabNavigator}
  options={{
    headerShown: false
  }}
/>
```

これで上のヘッダーを削除することができました。ちなみに両方に`headerShown: false`を渡すとヘッダーがなくなります。当然ですね。

## 参考文献
- [Type checking with TypeScript - reactnavigation.org](https://reactnavigation.org/docs/typescript)
- [Bottom Tabs Navigator - reactnavigation.org](https://reactnavigation.org/docs/bottom-tab-navigator#headershown)