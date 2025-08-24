# 📝 Flutter 国际化文本处理详解

> 掌握 Flutter 中多语言支持和国际化文本处理的核心技术

![Flutter Internationalization](https://img.shields.io/badge/Flutter-Internationalization-blue?style=for-the-badge&logo=flutter)
![Version](https://img.shields.io/badge/Version-1.0.0-green?style=for-the-badge)
![License](https://img.shields.io/badge/License-MIT-yellow?style=for-the-badge)

## 📋 目录导航

- [国际化概述](#国际化概述)
- [基础配置](#基础配置)
- [文本本地化](#文本本地化)
- [日期和数字格式化](#日期和数字格式化)
- [复数处理](#复数处理)
- [RTL 支持](#rtl-支持)
- [最佳实践](#最佳实践)

---

## 🎯 国际化概述

Flutter 提供了完整的国际化支持，能够轻松实现多语言应用，支持文本、日期、数字等内容的本地化。

### 核心特性

- **多语言支持**：支持多种语言的文本显示
- **日期格式化**：根据地区格式化日期和时间
- **数字格式化**：根据地区格式化数字和货币
- **复数处理**：支持不同语言的复数规则
- **RTL 支持**：支持从右到左的语言布局

### 支持的语言

- **中文**：简体中文、繁体中文
- **英文**：美式英语、英式英语
- **日文**：日语
- **韩文**：韩语
- **阿拉伯文**：阿拉伯语（RTL）
- **其他语言**：法语、德语、西班牙语等

## ⚙️ 基础配置

### 1. 添加依赖

```yaml
dependencies:
  flutter:
    sdk: flutter
  flutter_localizations:
    sdk: flutter
  intl: ^0.18.0

flutter:
  generate: true
```

### 2. 配置支持的语言

```dart
import 'package:flutter/material.dart';
import 'package:flutter_localizations/flutter_localizations.dart';
import 'package:flutter_gen/gen_l10n/app_localizations.dart';

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: '国际化示例',
      
      // 支持的语言列表
      supportedLocales: [
        Locale('zh', 'CN'), // 简体中文
        Locale('zh', 'TW'), // 繁体中文
        Locale('en', 'US'), // 美式英语
        Locale('en', 'GB'), // 英式英语
        Locale('ja', 'JP'), // 日语
        Locale('ko', 'KR'), // 韩语
        Locale('ar', 'SA'), // 阿拉伯语
      ],
      
      // 本地化代理
      localizationsDelegates: [
        AppLocalizations.delegate,
        GlobalMaterialLocalizations.delegate,
        GlobalWidgetsLocalizations.delegate,
        GlobalCupertinoLocalizations.delegate,
      ],
      
      // 语言代码映射
      localeResolutionCallback: (locale, supportedLocales) {
        // 检查支持的语言
        for (var supportedLocale in supportedLocales) {
          if (supportedLocale.languageCode == locale?.languageCode &&
              supportedLocale.countryCode == locale?.countryCode) {
            return supportedLocale;
          }
        }
        
        // 如果没有完全匹配，尝试匹配语言代码
        for (var supportedLocale in supportedLocales) {
          if (supportedLocale.languageCode == locale?.languageCode) {
            return supportedLocale;
          }
        }
        
        // 默认返回第一个支持的语言
        return supportedLocales.first;
      },
      
      home: InternationalizationExample(),
    );
  }
}
```

### 3. 创建本地化文件

在 `lib/l10n/` 目录下创建 `app_en.arb` 和 `app_zh.arb` 文件：

**app_en.arb**：
```json
{
  "appTitle": "Internationalization Example",
  "welcome": "Welcome",
  "hello": "Hello, {name}!",
  "@hello": {
    "description": "A greeting message",
    "placeholders": {
      "name": {
        "type": "String",
        "example": "John"
      }
    }
  },
  "userCount": "{count, plural, =0{No users} =1{1 user} other{{count} users}}",
  "@userCount": {
    "description": "Number of users",
    "placeholders": {
      "count": {
        "type": "int"
      }
    }
  },
  "today": "Today",
  "yesterday": "Yesterday",
  "daysAgo": "{count} days ago",
  "@daysAgo": {
    "description": "Days ago",
    "placeholders": {
      "count": {
        "type": "int"
      }
    }
  },
  "price": "Price: {amount}",
  "@price": {
    "description": "Price display",
    "placeholders": {
      "amount": {
        "type": "double"
      }
    }
  },
  "settings": "Settings",
  "language": "Language",
  "theme": "Theme",
  "light": "Light",
  "dark": "Dark",
  "system": "System"
}
```

**app_zh.arb**：
```json
{
  "appTitle": "国际化示例",
  "welcome": "欢迎",
  "hello": "你好，{name}！",
  "userCount": "{count, plural, =0{没有用户} =1{1个用户} other{{count}个用户}}",
  "today": "今天",
  "yesterday": "昨天",
  "daysAgo": "{count}天前",
  "price": "价格：{amount}",
  "settings": "设置",
  "language": "语言",
  "theme": "主题",
  "light": "浅色",
  "dark": "深色",
  "system": "系统"
}
```

## 📝 文本本地化

### 基础文本本地化

```dart
class InternationalizationExample extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    // 获取本地化实例
    final l10n = AppLocalizations.of(context)!;
    
    return Scaffold(
      appBar: AppBar(
        title: Text(l10n.appTitle),
      ),
      body: SingleChildScrollView(
        padding: EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            // 基础文本
            Text(l10n.welcome, style: TextStyle(fontSize: 24)),
            SizedBox(height: 16),
            
            // 带参数的文本
            Text(l10n.hello('张三'), style: TextStyle(fontSize: 18)),
            SizedBox(height: 16),
            
            // 复数处理
            Text(l10n.userCount(0), style: TextStyle(fontSize: 16)),
            Text(l10n.userCount(1), style: TextStyle(fontSize: 16)),
            Text(l10n.userCount(5), style: TextStyle(fontSize: 16)),
            SizedBox(height: 16),
            
            // 时间相关
            Text(l10n.today, style: TextStyle(fontSize: 16)),
            Text(l10n.yesterday, style: TextStyle(fontSize: 16)),
            Text(l10n.daysAgo(3), style: TextStyle(fontSize: 16)),
            SizedBox(height: 16),
            
            // 价格格式化
            Text(l10n.price(99.99), style: TextStyle(fontSize: 16)),
            SizedBox(height: 32),
            
            // 语言切换
            _buildLanguageSelector(context),
          ],
        ),
      ),
    );
  }

  Widget _buildLanguageSelector(BuildContext context) {
    return Card(
      child: Padding(
        padding: EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Text(
              'Language Settings',
              style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold),
            ),
            SizedBox(height: 16),
            
            Wrap(
              spacing: 8,
              children: [
                _buildLanguageButton(context, Locale('zh', 'CN'), '中文'),
                _buildLanguageButton(context, Locale('en', 'US'), 'English'),
                _buildLanguageButton(context, Locale('ja', 'JP'), '日本語'),
                _buildLanguageButton(context, Locale('ko', 'KR'), '한국어'),
              ],
            ),
          ],
        ),
      ),
    );
  }

  Widget _buildLanguageButton(BuildContext context, Locale locale, String label) {
    return ElevatedButton(
      onPressed: () {
        // 切换语言
        _changeLanguage(context, locale);
      },
      child: Text(label),
    );
  }

  void _changeLanguage(BuildContext context, Locale locale) {
    // 这里需要实现语言切换逻辑
    // 通常使用 Provider 或 GetX 等状态管理工具
    print('切换到语言：$locale');
  }
}
```

## 📅 日期和数字格式化

### 日期格式化

```dart
class DateFormattingExample extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final now = DateTime.now();
    final locale = Localizations.localeOf(context);
    
    return Column(
      crossAxisAlignment: CrossAxisAlignment.start,
      children: [
        Text('日期格式化示例', style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold)),
        SizedBox(height: 16),
        
        // 完整日期
        Text(
          DateFormat.yMMMMd(locale.languageCode).format(now),
          style: TextStyle(fontSize: 16),
        ),
        
        // 短日期
        Text(
          DateFormat.yMMMd(locale.languageCode).format(now),
          style: TextStyle(fontSize: 16),
        ),
        
        // 时间
        Text(
          DateFormat.Hm(locale.languageCode).format(now),
          style: TextStyle(fontSize: 16),
        ),
        
        // 完整日期时间
        Text(
          DateFormat.yMMMMd(locale.languageCode).add_jm().format(now),
          style: TextStyle(fontSize: 16),
        ),
        
        // 相对时间
        Text(
          _getRelativeTime(now, locale),
          style: TextStyle(fontSize: 16),
        ),
      ],
    );
  }

  String _getRelativeTime(DateTime dateTime, Locale locale) {
    final now = DateTime.now();
    final difference = now.difference(dateTime);
    
    if (difference.inDays > 0) {
      return DateFormat.yMMMd(locale.languageCode).format(dateTime);
    } else if (difference.inHours > 0) {
      return '${difference.inHours} hours ago';
    } else if (difference.inMinutes > 0) {
      return '${difference.inMinutes} minutes ago';
    } else {
      return 'Just now';
    }
  }
}
```

### 数字格式化

```dart
class NumberFormattingExample extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final locale = Localizations.localeOf(context);
    final number = 1234567.89;
    final currency = 99.99;
    
    return Column(
      crossAxisAlignment: CrossAxisAlignment.start,
      children: [
        Text('数字格式化示例', style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold)),
        SizedBox(height: 16),
        
        // 数字格式化
        Text(
          NumberFormat('#,###.##', locale.languageCode).format(number),
          style: TextStyle(fontSize: 16),
        ),
        
        // 货币格式化
        Text(
          NumberFormat.currency(
            locale: locale.languageCode,
            symbol: _getCurrencySymbol(locale.languageCode),
          ).format(currency),
          style: TextStyle(fontSize: 16),
        ),
        
        // 百分比
        Text(
          NumberFormat.percentPattern(locale.languageCode).format(0.85),
          style: TextStyle(fontSize: 16),
        ),
        
        // 科学计数法
        Text(
          NumberFormat.scientificPattern(locale.languageCode).format(1234567),
          style: TextStyle(fontSize: 16),
        ),
      ],
    );
  }

  String _getCurrencySymbol(String languageCode) {
    switch (languageCode) {
      case 'zh':
        return '¥';
      case 'en':
        return '\$';
      case 'ja':
        return '¥';
      case 'ko':
        return '₩';
      default:
        return '\$';
    }
  }
}
```

## 🔢 复数处理

### 复数规则处理

```dart
class PluralExample extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final l10n = AppLocalizations.of(context)!;
    
    return Column(
      crossAxisAlignment: CrossAxisAlignment.start,
      children: [
        Text('复数处理示例', style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold)),
        SizedBox(height: 16),
        
        // 不同数量的用户
        Text(l10n.userCount(0)),
        Text(l10n.userCount(1)),
        Text(l10n.userCount(2)),
        Text(l10n.userCount(5)),
        Text(l10n.userCount(10)),
        
        SizedBox(height: 16),
        
        // 自定义复数处理
        _buildCustomPlural(context),
      ],
    );
  }

  Widget _buildCustomPlural(BuildContext context) {
    final locale = Localizations.localeOf(context);
    
    return Column(
      crossAxisAlignment: CrossAxisAlignment.start,
      children: [
        Text('自定义复数处理：'),
        Text(_getPluralText(0, 'item', locale)),
        Text(_getPluralText(1, 'item', locale)),
        Text(_getPluralText(2, 'item', locale)),
        Text(_getPluralText(5, 'item', locale)),
      ],
    );
  }

  String _getPluralText(int count, String item, Locale locale) {
    // 简单的复数规则
    if (locale.languageCode == 'zh') {
      return count == 0 ? '没有$item' : '$count个$item';
    } else {
      return count == 1 ? '1 $item' : '$count ${item}s';
    }
  }
}
```

## 🌐 RTL 支持

### 从右到左语言支持

```dart
class RTLExample extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final locale = Localizations.localeOf(context);
    final isRTL = locale.languageCode == 'ar';
    
    return Directionality(
      textDirection: isRTL ? TextDirection.rtl : TextDirection.ltr,
      child: Scaffold(
        appBar: AppBar(
          title: Text('RTL 支持示例'),
        ),
        body: SingleChildScrollView(
          padding: EdgeInsets.all(16),
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: [
              // 文本对齐
              Text(
                '这是测试文本',
                textAlign: isRTL ? TextAlign.right : TextAlign.left,
                style: TextStyle(fontSize: 18),
              ),
              
              SizedBox(height: 16),
              
              // 按钮布局
              Row(
                mainAxisAlignment: isRTL 
                    ? MainAxisAlignment.end 
                    : MainAxisAlignment.start,
                children: [
                  ElevatedButton(
                    onPressed: () {},
                    child: Text('按钮 1'),
                  ),
                  SizedBox(width: 8),
                  ElevatedButton(
                    onPressed: () {},
                    child: Text('按钮 2'),
                  ),
                ],
              ),
              
              SizedBox(height: 16),
              
              // 图标和文本
              Row(
                children: [
                  if (!isRTL) ...[
                    Icon(Icons.star),
                    SizedBox(width: 8),
                    Text('评分'),
                  ] else ...[
                    Text('评分'),
                    SizedBox(width: 8),
                    Icon(Icons.star),
                  ],
                ],
              ),
            ],
          ),
        ),
      ),
    );
  }
}
```

## 🎯 最佳实践

### 1. 文本管理

- **集中管理**：将所有文本字符串集中管理
- **语义化命名**：使用有意义的键名
- **参数化**：使用占位符处理动态内容
- **注释说明**：为复杂文本添加说明

### 2. 性能优化

```dart
class LocalizationHelper {
  // 缓存格式化器
  static final Map<String, DateFormat> _dateFormatters = {};
  static final Map<String, NumberFormat> _numberFormatters = {};
  
  static DateFormat getDateFormatter(String locale) {
    if (!_dateFormatters.containsKey(locale)) {
      _dateFormatters[locale] = DateFormat.yMMMd(locale);
    }
    return _dateFormatters[locale]!;
  }
  
  static NumberFormat getNumberFormatter(String locale) {
    if (!_numberFormatters.containsKey(locale)) {
      _numberFormatters[locale] = NumberFormat('#,###.##', locale);
    }
    return _numberFormatters[locale]!;
  }
}
```

### 3. 错误处理

```dart
class SafeLocalization {
  static String getText(BuildContext context, String key, [List<Object>? args]) {
    try {
      final l10n = AppLocalizations.of(context);
      if (l10n == null) {
        return key; // 返回键名作为后备
      }
      
      // 使用反射或其他方式获取文本
      // 这里需要根据实际的本地化实现来调整
      return key;
    } catch (e) {
      print('本地化错误：$e');
      return key; // 返回键名作为后备
    }
  }
}
```

### 4. 测试支持

```dart
class LocalizationTestHelper {
  static void testLocalization() {
    // 测试所有支持的语言
    final supportedLocales = [
      Locale('zh', 'CN'),
      Locale('en', 'US'),
      Locale('ja', 'JP'),
    ];
    
    for (final locale in supportedLocales) {
      print('测试语言：$locale');
      // 在这里添加具体的测试逻辑
    }
  }
}
```

### 5. 动态语言切换

```dart
class LanguageManager extends ChangeNotifier {
  Locale _currentLocale = Locale('zh', 'CN');
  
  Locale get currentLocale => _currentLocale;
  
  void changeLanguage(Locale locale) {
    _currentLocale = locale;
    notifyListeners();
  }
  
  // 保存语言设置
  Future<void> saveLanguagePreference(Locale locale) async {
    final prefs = await SharedPreferences.getInstance();
    await prefs.setString('language_code', locale.languageCode);
    await prefs.setString('country_code', locale.countryCode ?? '');
  }
  
  // 加载语言设置
  Future<Locale?> loadLanguagePreference() async {
    final prefs = await SharedPreferences.getInstance();
    final languageCode = prefs.getString('language_code');
    final countryCode = prefs.getString('country_code');
    
    if (languageCode != null) {
      return Locale(languageCode, countryCode);
    }
    return null;
  }
}
```

## 📚 相关链接

- [Text 基础文本](text-basic.md) - 学习基础文本显示
- [RichText 富文本](text-richtext.md) - 学习富文本显示
- [文本样式系统](text-styling.md) - 学习样式管理
- [Flutter 官方文档](https://flutter.dev/docs/development/accessibility-and-localization/internationalization)

---

**总结**：Flutter 的国际化系统提供了完整的多语言支持。通过合理配置和使用本地化功能，可以轻松创建支持多种语言的应用，提供更好的用户体验。 