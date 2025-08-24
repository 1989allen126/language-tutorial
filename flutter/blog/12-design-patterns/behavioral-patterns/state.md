# Flutter 状态模式 (State Pattern)

## 模式概述

### 定义
状态模式允许对象在其内部状态改变时改变它的行为。对象看起来好像修改了它的类。

### 适用场景
- 对象的行为依赖于它的状态，并且它必须在运行时根据状态改变它的行为
- 代码中包含大量与对象状态有关的条件语句
- 需要避免使用大量的 if-else 或 switch-case 语句
- 状态转换逻辑复杂，需要清晰的状态管理

### 优点
- 将与特定状态相关的行为局部化，并且将不同状态的行为分割开来
- 使状态转换显式化
- State 对象可以被共享
- 符合开闭原则，易于扩展新状态

### 缺点
- 增加系统类和对象的个数
- 状态模式的结构与实现都较为复杂
- 对开闭原则的支持并不太好

## 基础实现

### 传统状态模式

```dart
// 抽象状态类
abstract class State {
  void handle(Context context);
  String get stateName;
}

// 上下文类
class Context {
  State _state;
  
  Context(this._state);
  
  void setState(State state) {
    print('状态从 ${_state.stateName} 切换到 ${state.stateName}');
    _state = state;
  }
  
  void request() {
    _state.handle(this);
  }
  
  State get currentState => _state;
}

// 具体状态A
class ConcreteStateA extends State {
  @override
  void handle(Context context) {
    print('在状态A中处理请求');
    // 状态转换逻辑
    context.setState(ConcreteStateB());
  }
  
  @override
  String get stateName => 'StateA';
}

// 具体状态B
class ConcreteStateB extends State {
  @override
  void handle(Context context) {
    print('在状态B中处理请求');
    // 状态转换逻辑
    context.setState(ConcreteStateA());
  }
  
  @override
  String get stateName => 'StateB';
}
```

### 基于枚举的状态模式

```dart
// 状态枚举
enum PlayerState {
  stopped,
  playing,
  paused,
}

// 状态处理器接口
abstract class StateHandler {
  void play();
  void pause();
  void stop();
}

// 音乐播放器上下文
class MusicPlayer {
  PlayerState _currentState = PlayerState.stopped;
  late Map<PlayerState, StateHandler> _stateHandlers;
  
  MusicPlayer() {
    _initializeStateHandlers();
  }
  
  void _initializeStateHandlers() {
    _stateHandlers = {
      PlayerState.stopped: StoppedStateHandler(this),
      PlayerState.playing: PlayingStateHandler(this),
      PlayerState.paused: PausedStateHandler(this),
    };
  }
  
  void play() {
    _stateHandlers[_currentState]?.play();
  }
  
  void pause() {
    _stateHandlers[_currentState]?.pause();
  }
  
  void stop() {
    _stateHandlers[_currentState]?.stop();
  }
  
  void changeState(PlayerState newState) {
    print('状态从 $_currentState 切换到 $newState');
    _currentState = newState;
  }
  
  PlayerState get currentState => _currentState;
}

// 停止状态处理器
class StoppedStateHandler implements StateHandler {
  final MusicPlayer player;
  
  StoppedStateHandler(this.player);
  
  @override
  void play() {
    print('开始播放音乐');
    player.changeState(PlayerState.playing);
  }
  
  @override
  void pause() {
    print('当前已停止，无法暂停');
  }
  
  @override
  void stop() {
    print('当前已停止');
  }
}

// 播放状态处理器
class PlayingStateHandler implements StateHandler {
  final MusicPlayer player;
  
  PlayingStateHandler(this.player);
  
  @override
  void play() {
    print('当前正在播放');
  }
  
  @override
  void pause() {
    print('暂停播放');
    player.changeState(PlayerState.paused);
  }
  
  @override
  void stop() {
    print('停止播放');
    player.changeState(PlayerState.stopped);
  }
}

// 暂停状态处理器
class PausedStateHandler implements StateHandler {
  final MusicPlayer player;
  
  PausedStateHandler(this.player);
  
  @override
  void play() {
    print('继续播放');
    player.changeState(PlayerState.playing);
  }
  
  @override
  void pause() {
    print('当前已暂停');
  }
  
  @override
  void stop() {
    print('停止播放');
    player.changeState(PlayerState.stopped);
  }
}
```

