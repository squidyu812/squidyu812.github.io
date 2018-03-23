---
title: 關於全域常數
date: 2017-05-18 17:15:53
tags:
---

## 源起
最早開始寫程式時，我都是直接寫一些字串在程式碼上面，常常會面臨到打錯字卻不自知，雖然 Copy 來 Copy 去，但有時候還是會不小心弄錯，當我在 RunTime 時就會發生很愚蠢的 Bug 或未知的情況。

<!-- more -->

例如：

```swift
UserDefaults.standard.set("value", forKey: "UserDefaultsKey")
UserDefaults.standard.synchronize()
	
let any = UserDefaults.standard.object(forKey: "UserDefaultsKey")
```

後來為了避免這些災難，我就改成這樣。

```swift
let userDefaultsKey = "UserDefaultsKey"
UserDefaults.standard.set("value", forKey: userDefaultsKey)
UserDefaults.standard.synchronize()

let any = UserDefaults.standard.object(forKey: userDefaultsKey)
```


## 後來

而我們有時候在不同的檔案需要用到同一個常數，這時候我們就會使用全域常數 (Global Constant)。

在 Swift 上定義全域常數很簡單，其實只要隨便一個檔案，定義全域常數即可。

```swift
//
//  Constants.swift
// 

let userDefaultsKey = "UserDefaultsKey"
```

其實這樣子在其他檔案上就可以呼叫到，但為了可讀性，我可以用一個 Struct 或 Enum 把它包起來。

例如：

```swift
// struct example
struct UserDefaultsKey {
    static let key1 = "UserDefaultsKey1"
    static let key2 = "UserDefaultsKey2"
}
```

或者

```swift
// enum example
enum UserDefaultsKey {
    static let key1 = "UserDefaultsKey1"
    static let key2 = "UserDefaultsKey2"
}
```

使用這兩者並無太大差異，但在 Ray Wenderlich 的 [swift-style-guide](https://github.com/raywenderlich/swift-style-guide#constants) 有提到使用 Enum 的好處。

>The advantage of using a case-less enumeration is that it can't accidentally be instantiated and works as a pure namespace.

## 結論

其實我之前都是使用 Struct，也是看資料的時候才發現 Enum 的好處，在之前的專案裡我不是沒包不然就是用 Struct，之後應該會開始試著用 Enum 去做全域常數的命名。

題外話一下，如果是 Objective-C 就是使用其他方式了。
