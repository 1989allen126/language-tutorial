# 单例模式 (Singleton Pattern)

单例模式确保一个类只有一个实例，并提供全局访问点。在Flutter开发中，单例模式常用于应用配置、数据库连接、网络客户端等需要全局唯一实例的场景。

## 📋 模式概述

### 定义
单例模式是一种创建型设计模式，它保证一个类只有一个实例，并提供一个全局访问点来获取该实例。

### 适用场景
- 应用配置管理
- 数据库连接池
- 网络请求客户端
- 缓存管理器
- 日志记录器
- 主题管理器

### 优点
- 控制实例数量，节省内存
- 提供全局访问点
- 延迟初始化，提高启动性能
- 线程安全（正确实现时）

### 缺点
- 违反单一职责原则
- 难以进行单元测试
- 可能导致代码耦合
- 在多线程环境下需要特别注意

## 🏗 实现方式

### 1. 饿汉式单例

```dart
// 饿汉式单例 - 类加载时就创建实例
class EagerSingleton {
  // 静态实例，类加载时创建
  static final EagerSingleton _instance = EagerSingleton._internal();
  
  // 私有构造函数
  EagerSingleton._internal();
  
  // 获取实例的静态方法
  static EagerSingleton get instance => _instance;
  
  // 业务方法
  void doSomething() {
    print('EagerSingleton doing something...');
  }
}

// 使用方式
void main() {
  final singleton1 = EagerSingleton.instance;
  final singleton2 = EagerSingleton.instance;
  
  print(identical(singleton1, singleton2)); // true
  singleton1.doSomething();
}
```

### 2. 懒汉式单例

```dart
// 懒汉式单例 - 第一次使用时创建实例
class LazySingleton {
  static LazySingleton? _instance;
  
  // 私有构造函数
  LazySingleton._internal();
  
  // 获取实例的静态方法
  static LazySingleton get instance {
    _instance ??= LazySingleton._internal();
    return _instance!;
  }
  
  // 业务方法
  void doSomething() {
    print('LazySingleton doing something...');
  }
}

// 使用方式
void main() {
  final singleton1 = LazySingleton.instance;
  final singleton2 = LazySingleton.instance;
  
  print(identical(singleton1, singleton2)); // true
  singleton1.doSomething();
}
```

### 3. 线程安全的单例

```dart
import 'dart:async';

// 线程安全的单例
class ThreadSafeSingleton {
  static ThreadSafeSingleton? _instance;
  static final Completer<ThreadSafeSingleton> _completer = Completer();
  static bool _isInitialized = false;
  
  // 私有构造函数
  ThreadSafeSingleton._internal() {
    // 模拟初始化过程
    print('ThreadSafeSingleton initialized');
  }
  
  // 获取实例的静态方法
  static Future<ThreadSafeSingleton> get instance async {
    if (_isInitialized) {
      return _instance!;
    }
    
    if (!_completer.isCompleted) {
      _instance = ThreadSafeSingleton._internal();
      _isInitialized = true;
      _completer.complete(_instance!);
    }
    
    return _completer.future;
  }
  
  // 同步获取实例（如果已初始化）
  static ThreadSafeSingleton? get instanceSync {
    return _isInitialized ? _instance : null;
  }
  
  // 业务方法
  void doSomething() {
    print('ThreadSafeSingleton doing something...');
  }
}

// 使用方式
void main() async {
  final singleton1 = await ThreadSafeSingleton.instance;
  final singleton2 = await ThreadSafeSingleton.instance;
  
  print(identical(singleton1, singleton2)); // true
  singleton1.doSomething();
}
```

## 🎯 Flutter应用实例

### 1. 应用配置管理器

