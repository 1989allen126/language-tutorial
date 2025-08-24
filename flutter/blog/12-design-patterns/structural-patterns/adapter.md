# 适配器模式 (Adapter Pattern)

适配器模式允许接口不兼容的类一起工作，它将一个类的接口转换成客户希望的另一个接口。在Flutter开发中，适配器模式常用于第三方库集成、数据格式转换、平台适配等场景。

## 📋 模式概述

### 定义
适配器模式是一种结构型设计模式，它允许将一个类的接口转换成另一个接口，使得原本由于接口不兼容而不能一起工作的类可以协同工作。

### 适用场景
- 第三方库接口适配
- 数据格式转换
- 平台特定功能适配
- 旧系统与新系统集成
- API版本兼容
- 数据源适配

### 优点
- 提高代码复用性
- 分离接口和实现
- 符合开闭原则
- 提供良好的扩展性

### 缺点
- 增加系统复杂性
- 可能影响性能
- 需要额外的适配器类

## 🏗 基础实现

### 1. 对象适配器

```dart
// 目标接口 - 客户期望的接口
abstract class MediaPlayer {
  void play(String audioType, String fileName);
}

// 被适配者 - 需要适配的类
class AdvancedMediaPlayer {
  void playVlc(String fileName) {
    print('播放VLC文件: $fileName');
  }
  
  void playMp4(String fileName) {
    print('播放MP4文件: $fileName');
  }
}

// 适配器类
class MediaAdapter implements MediaPlayer {
  final AdvancedMediaPlayer _advancedPlayer;
  
  MediaAdapter() : _advancedPlayer = AdvancedMediaPlayer();
  
  @override
  void play(String audioType, String fileName) {
    switch (audioType.toLowerCase()) {
      case 'vlc':
        _advancedPlayer.playVlc(fileName);
        break;
      case 'mp4':
        _advancedPlayer.playMp4(fileName);
        break;
      default:
        throw ArgumentError('不支持的音频格式: $audioType');
    }
  }
}

// 客户端类
class AudioPlayer implements MediaPlayer {
  MediaAdapter? _mediaAdapter;
  
  @override
  void play(String audioType, String fileName) {
    // 内置支持MP3格式
    if (audioType.toLowerCase() == 'mp3') {
      print('播放MP3文件: $fileName');
    }
    // 使用适配器支持其他格式
    else if (['vlc', 'mp4'].contains(audioType.toLowerCase())) {
      _mediaAdapter ??= MediaAdapter();
      _mediaAdapter!.play(audioType, fileName);
    }
    else {
      print('不支持的音频格式: $audioType');
    }
  }
}

// 使用示例
void main() {
  final player = AudioPlayer();
  
  player.play('mp3', 'song.mp3');
  player.play('vlc', 'movie.vlc');
  player.play('mp4', 'video.mp4');
  player.play('avi', 'clip.avi'); // 不支持的格式
}
```

### 2. 类适配器（使用Mixin）

```dart
// 目标接口
abstract class Target {
  String request();
}

// 被适配者
class Adaptee {
  String specificRequest() {
    return '特殊请求的结果';
  }
}

// 适配器Mixin
mixin AdapterMixin on Adaptee implements Target {
  @override
  String request() {
    return '适配器: ${specificRequest()}';
  }
}

// 具体适配器
class Adapter extends Adaptee with AdapterMixin {}

// 使用示例
void main() {
  final adapter = Adapter();
  print(adapter.request()); // 输出: 适配器: 特殊请求的结果
}
```

## 🎯 Flutter应用实例

### 1. 第三方库适配器

