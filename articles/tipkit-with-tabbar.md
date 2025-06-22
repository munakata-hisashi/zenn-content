---
title: "TipKitがタブバーに表示できない"
emoji: "🐈"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Swift", "TipKit", "SwiftUI"]
published: true
---

## TipKitはタブバーへの表示ができない

アプリに新しいタブが追加されたとき、そのタブの使い方のヒントを表示するのにTipKitを使いたくなるかもしれません。しかし以下のようなコードを書いてもヒントは表示されません。

```swift
import SwiftUI
import TipKit

struct ContentView: View {
    var body: some View {
        TabView {
            VStack {
                Image(systemName: "globe")
                    .imageScale(.large)
                    .foregroundColor(.accentColor)
                Text("Hello, world!")

            }
            .tabItem {
                Image(systemName: "house")
                Text("ホーム")
                    .popoverTip(MyTip())
            }

            VStack {
                Image(systemName: "magnifyingglass")
                    .imageScale(.large)
                    .foregroundColor(.accentColor)
                Text("検索")

            }
            .tabItem {
                Image(systemName: "magnifyingglass")
                Text("検索")
            }
        }

    }
}

struct MyTip: Tip {
    var title: Text {
        Text("New")
    }
    var message: Text? {
        Text("新しい機能を試してみてください！")
    }
}
```


## 無理矢理出してみる

調べると https://stackoverflow.com/questions/79402691/tipkit-with-tabview でワークアラウンドが投稿されています。
以下のようなコードを書くとヒントを表示することができました。

```swift
struct ContentView: View {
    var body: some View {
        TabView {
            VStack {
                Image(systemName: "globe")
                    .imageScale(.large)
                    .foregroundColor(.accentColor)
                Text("Hello, world!")

            }
            .tabItem {
                Image(systemName: "house")
                Text("ホーム")
            }
            .popoverTip(MyTip())

            VStack {
                Image(systemName: "magnifyingglass")
                    .imageScale(.large)
                    .foregroundColor(.accentColor)
                Text("検索")

            }
            .tabItem {
                Image(systemName: "magnifyingglass")
                Text("検索")
            }
        }
        .background(alignment: .bottomTrailing) {
            HStack(spacing: 0) {
                Color.clear
                Color.clear
                    .popoverTip(MyTip())
            }
            .frame(height: 50)
        }
    }
}
```

![](/images/tipkit-with-tabbar-image.png)

## 注意点

iPadOS18ではデフォルトでタブバーが上部に配置されるため、上記のコードは機能しません。またiOS26からタブバーのデザインも変わることを考えると、タブバーにTipKitでヒントを出すのは避けた方が良さそうです。