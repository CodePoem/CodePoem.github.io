---
title: 优雅Intent ActivityResult
date: 2020-12-13 20:49:21
updated: 2020-12-13 20:49:25
categories:
  - Android
tags:
  - Android
  - Intent
  - ActivityResult
---

## ActivityResult 新时代

跳转 Activity 获取返回值，我们怎么做?

AndroidX Activity 1.2.0-alpha02 和 Fragment 1.3.0-alpha02 是新旧时代的划分线。

调用方假设为 IntentsActivity，被调用方假设为 IntentResultActivity。

- 旧石器时代：

  调用方：

  1. 调用 startActivityForResult 传入请求码，传输数据（可选）。
  2. 覆写 onActivityResult 方法，根据请求码和结果码获取返回结果信息。

  ```kotlin
  class IntentsConstants {
      companion object {
          const val REQUEST_CODE_OLD = 1
          const val RESULT_CODE_OLD = 100
          const val EXTRA_RESULT_OLD_REQUEST = "extra_result_old_request"
          const val EXTRA_RESULT_OLD_RESULT = "extra_result_old_result"
      }
  }
  ```

  ```kotlin
  fun start() {
    val intent = Intent(this@IntentsActivity, IntentResultActivity::class.java)
    val bundle = Bundle()
    bundle.putString(EXTRA_RESULT_OLD_REQUEST, "OLD")
    intent.putExtras(bundle)
    startActivityForResult(
        intent,
        IntentsConstants.REQUEST_CODE_OLD
    )
  }

  override fun onActivityResult(requestCode: Int,   resultCode: Int, data: Intent?) {
    super.onActivityResult(requestCode, resultCode, data)
    if (requestCode == IntentsConstants.REQUEST_CODE_OLD && resultCode == IntentsConstants.RESULT_CODE_OLD) {
        val result = data?.extras?.getString(IntentsConstants.EXTRA_RESULT_OLD_RESULT)
        Toast.makeText(this@IntentsActivity, "result: $result", Toast.LENGTH_SHORT).show()
    }
  }
  ```

  被调用方：

  1. finish 前 调用 setResult，传入结果码，返回数据（可选）。

  ```kotlin
  override fun onBackPressed() {
      val resultIntent = Intent()
      val resultBundle = Bundle()
      resultBundle.putString(IntentsConstants.EXTRA_RESULT_OLD_RESULT, "resultold")
      resultIntent.putExtras(resultBundle)

      setResult(IntentsConstants.RESULT_CODE_OLD, resultIntent)
      super.onBackPressed()
  }
  ```

- 新石器时代

  1. 引入依赖 AndroidX Activity 1.2.0-alpha02 和 Fragment 1.3.0-alpha02 或以上版本。
  2. 定义协议

  ```gradle
  implementation 'androidxactivity:activity-ktx:1.2.0-beta02'
  implementation 'androidxfragment:fragment-ktx:1.3.0-beta02'
  ```

  ```kotlin
  class IntentResultContract : ActivityResultContract<String, String>() {
    override fun createIntent(context: Context, input: String?): Intent {
        return Intent(context, IntentResultActivity::class.java).apply {
            putExtras(Bundle().apply {
                putString(EXTRA_RESULT_NEW_REQUEST, "NEW")
            })
        }
    }

    override fun parseResult(resultCode: Int, intent: Intent?): String? {
        val data = intent?.extras?.getString(EXTRA_RESULT_NEW_RESULT)
        return if (resultCode == RESULT_CODE_NEW && data != null) data
        else null
    }

    companion object {
        const val RESULT_CODE_NEW = 200

        const val EXTRA_RESULT_NEW_REQUEST = "extra_result_new_request"
        const val EXTRA_RESULT_NEW_RESULT = "extra_result_new_result"
    }
  }
  ```

  调用方：

  1. 注册协议
  2. 启动协议

  ```kotlin
  private val intentsActivityLauncher =
  registerForActivityResult(IntentResultContract()) { result ->
      Toast.makeText(this@IntentsActivity, "result: $result", Toast.LENGTH_SHORT).show()
  }

  fun start() {
    intentsActivityLauncher.launch("NEW")
  }
  ```

  被调用方：

  1. finish 前 调用 setResult，传入结果码，返回数据（可选）。

  ```kotlin
  override fun onBackPressed() {
    val resultIntent = Intent()
    val resultBundle = Bundle()
    resultBundle.putString(IntentsConstants.EXTRA_RESULT_NEW_RESULT, "resultnew")
    resultIntent.putExtras(resultBundle)

    setResult(IntentsConstants.RESULT_CODE_NEW, resultIntent)
    super.onBackPressed()
  }
  ```

### 基础概念

- ActivityResultContract：协议，定义了如何传递数据和如何处理返回的数据。
- ActivityResultLauncher： 启动器，调用 ActivityResultLauncher 的 launch 方法来启动页面跳转，作用相当于原来的 startActivity()。

### 库中默认已实现 Contract

库中默认已实现的 Contract 都定义在 androidx.activity.result.contract.ActivityResultContracts 中。

- StartActivityForResult: 通用的 Contract，不做任何转换，Intent 作为输入，ActivityResult 作为输出，这也是最常用的一个协定。

- RequestMultiplePermissions： 用于请求一组权限。

- RequestPermission: 用于请求单个权限。

- TakePicturePreview: 调用 MediaStore.ACTION_IMAGE_CAPTURE 拍照，返回值为 Bitmap 图片。

- TakePicture: 调用 MediaStore.ACTION_IMAGE_CAPTURE 拍照，并将图片保存到给定的 Uri 地址，返回 true 表示保存成功。

- TakeVideo: 调用 MediaStore.ACTION_VIDEO_CAPTURE 拍摄视频，保存到给定的 Uri 地址，返回一张缩略图。

- PickContact: 从通讯录 APP 获取联系人。

- GetContent: 提示用选择一条内容，返回一个通过 ContentResolver#openInputStream(Uri) 访问原生数据的 Uri 地址（content://形式） 。默认情况下，它增加了 Intent#CATEGORY_OPENABLE, 返回可以表示流的内容。

- CreateDocument: 提示用户选择一个文档，返回一个(file:/http:/content:)开头的 Uri。

- OpenMultipleDocuments: 提示用户选择文档（可以选择多个），分别返回它们的 Uri，以 List 的形式。

- OpenDocumentTree: 提示用户选择一个目录，并返回用户选择的作为一个 Uri 返回，应用程序可以完全管理返回目录中的文档。

## ActivityResult 新时代优势

1. 以回调的方式获取返回值，而不是覆写方法。个人认为覆写方法获取返回值有一种“割裂”的感觉。

2. 减少了样板代码，自定义 Contract 可以复用传值和解析值的逻辑。

## Activity 销毁
