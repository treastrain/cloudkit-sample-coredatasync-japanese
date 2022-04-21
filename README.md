# CloudKit サンプル: CloudKit で Core Data を使う

### 概要

このプロジェクトは、Core Data のスタックをアプリケーションにカプセル化し、永続的なストアを CloudKit のプライベートデータベースにミラーリングしてくれる `NSPersistentCloudKitContainer` で Core Data を使うサンプルです。ローカルで行われた変更は自動的にリモートの CloudKit データベースに送られ、リモートの変更はバックグラウンドのプッシュ通知によってデバイス間でフェッチされローカルストアに取り込まれます。

### 環境

* このプロジェクトのビルドとテストを実行するには、[Xcode 12](https://developer.apple.com/xcode/) 以降のインストールされた Mac が必要です。
* iOS Simulator ではリモートからのプッシュ通知を受信できないため、アプリをインストールし実行するには、CloudKit のプッシュ通知（変更通知）を受信できるiOSデバイスが必要です。
* CloudKit コンテナを作成し、アプリを署名してデバイス上で実行するには、アクティブな [Apple Developer Program メンバーシップ](https://developer.apple.com/support/compare-memberships/)が必要です。

### セットアップ手順

1. Apple Developer Program メンバーシップがアクティブになっている Apple ID が Xcode に追加されている状態にします。
1. `CoreDataSync` ターゲットの「Signing & Capabilities」タブにある "Signing" セクションでチームが選択されていることと、「iCloud」セクションで有効なコンテナが選択されていることを確認します。
1. 使用する iOS Simulator とアプリを実行する実機デバイスの両方が、同じ iCloud アカウントにログインしていることを確認します。

#### 自分の iCloud コンテナを使用する

* Xcode の `CoreDataSync` アプリターゲットの「Signing & Capabilities」タブから、新しい iCloud コンテナを作成します。
* [Config.swift](CoreDataSync/App/Config.swift) にある `containerIdentifier` プロパティの値を、作成した新しい iCloud コンテナ ID に変更します。

### 動作の解説

* `CoreDataSync` モデルでは、名前（`String`）と写真（`UIImage` Transformable）を格納する `Contact` エンティティを定義しています。
* `PersistenceController` クラスは、`NSPersistentCloudKitContainer` オブジェクトを `CoreDataSync` モデルとともに生成します。
* 一覧画面（`ContentView`）では、[`@FetchRequest`](https://developer.apple.com/documentation/swiftui/fetchrequest) を使用して Core Data ストアから `Contact` オブジェクトを取得します。
* 新しい `Contact` の作成と既存の `Contact` の削除は、通常の Core Data の操作を介して行われます。
* `NSPersistentCloudKitContainer` は、アプリの Entitlements ファイルにリストされている iCloud コンテナ内のユーザーのプライベートデータベースと同期します。

### フローの例

1. 実機デバイスでアプリを実行します。ストアを初期化した後、Core Data と CloudKit のミラーリングによって、リモートにあるプライベートデータベースから既存のレコードをフェッチされます。
1. iOS Simulator 側のアプリで UI から新しい `Contact` を追加します。
1. 実機デバイス側ではバックグラウンドでプッシュ通知を受信し、Core Data と CloudKit によって自動的に処理されます。変更はローカルストアにフェッチされマージされ、iOS Simulator 側と実機デバイス側の両方で同じコンテンツが再び表示されます。セルを左にスワイプして `Contact` を削除しても、同様の動作が行われます。

iOS Simulator ではリモートからのプッシュ通知がサポートされていないため、実行中の実機デバイス側での変更は、逆に実行中の iOS Simulator 側に自動的に反映されないことに注意してください。iOS Simulator 上でアプリを再度実行することで、Core Data と CloudKit は実機デバイス側で行われた変更を同期するようになります。

### このプロジェクトでわかること

* `@FetchRequest` を使用した SwiftUI での Core Data 管理対象オブジェクトのリストの表示。
* `NSManagedObjectContext` を使用したオブジェクトの追加と削除。
* `NSPersistentCloudKitContainer` を使用した Core Data オブジェクトの自動同期。
* `NSPersistentCloudKitContainer` が行う、別なデバイスからの更新をプッシュ通知経由で受けたときの表示の更新。
* Core Data および CloudKit プライベートデータベースでの `UIImage` の保存とデコード。

### 参考

* [NSPersistentCloudKitContainer](https://developer.apple.com/documentation/coredata/nspersistentcloudkitcontainer)
* [Mirroring a Core Data Store with CloudKit](https://developer.apple.com/documentation/coredata/mirroring_a_core_data_store_with_cloudkit)
