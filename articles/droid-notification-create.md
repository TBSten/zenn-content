---
title: "【Androidで通知を出すために】通知を作成する"
emoji: "📣"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["android"]
published: true
published_at: 2022-05-08 08:10
---

権限の要求周りの話は[こちら](https://zenn.dev/tbsten/articles/droid-notification-permission)

---

公式ドキュメント:

https://developer.android.com/guide/topics/ui/notifiers/notifications?hl=ja

# 通知を作成する
なんやかんやして権限をチェックできたら以下を参考に通知を作っていきます。

https://developer.android.com/training/notify-user/build-notification?hl=ja

```groovy:build.gradle (:app)
dependencies {
    // androidx.core:core-ktx:<version> に含まれているっぽいのでなくても動く
    // (追加するとバージョンが競合してむしろ動かなくなることも)
    implementation "com.android.support:support-compat:28.0.0"
}
```

```kotlin
val channelId = /* 後述 */
var builder = NotificationCompat.Builder(context, channelId)
    .setSmallIcon(R.drawable.notification_icon)
    .setContentTitle("通知のタイトル")
    .setContentText("通知の説明")
    .setPriority(NotificationCompat.PRIORITY_DEFAULT)
```

基本的には[ドキュメント](https://developer.android.com/training/notify-user/build-notification?hl=ja)を参考に適宜設定していきます。

# 通知チャンネルを作成する

ところで `channelId` とはなんでしょうか？
ドキュメントを辿ると **Android 8.0 以上で通知を配信するにはアプリの通知チャネルをシステムに登録しておく必要があり** とあります。

通知チャンネルは設定アプリなどではカテゴリと呼ばれるもので、通知をジャンル分けるするためのものです。

注意:[通知グループ](https://developer.android.com/guide/topics/ui/notifiers/notifications?hl=ja#bundle)とは別物です。

| Chrome                                | Twitter                               |
| ------------------------------------- | ------------------------------------- |
| ![](/images/droid-notification/1.png) | ![](/images/droid-notification/2.png) |

このように通知をチャンネルでジャンル分けすることでユーザがジャンルごとに通知をON/OFFすることができるようになっています。

通知チャンネルはドキュメント通り以下のように作成します。
```kotlin:
private fun createNotificationChannel() {
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
        // 
        val name = "通知チャンネルの名前"
        val descriptionText = "通知チャンネルの説明"
        val importance = NotificationManager.IMPORTANCE_DEFAULT // 通知チャンネルの優先度
        val channel = NotificationChannel(channelId, name, importance).apply {
            description = descriptionText
        }
        val notificationManager: NotificationManager =
            getSystemService(Context.NOTIFICATION_SERVICE) as NotificationManager
        notificationManager.createNotificationChannel(channel)
    }
}
```

# 通知を表示する

表示は難しいことはなく、以下でNotificationのIdとNotificationを渡して表示できます。
なお同じnotificationIdで別のNotificationインスタンスを渡すと通知を更新できるみたいです。

```kotlin:
NotificationManagerCompat.from(context)
    .notify(notificationId, builder.build())
```

ここまで表示できなかった場合は

- [権限周りの設定](https://zenn.dev/tbsten/droid-notification-permission)が悪いか、
- 通知チャンネルが作成されていない
- 通知にアイコンが設定されていない

などが考えられます...

(自分がわかる範囲であればこの記事のコメントで質問してもらっても構いません)

# まとめ

以上をまとめると

1. [権限周りの設定](https://zenn.dev/tbsten/droid-notification-permission)をすましておく

2. 通知チャンネル・通知を設計する

どんな通知チャンネルを作成するか、またそれぞれの設定値を考えておきます。

```md:通知チャンネル・通知の設計例
1. お知らせ (通知チャンネル)
    - アップデートのお知らせ (通知)
    - 新機能のお知らせ (通知)
    ...

2. ダウンロード (通知チャンネル)
    - ファイル1のダウンロード (通知)
    - ファイル2のダウンロード (通知)
    ...

```

アプリの設定を想像しながら設計すると良きでしょう！

3. 通知チャンネルを作成する

```kotlin:通知チャンネルの作成
val channelId = "通知チャンネルのID"

fun createNotificationChannel() {
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
        // 
        val name = "通知チャンネルの名前"
        val descriptionText = "通知チャンネルの説明"
        val importance = NotificationManager.IMPORTANCE_DEFAULT // 通知チャンネルの優先度
        val channel = NotificationChannel(channelId, name, importance).apply {
            description = descriptionText
        }
        val notificationManager: NotificationManager =
            getSystemService(Context.NOTIFICATION_SERVICE) as NotificationManager
        notificationManager.createNotificationChannel(channel)
    }
}
```

4. 通知を作成する

```kotlin:通知の作成
var builder = NotificationCompat.Builder(context, channelId)
    .setSmallIcon(R.drawable.notification_icon)
    .setContentTitle("通知のタイトル")
    .setContentText("通知の説明")
    .setPriority(NotificationCompat.PRIORITY_DEFAULT)
```

5. 通知を表示する

```kotlin:通知の表示
NotificationManagerCompat.from(context)
    .notify(notificationId, builder.build())
```
