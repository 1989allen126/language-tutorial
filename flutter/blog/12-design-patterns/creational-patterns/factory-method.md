# 工厂方法模式 (Factory Method Pattern)

工厂方法模式定义了一个创建对象的接口，但让子类决定实例化哪一个类。在Flutter开发中，工厂方法模式常用于Widget创建、数据解析、服务实例化等场景。

## 📋 模式概述

### 定义
工厂方法模式是一种创建型设计模式，它提供了一种创建对象的接口，而无需指定它们具体的类。

### 适用场景
- Widget动态创建
- 数据模型解析
- 服务实例化
- 插件系统
- 主题组件创建
- 网络请求处理器

### 优点
- 解耦对象的创建和使用
- 符合开闭原则
- 符合单一职责原则
- 提供了良好的扩展性

### 缺点
- 增加了系统复杂性
- 需要创建更多的类
- 可能导致类层次结构复杂

## 🏗 基础实现

### 1. 简单工厂模式

```dart
// 产品接口
abstract class Button {
  void render();
  void onClick();
}

// 具体产品
class IOSButton implements Button {
  @override
  void render() {
    print('渲染iOS风格按钮');
  }
  
  @override
  void onClick() {
    print('iOS按钮点击事件');
  }
}

class AndroidButton implements Button {
  @override
  void render() {
    print('渲染Android风格按钮');
  }
  
  @override
  void onClick() {
    print('Android按钮点击事件');
  }
}

class WebButton implements Button {
  @override
  void render() {
    print('渲染Web风格按钮');
  }
  
  @override
  void onClick() {
    print('Web按钮点击事件');
  }
}

// 简单工厂
class ButtonFactory {
  static Button createButton(String platform) {
    switch (platform.toLowerCase()) {
      case 'ios':
        return IOSButton();
      case 'android':
        return AndroidButton();
      case 'web':
        return WebButton();
      default:
        throw ArgumentError('不支持的平台: $platform');
    }
  }
}

// 使用示例
void main() {
  final iosButton = ButtonFactory.createButton('ios');
  iosButton.render();
  iosButton.onClick();
  
  final androidButton = ButtonFactory.createButton('android');
  androidButton.render();
  androidButton.onClick();
}
```

### 2. 工厂方法模式

```dart
// 产品接口
abstract class Dialog {
  void show();
  void hide();
  Widget build();
}

// 具体产品
class AlertDialog extends Dialog {
  final String title;
  final String message;
  
  AlertDialog({required this.title, required this.message});
  
  @override
  void show() {
    print('显示警告对话框: $title');
  }
  
  @override
  void hide() {
    print('隐藏警告对话框');
  }
  
  @override
  Widget build() {
    return AlertDialog(
      title: Text(title),
      content: Text(message),
      actions: [
        TextButton(
          onPressed: () => hide(),
          child: Text('确定'),
        ),
      ],
    );
  }
}

class ConfirmDialog extends Dialog {
  final String title;
  final String message;
  final VoidCallback? onConfirm;
  final VoidCallback? onCancel;
  
  ConfirmDialog({
    required this.title,
    required this.message,
    this.onConfirm,
    this.onCancel,
  });
  
  @override
  void show() {
    print('显示确认对话框: $title');
  }
  
  @override
  void hide() {
    print('隐藏确认对话框');
  }
  
  @override
  Widget build() {
    return AlertDialog(
      title: Text(title),
      content: Text(message),
      actions: [
        TextButton(
          onPressed: () {
            onCancel?.call();
            hide();
          },
          child: Text('取消'),
        ),
        TextButton(
          onPressed: () {
            onConfirm?.call();
            hide();
          },
          child: Text('确定'),
        ),
      ],
    );
  }
}

// 创建者抽象类
abstract class DialogCreator {
  // 工厂方法
  Dialog createDialog();
  
  // 业务逻辑
  void showDialog() {
    final dialog = createDialog();
    dialog.show();
  }
}

// 具体创建者
class AlertDialogCreator extends DialogCreator {
  final String title;
  final String message;
  
  AlertDialogCreator({required this.title, required this.message});
  
  @override
  Dialog createDialog() {
    return AlertDialog(title: title, message: message);
  }
}

class ConfirmDialogCreator extends DialogCreator {
  final String title;
  final String message;
  final VoidCallback? onConfirm;
  final VoidCallback? onCancel;
  
  ConfirmDialogCreator({
    required this.title,
    required this.message,
    this.onConfirm,
    this.onCancel,
  });
  
  @override
  Dialog createDialog() {
    return ConfirmDialog(
      title: title,
      message: message,
      onConfirm: onConfirm,
      onCancel: onCancel,
    );
  }
}

// 使用示例
void main() {
  final alertCreator = AlertDialogCreator(
    title: '提示',
    message: '操作成功',
  );
  alertCreator.showDialog();
  
  final confirmCreator = ConfirmDialogCreator(
    title: '确认',
    message: '确定要删除吗？',
    onConfirm: () => print('确认删除'),
    onCancel: () => print('取消删除'),
  );
  confirmCreator.showDialog();
}
```

