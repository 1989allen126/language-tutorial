# 📝 Flutter 表单组件深度解析：从基础到高级

> 通过丰富的图表、对比分析和实际案例，全面掌握 Flutter 表单组件的使用技巧

![Flutter Form Widgets](https://img.shields.io/badge/Flutter-Form%20Widgets-blue?style=for-the-badge&logo=flutter)
![Version](https://img.shields.io/badge/Version-3.0.0-green?style=for-the-badge)
![License](https://img.shields.io/badge/License-MIT-yellow?style=for-the-badge)

## 📊 文章概览

| 章节                          | 内容         | 难度等级 |
| ----------------------------- | ------------ | -------- |
| [表单基础](#表单基础)         | 基础表单实现 | ⭐⭐     |
| [输入组件](#输入组件)         | 各种输入控件 | ⭐⭐⭐   |
| [选择组件](#选择组件)         | 选择器组件   | ⭐⭐⭐   |
| [表单验证](#表单验证)         | 表单验证机制 | ⭐⭐⭐⭐ |
| [表单状态管理](#表单状态管理) | 状态管理策略 | ⭐⭐⭐⭐ |

## 🎯 学习目标

- ✅ 掌握表单组件的核心概念和使用方法
- ✅ 学会表单验证和错误处理
- ✅ 理解表单状态管理和性能优化
- ✅ 能够实现复杂的表单应用
- ✅ 掌握最佳实践和设计模式

## 📋 目录导航

<details>
<summary>🎯 快速导航</summary>

- [表单基础](#表单基础) - 基础表单实现
- [输入组件](#输入组件) - 各种输入控件
- [选择组件](#选择组件) - 选择器组件
- [表单验证](#表单验证) - 表单验证机制
- [表单状态管理](#表单状态管理) - 状态管理策略
- [自定义表单组件](#自定义表单组件) - 自定义组件开发
- [表单性能优化](#表单性能优化) - 性能优化技巧

</details>

---

## 📋 概述

本文档详细介绍 Flutter 中各种表单组件的使用方法、最佳实践和高级技巧。

## 🎯 表单基础

### Form Widget

```dart
// lib/widgets/basic_form.dart
import 'package:flutter/material.dart';

class BasicFormExample extends StatefulWidget {
  const BasicFormExample({super.key});

  @override
  State<BasicFormExample> createState() => _BasicFormExampleState();
}

class _BasicFormExampleState extends State<BasicFormExample> {
  final _formKey = GlobalKey<FormState>();
  final _nameController = TextEditingController();
  final _emailController = TextEditingController();
  final _passwordController = TextEditingController();

  bool _isLoading = false;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('基础表单'),
        backgroundColor: Theme.of(context).colorScheme.inversePrimary,
      ),
      body: Padding(
        padding: const EdgeInsets.all(16.0),
        child: Form(
          key: _formKey,
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.stretch,
            children: [
              // 姓名输入
              TextFormField(
                controller: _nameController,
                decoration: const InputDecoration(
                  labelText: '姓名',
                  hintText: '请输入您的姓名',
                  prefixIcon: Icon(Icons.person),
                  border: OutlineInputBorder(),
                ),
                validator: (value) {
                  if (value == null || value.isEmpty) {
                    return '请输入姓名';
                  }
                  if (value.length < 2) {
                    return '姓名至少需要2个字符';
                  }
                  return null;
                },
              ),

              const SizedBox(height: 16),

              // 邮箱输入
              TextFormField(
                controller: _emailController,
                keyboardType: TextInputType.emailAddress,
                decoration: const InputDecoration(
                  labelText: '邮箱',
                  hintText: '请输入邮箱地址',
                  prefixIcon: Icon(Icons.email),
                  border: OutlineInputBorder(),
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

              const SizedBox(height: 16),

              // 密码输入
              TextFormField(
                controller: _passwordController,
                obscureText: true,
                decoration: const InputDecoration(
                  labelText: '密码',
                  hintText: '请输入密码',
                  prefixIcon: Icon(Icons.lock),
                  border: OutlineInputBorder(),
                ),
                validator: (value) {
                  if (value == null || value.isEmpty) {
                    return '请输入密码';
                  }
                  if (value.length < 6) {
                    return '密码至少需要6个字符';
                  }
                  return null;
                },
              ),

              const SizedBox(height: 24),

              // 提交按钮
              ElevatedButton(
                onPressed: _isLoading ? null : _submitForm,
                child: _isLoading
                    ? const SizedBox(
                        width: 20,
                        height: 20,
                        child: CircularProgressIndicator(strokeWidth: 2),
                      )
                    : const Text('提交'),
              ),
            ],
          ),
        ),
      ),
    );
  }

  Future<void> _submitForm() async {
    if (_formKey.currentState!.validate()) {
      setState(() {
        _isLoading = true;
      });

      try {
        // 模拟网络请求
        await Future.delayed(const Duration(seconds: 2));

        if (mounted) {
          ScaffoldMessenger.of(context).showSnackBar(
            const SnackBar(
              content: Text('表单提交成功！'),
              backgroundColor: Colors.green,
            ),
          );

          // 清空表单
          _formKey.currentState!.reset();
          _nameController.clear();
          _emailController.clear();
          _passwordController.clear();
        }
      } catch (e) {
        if (mounted) {
          ScaffoldMessenger.of(context).showSnackBar(
            SnackBar(
              content: Text('提交失败: $e'),
              backgroundColor: Colors.red,
            ),
          );
        }
      } finally {
        if (mounted) {
          setState(() {
            _isLoading = false;
          });
        }
      }
    }
  }

  @override
  void dispose() {
    _nameController.dispose();
    _emailController.dispose();
    _passwordController.dispose();
    super.dispose();
  }
}
```

## 📝 输入组件

### TextFormField 高级用法

```dart
// lib/widgets/advanced_text_field.dart
class AdvancedTextFieldExample extends StatefulWidget {
  const AdvancedTextFieldExample({super.key});

  @override
  State<AdvancedTextFieldExample> createState() => _AdvancedTextFieldExampleState();
}

class _AdvancedTextFieldExampleState extends State<AdvancedTextFieldExample> {
  final _phoneController = TextEditingController();
  final _priceController = TextEditingController();
  final _descriptionController = TextEditingController();

  bool _obscurePassword = true;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('高级输入组件')),
      body: Padding(
        padding: const EdgeInsets.all(16.0),
        child: Column(
          children: [
            // 手机号输入（带格式化）
            TextFormField(
              controller: _phoneController,
              keyboardType: TextInputType.phone,
              inputFormatters: [
                FilteringTextInputFormatter.digitsOnly,
                LengthLimitingTextInputFormatter(11),
                PhoneNumberFormatter(),
              ],
              decoration: const InputDecoration(
                labelText: '手机号',
                hintText: '138 0013 8000',
                prefixIcon: Icon(Icons.phone),
                border: OutlineInputBorder(),
              ),
              validator: (value) {
                if (value == null || value.isEmpty) {
                  return '请输入手机号';
                }
                final phone = value.replaceAll(' ', '');
                if (!RegExp(r'^1[3-9]\d{9}$').hasMatch(phone)) {
                  return '请输入有效的手机号';
                }
                return null;
              },
            ),

            const SizedBox(height: 16),

            // 价格输入（带货币格式）
            TextFormField(
              controller: _priceController,
              keyboardType: const TextInputType.numberWithOptions(decimal: true),
              inputFormatters: [
                FilteringTextInputFormatter.allow(RegExp(r'^\d+\.?\d{0,2}')),
                CurrencyFormatter(),
              ],
              decoration: const InputDecoration(
                labelText: '价格',
                hintText: '¥0.00',
                prefixIcon: Icon(Icons.attach_money),
                border: OutlineInputBorder(),
              ),
            ),

            const SizedBox(height: 16),

            // 密码输入（带显示/隐藏切换）
            TextFormField(
              obscureText: _obscurePassword,
              decoration: InputDecoration(
                labelText: '密码',
                hintText: '请输入密码',
                prefixIcon: const Icon(Icons.lock),
                suffixIcon: IconButton(
                  icon: Icon(
                    _obscurePassword ? Icons.visibility : Icons.visibility_off,
                  ),
                  onPressed: () {
                    setState(() {
                      _obscurePassword = !_obscurePassword;
                    });
                  },
                ),
                border: const OutlineInputBorder(),
              ),
            ),

            const SizedBox(height: 16),

            // 多行文本输入
            TextFormField(
              controller: _descriptionController,
              maxLines: 4,
              maxLength: 200,
              decoration: const InputDecoration(
                labelText: '描述',
                hintText: '请输入详细描述...',
                alignLabelWithHint: true,
                border: OutlineInputBorder(),
              ),
              buildCounter: (context, {required currentLength, required isFocused, maxLength}) {
                return Text(
                  '$currentLength/${maxLength ?? 0}',
                  style: TextStyle(
                    color: currentLength > (maxLength ?? 0) * 0.8
                        ? Colors.orange
                        : Colors.grey,
                  ),
                );
              },
            ),
          ],
        ),
      ),
    );
  }

  @override
  void dispose() {
    _phoneController.dispose();
    _priceController.dispose();
    _descriptionController.dispose();
    super.dispose();
  }
}

// 手机号格式化器
class PhoneNumberFormatter extends TextInputFormatter {
  @override
  TextEditingValue formatEditUpdate(
    TextEditingValue oldValue,
    TextEditingValue newValue,
  ) {
    final text = newValue.text.replaceAll(' ', '');
    final buffer = StringBuffer();

    for (int i = 0; i < text.length; i++) {
      if (i == 3 || i == 7) {
        buffer.write(' ');
      }
      buffer.write(text[i]);
    }

    final formatted = buffer.toString();
    return TextEditingValue(
      text: formatted,
      selection: TextSelection.collapsed(offset: formatted.length),
    );
  }
}

// 货币格式化器
class CurrencyFormatter extends TextInputFormatter {
  @override
  TextEditingValue formatEditUpdate(
    TextEditingValue oldValue,
    TextEditingValue newValue,
  ) {
    if (newValue.text.isEmpty) {
      return newValue;
    }

    final value = double.tryParse(newValue.text);
    if (value == null) {
      return oldValue;
    }

    final formatted = '¥${value.toStringAsFixed(2)}';
    return TextEditingValue(
      text: formatted,
      selection: TextSelection.collapsed(offset: formatted.length),
    );
  }
}
```

## 🎛️ 选择组件

### Dropdown 和 Checkbox 组件

```dart
// lib/widgets/selection_widgets.dart
class SelectionWidgetsExample extends StatefulWidget {
  const SelectionWidgetsExample({super.key});

  @override
  State<SelectionWidgetsExample> createState() => _SelectionWidgetsExampleState();
}

class _SelectionWidgetsExampleState extends State<SelectionWidgetsExample> {
  String? _selectedCity;
  final List<String> _cities = ['北京', '上海', '广州', '深圳', '杭州'];

  String? _selectedGender;
  final List<String> _genders = ['男', '女', '其他'];

  final Set<String> _selectedHobbies = {};
  final List<String> _hobbies = ['阅读', '运动', '音乐', '旅行', '摄影', '编程'];

  bool _agreeTerms = false;
  bool _receiveNotifications = true;

  double _ageRange = 25;
  RangeValues _priceRange = const RangeValues(100, 500);

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('选择组件')),
      body: Padding(
        padding: const EdgeInsets.all(16.0),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            // 下拉选择
            const Text('城市选择', style: TextStyle(fontSize: 16, fontWeight: FontWeight.bold)),
            const SizedBox(height: 8),
            DropdownButtonFormField<String>(
              value: _selectedCity,
              decoration: const InputDecoration(
                border: OutlineInputBorder(),
                hintText: '请选择城市',
              ),
              items: _cities.map((city) {
                return DropdownMenuItem(
                  value: city,
                  child: Text(city),
                );
              }).toList(),
              onChanged: (value) {
                setState(() {
                  _selectedCity = value;
                });
              },
              validator: (value) {
                if (value == null) {
                  return '请选择城市';
                }
                return null;
              },
            ),

            const SizedBox(height: 24),

            // 单选按钮组
            const Text('性别', style: TextStyle(fontSize: 16, fontWeight: FontWeight.bold)),
            const SizedBox(height: 8),
            Column(
              children: _genders.map((gender) {
                return RadioListTile<String>(
                  title: Text(gender),
                  value: gender,
                  groupValue: _selectedGender,
                  onChanged: (value) {
                    setState(() {
                      _selectedGender = value;
                    });
                  },
                );
              }).toList(),
            ),

            const SizedBox(height: 24),

            // 多选框组
            const Text('兴趣爱好', style: TextStyle(fontSize: 16, fontWeight: FontWeight.bold)),
            const SizedBox(height: 8),
            Wrap(
              spacing: 8,
              children: _hobbies.map((hobby) {
                return FilterChip(
                  label: Text(hobby),
                  selected: _selectedHobbies.contains(hobby),
                  onSelected: (selected) {
                    setState(() {
                      if (selected) {
                        _selectedHobbies.add(hobby);
                      } else {
                        _selectedHobbies.remove(hobby);
                      }
                    });
                  },
                );
              }).toList(),
            ),

            const SizedBox(height: 24),

            // 开关和复选框
            CheckboxListTile(
              title: const Text('同意用户协议'),
              value: _agreeTerms,
              onChanged: (value) {
                setState(() {
                  _agreeTerms = value ?? false;
                });
              },
              controlAffinity: ListTileControlAffinity.leading,
            ),

            SwitchListTile(
              title: const Text('接收通知'),
              subtitle: const Text('允许应用发送推送通知'),
              value: _receiveNotifications,
              onChanged: (value) {
                setState(() {
                  _receiveNotifications = value;
                });
              },
            ),

            const SizedBox(height: 24),

            // 滑块
            const Text('年龄', style: TextStyle(fontSize: 16, fontWeight: FontWeight.bold)),
            Slider(
              value: _ageRange,
              min: 18,
              max: 60,
              divisions: 42,
              label: '${_ageRange.round()}岁',
              onChanged: (value) {
                setState(() {
                  _ageRange = value;
                });
              },
            ),

            const SizedBox(height: 16),

            // 范围滑块
            const Text('价格区间', style: TextStyle(fontSize: 16, fontWeight: FontWeight.bold)),
            RangeSlider(
              values: _priceRange,
              min: 0,
              max: 1000,
              divisions: 20,
              labels: RangeLabels(
                '¥${_priceRange.start.round()}',
                '¥${_priceRange.end.round()}',
              ),
              onChanged: (values) {
                setState(() {
                  _priceRange = values;
                });
              },
            ),

            const Spacer(),

            // 提交按钮
            SizedBox(
              width: double.infinity,
              child: ElevatedButton(
                onPressed: _agreeTerms ? _submitSelection : null,
                child: const Text('提交选择'),
              ),
            ),
          ],
        ),
      ),
    );
  }

  void _submitSelection() {
    final result = {
      '城市': _selectedCity,
      '性别': _selectedGender,
      '兴趣爱好': _selectedHobbies.toList(),
      '年龄': _ageRange.round(),
      '价格区间': '¥${_priceRange.start.round()} - ¥${_priceRange.end.round()}',
      '接收通知': _receiveNotifications,
    };

    showDialog(
      context: context,
      builder: (context) {
        return AlertDialog(
          title: const Text('选择结果'),
          content: Column(
            mainAxisSize: MainAxisSize.min,
            crossAxisAlignment: CrossAxisAlignment.start,
            children: result.entries.map((entry) {
              return Padding(
                padding: const EdgeInsets.symmetric(vertical: 4),
                child: Text('${entry.key}: ${entry.value}'),
              );
            }).toList(),
          ),
          actions: [
            TextButton(
              onPressed: () => Navigator.of(context).pop(),
              child: const Text('确定'),
            ),
          ],
        );
      },
    );
  }
}
```

## ✅ 表单验证

### 自定义验证器

```dart
// lib/utils/form_validators.dart
class FormValidators {
  // 必填验证
  static String? required(String? value, [String? fieldName]) {
    if (value == null || value.trim().isEmpty) {
      return '${fieldName ?? '此字段'}不能为空';
    }
    return null;
  }

  // 邮箱验证
  static String? email(String? value) {
    if (value == null || value.isEmpty) {
      return null; // 允许为空，如需必填请配合 required 使用
    }

    final emailRegex = RegExp(r'^[\w-\.]+@([\w-]+\.)+[\w-]{2,4}$');
    if (!emailRegex.hasMatch(value)) {
      return '请输入有效的邮箱地址';
    }
    return null;
  }

  // 手机号验证
  static String? phone(String? value) {
    if (value == null || value.isEmpty) {
      return null;
    }

    final phone = value.replaceAll(RegExp(r'\s+'), '');
    final phoneRegex = RegExp(r'^1[3-9]\d{9}$');
    if (!phoneRegex.hasMatch(phone)) {
      return '请输入有效的手机号';
    }
    return null;
  }

  // 密码强度验证
  static String? password(String? value) {
    if (value == null || value.isEmpty) {
      return null;
    }

    if (value.length < 8) {
      return '密码至少需要8个字符';
    }

    if (!RegExp(r'^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)').hasMatch(value)) {
      return '密码必须包含大小写字母和数字';
    }

    return null;
  }

  // 长度验证
  static String? length(String? value, int min, [int? max]) {
    if (value == null || value.isEmpty) {
      return null;
    }

    if (value.length < min) {
      return '至少需要$min个字符';
    }

    if (max != null && value.length > max) {
      return '最多只能输入$max个字符';
    }

    return null;
  }

  // 数字范围验证
  static String? numberRange(String? value, double min, double max) {
    if (value == null || value.isEmpty) {
      return null;
    }

    final number = double.tryParse(value);
    if (number == null) {
      return '请输入有效的数字';
    }

    if (number < min || number > max) {
      return '请输入$min到$max之间的数字';
    }

    return null;
    }

  // 确认密码验证
  static String? confirmPassword(String? value, String? originalPassword) {
    if (value == null || value.isEmpty) {
      return null;
    }

    if (value != originalPassword) {
      return '两次输入的密码不一致';
    }

    return null;
  }

  // 组合验证器
  static String? combine(String? value, List<String? Function(String?)> validators) {
    for (final validator in validators) {
      final result = validator(value);
      if (result != null) {
        return result;
      }
    }
    return null;
  }
}

// 使用示例
class ValidatedFormExample extends StatefulWidget {
  const ValidatedFormExample({super.key});

  @override
  State<ValidatedFormExample> createState() => _ValidatedFormExampleState();
}

class _ValidatedFormExampleState extends State<ValidatedFormExample> {
  final _formKey = GlobalKey<FormState>();
  final _emailController = TextEditingController();
  final _passwordController = TextEditingController();
  final _confirmPasswordController = TextEditingController();

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('表单验证示例')),
      body: Padding(
        padding: const EdgeInsets.all(16.0),
        child: Form(
          key: _formKey,
          child: Column(
            children: [
              TextFormField(
                controller: _emailController,
                decoration: const InputDecoration(
                  labelText: '邮箱',
                  border: OutlineInputBorder(),
                ),
                validator: (value) => FormValidators.combine(value, [
                  (v) => FormValidators.required(v, '邮箱'),
                  FormValidators.email,
                ]),
              ),

              const SizedBox(height: 16),

              TextFormField(
                controller: _passwordController,
                obscureText: true,
                decoration: const InputDecoration(
                  labelText: '密码',
                  border: OutlineInputBorder(),
                ),
                validator: (value) => FormValidators.combine(value, [
                  (v) => FormValidators.required(v, '密码'),
                  FormValidators.password,
                ]),
              ),

              const SizedBox(height: 16),

              TextFormField(
                controller: _confirmPasswordController,
                obscureText: true,
                decoration: const InputDecoration(
                  labelText: '确认密码',
                  border: OutlineInputBorder(),
                ),
                validator: (value) => FormValidators.combine(value, [
                  (v) => FormValidators.required(v, '确认密码'),
                  (v) => FormValidators.confirmPassword(v, _passwordController.text),
                ]),
              ),

              const SizedBox(height: 24),

              ElevatedButton(
                onPressed: () {
                  if (_formKey.currentState!.validate()) {
                    ScaffoldMessenger.of(context).showSnackBar(
                      const SnackBar(content: Text('验证通过！')),
                    );
                  }
                },
                child: const Text('提交'),
              ),
            ],
          ),
        ),
      ),
    );
  }

  @override
  void dispose() {
    _emailController.dispose();
    _passwordController.dispose();
    _confirmPasswordController.dispose();
    super.dispose();
  }
}
```

## 🎨 自定义表单组件

### 自定义输入组件

```dart
// lib/widgets/custom_form_field.dart
class CustomFormField extends StatelessWidget {
  const CustomFormField({
    super.key,
    required this.label,
    this.hintText,
    this.controller,
    this.validator,
    this.keyboardType,
    this.obscureText = false,
    this.maxLines = 1,
    this.prefixIcon,
    this.suffixIcon,
    this.enabled = true,
    this.onChanged,
    this.inputFormatters,
  });

  final String label;
  final String? hintText;
  final TextEditingController? controller;
  final String? Function(String?)? validator;
  final TextInputType? keyboardType;
  final bool obscureText;
  final int maxLines;
  final Widget? prefixIcon;
  final Widget? suffixIcon;
  final bool enabled;
  final void Function(String)? onChanged;
  final List<TextInputFormatter>? inputFormatters;

  @override
  Widget build(BuildContext context) {
    return Column(
      crossAxisAlignment: CrossAxisAlignment.start,
      children: [
        Text(
          label,
          style: Theme.of(context).textTheme.titleSmall?.copyWith(
            fontWeight: FontWeight.w600,
          ),
        ),
        const SizedBox(height: 8),
        TextFormField(
          controller: controller,
          validator: validator,
          keyboardType: keyboardType,
          obscureText: obscureText,
          maxLines: maxLines,
          enabled: enabled,
          onChanged: onChanged,
          inputFormatters: inputFormatters,
          decoration: InputDecoration(
            hintText: hintText,
            prefixIcon: prefixIcon,
            suffixIcon: suffixIcon,
            border: OutlineInputBorder(
              borderRadius: BorderRadius.circular(12),
            ),
            enabledBorder: OutlineInputBorder(
              borderRadius: BorderRadius.circular(12),
              borderSide: BorderSide(
                color: Colors.grey.shade300,
              ),
            ),
            focusedBorder: OutlineInputBorder(
              borderRadius: BorderRadius.circular(12),
              borderSide: BorderSide(
                color: Theme.of(context).primaryColor,
                width: 2,
              ),
            ),
            errorBorder: OutlineInputBorder(
              borderRadius: BorderRadius.circular(12),
              borderSide: const BorderSide(
                color: Colors.red,
              ),
            ),
            filled: true,
            fillColor: enabled ? Colors.grey.shade50 : Colors.grey.shade100,
          ),
        ),
      ],
    );
  }
}

// 自定义选择器组件
class CustomSelector<T> extends StatelessWidget {
  const CustomSelector({
    super.key,
    required this.label,
    required this.value,
    required this.items,
    required this.onChanged,
    this.hintText,
    this.validator,
  });

  final String label;
  final T? value;
  final List<DropdownMenuItem<T>> items;
  final void Function(T?) onChanged;
  final String? hintText;
  final String? Function(T?)? validator;

  @override
  Widget build(BuildContext context) {
    return Column(
      crossAxisAlignment: CrossAxisAlignment.start,
      children: [
        Text(
          label,
          style: Theme.of(context).textTheme.titleSmall?.copyWith(
            fontWeight: FontWeight.w600,
          ),
        ),
        const SizedBox(height: 8),
        DropdownButtonFormField<T>(
          value: value,
          items: items,
          onChanged: onChanged,
          validator: validator,
          decoration: InputDecoration(
            hintText: hintText,
            border: OutlineInputBorder(
              borderRadius: BorderRadius.circular(12),
            ),
            enabledBorder: OutlineInputBorder(
              borderRadius: BorderRadius.circular(12),
              borderSide: BorderSide(
                color: Colors.grey.shade300,
              ),
            ),
            focusedBorder: OutlineInputBorder(
              borderRadius: BorderRadius.circular(12),
              borderSide: BorderSide(
                color: Theme.of(context).primaryColor,
                width: 2,
              ),
            ),
            filled: true,
            fillColor: Colors.grey.shade50,
          ),
        ),
      ],
    );
  }
}
```

## 📊 总结

Flutter 表单组件提供了丰富的功能：

### 核心组件

1. **TextFormField**：文本输入的核心组件
2. **DropdownButtonFormField**：下拉选择组件
3. **Checkbox/Radio**：单选和多选组件
4. **Slider**：滑块选择组件

### 最佳实践

1. **表单验证**：使用组合验证器提高代码复用性
2. **状态管理**：合理管理表单状态和用户输入
3. **用户体验**：提供清晰的错误提示和加载状态
4. **自定义组件**：创建符合设计规范的表单组件

### 推荐工具

- **flutter_form_builder**：强大的表单构建器
- **reactive_forms**：响应式表单管理
- **form_validator**：表单验证工具
- **mask_text_input_formatter**：输入格式化工具

通过掌握这些表单组件的使用方法，可以构建出功能完善、用户体验良好的表单界面。
