# Flutter Widget 组合模式 (Widget Composition Pattern)

## 模式概述

### 定义
Widget 组合模式是 Flutter 框架的核心设计理念，通过将简单的 Widget 组合成复杂的 UI 组件，实现高度可复用和可维护的用户界面。这种模式强调"组合优于继承"的设计原则。

### 适用场景
- 构建复杂的用户界面
- 创建可复用的 UI 组件
- 实现响应式布局
- 构建主题化的界面元素
- 创建自定义的复合控件

### 优点
- 高度的可复用性
- 清晰的组件层次结构
- 易于测试和维护
- 支持热重载
- 声明式 UI 编程
- 性能优化（Widget 树优化）

### 缺点
- 可能产生深层嵌套
- 学习曲线相对陡峭
- 过度组合可能影响性能

## 基础实现

### 简单组合示例

```dart
// 基础组合：按钮 + 图标
class IconButton extends StatelessWidget {
  final IconData icon;
  final VoidCallback? onPressed;
  final Color? color;
  final double size;
  
  const IconButton({
    Key? key,
    required this.icon,
    this.onPressed,
    this.color,
    this.size = 24.0,
  }) : super(key: key);
  
  @override
  Widget build(BuildContext context) {
    return GestureDetector(
      onTap: onPressed,
      child: Container(
        padding: EdgeInsets.all(8.0),
        child: Icon(
          icon,
          color: color ?? Theme.of(context).iconTheme.color,
          size: size,
        ),
      ),
    );
  }
}

// 复合组合：卡片 + 内容
class CustomCard extends StatelessWidget {
  final Widget child;
  final EdgeInsetsGeometry? padding;
  final Color? backgroundColor;
  final double? elevation;
  final BorderRadius? borderRadius;
  
  const CustomCard({
    Key? key,
    required this.child,
    this.padding,
    this.backgroundColor,
    this.elevation,
    this.borderRadius,
  }) : super(key: key);
  
  @override
  Widget build(BuildContext context) {
    return Card(
      elevation: elevation ?? 2.0,
      color: backgroundColor,
      shape: RoundedRectangleBorder(
        borderRadius: borderRadius ?? BorderRadius.circular(8.0),
      ),
      child: Padding(
        padding: padding ?? EdgeInsets.all(16.0),
        child: child,
      ),
    );
  }
}
```

### 高级组合示例

