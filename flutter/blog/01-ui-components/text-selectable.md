# 📝 Flutter SelectableText 可选择文本控件详解

> 掌握 Flutter 中可选择文本的核心组件，实现文本选择和复制功能

![Flutter SelectableText](https://img.shields.io/badge/Flutter-SelectableText-blue?style=for-the-badge&logo=flutter)
![Version](https://img.shields.io/badge/Version-1.0.0-green?style=for-the-badge)
![License](https://img.shields.io/badge/License-MIT-yellow?style=for-the-badge)

## 📋 目录导航

- [组件概述](#组件概述)
- [基础用法](#基础用法)
- [高级功能](#高级功能)
- [交互处理](#交互处理)
- [样式定制](#样式定制)
- [最佳实践](#最佳实践)

---

## 🎯 组件概述

`SelectableText` 是 Flutter 中用于显示可选择文本的重要组件，允许用户选择、复制和操作文本内容。

### 核心特性

- **文本选择**：支持用户选择文本内容
- **复制功能**：支持复制选中的文本
- **样式定制**：支持丰富的文本样式配置
- **交互控制**：支持自定义选择行为
- **多语言支持**：支持不同语言的文本选择

### 应用场景

- **长文本显示**：文章、说明文档等长文本内容
- **代码显示**：代码片段的可选择显示
- **聊天记录**：聊天消息的选择和复制
- **设置页面**：配置信息的可选择显示

## 📝 基础用法

### 简单可选择文本

```dart
// 基础可选择文本
SelectableText(
  '这是一段可以选择和复制的文本内容。用户可以长按选择文本，然后复制到剪贴板。',
  style: TextStyle(fontSize: 16),
)

// 带样式的可选择文本
SelectableText(
  '带样式的可选择文本',
  style: TextStyle(
    fontSize: 18,
    fontWeight: FontWeight.bold,
    color: Colors.blue,
  ),
  textAlign: TextAlign.center,
)
```

### 多行文本选择

```dart
SelectableText(
  '''这是一段多行文本内容。
第一行：包含一些基本信息。
第二行：包含更多详细信息。
第三行：包含最后的总结信息。

用户可以选择任意部分文本进行复制操作。''',
  style: TextStyle(
    fontSize: 14,
    height: 1.5,
  ),
  textAlign: TextAlign.left,
)
```

## 🚀 高级功能

### 自定义选择控制器

```dart
class SelectableTextExample extends StatefulWidget {
  @override
  _SelectableTextExampleState createState() => _SelectableTextExampleState();
}

class _SelectableTextExampleState extends State<SelectableTextExample> {
  final TextSelectionControls _customSelectionControls = 
      CustomTextSelectionControls();

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('SelectableText 示例')),
      body: SingleChildScrollView(
        padding: EdgeInsets.all(16),
        child: Column(
          children: [
            // 基础可选择文本
            _buildBasicSelectableText(),
            
            SizedBox(height: 20),
            
            // 自定义选择控制器
            _buildCustomSelectionText(),
            
            SizedBox(height: 20),
            
            // 富文本选择
            _buildRichSelectableText(),
          ],
        ),
      ),
    );
  }

  Widget _buildBasicSelectableText() {
    return Card(
      child: Padding(
        padding: EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Text(
              '基础可选择文本',
              style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold),
            ),
            SizedBox(height: 10),
            
            SelectableText(
              '这是一段可以选择和复制的文本内容。用户可以长按选择文本，然后复制到剪贴板。选择文本后，系统会显示复制、全选等操作选项。',
              style: TextStyle(fontSize: 16),
              onSelectionChanged: (selection, cause) {
                print('选择变化：$selection, 原因：$cause');
              },
            ),
          ],
        ),
      ),
    );
  }

  Widget _buildCustomSelectionText() {
    return Card(
      child: Padding(
        padding: EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Text(
              '自定义选择控制器',
              style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold),
            ),
            SizedBox(height: 10),
            
            SelectableText(
              '这段文本使用了自定义的选择控制器，可以自定义选择菜单的样式和行为。',
              style: TextStyle(fontSize: 16),
              selectionControls: _customSelectionControls,
              onSelectionChanged: (selection, cause) {
                if (selection.isValid) {
                  print('选中文本：${selection.textInside(_text)}');
                }
              },
            ),
          ],
        ),
      ),
    );
  }

  Widget _buildRichSelectableText() {
    return Card(
      child: Padding(
        padding: EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Text(
              '富文本选择',
              style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold),
            ),
            SizedBox(height: 10),
            
            SelectableText.rich(
              TextSpan(
                children: [
                  TextSpan(
                    text: '普通文本 ',
                    style: TextStyle(fontSize: 16),
                  ),
                  TextSpan(
                    text: '粗体文本 ',
                    style: TextStyle(
                      fontSize: 16,
                      fontWeight: FontWeight.bold,
                    ),
                  ),
                  TextSpan(
                    text: '彩色文本',
                    style: TextStyle(
                      fontSize: 16,
                      color: Colors.red,
                    ),
                  ),
                ],
              ),
              onSelectionChanged: (selection, cause) {
                print('富文本选择变化：$selection');
              },
            ),
          ],
        ),
      ),
    );
  }
}

// 自定义文本选择控制器
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
        color: Colors.blue,
        borderRadius: BorderRadius.circular(8),
      ),
      child: Row(
        mainAxisSize: MainAxisSize.min,
        children: [
          _buildToolbarButton(
            context,
            '复制',
            Icons.copy,
            () => delegate.cutSelection(SelectionChangedCause.toolbar),
          ),
          _buildToolbarButton(
            context,
            '全选',
            Icons.select_all,
            () => delegate.selectAll(SelectionChangedCause.toolbar),
          ),
        ],
      ),
    );
  }

  Widget _buildToolbarButton(
    BuildContext context,
    String label,
    IconData icon,
    VoidCallback onPressed,
  ) {
    return Material(
      color: Colors.transparent,
      child: InkWell(
        onTap: onPressed,
        child: Padding(
          padding: EdgeInsets.symmetric(horizontal: 8, vertical: 4),
          child: Row(
            mainAxisSize: MainAxisSize.min,
            children: [
              Icon(icon, color: Colors.white, size: 16),
              SizedBox(width: 4),
              Text(
                label,
                style: TextStyle(color: Colors.white, fontSize: 12),
              ),
            ],
          ),
        ),
      ),
    );
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
        shape: BoxShape.circle,
      ),
      child: Icon(
        type == TextSelectionHandleType.start
            ? Icons.arrow_back_ios
            : Icons.arrow_forward_ios,
        color: Colors.white,
        size: 12,
      ),
    );
  }

  @override
  Offset getHandleAnchor(TextSelectionHandleType type, double textLineHeight) {
    return Offset(10, 10);
  }

  @override
  Size getHandleSize(double textLineHeight) {
    return Size(20, 20);
  }
}
```

## 🎯 交互处理

### 选择事件处理

```dart
SelectableText(
  '可交互的文本内容',
  style: TextStyle(fontSize: 16),
  onSelectionChanged: (selection, cause) {
    // 处理选择变化事件
    if (selection.isValid) {
      print('选中文本：${selection.textInside(text)}');
      print('选择原因：$cause');
    }
  },
  onTap: () {
    // 处理点击事件
    print('文本被点击');
  },
)
```

### 自定义选择行为

```dart
class CustomSelectableText extends StatefulWidget {
  final String text;
  final TextStyle? style;

  CustomSelectableText({
    required this.text,
    this.style,
  });

  @override
  _CustomSelectableTextState createState() => _CustomSelectableTextState();
}

class _CustomSelectableTextState extends State<CustomSelectableText> {
  bool _isSelected = false;
  String _selectedText = '';

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        SelectableText(
          widget.text,
          style: widget.style,
          onSelectionChanged: (selection, cause) {
            setState(() {
              _isSelected = selection.isValid;
              if (_isSelected) {
                _selectedText = selection.textInside(widget.text);
              } else {
                _selectedText = '';
              }
            });
          },
        ),
        
        if (_isSelected) ...[
          SizedBox(height: 8),
          Container(
            padding: EdgeInsets.all(8),
            decoration: BoxDecoration(
              color: Colors.blue[50],
              borderRadius: BorderRadius.circular(4),
            ),
            child: Row(
              children: [
                Text('已选择：$_selectedText'),
                Spacer(),
                TextButton(
                  onPressed: () {
                    Clipboard.setData(ClipboardData(text: _selectedText));
                    ScaffoldMessenger.of(context).showSnackBar(
                      SnackBar(content: Text('已复制到剪贴板')),
                    );
                  },
                  child: Text('复制'),
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

## 🎨 样式定制

### 自定义文本样式

```dart
SelectableText(
  '自定义样式的可选择文本',
  style: TextStyle(
    fontSize: 18,
    fontWeight: FontWeight.w500,
    color: Colors.blue[700],
    height: 1.6,
    letterSpacing: 0.5,
  ),
  textAlign: TextAlign.justify,
  textDirection: TextDirection.ltr,
  showCursor: true,
  cursorWidth: 2,
  cursorHeight: 20,
  cursorRadius: Radius.circular(2),
  cursorColor: Colors.red,
)
```

### 富文本样式

```dart
SelectableText.rich(
  TextSpan(
    children: [
      TextSpan(
        text: '标题：',
        style: TextStyle(
          fontSize: 20,
          fontWeight: FontWeight.bold,
          color: Colors.black,
        ),
      ),
      TextSpan(
        text: '这是正文内容，',
        style: TextStyle(
          fontSize: 16,
          color: Colors.grey[700],
        ),
      ),
      TextSpan(
        text: '重要信息',
        style: TextStyle(
          fontSize: 16,
          fontWeight: FontWeight.bold,
          color: Colors.red,
          decoration: TextDecoration.underline,
        ),
      ),
      TextSpan(
        text: '，可以继续阅读。',
        style: TextStyle(
          fontSize: 16,
          color: Colors.grey[700],
        ),
      ),
    ],
  ),
  style: TextStyle(fontSize: 16),
)
```

## 🎯 最佳实践

### 1. 性能优化

- **避免频繁重建**：使用 `const` 构造函数
- **合理使用选择控制器**：避免创建不必要的自定义控制器
- **控制文本长度**：对于超长文本考虑分页显示

### 2. 用户体验

- **提供选择提示**：在适当的地方提示用户可以选择文本
- **保持一致性**：与系统默认的选择行为保持一致
- **响应式设计**：在不同屏幕尺寸下保持良好的显示效果

### 3. 可访问性

- **语义标签**：为可选择文本提供合适的语义信息
- **键盘导航**：支持键盘操作进行文本选择
- **屏幕阅读器**：确保屏幕阅读器能正确读取可选择文本

### 4. 代码组织

```dart
class SelectableTextHelper {
  // 提取常用的样式配置
  static TextStyle getDefaultStyle() {
    return TextStyle(
      fontSize: 16,
      color: Colors.black87,
      height: 1.5,
    );
  }

  static TextStyle getTitleStyle() {
    return TextStyle(
      fontSize: 18,
      fontWeight: FontWeight.bold,
      color: Colors.black,
    );
  }

  // 提取常用的选择处理逻辑
  static void handleSelectionChanged(
    TextSelection selection,
    SelectionChangedCause cause,
    String text,
  ) {
    if (selection.isValid) {
      String selectedText = selection.textInside(text);
      print('选中文本：$selectedText');
      
      // 可以在这里添加自定义的处理逻辑
      // 比如高亮显示、保存到历史记录等
    }
  }
}
```

## 📚 相关链接

- [Text 基础文本](text-basic.md) - 学习基础文本显示
- [RichText 富文本](text-richtext.md) - 学习富文本显示
- [文本输入控件](text-input.md) - 学习文本输入
- [Flutter 官方文档](https://api.flutter.dev/flutter/material/SelectableText-class.html)

---

**总结**：SelectableText 是 Flutter 中功能强大的可选择文本组件。通过合理使用选择控制器、样式定制和交互处理，可以创建出用户友好、功能完善的文本选择体验。 