```dart
import 'package:dio/dio.dart';
import 'package:http/http.dart' as http;

// 统一的HTTP客户端接口
abstract class HttpClient {
  Future<HttpResponse> get(String url, {Map<String, String>? headers});
  Future<HttpResponse> post(String url, {dynamic body, Map<String, String>? headers});
  Future<HttpResponse> put(String url, {dynamic body, Map<String, String>? headers});
  Future<HttpResponse> delete(String url, {Map<String, String>? headers});
}

// 统一的HTTP响应
class HttpResponse {
  final int statusCode;
  final String body;
  final Map<String, String> headers;
  
  HttpResponse({
    required this.statusCode,
    required this.body,
    required this.headers,
  });
  
  bool get isSuccess => statusCode >= 200 && statusCode < 300;
}

// Dio适配器
class DioAdapter implements HttpClient {
  final Dio _dio;
  
  DioAdapter({Dio? dio}) : _dio = dio ?? Dio();
  
  @override
  Future<HttpResponse> get(String url, {Map<String, String>? headers}) async {
    try {
      final response = await _dio.get(
        url,
        options: Options(headers: headers),
      );
      return _convertResponse(response);
    } on DioException catch (e) {
      throw _convertError(e);
    }
  }
  
  @override
  Future<HttpResponse> post(String url, {dynamic body, Map<String, String>? headers}) async {
    try {
      final response = await _dio.post(
        url,
        data: body,
        options: Options(headers: headers),
      );
      return _convertResponse(response);
    } on DioException catch (e) {
      throw _convertError(e);
    }
  }
  
  @override
  Future<HttpResponse> put(String url, {dynamic body, Map<String, String>? headers}) async {
    try {
      final response = await _dio.put(
        url,
        data: body,
        options: Options(headers: headers),
      );
      return _convertResponse(response);
    } on DioException catch (e) {
      throw _convertError(e);
    }
  }
  
  @override
  Future<HttpResponse> delete(String url, {Map<String, String>? headers}) async {
    try {
      final response = await _dio.delete(
        url,
        options: Options(headers: headers),
      );
      return _convertResponse(response);
    } on DioException catch (e) {
      throw _convertError(e);
    }
  }
  
  HttpResponse _convertResponse(Response response) {
    return HttpResponse(
      statusCode: response.statusCode ?? 0,
      body: response.data?.toString() ?? '',
      headers: response.headers.map.map(
        (key, value) => MapEntry(key, value.join(', ')),
      ),
    );
  }
  
  Exception _convertError(DioException error) {
    return Exception('Dio错误: ${error.message}');
  }
}

// HTTP包适配器
class HttpPackageAdapter implements HttpClient {
  final http.Client _client;
  
  HttpPackageAdapter({http.Client? client}) : _client = client ?? http.Client();
  
  @override
  Future<HttpResponse> get(String url, {Map<String, String>? headers}) async {
    try {
      final response = await _client.get(
        Uri.parse(url),
        headers: headers,
      );
      return _convertResponse(response);
    } catch (e) {
      throw Exception('HTTP错误: $e');
    }
  }
  
  @override
  Future<HttpResponse> post(String url, {dynamic body, Map<String, String>? headers}) async {
    try {
      final response = await _client.post(
        Uri.parse(url),
        body: body,
        headers: headers,
      );
      return _convertResponse(response);
    } catch (e) {
      throw Exception('HTTP错误: $e');
    }
  }
  
  @override
  Future<HttpResponse> put(String url, {dynamic body, Map<String, String>? headers}) async {
    try {
      final response = await _client.put(
        Uri.parse(url),
        body: body,
        headers: headers,
      );
      return _convertResponse(response);
    } catch (e) {
      throw Exception('HTTP错误: $e');
    }
  }
  
  @override
  Future<HttpResponse> delete(String url, {Map<String, String>? headers}) async {
    try {
      final response = await _client.delete(
        Uri.parse(url),
        headers: headers,
      );
      return _convertResponse(response);
    } catch (e) {
      throw Exception('HTTP错误: $e');
    }
  }
  
  HttpResponse _convertResponse(http.Response response) {
    return HttpResponse(
      statusCode: response.statusCode,
      body: response.body,
      headers: response.headers,
    );
  }
}

// HTTP客户端工厂
class HttpClientFactory {
  static HttpClient create(String type) {
    switch (type.toLowerCase()) {
      case 'dio':
        return DioAdapter();
      case 'http':
        return HttpPackageAdapter();
      default:
        throw ArgumentError('不支持的HTTP客户端类型: $type');
    }
  }
}

// 使用示例
class HttpAdapterExample {
  static Future<void> example() async {
    // 使用Dio适配器
    final dioClient = HttpClientFactory.create('dio');
    
    try {
      final response = await dioClient.get('https://jsonplaceholder.typicode.com/posts/1');
      print('Dio响应状态: ${response.statusCode}');
      print('Dio响应内容: ${response.body}');
    } catch (e) {
      print('Dio请求失败: $e');
    }
    
    // 使用HTTP包适配器
    final httpClient = HttpClientFactory.create('http');
    
    try {
      final response = await httpClient.get('https://jsonplaceholder.typicode.com/posts/2');
      print('HTTP响应状态: ${response.statusCode}');
      print('HTTP响应内容: ${response.body}');
    } catch (e) {
      print('HTTP请求失败: $e');
    }
  }
}
```

