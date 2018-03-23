---
title: Complications for watchOS
date: 2017-07-18 17:40:28
tags:
---

因為最近有工作上的需求，就將 Apple 官方文件對 Complications for watchOS 做個簡單的整理。

## Complications 簡介

是一個在 Apple Watch 上的小元素，可以提供快速存取 Watch App，使用者可以在 Apple Watch 上客製自己想要的介面，在每種錶面上會提供大小跟數量不同的 Complications，從 Apple 的文件可以發現，即使 Watch App 沒有 Complication 的 idea，Apple 還是建議可以做個簡單的執行功能。

<!-- more -->

>**Note**
> 
>Apps are not required to provide a complication; however, they are highly recommended, even if they just act as a launcher for the app. Having a complication on the active watch face provides the following benefits:

>* It gives your app the opportunity to provide vital information to the user whenever they look at the watch.

>* It keeps your app suspended in memory. This lets the system rapidly wake your app when the user taps on the complication.

>*  It gives your app a larger budget for background tasks.

### Complication Families

在每種錶面上會有不同的 Family，ClockKit 將透過你選擇的 Template ，來渲染你的資料。

以下是現行 Watch OS 3 擁有的 Family:

```swift
public enum CLKComplicationFamily : Int {

    case modularSmall

    case modularLarge

    case utilitarianSmall

    @available(watchOS 3.0, *)
    case utilitarianSmallFlat /* subset of UtilitarianSmall */

    case utilitarianLarge

    case circularSmall

    @available(watchOS 3.0, *)
    case extraLarge
}
```

### How Complications Work

為了讓使用者可以得到即時的 Complication 還有電源使用量降低，所以 ClockKit 會提前要求開發者目前可以提供的所有可用資料並渲染，ClockKit 會將其快取並在需要他的時候渲染。

透過 `CLKComplicationDataSource` 來提供過去、現在、未來的資料。

以時間軸的方式來讓使用者可以獲得更好的體驗，當時間到時，ClockKit 會去更新 Complication，另一個優點是當使用者使用時間旅行功能時，可以去回顧或預覽 Complication。

## Managing Complications

要支援 Complication，有以下幾點：

1. 定義出你想要顯現的資料
2. 對每個 Family 選擇你要的 Templates
3. 執行 `CLKComplicationDataSource` 物件
4. 配置 WatchKit extension

### Designing the Complication

再開始前，需要決定 Complication 內容，並考量以下幾點：

* 資料是否有 Template 可用?
	- 因為空間是有限的，大部分的 Template 僅僅支援少量的文字或者一個小圖片。
* 是否已經使用推播 (notifications) 來傳達即時訊息？ 
	- Complication 是以一個比較不明顯的方式來傳達即時訊息。
* 可以提前提供多少數據？
	- 如果你的資料頻繁更新，可能無法提供足夠的資訊。
	- 更糟糕的是，如果太頻繁更新，可能會 exceed your background execution 或 transfer budgets 。

Watch App 的背景工作執行在每個小時內是有限的，也可以透過手機傳送資料到 Watch app 上，可是一天只有 50 Complication 轉移次數，因此需要在這些限制下去提供有用的資訊

### Implementing the Data Source Object

ClockKit 的互動方式是透過遵守 `CLKComplicationDataSource` 的物件，可以定義這個物件，但不可以在 runtime 時初始化，透過 Xcode project settings 來指定物件名稱，ClockKit 會在適當的時間來初始化 (透過 init 方法)。
 
不要再此物件去做延遲傳遞資料的事情，例如計算或者從網路取得資料。如果真的需要的話，就從 iOS app 或者 WatchKit extension 來做，並在某個可以存取的地方暫存這些資料，DataSource 物件只做去暫存的地方取得並且放入對應的格式。

更詳盡的 DataSource 物件請參考 [CLKComplicationDataSource Protocol Reference](https://developer.apple.com/documentation/clockkit/clkcomplicationdatasource)。

### Providing Your Timeline Data

當你選定好 Template 後，使用 `CLKTextProvider` 或 `CLKImageProvider` 來提供值。因為 Provider 會在適當的時候將資料顯現出來。

[Apple 官方範例](https://developer.apple.com/library/content/documentation/General/Conceptual/WatchKitProgrammingGuide/ManagingComplications.html#//apple_ref/doc/uid/TP40014969-CH28-SW1)

### Updating Your Complication Data
在 Runtime 下有以下幾種方式可以更新資料：

* 當 WatchKit Extension 在跑時，直接明確的更新
* 排時程後在背景執行更新
* 透過 iOS app 來傳輸資料
* 透過推播來更新

透過 `CLKComplicationServer` 的 `reloadTimelineForComplication:` 或  `extendTimelineForComplication:` 來更新時間軸，`reloadTimelineForComplication:` 是全部取代並刪除原本的時間軸，而 `extendTimelineForComplication:` 是在原本的時間軸上增加資料。

#### 注意要點

* 要確保 Watch App、Complications 及 notifications 三者都在同一個狀態。
* 如果是可以預測的時間，可以透過背景執行來更新，更新完後計劃下一次的更新。
* 如果很複雜的資料，就透過 iOS App 收集完後在傳遞給手錶。

### Providing Placeholder Templates

在使用者客製錶面時，需要提供一個預設的替代字符。ClockKit 只會在安裝的時候要求一次並把它給存下來，之後除非更新或是重新安裝，預設的替代字符都不會改變。

可以使用這兩個方法，`getPlaceholderTemplateForComplication:withHandler:`或 `getLocalizableSampleTemplateForComplication:withHandler:`。

P.S. 筆記一下，如果沒有的話，客製時會看到一片空白。（之前找很久才找到 ... 果然不看文件自食其果）

## Configuring Your Xcode Project’s Complication Support

Complication 在 Info.plist 相關的主要就這兩個 Key 值 `CLKComplicationsPrincipalClass` 及 `CLKSupportedComplicationFamilies`，稍微自己對應一下或查一下應該就能懂。

## Complications and the Watch App Gallery

Watch OS 3 後，使用者可以透過 Apple Watch app 來客製他們的錶面，因此需要提供一個 Complication Bundle 來在 App 內預覽，詳細操作請參考 [Adding Complications to the Gallery](https://developer.apple.com/documentation/clockkit/adding_complications_to_the_gallery?language=objc) 。

## Reference

[Apple 官方文件](https://developer.apple.com/library/content/documentation/General/Conceptual/WatchKitProgrammingGuide/ComplicationEssentials.html#//apple_ref/doc/uid/TP40014969-CH27-SW1)