## 🎯 Flutter应用实例

### 1. Widget工厂

```dart
import 'package:flutter/material.dart';

// Widget配置
class WidgetConfig {
  final String type;
  final Map<String, dynamic> properties;
  
  WidgetConfig({required this.type, required this.properties});
}

// Widget工厂接口
abstract class WidgetFactory {
  Widget createWidget(WidgetConfig config);
  bool canCreate(String type);
}

// 按钮工厂
class ButtonWidgetFactory implements WidgetFactory {
  @override
  bool canCreate(String type) {
    return ['button', 'elevated_button', 'text_button', 'outlined_button']
        .contains(type.toLowerCase());
  }
  
  @override
  Widget createWidget(WidgetConfig config) {
    final props = config.properties;
    final text = props['text'] as String? ?? '';
    final onPressed = props['onPressed'] as VoidCallback?;
    
    switch (config.type.toLowerCase()) {
      case 'button':
      case 'elevated_button':
        return ElevatedButton(
          onPressed: onPressed,
          style: _getButtonStyle(props),
          child: Text(text),
        );
      case 'text_button':
        return TextButton(
          onPressed: onPressed,
          style: _getButtonStyle(props),
          child: Text(text),
        );
      case 'outlined_button':
        return OutlinedButton(
          onPressed: onPressed,
          style: _getButtonStyle(props),
          child: Text(text),
        );
      default:
        throw ArgumentError('不支持的按钮类型: ${config.type}');
    }
  }
  
  ButtonStyle? _getButtonStyle(Map<String, dynamic> props) {
    final backgroundColor = props['backgroundColor'] as Color?;
    final foregroundColor = props['foregroundColor'] as Color?;
    final padding = props['padding'] as EdgeInsets?;
    
    if (backgroundColor == null && foregroundColor == null && padding == null) {
      return null;
    }
    
    return ButtonStyle(
      backgroundColor: backgroundColor != null 
          ? MaterialStateProperty.all(backgroundColor) 
          : null,
      foregroundColor: foregroundColor != null 
          ? MaterialStateProperty.all(foregroundColor) 
          : null,
      padding: padding != null 
          ? MaterialStateProperty.all(padding) 
          : null,
    );
  }
}

// 输入框工厂
class InputWidgetFactory implements WidgetFactory {
  @override
  bool canCreate(String type) {
    return ['text_field', 'text_form_field'].contains(type.toLowerCase());
  }
  
  @override
  Widget createWidget(WidgetConfig config) {
    final props = config.properties;
    final controller = props['controller'] as TextEditingController?;
    final decoration = _getInputDecoration(props);
    final validator = props['validator'] as String? Function(String?)?;
    
    switch (config.type.toLowerCase()) {
      case 'text_field':
        return TextField(
          controller: controller,
          decoration: decoration,
          obscureText: props['obscureText'] as bool? ?? false,
          keyboardType: _getKeyboardType(props['keyboardType'] as String?),
        );
      case 'text_form_field':
        return TextFormField(
          controller: controller,
          decoration: decoration,
          validator: validator,
          obscureText: props['obscureText'] as bool? ?? false,
          keyboardType: _getKeyboardType(props['keyboardType'] as String?),
        );
      default:
        throw ArgumentError('不支持的输入框类型: ${config.type}');
    }
  }
  
  InputDecoration? _getInputDecoration(Map<String, dynamic> props) {
    final labelText = props['labelText'] as String?;
    final hintText = props['hintText'] as String?;
    final prefixIcon = props['prefixIcon'] as Widget?;
    final suffixIcon = props['suffixIcon'] as Widget?;
    
    if (labelText == null && hintText == null && 
        prefixIcon == null && suffixIcon == null) {
      return null;
    }
    
    return InputDecoration(
      labelText: labelText,
      hintText: hintText,
      prefixIcon: prefixIcon,
      suffixIcon: suffixIcon,
    );
  }
  
  TextInputType? _getKeyboardType(String? type) {
    switch (type?.toLowerCase()) {
      case 'number':
        return TextInputType.number;
      case 'email':
        return TextInputType.emailAddress;
      case 'phone':
        return TextInputType.phone;
      case 'url':
        return TextInputType.url;
      default:
        return null;
    }
  }
}

// Widget工厂管理器
class WidgetFactoryManager {
  static WidgetFactoryManager? _instance;
  static WidgetFactoryManager get instance => 
      _instance ??= WidgetFactoryManager._internal();
  
  WidgetFactoryManager._internal();
  
  final List<WidgetFactory> _factories = [];
  
  // 注册工厂
  void registerFactory(WidgetFactory factory) {
    _factories.add(factory);
  }
  
  // 创建Widget
  Widget createWidget(WidgetConfig config) {
    for (final factory in _factories) {
      if (factory.canCreate(config.type)) {
        return factory.createWidget(config);
      }
    }
    throw ArgumentError('没有找到支持类型 ${config.type} 的工厂');
  }
  
  // 检查是否支持某种类型
  bool canCreate(String type) {
    return _factories.any((factory) => factory.canCreate(type));
  }
  
  // 获取支持的类型列表
  List<String> getSupportedTypes() {
    final types = <String>[];
    // 这里需要各个工厂提供支持的类型列表
    return types;
  }
}

// 使用示例
class WidgetFactoryExample extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    // 初始化工厂管理器
    final manager = WidgetFactoryManager.instance;
    manager.registerFactory(ButtonWidgetFactory());
    manager.registerFactory(InputWidgetFactory());
    
    return Scaffold(
      appBar: AppBar(title: Text('Widget工厂示例')),
      body: Padding(
        padding: EdgeInsets.all(16),
        child: Column(
          children: [
            // 创建按钮
            manager.createWidget(WidgetConfig(
              type: 'elevated_button',
              properties: {
                'text': '提交',
                'backgroundColor': Colors.blue,
                'onPressed': () => print('提交按钮点击'),
              },
            )),
            
            SizedBox(height: 16),
            
            // 创建输入框
            manager.createWidget(WidgetConfig(
              type: 'text_form_field',
              properties: {
                'labelText': '用户名',
                'hintText': '请输入用户名',
                'prefixIcon': Icon(Icons.person),
                'validator': (String? value) {
                  if (value == null || value.isEmpty) {
                    return '请输入用户名';
                  }
                  return null;
                },
              },
            )),
            
            SizedBox(height: 16),
            
            // 创建文本按钮
            manager.createWidget(WidgetConfig(
              type: 'text_button',
              properties: {
                'text': '取消',
                'foregroundColor': Colors.red,
                'onPressed': () => print('取消按钮点击'),
              },
            )),
          ],
        ),
      ),
    );
  }
}
```