```dart
// 复杂组合：用户信息卡片
class UserInfoCard extends StatelessWidget {
  final User user;
  final VoidCallback? onTap;
  final bool showActions;
  
  const UserInfoCard({
    Key? key,
    required this.user,
    this.onTap,
    this.showActions = true,
  }) : super(key: key);
  
  @override
  Widget build(BuildContext context) {
    return CustomCard(
      child: InkWell(
        onTap: onTap,
        borderRadius: BorderRadius.circular(8.0),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            _buildHeader(),
            SizedBox(height: 12),
            _buildContent(),
            if (showActions) ..[
              SizedBox(height: 16),
              _buildActions(context),
            ],
          ],
        ),
      ),
    );
  }
  
  Widget _buildHeader() {
    return Row(
      children: [
        CircleAvatar(
          radius: 24,
          backgroundImage: user.avatarUrl != null
              ? NetworkImage(user.avatarUrl!)
              : null,
          child: user.avatarUrl == null
              ? Icon(Icons.person)
              : null,
        ),
        SizedBox(width: 12),
        Expanded(
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: [
              Text(
                user.name,
                style: TextStyle(
                  fontSize: 16,
                  fontWeight: FontWeight.bold,
                ),
              ),
              if (user.title != null)
                Text(
                  user.title!,
                  style: TextStyle(
                    fontSize: 14,
                    color: Colors.grey[600],
                  ),
                ),
            ],
          ),
        ),
        if (user.isOnline)
          Container(
            width: 8,
            height: 8,
            decoration: BoxDecoration(
              color: Colors.green,
              shape: BoxShape.circle,
            ),
          ),
      ],
    );
  }
  
  Widget _buildContent() {
    return Column(
      crossAxisAlignment: CrossAxisAlignment.start,
      children: [
        if (user.bio != null)
          Text(
            user.bio!,
            style: TextStyle(fontSize: 14),
            maxLines: 3,
            overflow: TextOverflow.ellipsis,
          ),
        SizedBox(height: 8),
        Row(
          children: [
            Icon(Icons.location_on, size: 16, color: Colors.grey[600]),
            SizedBox(width: 4),
            Text(
              user.location ?? '未知位置',
              style: TextStyle(
                fontSize: 12,
                color: Colors.grey[600],
              ),
            ),
            SizedBox(width: 16),
            Icon(Icons.access_time, size: 16, color: Colors.grey[600]),
            SizedBox(width: 4),
            Text(
              _formatJoinDate(user.joinDate),
              style: TextStyle(
                fontSize: 12,
                color: Colors.grey[600],
              ),
            ),
          ],
        ),
      ],
    );
  }
  
  Widget _buildActions(BuildContext context) {
    return Row(
      mainAxisAlignment: MainAxisAlignment.end,
      children: [
        TextButton(
          onPressed: () => _sendMessage(context),
          child: Text('发消息'),
        ),
        SizedBox(width: 8),
        ElevatedButton(
          onPressed: () => _followUser(context),
          child: Text(user.isFollowing ? '已关注' : '关注'),
        ),
      ],
    );
  }
  
  String _formatJoinDate(DateTime date) {
    final now = DateTime.now();
    final difference = now.difference(date);
    
    if (difference.inDays > 365) {
      return '${(difference.inDays / 365).floor()}年前加入';
    } else if (difference.inDays > 30) {
      return '${(difference.inDays / 30).floor()}个月前加入';
    } else {
      return '${difference.inDays}天前加入';
    }
  }
  
  void _sendMessage(BuildContext context) {
    // 发送消息逻辑
  }
  
  void _followUser(BuildContext context) {
    // 关注用户逻辑
  }
}

// 用户数据模型
class User {
  final String id;
  final String name;
  final String? title;
  final String? bio;
  final String? avatarUrl;
  final String? location;
  final DateTime joinDate;
  final bool isOnline;
  final bool isFollowing;
  
  User({
    required this.id,
    required this.name,
    this.title,
    this.bio,
    this.avatarUrl,
    this.location,
    required this.joinDate,
    this.isOnline = false,
    this.isFollowing = false,
  });
}
```

## Flutter 应用实例

### 响应式布局组合

```dart
// 响应式容器组合
class ResponsiveContainer extends StatelessWidget {
  final Widget child;
  final EdgeInsetsGeometry? padding;
  final double? maxWidth;
  
  const ResponsiveContainer({
    Key? key,
    required this.child,
    this.padding,
    this.maxWidth,
  }) : super(key: key);
  
  @override
  Widget build(BuildContext context) {
    return LayoutBuilder(
      builder: (context, constraints) {
        final screenWidth = constraints.maxWidth;
        final isTablet = screenWidth > 768;
        final isDesktop = screenWidth > 1024;
        
        return Center(
          child: Container(
            width: maxWidth ?? _getMaxWidth(screenWidth, isTablet, isDesktop),
            padding: padding ?? _getPadding(isTablet, isDesktop),
            child: child,
          ),
        );
      },
    );
  }
  
  double _getMaxWidth(double screenWidth, bool isTablet, bool isDesktop) {
    if (isDesktop) return 1200;
    if (isTablet) return 800;
    return screenWidth;
  }
  
  EdgeInsetsGeometry _getPadding(bool isTablet, bool isDesktop) {
    if (isDesktop) return EdgeInsets.symmetric(horizontal: 32, vertical: 24);
    if (isTablet) return EdgeInsets.symmetric(horizontal: 24, vertical: 16);
    return EdgeInsets.all(16);
  }
}

// 响应式网格组合
class ResponsiveGrid extends StatelessWidget {
  final List<Widget> children;
  final double spacing;
  final double runSpacing;
  
  const ResponsiveGrid({
    Key? key,
    required this.children,
    this.spacing = 16.0,
    this.runSpacing = 16.0,
  }) : super(key: key);
  
  @override
  Widget build(BuildContext context) {
    return LayoutBuilder(
      builder: (context, constraints) {
        final crossAxisCount = _getCrossAxisCount(constraints.maxWidth);
        
        return GridView.builder(
          gridDelegate: SliverGridDelegateWithFixedCrossAxisCount(
            crossAxisCount: crossAxisCount,
            crossAxisSpacing: spacing,
            mainAxisSpacing: runSpacing,
            childAspectRatio: _getChildAspectRatio(crossAxisCount),
          ),
          itemCount: children.length,
          itemBuilder: (context, index) => children[index],
        );
      },
    );
  }
  
  int _getCrossAxisCount(double width) {
    if (width > 1200) return 4;
    if (width > 800) return 3;
    if (width > 600) return 2;
    return 1;
  }
  
  double _getChildAspectRatio(int crossAxisCount) {
    switch (crossAxisCount) {
      case 1: return 16 / 9;
      case 2: return 4 / 3;
      case 3: return 1;
      case 4: return 3 / 4;
      default: return 1;
    }
  }
}
```