### 2. 数据格式适配器

```dart
import 'dart:convert';

// 统一的数据模型
class User {
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
  
  @override
  String toString() {
    return 'User(id: $id, name: $name, email: $email, createdAt: $createdAt)';
  }
}

// 数据源接口
abstract class DataSource {
  Future<List<User>> getUsers();
  Future<User?> getUser(String id);
  Future<User> createUser(User user);
  Future<User> updateUser(User user);
  Future<void> deleteUser(String id);
}

// REST API数据源（被适配者）
class RestApiDataSource {
  final HttpClient _httpClient;
  final String _baseUrl;
  
  RestApiDataSource(this._httpClient, this._baseUrl);
  
  Future<Map<String, dynamic>> getUsers() async {
    final response = await _httpClient.get('$_baseUrl/users');
    if (response.isSuccess) {
      return json.decode(response.body);
    }
    throw Exception('获取用户列表失败: ${response.statusCode}');
  }
  
  Future<Map<String, dynamic>?> getUser(String id) async {
    final response = await _httpClient.get('$_baseUrl/users/$id');
    if (response.isSuccess) {
      return json.decode(response.body);
    }
    if (response.statusCode == 404) {
      return null;
    }
    throw Exception('获取用户失败: ${response.statusCode}');
  }
  
  Future<Map<String, dynamic>> createUser(Map<String, dynamic> userData) async {
    final response = await _httpClient.post(
      '$_baseUrl/users',
      body: json.encode(userData),
      headers: {'Content-Type': 'application/json'},
    );
    if (response.isSuccess) {
      return json.decode(response.body);
    }
    throw Exception('创建用户失败: ${response.statusCode}');
  }
  
  Future<Map<String, dynamic>> updateUser(String id, Map<String, dynamic> userData) async {
    final response = await _httpClient.put(
      '$_baseUrl/users/$id',
      body: json.encode(userData),
      headers: {'Content-Type': 'application/json'},
    );
    if (response.isSuccess) {
      return json.decode(response.body);
    }
    throw Exception('更新用户失败: ${response.statusCode}');
  }
  
  Future<void> deleteUser(String id) async {
    final response = await _httpClient.delete('$_baseUrl/users/$id');
    if (!response.isSuccess) {
      throw Exception('删除用户失败: ${response.statusCode}');
    }
  }
}

// GraphQL数据源（被适配者）
class GraphQLDataSource {
  final HttpClient _httpClient;
  final String _endpoint;
  
  GraphQLDataSource(this._httpClient, this._endpoint);
  
  Future<Map<String, dynamic>> query(String query, {Map<String, dynamic>? variables}) async {
    final response = await _httpClient.post(
      _endpoint,
      body: json.encode({
        'query': query,
        'variables': variables,
      }),
      headers: {'Content-Type': 'application/json'},
    );
    
    if (response.isSuccess) {
      final data = json.decode(response.body);
      if (data['errors'] != null) {
        throw Exception('GraphQL错误: ${data['errors']}');
      }
      return data['data'];
    }
    throw Exception('GraphQL请求失败: ${response.statusCode}');
  }
  
  Future<Map<String, dynamic>> mutate(String mutation, {Map<String, dynamic>? variables}) async {
    return await query(mutation, variables: variables);
  }
}

// REST API适配器
class RestApiAdapter implements DataSource {
  final RestApiDataSource _restApi;
  
  RestApiAdapter(this._restApi);
  
  @override
  Future<List<User>> getUsers() async {
    final data = await _restApi.getUsers();
    final userList = data['users'] as List<dynamic>? ?? data as List<dynamic>;
    return userList.map((userData) => _convertToUser(userData)).toList();
  }
  
  @override
  Future<User?> getUser(String id) async {
    final userData = await _restApi.getUser(id);
    return userData != null ? _convertToUser(userData) : null;
  }
  
  @override
  Future<User> createUser(User user) async {
    final userData = _convertFromUser(user);
    final result = await _restApi.createUser(userData);
    return _convertToUser(result);
  }
  
  @override
  Future<User> updateUser(User user) async {
    final userData = _convertFromUser(user);
    final result = await _restApi.updateUser(user.id, userData);
    return _convertToUser(result);
  }
  
  @override
  Future<void> deleteUser(String id) async {
    await _restApi.deleteUser(id);
  }
  
  User _convertToUser(Map<String, dynamic> data) {
    return User(
      id: data['id']?.toString() ?? '',
      name: data['name']?.toString() ?? '',
      email: data['email']?.toString() ?? '',
      createdAt: DateTime.tryParse(data['created_at']?.toString() ?? '') ?? DateTime.now(),
    );
  }
  
  Map<String, dynamic> _convertFromUser(User user) {
    return {
      'id': user.id,
      'name': user.name,
      'email': user.email,
      'created_at': user.createdAt.toIso8601String(),
    };
  }
}

// GraphQL适配器
class GraphQLAdapter implements DataSource {
  final GraphQLDataSource _graphQL;
  
  GraphQLAdapter(this._graphQL);
  
  @override
  Future<List<User>> getUsers() async {
    const query = '''
      query GetUsers {
        users {
          id
          name
          email
          createdAt
        }
      }
    ''';
    
    final data = await _graphQL.query(query);
    final userList = data['users'] as List<dynamic>;
    return userList.map((userData) => _convertToUser(userData)).toList();
  }
  
  @override
  Future<User?> getUser(String id) async {
    const query = '''
      query GetUser(\$id: ID!) {
        user(id: \$id) {
          id
          name
          email
          createdAt
        }
      }
    ''';
    
    final data = await _graphQL.query(query, variables: {'id': id});
    final userData = data['user'];
    return userData != null ? _convertToUser(userData) : null;
  }
  
  @override
  Future<User> createUser(User user) async {
    const mutation = '''
      mutation CreateUser(\$input: UserInput!) {
        createUser(input: \$input) {
          id
          name
          email
          createdAt
        }
      }
    ''';
    
    final input = {
      'name': user.name,
      'email': user.email,
    };
    
    final data = await _graphQL.mutate(mutation, variables: {'input': input});
    return _convertToUser(data['createUser']);
  }
  
  @override
  Future<User> updateUser(User user) async {
    const mutation = '''
      mutation UpdateUser(\$id: ID!, \$input: UserInput!) {
        updateUser(id: \$id, input: \$input) {
          id
          name
          email
          createdAt
        }
      }
    ''';
    
    final input = {
      'name': user.name,
      'email': user.email,
    };
    
    final data = await _graphQL.mutate(mutation, variables: {
      'id': user.id,
      'input': input,
    });
    return _convertToUser(data['updateUser']);
  }
  
  @override
  Future<void> deleteUser(String id) async {
    const mutation = '''
      mutation DeleteUser(\$id: ID!) {
        deleteUser(id: \$id)
      }
    ''';
    
    await _graphQL.mutate(mutation, variables: {'id': id});
  }
  
  User _convertToUser(Map<String, dynamic> data) {
    return User(
      id: data['id']?.toString() ?? '',
      name: data['name']?.toString() ?? '',
      email: data['email']?.toString() ?? '',
      createdAt: DateTime.tryParse(data['createdAt']?.toString() ?? '') ?? DateTime.now(),
    );
  }
}

// 数据源工厂
class DataSourceFactory {
  static DataSource create(String type, HttpClient httpClient) {
    switch (type.toLowerCase()) {
      case 'rest':
        final restApi = RestApiDataSource(httpClient, 'https://api.example.com');
        return RestApiAdapter(restApi);
      case 'graphql':
        final graphQL = GraphQLDataSource(httpClient, 'https://api.example.com/graphql');
        return GraphQLAdapter(graphQL);
      default:
        throw ArgumentError('不支持的数据源类型: $type');
    }
  }
}

// 使用示例
class DataAdapterExample {
  static Future<void> example() async {
    final httpClient = HttpClientFactory.create('dio');
    
    // 使用REST API数据源
    final restDataSource = DataSourceFactory.create('rest', httpClient);
    
    try {
      final users = await restDataSource.getUsers();
      print('REST API用户列表: ${users.length} 个用户');
      
      if (users.isNotEmpty) {
        final user = await restDataSource.getUser(users.first.id);
        print('获取用户: $user');
      }
    } catch (e) {
      print('REST API操作失败: $e');
    }
    
    // 使用GraphQL数据源
    final graphqlDataSource = DataSourceFactory.create('graphql', httpClient);
    
    try {
      final users = await graphqlDataSource.getUsers();
      print('GraphQL用户列表: ${users.length} 个用户');
      
      // 创建新用户
      final newUser = User(
        id: '',
        name: '张三',
        email: 'zhangsan@example.com',
        createdAt: DateTime.now(),
      );
      
      final createdUser = await graphqlDataSource.createUser(newUser);
      print('创建用户: $createdUser');
    } catch (e) {
      print('GraphQL操作失败: $e');
    }
  }
}
```

