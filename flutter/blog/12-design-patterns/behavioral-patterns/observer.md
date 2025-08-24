# 观察者模式 (Observer Pattern)

观察者模式定义了对象间的一对多依赖关系，当一个对象的状态发生改变时，所有依赖于它的对象都会得到通知并自动更新。在Flutter中，观察者模式广泛应用于状态管理、事件处理、数据绑定等场景。

## 📋 模式概述

### 定义
观察者模式是一种行为型设计模式，它定义了一种一对多的依赖关系，让多个观察者对象同时监听某一个主题对象，当主题对象状态发生变化时，会通知所有观察者对象，使它们能够自动更新自己。

### 适用场景
- 状态管理系统
- 事件处理机制
- 数据绑定
- 模型-视图架构
- 消息通知系统
- 实时数据更新

### 优点
- 降低耦合度
- 支持广播通信
- 符合开闭原则
- 动态建立关系

### 缺点
- 可能导致内存泄漏
- 通知顺序不确定
- 调试困难
- 性能开销

## 🏗 基础实现

### 1. 传统观察者模式

```dart
// 观察者接口
abstract class Observer {
  void update(String message);
}

// 主题接口
abstract class Subject {
  void attach(Observer observer);
  void detach(Observer observer);
  void notify(String message);
}

// 具体主题
class ConcreteSubject implements Subject {
  final List<Observer> _observers = [];
  String _state = '';
  
  String get state => _state;
  
  set state(String newState) {
    _state = newState;
    notify('状态已更新: $newState');
  }
  
  @override
  void attach(Observer observer) {
    _observers.add(observer);
    print('观察者已添加，当前观察者数量: ${_observers.length}');
  }
  
  @override
  void detach(Observer observer) {
    _observers.remove(observer);
    print('观察者已移除，当前观察者数量: ${_observers.length}');
  }
  
  @override
  void notify(String message) {
    print('通知所有观察者: $message');
    for (final observer in _observers) {
      observer.update(message);
    }
  }
}

// 具体观察者A
class ConcreteObserverA implements Observer {
  final String _name;
  
  ConcreteObserverA(this._name);
  
  @override
  void update(String message) {
    print('观察者A ($_name) 收到通知: $message');
  }
}

// 具体观察者B
class ConcreteObserverB implements Observer {
  final String _name;
  
  ConcreteObserverB(this._name);
  
  @override
  void update(String message) {
    print('观察者B ($_name) 收到通知: $message');
  }
}

// 使用示例
void main() {
  final subject = ConcreteSubject();
  final observerA = ConcreteObserverA('用户界面');
  final observerB = ConcreteObserverB('日志系统');
  
  // 添加观察者
  subject.attach(observerA);
  subject.attach(observerB);
  
  // 改变状态
  subject.state = '数据已加载';
  subject.state = '用户已登录';
  
  // 移除观察者
  subject.detach(observerA);
  subject.state = '数据已更新';
}
```

### 2. 基于Stream的观察者模式

```dart
import 'dart:async';

// 基于Stream的主题
class StreamSubject<T> {
  final StreamController<T> _controller = StreamController<T>.broadcast();
  
  // 获取流
  Stream<T> get stream => _controller.stream;
  
  // 发送数据
  void emit(T data) {
    if (!_controller.isClosed) {
      _controller.add(data);
    }
  }
  
  // 发送错误
  void emitError(Object error, [StackTrace? stackTrace]) {
    if (!_controller.isClosed) {
      _controller.addError(error, stackTrace);
    }
  }
  
  // 关闭流
  void close() {
    _controller.close();
  }
  
  // 是否已关闭
  bool get isClosed => _controller.isClosed;
}

// 基于Stream的观察者
class StreamObserver<T> {
  final String name;
  StreamSubscription<T>? _subscription;
  
  StreamObserver(this.name);
  
  // 订阅主题
  void subscribe(StreamSubject<T> subject) {
    _subscription = subject.stream.listen(
      (data) => onData(data),
      onError: (error, stackTrace) => onError(error, stackTrace),
      onDone: () => onDone(),
    );
  }
  
  // 取消订阅
  void unsubscribe() {
    _subscription?.cancel();
    _subscription = null;
  }
  
  // 处理数据
  void onData(T data) {
    print('观察者 $name 收到数据: $data');
  }
  
  // 处理错误
  void onError(Object error, StackTrace stackTrace) {
    print('观察者 $name 收到错误: $error');
  }
  
  // 处理完成
  void onDone() {
    print('观察者 $name: 流已关闭');
  }
}

// 使用示例
void main() async {
  final subject = StreamSubject<String>();
  final observer1 = StreamObserver<String>('UI');
  final observer2 = StreamObserver<String>('Logger');
  
  // 订阅
  observer1.subscribe(subject);
  observer2.subscribe(subject);
  
  // 发送数据
  subject.emit('用户登录');
  subject.emit('数据加载完成');
  
  // 等待处理
  await Future.delayed(Duration(milliseconds: 100));
  
  // 取消订阅
  observer1.unsubscribe();
  subject.emit('只有Logger能收到');
  
  await Future.delayed(Duration(milliseconds: 100));
  
  // 关闭
  subject.close();
  observer2.unsubscribe();
}
```

