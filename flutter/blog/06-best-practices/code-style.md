# Flutter 代码风格和规范指南

本文档详细介绍了 Flutter 开发中的代码风格规范，包括 Dart 语言规范、项目结构、命名约定、代码格式化等最佳实践。

## 📋 目录

- [Dart 代码风格](#dart-代码风格)
- [命名约定](#命名约定)
- [文件和目录组织](#文件和目录组织)
- [代码格式化](#代码格式化)
- [注释和文档](#注释和文档)
- [导入和依赖](#导入和依赖)
- [Widget 编写规范](#widget-编写规范)
- [状态管理规范](#状态管理规范)
- [错误处理规范](#错误处理规范)
- [Git 提交规范](#git-提交规范)

## 🎨 Dart 代码风格

### 基本原则

遵循 [Dart 官方风格指南](https://dart.dev/guides/language/effective-dart/style)：

```dart
// ✅ 好的示例
class UserRepository {
  final ApiService _apiService;
  final CacheService _cacheService;
  
  const UserRepository({
    required ApiService apiService,
    required CacheService cacheService,
  }) : _apiService = apiService,
       _cacheService = cacheService;
  
  Future<User?> getUserById(String id) async {
    try {
      // 先检查缓存
      final cachedUser = await _cacheService.getUser(id);
      if (cachedUser != null) {
        return cachedUser;
      }
      
      // 从 API 获取
      final user = await _apiService.fetchUser(id);
      if (user != null) {
        await _cacheService.saveUser(user);
      }
      
      return user;
    } catch (e) {
      throw UserRepositoryException('Failed to get user: $e');
    }
  }
}

// ❌ 不好的示例
class userrepository {
  var api;
  var cache;
  
  userrepository(api, cache) {
    this.api = api;
    this.cache = cache;
  }
  
  getUser(id) async {
    var user = await cache.getUser(id);
    if (user == null) {
      user = await api.fetchUser(id);
      cache.saveUser(user);
    }
    return user;
  }
}
```

### 代码风格要点

1. **使用 `const` 构造函数**
```dart
// ✅ 好的
class MyWidget extends StatelessWidget {
  const MyWidget({super.key, required this.title});
  
  final String title;
}

// ❌ 不好的
class MyWidget extends StatelessWidget {
  MyWidget({Key? key, required this.title}) : super(key: key);
  
  final String title;
}
```

2. **优先使用单引号**
```dart
// ✅ 好的
const String message = 'Hello, World!';
const String template = 'User: $userName';

// ❌ 不好的
const String message = "Hello, World!";
```

3. **避免不必要的 `new` 和 `const`**
```dart
// ✅ 好的
final list = <String>[];
final widget = Container();
const padding = EdgeInsets.all(8.0);

// ❌ 不好的
final list = new List<String>();
final widget = new Container();
const padding = const EdgeInsets.all(8.0);
```

## 📝 命名约定

### 类和枚举

使用 **PascalCase**（大驼峰命名法）：

```dart
// ✅ 好的示例
class UserProfile {}
class ApiService {}
enum UserStatus { active, inactive, pending }
enum NetworkState { connected, disconnected, connecting }

// ❌ 不好的示例
class userProfile {}
class apiservice {}
enum userStatus { active, inactive, pending }
```

### 变量、函数和参数

使用 **camelCase**（小驼峰命名法）：

```dart
// ✅ 好的示例
String userName = 'John';
int maxRetryCount = 3;
bool isLoggedIn = false;

void fetchUserData() {}
Future<void> saveUserPreferences() async {}

void updateUser({
  required String userId,
  required String displayName,
  String? profileImageUrl,
}) {}

// ❌ 不好的示例
String user_name = 'John';
int MaxRetryCount = 3;
bool IsLoggedIn = false;

void fetch_user_data() {}
void FetchUserData() {}
```

### 常量

使用 **lowerCamelCase**：

```dart
// ✅ 好的示例
const int maxLoginAttempts = 3;
const String apiBaseUrl = 'https://api.example.com';
const Duration requestTimeout = Duration(seconds: 30);

// ❌ 不好的示例
const int MAX_LOGIN_ATTEMPTS = 3;
const String API_BASE_URL = 'https://api.example.com';
```

### 私有成员

使用下划线前缀：

```dart
class UserService {
  final ApiClient _apiClient;
  final Logger _logger;
  
  String? _cachedToken;
  
  Future<void> _refreshToken() async {
    // 私有方法实现
  }
  
  void _logError(String message) {
    _logger.error(message);
  }
}
```

### 文件命名

使用 **snake_case**（下划线命名法）：

```
✅ 好的示例
user_profile.dart
api_service.dart
network_state.dart
home_page.dart
user_repository.dart

❌ 不好的示例
UserProfile.dart
apiService.dart
NetworkState.dart
HomePage.dart
```

## 📁 文件和目录组织

### 推荐的项目结构

```
lib/
├── main.dart                    # 应用入口
├── app/
│   ├── app.dart                # 应用配置
│   ├── routes.dart             # 路由配置
│   └── theme.dart              # 主题配置
├── core/
│   ├── constants/
│   │   ├── app_constants.dart
│   │   ├── api_constants.dart
│   │   └── ui_constants.dart
│   ├── errors/
│   │   ├── exceptions.dart
│   │   └── failures.dart
│   ├── network/
│   │   ├── api_client.dart
│   │   └── network_info.dart
│   ├── utils/
│   │   ├── date_utils.dart
│   │   ├── validation_utils.dart
│   │   └── format_utils.dart
│   └── services/
│       ├── storage_service.dart
│       ├── notification_service.dart
│       └── analytics_service.dart
├── features/
│   ├── authentication/
│   │   ├── data/
│   │   │   ├── datasources/
│   │   │   ├── models/
│   │   │   └── repositories/
│   │   ├── domain/
│   │   │   ├── entities/
│   │   │   ├── repositories/
│   │   │   └── usecases/
│   │   └── presentation/
│   │       ├── pages/
│   │       ├── widgets/
│   │       └── providers/
│   └── user_profile/
│       ├── data/
│       ├── domain/
│       └── presentation/
└── shared/
    ├── widgets/
    │   ├── buttons/
    │   ├── forms/
    │   └── loading/
    ├── models/
    ├── extensions/
    └── mixins/
```

### 文件组织原则

1. **按功能分组**：相关的文件放在同一目录
2. **分层架构**：数据层、业务层、表示层分离
3. **共享组件**：可复用的组件放在 `shared` 目录
4. **核心功能**：基础设施代码放在 `core` 目录

### 导入顺序

```dart
// 1. Dart 核心库
import 'dart:async';
import 'dart:convert';

// 2. Flutter 框架
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';

// 3. 第三方包
import 'package:dio/dio.dart';
import 'package:provider/provider.dart';
import 'package:shared_preferences/shared_preferences.dart';

// 4. 项目内部导入
import '../../../core/constants/app_constants.dart';
import '../../../core/errors/exceptions.dart';
import '../../../shared/widgets/loading_widget.dart';
import '../data/models/user_model.dart';
import '../domain/entities/user.dart';
```

## 🎯 代码格式化

### 配置 `analysis_options.yaml`

```yaml
analyzer:
  exclude:
    - "**/*.g.dart"
    - "**/*.freezed.dart"
    - "**/*.gr.dart"
  
  strong-mode:
    implicit-casts: false
    implicit-dynamic: false
  
  errors:
    invalid_annotation_target: ignore
    missing_required_param: error
    missing_return: error
    todo: ignore

linter:
  rules:
    # 错误预防
    - avoid_print
    - avoid_returning_null_for_void
    - avoid_slow_async_io
    - cancel_subscriptions
    - close_sinks
    - comment_references
    - control_flow_in_finally
    - empty_statements
    - hash_and_equals
    - invariant_booleans
    - iterable_contains_unrelated_type
    - list_remove_unrelated_type
    - literal_only_boolean_expressions
    - no_adjacent_strings_in_list
    - no_duplicate_case_values
    - no_logic_in_create_state
    - prefer_void_to_null
    - test_types_in_equals
    - throw_in_finally
    - unnecessary_statements
    - unrelated_type_equality_checks
    - use_key_in_widget_constructors
    - valid_regexps
    
    # 风格规范
    - always_declare_return_types
    - always_put_control_body_on_new_line
    - always_put_required_named_parameters_first
    - always_specify_types
    - annotate_overrides
    - avoid_annotating_with_dynamic
    - avoid_as
    - avoid_bool_literals_in_conditional_expressions
    - avoid_catches_without_on_clauses
    - avoid_catching_errors
    - avoid_double_and_int_checks
    - avoid_empty_else
    - avoid_field_initializers_in_const_classes
    - avoid_function_literals_in_foreach_calls
    - avoid_init_to_null
    - avoid_null_checks_in_equality_operators
    - avoid_positional_boolean_parameters
    - avoid_redundant_argument_values
    - avoid_renaming_method_parameters
    - avoid_return_types_on_setters
    - avoid_returning_null
    - avoid_returning_null_for_future
    - avoid_single_cascade_in_expression_statements
    - avoid_types_as_parameter_names
    - avoid_unnecessary_containers
    - avoid_unused_constructor_parameters
    - avoid_void_async
    - await_only_futures
    - camel_case_extensions
    - camel_case_types
    - cascade_invocations
    - constant_identifier_names
    - curly_braces_in_flow_control_structures
    - directives_ordering
    - empty_catches
    - empty_constructor_bodies
    - file_names
    - flutter_style_todos
    - implementation_imports
    - join_return_with_assignment
    - library_names
    - library_prefixes
    - lines_longer_than_80_chars
    - non_constant_identifier_names
    - null_closures
    - omit_local_variable_types
    - one_member_abstracts
    - only_throw_errors
    - overridden_fields
    - package_api_docs
    - package_names
    - package_prefixed_library_names
    - parameter_assignments
    - prefer_adjacent_string_concatenation
    - prefer_asserts_in_initializer_lists
    - prefer_asserts_with_message
    - prefer_collection_literals
    - prefer_conditional_assignment
    - prefer_const_constructors
    - prefer_const_constructors_in_immutables
    - prefer_const_declarations
    - prefer_const_literals_to_create_immutables
    - prefer_constructors_over_static_methods
    - prefer_contains
    - prefer_equal_for_default_values
    - prefer_expression_function_bodies
    - prefer_final_fields
    - prefer_final_in_for_each
    - prefer_final_locals
    - prefer_for_elements_to_map_fromIterable
    - prefer_foreach
    - prefer_function_declarations_over_variables
    - prefer_generic_function_type_aliases
    - prefer_if_elements_to_conditional_expressions
    - prefer_if_null_operators
    - prefer_initializing_formals
    - prefer_inlined_adds
    - prefer_int_literals
    - prefer_interpolation_to_compose_strings
    - prefer_is_empty
    - prefer_is_not_empty
    - prefer_is_not_operator
    - prefer_iterable_whereType
    - prefer_null_aware_operators
    - prefer_relative_imports
    - prefer_single_quotes
    - prefer_spread_collections
    - prefer_typing_uninitialized_variables
    - provide_deprecation_message
    - recursive_getters
    - slash_for_doc_comments
    - sort_child_properties_last
    - sort_constructors_first
    - sort_unnamed_constructors_first
    - type_annotate_public_apis
    - type_init_formals
    - unawaited_futures
    - unnecessary_await_in_return
    - unnecessary_brace_in_string_interps
    - unnecessary_const
    - unnecessary_getters_setters
    - unnecessary_lambdas
    - unnecessary_new
    - unnecessary_null_aware_assignments
    - unnecessary_null_in_if_null_operators
    - unnecessary_overrides
    - unnecessary_parenthesis
    - unnecessary_raw_strings
    - unnecessary_string_escapes
    - unnecessary_string_interpolations
    - unnecessary_this
    - use_full_hex_values_for_flutter_colors
    - use_function_type_syntax_for_parameters
    - use_rethrow_when_possible
    - void_checks
```

### 自动格式化配置

在 VS Code 中配置 `settings.json`：

```json
{
  "dart.lineLength": 80,
  "editor.formatOnSave": true,
  "editor.formatOnType": true,
  "dart.insertArgumentPlaceholders": false,
  "dart.updateImportsOnRename": true,
  "dart.organizeDirectivesOnSave": true,
  "dart.fixImportsOnSave": true
}
```

## 📝 注释和文档

### 文档注释

使用三斜杠 `///` 进行文档注释：

```dart
/// 用户认证服务
/// 
/// 提供用户登录、注册、登出等功能。
/// 支持多种认证方式：邮箱密码、第三方登录等。
/// 
/// 示例用法：
/// ```dart
/// final authService = AuthService();
/// final user = await authService.signInWithEmail(
///   email: 'user@example.com',
///   password: 'password123',
/// );
/// ```
class AuthService {
  /// 使用邮箱和密码登录
  /// 
  /// [email] 用户邮箱地址
  /// [password] 用户密码
  /// 
  /// 返回登录成功的用户信息，失败时抛出 [AuthException]
  /// 
  /// 抛出异常：
  /// - [InvalidEmailException] 邮箱格式不正确
  /// - [InvalidPasswordException] 密码不符合要求
  /// - [UserNotFoundException] 用户不存在
  /// - [NetworkException] 网络连接失败
  Future<User> signInWithEmail({
    required String email,
    required String password,
  }) async {
    // 实现代码
  }
  
  /// 检查用户是否已登录
  /// 
  /// 返回 `true` 表示用户已登录，`false` 表示未登录
  bool get isSignedIn => _currentUser != null;
}
```

### 代码注释

```dart
class DataProcessor {
  Future<List<ProcessedData>> processData(List<RawData> rawData) async {
    // 过滤无效数据
    final validData = rawData.where((data) => data.isValid).toList();
    
    // TODO: 添加数据验证逻辑
    // FIXME: 处理空数据的情况
    
    final processedData = <ProcessedData>[];
    
    for (final data in validData) {
      // 转换数据格式
      final processed = await _transformData(data);
      
      // 应用业务规则
      if (_shouldIncludeData(processed)) {
        processedData.add(processed);
      }
    }
    
    return processedData;
  }
  
  /// 检查数据是否应该被包含在结果中
  /// 
  /// 基于业务规则判断数据的有效性
  bool _shouldIncludeData(ProcessedData data) {
    // 业务逻辑实现
    return data.score > 0.5 && data.category != Category.excluded;
  }
}
```

## 🏗️ Widget 编写规范

### StatelessWidget 规范

```dart
/// 用户头像组件
/// 
/// 显示用户头像，支持占位符和加载状态
class UserAvatar extends StatelessWidget {
  /// 创建用户头像组件
  const UserAvatar({
    super.key,
    required this.user,
    this.size = 40.0,
    this.onTap,
  });
  
  /// 用户信息
  final User user;
  
  /// 头像大小，默认为 40.0
  final double size;
  
  /// 点击回调
  final VoidCallback? onTap;
  
  @override
  Widget build(BuildContext context) {
    return GestureDetector(
      onTap: onTap,
      child: CircleAvatar(
        radius: size / 2,
        backgroundImage: user.avatarUrl != null
            ? NetworkImage(user.avatarUrl!)
            : null,
        child: user.avatarUrl == null
            ? Icon(
                Icons.person,
                size: size * 0.6,
                color: Theme.of(context).colorScheme.onSurface,
              )
            : null,
      ),
    );
  }
}
```

### StatefulWidget 规范

```dart
/// 搜索输入框组件
/// 
/// 提供实时搜索功能，支持防抖和清空操作
class SearchInput extends StatefulWidget {
  /// 创建搜索输入框
  const SearchInput({
    super.key,
    required this.onSearch,
    this.hintText = '搜索...',
    this.debounceTime = const Duration(milliseconds: 500),
  });
  
  /// 搜索回调函数
  final ValueChanged<String> onSearch;
  
  /// 提示文本
  final String hintText;
  
  /// 防抖时间
  final Duration debounceTime;
  
  @override
  State<SearchInput> createState() => _SearchInputState();
}

class _SearchInputState extends State<SearchInput> {
  late final TextEditingController _controller;
  Timer? _debounceTimer;
  
  @override
  void initState() {
    super.initState();
    _controller = TextEditingController();
  }
  
  @override
  void dispose() {
    _controller.dispose();
    _debounceTimer?.cancel();
    super.dispose();
  }
  
  void _onSearchChanged(String value) {
    _debounceTimer?.cancel();
    _debounceTimer = Timer(widget.debounceTime, () {
      widget.onSearch(value);
    });
  }
  
  void _clearSearch() {
    _controller.clear();
    widget.onSearch('');
  }
  
  @override
  Widget build(BuildContext context) {
    return TextField(
      controller: _controller,
      onChanged: _onSearchChanged,
      decoration: InputDecoration(
        hintText: widget.hintText,
        prefixIcon: const Icon(Icons.search),
        suffixIcon: _controller.text.isNotEmpty
            ? IconButton(
                icon: const Icon(Icons.clear),
                onPressed: _clearSearch,
              )
            : null,
        border: OutlineInputBorder(
          borderRadius: BorderRadius.circular(8.0),
        ),
      ),
    );
  }
}
```

## 🔄 状态管理规范

### Provider 使用规范

```dart
/// 用户状态管理
class UserProvider extends ChangeNotifier {
  User? _currentUser;
  bool _isLoading = false;
  String? _errorMessage;
  
  /// 当前用户
  User? get currentUser => _currentUser;
  
  /// 是否正在加载
  bool get isLoading => _isLoading;
  
  /// 错误信息
  String? get errorMessage => _errorMessage;
  
  /// 是否已登录
  bool get isSignedIn => _currentUser != null;
  
  /// 登录用户
  Future<void> signIn({
    required String email,
    required String password,
  }) async {
    _setLoading(true);
    _clearError();
    
    try {
      final user = await _authService.signInWithEmail(
        email: email,
        password: password,
      );
      
      _currentUser = user;
      notifyListeners();
    } catch (e) {
      _setError(e.toString());
    } finally {
      _setLoading(false);
    }
  }
  
  /// 登出用户
  Future<void> signOut() async {
    _setLoading(true);
    
    try {
      await _authService.signOut();
      _currentUser = null;
      notifyListeners();
    } catch (e) {
      _setError(e.toString());
    } finally {
      _setLoading(false);
    }
  }
  
  void _setLoading(bool loading) {
    _isLoading = loading;
    notifyListeners();
  }
  
  void _setError(String error) {
    _errorMessage = error;
    notifyListeners();
  }
  
  void _clearError() {
    _errorMessage = null;
  }
}
```

## ⚠️ 错误处理规范

### 自定义异常

```dart
/// 基础异常类
abstract class AppException implements Exception {
  const AppException(this.message, [this.code]);
  
  final String message;
  final String? code;
  
  @override
  String toString() => 'AppException: $message';
}

/// 网络异常
class NetworkException extends AppException {
  const NetworkException(super.message, [super.code]);
}

/// 认证异常
class AuthException extends AppException {
  const AuthException(super.message, [super.code]);
}

/// 验证异常
class ValidationException extends AppException {
  const ValidationException(super.message, [super.code]);
}
```

### 错误处理模式

```dart
class ApiService {
  Future<T> _handleRequest<T>(
    Future<Response> Function() request,
    T Function(Map<String, dynamic>) fromJson,
  ) async {
    try {
      final response = await request();
      
      if (response.statusCode == 200) {
        return fromJson(response.data as Map<String, dynamic>);
      } else {
        throw NetworkException(
          'Request failed with status: ${response.statusCode}',
          response.statusCode.toString(),
        );
      }
    } on DioException catch (e) {
      switch (e.type) {
        case DioExceptionType.connectionTimeout:
        case DioExceptionType.sendTimeout:
        case DioExceptionType.receiveTimeout:
          throw NetworkException('Request timeout', 'TIMEOUT');
        case DioExceptionType.connectionError:
          throw NetworkException('Connection error', 'CONNECTION_ERROR');
        default:
          throw NetworkException('Network error: ${e.message}', 'UNKNOWN');
      }
    } catch (e) {
      throw NetworkException('Unexpected error: $e', 'UNEXPECTED');
    }
  }
}
```

## 📋 Git 提交规范

### 提交信息格式

```
<type>(<scope>): <subject>

<body>

<footer>
```

### 类型说明

- `feat`: 新功能
- `fix`: 修复 bug
- `docs`: 文档更新
- `style`: 代码格式化（不影响功能）
- `refactor`: 代码重构
- `test`: 添加或修改测试
- `chore`: 构建过程或辅助工具的变动
- `perf`: 性能优化
- `ci`: CI/CD 相关变更

### 提交示例

```bash
# 新功能
git commit -m "feat(auth): add Google sign-in support

Implement Google OAuth integration for user authentication.
Add GoogleSignInService and update AuthProvider.

Closes #123"

# 修复 bug
git commit -m "fix(ui): resolve button alignment issue on small screens

Fix button layout problem in LoginPage when screen width < 360px.
Update responsive breakpoints and button constraints.

Fixes #456"

# 重构
git commit -m "refactor(network): extract common HTTP error handling

Create BaseApiService with shared error handling logic.
Reduce code duplication across service classes.

No breaking changes."
```

### 分支命名规范

```bash
# 功能分支
feature/user-authentication
feature/payment-integration

# 修复分支
fix/login-validation-error
fix/memory-leak-in-image-cache

# 热修复分支
hotfix/critical-crash-on-startup

# 发布分支
release/v1.2.0
```

## 🛠️ 开发工具配置

### VS Code 扩展推荐

```json
{
  "recommendations": [
    "dart-code.dart-code",
    "dart-code.flutter",
    "ms-vscode.vscode-json",
    "bradlc.vscode-tailwindcss",
    "esbenp.prettier-vscode",
    "ms-vscode.vscode-typescript-next",
    "usernamehw.errorlens",
    "gruntfuggly.todo-tree",
    "streetsidesoftware.code-spell-checker"
  ]
}
```

### 代码片段配置

```json
{
  "Flutter StatelessWidget": {
    "prefix": "stless",
    "body": [
      "class ${1:WidgetName} extends StatelessWidget {",
      "  const ${1:WidgetName}({super.key});",
      "",
      "  @override",
      "  Widget build(BuildContext context) {",
      "    return ${2:Container}();",
      "  }",
      "}"
    ],
    "description": "Create a StatelessWidget"
  },
  "Flutter StatefulWidget": {
    "prefix": "stful",
    "body": [
      "class ${1:WidgetName} extends StatefulWidget {",
      "  const ${1:WidgetName}({super.key});",
      "",
      "  @override",
      "  State<${1:WidgetName}> createState() => _${1:WidgetName}State();",
      "}",
      "",
      "class _${1:WidgetName}State extends State<${1:WidgetName}> {",
      "  @override",
      "  Widget build(BuildContext context) {",
      "    return ${2:Container}();",
      "  }",
      "}"
    ],
    "description": "Create a StatefulWidget"
  }
}
```

## 📚 总结

遵循这些代码风格和规范将帮助你：

1. **提高代码质量**：统一的风格让代码更易读、易维护
2. **减少错误**：规范的命名和结构减少理解偏差
3. **提升协作效率**：团队成员能快速理解彼此的代码
4. **便于维护**：清晰的组织结构便于后期维护和扩展

记住，代码风格规范不是一成不变的，应该根据团队需求和项目特点进行适当调整。最重要的是保持一致性，让整个团队都遵循相同的标准。