```dart
import 'package:shared_preferences/shared_preferences.dart';

class AppConfig {
  static AppConfig? _instance;
  static AppConfig get instance => _instance ??= AppConfig._internal();
  
  AppConfig._internal();
  
  SharedPreferences? _prefs;
  Map<String, dynamic> _cache = {};
  
  // 初始化配置
  Future<void> initialize() async {
    _prefs = await SharedPreferences.getInstance();
    await _loadConfig();
  }
  
  // 加载配置
  Future<void> _loadConfig() async {
    final keys = _prefs!.getKeys();
    for (final key in keys) {
      _cache[key] = _prefs!.get(key);
    }
  }
  
  // 获取配置值
  T? get<T>(String key, {T? defaultValue}) {
    return _cache[key] as T? ?? defaultValue;
  }
  
  // 设置配置值
  Future<void> set<T>(String key, T value) async {
    _cache[key] = value;
    
    // 持久化到本地存储
    if (value is String) {
      await _prefs!.setString(key, value);
    } else if (value is int) {
      await _prefs!.setInt(key, value);
    } else if (value is double) {
      await _prefs!.setDouble(key, value);
    } else if (value is bool) {
      await _prefs!.setBool(key, value);
    } else if (value is List<String>) {
      await _prefs!.setStringList(key, value);
    }
  }
  
  // 移除配置
  Future<void> remove(String key) async {
    _cache.remove(key);
    await _prefs!.remove(key);
  }
  
  // 清空所有配置
  Future<void> clear() async {
    _cache.clear();
    await _prefs!.clear();
  }
  
  // 获取所有配置
  Map<String, dynamic> getAll() {
    return Map.from(_cache);
  }
}

// 使用示例
class ConfigExample {
  static Future<void> example() async {
    final config = AppConfig.instance;
    await config.initialize();
    
    // 设置配置
    await config.set('theme', 'dark');
    await config.set('language', 'zh-CN');
    await config.set('autoSave', true);
    await config.set('maxRetries', 3);
    
    // 获取配置
    final theme = config.get<String>('theme', defaultValue: 'light');
    final language = config.get<String>('language', defaultValue: 'en-US');
    final autoSave = config.get<bool>('autoSave', defaultValue: false);
    final maxRetries = config.get<int>('maxRetries', defaultValue: 1);
    
    print('主题: $theme');
    print('语言: $language');
    print('自动保存: $autoSave');
    print('最大重试次数: $maxRetries');
  }
}
```

### 2. 网络请求客户端

```dart
import 'package:dio/dio.dart';

class ApiClient {
  static ApiClient? _instance;
  static ApiClient get instance => _instance ??= ApiClient._internal();
  
  ApiClient._internal();
  
  late Dio _dio;
  
  // 初始化网络客户端
  void initialize({
    String? baseUrl,
    Duration? connectTimeout,
    Duration? receiveTimeout,
    Map<String, String>? headers,
  }) {
    _dio = Dio(BaseOptions(
      baseUrl: baseUrl ?? 'https://api.example.com',
      connectTimeout: connectTimeout ?? const Duration(seconds: 30),
      receiveTimeout: receiveTimeout ?? const Duration(seconds: 30),
      headers: headers ?? {
        'Content-Type': 'application/json',
        'Accept': 'application/json',
      },
    ));
    
    // 添加拦截器
    _dio.interceptors.add(LogInterceptor(
      requestBody: true,
      responseBody: true,
    ));
    
    _dio.interceptors.add(InterceptorsWrapper(
      onRequest: (options, handler) {
        // 添加认证token
        final token = AppConfig.instance.get<String>('auth_token');
        if (token != null) {
          options.headers['Authorization'] = 'Bearer $token';
        }
        handler.next(options);
      },
      onError: (error, handler) {
        // 统一错误处理
        _handleError(error);
        handler.next(error);
      },
    ));
  }
  
  // GET请求
  Future<Response<T>> get<T>(
    String path, {
    Map<String, dynamic>? queryParameters,
    Options? options,
  }) async {
    return await _dio.get<T>(
      path,
      queryParameters: queryParameters,
      options: options,
    );
  }
  
  // POST请求
  Future<Response<T>> post<T>(
    String path, {
    dynamic data,
    Map<String, dynamic>? queryParameters,
    Options? options,
  }) async {
    return await _dio.post<T>(
      path,
      data: data,
      queryParameters: queryParameters,
      options: options,
    );
  }
  
  // PUT请求
  Future<Response<T>> put<T>(
    String path, {
    dynamic data,
    Map<String, dynamic>? queryParameters,
    Options? options,
  }) async {
    return await _dio.put<T>(
      path,
      data: data,
      queryParameters: queryParameters,
      options: options,
    );
  }
  
  // DELETE请求
  Future<Response<T>> delete<T>(
    String path, {
    dynamic data,
    Map<String, dynamic>? queryParameters,
    Options? options,
  }) async {
    return await _dio.delete<T>(
      path,
      data: data,
      queryParameters: queryParameters,
      options: options,
    );
  }
  
  // 文件上传
  Future<Response> uploadFile(
    String path,
    String filePath, {
    String? fileName,
    Map<String, dynamic>? data,
    ProgressCallback? onSendProgress,
  }) async {
    final formData = FormData.fromMap({
      'file': await MultipartFile.fromFile(
        filePath,
        filename: fileName,
      ),
      ...?data,
    });
    
    return await _dio.post(
      path,
      data: formData,
      onSendProgress: onSendProgress,
    );
  }
  
  // 错误处理
  void _handleError(DioException error) {
    switch (error.type) {
      case DioExceptionType.connectionTimeout:
        print('连接超时');
        break;
      case DioExceptionType.receiveTimeout:
        print('接收超时');
        break;
      case DioExceptionType.badResponse:
        print('响应错误: ${error.response?.statusCode}');
        break;
      case DioExceptionType.cancel:
        print('请求取消');
        break;
      default:
        print('网络错误: ${error.message}');
    }
  }
}

// 使用示例
class ApiExample {
  static Future<void> example() async {
    final apiClient = ApiClient.instance;
    apiClient.initialize(
      baseUrl: 'https://jsonplaceholder.typicode.com',
    );
    
    try {
      // GET请求
      final response = await apiClient.get('/posts/1');
      print('获取文章: ${response.data}');
      
      // POST请求
      final createResponse = await apiClient.post('/posts', data: {
        'title': '新文章',
        'body': '文章内容',
        'userId': 1,
      });
      print('创建文章: ${createResponse.data}');
      
    } catch (e) {
      print('请求失败: $e');
    }
  }
}
```