### 3. 平台适配器

```dart
import 'dart:io';
import 'package:flutter/foundation.dart';
import 'package:flutter/services.dart';

// 平台服务接口
abstract class PlatformService {
  Future<String> getDeviceInfo();
  Future<void> showNotification(String title, String body);
  Future<bool> requestPermission(String permission);
  Future<void> openUrl(String url);
  Future<String> getAppVersion();
}

// Android平台服务（被适配者）
class AndroidPlatformService {
  static const MethodChannel _channel = MethodChannel('android_platform');
  
  Future<Map<String, dynamic>> getDeviceInfo() async {
    return await _channel.invokeMethod('getDeviceInfo');
  }
  
  Future<void> showNotification(Map<String, dynamic> notification) async {
    await _channel.invokeMethod('showNotification', notification);
  }
  
  Future<bool> requestPermission(String permission) async {
    return await _channel.invokeMethod('requestPermission', {'permission': permission});
  }
  
  Future<void> openUrl(String url) async {
    await _channel.invokeMethod('openUrl', {'url': url});
  }
  
  Future<String> getAppVersion() async {
    return await _channel.invokeMethod('getAppVersion');
  }
}

// iOS平台服务（被适配者）
class IOSPlatformService {
  static const MethodChannel _channel = MethodChannel('ios_platform');
  
  Future<Map<String, dynamic>> getDeviceInfo() async {
    return await _channel.invokeMethod('getDeviceInfo');
  }
  
  Future<void> showNotification(Map<String, dynamic> notification) async {
    await _channel.invokeMethod('showNotification', notification);
  }
  
  Future<bool> requestPermission(String permission) async {
    return await _channel.invokeMethod('requestPermission', {'permission': permission});
  }
  
  Future<void> openUrl(String url) async {
    await _channel.invokeMethod('openUrl', {'url': url});
  }
  
  Future<String> getAppVersion() async {
    return await _channel.invokeMethod('getAppVersion');
  }
}

// Web平台服务（被适配者）
class WebPlatformService {
  Future<Map<String, dynamic>> getDeviceInfo() async {
    return {
      'platform': 'web',
      'userAgent': 'Mozilla/5.0...',
      'language': 'zh-CN',
    };
  }
  
  Future<void> showNotification(Map<String, dynamic> notification) async {
    // Web通知实现
    print('Web通知: ${notification['title']} - ${notification['body']}');
  }
  
  Future<bool> requestPermission(String permission) async {
    // Web权限请求
    print('Web权限请求: $permission');
    return true;
  }
  
  Future<void> openUrl(String url) async {
    // Web打开URL
    print('Web打开URL: $url');
  }
  
  Future<String> getAppVersion() async {
    return '1.0.0-web';
  }
}

// Android适配器
class AndroidAdapter implements PlatformService {
  final AndroidPlatformService _androidService;
  
  AndroidAdapter() : _androidService = AndroidPlatformService();
  
  @override
  Future<String> getDeviceInfo() async {
    final info = await _androidService.getDeviceInfo();
    return 'Android设备: ${info['model']} (${info['version']})';
  }
  
  @override
  Future<void> showNotification(String title, String body) async {
    await _androidService.showNotification({
      'title': title,
      'body': body,
      'channelId': 'default',
    });
  }
  
  @override
  Future<bool> requestPermission(String permission) async {
    return await _androidService.requestPermission(permission);
  }
  
  @override
  Future<void> openUrl(String url) async {
    await _androidService.openUrl(url);
  }
  
  @override
  Future<String> getAppVersion() async {
    return await _androidService.getAppVersion();
  }
}

// iOS适配器
class IOSAdapter implements PlatformService {
  final IOSPlatformService _iosService;
  
  IOSAdapter() : _iosService = IOSPlatformService();
  
  @override
  Future<String> getDeviceInfo() async {
    final info = await _iosService.getDeviceInfo();
    return 'iOS设备: ${info['model']} (${info['version']})';
  }
  
  @override
  Future<void> showNotification(String title, String body) async {
    await _iosService.showNotification({
      'title': title,
      'body': body,
      'sound': 'default',
    });
  }
  
  @override
  Future<bool> requestPermission(String permission) async {
    return await _iosService.requestPermission(permission);
  }
  
  @override
  Future<void> openUrl(String url) async {
    await _iosService.openUrl(url);
  }
  
  @override
  Future<String> getAppVersion() async {
    return await _iosService.getAppVersion();
  }
}

// Web适配器
class WebAdapter implements PlatformService {
  final WebPlatformService _webService;
  
  WebAdapter() : _webService = WebPlatformService();
  
  @override
  Future<String> getDeviceInfo() async {
    final info = await _webService.getDeviceInfo();
    return 'Web平台: ${info['userAgent']}';
  }
  
  @override
  Future<void> showNotification(String title, String body) async {
    await _webService.showNotification({
      'title': title,
      'body': body,
    });
  }
  
  @override
  Future<bool> requestPermission(String permission) async {
    return await _webService.requestPermission(permission);
  }
  
  @override
  Future<void> openUrl(String url) async {
    await _webService.openUrl(url);
  }
  
  @override
  Future<String> getAppVersion() async {
    return await _webService.getAppVersion();
  }
}

// 平台服务工厂
class PlatformServiceFactory {
  static PlatformService create() {
    if (kIsWeb) {
      return WebAdapter();
    } else if (Platform.isAndroid) {
      return AndroidAdapter();
    } else if (Platform.isIOS) {
      return IOSAdapter();
    } else {
      throw UnsupportedError('不支持的平台');
    }
  }
}

// 使用示例
class PlatformAdapterExample {
  static Future<void> example() async {
    final platformService = PlatformServiceFactory.create();
    
    try {
      // 获取设备信息
      final deviceInfo = await platformService.getDeviceInfo();
      print('设备信息: $deviceInfo');
      
      // 获取应用版本
      final appVersion = await platformService.getAppVersion();
      print('应用版本: $appVersion');
      
      // 请求权限
      final hasPermission = await platformService.requestPermission('camera');
      print('相机权限: ${hasPermission ? '已授权' : '未授权'}');
      
      // 显示通知
      await platformService.showNotification(
        '测试通知',
        '这是一个跨平台通知示例',
      );
      
      // 打开URL
      await platformService.openUrl('https://flutter.dev');
      
    } catch (e) {
      print('平台服务操作失败: $e');
    }
  }
}
```