### 2. 数据解析工厂

```dart
// 数据模型基类
abstract class DataModel {
  Map<String, dynamic> toJson();
  String get type;
}

// 用户模型
class User implements DataModel {
  final String id;
  final String name;
  final String email;
  final DateTime createdAt;
  
  User({
    required this.id,
    required this.name,
    required this.email,
    required this.createdAt,
  });
  
  factory User.fromJson(Map<String, dynamic> json) {
    return User(
      id: json['id'] as String,
      name: json['name'] as String,
      email: json['email'] as String,
      createdAt: DateTime.parse(json['created_at'] as String),
    );
  }
  
  @override
  Map<String, dynamic> toJson() {
    return {
      'id': id,
      'name': name,
      'email': email,
      'created_at': createdAt.toIso8601String(),
      'type': type,
    };
  }
  
  @override
  String get type => 'user';
}

// 文章模型
class Article implements DataModel {
  final String id;
  final String title;
  final String content;
  final String authorId;
  final DateTime publishedAt;
  
  Article({
    required this.id,
    required this.title,
    required this.content,
    required this.authorId,
    required this.publishedAt,
  });
  
  factory Article.fromJson(Map<String, dynamic> json) {
    return Article(
      id: json['id'] as String,
      title: json['title'] as String,
      content: json['content'] as String,
      authorId: json['author_id'] as String,
      publishedAt: DateTime.parse(json['published_at'] as String),
    );
  }
  
  @override
  Map<String, dynamic> toJson() {
    return {
      'id': id,
      'title': title,
      'content': content,
      'author_id': authorId,
      'published_at': publishedAt.toIso8601String(),
      'type': type,
    };
  }
  
  @override
  String get type => 'article';
}

// 评论模型
class Comment implements DataModel {
  final String id;
  final String content;
  final String authorId;
  final String articleId;
  final DateTime createdAt;
  
  Comment({
    required this.id,
    required this.content,
    required this.authorId,
    required this.articleId,
    required this.createdAt,
  });
  
  factory Comment.fromJson(Map<String, dynamic> json) {
    return Comment(
      id: json['id'] as String,
      content: json['content'] as String,
      authorId: json['author_id'] as String,
      articleId: json['article_id'] as String,
      createdAt: DateTime.parse(json['created_at'] as String),
    );
  }
  
  @override
  Map<String, dynamic> toJson() {
    return {
      'id': id,
      'content': content,
      'author_id': authorId,
      'article_id': articleId,
      'created_at': createdAt.toIso8601String(),
      'type': type,
    };
  }
  
  @override
  String get type => 'comment';
}

// 数据解析工厂接口
abstract class DataParserFactory {
  DataModel parseFromJson(Map<String, dynamic> json);
  bool canParse(String type);
  String get supportedType;
}

// 用户解析工厂
class UserParserFactory implements DataParserFactory {
  @override
  String get supportedType => 'user';
  
  @override
  bool canParse(String type) => type == supportedType;
  
  @override
  DataModel parseFromJson(Map<String, dynamic> json) {
    return User.fromJson(json);
  }
}

// 文章解析工厂
class ArticleParserFactory implements DataParserFactory {
  @override
  String get supportedType => 'article';
  
  @override
  bool canParse(String type) => type == supportedType;
  
  @override
  DataModel parseFromJson(Map<String, dynamic> json) {
    return Article.fromJson(json);
  }
}

// 评论解析工厂
class CommentParserFactory implements DataParserFactory {
  @override
  String get supportedType => 'comment';
  
  @override
  bool canParse(String type) => type == supportedType;
  
  @override
  DataModel parseFromJson(Map<String, dynamic> json) {
    return Comment.fromJson(json);
  }
}

// 数据解析管理器
class DataParserManager {
  static DataParserManager? _instance;
  static DataParserManager get instance => 
      _instance ??= DataParserManager._internal();
  
  DataParserManager._internal() {
    // 注册默认解析器
    registerFactory(UserParserFactory());
    registerFactory(ArticleParserFactory());
    registerFactory(CommentParserFactory());
  }
  
  final Map<String, DataParserFactory> _factories = {};
  
  // 注册解析工厂
  void registerFactory(DataParserFactory factory) {
    _factories[factory.supportedType] = factory;
  }
  
  // 解析单个对象
  DataModel parseFromJson(Map<String, dynamic> json) {
    final type = json['type'] as String?;
    if (type == null) {
      throw ArgumentError('JSON数据缺少type字段');
    }
    
    final factory = _factories[type];
    if (factory == null) {
      throw ArgumentError('不支持的数据类型: $type');
    }
    
    return factory.parseFromJson(json);
  }
  
  // 解析对象列表
  List<DataModel> parseListFromJson(List<dynamic> jsonList) {
    return jsonList
        .cast<Map<String, dynamic>>()
        .map((json) => parseFromJson(json))
        .toList();
  }
  
  // 解析混合类型响应
  List<DataModel> parseMixedResponse(Map<String, dynamic> response) {
    final results = <DataModel>[];
    
    // 解析用户列表
    final users = response['users'] as List<dynamic>?;
    if (users != null) {
      results.addAll(users
          .cast<Map<String, dynamic>>()
          .map((json) => User.fromJson(json)));
    }
    
    // 解析文章列表
    final articles = response['articles'] as List<dynamic>?;
    if (articles != null) {
      results.addAll(articles
          .cast<Map<String, dynamic>>()
          .map((json) => Article.fromJson(json)));
    }
    
    // 解析评论列表
    final comments = response['comments'] as List<dynamic>?;
    if (comments != null) {
      results.addAll(comments
          .cast<Map<String, dynamic>>()
          .map((json) => Comment.fromJson(json)));
    }
    
    return results;
  }
  
  // 检查是否支持某种类型
  bool canParse(String type) {
    return _factories.containsKey(type);
  }
  
  // 获取支持的类型列表
  List<String> getSupportedTypes() {
    return _factories.keys.toList();
  }
}

// 使用示例
class DataParserExample {
  static void example() {
    final parser = DataParserManager.instance;
    
    // 解析用户数据
    final userJson = {
      'id': '1',
      'name': '张三',
      'email': 'zhangsan@example.com',
      'created_at': '2023-01-01T00:00:00Z',
      'type': 'user',
    };
    
    final user = parser.parseFromJson(userJson) as User;
    print('解析用户: ${user.name} (${user.email})');
    
    // 解析文章数据
    final articleJson = {
      'id': '1',
      'title': 'Flutter设计模式',
      'content': '这是一篇关于Flutter设计模式的文章',
      'author_id': '1',
      'published_at': '2023-01-02T00:00:00Z',
      'type': 'article',
    };
    
    final article = parser.parseFromJson(articleJson) as Article;
    print('解析文章: ${article.title}');
    
    // 解析混合数据
    final mixedResponse = {
      'users': [userJson],
      'articles': [articleJson],
      'comments': [
        {
          'id': '1',
          'content': '很好的文章！',
          'author_id': '1',
          'article_id': '1',
          'created_at': '2023-01-03T00:00:00Z',
          'type': 'comment',
        }
      ],
    };
    
    final results = parser.parseMixedResponse(mixedResponse);
    print('解析混合数据，共 ${results.length} 个对象');
    
    for (final result in results) {
      print('- ${result.type}: ${result.toJson()}');
    }
  }
}
```