### 表单组合模式

```dart
// 表单字段组合
class FormFieldWrapper extends StatelessWidget {
  final String label;
  final Widget child;
  final String? errorText;
  final String? helperText;
  final bool required;
  
  const FormFieldWrapper({
    Key? key,
    required this.label,
    required this.child,
    this.errorText,
    this.helperText,
    this.required = false,
  }) : super(key: key);
  
  @override
  Widget build(BuildContext context) {
    return Column(
      crossAxisAlignment: CrossAxisAlignment.start,
      children: [
        _buildLabel(),
        SizedBox(height: 8),
        child,
        if (errorText != null || helperText != null)
          SizedBox(height: 4),
        if (errorText != null)
          _buildErrorText()
        else if (helperText != null)
          _buildHelperText(),
      ],
    );
  }
  
  Widget _buildLabel() {
    return RichText(
      text: TextSpan(
        style: TextStyle(
          fontSize: 14,
          fontWeight: FontWeight.w500,
          color: Colors.grey[700],
        ),
        children: [
          TextSpan(text: label),
          if (required)
            TextSpan(
              text: ' *',
              style: TextStyle(color: Colors.red),
            ),
        ],
      ),
    );
  }
  
  Widget _buildErrorText() {
    return Text(
      errorText!,
      style: TextStyle(
        fontSize: 12,
        color: Colors.red,
      ),
    );
  }
  
  Widget _buildHelperText() {
    return Text(
      helperText!,
      style: TextStyle(
        fontSize: 12,
        color: Colors.grey[600],
      ),
    );
  }
}

// 复合表单组件
class CustomTextFormField extends StatelessWidget {
  final String label;
  final String? hintText;
  final TextEditingController? controller;
  final String? Function(String?)? validator;
  final TextInputType? keyboardType;
  final bool obscureText;
  final bool required;
  final String? helperText;
  final Widget? prefixIcon;
  final Widget? suffixIcon;
  
  const CustomTextFormField({
    Key? key,
    required this.label,
    this.hintText,
    this.controller,
    this.validator,
    this.keyboardType,
    this.obscureText = false,
    this.required = false,
    this.helperText,
    this.prefixIcon,
    this.suffixIcon,
  }) : super(key: key);
  
  @override
  Widget build(BuildContext context) {
    return FormFieldWrapper(
      label: label,
      required: required,
      helperText: helperText,
      child: TextFormField(
        controller: controller,
        validator: validator,
        keyboardType: keyboardType,
        obscureText: obscureText,
        decoration: InputDecoration(
          hintText: hintText,
          prefixIcon: prefixIcon,
          suffixIcon: suffixIcon,
          border: OutlineInputBorder(
            borderRadius: BorderRadius.circular(8),
          ),
          enabledBorder: OutlineInputBorder(
            borderRadius: BorderRadius.circular(8),
            borderSide: BorderSide(color: Colors.grey[300]!),
          ),
          focusedBorder: OutlineInputBorder(
            borderRadius: BorderRadius.circular(8),
            borderSide: BorderSide(color: Theme.of(context).primaryColor),
          ),
          errorBorder: OutlineInputBorder(
            borderRadius: BorderRadius.circular(8),
            borderSide: BorderSide(color: Colors.red),
          ),
        ),
      ),
    );
  }
}

// 表单组合使用示例
class RegistrationForm extends StatefulWidget {
  @override
  _RegistrationFormState createState() => _RegistrationFormState();
}

class _RegistrationFormState extends State<RegistrationForm> {
  final _formKey = GlobalKey<FormState>();
  final _usernameController = TextEditingController();
  final _emailController = TextEditingController();
  final _passwordController = TextEditingController();
  bool _obscurePassword = true;
  
  @override
  Widget build(BuildContext context) {
    return ResponsiveContainer(
      child: Form(
        key: _formKey,
        child: Column(
          children: [
            CustomTextFormField(
              label: '用户名',
              hintText: '请输入用户名',
              controller: _usernameController,
              required: true,
              helperText: '用户名长度为3-20个字符',
              prefixIcon: Icon(Icons.person),
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
            CustomTextFormField(
              label: '邮箱',
              hintText: '请输入邮箱地址',
              controller: _emailController,
              keyboardType: TextInputType.emailAddress,
              required: true,
              prefixIcon: Icon(Icons.email),
              validator: (value) {
                if (value == null || value.isEmpty) {
                  return '请输入邮箱';
                }
                if (!RegExp(r'^[\w-\.]+@([\w-]+\.)+[\w-]{2,4}$').hasMatch(value)) {
                  return '邮箱格式不正确';
                }
                return null;
              },
            ),
            SizedBox(height: 16),
            CustomTextFormField(
              label: '密码',
              hintText: '请输入密码',
              controller: _passwordController,
              obscureText: _obscurePassword,
              required: true,
              helperText: '密码至少8个字符，包含字母和数字',
              prefixIcon: Icon(Icons.lock),
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
              validator: (value) {
                if (value == null || value.isEmpty) {
                  return '请输入密码';
                }
                if (value.length < 8) {
                  return '密码至少8个字符';
                }
                return null;
              },
            ),
            SizedBox(height: 24),
            SizedBox(
              width: double.infinity,
              height: 48,
              child: ElevatedButton(
                onPressed: _submitForm,
                child: Text('注册'),
              ),
            ),
          ],
        ),
      ),
    );
  }
  
  void _submitForm() {
    if (_formKey.currentState!.validate()) {
      // 处理表单提交
      print('表单验证通过');
    }
  }
  
  @override
  void dispose() {
    _usernameController.dispose();
    _emailController.dispose();
    _passwordController.dispose();
    super.dispose();
  }
}
```

