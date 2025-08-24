# 📝 Flutter 文本输入控件详解

> 掌握 Flutter 中文本输入的核心组件，实现用户交互和数据收集

![Flutter Text Input Widgets](https://img.shields.io/badge/Flutter-Text%20Input-blue?style=for-the-badge&logo=flutter)
![Version](https://img.shields.io/badge/Version-1.0.0-green?style=for-the-badge)
![License](https://img.shields.io/badge/License-MIT-yellow?style=for-the-badge)

## 📋 目录导航

- [组件概述](#组件概述)
- [TextField 基础输入](#textfield-基础输入)
- [TextFormField 表单输入](#textformfield-表单输入)
- [输入验证](#输入验证)
- [高级功能](#高级功能)
- [最佳实践](#最佳实践)

---

## 🎯 组件概述

Flutter 提供了强大的文本输入组件来处理用户输入需求，主要包括 `TextField` 和 `TextFormField` 两个核心组件。

### 核心特性

- **丰富的装饰**：支持标签、提示、图标等装饰元素
- **输入控制**：支持键盘类型、输入格式、验证等功能
- **交互反馈**：提供焦点、错误、成功等状态反馈
- **样式定制**：支持边框、背景、字体等样式定制

### 组件对比

| 组件 | 主要用途 | 验证功能 | 表单集成 | 复杂度 |
|------|----------|----------|----------|--------|
| **TextField** | 基础文本输入 | 手动验证 | 需要手动集成 | ⭐⭐ |
| **TextFormField** | 表单文本输入 | 内置验证 | 自动集成 | ⭐⭐⭐ |

## 📝 TextField 基础输入

### 核心价值

**TextField 组件概述**：
TextField 是 Flutter 中用于文本输入的核心组件，提供了丰富的配置选项和交互功能，能够满足各种输入需求。

**TextField 的优势**：
- **丰富的装饰**：支持标签、提示、图标等装饰元素
- **输入控制**：支持键盘类型、输入格式、验证等功能
- **交互反馈**：提供焦点、错误、成功等状态反馈
- **样式定制**：支持边框、背景、字体等样式定制

**应用场景**：
- **用户注册**：用户名、密码、邮箱等输入
- **搜索功能**：搜索关键词输入
- **表单填写**：各种表单字段输入
- **聊天界面**：消息输入框

### 基础用法

```dart
// 简单输入框
TextField(
  decoration: InputDecoration(
    labelText: '请输入内容',
    hintText: '这是一个简单的输入框',
  ),
)

// 带样式的输入框
TextField(
  decoration: InputDecoration(
    labelText: '邮箱',
    hintText: '请输入邮箱地址',
    prefixIcon: Icon(Icons.email),
    border: OutlineInputBorder(),
  ),
)

// 密码输入框
TextField(
  obscureText: true,
  decoration: InputDecoration(
    labelText: '密码',
    hintText: '请输入密码',
    prefixIcon: Icon(Icons.lock),
    border: OutlineInputBorder(),
  ),
)
```

## 📋 TextFormField 表单输入

### 核心价值

**TextFormField 的核心价值**：
TextFormField 是 Flutter 中用于表单输入的重要组件，提供了内置的验证功能和表单管理能力。

**TextFormField 的优势**：

- **内置验证**：支持实时验证和错误提示
- **表单管理**：与 Form 组件集成，支持表单状态管理
- **样式一致**：与 TextField 保持一致的样式和交互
- **易于使用**：简化了表单验证的实现

**应用场景**：

- **用户注册**：用户名、密码、邮箱等验证
- **数据录入**：各种需要验证的数据输入
- **设置页面**：用户偏好设置
- **搜索表单**：带验证的搜索条件输入

### 基础用法

```dart
Form(
  key: _formKey,
  child: Column(
    children: [
      TextFormField(
        decoration: InputDecoration(
          labelText: '姓名 *',
          hintText: '请输入您的姓名',
        ),
        validator: (value) {
          if (value == null || value.isEmpty) {
            return '请输入姓名';
          }
          return null;
        },
      ),
      
      TextFormField(
        decoration: InputDecoration(
          labelText: '邮箱 *',
          hintText: '请输入邮箱地址',
        ),
        validator: (value) {
          if (value == null || value.isEmpty) {
            return '请输入邮箱';
          }
          if (!RegExp(r'^[\w-\.]+@([\w-]+\.)+[\w-]{2,4}$').hasMatch(value)) {
            return '请输入有效的邮箱地址';
          }
          return null;
        },
      ),
    ],
  ),
)
```

## ✅ 输入验证

### 验证策略

**验证时机**：
- **实时验证**：用户输入时立即验证
- **失焦验证**：用户离开输入框时验证
- **提交验证**：用户提交表单时验证

**验证类型**：
- **必填验证**：检查字段是否为空
- **格式验证**：检查输入格式是否正确
- **长度验证**：检查输入长度是否符合要求
- **自定义验证**：根据业务需求自定义验证规则

### 验证示例

```dart
// 邮箱验证
String? validateEmail(String? value) {
  if (value == null || value.isEmpty) {
    return '请输入邮箱';
  }
  if (!RegExp(r'^[\w-\.]+@([\w-]+\.)+[\w-]{2,4}$').hasMatch(value)) {
    return '请输入有效的邮箱地址';
  }
  return null;
}

// 密码验证
String? validatePassword(String? value) {
  if (value == null || value.isEmpty) {
    return '请输入密码';
  }
  if (value.length < 8) {
    return '密码长度至少 8 位';
  }
  if (!RegExp(r'^(?=.*[A-Za-z])(?=.*\d)').hasMatch(value)) {
    return '密码必须包含字母和数字';
  }
  return null;
}
```

## 🚀 高级功能

### 自定义输入格式

```dart
class CustomInputFormatter extends TextInputFormatter {
  @override
  TextEditingValue formatEditUpdate(
    TextEditingValue oldValue,
    TextEditingValue newValue,
  ) {
    // 自定义格式化逻辑
    String formatted = newValue.text;
    
    // 示例：手机号格式化
    if (formatted.length > 3 && !formatted.contains(' ')) {
      formatted = formatted.replaceAllMapped(
        RegExp(r'(\d{3})(\d{4})(\d{4})'),
        (Match match) => '${match[1]} ${match[2]} ${match[3]}',
      );
    }
    
    return TextEditingValue(
      text: formatted,
      selection: TextSelection.collapsed(offset: formatted.length),
    );
  }
}

// 使用自定义格式化器
TextField(
  inputFormatters: [CustomInputFormatter()],
  decoration: InputDecoration(
    labelText: '手机号',
    hintText: '请输入手机号',
  ),
)
```

## 🎯 最佳实践

### 1. 性能优化

- **使用 TextEditingController**：避免在 build 中创建控制器
- **合理使用 onChanged**：避免频繁的状态更新
- **防抖处理**：对实时验证使用防抖机制

### 2. 用户体验

- **清晰的标签和提示**：帮助用户理解输入要求
- **实时反馈**：提供即时的验证反馈
- **友好的错误信息**：使用用户易懂的错误提示

### 3. 可访问性

- **语义标签**：为输入框提供合适的语义信息
- **键盘导航**：支持键盘导航和操作
- **屏幕阅读器**：确保屏幕阅读器能正确读取内容

### 4. 代码组织

```dart
class InputFieldHelper {
  // 提取常用的验证规则
  static String? validateEmail(String? value) {
    if (value == null || value.isEmpty) {
      return '请输入邮箱';
    }
    if (!RegExp(r'^[\w-\.]+@([\w-]+\.)+[\w-]{2,4}$').hasMatch(value)) {
      return '请输入有效的邮箱地址';
    }
    return null;
  }

  static String? validatePassword(String? value) {
    if (value == null || value.isEmpty) {
      return '请输入密码';
    }
    if (value.length < 8) {
      return '密码长度至少 8 位';
    }
    return null;
  }

  // 提取常用的装饰配置
  static InputDecoration getEmailDecoration() {
    return InputDecoration(
      labelText: '邮箱',
      hintText: '请输入邮箱地址',
      prefixIcon: Icon(Icons.email),
      border: OutlineInputBorder(),
    );
  }
}
```

## 📚 相关链接

- [Text 基础文本](text-basic.md) - 学习基础文本显示
- [RichText 富文本](text-richtext.md) - 学习富文本显示
- [文本样式系统](text-styling.md) - 学习样式管理
- [Flutter 官方文档](https://api.flutter.dev/flutter/material/TextField-class.html)

---

**总结**：TextField 和 TextFormField 是 Flutter 中功能强大的文本输入组件。通过合理使用验证、装饰和交互功能，可以创建出用户友好、功能完善的输入界面。 