## 🎯 Flutter应用实例

### 1. 状态管理观察者

```dart
import 'package:flutter/material.dart';
import 'dart:async';

// 应用状态
enum AppState {
  loading,
  loaded,
  error,
}

// 状态数据
class StateData {
  final AppState state;
  final String? message;
  final dynamic data;
  
  StateData({
    required this.state,
    this.message,
    this.data,
  });
  
  @override
  String toString() {
    return 'StateData(state: $state, message: $message, data: $data)';
  }
}

// 状态管理器
class StateManager {
  static final StateManager _instance = StateManager._internal();
  factory StateManager() => _instance;
  StateManager._internal();
  
  final StreamController<StateData> _stateController = 
      StreamController<StateData>.broadcast();
  
  StateData _currentState = StateData(state: AppState.loading);
  
  // 获取当前状态
  StateData get currentState => _currentState;
  
  // 状态流
  Stream<StateData> get stateStream => _stateController.stream;
  
  // 更新状态
  void updateState(StateData newState) {
    _currentState = newState;
    _stateController.add(newState);
    print('状态已更新: $newState');
  }
  
  // 设置加载状态
  void setLoading([String? message]) {
    updateState(StateData(
      state: AppState.loading,
      message: message ?? '加载中...',
    ));
  }
  
  // 设置加载完成状态
  void setLoaded(dynamic data, [String? message]) {
    updateState(StateData(
      state: AppState.loaded,
      message: message ?? '加载完成',
      data: data,
    ));
  }
  
  // 设置错误状态
  void setError(String message, [dynamic error]) {
    updateState(StateData(
      state: AppState.error,
      message: message,
      data: error,
    ));
  }
  
  // 释放资源
  void dispose() {
    _stateController.close();
  }
}

// 状态观察者Widget
class StateObserverWidget extends StatefulWidget {
  final Widget Function(BuildContext context, StateData state) builder;
  final Widget? loadingWidget;
  final Widget Function(BuildContext context, String message)? errorBuilder;
  
  const StateObserverWidget({
    Key? key,
    required this.builder,
    this.loadingWidget,
    this.errorBuilder,
  }) : super(key: key);
  
  @override
  State<StateObserverWidget> createState() => _StateObserverWidgetState();
}

class _StateObserverWidgetState extends State<StateObserverWidget> {
  final StateManager _stateManager = StateManager();
  StreamSubscription<StateData>? _subscription;
  late StateData _currentState;
  
  @override
  void initState() {
    super.initState();
    _currentState = _stateManager.currentState;
    _subscription = _stateManager.stateStream.listen((state) {
      if (mounted) {
        setState(() {
          _currentState = state;
        });
      }
    });
  }
  
  @override
  void dispose() {
    _subscription?.cancel();
    super.dispose();
  }
  
  @override
  Widget build(BuildContext context) {
    switch (_currentState.state) {
      case AppState.loading:
        return widget.loadingWidget ?? 
            const Center(child: CircularProgressIndicator());
      
      case AppState.error:
        if (widget.errorBuilder != null) {
          return widget.errorBuilder!(context, _currentState.message ?? '未知错误');
        }
        return Center(
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              const Icon(Icons.error, color: Colors.red, size: 48),
              const SizedBox(height: 16),
              Text(
                _currentState.message ?? '发生错误',
                style: const TextStyle(color: Colors.red),
              ),
            ],
          ),
        );
      
      case AppState.loaded:
        return widget.builder(context, _currentState);
    }
  }
}

// 数据服务
class DataService {
  final StateManager _stateManager = StateManager();
  
  // 模拟加载数据
  Future<void> loadData() async {
    _stateManager.setLoading('正在加载数据...');
    
    try {
      // 模拟网络请求
      await Future.delayed(const Duration(seconds: 2));
      
      // 模拟随机成功/失败
      if (DateTime.now().millisecond % 2 == 0) {
        final data = {
          'users': ['Alice', 'Bob', 'Charlie'],
          'timestamp': DateTime.now().toIso8601String(),
        };
        _stateManager.setLoaded(data, '数据加载成功');
      } else {
        _stateManager.setError('网络连接失败');
      }
    } catch (e) {
      _stateManager.setError('加载数据时发生错误: $e');
    }
  }
  
  // 刷新数据
  Future<void> refreshData() async {
    await loadData();
  }
}

// 使用示例页面
class StateObserverExample extends StatelessWidget {
  final DataService _dataService = DataService();
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('状态观察者示例'),
        actions: [
          IconButton(
            icon: const Icon(Icons.refresh),
            onPressed: () => _dataService.refreshData(),
          ),
        ],
      ),
      body: StateObserverWidget(
        builder: (context, state) {
          final data = state.data as Map<String, dynamic>?;
          final users = data?['users'] as List<String>? ?? [];
          
          return Column(
            children: [
              Padding(
                padding: const EdgeInsets.all(16),
                child: Text(
                  state.message ?? '数据已加载',
                  style: Theme.of(context).textTheme.titleMedium,
                ),
              ),
              Expanded(
                child: ListView.builder(
                  itemCount: users.length,
                  itemBuilder: (context, index) {
                    return ListTile(
                      leading: CircleAvatar(
                        child: Text('${index + 1}'),
                      ),
                      title: Text(users[index]),
                      subtitle: Text('用户 ${index + 1}'),
                    );
                  },
                ),
              ),
            ],
          );
        },
        errorBuilder: (context, message) {
          return Center(
            child: Column(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                const Icon(Icons.error_outline, size: 64, color: Colors.red),
                const SizedBox(height: 16),
                Text(
                  message,
                  style: const TextStyle(fontSize: 16, color: Colors.red),
                  textAlign: TextAlign.center,
                ),
                const SizedBox(height: 24),
                ElevatedButton(
                  onPressed: () => _dataService.refreshData(),
                  child: const Text('重试'),
                ),
              ],
            ),
          );
        },
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () => _dataService.loadData(),
        child: const Icon(Icons.download),
      ),
    );
  }
}
```