### 列表组合模式

```dart
// 列表项组合基类
abstract class ListItemWidget extends StatelessWidget {
  const ListItemWidget({Key? key}) : super(key: key);
  
  @override
  Widget build(BuildContext context) {
    return Container(
      margin: EdgeInsets.symmetric(horizontal: 16, vertical: 4),
      decoration: BoxDecoration(
        color: Colors.white,
        borderRadius: BorderRadius.circular(8),
        boxShadow: [
          BoxShadow(
            color: Colors.grey.withOpacity(0.1),
            spreadRadius: 1,
            blurRadius: 3,
            offset: Offset(0, 1),
          ),
        ],
      ),
      child: buildContent(context),
    );
  }
  
  Widget buildContent(BuildContext context);
}

// 文本列表项
class TextListItem extends ListItemWidget {
  final String title;
  final String? subtitle;
  final Widget? leading;
  final Widget? trailing;
  final VoidCallback? onTap;
  
  const TextListItem({
    Key? key,
    required this.title,
    this.subtitle,
    this.leading,
    this.trailing,
    this.onTap,
  }) : super(key: key);
  
  @override
  Widget buildContent(BuildContext context) {
    return ListTile(
      leading: leading,
      title: Text(title),
      subtitle: subtitle != null ? Text(subtitle!) : null,
      trailing: trailing,
      onTap: onTap,
    );
  }
}

// 图片列表项
class ImageListItem extends ListItemWidget {
  final String imageUrl;
  final String title;
  final String? description;
  final VoidCallback? onTap;
  
  const ImageListItem({
    Key? key,
    required this.imageUrl,
    required this.title,
    this.description,
    this.onTap,
  }) : super(key: key);
  
  @override
  Widget buildContent(BuildContext context) {
    return InkWell(
      onTap: onTap,
      borderRadius: BorderRadius.circular(8),
      child: Padding(
        padding: EdgeInsets.all(12),
        child: Row(
          children: [
            ClipRRect(
              borderRadius: BorderRadius.circular(6),
              child: Image.network(
                imageUrl,
                width: 60,
                height: 60,
                fit: BoxFit.cover,
                errorBuilder: (context, error, stackTrace) {
                  return Container(
                    width: 60,
                    height: 60,
                    color: Colors.grey[300],
                    child: Icon(Icons.image_not_supported),
                  );
                },
              ),
            ),
            SizedBox(width: 12),
            Expanded(
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  Text(
                    title,
                    style: TextStyle(
                      fontSize: 16,
                      fontWeight: FontWeight.w500,
                    ),
                    maxLines: 1,
                    overflow: TextOverflow.ellipsis,
                  ),
                  if (description != null) ..[
                    SizedBox(height: 4),
                    Text(
                      description!,
                      style: TextStyle(
                        fontSize: 14,
                        color: Colors.grey[600],
                      ),
                      maxLines: 2,
                      overflow: TextOverflow.ellipsis,
                    ),
                  ],
                ],
              ),
            ),
          ],
        ),
      ),
    );
  }
}

// 复合列表组合
class CompositeList extends StatelessWidget {
  final List<ListItemWidget> items;
  final Widget? header;
  final Widget? footer;
  final bool shrinkWrap;
  final ScrollPhysics? physics;
  
  const CompositeList({
    Key? key,
    required this.items,
    this.header,
    this.footer,
    this.shrinkWrap = false,
    this.physics,
  }) : super(key: key);
  
  @override
  Widget build(BuildContext context) {
    return ListView.builder(
      shrinkWrap: shrinkWrap,
      physics: physics,
      itemCount: _getTotalItemCount(),
      itemBuilder: (context, index) {
        if (header != null && index == 0) {
          return header!;
        }
        
        final itemIndex = header != null ? index - 1 : index;
        
        if (itemIndex < items.length) {
          return items[itemIndex];
        }
        
        // Footer
        return footer!;
      },
    );
  }
  
  int _getTotalItemCount() {
    int count = items.length;
    if (header != null) count++;
    if (footer != null) count++;
    return count;
  }
}
```

