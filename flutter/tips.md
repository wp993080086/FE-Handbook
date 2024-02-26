# TextField

```dart
/// prompt滚动控制器
final ScrollController promptController = ScrollController();

TextField(
  scrollController:controller.promptController,
  // 获取焦点
  focusNode: controller.myFocusNode,
  controller:
      controller.promptText.value,
  // 设置为 null 表示多行输入
  maxLines: null,
  maxLength: 500,
  keyboardType: TextInputType.multiline,
  textInputAction: TextInputAction.done,
  autofocus: true,
  decoration: InputDecoration(
    // 塌陷去掉padding内边距
    isCollapsed: true,
    contentPadding: EdgeInsets.zero,
    hintText: 'Enter Prompt',
    // 去除输入框底部的字符计数
    counterText: "",
    hintStyle: TextStyle(
        color: Colors.white.withOpacity(0.5)),
    border: InputBorder.none,
  ),
  cursorColor: Colors.white,
  style: TextStyle(
      fontSize: 16.sp,
      color: Colors.white,
      fontWeight: FontWeight.w500,
      height: 1.5),
  onChanged:
      controller.handleEnterChange,
  onEditingComplete: () {
    controller.handleAutoBack(context);
  }
);
```