### 2. 事件总线观察者

```dart
import 'dart:async';

// 事件基类
abstract class Event {
  final DateTime timestamp;
  
  Event() : timestamp = DateTime.now();
}

// 具体事件类型
class UserLoginEvent extends Event {
  final String userId;
  final String username;
  
  UserLoginEvent(this.userId, this.username);
  
  @override
  String toString() => 'UserLoginEvent(userId: $userId, username: $username)';
}

class UserLogoutEvent extends Event {
  final String userId;
  
  UserLogoutEvent(this.userId);
  
  @override
  String toString() => 'UserLogoutEvent(userId: $userId)';
}

class DataUpdateEvent extends Event {
  final String dataType;
  final dynamic data;
  
  DataUpdateEvent(this.dataType, this.data);
  
  @override
  String toString() => 'DataUpdateEvent(dataType: $dataType, data: $data)';
}

class ErrorEvent extends Event {
  final String message;
  final dynamic error;
  
  ErrorEvent(this.message, [this.error]);
  
  @override
  String toString() => 'ErrorEvent(message: $message, error: $error)';
}

// 事件总线
class EventBus {
  static final EventBus _instance = EventBus._internal();
  factory EventBus() => _instance;
  EventBus._internal();
  
  final Map<Type, StreamController<Event>> _controllers = {};
  final Map<Type, List<StreamSubscription>> _subscriptions = {};
  
  // 发布事件
  void publish<T extends Event>(T event) {
    final controller = _getController<T>();
    if (!controller.isClosed) {
      controller.add(event);
      print('事件已发布: $event');
    }
  }
  
  // 订阅事件
  StreamSubscription<T> subscribe<T extends Event>(
    void Function(T event) onEvent, {
    Function? onError,
    void Function()? onDone,
  }) {
    final controller = _getController<T>();
    final subscription = controller.stream
        .where((event) => event is T)
        .cast<T>()
        .listen(
          onEvent,
          onError: onError,
          onDone: onDone,
        );
    
    _subscriptions.putIfAbsent(T, () => []).add(subscription);
    print('已订阅事件类型: $T');
    
    return subscription;
  }
  
  // 取消订阅
  void unsubscribe<T extends Event>(StreamSubscription<T> subscription) {
    subscription.cancel();
    _subscriptions[T]?.remove(subscription);
    print('已取消订阅事件类型: $T');
  }
  
  // 取消所有订阅
  void unsubscribeAll<T extends Event>() {
    final subscriptions = _subscriptions[T];
    if (subscriptions != null) {
      for (final subscription in subscriptions) {
        subscription.cancel();
      }
      subscriptions.clear();
      print('已取消所有 $T 类型的订阅');
    }
  }
  
  // 获取控制器
  StreamController<Event> _getController<T extends Event>() {
    return _controllers.putIfAbsent(
      T,
      () => StreamController<Event>.broadcast(),
    );
  }
  
  // 获取订阅数量
  int getSubscriptionCount<T extends Event>() {
    return _subscriptions[T]?.length ?? 0;
  }
  
  // 清理资源
  void dispose() {
    for (final subscriptions in _subscriptions.values) {
      for (final subscription in subscriptions) {
        subscription.cancel();
      }
    }
    _subscriptions.clear();
    
    for (final controller in _controllers.values) {
      controller.close();
    }
    _controllers.clear();
  }
}

// 事件监听器基类
abstract class EventListener {
  final EventBus _eventBus = EventBus();
  final List<StreamSubscription> _subscriptions = [];
  
  // 订阅事件
  void subscribe<T extends Event>(void Function(T event) onEvent) {
    final subscription = _eventBus.subscribe<T>(onEvent);
    _subscriptions.add(subscription);
  }
  
  // 发布事件
  void publish<T extends Event>(T event) {
    _eventBus.publish(event);
  }
  
  // 清理订阅
  void dispose() {
    for (final subscription in _subscriptions) {
      subscription.cancel();
    }
    _subscriptions.clear();
  }
}

// 用户服务监听器
class UserServiceListener extends EventListener {
  UserServiceListener() {
    _setupListeners();
  }
  
  void _setupListeners() {
    // 监听用户登录事件
    subscribe<UserLoginEvent>((event) {
      print('用户服务: 用户 ${event.username} 已登录');
      _updateUserStatus(event.userId, true);
    });
    
    // 监听用户登出事件
    subscribe<UserLogoutEvent>((event) {
      print('用户服务: 用户 ${event.userId} 已登出');
      _updateUserStatus(event.userId, false);
    });
  }
  
  void _updateUserStatus(String userId, bool isOnline) {
    // 更新用户在线状态
    print('更新用户 $userId 在线状态: ${isOnline ? "在线" : "离线"}');
  }
  
  // 模拟用户登录
  void login(String userId, String username) {
    publish(UserLoginEvent(userId, username));
  }
  
  // 模拟用户登出
  void logout(String userId) {
    publish(UserLogoutEvent(userId));
  }
}

// 日志服务监听器
class LogServiceListener extends EventListener {
  LogServiceListener() {
    _setupListeners();
  }
  
  void _setupListeners() {
    // 监听所有事件类型
    subscribe<UserLoginEvent>((event) {
      _logEvent('用户登录', event.toString());
    });
    
    subscribe<UserLogoutEvent>((event) {
      _logEvent('用户登出', event.toString());
    });
    
    subscribe<DataUpdateEvent>((event) {
      _logEvent('数据更新', event.toString());
    });
    
    subscribe<ErrorEvent>((event) {
      _logEvent('错误', event.toString());
    });
  }
  
  void _logEvent(String type, String details) {
    final timestamp = DateTime.now().toIso8601String();
    print('日志 [$timestamp] $type: $details');
  }
}

// 通知服务监听器
class NotificationServiceListener extends EventListener {
  NotificationServiceListener() {
    _setupListeners();
  }
  
  void _setupListeners() {
    subscribe<UserLoginEvent>((event) {
      _showNotification('欢迎回来', '用户 ${event.username} 已登录');
    });
    
    subscribe<ErrorEvent>((event) {
      _showNotification('错误', event.message);
    });
    
    subscribe<DataUpdateEvent>((event) {
      _showNotification('数据更新', '${event.dataType} 数据已更新');
    });
  }
  
  void _showNotification(String title, String message) {
    print('通知: $title - $message');
  }
}

// 数据服务
class DataService extends EventListener {
  DataService() {
    _setupListeners();
  }
  
  void _setupListeners() {
    subscribe<UserLoginEvent>((event) {
      // 用户登录后加载个人数据
      _loadUserData(event.userId);
    });
    
    subscribe<UserLogoutEvent>((event) {
      // 用户登出后清理数据
      _clearUserData(event.userId);
    });
  }
  
  void _loadUserData(String userId) {
    print('数据服务: 加载用户 $userId 的数据');
    
    // 模拟异步加载
    Future.delayed(const Duration(seconds: 1), () {
      final userData = {
        'userId': userId,
        'profile': {'name': 'User $userId', 'email': 'user$userId@example.com'},
        'preferences': {'theme': 'dark', 'language': 'zh-CN'},
      };
      
      publish(DataUpdateEvent('user_data', userData));
    });
  }
  
  void _clearUserData(String userId) {
    print('数据服务: 清理用户 $userId 的数据');
    publish(DataUpdateEvent('user_data', null));
  }
  
  // 模拟数据更新
  void updateData(String dataType, dynamic data) {
    publish(DataUpdateEvent(dataType, data));
  }
  
  // 模拟错误
  void simulateError(String message) {
    publish(ErrorEvent(message));
  }
}

// 使用示例
class EventBusExample {
  static void example() {
    print('=== 事件总线示例 ===');
    
    // 创建服务监听器
    final userService = UserServiceListener();
    final logService = LogServiceListener();
    final notificationService = NotificationServiceListener();
    final dataService = DataService();
    
    print('\n--- 用户登录 ---');
    userService.login('user123', 'Alice');
    
    // 等待异步操作
    Future.delayed(const Duration(seconds: 2), () {
      print('\n--- 数据更新 ---');
      dataService.updateData('settings', {'theme': 'light'});
      
      print('\n--- 模拟错误 ---');
      dataService.simulateError('网络连接失败');
      
      print('\n--- 用户登出 ---');
      userService.logout('user123');
      
      // 清理资源
      Future.delayed(const Duration(seconds: 1), () {
        print('\n--- 清理资源 ---');
        userService.dispose();
        logService.dispose();
        notificationService.dispose();
        dataService.dispose();
        EventBus().dispose();
      });
    });
  }
}
```