### 3. 数据库管理器

```dart
import 'package:sqflite/sqflite.dart';
import 'package:path/path.dart';

class DatabaseManager {
  static DatabaseManager? _instance;
  static DatabaseManager get instance => _instance ??= DatabaseManager._internal();
  
  DatabaseManager._internal();
  
  Database? _database;
  
  // 获取数据库实例
  Future<Database> get database async {
    _database ??= await _initDatabase();
    return _database!;
  }
  
  // 初始化数据库
  Future<Database> _initDatabase() async {
    final databasesPath = await getDatabasesPath();
    final path = join(databasesPath, 'app_database.db');
    
    return await openDatabase(
      path,
      version: 1,
      onCreate: _onCreate,
      onUpgrade: _onUpgrade,
    );
  }
  
  // 创建表
  Future<void> _onCreate(Database db, int version) async {
    await db.execute('''
      CREATE TABLE users (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        name TEXT NOT NULL,
        email TEXT UNIQUE NOT NULL,
        created_at TEXT NOT NULL
      )
    ''');
    
    await db.execute('''
      CREATE TABLE posts (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        user_id INTEGER NOT NULL,
        title TEXT NOT NULL,
        content TEXT NOT NULL,
        created_at TEXT NOT NULL,
        FOREIGN KEY (user_id) REFERENCES users (id)
      )
    ''');
  }
  
  // 升级数据库
  Future<void> _onUpgrade(Database db, int oldVersion, int newVersion) async {
    // 处理数据库升级逻辑
    if (oldVersion < 2) {
      await db.execute('ALTER TABLE users ADD COLUMN avatar TEXT');
    }
  }
  
  // 插入数据
  Future<int> insert(String table, Map<String, dynamic> data) async {
    final db = await database;
    return await db.insert(table, data);
  }
  
  // 查询数据
  Future<List<Map<String, dynamic>>> query(
    String table, {
    List<String>? columns,
    String? where,
    List<dynamic>? whereArgs,
    String? orderBy,
    int? limit,
    int? offset,
  }) async {
    final db = await database;
    return await db.query(
      table,
      columns: columns,
      where: where,
      whereArgs: whereArgs,
      orderBy: orderBy,
      limit: limit,
      offset: offset,
    );
  }
  
  // 更新数据
  Future<int> update(
    String table,
    Map<String, dynamic> data, {
    String? where,
    List<dynamic>? whereArgs,
  }) async {
    final db = await database;
    return await db.update(
      table,
      data,
      where: where,
      whereArgs: whereArgs,
    );
  }
  
  // 删除数据
  Future<int> delete(
    String table, {
    String? where,
    List<dynamic>? whereArgs,
  }) async {
    final db = await database;
    return await db.delete(
      table,
      where: where,
      whereArgs: whereArgs,
    );
  }
  
  // 执行原生SQL
  Future<List<Map<String, dynamic>>> rawQuery(
    String sql, [List<dynamic>? arguments]
  ) async {
    final db = await database;
    return await db.rawQuery(sql, arguments);
  }
  
  // 执行原生SQL（无返回值）
  Future<void> execute(String sql, [List<dynamic>? arguments]) async {
    final db = await database;
    await db.execute(sql, arguments);
  }
  
  // 事务处理
  Future<T> transaction<T>(Future<T> Function(Transaction txn) action) async {
    final db = await database;
    return await db.transaction(action);
  }
  
  // 关闭数据库
  Future<void> close() async {
    if (_database != null) {
      await _database!.close();
      _database = null;
    }
  }
}

// 使用示例
class DatabaseExample {
  static Future<void> example() async {
    final dbManager = DatabaseManager.instance;
    
    try {
      // 插入用户
      final userId = await dbManager.insert('users', {
        'name': '张三',
        'email': 'zhangsan@example.com',
        'created_at': DateTime.now().toIso8601String(),
      });
      print('插入用户ID: $userId');
      
      // 查询用户
      final users = await dbManager.query(
        'users',
        where: 'email = ?',
        whereArgs: ['zhangsan@example.com'],
      );
      print('查询用户: $users');
      
      // 更新用户
      final updateCount = await dbManager.update(
        'users',
        {'name': '李四'},
        where: 'id = ?',
        whereArgs: [userId],
      );
      print('更新用户数量: $updateCount');
      
      // 事务处理
      await dbManager.transaction((txn) async {
        await txn.insert('posts', {
          'user_id': userId,
          'title': '我的第一篇文章',
          'content': '这是文章内容',
          'created_at': DateTime.now().toIso8601String(),
        });
        
        await txn.insert('posts', {
          'user_id': userId,
          'title': '我的第二篇文章',
          'content': '这是另一篇文章内容',
          'created_at': DateTime.now().toIso8601String(),
        });
      });
      
      // 联表查询
      final postsWithUsers = await dbManager.rawQuery('''
        SELECT p.*, u.name as user_name, u.email as user_email
        FROM posts p
        JOIN users u ON p.user_id = u.id
        WHERE u.id = ?
      ''', [userId]);
      print('用户文章: $postsWithUsers');
      
    } catch (e) {
      print('数据库操作失败: $e');
    }
  }
}
```