## Flutter 应用实例

### 网络请求状态管理

```dart
// 网络请求状态
enum NetworkState {
  idle,
  loading,
  success,
  error,
}

// 网络请求状态处理器
abstract class NetworkStateHandler {
  void startRequest();
  void onSuccess(dynamic data);
  void onError(String error);
  void reset();
}

// 网络请求管理器
class NetworkRequestManager extends ChangeNotifier {
  NetworkState _currentState = NetworkState.idle;
  late Map<NetworkState, NetworkStateHandler> _stateHandlers;
  dynamic _data;
  String? _error;
  
  NetworkRequestManager() {
    _initializeStateHandlers();
  }
  
  void _initializeStateHandlers() {
    _stateHandlers = {
      NetworkState.idle: IdleStateHandler(this),
      NetworkState.loading: LoadingStateHandler(this),
      NetworkState.success: SuccessStateHandler(this),
      NetworkState.error: ErrorStateHandler(this),
    };
  }
  
  void startRequest() {
    _stateHandlers[_currentState]?.startRequest();
  }
  
  void onSuccess(dynamic data) {
    _stateHandlers[_currentState]?.onSuccess(data);
  }
  
  void onError(String error) {
    _stateHandlers[_currentState]?.onError(error);
  }
  
  void reset() {
    _stateHandlers[_currentState]?.reset();
  }
  
  void _changeState(NetworkState newState) {
    _currentState = newState;
    notifyListeners();
  }
  
  // Getters
  NetworkState get currentState => _currentState;
  dynamic get data => _data;
  String? get error => _error;
  bool get isLoading => _currentState == NetworkState.loading;
  bool get hasData => _currentState == NetworkState.success && _data != null;
  bool get hasError => _currentState == NetworkState.error;
}

// 空闲状态处理器
class IdleStateHandler implements NetworkStateHandler {
  final NetworkRequestManager manager;
  
  IdleStateHandler(this.manager);
  
  @override
  void startRequest() {
    manager._changeState(NetworkState.loading);
  }
  
  @override
  void onSuccess(dynamic data) {
    // 在空闲状态下不应该收到成功回调
  }
  
  @override
  void onError(String error) {
    // 在空闲状态下不应该收到错误回调
  }
  
  @override
  void reset() {
    // 已经是空闲状态
  }
}

// 加载状态处理器
class LoadingStateHandler implements NetworkStateHandler {
  final NetworkRequestManager manager;
  
  LoadingStateHandler(this.manager);
  
  @override
  void startRequest() {
    // 已经在加载中
  }
  
  @override
  void onSuccess(dynamic data) {
    manager._data = data;
    manager._error = null;
    manager._changeState(NetworkState.success);
  }
  
  @override
  void onError(String error) {
    manager._error = error;
    manager._data = null;
    manager._changeState(NetworkState.error);
  }
  
  @override
  void reset() {
    manager._changeState(NetworkState.idle);
  }
}

// 成功状态处理器
class SuccessStateHandler implements NetworkStateHandler {
  final NetworkRequestManager manager;
  
  SuccessStateHandler(this.manager);
  
  @override
  void startRequest() {
    manager._changeState(NetworkState.loading);
  }
  
  @override
  void onSuccess(dynamic data) {
    // 已经是成功状态
  }
  
  @override
  void onError(String error) {
    // 在成功状态下不应该收到错误回调
  }
  
  @override
  void reset() {
    manager._data = null;
    manager._changeState(NetworkState.idle);
  }
}

// 错误状态处理器
class ErrorStateHandler implements NetworkStateHandler {
  final NetworkRequestManager manager;
  
  ErrorStateHandler(this.manager);
  
  @override
  void startRequest() {
    manager._changeState(NetworkState.loading);
  }
  
  @override
  void onSuccess(dynamic data) {
    // 在错误状态下不应该收到成功回调
  }
  
  @override
  void onError(String error) {
    // 已经是错误状态
  }
  
  @override
  void reset() {
    manager._error = null;
    manager._changeState(NetworkState.idle);
  }
}
```