### 3. 数据绑定观察者

```dart
import 'package:flutter/material.dart';
import 'dart:async';

// 可观察的数据模型
class ObservableModel<T> {
  T _value;
  final StreamController<T> _controller = StreamController<T>.broadcast();
  
  ObservableModel(this._value);
  
  // 获取当前值
  T get value => _value;
  
  // 设置新值
  set value(T newValue) {
    if (_value != newValue) {
      _value = newValue;
      _controller.add(newValue);
    }
  }
  
  // 获取流
  Stream<T> get stream => _controller.stream;
  
  // 释放资源
  void dispose() {
    _controller.close();
  }
}

// 用户模型
class User {
  final String id;
  final String name;
  final String email;
  final String avatar;
  
  User({
    required this.id,
    required this.name,
    required this.email,
    required this.avatar,
  });
  
  User copyWith({
    String? id,
    String? name,
    String? email,
    String? avatar,
  }) {
    return User(
      id: id ?? this.id,
      name: name ?? this.name,
      email: email ?? this.email,
      avatar: avatar ?? this.avatar,
    );
  }
  
  @override
  bool operator ==(Object other) {
    if (identical(this, other)) return true;
    return other is User &&
        other.id == id &&
        other.name == name &&
        other.email == email &&
        other.avatar == avatar;
  }
  
  @override
  int get hashCode => Object.hash(id, name, email, avatar);
  
  @override
  String toString() {
    return 'User(id: $id, name: $name, email: $email, avatar: $avatar)';
  }
}

// 用户数据管理器
class UserDataManager {
  static final UserDataManager _instance = UserDataManager._internal();
  factory UserDataManager() => _instance;
  UserDataManager._internal();
  
  final ObservableModel<User?> _currentUser = ObservableModel<User?>(null);
  final ObservableModel<bool> _isLoading = ObservableModel<bool>(false);
  final ObservableModel<String?> _error = ObservableModel<String?>(null);
  
  // 获取可观察的用户数据
  ObservableModel<User?> get currentUser => _currentUser;
  ObservableModel<bool> get isLoading => _isLoading;
  ObservableModel<String?> get error => _error;
  
  // 登录
  Future<void> login(String email, String password) async {
    _isLoading.value = true;
    _error.value = null;
    
    try {
      // 模拟网络请求
      await Future.delayed(const Duration(seconds: 2));
      
      // 模拟登录成功
      final user = User(
        id: 'user_${DateTime.now().millisecondsSinceEpoch}',
        name: email.split('@').first,
        email: email,
        avatar: 'https://avatar.example.com/${email.split('@').first}',
      );
      
      _currentUser.value = user;
    } catch (e) {
      _error.value = '登录失败: $e';
    } finally {
      _isLoading.value = false;
    }
  }
  
  // 登出
  void logout() {
    _currentUser.value = null;
    _error.value = null;
  }
  
  // 更新用户信息
  Future<void> updateUser(User updatedUser) async {
    _isLoading.value = true;
    _error.value = null;
    
    try {
      // 模拟网络请求
      await Future.delayed(const Duration(seconds: 1));
      
      _currentUser.value = updatedUser;
    } catch (e) {
      _error.value = '更新失败: $e';
    } finally {
      _isLoading.value = false;
    }
  }
  
  // 清理资源
  void dispose() {
    _currentUser.dispose();
    _isLoading.dispose();
    _error.dispose();
  }
}

// 数据绑定Widget
class DataBindingWidget<T> extends StatefulWidget {
  final ObservableModel<T> model;
  final Widget Function(BuildContext context, T value) builder;
  
  const DataBindingWidget({
    Key? key,
    required this.model,
    required this.builder,
  }) : super(key: key);
  
  @override
  State<DataBindingWidget<T>> createState() => _DataBindingWidgetState<T>();
}

class _DataBindingWidgetState<T> extends State<DataBindingWidget<T>> {
  StreamSubscription<T>? _subscription;
  late T _currentValue;
  
  @override
  void initState() {
    super.initState();
    _currentValue = widget.model.value;
    _subscription = widget.model.stream.listen((value) {
      if (mounted) {
        setState(() {
          _currentValue = value;
        });
      }
    });
  }
  
  @override
  void dispose() {
    _subscription?.cancel();
    super.dispose();
  }
  
  @override
  Widget build(BuildContext context) {
    return widget.builder(context, _currentValue);
  }
}

// 用户信息Widget
class UserInfoWidget extends StatelessWidget {
  final UserDataManager _userManager = UserDataManager();
  
  @override
  Widget build(BuildContext context) {
    return Card(
      margin: const EdgeInsets.all(16),
      child: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Text(
              '用户信息',
              style: Theme.of(context).textTheme.titleLarge,
            ),
            const SizedBox(height: 16),
            DataBindingWidget<User?>(
              model: _userManager.currentUser,
              builder: (context, user) {
                if (user == null) {
                  return const Text('未登录');
                }
                
                return Column(
                  crossAxisAlignment: CrossAxisAlignment.start,
                  children: [
                    Row(
                      children: [
                        CircleAvatar(
                          backgroundImage: NetworkImage(user.avatar),
                          radius: 24,
                        ),
                        const SizedBox(width: 16),
                        Expanded(
                          child: Column(
                            crossAxisAlignment: CrossAxisAlignment.start,
                            children: [
                              Text(
                                user.name,
                                style: Theme.of(context).textTheme.titleMedium,
                              ),
                              Text(
                                user.email,
                                style: Theme.of(context).textTheme.bodyMedium,
                              ),
                            ],
                          ),
                        ),
                      ],
                    ),
                    const SizedBox(height: 16),
                    Row(
                      children: [
                        ElevatedButton(
                          onPressed: () => _showEditDialog(context, user),
                          child: const Text('编辑'),
                        ),
                        const SizedBox(width: 8),
                        OutlinedButton(
                          onPressed: () => _userManager.logout(),
                          child: const Text('登出'),
                        ),
                      ],
                    ),
                  ],
                );
              },
            ),
          ],
        ),
      ),
    );
  }
  
  void _showEditDialog(BuildContext context, User user) {
    final nameController = TextEditingController(text: user.name);
    final emailController = TextEditingController(text: user.email);
    
    showDialog(
      context: context,
      builder: (context) => AlertDialog(
        title: const Text('编辑用户信息'),
        content: Column(
          mainAxisSize: MainAxisSize.min,
          children: [
            TextField(
              controller: nameController,
              decoration: const InputDecoration(labelText: '姓名'),
            ),
            const SizedBox(height: 16),
            TextField(
              controller: emailController,
              decoration: const InputDecoration(labelText: '邮箱'),
            ),
          ],
        ),
        actions: [
          TextButton(
            onPressed: () => Navigator.of(context).pop(),
            child: const Text('取消'),
          ),
          ElevatedButton(
            onPressed: () {
              final updatedUser = user.copyWith(
                name: nameController.text,
                email: emailController.text,
              );
              _userManager.updateUser(updatedUser);
              Navigator.of(context).pop();
            },
            child: const Text('保存'),
          ),
        ],
      ),
    );
  }
}

// 登录Widget
class LoginWidget extends StatefulWidget {
  @override
  State<LoginWidget> createState() => _LoginWidgetState();
}

class _LoginWidgetState extends State<LoginWidget> {
  final UserDataManager _userManager = UserDataManager();
  final _emailController = TextEditingController();
  final _passwordController = TextEditingController();
  
  @override
  void dispose() {
    _emailController.dispose();
    _passwordController.dispose();
    super.dispose();
  }
  
  @override
  Widget build(BuildContext context) {
    return Card(
      margin: const EdgeInsets.all(16),
      child: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Text(
              '登录',
              style: Theme.of(context).textTheme.titleLarge,
            ),
            const SizedBox(height: 16),
            TextField(
              controller: _emailController,
              decoration: const InputDecoration(
                labelText: '邮箱',
                hintText: '请输入邮箱地址',
              ),
            ),
            const SizedBox(height: 16),
            TextField(
              controller: _passwordController,
              decoration: const InputDecoration(
                labelText: '密码',
                hintText: '请输入密码',
              ),
              obscureText: true,
            ),
            const SizedBox(height: 16),
            DataBindingWidget<String?>(
              model: _userManager.error,
              builder: (context, error) {
                if (error == null) return const SizedBox.shrink();
                return Padding(
                  padding: const EdgeInsets.only(bottom: 16),
                  child: Text(
                    error,
                    style: const TextStyle(color: Colors.red),
                  ),
                );
              },
            ),
            DataBindingWidget<bool>(
              model: _userManager.isLoading,
              builder: (context, isLoading) {
                return SizedBox(
                  width: double.infinity,
                  child: ElevatedButton(
                    onPressed: isLoading ? null : _login,
                    child: isLoading
                        ? const SizedBox(
                            height: 20,
                            width: 20,
                            child: CircularProgressIndicator(strokeWidth: 2),
                          )
                        : const Text('登录'),
                  ),
                );
              },
            ),
          ],
        ),
      ),
    );
  }
  
  void _login() {
    final email = _emailController.text.trim();
    final password = _passwordController.text.trim();
    
    if (email.isEmpty || password.isEmpty) {
      _userManager.error.value = '请输入邮箱和密码';
      return;
    }
    
    _userManager.login(email, password);
  }
}

// 数据绑定示例页面
class DataBindingExample extends StatelessWidget {
  final UserDataManager _userManager = UserDataManager();
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('数据绑定示例'),
      ),
      body: SingleChildScrollView(
        child: Column(
          children: [
            DataBindingWidget<User?>(
              model: _userManager.currentUser,
              builder: (context, user) {
                return user == null ? LoginWidget() : UserInfoWidget();
              },
            ),
            // 状态指示器
            DataBindingWidget<bool>(
              model: _userManager.isLoading,
              builder: (context, isLoading) {
                if (!isLoading) return const SizedBox.shrink();
                return const Padding(
                  padding: EdgeInsets.all(16),
                  child: Row(
                    mainAxisAlignment: MainAxisAlignment.center,
                    children: [
                      SizedBox(
                        width: 16,
                        height: 16,
                        child: CircularProgressIndicator(strokeWidth: 2),
                      ),
                      SizedBox(width: 8),
                      Text('处理中...'),
                    ],
                  ),
                );
              },
            ),
          ],
        ),
      ),
    );
  }
}
```

