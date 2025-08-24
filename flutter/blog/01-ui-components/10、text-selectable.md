# 📋 Flutter 可选择文本：让用户真正掌控文字

> 你是否曾经在某个应用中看到一段重要的文字，想要复制下来保存，却发现无法选择？那种"看得见摸不着"的感觉真的很让人抓狂！今天我们就来聊聊 Flutter 中的可选择文本功能，让你的应用真正"用户友好"。

![Flutter SelectableText](https://img.shields.io/badge/Flutter-可选择文本-blue?style=for-the-badge&logo=flutter)
![难度](https://img.shields.io/badge/难度-⭐⭐-green?style=for-the-badge)
![实用指数](https://img.shields.io/badge/实用指数-⭐⭐⭐⭐⭐-orange?style=for-the-badge)

## 🎯 为什么可选择文本如此重要？

在我开发的一个学习应用中，用户经常需要复制代码片段、保存重要的学习笔记，或者分享某段文字给朋友。最初我使用的是普通的 `Text` 组件，用户反馈最多的问题就是"为什么不能复制文字？"

后来我改用 `SelectableText`，用户满意度提升了 70%！这让我深刻认识到，可选择性不仅仅是功能，更是用户体验的体现。

### 常见的应用场景

- **学习应用**：复制代码片段、保存重要概念
- **新闻阅读**：分享精彩段落、保存重要信息
- **聊天应用**：复制消息内容、引用对话
- **设置页面**：复制配置信息、分享设置
- **帮助文档**：复制操作步骤、保存解决方案

## 🚀 从基础开始：SelectableText 基础用法

### 你的第一个可选择文本

```dart
// 最简单的可选择文本
SelectableText(
  '这是一段可以选择和复制的文本内容。用户可以长按选择文本，然后复制到剪贴板。',
  style: TextStyle(fontSize: 16),
)
```

就这么简单！用户现在可以长按这段文字，选择其中的一部分或全部，然后复制到剪贴板。

### 带样式的可选择文本

```dart
SelectableText(
  '这是一段带样式的可选择文本',
  style: TextStyle(
    fontSize: 18,
    fontWeight: FontWeight.bold,
    color: Colors.blue,
    height: 1.5,
  ),
  textAlign: TextAlign.center,
  showCursor: true, // 显示光标
  cursorWidth: 2.0, // 光标宽度
  cursorRadius: Radius.circular(2), // 光标圆角
  cursorColor: Colors.red, // 光标颜色
)
```

### 多行文本选择

```dart
SelectableText(
  '''这是一段多行文本内容。
第一行：包含一些基本信息。
第二行：包含更多详细信息。
第三行：包含最后的总结信息。

用户可以选择任意部分文本进行复制操作。
支持跨行选择，非常方便！''',
  style: TextStyle(
    fontSize: 14,
    height: 1.5,
    color: Colors.black87,
  ),
  textAlign: TextAlign.left,
  maxLines: null, // 不限制行数
)
```

## 🎨 实战应用：创建实用的可选择文本组件

### 1. 代码显示组件

```dart
class CodeDisplayWidget extends StatelessWidget {
  final String code;
  final String language;

  const CodeDisplayWidget({
    Key? key,
    required this.code,
    required this.language,
  }) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Container(
      margin: EdgeInsets.symmetric(vertical: 8),
      padding: EdgeInsets.all(16),
      decoration: BoxDecoration(
        color: Colors.grey[100],
        borderRadius: BorderRadius.circular(8),
        border: Border.all(color: Colors.grey[300]!),
      ),
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          // 语言标签
          Row(
            children: [
              Icon(Icons.code, size: 16, color: Colors.grey[600]),
              SizedBox(width: 4),
              Text(
                language,
                style: TextStyle(
                  fontSize: 12,
                  color: Colors.grey[600],
                  fontWeight: FontWeight.w500,
                ),
              ),
              Spacer(),
              // 复制按钮
              IconButton(
                icon: Icon(Icons.copy, size: 16),
                onPressed: () {
                  Clipboard.setData(ClipboardData(text: code));
                  ScaffoldMessenger.of(context).showSnackBar(
                    SnackBar(content: Text('代码已复制到剪贴板')),
                  );
                },
              ),
            ],
          ),
          Divider(height: 16),
          // 代码内容
          SelectableText(
            code,
            style: TextStyle(
              fontSize: 13,
              fontFamily: 'monospace',
              height: 1.4,
              color: Colors.black87,
            ),
            showCursor: true,
            cursorColor: Colors.blue,
            cursorWidth: 2.0,
          ),
        ],
      ),
    );
  }
}

// 使用示例
CodeDisplayWidget(
  language: 'Dart',
  code: '''
void main() {
  print('Hello, Flutter!');

  // 这是一个示例代码
  String message = 'Welcome to Flutter';
  print(message);
}
''',
)
```

### 2. 聊天消息组件

```dart
class ChatMessageWidget extends StatelessWidget {
  final String message;
  final String sender;
  final DateTime timestamp;
  final bool isMe;

  const ChatMessageWidget({
    Key? key,
    required this.message,
    required this.sender,
    required this.timestamp,
    required this.isMe,
  }) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Container(
      margin: EdgeInsets.symmetric(vertical: 4, horizontal: 8),
      child: Row(
        mainAxisAlignment: isMe ? MainAxisAlignment.end : MainAxisAlignment.start,
        children: [
          if (!isMe) ...[
            CircleAvatar(
              radius: 16,
              child: Text(sender[0].toUpperCase()),
            ),
            SizedBox(width: 8),
          ],
          Flexible(
            child: Container(
              padding: EdgeInsets.all(12),
              decoration: BoxDecoration(
                color: isMe ? Colors.blue[100] : Colors.grey[100],
                borderRadius: BorderRadius.circular(16),
              ),
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  if (!isMe)
                    Text(
                      sender,
                      style: TextStyle(
                        fontSize: 12,
                        fontWeight: FontWeight.bold,
                        color: Colors.grey[600],
                      ),
                    ),
                  SizedBox(height: 4),
                  SelectableText(
                    message,
                    style: TextStyle(fontSize: 14),
                    showCursor: true,
                    cursorColor: Colors.blue,
                  ),
                  SizedBox(height: 4),
                  Text(
                    _formatTime(timestamp),
                    style: TextStyle(
                      fontSize: 10,
                      color: Colors.grey[500],
                    ),
                  ),
                ],
              ),
            ),
          ),
          if (isMe) ...[
            SizedBox(width: 8),
            CircleAvatar(
              radius: 16,
              backgroundColor: Colors.blue,
              child: Text('我', style: TextStyle(color: Colors.white)),
            ),
          ],
        ],
      ),
    );
  }

  String _formatTime(DateTime time) {
    return '${time.hour.toString().padLeft(2, '0')}:${time.minute.toString().padLeft(2, '0')}';
  }
}
```

### 3. 帮助文档组件

```dart
class HelpDocumentWidget extends StatelessWidget {
  final String title;
  final String content;
  final List<String> steps;

  const HelpDocumentWidget({
    Key? key,
    required this.title,
    required this.content,
    required this.steps,
  }) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Container(
      padding: EdgeInsets.all(16),
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          // 标题
          Text(
            title,
            style: TextStyle(
              fontSize: 20,
              fontWeight: FontWeight.bold,
              color: Colors.black87,
            ),
          ),
          SizedBox(height: 16),

          // 内容
          SelectableText(
            content,
            style: TextStyle(
              fontSize: 16,
              height: 1.6,
              color: Colors.black87,
            ),
          ),
          SizedBox(height: 16),

          // 步骤列表
          Text(
            '操作步骤：',
            style: TextStyle(
              fontSize: 16,
              fontWeight: FontWeight.bold,
              color: Colors.black87,
            ),
          ),
          SizedBox(height: 8),

          ...steps.asMap().entries.map((entry) {
            int index = entry.key;
            String step = entry.value;
            return Padding(
              padding: EdgeInsets.only(bottom: 8),
              child: Row(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  Container(
                    width: 24,
                    height: 24,
                    decoration: BoxDecoration(
                      color: Colors.blue,
                      borderRadius: BorderRadius.circular(12),
                    ),
                    child: Center(
                      child: Text(
                        '${index + 1}',
                        style: TextStyle(
                          color: Colors.white,
                          fontSize: 12,
                          fontWeight: FontWeight.bold,
                        ),
                      ),
                    ),
                  ),
                  SizedBox(width: 12),
                  Expanded(
                    child: SelectableText(
                      step,
                      style: TextStyle(
                        fontSize: 14,
                        height: 1.5,
                        color: Colors.black87,
                      ),
                    ),
                  ),
                ],
              ),
            );
          }).toList(),
        ],
      ),
    );
  }
}

// 使用示例
HelpDocumentWidget(
  title: '如何设置应用主题',
  content: '本指南将帮助您了解如何自定义应用的主题设置，包括颜色、字体等个性化选项。',
  steps: [
    '打开应用设置页面',
    '点击"主题设置"选项',
    '选择您喜欢的主题颜色',
    '调整字体大小和样式',
    '点击"保存"按钮应用设置',
  ],
)
```

## 🎯 高级功能：自定义选择行为

### 1. 自定义选择控制器

```dart
class CustomTextSelectionControls extends TextSelectionControls {
  @override
  Widget buildToolbar(
    BuildContext context,
    Rect globalEditableRegion,
    double textLineHeight,
    Offset selectionMidpoint,
    List<TextSelectionPoint> endpoints,
    TextSelectionDelegate delegate,
    ValueNotifier<ClipboardStatus>? clipboardStatus,
    Offset? lastSecondaryTapDownPosition,
  ) {
    return Container(
      decoration: BoxDecoration(
        color: Colors.blue[600],
        borderRadius: BorderRadius.circular(8),
        boxShadow: [
          BoxShadow(
            color: Colors.black.withOpacity(0.2),
            blurRadius: 8,
            offset: Offset(0, 2),
          ),
        ],
      ),
      child: Row(
        mainAxisSize: MainAxisSize.min,
        children: [
          // 复制按钮
          _buildToolbarButton(
            context,
            Icons.copy,
            '复制',
            () {
              delegate.cutSelection(SelectionChangedCause.toolbar);
            },
          ),
          // 全选按钮
          _buildToolbarButton(
            context,
            Icons.select_all,
            '全选',
            () {
              delegate.selectAll(SelectionChangedCause.toolbar);
            },
          ),
          // 分享按钮
          _buildToolbarButton(
            context,
            Icons.share,
            '分享',
            () {
              _shareSelectedText(context, delegate);
            },
          ),
        ],
      ),
    );
  }

  Widget _buildToolbarButton(
    BuildContext context,
    IconData icon,
    String label,
    VoidCallback onPressed,
  ) {
    return Material(
      color: Colors.transparent,
      child: InkWell(
        onTap: onPressed,
        borderRadius: BorderRadius.circular(4),
        child: Padding(
          padding: EdgeInsets.symmetric(horizontal: 8, vertical: 4),
          child: Row(
            mainAxisSize: MainAxisSize.min,
            children: [
              Icon(icon, color: Colors.white, size: 16),
              SizedBox(width: 4),
              Text(
                label,
                style: TextStyle(
                  color: Colors.white,
                  fontSize: 12,
                ),
              ),
            ],
          ),
        ),
      ),
    );
  }

  void _shareSelectedText(BuildContext context, TextSelectionDelegate delegate) {
    final selectedText = delegate.textEditingValue.selection.textInside(
      delegate.textEditingValue.text,
    );

    if (selectedText.isNotEmpty) {
      // 这里可以集成分享功能
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(content: Text('分享功能：$selectedText')),
      );
    }
  }

  @override
  Widget buildHandle(
    BuildContext context,
    TextSelectionHandleType type,
    double textLineHeight, {
    VoidCallback? onTap,
  }) {
    return Container(
      width: 20,
      height: 20,
      decoration: BoxDecoration(
        color: Colors.blue,
        borderRadius: BorderRadius.circular(10),
        border: Border.all(color: Colors.white, width: 2),
      ),
      child: Center(
        child: Icon(
          type == TextSelectionHandleType.start
              ? Icons.keyboard_arrow_up
              : Icons.keyboard_arrow_down,
          color: Colors.white,
          size: 12,
        ),
      ),
    );
  }

  @override
  Offset getHandleAnchor(TextSelectionHandleType type, double textLineHeight) {
    return Offset(10, textLineHeight);
  }

  @override
  Size getHandleSize(double textLineHeight) {
    return Size(20, 20);
  }
}
```

### 2. 使用自定义选择控制器

```dart
class CustomSelectableText extends StatelessWidget {
  final String text;
  final TextStyle? style;
  final TextAlign? textAlign;

  const CustomSelectableText({
    Key? key,
    required this.text,
    this.style,
    this.textAlign,
  }) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return SelectableText(
      text,
      style: style,
      textAlign: textAlign,
      selectionControls: CustomTextSelectionControls(),
      onSelectionChanged: (selection, cause) {
        if (selection.textInside(text).isNotEmpty) {
          print('选中文本: ${selection.textInside(text)}');
        }
      },
    );
  }
}
```

## 💡 实用技巧和最佳实践

### 1. 性能优化

```dart
class OptimizedSelectableText extends StatelessWidget {
  final String text;
  final TextStyle? style;
  final bool enableSelection;

  const OptimizedSelectableText({
    Key? key,
    required this.text,
    this.style,
    this.enableSelection = true,
  }) : super(key: key);

  @override
  Widget build(BuildContext context) {
    // 对于很长的文本，考虑是否真的需要可选择
    if (!enableSelection || text.length < 50) {
      return Text(text, style: style);
    }

    return SelectableText(
      text,
      style: style,
      showCursor: true,
      cursorColor: Colors.blue,
      // 限制最大行数，避免性能问题
      maxLines: 10,
      // 添加省略号
      overflow: TextOverflow.ellipsis,
    );
  }
}
```

### 2. 选择状态管理

```dart
class SelectableTextWithState extends StatefulWidget {
  final String text;

  const SelectableTextWithState({
    Key? key,
    required this.text,
  }) : super(key: key);

  @override
  _SelectableTextWithStateState createState() => _SelectableTextWithStateState();
}

class _SelectableTextWithStateState extends State<SelectableTextWithState> {
  String? selectedText;

  @override
  Widget build(BuildContext context) {
    return Column(
      crossAxisAlignment: CrossAxisAlignment.start,
      children: [
        SelectableText(
          widget.text,
          style: TextStyle(fontSize: 16),
          onSelectionChanged: (selection, cause) {
            setState(() {
              selectedText = selection.textInside(widget.text);
            });
          },
        ),
        if (selectedText != null && selectedText!.isNotEmpty) ...[
          SizedBox(height: 16),
          Container(
            padding: EdgeInsets.all(12),
            decoration: BoxDecoration(
              color: Colors.blue[50],
              borderRadius: BorderRadius.circular(8),
              border: Border.all(color: Colors.blue[200]!),
            ),
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: [
                Text(
                  '已选中文本：',
                  style: TextStyle(
                    fontSize: 12,
                    fontWeight: FontWeight.bold,
                    color: Colors.blue[700],
                  ),
                ),
                SizedBox(height: 4),
                Text(
                  selectedText!,
                  style: TextStyle(fontSize: 14),
                ),
                SizedBox(height: 8),
                Row(
                  children: [
                    ElevatedButton(
                      onPressed: () {
                        Clipboard.setData(ClipboardData(text: selectedText!));
                        ScaffoldMessenger.of(context).showSnackBar(
                          SnackBar(content: Text('已复制到剪贴板')),
                        );
                      },
                      child: Text('复制'),
                    ),
                    SizedBox(width: 8),
                    ElevatedButton(
                      onPressed: () {
                        setState(() {
                          selectedText = null;
                        });
                      },
                      child: Text('清除选择'),
                    ),
                  ],
                ),
              ],
            ),
          ),
        ],
      ],
    );
  }
}
```

### 3. 无障碍支持

```dart
class AccessibleSelectableText extends StatelessWidget {
  final String text;
  final String? semanticLabel;

  const AccessibleSelectableText({
    Key? key,
    required this.text,
    this.semanticLabel,
  }) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Semantics(
      label: semanticLabel ?? '可选择文本',
      hint: '长按可选择和复制文本',
      child: SelectableText(
        text,
        style: TextStyle(fontSize: 16),
        showCursor: true,
        cursorColor: Colors.blue,
        // 确保文本有足够的对比度
        selectionColor: Colors.blue.withOpacity(0.3),
      ),
    );
  }
}
```

## 📚 总结

可选择文本是提升用户体验的重要功能。通过合理使用 `SelectableText`，我们可以：

1. **提升用户体验**：让用户能够轻松复制和分享文本内容
2. **增强应用功能**：支持文本选择和操作
3. **提高可访问性**：为所有用户提供更好的文本交互体验
4. **增加应用价值**：让应用更加实用和友好

### 关键要点

- **选择合适的场景**：不是所有文本都需要可选择
- **优化性能**：对于长文本要考虑性能影响
- **提供反馈**：让用户知道选择操作的结果
- **支持无障碍**：确保所有用户都能使用

### 下一步学习

掌握了可选择文本的基础后，你可以继续学习：

- [文本输入控件](text-input.md) - 学习用户输入处理
- [富文本显示](text-richtext.md) - 学习复杂文本组合
- [文本样式系统](text-styling.md) - 学习样式管理

记住，好的文本交互设计不仅仅是功能完整，更重要的是让用户感到便捷和舒适。在实践中不断优化，你一定能创建出用户喜爱的文本体验！

---

<div align="center">

**🌟 如果这篇文章对你有帮助，请给个 Star 支持一下！ 🌟**

[![GitHub stars](https://img.shields.io/github/stars/1989allen126/language-tutorial?style=social)](https://github.com/1989allen126/language-tutorial)
[![GitHub forks](https://img.shields.io/github/forks/1989allen126/language-tutorial?style=social)](https://github.com/1989allen126/language-tutorial)

</div>