## 测试和调试

### Widget 组合测试

```dart
import 'package:flutter/material.dart';
import 'package:flutter_test/flutter_test.dart';

void main() {
  group('Widget 组合测试', () {
    testWidgets('CustomCard 应该正确渲染子组件', (WidgetTester tester) async {
      const testText = '测试内容';
      
      await tester.pumpWidget(
        MaterialApp(
          home: Scaffold(
            body: CustomCard(
              child: Text(testText),
            ),
          ),
        ),
      );
      
      expect(find.text(testText), findsOneWidget);
      expect(find.byType(Card), findsOneWidget);
    });
    
    testWidgets('UserInfoCard 应该显示用户信息', (WidgetTester tester) async {
      final user = User(
        id: '1',
        name: '测试用户',
        title: '软件工程师',
        bio: '这是一个测试用户',
        joinDate: DateTime.now().subtract(Duration(days: 30)),
      );
      
      await tester.pumpWidget(
        MaterialApp(
          home: Scaffold(
            body: UserInfoCard(user: user),
          ),
        ),
      );
      
      expect(find.text('测试用户'), findsOneWidget);
      expect(find.text('软件工程师'), findsOneWidget);
      expect(find.text('这是一个测试用户'), findsOneWidget);
    });
    
    testWidgets('ResponsiveGrid 应该根据屏幕宽度调整列数', (WidgetTester tester) async {
      await tester.pumpWidget(
        MaterialApp(
          home: Scaffold(
            body: ResponsiveGrid(
              children: List.generate(10, (index) => Container()),
            ),
          ),
        ),
      );
      
      expect(find.byType(GridView), findsOneWidget);
    });
  });
  
  group('表单组合测试', () {
    testWidgets('CustomTextFormField 应该显示标签和输入框', (WidgetTester tester) async {
      await tester.pumpWidget(
        MaterialApp(
          home: Scaffold(
            body: CustomTextFormField(
              label: '用户名',
              hintText: '请输入用户名',
            ),
          ),
        ),
      );
      
      expect(find.text('用户名'), findsOneWidget);
      expect(find.byType(TextFormField), findsOneWidget);
    });
    
    testWidgets('FormFieldWrapper 应该显示必填标记', (WidgetTester tester) async {
      await tester.pumpWidget(
        MaterialApp(
          home: Scaffold(
            body: FormFieldWrapper(
              label: '必填字段',
              required: true,
              child: Container(),
            ),
          ),
        ),
      );
      
      expect(find.text('必填字段'), findsOneWidget);
      expect(find.text(' *'), findsOneWidget);
    });
  });
}
```

