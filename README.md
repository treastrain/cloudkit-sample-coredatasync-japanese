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
* 新しい `Contacts` の作成と既存の `Contacts` の削除は、通常の Core Data の操作を介して行われます。
* `NSPersistentCloudKitContainer` は、アプリの Entitlements ファイルにリストされている iCloud コンテナ内のユーザーのプライベートデータベースと同期します。

### Example Flow

1. Run the app on a device. After initializing the store, CoreData+CloudKit mirroring will fetch any existing records from the remote private database.
1. Repeat the above on a simulator, and add a new contact through the UI.
1. The device receives a background push notification which is automatically processed by CoreData+CloudKit. Changes are fetched and merged into the local store, and both simulator and device once again show the same content. Deleting a contact by swiping left on a cell produces similar behavior.

Note that because remote push notifications are not supported on simulators, changes on a running device will not be automatically reflected on a running simulator as they are the other way around. Running the app again on a simulator will cause CoreData+CloudKit to sync changes made on a device.

### Things To Learn

* Showing a live list of Core Data managed objects with SwiftUI using the `@FetchRequest` property wrapper.
* Adding and deleting objects with `NSManagedObjectContext`.
* Automatic syncing of Core Data objects with `NSPersistentCloudKitContainer`.
* `NSPersistentCloudKitContainer` processing of push notifications to receive and display updates initiated on another device.
* Storage and decoding of image types in Core Data and CloudKit Private Database.

### Further Reading

* [NSPersistentCloudKitContainer](https://developer.apple.com/documentation/coredata/nspersistentcloudkitcontainer)
* [Mirroring a Core Data Store with CloudKit](https://developer.apple.com/documentation/coredata/mirroring_a_core_data_store_with_cloudkit)
