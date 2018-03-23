---
layout: hexo
title: 彈力 UIView
date: 2017-10-20 12:00:20
tags:
---

為了方便，這邊利用 UIButton 當成範例，也可以對 UIView 類別觸發彈力動畫

### 初步介紹

#### 利用 Method

```
animate(withDuration:delay:usingSpringWithDamping:initialSpringVelocity:options:animations:completion:)
```

<!-- more -->

[可參考 Apple 文件](https://developer.apple.com/documentation/uikit/uiview/1622594-animate)

#### Animation parameters

* withDuration: 動畫的持續時間
* delay: 多久開始動畫
* usingSpringWithDamping: 阻尼比，0 ~ 1 之間，簡單來說就是越靠近 0，晃動越大
* initialSpringVelocity: 初始的彈速，距離跟時間得速率比
 
>The initial spring velocity. For smooth start to the animation, match this value to the view’s velocity as it was prior to attachment.

> A value of 1 corresponds to the total animation distance traversed in one second. For example, if the total animation distance is 200 points and you want the start of the animation to match a view velocity of 100 pt/s, use a value of 0.5.

### 範例

ViewController.swift

```swift
class ViewController: UIViewController {

    @IBOutlet weak var springButton: SpringButton!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        springButton.layer.cornerRadius = 10
    }
    
    @IBAction func springAnimation(_ sender: SpringButton) {
        sender.beginSpringAnimation()
    }
}
```

SpringButton.swift

```swift
class SpringButton: UIButton {}

extension SpringButton: Springable {}
```

Springable.swift

```swift
protocol Springable {
    func beginSpringAnimation()
}

extension Springable where Self: UIView {
    func beginSpringAnimation() {
        transform = CGAffineTransform(scaleX: 0.1, y: 0.1)
        
        UIView.animate(
            withDuration: 2.0,
            delay: 0,
            usingSpringWithDamping: 0.2,
            initialSpringVelocity: 6,
            options: .allowUserInteraction,
            animations: { [weak self] in
                self?.transform = .identity
            },
            completion: nil)
    }
}
```

### 參考

[Spring button animation in iOS/Swift](http://evgenii.com/blog/spring-button-animation-with-swift/)
[Demo](https://github.com/squidyu812/SpringAnimation)