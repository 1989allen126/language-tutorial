# 📝 Flutter 文本输入：让用户与你的应用对话

> 想象一下，如果用户无法在你的应用中输入任何内容，那会是多么糟糕的体验！文本输入是用户与应用交互的重要桥梁。今天我们就来聊聊 Flutter 中的文本输入组件，让你的应用能够真正"听懂"用户的心声。

![Flutter Text Input](https://img.shields.io/badge/Flutter-文本输入-blue?style=for-the-badge&logo=flutter)

## 🎯 为什么文本输入如此重要？

在我开发的一个社交应用中，用户反馈最多的问题就是"输入框太难用了"。有的用户说"输入密码时看不到自己输入了什么"，有的用户抱怨"邮箱格式错误提示不够清楚"，还有用户反映"搜索框太小了，手指点不准"。

这些看似小问题，却直接影响着用户体验。一个好的文本输入组件应该：

- **直观易用**：用户一眼就知道要输入什么
- **智能提示**：在用户犯错前给出友好提醒
- **响应迅速**：输入时立即给出反馈
- **安全可靠**：保护用户的隐私信息

## 🚀 从简单开始：TextField 基础用法

### 你的第一个输入框

让我们从一个简单的例子开始：

```dart
TextField(
  decoration: InputDecoration(
    labelText: '请输入你的名字',
    hintText: '例如：张三',
    border: OutlineInputBorder(),
  ),
)
```

这个简单的输入框包含了：

- **标签文本**：告诉用户要输入什么
- **提示文本**：给出输入示例
- **边框样式**：让输入框更明显

### 实用的输入框样式

```dart
// 邮箱输入框
TextField(
  keyboardType: TextInputType.emailAddress,
  decoration: InputDecoration(
    labelText: '邮箱地址',
    hintText: 'example@email.com',
    prefixIcon: Icon(Icons.email, color: Colors.blue),
    border: OutlineInputBorder(
      borderRadius: BorderRadius.circular(8),
    ),
    focusedBorder: OutlineInputBorder(
      borderRadius: BorderRadius.circular(8),
      borderSide: BorderSide(color: Colors.blue, width: 2),
    ),
  ),
)

// 密码输入框
TextField(
  obscureText: true, // 隐藏密码
  decoration: InputDecoration(
    labelText: '密码',
    hintText: '请输入密码',
    prefixIcon: Icon(Icons.lock, color: Colors.green),
    suffixIcon: IconButton(
      icon: Icon(Icons.visibility),
      onPressed: () {
        // 切换密码显示/隐藏
      },
    ),
    border: OutlineInputBorder(
      borderRadius: BorderRadius.circular(8),
    ),
  ),
)

// 搜索框
TextField(
  decoration: InputDecoration(
    labelText: '搜索',
    hintText: '输入关键词搜索...',
    prefixIcon: Icon(Icons.search, color: Colors.grey),
    suffixIcon: IconButton(
      icon: Icon(Icons.clear),
      onPressed: () {
        // 清空输入内容
      },
    ),
    filled: true,
    fillColor: Colors.grey[100],
    border: OutlineInputBorder(
      borderRadius: BorderRadius.circular(25),
      borderSide: BorderSide.none,
    ),
  ),
)
```

## 📋 表单输入：TextFormField 的强大功能

### 为什么选择 TextFormField？

TextField 很好，但在表单场景下，TextFormField 更加强大。它提供了：

- **内置验证**：自动检查输入格式
- **错误提示**：友好的错误信息显示
- **表单集成**：与 Form 组件完美配合
- **状态管理**：自动管理输入状态

### 完整的表单示例

```dart
class UserRegistrationForm extends StatefulWidget {
  @override
  _UserRegistrationFormState createState() => _UserRegistrationFormState();
}

class _UserRegistrationFormState extends State<UserRegistrationForm> {
  final _formKey = GlobalKey<FormState>();
  final _nameController = TextEditingController();
  final _emailController = TextEditingController();
  final _passwordController = TextEditingController();
  bool _isPasswordVisible = false;

  @override
  void dispose() {
    _nameController.dispose();
    _emailController.dispose();
    _passwordController.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('用户注册'),
      ),
      body: Padding(
        padding: EdgeInsets.all(16),
        child: Form(
          key: _formKey,
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.stretch,
            children: [
              // 用户名输入
              TextFormField(
                controller: _nameController,
                decoration: InputDecoration(
                  labelText: '用户名',
                  hintText: '请输入用户名',
                  prefixIcon: Icon(Icons.person),
                  border: OutlineInputBorder(
                    borderRadius: BorderRadius.circular(8),
                  ),
                ),
                validator: (value) {
                  if (value == null || value.isEmpty) {
                    return '请输入用户名';
                  }
                  if (value.length < 3) {
                    return '用户名至少3个字符';
                  }
                  return null;
                },
              ),
              SizedBox(height: 16),

              // 邮箱输入
              TextFormField(
                controller: _emailController,
                keyboardType: TextInputType.emailAddress,
                decoration: InputDecoration(
                  labelText: '邮箱地址',
                  hintText: 'example@email.com',
                  prefixIcon: Icon(Icons.email),
                  border: OutlineInputBorder(
                    borderRadius: BorderRadius.circular(8),
                  ),
                ),
                validator: (value) {
                  if (value == null || value.isEmpty) {
                    return '请输入邮箱地址';
                  }
                  if (!RegExp(r'^[\w-\.]+@([\w-]+\.)+[\w-]{2,4}$').hasMatch(value)) {
                    return '请输入有效的邮箱地址';
                  }
                  return null;
                },
              ),
              SizedBox(height: 16),

              // 密码输入
              TextFormField(
                controller: _passwordController,
                obscureText: !_isPasswordVisible,
                decoration: InputDecoration(
                  labelText: '密码',
                  hintText: '请输入密码',
                  prefixIcon: Icon(Icons.lock),
                  suffixIcon: IconButton(
                    icon: Icon(
                      _isPasswordVisible ? Icons.visibility : Icons.visibility_off,
                    ),
                    onPressed: () {
                      setState(() {
                        _isPasswordVisible = !_isPasswordVisible;
                      });
                    },
                  ),
                  border: OutlineInputBorder(
                    borderRadius: BorderRadius.circular(8),
                  ),
                ),
                validator: (value) {
                  if (value == null || value.isEmpty) {
                    return '请输入密码';
                  }
                  if (value.length < 6) {
                    return '密码至少6个字符';
                  }
                  if (!RegExp(r'^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)').hasMatch(value)) {
                    return '密码必须包含大小写字母和数字';
                  }
                  return null;
                },
              ),
              SizedBox(height: 24),

              // 提交按钮
              ElevatedButton(
                onPressed: _submitForm,
                child: Text('注册'),
                style: ElevatedButton.styleFrom(
                  padding: EdgeInsets.symmetric(vertical: 16),
                  shape: RoundedRectangleBorder(
                    borderRadius: BorderRadius.circular(8),
                  ),
                ),
              ),
            ],
          ),
        ),
      ),
    );
  }

  void _submitForm() {
    if (_formKey.currentState!.validate()) {
      // 表单验证通过，处理注册逻辑
      print('用户名: ${_nameController.text}');
      print('邮箱: ${_emailController.text}');
      print('密码: ${_passwordController.text}');

      // 显示成功提示
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(
          content: Text('注册成功！'),
          backgroundColor: Colors.green,
        ),
      );
    }
  }
}
```

## 🎯 实战应用：创建智能输入组件

### 1. 智能搜索框

```dart
class SmartSearchBox extends StatefulWidget {
  final Function(String) onSearch;
  final String hintText;

  const SmartSearchBox({
    Key? key,
    required this.onSearch,
    this.hintText = '搜索...',
  }) : super(key: key);

  @override
  _SmartSearchBoxState createState() => _SmartSearchBoxState();
}

class _SmartSearchBoxState extends State<SmartSearchBox> {
  final _controller = TextEditingController();
  Timer? _debounceTimer;
  bool _isSearching = false;

  @override
  void dispose() {
    _controller.dispose();
    _debounceTimer?.cancel();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return TextField(
      controller: _controller,
      decoration: InputDecoration(
        labelText: '搜索',
        hintText: widget.hintText,
        prefixIcon: Icon(Icons.search),
        suffixIcon: _isSearching
            ? Padding(
                padding: EdgeInsets.all(8),
                child: CircularProgressIndicator(strokeWidth: 2),
              )
            : _controller.text.isNotEmpty
                ? IconButton(
                    icon: Icon(Icons.clear),
                    onPressed: () {
                      _controller.clear();
                      widget.onSearch('');
                    },
                  )
                : null,
        border: OutlineInputBorder(
          borderRadius: BorderRadius.circular(25),
        ),
        filled: true,
        fillColor: Colors.grey[100],
      ),
      onChanged: (value) {
        // 防抖处理，避免频繁搜索
        _debounceTimer?.cancel();
        _debounceTimer = Timer(Duration(milliseconds: 500), () {
          setState(() {
            _isSearching = true;
          });

          widget.onSearch(value);

          // 模拟搜索完成
          Future.delayed(Duration(milliseconds: 300), () {
            if (mounted) {
              setState(() {
                _isSearching = false;
              });
            }
          });
        });
      },
    );
  }
}
```

### 2. 金额输入框

```dart
class CurrencyInputField extends StatefulWidget {
  final Function(double) onChanged;
  final String label;
  final double? initialValue;

  const CurrencyInputField({
    Key? key,
    required this.onChanged,
    required this.label,
    this.initialValue,
  }) : super(key: key);

  @override
  _CurrencyInputFieldState createState() => _CurrencyInputFieldState();
}

class _CurrencyInputFieldState extends State<CurrencyInputField> {
  final _controller = TextEditingController();

  @override
  void initState() {
    super.initState();
    if (widget.initialValue != null) {
      _controller.text = _formatCurrency(widget.initialValue!);
    }
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  String _formatCurrency(double value) {
    return '¥${value.toStringAsFixed(2)}';
  }

  double? _parseCurrency(String text) {
    // 移除货币符号和空格
    String cleanText = text.replaceAll(RegExp(r'[¥\s]'), '');
    return double.tryParse(cleanText);
  }

  @override
  Widget build(BuildContext context) {
    return TextFormField(
      controller: _controller,
      keyboardType: TextInputType.numberWithOptions(decimal: true),
      decoration: InputDecoration(
        labelText: widget.label,
        hintText: '0.00',
        prefixIcon: Icon(Icons.attach_money, color: Colors.green),
        border: OutlineInputBorder(
          borderRadius: BorderRadius.circular(8),
        ),
      ),
      validator: (value) {
        if (value == null || value.isEmpty) {
          return '请输入金额';
        }
        double? amount = _parseCurrency(value);
        if (amount == null) {
          return '请输入有效的金额';
        }
        if (amount < 0) {
          return '金额不能为负数';
        }
        return null;
      },
      onChanged: (value) {
        double? amount = _parseCurrency(value);
        if (amount != null) {
          widget.onChanged(amount);
        }
      },
      onTap: () {
        // 选中所有文本，方便用户重新输入
        _controller.selection = TextSelection(
          baseOffset: 0,
          extentOffset: _controller.text.length,
        );
      },
    );
  }
}
```

### 3. 手机号输入框

```dart
class PhoneNumberField extends StatefulWidget {
  final Function(String) onChanged;
  final String label;

  const PhoneNumberField({
    Key? key,
    required this.onChanged,
    required this.label,
  }) : super(key: key);

  @override
  _PhoneNumberFieldState createState() => _PhoneNumberFieldState();
}

class _PhoneNumberFieldState extends State<PhoneNumberField> {
  final _controller = TextEditingController();

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  String _formatPhoneNumber(String text) {
    // 移除所有非数字字符
    String digits = text.replaceAll(RegExp(r'[^\d]'), '');

    // 格式化手机号：138 1234 5678
    if (digits.length <= 3) {
      return digits;
    } else if (digits.length <= 7) {
      return '${digits.substring(0, 3)} ${digits.substring(3)}';
    } else {
      return '${digits.substring(0, 3)} ${digits.substring(3, 7)} ${digits.substring(7, 11)}';
    }
  }

  @override
  Widget build(BuildContext context) {
    return TextFormField(
      controller: _controller,
      keyboardType: TextInputType.phone,
      decoration: InputDecoration(
        labelText: widget.label,
        hintText: '138 1234 5678',
        prefixIcon: Icon(Icons.phone, color: Colors.blue),
        border: OutlineInputBorder(
          borderRadius: BorderRadius.circular(8),
        ),
      ),
      validator: (value) {
        if (value == null || value.isEmpty) {
          return '请输入手机号';
        }
        String digits = value.replaceAll(RegExp(r'[^\d]'), '');
        if (digits.length != 11) {
          return '请输入11位手机号';
        }
        if (!RegExp(r'^1[3-9]\d{9}$').hasMatch(digits)) {
          return '请输入有效的手机号';
        }
        return null;
      },
      onChanged: (value) {
        String formatted = _formatPhoneNumber(value);
        if (formatted != value) {
          _controller.value = TextEditingValue(
            text: formatted,
            selection: TextSelection.collapsed(offset: formatted.length),
          );
        }
        widget.onChanged(value.replaceAll(RegExp(r'[^\d]'), ''));
      },
    );
  }
}
```

## 💡 实用技巧和最佳实践

### 1. 输入验证技巧

```dart
class InputValidators {
  // 邮箱验证
  static String? validateEmail(String? value) {
    if (value == null || value.isEmpty) {
      return '请输入邮箱地址';
    }
    if (!RegExp(r'^[\w-\.]+@([\w-]+\.)+[\w-]{2,4}$').hasMatch(value)) {
      return '请输入有效的邮箱地址';
    }
    return null;
  }

  // 手机号验证
  static String? validatePhone(String? value) {
    if (value == null || value.isEmpty) {
      return '请输入手机号';
    }
    String digits = value.replaceAll(RegExp(r'[^\d]'), '');
    if (digits.length != 11) {
      return '请输入11位手机号';
    }
    if (!RegExp(r'^1[3-9]\d{9}$').hasMatch(digits)) {
      return '请输入有效的手机号';
    }
    return null;
  }

  // 密码验证
  static String? validatePassword(String? value) {
    if (value == null || value.isEmpty) {
      return '请输入密码';
    }
    if (value.length < 6) {
      return '密码至少6个字符';
    }
    if (!RegExp(r'^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)').hasMatch(value)) {
      return '密码必须包含大小写字母和数字';
    }
    return null;
  }

  // 用户名验证
  static String? validateUsername(String? value) {
    if (value == null || value.isEmpty) {
      return '请输入用户名';
    }
    if (value.length < 3) {
      return '用户名至少3个字符';
    }
    if (value.length > 20) {
      return '用户名不能超过20个字符';
    }
    if (!RegExp(r'^[a-zA-Z0-9_]+$').hasMatch(value)) {
      return '用户名只能包含字母、数字和下划线';
    }
    return null;
  }
}
```

### 2. 输入框状态管理

```dart
class InputStateManager extends ChangeNotifier {
  final Map<String, bool> _focusStates = {};
  final Map<String, bool> _errorStates = {};
  final Map<String, String> _errorMessages = {};

  bool isFocused(String fieldName) => _focusStates[fieldName] ?? false;
  bool hasError(String fieldName) => _errorStates[fieldName] ?? false;
  String? getErrorMessage(String fieldName) => _errorMessages[fieldName];

  void setFocus(String fieldName, bool focused) {
    _focusStates[fieldName] = focused;
    notifyListeners();
  }

  void setError(String fieldName, bool hasError, [String? message]) {
    _errorStates[fieldName] = hasError;
    if (message != null) {
      _errorMessages[fieldName] = message;
    } else {
      _errorMessages.remove(fieldName);
    }
    notifyListeners();
  }

  void clearErrors() {
    _errorStates.clear();
    _errorMessages.clear();
    notifyListeners();
  }
}
```

### 3. 性能优化技巧

```dart
// 使用 const 构造函数
class OptimizedTextField extends StatelessWidget {
  const OptimizedTextField({
    Key? key,
    required this.label,
    required this.onChanged,
  }) : super(key: key);

  final String label;
  final Function(String) onChanged;

  @override
  Widget build(BuildContext context) {
    return TextField(
      decoration: InputDecoration(
        labelText: label,
        border: const OutlineInputBorder(),
      ),
      onChanged: onChanged,
    );
  }
}

// 使用 TextEditingController 缓存
class CachedTextController {
  static final Map<String, TextEditingController> _controllers = {};

  static TextEditingController getController(String key) {
    if (!_controllers.containsKey(key)) {
      _controllers[key] = TextEditingController();
    }
    return _controllers[key]!;
  }

  static void disposeController(String key) {
    _controllers[key]?.dispose();
    _controllers.remove(key);
  }

  static void clearAll() {
    for (var controller in _controllers.values) {
      controller.dispose();
    }
    _controllers.clear();
  }
}
```

## 🎨 用户体验优化

### 1. 键盘类型优化

```dart
// 根据输入内容选择合适的键盘类型
TextFormField(
  keyboardType: TextInputType.emailAddress, // 邮箱键盘
  // 或者
  keyboardType: TextInputType.phone, // 电话键盘
  // 或者
  keyboardType: TextInputType.number, // 数字键盘
  // 或者
  keyboardType: TextInputType.multiline, // 多行文本键盘
)
```

### 2. 自动完成功能

```dart
TextFormField(
  autofillHints: [AutofillHints.email], // 自动填充邮箱
  // 或者
  autofillHints: [AutofillHints.telephoneNumber], // 自动填充电话
  // 或者
  autofillHints: [AutofillHints.password], // 自动填充密码
)
```

### 3. 输入限制

```dart
TextFormField(
  maxLength: 50, // 最大长度限制
  maxLines: 3, // 最大行数限制
  inputFormatters: [
    FilteringTextInputFormatter.allow(RegExp(r'[0-9]')), // 只允许数字
    // 或者
    FilteringTextInputFormatter.deny(RegExp(r'[0-9]')), // 不允许数字
  ],
)
```

## 📚 总结

文本输入是用户与应用交互的重要方式。通过合理使用 TextField 和 TextFormField，我们可以：

1. **提供良好的用户体验**：直观的界面和智能的提示
2. **确保数据质量**：通过验证确保输入数据的正确性
3. **提升应用安全性**：保护用户的隐私信息
4. **优化交互流程**：减少用户的输入错误和困惑

### 关键要点

- **TextField** 适合简单的文本输入场景
- **TextFormField** 适合需要验证的表单场景
- **输入验证** 是确保数据质量的重要手段
- **用户体验** 应该始终放在第一位

### 下一步学习

掌握了文本输入的基础后，你可以继续学习：

- [表单组件](form-widgets.md) - 学习完整的表单处理
- [验证系统](validation-system.md) - 深入学习输入验证
- [用户交互](interactive-widgets.md) - 学习更多交互组件

记住，好的文本输入设计不仅仅是功能完整，更重要的是让用户感到舒适和便捷。在实践中不断优化，你一定能创建出用户喜爱的输入体验！

---

<div align="center">

**🌟 如果这篇文章对你有帮助，请给个 Star 支持一下！ 🌟**

[![GitHub stars](https://img.shields.io/github/stars/1989allen126/language-tutorial?style=social)](https://github.com/1989allen126/language-tutorial)
[![GitHub forks](https://img.shields.io/github/forks/1989allen126/language-tutorial?style=social)](https://github.com/1989allen126/language-tutorial)

</div>
