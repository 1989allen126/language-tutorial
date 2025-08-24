# 📝 Flutter 文本样式和主题系统详解

> 掌握 Flutter 中文本样式配置和主题管理的核心概念

![Flutter Text Styling](https://img.shields.io/badge/Flutter-Text%20Styling-blue?style=for-the-badge&logo=flutter)
![Version](https://img.shields.io/badge/Version-1.0.0-green?style=for-the-badge)
![License](https://img.shields.io/badge/License-MIT-yellow?style=for-the-badge)

## 📋 目录导航

- [样式系统概述](#样式系统概述)
- [TextStyle 详解](#textstyle-详解)
- [TextTheme 主题管理](#texttheme-主题管理)
- [自定义样式](#自定义样式)
- [主题切换](#主题切换)
- [最佳实践](#最佳实践)

---

## 🎯 样式系统概述

Flutter 提供了强大的文本样式系统，通过 `TextStyle` 和 `TextTheme` 来管理文本的外观和主题。

### 核心组件

- **TextStyle**：定义单个文本的样式属性
- **TextTheme**：定义应用整体的文本主题
- **DefaultTextStyle**：为子组件提供默认文本样式
- **Theme**：管理应用的整体主题

### 样式层次结构

```
Theme (应用主题)
├── TextTheme (文本主题)
│   ├── displayLarge (大标题)
│   ├── displayMedium (中标题)
│   ├── displaySmall (小标题)
│   ├── headlineLarge (大标题)
│   ├── headlineMedium (中标题)
│   ├── headlineSmall (小标题)
│   ├── titleLarge (大标题)
│   ├── titleMedium (中标题)
│   ├── titleSmall (小标题)
│   ├── bodyLarge (大正文)
│   ├── bodyMedium (中正文)
│   ├── bodySmall (小正文)
│   ├── labelLarge (大标签)
│   ├── labelMedium (中标签)
│   └── labelSmall (小标签)
└── 其他主题配置
```

## 📝 TextStyle 详解

### 基础样式属性

```dart
TextStyle(
  // 字体大小
  fontSize: 16.0,
  
  // 字体粗细
  fontWeight: FontWeight.normal,
  
  // 字体颜色
  color: Colors.black,
  
  // 字体族
  fontFamily: 'Roboto',
  
  // 行高
  height: 1.5,
  
  // 字母间距
  letterSpacing: 0.5,
  
  // 单词间距
  wordSpacing: 1.0,
  
  // 文本装饰
  decoration: TextDecoration.none,
  decorationColor: Colors.red,
  decorationStyle: TextDecorationStyle.solid,
  decorationThickness: 1.0,
  
  // 背景色
  backgroundColor: Colors.yellow,
  
  // 阴影
  shadows: [
    Shadow(
      offset: Offset(1, 1),
      blurRadius: 2.0,
      color: Colors.grey,
    ),
  ],
)
```

### 常用样式组合

```dart
// 标题样式
TextStyle titleStyle = TextStyle(
  fontSize: 24,
  fontWeight: FontWeight.bold,
  color: Colors.black87,
  letterSpacing: 0.5,
);

// 正文样式
TextStyle bodyStyle = TextStyle(
  fontSize: 16,
  fontWeight: FontWeight.normal,
  color: Colors.black54,
  height: 1.5,
);

// 强调样式
TextStyle emphasisStyle = TextStyle(
  fontSize: 16,
  fontWeight: FontWeight.w600,
  color: Colors.blue,
  decoration: TextDecoration.underline,
);

// 小字样式
TextStyle captionStyle = TextStyle(
  fontSize: 12,
  fontWeight: FontWeight.normal,
  color: Colors.grey,
  fontStyle: FontStyle.italic,
);
```

## 🎨 TextTheme 主题管理

### 创建自定义主题

```dart
class CustomTextTheme {
  static TextTheme get lightTheme {
    return TextTheme(
      // 显示文本
      displayLarge: TextStyle(
        fontSize: 57,
        fontWeight: FontWeight.w400,
        letterSpacing: -0.25,
        color: Colors.black87,
      ),
      displayMedium: TextStyle(
        fontSize: 45,
        fontWeight: FontWeight.w400,
        letterSpacing: 0,
        color: Colors.black87,
      ),
      displaySmall: TextStyle(
        fontSize: 36,
        fontWeight: FontWeight.w400,
        letterSpacing: 0,
        color: Colors.black87,
      ),
      
      // 标题文本
      headlineLarge: TextStyle(
        fontSize: 32,
        fontWeight: FontWeight.w400,
        letterSpacing: 0,
        color: Colors.black87,
      ),
      headlineMedium: TextStyle(
        fontSize: 28,
        fontWeight: FontWeight.w400,
        letterSpacing: 0,
        color: Colors.black87,
      ),
      headlineSmall: TextStyle(
        fontSize: 24,
        fontWeight: FontWeight.w400,
        letterSpacing: 0,
        color: Colors.black87,
      ),
      
      // 标题
      titleLarge: TextStyle(
        fontSize: 22,
        fontWeight: FontWeight.w500,
        letterSpacing: 0,
        color: Colors.black87,
      ),
      titleMedium: TextStyle(
        fontSize: 16,
        fontWeight: FontWeight.w500,
        letterSpacing: 0.15,
        color: Colors.black87,
      ),
      titleSmall: TextStyle(
        fontSize: 14,
        fontWeight: FontWeight.w500,
        letterSpacing: 0.1,
        color: Colors.black87,
      ),
      
      // 正文
      bodyLarge: TextStyle(
        fontSize: 16,
        fontWeight: FontWeight.w400,
        letterSpacing: 0.5,
        color: Colors.black87,
      ),
      bodyMedium: TextStyle(
        fontSize: 14,
        fontWeight: FontWeight.w400,
        letterSpacing: 0.25,
        color: Colors.black87,
      ),
      bodySmall: TextStyle(
        fontSize: 12,
        fontWeight: FontWeight.w400,
        letterSpacing: 0.4,
        color: Colors.black54,
      ),
      
      // 标签
      labelLarge: TextStyle(
        fontSize: 14,
        fontWeight: FontWeight.w500,
        letterSpacing: 0.1,
        color: Colors.black87,
      ),
      labelMedium: TextStyle(
        fontSize: 12,
        fontWeight: FontWeight.w500,
        letterSpacing: 0.5,
        color: Colors.black87,
      ),
      labelSmall: TextStyle(
        fontSize: 11,
        fontWeight: FontWeight.w500,
        letterSpacing: 0.5,
        color: Colors.black87,
      ),
    );
  }

  static TextTheme get darkTheme {
    return TextTheme(
      // 显示文本
      displayLarge: TextStyle(
        fontSize: 57,
        fontWeight: FontWeight.w400,
        letterSpacing: -0.25,
        color: Colors.white,
      ),
      displayMedium: TextStyle(
        fontSize: 45,
        fontWeight: FontWeight.w400,
        letterSpacing: 0,
        color: Colors.white,
      ),
      displaySmall: TextStyle(
        fontSize: 36,
        fontWeight: FontWeight.w400,
        letterSpacing: 0,
        color: Colors.white,
      ),
      
      // 标题文本
      headlineLarge: TextStyle(
        fontSize: 32,
        fontWeight: FontWeight.w400,
        letterSpacing: 0,
        color: Colors.white,
      ),
      headlineMedium: TextStyle(
        fontSize: 28,
        fontWeight: FontWeight.w400,
        letterSpacing: 0,
        color: Colors.white,
      ),
      headlineSmall: TextStyle(
        fontSize: 24,
        fontWeight: FontWeight.w400,
        letterSpacing: 0,
        color: Colors.white,
      ),
      
      // 标题
      titleLarge: TextStyle(
        fontSize: 22,
        fontWeight: FontWeight.w500,
        letterSpacing: 0,
        color: Colors.white,
      ),
      titleMedium: TextStyle(
        fontSize: 16,
        fontWeight: FontWeight.w500,
        letterSpacing: 0.15,
        color: Colors.white,
      ),
      titleSmall: TextStyle(
        fontSize: 14,
        fontWeight: FontWeight.w500,
        letterSpacing: 0.1,
        color: Colors.white,
      ),
      
      // 正文
      bodyLarge: TextStyle(
        fontSize: 16,
        fontWeight: FontWeight.w400,
        letterSpacing: 0.5,
        color: Colors.white70,
      ),
      bodyMedium: TextStyle(
        fontSize: 14,
        fontWeight: FontWeight.w400,
        letterSpacing: 0.25,
        color: Colors.white70,
      ),
      bodySmall: TextStyle(
        fontSize: 12,
        fontWeight: FontWeight.w400,
        letterSpacing: 0.4,
        color: Colors.white60,
      ),
      
      // 标签
      labelLarge: TextStyle(
        fontSize: 14,
        fontWeight: FontWeight.w500,
        letterSpacing: 0.1,
        color: Colors.white,
      ),
      labelMedium: TextStyle(
        fontSize: 12,
        fontWeight: FontWeight.w500,
        letterSpacing: 0.5,
        color: Colors.white,
      ),
      labelSmall: TextStyle(
        fontSize: 11,
        fontWeight: FontWeight.w500,
        letterSpacing: 0.5,
        color: Colors.white,
      ),
    );
  }
}
```

### 应用主题

```dart
class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: '文本样式示例',
      theme: ThemeData(
        textTheme: CustomTextTheme.lightTheme,
        primarySwatch: Colors.blue,
      ),
      darkTheme: ThemeData(
        textTheme: CustomTextTheme.darkTheme,
        primarySwatch: Colors.blue,
      ),
      home: TextStylingExample(),
    );
  }
}
```

## 🎨 自定义样式

### 创建样式工具类

```dart
class AppTextStyles {
  // 私有构造函数，防止实例化
  AppTextStyles._();

  // 标题样式
  static TextStyle get h1 => TextStyle(
    fontSize: 32,
    fontWeight: FontWeight.bold,
    color: Colors.black87,
    letterSpacing: -0.5,
  );

  static TextStyle get h2 => TextStyle(
    fontSize: 28,
    fontWeight: FontWeight.w600,
    color: Colors.black87,
    letterSpacing: -0.25,
  );

  static TextStyle get h3 => TextStyle(
    fontSize: 24,
    fontWeight: FontWeight.w600,
    color: Colors.black87,
    letterSpacing: 0,
  );

  static TextStyle get h4 => TextStyle(
    fontSize: 20,
    fontWeight: FontWeight.w500,
    color: Colors.black87,
    letterSpacing: 0.15,
  );

  // 正文样式
  static TextStyle get body1 => TextStyle(
    fontSize: 16,
    fontWeight: FontWeight.normal,
    color: Colors.black87,
    height: 1.5,
    letterSpacing: 0.5,
  );

  static TextStyle get body2 => TextStyle(
    fontSize: 14,
    fontWeight: FontWeight.normal,
    color: Colors.black54,
    height: 1.4,
    letterSpacing: 0.25,
  );

  // 强调样式
  static TextStyle get emphasis => TextStyle(
    fontSize: 16,
    fontWeight: FontWeight.w600,
    color: Colors.blue[700],
    decoration: TextDecoration.underline,
  );

  // 链接样式
  static TextStyle get link => TextStyle(
    fontSize: 16,
    fontWeight: FontWeight.normal,
    color: Colors.blue,
    decoration: TextDecoration.underline,
  );

  // 错误样式
  static TextStyle get error => TextStyle(
    fontSize: 14,
    fontWeight: FontWeight.normal,
    color: Colors.red,
  );

  // 成功样式
  static TextStyle get success => TextStyle(
    fontSize: 14,
    fontWeight: FontWeight.normal,
    color: Colors.green,
  );

  // 警告样式
  static TextStyle get warning => TextStyle(
    fontSize: 14,
    fontWeight: FontWeight.normal,
    color: Colors.orange,
  );

  // 小字样式
  static TextStyle get caption => TextStyle(
    fontSize: 12,
    fontWeight: FontWeight.normal,
    color: Colors.grey[600],
    fontStyle: FontStyle.italic,
  );

  // 按钮文本样式
  static TextStyle get button => TextStyle(
    fontSize: 14,
    fontWeight: FontWeight.w500,
    color: Colors.white,
    letterSpacing: 0.5,
  );

  // 输入框标签样式
  static TextStyle get inputLabel => TextStyle(
    fontSize: 14,
    fontWeight: FontWeight.w500,
    color: Colors.black87,
    letterSpacing: 0.1,
  );

  // 输入框提示样式
  static TextStyle get inputHint => TextStyle(
    fontSize: 14,
    fontWeight: FontWeight.normal,
    color: Colors.grey[500],
    letterSpacing: 0.25,
  );
}
```

### 使用自定义样式

```dart
class TextStylingExample extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('文本样式示例', style: AppTextStyles.h3),
      ),
      body: SingleChildScrollView(
        padding: EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            // 使用自定义样式
            Text('主标题', style: AppTextStyles.h1),
            SizedBox(height: 16),
            
            Text('副标题', style: AppTextStyles.h2),
            SizedBox(height: 16),
            
            Text('正文内容', style: AppTextStyles.body1),
            SizedBox(height: 16),
            
            Text('强调文本', style: AppTextStyles.emphasis),
            SizedBox(height: 16),
            
            Text('链接文本', style: AppTextStyles.link),
            SizedBox(height: 16),
            
            Text('错误信息', style: AppTextStyles.error),
            SizedBox(height: 16),
            
            Text('成功信息', style: AppTextStyles.success),
            SizedBox(height: 16),
            
            Text('警告信息', style: AppTextStyles.warning),
            SizedBox(height: 16),
            
            Text('小字说明', style: AppTextStyles.caption),
            SizedBox(height: 32),
            
            // 使用主题样式
            _buildThemeTexts(context),
          ],
        ),
      ),
    );
  }

  Widget _buildThemeTexts(BuildContext context) {
    final textTheme = Theme.of(context).textTheme;
    
    return Column(
      crossAxisAlignment: CrossAxisAlignment.start,
      children: [
        Text('主题样式示例', style: AppTextStyles.h3),
        SizedBox(height: 16),
        
        Text('Display Large', style: textTheme.displayLarge),
        SizedBox(height: 8),
        
        Text('Display Medium', style: textTheme.displayMedium),
        SizedBox(height: 8),
        
        Text('Display Small', style: textTheme.displaySmall),
        SizedBox(height: 16),
        
        Text('Headline Large', style: textTheme.headlineLarge),
        SizedBox(height: 8),
        
        Text('Headline Medium', style: textTheme.headlineMedium),
        SizedBox(height: 8),
        
        Text('Headline Small', style: textTheme.headlineSmall),
        SizedBox(height: 16),
        
        Text('Title Large', style: textTheme.titleLarge),
        SizedBox(height: 8),
        
        Text('Title Medium', style: textTheme.titleMedium),
        SizedBox(height: 8),
        
        Text('Title Small', style: textTheme.titleSmall),
        SizedBox(height: 16),
        
        Text('Body Large', style: textTheme.bodyLarge),
        SizedBox(height: 8),
        
        Text('Body Medium', style: textTheme.bodyMedium),
        SizedBox(height: 8),
        
        Text('Body Small', style: textTheme.bodySmall),
        SizedBox(height: 16),
        
        Text('Label Large', style: textTheme.labelLarge),
        SizedBox(height: 8),
        
        Text('Label Medium', style: textTheme.labelMedium),
        SizedBox(height: 8),
        
        Text('Label Small', style: textTheme.labelSmall),
      ],
    );
  }
}
```

## 🌓 主题切换

### 主题管理器

```dart
class ThemeManager extends ChangeNotifier {
  static final ThemeManager _instance = ThemeManager._internal();
  factory ThemeManager() => _instance;
  ThemeManager._internal();

  bool _isDarkMode = false;

  bool get isDarkMode => _isDarkMode;

  void toggleTheme() {
    _isDarkMode = !_isDarkMode;
    notifyListeners();
  }

  void setTheme(bool isDark) {
    _isDarkMode = isDark;
    notifyListeners();
  }

  ThemeData get lightTheme => ThemeData(
    brightness: Brightness.light,
    textTheme: CustomTextTheme.lightTheme,
    primarySwatch: Colors.blue,
  );

  ThemeData get darkTheme => ThemeData(
    brightness: Brightness.dark,
    textTheme: CustomTextTheme.darkTheme,
    primarySwatch: Colors.blue,
  );
}
```

### 主题切换示例

```dart
class ThemeSwitcherApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return ChangeNotifierProvider(
      create: (context) => ThemeManager(),
      child: Consumer<ThemeManager>(
        builder: (context, themeManager, child) {
          return MaterialApp(
            title: '主题切换示例',
            theme: themeManager.lightTheme,
            darkTheme: themeManager.darkTheme,
            themeMode: themeManager.isDarkMode 
                ? ThemeMode.dark 
                : ThemeMode.light,
            home: ThemeSwitcherExample(),
          );
        },
      ),
    );
  }
}

class ThemeSwitcherExample extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('主题切换示例'),
        actions: [
          IconButton(
            icon: Icon(Icons.brightness_6),
            onPressed: () {
              context.read<ThemeManager>().toggleTheme();
            },
          ),
        ],
      ),
      body: SingleChildScrollView(
        padding: EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Text('主标题', style: AppTextStyles.h1),
            SizedBox(height: 16),
            
            Text('正文内容', style: AppTextStyles.body1),
            SizedBox(height: 16),
            
            Text('强调文本', style: AppTextStyles.emphasis),
            SizedBox(height: 16),
            
            ElevatedButton(
              onPressed: () {
                context.read<ThemeManager>().toggleTheme();
              },
              child: Text('切换主题'),
            ),
          ],
        ),
      ),
    );
  }
}
```

## 🎯 最佳实践

### 1. 样式命名规范

- **语义化命名**：使用有意义的名称，如 `h1`、`body1`、`emphasis`
- **一致性**：保持命名风格的一致性
- **层次化**：使用数字或字母表示层次关系

### 2. 主题管理

- **集中管理**：将所有样式定义集中在一个地方
- **可扩展性**：设计易于扩展的样式系统
- **复用性**：创建可复用的样式组件

### 3. 性能优化

- **避免重复创建**：使用静态方法或常量
- **合理使用主题**：避免过度定制主题
- **缓存样式**：对于复杂样式进行缓存

### 4. 响应式设计

```dart
class ResponsiveTextStyle {
  static TextStyle getResponsiveTitle(BuildContext context) {
    double screenWidth = MediaQuery.of(context).size.width;
    
    if (screenWidth > 1200) {
      return AppTextStyles.h1;
    } else if (screenWidth > 800) {
      return AppTextStyles.h2;
    } else {
      return AppTextStyles.h3;
    }
  }

  static TextStyle getResponsiveBody(BuildContext context) {
    double screenWidth = MediaQuery.of(context).size.width;
    
    if (screenWidth > 800) {
      return AppTextStyles.body1;
    } else {
      return AppTextStyles.body2;
    }
  }
}
```

### 5. 国际化支持

```dart
class LocalizedTextStyle {
  static TextStyle getLocalizedTitle(BuildContext context) {
    Locale locale = Localizations.localeOf(context);
    
    if (locale.languageCode == 'zh') {
      return AppTextStyles.h1.copyWith(
        fontFamily: 'PingFang SC',
      );
    } else {
      return AppTextStyles.h1.copyWith(
        fontFamily: 'Roboto',
      );
    }
  }
}
```

## 📚 相关链接

- [Text 基础文本](text-basic.md) - 学习基础文本显示
- [RichText 富文本](text-richtext.md) - 学习富文本显示
- [文本输入控件](text-input.md) - 学习文本输入
- [Flutter 官方文档](https://api.flutter.dev/flutter/material/TextStyle-class.html)

---

**总结**：Flutter 的文本样式系统提供了强大的定制能力。通过合理使用 TextStyle、TextTheme 和主题管理，可以创建出统一、美观且易于维护的文本样式系统。 