### 4. 主题管理器

```dart
import 'package:flutter/material.dart';

class ThemeManager extends ChangeNotifier {
  static ThemeManager? _instance;
  static ThemeManager get instance => _instance ??= ThemeManager._internal();
  
  ThemeManager._internal() {
    _loadTheme();
  }
  
  ThemeMode _themeMode = ThemeMode.system;
  String _currentTheme = 'default';
  
  ThemeMode get themeMode => _themeMode;
  String get currentTheme => _currentTheme;
  
  // 预定义主题
  final Map<String, ThemeData> _themes = {
    'default': ThemeData(
      primarySwatch: Colors.blue,
      brightness: Brightness.light,
    ),
    'dark': ThemeData(
      primarySwatch: Colors.blue,
      brightness: Brightness.dark,
    ),
    'green': ThemeData(
      primarySwatch: Colors.green,
      brightness: Brightness.light,
    ),
    'purple': ThemeData(
      primarySwatch: Colors.purple,
      brightness: Brightness.light,
    ),
  };
  
  // 获取当前主题数据
  ThemeData get lightTheme {
    return _themes[_currentTheme] ?? _themes['default']!;
  }
  
  ThemeData get darkTheme {
    final baseTheme = _themes[_currentTheme] ?? _themes['default']!;
    return baseTheme.copyWith(
      brightness: Brightness.dark,
      colorScheme: baseTheme.colorScheme.copyWith(
        brightness: Brightness.dark,
      ),
    );
  }
  
  // 设置主题模式
  Future<void> setThemeMode(ThemeMode mode) async {
    if (_themeMode != mode) {
      _themeMode = mode;
      await _saveTheme();
      notifyListeners();
    }
  }
  
  // 设置主题
  Future<void> setTheme(String themeName) async {
    if (_themes.containsKey(themeName) && _currentTheme != themeName) {
      _currentTheme = themeName;
      await _saveTheme();
      notifyListeners();
    }
  }
  
  // 切换深色模式
  Future<void> toggleDarkMode() async {
    final newMode = _themeMode == ThemeMode.dark 
        ? ThemeMode.light 
        : ThemeMode.dark;
    await setThemeMode(newMode);
  }
  
  // 是否为深色模式
  bool isDarkMode(BuildContext context) {
    switch (_themeMode) {
      case ThemeMode.dark:
        return true;
      case ThemeMode.light:
        return false;
      case ThemeMode.system:
        return MediaQuery.of(context).platformBrightness == Brightness.dark;
    }
  }
  
  // 获取可用主题列表
  List<String> get availableThemes => _themes.keys.toList();
  
  // 添加自定义主题
  void addTheme(String name, ThemeData theme) {
    _themes[name] = theme;
    notifyListeners();
  }
  
  // 移除主题
  void removeTheme(String name) {
    if (name != 'default' && _themes.containsKey(name)) {
      _themes.remove(name);
      if (_currentTheme == name) {
        _currentTheme = 'default';
      }
      notifyListeners();
    }
  }
  
  // 加载主题设置
  Future<void> _loadTheme() async {
    final config = AppConfig.instance;
    
    final themeModeString = config.get<String>('theme_mode', defaultValue: 'system');
    _themeMode = ThemeMode.values.firstWhere(
      (mode) => mode.toString().split('.').last == themeModeString,
      orElse: () => ThemeMode.system,
    );
    
    _currentTheme = config.get<String>('current_theme', defaultValue: 'default');
  }
  
  // 保存主题设置
  Future<void> _saveTheme() async {
    final config = AppConfig.instance;
    
    await config.set('theme_mode', _themeMode.toString().split('.').last);
    await config.set('current_theme', _currentTheme);
  }
}

// 主题选择器Widget
class ThemeSelector extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return ListenableBuilder(
      listenable: ThemeManager.instance,
      builder: (context, child) {
        final themeManager = ThemeManager.instance;
        
        return Column(
          children: [
            // 主题模式选择
            ListTile(
              title: Text('主题模式'),
              trailing: DropdownButton<ThemeMode>(
                value: themeManager.themeMode,
                items: [
                  DropdownMenuItem(
                    value: ThemeMode.system,
                    child: Text('跟随系统'),
                  ),
                  DropdownMenuItem(
                    value: ThemeMode.light,
                    child: Text('浅色模式'),
                  ),
                  DropdownMenuItem(
                    value: ThemeMode.dark,
                    child: Text('深色模式'),
                  ),
                ],
                onChanged: (mode) {
                  if (mode != null) {
                    themeManager.setThemeMode(mode);
                  }
                },
              ),
            ),
            
            // 主题颜色选择
            ListTile(
              title: Text('主题颜色'),
              trailing: DropdownButton<String>(
                value: themeManager.currentTheme,
                items: themeManager.availableThemes.map((theme) {
                  return DropdownMenuItem(
                    value: theme,
                    child: Text(theme),
                  );
                }).toList(),
                onChanged: (theme) {
                  if (theme != null) {
                    themeManager.setTheme(theme);
                  }
                },
              ),
            ),
            
            // 深色模式开关
            SwitchListTile(
              title: Text('深色模式'),
              value: themeManager.isDarkMode(context),
              onChanged: (value) {
                themeManager.toggleDarkMode();
              },
            ),
          ],
        );
      },
    );
  }
}

// 使用示例
class ThemeExample extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return ListenableBuilder(
      listenable: ThemeManager.instance,
      builder: (context, child) {
        final themeManager = ThemeManager.instance;
        
        return MaterialApp(
          title: 'Theme Example',
          theme: themeManager.lightTheme,
          darkTheme: themeManager.darkTheme,
          themeMode: themeManager.themeMode,
          home: Scaffold(
            appBar: AppBar(
              title: Text('主题示例'),
            ),
            body: ThemeSelector(),
          ),
        );
      },
    );
  }
}
```