## 🛠 测试和调试

### 1. 适配器测试

```dart
import 'package:flutter_test/flutter_test.dart';
import 'package:mockito/mockito.dart';

// Mock类
class MockAdvancedMediaPlayer extends Mock implements AdvancedMediaPlayer {}

void main() {
  group('Adapter Pattern Tests', () {
    test('MediaAdapter应该正确适配VLC播放', () {
      final adapter = MediaAdapter();
      
      // 测试不会抛出异常
      expect(() => adapter.play('vlc', 'test.vlc'), returnsNormally);
    });
    
    test('MediaAdapter应该正确适配MP4播放', () {
      final adapter = MediaAdapter();
      
      expect(() => adapter.play('mp4', 'test.mp4'), returnsNormally);
    });
    
    test('MediaAdapter应该抛出异常对于不支持的格式', () {
      final adapter = MediaAdapter();
      
      expect(
        () => adapter.play('avi', 'test.avi'),
        throwsArgumentError,
      );
    });
    
    test('HttpClientFactory应该创建正确的适配器', () {
      final dioClient = HttpClientFactory.create('dio');
      expect(dioClient, isA<DioAdapter>());
      
      final httpClient = HttpClientFactory.create('http');
      expect(httpClient, isA<HttpPackageAdapter>());
    });
  });
}
```