## 🛠 测试和调试

### 1. 观察者模式测试

```dart
import 'package:flutter_test/flutter_test.dart';
import 'package:mockito/mockito.dart';

// Mock类
class MockObserver extends Mock implements Observer {}

void main() {
  group('Observer Pattern Tests', () {
    late ConcreteSubject subject;
    late MockObserver mockObserver;
    
    setUp(() {
      subject = ConcreteSubject();
      mockObserver = MockObserver();
    });
    
    test('应该能够添加观察者', () {
      subject.attach(mockObserver);
      
      // 验证观察者被添加
      expect(subject._observers.length, equals(1));
    });
    
    test('应该能够移除观察者', () {
      subject.attach(mockObserver);
      subject.detach(mockObserver);
      
      // 验证观察者被移除
      expect(subject._observers.length, equals(0));
    });
    
    test('状态改变时应该通知所有观察者', () {
      subject.attach(mockObserver);
      
      subject.state = '新状态';
      
      // 验证观察者被通知
      verify(mockObserver.update('状态已更新: 新状态')).called(1);
    });
    
    test('多个观察者都应该收到通知', () {
      final observer1 = MockObserver();
      final observer2 = MockObserver();
      
      subject.attach(observer1);
      subject.attach(observer2);
      
      subject.state = '测试状态';
      
      verify(observer1.update('状态已更新: 测试状态')).called(1);
      verify(observer2.update('状态已更新: 测试状态')).called(1);
    });
  });
  
  group('EventBus Tests', () {
    late EventBus eventBus;
    
    setUp(() {
      eventBus = EventBus();
    });
    
    tearDown(() {
      eventBus.dispose();
    });
    
    test('应该能够发布和订阅事件', () async {
      String? receivedMessage;
      
      eventBus.subscribe<UserLoginEvent>((event) {
        receivedMessage = event.username;
      });
      
      eventBus.publish(UserLoginEvent('user123', 'Alice'));
      
      // 等待异步处理
      await Future.delayed(Duration(milliseconds: 10));
      
      expect(receivedMessage, equals('Alice'));
    });
    
    test('应该能够取消订阅', () async {
      int callCount = 0;
      
      final subscription = eventBus.subscribe<UserLoginEvent>((event) {
        callCount++;
      });
      
      eventBus.publish(UserLoginEvent('user123', 'Alice'));
      await Future.delayed(Duration(milliseconds: 10));
      
      eventBus.unsubscribe(subscription);
      eventBus.publish(UserLoginEvent('user456', 'Bob'));
      await Future.delayed(Duration(milliseconds: 10));
      
      expect(callCount, equals(1));
    });
  });
}
```