### 3. 服务工厂

```dart
// 服务接口
abstract class Service {
  Future<void> initialize();
  Future<void> dispose();
  String get name;
}

// 认证服务
class AuthService implements Service {
  @override
  String get name => 'AuthService';
  
  String? _token;
  
  @override
  Future<void> initialize() async {
    print('初始化认证服务');
    // 从本地存储加载token
    _token = await _loadTokenFromStorage();
  }
  
  @override
  Future<void> dispose() async {
    print('销毁认证服务');
    _token = null;
  }
  
  Future<String?> _loadTokenFromStorage() async {
    // 模拟从本地存储加载token
    await Future.delayed(Duration(milliseconds: 100));
    return 'mock_token_123';
  }
  
  Future<bool> login(String username, String password) async {
    // 模拟登录
    await Future.delayed(Duration(seconds: 1));
    _token = 'new_token_${DateTime.now().millisecondsSinceEpoch}';
    return true;
  }
  
  Future<void> logout() async {
    _token = null;
  }
  
  bool get isLoggedIn => _token != null;
  String? get token => _token;
}

// 数据服务
class DataService implements Service {
  @override
  String get name => 'DataService';
  
  late Database _database;
  
  @override
  Future<void> initialize() async {
    print('初始化数据服务');
    // 初始化数据库连接
    _database = await _initDatabase();
  }
  
  @override
  Future<void> dispose() async {
    print('销毁数据服务');
    await _database.close();
  }
  
  Future<Database> _initDatabase() async {
    // 模拟数据库初始化
    await Future.delayed(Duration(milliseconds: 200));
    return Database(); // 假设的数据库类
  }
  
  Future<List<Map<String, dynamic>>> query(String sql) async {
    // 模拟数据库查询
    await Future.delayed(Duration(milliseconds: 50));
    return [];
  }
}

// 网络服务
class NetworkService implements Service {
  @override
  String get name => 'NetworkService';
  
  late Dio _dio;
  
  @override
  Future<void> initialize() async {
    print('初始化网络服务');
    _dio = Dio(BaseOptions(
      baseUrl: 'https://api.example.com',
      connectTimeout: Duration(seconds: 30),
      receiveTimeout: Duration(seconds: 30),
    ));
  }
  
  @override
  Future<void> dispose() async {
    print('销毁网络服务');
    _dio.close();
  }
  
  Future<Response> get(String path) async {
    return await _dio.get(path);
  }
  
  Future<Response> post(String path, {dynamic data}) async {
    return await _dio.post(path, data: data);
  }
}

// 服务工厂接口
abstract class ServiceFactory {
  Service createService();
  String get serviceType;
}

// 认证服务工厂
class AuthServiceFactory implements ServiceFactory {
  @override
  String get serviceType => 'auth';
  
  @override
  Service createService() {
    return AuthService();
  }
}

// 数据服务工厂
class DataServiceFactory implements ServiceFactory {
  @override
  String get serviceType => 'data';
  
  @override
  Service createService() {
    return DataService();
  }
}

// 网络服务工厂
class NetworkServiceFactory implements ServiceFactory {
  @override
  String get serviceType => 'network';
  
  @override
  Service createService() {
    return NetworkService();
  }
}

// 服务管理器
class ServiceManager {
  static ServiceManager? _instance;
  static ServiceManager get instance => 
      _instance ??= ServiceManager._internal();
  
  ServiceManager._internal();
  
  final Map<String, ServiceFactory> _factories = {};
  final Map<String, Service> _services = {};
  
  // 注册服务工厂
  void registerFactory(ServiceFactory factory) {
    _factories[factory.serviceType] = factory;
  }
  
  // 获取服务实例
  T getService<T extends Service>(String type) {
    // 如果服务已存在，直接返回
    if (_services.containsKey(type)) {
      return _services[type]! as T;
    }
    
    // 创建新的服务实例
    final factory = _factories[type];
    if (factory == null) {
      throw ArgumentError('未注册的服务类型: $type');
    }
    
    final service = factory.createService();
    _services[type] = service;
    
    return service as T;
  }
  
  // 初始化所有服务
  Future<void> initializeAll() async {
    for (final service in _services.values) {
      await service.initialize();
    }
  }
  
  // 初始化指定服务
  Future<void> initializeService(String type) async {
    final service = getService(type);
    await service.initialize();
  }
  
  // 销毁所有服务
  Future<void> disposeAll() async {
    for (final service in _services.values) {
      await service.dispose();
    }
    _services.clear();
  }
  
  // 销毁指定服务
  Future<void> disposeService(String type) async {
    final service = _services[type];
    if (service != null) {
      await service.dispose();
      _services.remove(type);
    }
  }
  
  // 检查服务是否已注册
  bool isRegistered(String type) {
    return _factories.containsKey(type);
  }
  
  // 检查服务是否已创建
  bool isCreated(String type) {
    return _services.containsKey(type);
  }
  
  // 获取所有已注册的服务类型
  List<String> getRegisteredTypes() {
    return _factories.keys.toList();
  }
  
  // 获取所有已创建的服务
  List<Service> getCreatedServices() {
    return _services.values.toList();
  }
}

// 使用示例
class ServiceExample {
  static Future<void> example() async {
    final serviceManager = ServiceManager.instance;
    
    // 注册服务工厂
    serviceManager.registerFactory(AuthServiceFactory());
    serviceManager.registerFactory(DataServiceFactory());
    serviceManager.registerFactory(NetworkServiceFactory());
    
    // 获取服务实例
    final authService = serviceManager.getService<AuthService>('auth');
    final dataService = serviceManager.getService<DataService>('data');
    final networkService = serviceManager.getService<NetworkService>('network');
    
    // 初始化服务
    await serviceManager.initializeAll();
    
    // 使用服务
    final loginSuccess = await authService.login('user', 'password');
    print('登录结果: $loginSuccess');
    
    if (authService.isLoggedIn) {
      print('用户已登录，token: ${authService.token}');
    }
    
    // 查询数据
    final data = await dataService.query('SELECT * FROM users');
    print('查询结果: ${data.length} 条记录');
    
    // 网络请求
    try {
      final response = await networkService.get('/users');
      print('网络请求成功: ${response.statusCode}');
    } catch (e) {
      print('网络请求失败: $e');
    }
    
    // 销毁服务
    await serviceManager.disposeAll();
  }
}

// 假设的数据库类
class Database {
  Future<void> close() async {
    await Future.delayed(Duration(milliseconds: 50));
  }
}
```

