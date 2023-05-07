---
title: "【Androidで通知を出すために】通知の権限回りを整える"
emoji: "📣"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["android"]
published: true
published_at: 2022-05-08 08:00
---

公式ドキュメント:

https://developer.android.com/guide/topics/ui/notifiers/notifications?hl=ja

https://developer.android.com/guide/topics/ui/notifiers/notification-permission?hl=ja

# 通知の権限を要求する

https://developer.android.com/guide/topics/ui/notifiers/notification-permission?hl=ja

## 1.1. AndroidManifestに宣言

AndroidManifestに以下を追加します。

```xml:AndroidManifest.xml
<manifest ...>
    <uses-permission android:name="android.permission.POST_NOTIFICATIONS"/> <!-- ここ -->
    <application ...>
        ...
    </application>
</manifest>
```

## 1.2. 実行時の権限を要求する

Android13以降では **AndroidManifest** とは別に **実行時の権限** が必要になります。(以下参照)

https://developer.android.com/guide/topics/ui/notifiers/notification-permission?hl=ja

通知の権限回りについては↑↑↑のページにまとまっているのですが、この中には実行時の権限の要求の仕方まで載っていませんでした。(載せてくれたっていいのに。)

実行時の権限の要求(↑↑↑のドキュメントでいう権限ダイアログ)は以下のようにリクエストします。

```kotlin:実行時の権限の要求 (Activity バージョン)
private val requestPermissionLauncher =
    registerForActivityResult(ActivityResultContracts.RequestPermission()) { isGranted: Boolean ->
        // 権限ダイアログが閉じられた時
    }

// アプリ初期化時など
requestPermissionLauncher.launch(Manifest.permission.POST_NOTIFICATIONS)

```

しかしこの権限ダイアログ、**ユーザの行動次第で出てこなくなることがある**ようなので注意が必要です。

例えば、以下が考えられます。
- 権限ダイアログで「今後表示しない」を選択した場合
- Android11以降で同じ権限に対してユーザーが何度も許可しないをタップした場合

**権限ダイアログが表示されるかどうか** は以下のように取得できます。(本来はユーザになぜ権限が必要なのかを説明する必要があるかを判定するメソッド見たいですが...)

```kotlin:権限ダイアログが表示されるかを判定する
val shouldShowRequestPermissionRationale = ActivityCompat.shouldShowRequestPermissionRationale(context, Manifest.permission.POST_NOTIFICATIONS)
// trueの場合は権限ダイアログが表示される、falseの場合は権限ダイアログが表示されない
```

またすでに権限を持っているかは以下のようにして取得できます。

```kotlin:権限のチェック
fun hasNotifyPermission(context: Context): Boolean {
  val checkPermissionResult = ActivityCompat.checkSelfPermission(
    context,
    Manifest.permission.POST_NOTIFICATIONS,
  )
  return checkPermissionResult == PackageManager.PERMISSION_GRANTED
}
```

~~(なんで返り値Booleanじゃないねん？)~~

:::message 

Jetpack Composeを用いる場合は以下のように置き換えることができます。

```kotlin:実行時の権限の要求 (Jetpack Compose バージョン)
val requestPermissionLauncher =
    rememberLauncherForActivityResult(ActivityResultContracts.RequestPermission()) {
        // 権限ダイアログが閉じられた時
    }

// LaunchedEffect内など
requestPermissionLauncher.launch(Manifest.permission.POST_NOTIFICATIONS)
```

:::

---

通知の表示は[こちら](https://zenn.dev/tbsten/droid-notification-create)