### UI 状态管理

```dart
// UI 状态枚举
enum UIState {
  loading,
  content,
  empty,
  error,
}

// UI 状态管理器
class UIStateManager extends ChangeNotifier {
  UIState _currentState = UIState.loading;
  String? _errorMessage;
  bool _hasData = false;
  
  void showLoading() {
    _currentState = UIState.loading;
    notifyListeners();
  }
  
  void showContent({required bool hasData}) {
    _hasData = hasData;
    _currentState = hasData ? UIState.content : UIState.empty;
    _errorMessage = null;
    notifyListeners();
  }
  
  void showError(String message) {
    _currentState = UIState.error;
    _errorMessage = message;
    notifyListeners();
  }
  
  // Getters
  UIState get currentState => _currentState;
  String? get errorMessage => _errorMessage;
  bool get isLoading => _currentState == UIState.loading;
  bool get hasContent => _currentState == UIState.content;
  bool get isEmpty => _currentState == UIState.empty;
  bool get hasError => _currentState == UIState.error;
}

// UI 状态 Widget
class StateAwareWidget extends StatelessWidget {
  final UIStateManager stateManager;
  final Widget Function() loadingBuilder;
  final Widget Function() contentBuilder;
  final Widget Function() emptyBuilder;
  final Widget Function(String error) errorBuilder;
  
  const StateAwareWidget({
    Key? key,
    required this.stateManager,
    required this.loadingBuilder,
    required this.contentBuilder,
    required this.emptyBuilder,
    required this.errorBuilder,
  }) : super(key: key);
  
  @override
  Widget build(BuildContext context) {
    return AnimatedBuilder(
      animation: stateManager,
      builder: (context, child) {
        switch (stateManager.currentState) {
          case UIState.loading:
            return loadingBuilder();
          case UIState.content:
            return contentBuilder();
          case UIState.empty:
            return emptyBuilder();
          case UIState.error:
            return errorBuilder(stateManager.errorMessage ?? '未知错误');
        }
      },
    );
  }
}
```

## 测试和调试

### 状态模式测试

```dart
import 'package:flutter_test/flutter_test.dart';

void main() {
  group('状态模式测试', () {
    late MusicPlayer player;
    
    setUp(() {
      player = MusicPlayer();
    });
    
    test('初始状态应该是停止', () {
      expect(player.currentState, PlayerState.stopped);
    });
    
    test('从停止状态播放应该切换到播放状态', () {
      player.play();
      expect(player.currentState, PlayerState.playing);
    });
    
    test('从播放状态暂停应该切换到暂停状态', () {
      player.play();
      player.pause();
      expect(player.currentState, PlayerState.paused);
    });
    
    test('从暂停状态播放应该切换到播放状态', () {
      player.play();
      player.pause();
      player.play();
      expect(player.currentState, PlayerState.playing);
    });
    
    test('从任何状态停止都应该切换到停止状态', () {
      player.play();
      player.stop();
      expect(player.currentState, PlayerState.stopped);
      
      player.play();
      player.pause();
      player.stop();
      expect(player.currentState, PlayerState.stopped);
    });
  });
  
  group('网络状态管理器测试', () {
    late NetworkRequestManager manager;
    
    setUp(() {
      manager = NetworkRequestManager();
    });
    
    test('初始状态应该是空闲', () {
      expect(manager.currentState, NetworkState.idle);
    });
    
    test('开始请求应该切换到加载状态', () {
      manager.startRequest();
      expect(manager.currentState, NetworkState.loading);
    });
    
    test('请求成功应该切换到成功状态', () {
      manager.startRequest();
      manager.onSuccess({'data': 'test'});
      expect(manager.currentState, NetworkState.success);
      expect(manager.data, {'data': 'test'});
    });
    
    test('请求失败应该切换到错误状态', () {
      manager.startRequest();
      manager.onError('网络错误');
      expect(manager.currentState, NetworkState.error);
      expect(manager.error, '网络错误');
    });
    
    test('重置应该回到空闲状态', () {
      manager.startRequest();
      manager.onSuccess({'data': 'test'});
      manager.reset();
      expect(manager.currentState, NetworkState.idle);
      expect(manager.data, null);
    });
  });
}
```