## 🛠 测试和调试

### 1. 工厂测试

```dart
import 'package:flutter_test/flutter_test.dart';

void main() {
  group('Factory Method Tests', () {
    test('ButtonFactory应该创建正确的按钮类型', () {
      final iosButton = ButtonFactory.createButton('ios');
      expect(iosButton, isA<IOSButton>());
      
      final androidButton = ButtonFactory.createButton('android');
      expect(androidButton, isA<AndroidButton>());
      
      final webButton = ButtonFactory.createButton('web');
      expect(webButton, isA<WebButton>());
    });
    
    test('ButtonFactory应该抛出异常对于不支持的类型', () {
      expect(
        () => ButtonFactory.createButton('unknown'),
        throwsArgumentError,
      );
    });
    
    test('WidgetFactoryManager应该正确注册和创建Widget', () {
      final manager = WidgetFactoryManager.instance;
      manager.registerFactory(ButtonWidgetFactory());
      
      expect(manager.canCreate('button'), true);
      expect(manager.canCreate('unknown'), false);
      
      final widget = manager.createWidget(WidgetConfig(
        type: 'button',
        properties: {'text': 'Test'},
      ));
      
      expect(widget, isA<ElevatedButton>());
    });
  });
}
```

### 2. 工厂调试器

```dart
class FactoryDebugger {
  static final Map<String, int> _creationCounts = {};
  static final Map<String, List<DateTime>> _creationTimes = {};
  
  // 记录对象创建
  static void recordCreation(String type) {
    _creationCounts[type] = (_creationCounts[type] ?? 0) + 1;
    _creationTimes.putIfAbsent(type, () => []).add(DateTime.now());
  }
  
  // 获取创建统计
  static Map<String, dynamic> getCreationStats() {
    final stats = <String, dynamic>{};
    
    for (final type in _creationCounts.keys) {
      final count = _creationCounts[type]!;
      final times = _creationTimes[type]!;
      
      stats[type] = {
        'count': count,
        'firstCreation': times.first,
        'lastCreation': times.last,
        'averageInterval': count > 1 
            ? times.last.difference(times.first).inMilliseconds / (count - 1)
            : 0,
      };
    }
    
    return stats;
  }
  
  // 打印调试信息
  static void printDebugInfo() {
    final stats = getCreationStats();
    print('=== Factory Debug Info ===');
    
    for (final entry in stats.entries) {
      print('${entry.key}:');
      print('  创建次数: ${entry.value['count']}');
      print('  首次创建: ${entry.value['firstCreation']}');
      print('  最后创建: ${entry.value['lastCreation']}');
      print('  平均间隔: ${entry.value['averageInterval']}ms');
      print('');
    }
  }
  
  // 重置统计
  static void reset() {
    _creationCounts.clear();
    _creationTimes.clear();
  }
}
```

## 🔒 最佳实践

### 1. 设计原则
- **单一职责**: 每个工厂只负责创建一种类型的对象
- **开闭原则**: 对扩展开放，对修改关闭
- **依赖倒置**: 依赖抽象而不是具体实现
- **接口隔离**: 提供清晰的工厂接口

### 2. 实现建议
- **类型安全**: 使用泛型确保类型安全
- **异常处理**: 处理创建过程中的异常
- **参数验证**: 验证创建参数的有效性
- **文档注释**: 提供清晰的文档说明

### 3. 性能优化
- **对象池**: 复用对象减少创建开销
- **懒加载**: 延迟创建减少内存占用
- **缓存策略**: 缓存常用对象
- **批量创建**: 支持批量创建优化性能

### 4. 扩展性考虑
- **插件架构**: 支持动态注册工厂
- **配置驱动**: 通过配置文件驱动对象创建
- **版本兼容**: 考虑向后兼容性
- **国际化**: 支持多语言和地区

工厂方法模式是Flutter开发中非常实用的设计模式，它提供了灵活的对象创建机制，有助于构建可扩展和可维护的应用程序。