### 2. 观察者调试器

```dart
class ObserverDebugger {
  static final Map<String, List<String>> _observerLogs = {};
  static final Map<String, int> _notificationCounts = {};
  static final Map<String, int> _observerCounts = {};
  
  // 记录观察者操作
  static void logObserverOperation(
    String subjectType,
    String operation,
    String details,
  ) {
    final timestamp = DateTime.now().toIso8601String();
    final logEntry = '[$timestamp] $operation: $details';
    
    _observerLogs.putIfAbsent(subjectType, () => []).add(logEntry);
  }
  
  // 记录通知
  static void logNotification(String subjectType, String message) {
    _notificationCounts[subjectType] = 
        (_notificationCounts[subjectType] ?? 0) + 1;
    logObserverOperation(subjectType, 'notify', message);
  }
  
  // 记录观察者数量变化
  static void logObserverCountChange(String subjectType, int count) {
    _observerCounts[subjectType] = count;
    logObserverOperation(subjectType, 'count_change', '观察者数量: $count');
  }
  
  // 获取统计信息
  static Map<String, dynamic> getStats() {
    final stats = <String, dynamic>{};
    
    for (final subjectType in _observerLogs.keys) {
      stats[subjectType] = {
        'notificationCount': _notificationCounts[subjectType] ?? 0,
        'currentObserverCount': _observerCounts[subjectType] ?? 0,
        'logs': _observerLogs[subjectType],
      };
    }
    
    return stats;
  }
  
  // 打印调试信息
  static void printDebugInfo() {
    final stats = getStats();
    print('=== Observer Debug Info ===');
    
    for (final entry in stats.entries) {
      print('${entry.key}:');
      print('  通知次数: ${entry.value['notificationCount']}');
      print('  当前观察者数量: ${entry.value['currentObserverCount']}');
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
    _observerLogs.clear();
    _notificationCounts.clear();
    _observerCounts.clear();
  }
}

// 带调试功能的主题基类
abstract class DebuggableSubject implements Subject {
  String get subjectType;
  
  void logOperation(String operation, String details) {
    ObserverDebugger.logObserverOperation(subjectType, operation, details);
  }
  
  void logNotification(String message) {
    ObserverDebugger.logNotification(subjectType, message);
  }
  
  void logObserverCountChange(int count) {
    ObserverDebugger.logObserverCountChange(subjectType, count);
  }
}
```

## 🔒 最佳实践

### 1. 设计原则
- **松耦合**: 主题和观察者之间保持松耦合
- **开闭原则**: 可以添加新的观察者而不修改主题
- **单一职责**: 每个观察者只关注特定的事件
- **依赖倒置**: 依赖抽象而不是具体实现

### 2. 实现建议
- **内存管理**: 及时取消订阅避免内存泄漏
- **异常处理**: 处理观察者中的异常
- **性能优化**: 避免频繁通知影响性能
- **线程安全**: 考虑多线程环境下的安全性

### 3. 使用场景
- **状态管理**: 应用状态变化通知
- **事件系统**: 用户交互事件处理
- **数据绑定**: UI与数据的双向绑定
- **消息通信**: 组件间的消息传递

### 4. 注意事项
- **避免循环依赖**: 防止观察者之间的循环通知
- **控制通知频率**: 避免过于频繁的通知
- **资源清理**: 确保正确释放观察者资源
- **调试困难**: 复杂的观察者网络可能难以调试

观察者模式是Flutter开发中最重要的设计模式之一，它为状态管理、事件处理和数据绑定提供了强大的基础，是构建响应式应用的核心模式。