### 状态调试器

```dart
class StateDebugger {
  static final List<StateTransition> _transitions = [];
  
  static void logTransition(String context, String fromState, String toState) {
    final transition = StateTransition(
      context: context,
      fromState: fromState,
      toState: toState,
      timestamp: DateTime.now(),
    );
    _transitions.add(transition);
    print('状态转换: $context - $fromState -> $toState');
  }
  
  static List<StateTransition> getTransitions() => List.unmodifiable(_transitions);
  
  static void clearTransitions() => _transitions.clear();
  
  static void printTransitionHistory() {
    print('=== 状态转换历史 ===');
    for (final transition in _transitions) {
      print('${transition.timestamp}: ${transition.context} - '
            '${transition.fromState} -> ${transition.toState}');
    }
  }
}

class StateTransition {
  final String context;
  final String fromState;
  final String toState;
  final DateTime timestamp;
  
  StateTransition({
    required this.context,
    required this.fromState,
    required this.toState,
    required this.timestamp,
  });
}

// 带调试功能的状态管理器
class DebuggableStateManager {
  String _currentState = 'initial';
  final String context;
  
  DebuggableStateManager(this.context);
  
  void changeState(String newState) {
    final oldState = _currentState;
    _currentState = newState;
    StateDebugger.logTransition(context, oldState, newState);
  }
  
  String get currentState => _currentState;
}
```

## 最佳实践

### 设计原则

1. **单一职责原则**
   - 每个状态类只负责处理特定状态的行为
   - 状态转换逻辑清晰明确

2. **开闭原则**
   - 易于添加新状态而不修改现有代码
   - 状态行为的修改不影响其他状态

3. **里氏替换原则**
   - 所有状态类都可以替换抽象状态类
   - 状态切换对客户端透明

### 实现建议

1. **状态设计**
   ```dart
   // 好的做法：明确的状态定义
   enum OrderState {
     pending,    // 待处理
     confirmed,  // 已确认
     shipped,    // 已发货
     delivered,  // 已送达
     cancelled,  // 已取消
   }
   
   // 避免：模糊的状态定义
   enum BadOrderState {
     state1,
     state2,
     state3,
   }
   ```

2. **状态转换**
   ```dart
   // 好的做法：明确的转换规则
   class OrderStateMachine {
     static const Map<OrderState, List<OrderState>> _allowedTransitions = {
       OrderState.pending: [OrderState.confirmed, OrderState.cancelled],
       OrderState.confirmed: [OrderState.shipped, OrderState.cancelled],
       OrderState.shipped: [OrderState.delivered],
       OrderState.delivered: [],
       OrderState.cancelled: [],
     };
     
     bool canTransitionTo(OrderState from, OrderState to) {
       return _allowedTransitions[from]?.contains(to) ?? false;
     }
   }
   ```

3. **状态持久化**
   ```dart
   class PersistentStateManager {
     final SharedPreferences _prefs;
     
     PersistentStateManager(this._prefs);
     
     Future<void> saveState(String key, String state) async {
       await _prefs.setString(key, state);
     }
     
     String? loadState(String key) {
       return _prefs.getString(key);
     }
   }
   ```

### 使用场景

1. **适合使用状态模式的场景**
   - 游戏角色状态管理
   - 网络请求状态管理
   - UI 界面状态管理
   - 订单流程管理
   - 播放器状态管理

2. **不适合使用状态模式的场景**
   - 简单的布尔状态
   - 状态很少且不会增加
   - 状态转换逻辑简单

### 注意事项

1. **避免状态爆炸**
   - 合理设计状态粒度
   - 避免过多的状态组合

2. **状态一致性**
   - 确保状态转换的原子性
   - 处理并发状态变更

3. **内存管理**
   - 及时清理不需要的状态对象
   - 避免状态对象的循环引用

4. **调试支持**
   - 提供状态转换日志
   - 支持状态历史查询

状态模式在 Flutter 开发中非常有用，特别是在处理复杂的 UI 状态和业务流程时。通过合理使用状态模式，可以让代码更加清晰、可维护，并且易于扩展。