---
author:
- LTSlw
tags:
- android
date: 2024-05-02
lastmod: 2024-05-07
---

# Intent

`Intent`用于在activity、service、boardcast之间传递信息。`Intent`有两种

- `Explicit intents`（显式）：要求指明接收intent的类，一般用于应用内部传递信息
- `Implicit intents`（隐式）：无需指定接收组件，由系统寻找一个满足要求的组件接收intent，一般用于唤起另一个应用执行操作。如果有多个可以响应的应用，系统会提示用户选择一个应用启动。如果没有可以响应的应用，会抛出[ActivityNotFoundException](https://developer.android.com/reference/android/content/ActivityNotFoundException)

## 内部构成

intent可以包含以下内容

- `Component name`：可选，目标组件信息，如果设置了，则intent为显式，可以使用`setComponent()`、`setClass()`、`setClassName()`或构造函数来设置（[Activity话](第06话%20Activity.md)中用的都是这种），详细了解可以查看[文档](https://developer.android.com/reference/android/content/ComponentName)
- `action`：String，指示要执行的操作，比如拨打电话。Android内置了一些action，也可以在\<intent-filter\>中自定义action共其他应用调用。预定义的action可以在[文档](https://developer.android.com/reference/kotlin/android/content/Intent)中查看（`ACTION_`开头的常量）
- `data`：Uri，操作需要的[Uri](https://developer.android.com/reference/kotlin/android/net/Uri)数据，可以使用`setData()`设置
- `type`：String，MIME类型，可以使用`setType()`设置
- `category`：String，启动组件的类别，Android预定义了一些，也可以自定义。预定义的category可以在[文档]中查看(https://developer.android.com/reference/kotlin/android/content/Intent)（`CATEGORY_`开头的常量）
- `extras`：Bundle，可以在此添加额外的信息，在[Activity话](第06话%20Activity.md)中应该已经使用很多次了。需要注意，如果传递intent到其他应用，那么不应该使用*自定义的*`Parcelable`和`Serializable`类型的变量
- `flag`：指示系统启动组件的方式，可以使用`setFlags()`设定

*如果需要同时设置data和type应使用`setDataAndType()`*

*不建议嵌套intent（把一个intent放进另一个intent的extras中）发送到其他应用，应使用[PendingIntent](https://developer.android.com/reference/kotlin/android/app/PendingIntent)，[详细信息](https://medium.com/androiddevelopers/android-nesting-intents-e472fafc1933)*

如果要检测不安全的intent可以使用`detectUnsafeIntentLaunch()`

``` kotlin
fun onCreate() {
    StrictMode.setVmPolicy(VmPolicy.Builder()
        // Other StrictMode checks that you've previously added.
        // ...
        .detectUnsafeIntentLaunch()
        .penaltyLog()
        // Consider also adding penaltyDeath()
        .build())
}
```

## 注册隐式intent

如果希望程序的某个组件接收隐式intent要在`AndroidManifest.xml`中设置`<intent-filter>`

\<intent-filter\>必须包含`<action>`，它有一个属性`android:name`，intent的action匹配的就是这个字段。其他的字段非必须，`<data>`指定接受的URI或MIME类型，`<category>`指定组件类别。

*一个\<intent-filter\>可以包含的\<action\>，\<data\>，\<category\>的数量不限*

**如果设置了\<intent-filter\>，必须同时显式设置`android:exported`**，明确其他应用是否可以可以启动组件，**应该竟可能少地暴露组件**

一个\<intent-filter\>的例子

``` xml
<activity android:name="ShareActivity" android:exported="false">
    <intent-filter>
        <action android:name="android.intent.action.SEND"/>
        <category android:name="android.intent.category.DEFAULT"/>
        <data android:mimeType="text/plain"/>
    </intent-filter>
</activity>
```

\<intent-filter\>还有其他可以包含的属性，可以查看[文档](https://developer.android.com/guide/topics/manifest/intent-filter-element)

## 匹配规则

intent按照以下顺序匹配\<intent-filter\>

1. action，intent中的action必须匹配\<intent-filter\>中的其中之一
2. data
    - 由于uri和mime可能指定也可能不指定，intent和\<intent-filter\>二者存在与否相同时才会发生匹配（mime有时可从uri推出）
    - \<intent-filter\>与uri相关的属性生效要求比此属性在uri中靠左的属性都存在，不生效的或没设定的属性在匹配url时认为不限
    - 仅指定mime时，认为希望读取文件（scheme为`content:`和`file:`）
    - 如果intent指定了uri和mime但\<intent-filter\>没有指定\<data\>匹配会失败
3. category，intent中的每一个category都必须匹配\<intent-filter\>中的其中之一

## PendingIntent

`PendingIntent`用于授予其他应用执行intent调用内部组件的权限，可以限定intent能否多次执行，能否修改。有关PendingIntent比较重要的3个方法是`getActivity()`、`getService()`、`getBroadcast()`，分别用于启动不同类型的组件。他们使用方法相近，均接受4个参数，以`getActivity()`为例，`static fun getActivity(context: Context!, requestCode: Int, intent: Intent!, flags: Int): PendingIntent!`

需要注意intent应使用显式intent，flags必须指定可变性（设置`FLAG_IMMUTABLE`或`FLAG_MUTABLE`）。还有一个比较有用的`flag`是`FLAG_ONE_SHOT`，只允许PendingIntent使用一次

PendingIntent可以调用方法`send()`使用

``` kotlin
val intent = Intent(this, AnotherActivity::class.java) // 创建intent
val pendingIntent = PendingIntent.getActivity(this, REQUEST_CODE, intent, PendingIntent.FLAG_IMMUTABLE) // 使用intent构造pendingIntent
pendingIntent.send() // 执行intent（在外部应用中）
```

pendingIntent的更多用法，可以参考[文档](https://developer.android.com/reference/kotlin/android/app/PendingIntent)

## 强制使用应用选择器

Android允许用户选择默认响应intent的应用。如果你想强制不使用默认，可以使用`Intent.createChooser()`强制要求用户选择接收intent的应用

``` kotlin
val intent = Intent()                                     // 按照自己需要正常构造一个intent
val chooser = Intent.createChooser(intent, "choose from") // 第二个参数可以设置选择器标题
```

## 检查intent可以匹配的组件

相关函数包含在[`PackageManager`](https://developer.android.com/reference/kotlin/android/content/pm/PackageManager)中，分别是`queryIntentActivities()`、`queryIntentServices()`和`queryBroadcastReceivers()`

## 一些常用的intent

见[文档](https://developer.android.com/guide/components/intents-common)