### Widget 组合调试器

```dart
class WidgetCompositionDebugger {
  static final Map<String, int> _widgetCounts = {};
  static final List<WidgetBuildInfo> _buildHistory = [];
  
  static void logWidgetBuild(String widgetName, [Map<String, dynamic>? props]) {
    _widgetCounts[widgetName] = (_widgetCounts[widgetName] ?? 0) + 1;
    
    final buildInfo = WidgetBuildInfo(
      widgetName: widgetName,
      props: props ?? {},
      timestamp: DateTime.now(),
      buildCount: _widgetCounts[widgetName]!,
    );
    
    _buildHistory.add(buildInfo);
    
    if (kDebugMode) {
      print('Widget构建: $widgetName (第${buildInfo.buildCount}次)');
      if (props != null && props.isNotEmpty) {
        print('  属性: $props');
      }
    }
  }
  
  static Map<String, int> getWidgetCounts() => Map.unmodifiable(_widgetCounts);
  
  static List<WidgetBuildInfo> getBuildHistory() => List.unmodifiable(_buildHistory);
  
  static void clearHistory() {
    _widgetCounts.clear();
    _buildHistory.clear();
  }
  
  static void printStatistics() {
    print('=== Widget 组合统计 ===');
    print('总构建次数: ${_buildHistory.length}');
    print('Widget类型数量: ${_widgetCounts.length}');
    print('\n各Widget构建次数:');
    _widgetCounts.entries
        .toList()
        ..sort((a, b) => b.value.compareTo(a.value))
        ..forEach((entry) {
          print('  ${entry.key}: ${entry.value}次');
        });
  }
}

class WidgetBuildInfo {
  final String widgetName;
  final Map<String, dynamic> props;
  final DateTime timestamp;
  final int buildCount;
  
  WidgetBuildInfo({
    required this.widgetName,
    required this.props,
    required this.timestamp,
    required this.buildCount,
  });
}

// 带调试功能的Widget基类
abstract class DebuggableWidget extends StatelessWidget {
  const DebuggableWidget({Key? key}) : super(key: key);
  
  @override
  Widget build(BuildContext context) {
    if (kDebugMode) {
      WidgetCompositionDebugger.logWidgetBuild(
        runtimeType.toString(),
        getDebugProps(),
      );
    }
    return buildWidget(context);
  }
  
  Widget buildWidget(BuildContext context);
  
  Map<String, dynamic>? getDebugProps() => null;
}
```