## 🛠 测试和调试

### 1. 单例测试

```dart
import 'package:flutter_test/flutter_test.dart';

void main() {
  group('Singleton Tests', () {
    test('应该返回相同的实例', () {
      final instance1 = AppConfig.instance;
      final instance2 = AppConfig.instance;
      
      expect(identical(instance1, instance2), true);
    });
    
    test('应该保持状态', () async {
      final config = AppConfig.instance;
      await config.initialize();
      
      await config.set('test_key', 'test_value');
      final value = config.get<String>('test_key');
      
      expect(value, 'test_value');
    });
    
    test('应该在多个访问中保持状态', () async {
      final config1 = AppConfig.instance;
      await config1.set('shared_key', 'shared_value');
      
      final config2 = AppConfig.instance;
      final value = config2.get<String>('shared_key');
      
      expect(value, 'shared_value');
    });
  });
}
```

### 2. 单例调试器

```dart
class SingletonDebugger {
  static final Map<Type, dynamic> _instances = {};
  static final Map<Type, int> _accessCounts = {};
  static final Map<Type, DateTime> _creationTimes = {};
  
  // 注册单例实例
  static void register<T>(T instance) {
    _instances[T] = instance;
    _accessCounts[T] = 0;
    _creationTimes[T] = DateTime.now();
  }
  
  // 记录访问
  static void recordAccess<T>() {
    _accessCounts[T] = (_accessCounts[T] ?? 0) + 1;
  }
  
  // 获取调试信息
  static Map<String, dynamic> getDebugInfo() {
    final info = <String, dynamic>{};
    
    for (final type in _instances.keys) {
      info[type.toString()] = {
        'instance': _instances[type],
        'accessCount': _accessCounts[type],
        'creationTime': _creationTimes[type],
        'memoryAddress': _instances[type].hashCode,
      };
    }
    
    return info;
  }
  
  // 打印调试信息
  static void printDebugInfo() {
    final info = getDebugInfo();
    print('=== Singleton Debug Info ===');
    
    for (final entry in info.entries) {
      print('${entry.key}:');
      print('  访问次数: ${entry.value['accessCount']}');
      print('  创建时间: ${entry.value['creationTime']}');
      print('  内存地址: ${entry.value['memoryAddress']}');
      print('');
    }
  }
}
```

## 🔒 最佳实践

### 1. 设计原则
- **延迟初始化**: 使用懒汉式单例，避免不必要的资源消耗
- **线程安全**: 在多线程环境下确保线程安全
- **接口隔离**: 提供清晰的公共接口
- **状态管理**: 合理管理单例的内部状态

### 2. 实现建议
- **私有构造函数**: 防止外部直接创建实例
- **静态访问点**: 提供全局访问点
- **异常处理**: 处理初始化过程中的异常
- **资源清理**: 提供资源清理方法

### 3. 使用注意事项
- **避免过度使用**: 不要滥用单例模式
- **测试友好**: 考虑测试时的依赖注入
- **内存泄漏**: 注意避免内存泄漏
- **状态污染**: 避免全局状态污染

### 4. 性能优化
- **懒加载**: 延迟初始化减少启动时间
- **缓存策略**: 合理使用缓存提高性能
- **资源管理**: 及时释放不需要的资源
- **内存监控**: 监控单例的内存使用情况

单例模式是Flutter开发中最常用的设计模式之一，正确使用可以有效管理全局状态和资源，但也要注意避免过度使用导致的代码耦合问题。