### 2. 适配器调试器

```dart
class AdapterDebugger {
  static final Map<String, List<String>> _adaptationLogs = {};
  static final Map<String, int> _adaptationCounts = {};
  
  // 记录适配操作
  static void logAdaptation(String adapterType, String operation, String details) {
    final timestamp = DateTime.now().toIso8601String();
    final logEntry = '[$timestamp] $operation: $details';
    
    _adaptationLogs.putIfAbsent(adapterType, () => []).add(logEntry);
    _adaptationCounts[adapterType] = (_adaptationCounts[adapterType] ?? 0) + 1;
  }
  
  // 获取适配统计
  static Map<String, dynamic> getAdaptationStats() {
    final stats = <String, dynamic>{};
    
    for (final adapterType in _adaptationCounts.keys) {
      stats[adapterType] = {
        'count': _adaptationCounts[adapterType],
        'logs': _adaptationLogs[adapterType],
      };
    }
    
    return stats;
  }
  
  // 打印调试信息
  static void printDebugInfo() {
    final stats = getAdaptationStats();
    print('=== Adapter Debug Info ===');
    
    for (final entry in stats.entries) {
      print('${entry.key}:');
      print('  适配次数: ${entry.value['count']}');
      print('  最近操作:');
      
      final logs = entry.value['logs'] as List<String>;
      for (final log in logs.take(5)) {
        print('    $log');
      }
      print('');
    }
  }
  
  // 重置统计
  static void reset() {
    _adaptationLogs.clear();
    _adaptationCounts.clear();
  }
}

// 带调试功能的适配器基类
abstract class DebuggableAdapter {
  String get adapterType;
  
  void logOperation(String operation, String details) {
    AdapterDebugger.logAdaptation(adapterType, operation, details);
  }
}

// 示例：带调试功能的媒体适配器
class DebuggableMediaAdapter extends DebuggableAdapter implements MediaPlayer {
  final AdvancedMediaPlayer _advancedPlayer;
  
  @override
  String get adapterType => 'MediaAdapter';
  
  DebuggableMediaAdapter() : _advancedPlayer = AdvancedMediaPlayer();
  
  @override
  void play(String audioType, String fileName) {
    logOperation('play', 'type: $audioType, file: $fileName');
    
    switch (audioType.toLowerCase()) {
      case 'vlc':
        _advancedPlayer.playVlc(fileName);
        break;
      case 'mp4':
        _advancedPlayer.playMp4(fileName);
        break;
      default:
        logOperation('error', 'unsupported type: $audioType');
        throw ArgumentError('不支持的音频格式: $audioType');
    }
  }
}
```

## 🔒 最佳实践

### 1. 设计原则
- **单一职责**: 每个适配器只负责一种适配
- **开闭原则**: 对扩展开放，对修改关闭
- **里氏替换**: 适配器应该能够替换目标接口
- **接口隔离**: 提供清晰的适配接口

### 2. 实现建议
- **类型安全**: 确保类型转换的安全性
- **异常处理**: 处理适配过程中的异常
- **性能考虑**: 避免不必要的数据转换
- **文档说明**: 清楚说明适配的映射关系

### 3. 使用场景
- **第三方集成**: 适配第三方库的接口
- **版本兼容**: 处理API版本差异
- **数据转换**: 不同数据格式间的转换
- **平台适配**: 跨平台功能的统一接口

### 4. 注意事项
- **避免过度适配**: 不要为了适配而适配
- **性能影响**: 考虑适配对性能的影响
- **维护成本**: 平衡适配的复杂性和维护成本
- **测试覆盖**: 确保适配器的充分测试

适配器模式是Flutter开发中非常实用的设计模式，它帮助我们整合不同的系统和库，提供统一的接口，使代码更加灵活和可维护。