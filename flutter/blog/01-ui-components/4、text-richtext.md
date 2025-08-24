# 📝 Flutter RichText 富文本：让文字更有生命力

> 你是否遇到过这样的场景：一段文字中需要突出显示某些关键词，或者让某些文字可以点击？今天我们就来聊聊 Flutter 中的 RichText 组件，它能让你的文字"活"起来！

![Flutter RichText](https://img.shields.io/badge/Flutter-RichText%20富文本-blue?style=for-the-badge&logo=flutter)

## 🎯 为什么需要 RichText？

想象一下，你正在开发一个聊天应用。用户发送的消息可能是这样的：

> "今天天气真不错，要不要一起去**公园**散步？"

如果只用普通的 `Text` 组件，你无法让"公园"这个词高亮显示，也无法让用户点击它。这就是 `RichText` 的用武之地！

### 实际应用场景

- **聊天应用**：用户名高亮、链接可点击
- **文章阅读**：关键词高亮、标题加粗
- **商品展示**：价格突出、标签彩色
- **搜索结果**：匹配词高亮显示

## 🚀 从简单开始：基础用法

### 第一个 RichText 示例

让我们从一个简单的例子开始，看看 RichText 是如何工作的：

```dart
RichText(
  text: TextSpan(
    style: DefaultTextStyle.of(context).style, // 继承默认样式
    children: [
      TextSpan(text: '你好，'),
      TextSpan(
        text: 'Flutter',
        style: TextStyle(
          color: Colors.blue,
          fontWeight: FontWeight.bold,
        ),
      ),
      TextSpan(text: ' 开发者！'),
    ],
  ),
)
```

这段代码的效果是：普通的"你好，"和"开发者！"，而"Flutter"会以蓝色粗体显示。

### 价格展示的经典案例

电商应用中经常需要展示价格，包括原价和现价：

```dart
RichText(
  text: TextSpan(
    children: [
      TextSpan(
        text: '现价：',
        style: TextStyle(fontSize: 16),
      ),
      TextSpan(
        text: '¥99',
        style: TextStyle(
          fontSize: 24,
          fontWeight: FontWeight.bold,
          color: Colors.red,
        ),
      ),
      TextSpan(text: ' '),
      TextSpan(
        text: '¥199',
        style: TextStyle(
          fontSize: 16,
          decoration: TextDecoration.lineThrough,
          color: Colors.grey,
        ),
      ),
    ],
  ),
)
```

效果：现价：**¥99** ~~¥199~~

## 🎨 让文字可以点击：交互式富文本

### 添加点击事件

RichText 最强大的功能之一就是可以让特定文字响应点击事件：

```dart
RichText(
  text: TextSpan(
    children: [
      TextSpan(text: '点击'),
      TextSpan(
        text: '这里',
        style: TextStyle(
          color: Colors.blue,
          decoration: TextDecoration.underline,
        ),
        recognizer: TapGestureRecognizer()
          ..onTap = () {
            print('用户点击了链接！');
            // 可以导航到其他页面
            Navigator.pushNamed(context, '/detail');
          },
      ),
      TextSpan(text: '查看更多'),
    ],
  ),
)
```

### 实用的联系方式展示

```dart
RichText(
  text: TextSpan(
    children: [
      TextSpan(text: '联系我们：'),
      TextSpan(
        text: '400-123-4567',
        style: TextStyle(
          color: Colors.green,
          fontWeight: FontWeight.bold,
        ),
        recognizer: TapGestureRecognizer()
          ..onTap = () {
            // 拨打电话
            launchUrl(Uri.parse('tel:400-123-4567'));
          },
      ),
      TextSpan(text: ' 或发送邮件至 '),
      TextSpan(
        text: 'support@example.com',
        style: TextStyle(
          color: Colors.blue,
          decoration: TextDecoration.underline,
        ),
        recognizer: TapGestureRecognizer()
          ..onTap = () {
            // 发送邮件
            launchUrl(Uri.parse('mailto:support@example.com'));
          },
      ),
    ],
  ),
)
```

## 📱 实战应用：聊天消息组件

让我们创建一个真实的聊天消息组件，展示 RichText 在实际项目中的应用：

```dart
class ChatMessageWidget extends StatelessWidget {
  final String username;
  final String message;
  final DateTime timestamp;
  final bool isCurrentUser;

  const ChatMessageWidget({
    Key? key,
    required this.username,
    required this.message,
    required this.timestamp,
    required this.isCurrentUser,
  }) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Container(
      margin: EdgeInsets.symmetric(vertical: 4, horizontal: 8),
      child: Row(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          // 头像
          CircleAvatar(
            radius: 16,
            backgroundColor: isCurrentUser ? Colors.blue : Colors.grey,
            child: Text(
              username[0].toUpperCase(),
              style: TextStyle(color: Colors.white, fontSize: 12),
            ),
          ),
          SizedBox(width: 8),

          // 消息内容
          Expanded(
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: [
                // 用户名和时间
                RichText(
                  text: TextSpan(
                    style: TextStyle(fontSize: 12, color: Colors.grey),
                    children: [
                      TextSpan(
                        text: username,
                        style: TextStyle(
                          fontWeight: FontWeight.bold,
                          color: isCurrentUser ? Colors.blue : Colors.black54,
                        ),
                      ),
                      TextSpan(text: ' • '),
                      TextSpan(
                        text: _formatTime(timestamp),
                      ),
                    ],
                  ),
                ),
                SizedBox(height: 4),

                // 消息内容
                _buildMessageContent(context),
              ],
            ),
          ),
        ],
      ),
    );
  }

  Widget _buildMessageContent(BuildContext context) {
    return RichText(
      text: TextSpan(
        style: TextStyle(fontSize: 16, color: Colors.black87),
        children: _parseMessage(message),
      ),
    );
  }

  List<TextSpan> _parseMessage(String message) {
    List<TextSpan> spans = [];

    // 简单的消息解析逻辑
    // 匹配 @用户名 格式
    RegExp userMention = RegExp(r'@(\w+)');
    RegExp urlPattern = RegExp(r'https?://[^\s]+');

    int lastIndex = 0;

    // 查找所有匹配项
    List<RegExpMatch> matches = [];
    matches.addAll(userMention.allMatches(message));
    matches.addAll(urlPattern.allMatches(message));
    matches.sort((a, b) => a.start.compareTo(b.start));

    for (RegExpMatch match in matches) {
      // 添加匹配前的普通文本
      if (match.start > lastIndex) {
        spans.add(TextSpan(
          text: message.substring(lastIndex, match.start),
        ));
      }

      // 添加匹配的文本
      String matchedText = match.group(0)!;
      if (userMention.hasMatch(matchedText)) {
        // @用户名 格式
        spans.add(TextSpan(
          text: matchedText,
          style: TextStyle(
            color: Colors.blue,
            fontWeight: FontWeight.bold,
          ),
          recognizer: TapGestureRecognizer()
            ..onTap = () {
              print('点击了用户：$matchedText');
              // 可以导航到用户资料页
            },
        ));
      } else if (urlPattern.hasMatch(matchedText)) {
        // 链接格式
        spans.add(TextSpan(
          text: matchedText,
          style: TextStyle(
            color: Colors.blue,
            decoration: TextDecoration.underline,
          ),
          recognizer: TapGestureRecognizer()
            ..onTap = () {
              print('点击了链接：$matchedText');
              // 打开链接
              launchUrl(Uri.parse(matchedText));
            },
        ));
      }

      lastIndex = match.end;
    }

    // 添加剩余的普通文本
    if (lastIndex < message.length) {
      spans.add(TextSpan(
        text: message.substring(lastIndex),
      ));
    }

    return spans;
  }

  String _formatTime(DateTime time) {
    return '${time.hour.toString().padLeft(2, '0')}:${time.minute.toString().padLeft(2, '0')}';
  }
}
```

使用示例：

```dart
ChatMessageWidget(
  username: '张三',
  message: '今天天气真不错，要不要一起去公园散步？@李四 你觉得呢？',
  timestamp: DateTime.now(),
  isCurrentUser: false,
)
```

## 🎯 文章内容展示：更复杂的应用

让我们创建一个文章内容展示组件，展示 RichText 在内容展示中的应用：

```dart
class ArticleContentWidget extends StatelessWidget {
  final String title;
  final String author;
  final String content;
  final List<String> tags;

  const ArticleContentWidget({
    Key? key,
    required this.title,
    required this.author,
    required this.content,
    required this.tags,
  }) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Card(
      margin: EdgeInsets.all(16),
      child: Padding(
        padding: EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            // 标题
            RichText(
              text: TextSpan(
                style: TextStyle(
                  fontSize: 24,
                  fontWeight: FontWeight.bold,
                  color: Colors.black,
                ),
                children: [
                  TextSpan(text: title),
                ],
              ),
            ),
            SizedBox(height: 8),

            // 作者信息
            RichText(
              text: TextSpan(
                style: TextStyle(fontSize: 14, color: Colors.grey[600]),
                children: [
                  TextSpan(text: '作者：'),
                  TextSpan(
                    text: author,
                    style: TextStyle(
                      color: Colors.blue,
                      fontWeight: FontWeight.w500,
                    ),
                    recognizer: TapGestureRecognizer()
                      ..onTap = () {
                        print('查看作者资料：$author');
                      },
                  ),
                  TextSpan(text: ' • ${DateTime.now().toString().substring(0, 10)}'),
                ],
              ),
            ),
            SizedBox(height: 16),

            // 文章内容
            _buildArticleContent(context),
            SizedBox(height: 16),

            // 标签
            _buildTags(context),
          ],
        ),
      ),
    );
  }

  Widget _buildArticleContent(BuildContext context) {
    return RichText(
      text: TextSpan(
        style: TextStyle(
          fontSize: 16,
          height: 1.6,
          color: Colors.black87,
        ),
        children: _parseArticleContent(content),
      ),
    );
  }

  List<TextSpan> _parseArticleContent(String content) {
    List<TextSpan> spans = [];

    // 简单的文章内容解析
    // 这里可以根据实际需求实现更复杂的解析逻辑

    // 示例：高亮关键词
    List<String> keywords = ['Flutter', 'Dart', 'Widget', '性能'];

    String remainingContent = content;
    int offset = 0;

    for (String keyword in keywords) {
      int index = remainingContent.toLowerCase().indexOf(keyword.toLowerCase());
      if (index != -1) {
        // 添加关键词前的文本
        if (index > 0) {
          spans.add(TextSpan(
            text: remainingContent.substring(0, index),
          ));
        }

        // 添加高亮的关键词
        spans.add(TextSpan(
          text: remainingContent.substring(index, index + keyword.length),
          style: TextStyle(
            fontWeight: FontWeight.bold,
            backgroundColor: Colors.yellow[200],
          ),
        ));

        // 更新剩余内容
        remainingContent = remainingContent.substring(index + keyword.length);
        offset += index + keyword.length;
      }
    }

    // 添加剩余内容
    if (remainingContent.isNotEmpty) {
      spans.add(TextSpan(text: remainingContent));
    }

    return spans;
  }

  Widget _buildTags(BuildContext context) {
    return Wrap(
      spacing: 8,
      children: tags.map((tag) {
        return GestureDetector(
          onTap: () {
            print('点击了标签：$tag');
            // 可以导航到标签页面
          },
          child: Container(
            padding: EdgeInsets.symmetric(horizontal: 12, vertical: 6),
            decoration: BoxDecoration(
              color: Colors.blue[100],
              borderRadius: BorderRadius.circular(16),
            ),
            child: Text(
              '#$tag',
              style: TextStyle(
                color: Colors.blue[800],
                fontSize: 12,
                fontWeight: FontWeight.w500,
              ),
            ),
          ),
        );
      }).toList(),
    );
  }
}
```

## 💡 实用技巧和最佳实践

### 1. 性能优化

**问题**：当 RichText 内容很复杂时，可能会影响性能。

**解决方案**：

```dart
// 缓存复杂的富文本内容
class CachedRichText extends StatelessWidget {
  final String content;
  final TextStyle? style;

  // 使用缓存避免重复解析
  static final Map<String, TextSpan> _cache = {};

  CachedRichText({
    Key? key,
    required this.content,
    this.style,
  }) : super(key: key);

  @override
  Widget build(BuildContext context) {
    TextSpan? cachedSpan = _cache[content];
    if (cachedSpan == null) {
      cachedSpan = _parseContent(content);
      _cache[content] = cachedSpan;
    }

    return RichText(
      text: cachedSpan,
      style: style,
    );
  }

  TextSpan _parseContent(String content) {
    // 解析逻辑
    return TextSpan(text: content);
  }
}
```

### 2. 错误处理

**问题**：当 RichText 解析失败时，应用可能会崩溃。

**解决方案**：

```dart
Widget _buildSafeRichText(String content) {
  try {
    return RichText(
      text: _parseContent(content),
    );
  } catch (e) {
    // 降级到普通文本
    return Text(
      content,
      style: TextStyle(color: Colors.grey),
    );
  }
}
```

### 3. 可访问性支持

```dart
Widget _buildAccessibleRichText() {
  return Semantics(
    label: '包含链接和关键词的文本内容',
    child: RichText(
      text: TextSpan(
        children: [
          TextSpan(text: '点击'),
          TextSpan(
            text: '这里',
            style: TextStyle(color: Colors.blue),
            recognizer: TapGestureRecognizer()
              ..onTap = () {
                // 处理点击
              },
          ),
          TextSpan(text: '查看更多'),
        ],
      ),
    ),
  );
}
```

## 🎨 样式组合技巧

### 常用样式组合

```dart
class RichTextStyles {
  // 链接样式
  static TextStyle linkStyle = TextStyle(
    color: Colors.blue,
    decoration: TextDecoration.underline,
  );

  // 关键词高亮样式
  static TextStyle highlightStyle = TextStyle(
    fontWeight: FontWeight.bold,
    backgroundColor: Colors.yellow[200],
  );

  // 代码样式
  static TextStyle codeStyle = TextStyle(
    fontFamily: 'monospace',
    backgroundColor: Colors.grey[200],
    color: Colors.red[800],
  );

  // 标题样式
  static TextStyle titleStyle = TextStyle(
    fontSize: 20,
    fontWeight: FontWeight.bold,
    color: Colors.black,
  );
}
```

### 主题适配

```dart
Widget _buildThemedRichText(BuildContext context) {
  final isDark = Theme.of(context).brightness == Brightness.dark;

  return RichText(
    text: TextSpan(
      style: TextStyle(
        color: isDark ? Colors.white : Colors.black,
      ),
      children: [
        TextSpan(text: '普通文本'),
        TextSpan(
          text: '高亮文本',
          style: TextStyle(
            backgroundColor: isDark ? Colors.blue[900] : Colors.yellow[200],
            color: isDark ? Colors.white : Colors.black,
          ),
        ),
      ],
    ),
  );
}
```

## 🚀 进阶应用：自定义解析器

如果你需要更复杂的文本解析功能，可以创建自定义的解析器：

```dart
class MarkdownStyleParser {
  static List<TextSpan> parse(String text) {
    List<TextSpan> spans = [];

    // 解析 **粗体**
    RegExp boldPattern = RegExp(r'\*\*(.*?)\*\*');
    List<RegExpMatch> boldMatches = boldPattern.allMatches(text).toList();

    // 解析 *斜体*
    RegExp italicPattern = RegExp(r'\*(.*?)\*');
    List<RegExpMatch> italicMatches = italicPattern.allMatches(text).toList();

    // 解析 [链接](URL)
    RegExp linkPattern = RegExp(r'\[(.*?)\]\((.*?)\)');
    List<RegExpMatch> linkMatches = linkPattern.allMatches(text).toList();

    // 合并所有匹配项并排序
    List<MapEntry<int, TextSpan>> allMatches = [];

    // 处理粗体
    for (RegExpMatch match in boldMatches) {
      allMatches.add(MapEntry(
        match.start,
        TextSpan(
          text: match.group(1),
          style: TextStyle(fontWeight: FontWeight.bold),
        ),
      ));
    }

    // 处理斜体
    for (RegExpMatch match in italicMatches) {
      allMatches.add(MapEntry(
        match.start,
        TextSpan(
          text: match.group(1),
          style: TextStyle(fontStyle: FontStyle.italic),
        ),
      ));
    }

    // 处理链接
    for (RegExpMatch match in linkMatches) {
      allMatches.add(MapEntry(
        match.start,
        TextSpan(
          text: match.group(1),
          style: TextStyle(
            color: Colors.blue,
            decoration: TextDecoration.underline,
          ),
          recognizer: TapGestureRecognizer()
            ..onTap = () {
              launchUrl(Uri.parse(match.group(2)!));
            },
        ),
      ));
    }

    // 按位置排序
    allMatches.sort((a, b) => a.key.compareTo(b.key));

    // 构建最终的 TextSpan 列表
    int lastIndex = 0;
    for (MapEntry<int, TextSpan> entry in allMatches) {
      if (entry.key > lastIndex) {
        spans.add(TextSpan(text: text.substring(lastIndex, entry.key)));
      }
      spans.add(entry.value);
      lastIndex = entry.key + entry.value.text!.length;
    }

    if (lastIndex < text.length) {
      spans.add(TextSpan(text: text.substring(lastIndex)));
    }

    return spans;
  }
}
```

使用示例：

```dart
RichText(
  text: TextSpan(
    children: MarkdownStyleParser.parse(
      '这是**粗体**文本，这是*斜体*文本，这是[链接](https://flutter.dev)',
    ),
  ),
)
```

## 📚 总结

RichText 是 Flutter 中非常强大的文本组件，它让我们能够：

1. **组合多种样式**：在同一段文字中应用不同的字体、颜色、大小等样式
2. **添加交互功能**：让特定文字可以响应点击、长按等手势
3. **创建丰富的内容**：支持复杂的文本布局和展示需求

### 关键要点

- **TextSpan** 是 RichText 的核心，每个 TextSpan 可以有自己的样式和手势识别器
- **recognizer** 属性用于添加交互功能，常用的有 TapGestureRecognizer、LongPressGestureRecognizer 等
- **性能优化**：对于复杂的富文本内容，考虑使用缓存和错误处理
- **可访问性**：为重要的交互元素添加语义信息

### 下一步学习

掌握了 RichText 的基础用法后，你可以继续学习：

- [Text 基础文本](text-basic.md) - 深入学习基础文本组件
- [文本输入控件](text-input.md) - 学习文本输入和编辑
- [文本样式系统](text-styling.md) - 掌握 Flutter 的样式系统

记住，RichText 的强大之处在于它的灵活性。你可以根据实际需求创建各种复杂的文本展示效果，让用户界面更加生动和友好！

---

<div align="center">

**🌟 如果这篇文章对你有帮助，请给个 Star 支持一下！ 🌟**

[![GitHub stars](https://img.shields.io/github/stars/1989allen126/language-tutorial?style=social)](https://github.com/1989allen126/language-tutorial)
[![GitHub forks](https://img.shields.io/github/forks/1989allen126/language-tutorial?style=social)](https://github.com/1989allen126/language-tutorial)

</div>
