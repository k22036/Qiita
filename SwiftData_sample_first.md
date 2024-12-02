<!-- タイトル
SwiftDataをView以外の場所で扱う -->

# はじめに

SwiftDataにチェックを入れてプロジェクトを開始するとサンプルコードが生成される．
このときinsertやdeleteはModelContextを必要とするため，Viewの中に記述されている．これではView以外の場所でDataを扱うことができなくなる．

初めての記事投稿になるので，簡単な内容を投稿する．

# この記事でわかること

SwiftDataをView以外の場所で使えるようになる．

# 詳細

## 1. 基本方針

- データを操作するクラスを実装する
- ModelContextデータを操作する関数の引数として受け取る

## 2. 詳細実装（App）

```Swift:SwiftData_Qiita_testApp.swift
@main
struct SwiftData_Qiita_testApp: App {
    var sharedModelContainer: ModelContainer = {
        let schema = Schema([
            Item.self,
        ])
        let modelConfiguration = ModelConfiguration(schema: schema, isStoredInMemoryOnly: false)

        do {
            return try ModelContainer(for: schema, configurations: [modelConfiguration])
        } catch {
            fatalError("Could not create ModelContainer: \(error)")
        }
    }()

    var body: some Scene {
        WindowGroup {
            ContentView()
        }
        .modelContainer(sharedModelContainer)
    }
}
```

- 共有用のModelContainerを作成する
  - Schema: データ構造を決定する
  - ModelConfiguration: データの保存に関する設定を行う
  - これらを元にModelContainerを作成する
- ContentViewにModelContainerを渡す
  - ViewでSwiftDataを使うためには必ず渡す必要がある

## 2. 詳細実装（Model）

```Swift:Item.swift
@Model
final class Item {
    var timestamp: Date
    
    init(timestamp: Date) {
        self.timestamp = timestamp
    }
}


class ItemDatastore {
    // シングルトン
    @MainActor static let shared = ItemDatastore()
    
    func insert(modelContext: ModelContext, timestamp: Date) {
        modelContext.insert(Item(timestamp: timestamp))
        
        try? modelContext.save()
    }
    
    
    func delete(modelContext: ModelContext, item: Item) {
        modelContext.delete(item)
        
        try? modelContext.save()
    }
    
}
```

- 今回扱うデータ構造を定義する
  - データ構造のクラスには@Modelをつける
- データ操作を行うクラスを実装する
  - シングルトンで実装する
  - insert, delete関数を実装する
  - どちらも引数にModelContextを受け取る

## 3. 詳細実装（ContentView）

```Swift:ContentView.swift
struct ContentView: View {
    @Environment(\.modelContext) private var modelContext
    @Query private var items: [Item]

    var body: some View {
        NavigationSplitView {
            List {
                ForEach(items) { item in
                    NavigationLink {
                        Text("Item at \(item.timestamp, format: Date.FormatStyle(date: .numeric, time: .standard))")
                    } label: {
                        Text(item.timestamp, format: Date.FormatStyle(date: .numeric, time: .standard))
                    }
                }
                .onDelete(perform: deleteItems)
            }
            ...
    }

    private func addItem() {
        withAnimation {
            let newItem = Item(timestamp: Date())
//            modelContext.insert(newItem)
            ItemDatastore.shared.insert(modelContext: modelContext, timestamp: newItem.timestamp)
        }
    }

    private func deleteItems(offsets: IndexSet) {
        withAnimation {
            for index in offsets {
//                modelContext.delete(items[index])
                ItemDatastore.shared.delete(modelContext: modelContext, item: items[index])
            }
        }
    }
}
```

- データ操作を行う場合はmodelContextが必要である

```Swift:ContentView.swift
@Environment(\.modelContext) private var modelContext
```

- リアルタイムでデータを取得する場合は@Queryを使う
  - このように宣言された変数はデータが変更されると自動的に更新される

```Swift:ContentView.swift
@Query private var items: [Item]
```

- データ操作を行う関数を呼び出す際は，引数にmodelContextを渡す
  - これによりView以外の場所でデータ操作を行うことができる

```Swift:ContentView.swift
ItemDatastore.shared.insert(modelContext: modelContext, timestamp: newItem.timestamp)
```

# ここで紹介したコード

[https://github.com/k22036/SwiftData-Qiita-test](https://github.com/k22036/SwiftData-Qiita-test)

# まとめ

View以外の場所でSwiftDataを使う方法を紹介した．
ModelContextを引数に受け取ることで，View以外の場所でデータ操作を行うことができる．