## 最佳实践

### 设计原则

1. **组合优于继承**
   ```dart
   // 好的做法：使用组合
   class ComplexButton extends StatelessWidget {
     final Widget icon;
     final Widget label;
     final VoidCallback? onPressed;
     
     @override
     Widget build(BuildContext context) {
       return ElevatedButton(
         onPressed: onPressed,
         child: Row(
           children: [icon, SizedBox(width: 8), label],
         ),
       );
     }
   }
   
   // 避免：过度继承
   class BadComplexButton extends ElevatedButton {
     // 复杂的继承实现
   }
   ```

2. **单一职责原则**
   ```dart
   // 好的做法：职责明确
   class UserAvatar extends StatelessWidget {
     // 只负责显示头像
   }
   
   class UserInfo extends StatelessWidget {
     // 只负责显示用户信息
   }
   
   class UserCard extends StatelessWidget {
     // 组合头像和信息
     @override
     Widget build(BuildContext context) {
       return Row(
         children: [
           UserAvatar(user: user),
           UserInfo(user: user),
         ],
       );
     }
   }
   ```

3. **可配置性**
   ```dart
   class ConfigurableCard extends StatelessWidget {
     final Widget child;
     final EdgeInsetsGeometry? padding;
     final Color? backgroundColor;
     final double? elevation;
     final BorderRadius? borderRadius;
     final VoidCallback? onTap;
     
     // 提供丰富的配置选项
   }
   ```

### 实现建议

1. **合理的Widget粒度**
   - 不要创建过于细粒度的Widget
   - 避免过于粗粒度的Widget
   - 根据复用性决定粒度

2. **性能优化**
   ```dart
   class OptimizedWidget extends StatelessWidget {
     const OptimizedWidget({Key? key}) : super(key: key);
     
     @override
     Widget build(BuildContext context) {
       return const SizedBox(); // 使用const构造函数
     }
   }
   ```

3. **状态管理**
   ```dart
   // 将状态提升到合适的层级
   class StatefulParent extends StatefulWidget {
     @override
     _StatefulParentState createState() => _StatefulParentState();
   }
   
   class _StatefulParentState extends State<StatefulParent> {
     String _data = '';
     
     @override
     Widget build(BuildContext context) {
       return Column(
         children: [
           StatelessChild(data: _data),
           StatelessChild(data: _data),
         ],
       );
     }
   }
   ```

### 注意事项

1. **避免深层嵌套**
   ```dart
   // 避免：过深的嵌套
   Widget badNesting() {
     return Container(
       child: Padding(
         child: Column(
           children: [
             Container(
               child: Row(
                 children: [
                   Container(
                     child: Text('深层嵌套'),
                   ),
                 ],
               ),
             ),
           ],
         ),
       ),
     );
   }
   
   // 好的做法：提取子组件
   Widget goodComposition() {
     return Container(
       padding: EdgeInsets.all(16),
       child: Column(
         children: [
           _buildHeader(),
           _buildContent(),
         ],
       ),
     );
   }
   ```

2. **合理使用const**
   ```dart
   // 好的做法：使用const优化性能
   const Text('静态文本')
   const Icon(Icons.home)
   const SizedBox(height: 16)
   ```

3. **避免在build方法中创建复杂对象**
   ```dart
   class GoodWidget extends StatelessWidget {
     // 将复杂对象提取为类成员
     static const _textStyle = TextStyle(
       fontSize: 16,
       fontWeight: FontWeight.bold,
     );
     
     @override
     Widget build(BuildContext context) {
       return Text('文本', style: _textStyle);
     }
   }
   ```

Widget 组合模式是 Flutter 开发的核心，通过合理的组合设计，可以创建出高度可复用、可维护和高性能的用户界面。掌握这种模式对于 Flutter 开发者来说至关重要。