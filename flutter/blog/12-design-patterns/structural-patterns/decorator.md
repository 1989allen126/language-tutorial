# 装饰器模式 (Decorator Pattern)

装饰器模式允许向一个现有的对象添加新的功能，同时又不改变其结构。这种模式创建了一个装饰类，用来包装原有的类，并在保持类方法签名完整性的前提下，提供了额外的功能。在Flutter中，装饰器模式广泛应用于Widget装饰、功能增强等场景。

## 📋 模式概述

### 定义
装饰器模式是一种结构型设计模式，它允许你通过将对象放入包含行为的特殊封装对象中来为原对象绑定新的行为。

### 适用场景
- Widget功能增强
- 数据处理管道
- 缓存装饰
- 日志装饰
- 权限装饰
- 性能监控装饰

### 优点
- 比继承更灵活
- 可以动态添加功能
- 符合开闭原则
- 可以组合多个装饰器

### 缺点
- 增加系统复杂性
- 可能产生很多小对象
- 调试困难

## 🏗 基础实现

### 1. 基本装饰器模式

```dart
// 组件接口
abstract class Component {
  String operation();
}

// 具体组件
class ConcreteComponent implements Component {
  @override
  String operation() {
    return '基础功能';
  }
}

// 装饰器基类
abstract class Decorator implements Component {
  final Component _component;
  
  Decorator(this._component);
  
  @override
  String operation() {
    return _component.operation();
  }
}

// 具体装饰器A
class ConcreteDecoratorA extends Decorator {
  ConcreteDecoratorA(Component component) : super(component);
  
  @override
  String operation() {
    return '${super.operation()} + 装饰A';
  }
  
  String additionalBehaviorA() {
    return '额外行为A';
  }
}

// 具体装饰器B
class ConcreteDecoratorB extends Decorator {
  ConcreteDecoratorB(Component component) : super(component);
  
  @override
  String operation() {
    return '${super.operation()} + 装饰B';
  }
  
  String additionalBehaviorB() {
    return '额外行为B';
  }
}

// 使用示例
void main() {
  // 基础组件
  Component component = ConcreteComponent();
  print(component.operation()); // 输出: 基础功能
  
  // 添加装饰器A
  component = ConcreteDecoratorA(component);
  print(component.operation()); // 输出: 基础功能 + 装饰A
  
  // 添加装饰器B
  component = ConcreteDecoratorB(component);
  print(component.operation()); // 输出: 基础功能 + 装饰A + 装饰B
}
```

### 2. 函数式装饰器

```dart
// 函数类型定义
typedef Operation = String Function();

// 装饰器函数
Operation withLogging(Operation operation, String name) {
  return () {
    print('开始执行: $name');
    final result = operation();
    print('执行完成: $name, 结果: $result');
    return result;
  };
}

Operation withTiming(Operation operation) {
  return () {
    final stopwatch = Stopwatch()..start();
    final result = operation();
    stopwatch.stop();
    print('执行时间: ${stopwatch.elapsedMilliseconds}ms');
    return result;
  };
}

Operation withRetry(Operation operation, int maxRetries) {
  return () {
    for (int i = 0; i < maxRetries; i++) {
      try {
        return operation();
      } catch (e) {
        if (i == maxRetries - 1) rethrow;
        print('重试 ${i + 1}/$maxRetries: $e');
      }
    }
    throw Exception('所有重试都失败了');
  };
}

// 使用示例
void main() {
  Operation baseOperation = () => '基础操作结果';
  
  // 组合多个装饰器
  final decoratedOperation = withLogging(
    withTiming(
      withRetry(baseOperation, 3)
    ),
    '复合操作'
  );
  
  final result = decoratedOperation();
  print('最终结果: $result');
}
```

## 🎯 Flutter应用实例

### 1. Widget装饰器

```dart
import 'package:flutter/material.dart';

// Widget装饰器基类
abstract class WidgetDecorator extends StatelessWidget {
  final Widget child;
  
  const WidgetDecorator({Key? key, required this.child}) : super(key: key);
}

// 边框装饰器
class BorderDecorator extends WidgetDecorator {
  final Color borderColor;
  final double borderWidth;
  final BorderRadius? borderRadius;
  
  const BorderDecorator({
    Key? key,
    required Widget child,
    this.borderColor = Colors.grey,
    this.borderWidth = 1.0,
    this.borderRadius,
  }) : super(key: key, child: child);
  
  @override
  Widget build(BuildContext context) {
    return Container(
      decoration: BoxDecoration(
        border: Border.all(
          color: borderColor,
          width: borderWidth,
        ),
        borderRadius: borderRadius,
      ),
      child: child,
    );
  }
}

// 阴影装饰器
class ShadowDecorator extends WidgetDecorator {
  final List<BoxShadow> shadows;
  final BorderRadius? borderRadius;
  
  const ShadowDecorator({
    Key? key,
    required Widget child,
    this.shadows = const [
      BoxShadow(
        color: Colors.black26,
        blurRadius: 4,
        offset: Offset(0, 2),
      ),
    ],
    this.borderRadius,
  }) : super(key: key, child: child);
  
  @override
  Widget build(BuildContext context) {
    return Container(
      decoration: BoxDecoration(
        boxShadow: shadows,
        borderRadius: borderRadius,
      ),
      child: child,
    );
  }
}

// 渐变背景装饰器
class GradientDecorator extends WidgetDecorator {
  final Gradient gradient;
  final BorderRadius? borderRadius;
  
  const GradientDecorator({
    Key? key,
    required Widget child,
    required this.gradient,
    this.borderRadius,
  }) : super(key: key, child: child);
  
  @override
  Widget build(BuildContext context) {
    return Container(
      decoration: BoxDecoration(
        gradient: gradient,
        borderRadius: borderRadius,
      ),
      child: child,
    );
  }
}

// 内边距装饰器
class PaddingDecorator extends WidgetDecorator {
  final EdgeInsetsGeometry padding;
  
  const PaddingDecorator({
    Key? key,
    required Widget child,
    required this.padding,
  }) : super(key: key, child: child);
  
  @override
  Widget build(BuildContext context) {
    return Padding(
      padding: padding,
      child: child,
    );
  }
}

// 点击效果装饰器
class RippleDecorator extends WidgetDecorator {
  final VoidCallback? onTap;
  final Color? splashColor;
  final Color? highlightColor;
  final BorderRadius? borderRadius;
  
  const RippleDecorator({
    Key? key,
    required Widget child,
    this.onTap,
    this.splashColor,
    this.highlightColor,
    this.borderRadius,
  }) : super(key: key, child: child);
  
  @override
  Widget build(BuildContext context) {
    return Material(
      color: Colors.transparent,
      child: InkWell(
        onTap: onTap,
        splashColor: splashColor,
        highlightColor: highlightColor,
        borderRadius: borderRadius,
        child: child,
      ),
    );
  }
}

// 动画装饰器
class AnimationDecorator extends StatefulWidget {
  final Widget child;
  final Duration duration;
  final Curve curve;
  final bool autoStart;
  
  const AnimationDecorator({
    Key? key,
    required this.child,
    this.duration = const Duration(milliseconds: 300),
    this.curve = Curves.easeInOut,
    this.autoStart = true,
  }) : super(key: key);
  
  @override
  State<AnimationDecorator> createState() => _AnimationDecoratorState();
}

class _AnimationDecoratorState extends State<AnimationDecorator>
    with SingleTickerProviderStateMixin {
  late AnimationController _controller;
  late Animation<double> _animation;
  
  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      duration: widget.duration,
      vsync: this,
    );
    _animation = CurvedAnimation(
      parent: _controller,
      curve: widget.curve,
    );
    
    if (widget.autoStart) {
      _controller.forward();
    }
  }
  
  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }
  
  @override
  Widget build(BuildContext context) {
    return FadeTransition(
      opacity: _animation,
      child: SlideTransition(
        position: Tween<Offset>(
          begin: const Offset(0, 0.1),
          end: Offset.zero,
        ).animate(_animation),
        child: widget.child,
      ),
    );
  }
}

// Widget装饰器构建器
class DecoratedWidget {
  Widget _widget;
  
  DecoratedWidget(this._widget);
  
  DecoratedWidget withBorder({
    Color color = Colors.grey,
    double width = 1.0,
    BorderRadius? borderRadius,
  }) {
    _widget = BorderDecorator(
      borderColor: color,
      borderWidth: width,
      borderRadius: borderRadius,
      child: _widget,
    );
    return this;
  }
  
  DecoratedWidget withShadow({
    List<BoxShadow>? shadows,
    BorderRadius? borderRadius,
  }) {
    _widget = ShadowDecorator(
      shadows: shadows ?? [
        const BoxShadow(
          color: Colors.black26,
          blurRadius: 4,
          offset: Offset(0, 2),
        ),
      ],
      borderRadius: borderRadius,
      child: _widget,
    );
    return this;
  }
  
  DecoratedWidget withGradient({
    required Gradient gradient,
    BorderRadius? borderRadius,
  }) {
    _widget = GradientDecorator(
      gradient: gradient,
      borderRadius: borderRadius,
      child: _widget,
    );
    return this;
  }
  
  DecoratedWidget withPadding(EdgeInsetsGeometry padding) {
    _widget = PaddingDecorator(
      padding: padding,
      child: _widget,
    );
    return this;
  }
  
  DecoratedWidget withRipple({
    VoidCallback? onTap,
    Color? splashColor,
    Color? highlightColor,
    BorderRadius? borderRadius,
  }) {
    _widget = RippleDecorator(
      onTap: onTap,
      splashColor: splashColor,
      highlightColor: highlightColor,
      borderRadius: borderRadius,
      child: _widget,
    );
    return this;
  }
  
  DecoratedWidget withAnimation({
    Duration duration = const Duration(milliseconds: 300),
    Curve curve = Curves.easeInOut,
    bool autoStart = true,
  }) {
    _widget = AnimationDecorator(
      duration: duration,
      curve: curve,
      autoStart: autoStart,
      child: _widget,
    );
    return this;
  }
  
  Widget build() => _widget;
}

// 使用示例
class WidgetDecoratorExample extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Widget装饰器示例')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            // 基础Widget
            const Text('基础文本'),
            const SizedBox(height: 20),
            
            // 装饰后的Widget
            DecoratedWidget(
              const Text(
                '装饰文本',
                style: TextStyle(
                  fontSize: 18,
                  fontWeight: FontWeight.bold,
                ),
              ),
            )
                .withPadding(const EdgeInsets.all(16))
                .withGradient(
                  gradient: const LinearGradient(
                    colors: [Colors.blue, Colors.purple],
                  ),
                )
                .withBorder(
                  color: Colors.white,
                  width: 2,
                  borderRadius: BorderRadius.circular(12),
                )
                .withShadow(
                  borderRadius: BorderRadius.circular(12),
                )
                .withRipple(
                  onTap: () => print('点击了装饰文本'),
                  borderRadius: BorderRadius.circular(12),
                )
                .withAnimation()
                .build(),
          ],
        ),
      ),
    );
  }
}
```

### 2. 数据处理装饰器

```dart
// 数据处理器接口
abstract class DataProcessor<T> {
  Future<T> process(T data);
}

// 基础数据处理器
class BaseDataProcessor<T> implements DataProcessor<T> {
  @override
  Future<T> process(T data) async {
    return data;
  }
}

// 数据处理装饰器基类
abstract class DataProcessorDecorator<T> implements DataProcessor<T> {
  final DataProcessor<T> _processor;
  
  DataProcessorDecorator(this._processor);
  
  @override
  Future<T> process(T data) {
    return _processor.process(data);
  }
}

// 缓存装饰器
class CacheDecorator<T> extends DataProcessorDecorator<T> {
  final Map<String, T> _cache = {};
  final String Function(T) _keyGenerator;
  final Duration? _expiry;
  final Map<String, DateTime> _cacheTimestamps = {};
  
  CacheDecorator(
    DataProcessor<T> processor,
    this._keyGenerator, {
    Duration? expiry,
  }) : _expiry = expiry,
       super(processor);
  
  @override
  Future<T> process(T data) async {
    final key = _keyGenerator(data);
    
    // 检查缓存是否存在且未过期
    if (_cache.containsKey(key)) {
      if (_expiry == null || 
          _cacheTimestamps[key] == null ||
          DateTime.now().difference(_cacheTimestamps[key]!) < _expiry!) {
        print('缓存命中: $key');
        return _cache[key]!;
      } else {
        // 缓存过期，清除
        _cache.remove(key);
        _cacheTimestamps.remove(key);
      }
    }
    
    // 处理数据并缓存
    final result = await super.process(data);
    _cache[key] = result;
    _cacheTimestamps[key] = DateTime.now();
    print('缓存存储: $key');
    
    return result;
  }
  
  void clearCache() {
    _cache.clear();
    _cacheTimestamps.clear();
  }
  
  int get cacheSize => _cache.length;
}

// 日志装饰器
class LoggingDecorator<T> extends DataProcessorDecorator<T> {
  final String _name;
  final bool _logInput;
  final bool _logOutput;
  final bool _logTiming;
  
  LoggingDecorator(
    DataProcessor<T> processor,
    this._name, {
    bool logInput = true,
    bool logOutput = true,
    bool logTiming = true,
  }) : _logInput = logInput,
       _logOutput = logOutput,
       _logTiming = logTiming,
       super(processor);
  
  @override
  Future<T> process(T data) async {
    final stopwatch = Stopwatch();
    
    if (_logInput) {
      print('[$_name] 输入: $data');
    }
    
    if (_logTiming) {
      stopwatch.start();
    }
    
    try {
      final result = await super.process(data);
      
      if (_logTiming) {
        stopwatch.stop();
        print('[$_name] 处理时间: ${stopwatch.elapsedMilliseconds}ms');
      }
      
      if (_logOutput) {
        print('[$_name] 输出: $result');
      }
      
      return result;
    } catch (e) {
      if (_logTiming) {
        stopwatch.stop();
        print('[$_name] 错误处理时间: ${stopwatch.elapsedMilliseconds}ms');
      }
      print('[$_name] 错误: $e');
      rethrow;
    }
  }
}

// 重试装饰器
class RetryDecorator<T> extends DataProcessorDecorator<T> {
  final int _maxRetries;
  final Duration _delay;
  final bool Function(Exception)? _shouldRetry;
  
  RetryDecorator(
    DataProcessor<T> processor,
    this._maxRetries, {
    Duration delay = const Duration(milliseconds: 100),
    bool Function(Exception)? shouldRetry,
  }) : _delay = delay,
       _shouldRetry = shouldRetry,
       super(processor);
  
  @override
  Future<T> process(T data) async {
    Exception? lastException;
    
    for (int attempt = 0; attempt <= _maxRetries; attempt++) {
      try {
        return await super.process(data);
      } catch (e) {
        lastException = e is Exception ? e : Exception(e.toString());
        
        if (attempt == _maxRetries) {
          print('重试失败，已达到最大重试次数: ${_maxRetries + 1}');
          rethrow;
        }
        
        if (_shouldRetry != null && !_shouldRetry!(lastException)) {
          print('不应重试的错误: $e');
          rethrow;
        }
        
        print('重试 ${attempt + 1}/${_maxRetries + 1}: $e');
        await Future.delayed(_delay);
      }
    }
    
    throw lastException!;
  }
}

// 验证装饰器
class ValidationDecorator<T> extends DataProcessorDecorator<T> {
  final bool Function(T) _validator;
  final String _errorMessage;
  
  ValidationDecorator(
    DataProcessor<T> processor,
    this._validator,
    this._errorMessage,
  ) : super(processor);
  
  @override
  Future<T> process(T data) async {
    if (!_validator(data)) {
      throw ArgumentError(_errorMessage);
    }
    
    final result = await super.process(data);
    
    if (!_validator(result)) {
      throw StateError('处理结果验证失败: $_errorMessage');
    }
    
    return result;
  }
}

// 性能监控装饰器
class PerformanceDecorator<T> extends DataProcessorDecorator<T> {
  final String _operationName;
  final void Function(String, Duration, bool)? _onComplete;
  
  PerformanceDecorator(
    DataProcessor<T> processor,
    this._operationName, {
    void Function(String, Duration, bool)? onComplete,
  }) : _onComplete = onComplete,
       super(processor);
  
  @override
  Future<T> process(T data) async {
    final stopwatch = Stopwatch()..start();
    bool success = false;
    
    try {
      final result = await super.process(data);
      success = true;
      return result;
    } finally {
      stopwatch.stop();
      final duration = stopwatch.elapsed;
      
      print('性能监控 [$_operationName]: ${duration.inMilliseconds}ms (${success ? '成功' : '失败'})');
      
      _onComplete?.call(_operationName, duration, success);
    }
  }
}

// 数据处理器构建器
class ProcessorBuilder<T> {
  DataProcessor<T> _processor;
  
  ProcessorBuilder(this._processor);
  
  ProcessorBuilder<T> withCache(
    String Function(T) keyGenerator, {
    Duration? expiry,
  }) {
    _processor = CacheDecorator(_processor, keyGenerator, expiry: expiry);
    return this;
  }
  
  ProcessorBuilder<T> withLogging(
    String name, {
    bool logInput = true,
    bool logOutput = true,
    bool logTiming = true,
  }) {
    _processor = LoggingDecorator(
      _processor,
      name,
      logInput: logInput,
      logOutput: logOutput,
      logTiming: logTiming,
    );
    return this;
  }
  
  ProcessorBuilder<T> withRetry(
    int maxRetries, {
    Duration delay = const Duration(milliseconds: 100),
    bool Function(Exception)? shouldRetry,
  }) {
    _processor = RetryDecorator(
      _processor,
      maxRetries,
      delay: delay,
      shouldRetry: shouldRetry,
    );
    return this;
  }
  
  ProcessorBuilder<T> withValidation(
    bool Function(T) validator,
    String errorMessage,
  ) {
    _processor = ValidationDecorator(_processor, validator, errorMessage);
    return this;
  }
  
  ProcessorBuilder<T> withPerformanceMonitoring(
    String operationName, {
    void Function(String, Duration, bool)? onComplete,
  }) {
    _processor = PerformanceDecorator(
      _processor,
      operationName,
      onComplete: onComplete,
    );
    return this;
  }
  
  DataProcessor<T> build() => _processor;
}

// 具体的数据处理器示例
class StringProcessor extends BaseDataProcessor<String> {
  @override
  Future<String> process(String data) async {
    // 模拟处理时间
    await Future.delayed(const Duration(milliseconds: 100));
    return data.toUpperCase();
  }
}

class NumberProcessor extends BaseDataProcessor<int> {
  @override
  Future<int> process(int data) async {
    // 模拟可能失败的操作
    if (data < 0) {
      throw ArgumentError('数字不能为负数');
    }
    await Future.delayed(const Duration(milliseconds: 50));
    return data * 2;
  }
}

// 使用示例
class DataProcessorExample {
  static Future<void> example() async {
    // 字符串处理器示例
    final stringProcessor = ProcessorBuilder<String>(StringProcessor())
        .withValidation(
          (data) => data.isNotEmpty,
          '字符串不能为空',
        )
        .withCache(
          (data) => 'string_$data',
          expiry: const Duration(minutes: 5),
        )
        .withLogging('StringProcessor')
        .withPerformanceMonitoring('字符串处理')
        .build();
    
    try {
      print('=== 字符串处理示例 ===');
      final result1 = await stringProcessor.process('hello');
      print('结果1: $result1');
      
      // 第二次调用应该命中缓存
      final result2 = await stringProcessor.process('hello');
      print('结果2: $result2');
      
      final result3 = await stringProcessor.process('world');
      print('结果3: $result3');
    } catch (e) {
      print('字符串处理错误: $e');
    }
    
    // 数字处理器示例
    final numberProcessor = ProcessorBuilder<int>(NumberProcessor())
        .withValidation(
          (data) => data >= 0,
          '数字必须非负',
        )
        .withRetry(
          2,
          shouldRetry: (e) => e is! ArgumentError,
        )
        .withLogging('NumberProcessor')
        .withPerformanceMonitoring('数字处理')
        .build();
    
    try {
      print('\n=== 数字处理示例 ===');
      final result1 = await numberProcessor.process(5);
      print('结果1: $result1');
      
      final result2 = await numberProcessor.process(10);
      print('结果2: $result2');
      
      // 这个会失败
      final result3 = await numberProcessor.process(-1);
      print('结果3: $result3');
    } catch (e) {
      print('数字处理错误: $e');
    }
  }
}
```

### 3. 网络请求装饰器

```dart
import 'dart:convert';
import 'dart:math';

// HTTP请求接口
abstract class HttpRequest {
  Future<Map<String, dynamic>> execute();
}

// 基础HTTP请求
class BaseHttpRequest implements HttpRequest {
  final String url;
  final String method;
  final Map<String, String>? headers;
  final dynamic body;
  
  BaseHttpRequest({
    required this.url,
    this.method = 'GET',
    this.headers,
    this.body,
  });
  
  @override
  Future<Map<String, dynamic>> execute() async {
    // 模拟网络请求
    await Future.delayed(Duration(milliseconds: Random().nextInt(500) + 100));
    
    // 模拟随机失败
    if (Random().nextBool() && Random().nextDouble() < 0.3) {
      throw Exception('网络请求失败');
    }
    
    return {
      'status': 200,
      'data': {'message': '请求成功', 'url': url},
      'timestamp': DateTime.now().toIso8601String(),
    };
  }
}

// HTTP请求装饰器基类
abstract class HttpRequestDecorator implements HttpRequest {
  final HttpRequest _request;
  
  HttpRequestDecorator(this._request);
  
  @override
  Future<Map<String, dynamic>> execute() {
    return _request.execute();
  }
}

// 认证装饰器
class AuthDecorator extends HttpRequestDecorator {
  final String _token;
  
  AuthDecorator(HttpRequest request, this._token) : super(request);
  
  @override
  Future<Map<String, dynamic>> execute() async {
    print('添加认证头: Bearer $_token');
    // 在实际实现中，这里会修改请求头
    return await super.execute();
  }
}

// 请求限流装饰器
class RateLimitDecorator extends HttpRequestDecorator {
  static final Map<String, List<DateTime>> _requestTimes = {};
  final int _maxRequests;
  final Duration _timeWindow;
  final String _key;
  
  RateLimitDecorator(
    HttpRequest request,
    this._maxRequests,
    this._timeWindow, {
    String? key,
  }) : _key = key ?? 'default',
       super(request);
  
  @override
  Future<Map<String, dynamic>> execute() async {
    final now = DateTime.now();
    final requests = _requestTimes.putIfAbsent(_key, () => []);
    
    // 清理过期的请求记录
    requests.removeWhere((time) => now.difference(time) > _timeWindow);
    
    // 检查是否超过限制
    if (requests.length >= _maxRequests) {
      final oldestRequest = requests.first;
      final waitTime = _timeWindow - now.difference(oldestRequest);
      throw Exception('请求过于频繁，请等待 ${waitTime.inSeconds} 秒');
    }
    
    // 记录当前请求时间
    requests.add(now);
    print('请求限流检查通过: ${requests.length}/$_maxRequests');
    
    return await super.execute();
  }
}

// 超时装饰器
class TimeoutDecorator extends HttpRequestDecorator {
  final Duration _timeout;
  
  TimeoutDecorator(HttpRequest request, this._timeout) : super(request);
  
  @override
  Future<Map<String, dynamic>> execute() async {
    try {
      return await super.execute().timeout(_timeout);
    } on TimeoutException {
      throw Exception('请求超时: ${_timeout.inSeconds}秒');
    }
  }
}

// 响应缓存装饰器
class ResponseCacheDecorator extends HttpRequestDecorator {
  static final Map<String, Map<String, dynamic>> _cache = {};
  static final Map<String, DateTime> _cacheTimestamps = {};
  final Duration _cacheDuration;
  final String Function(HttpRequest) _keyGenerator;
  
  ResponseCacheDecorator(
    HttpRequest request,
    this._cacheDuration,
    this._keyGenerator,
  ) : super(request);
  
  @override
  Future<Map<String, dynamic>> execute() async {
    final key = _keyGenerator(_request);
    final now = DateTime.now();
    
    // 检查缓存
    if (_cache.containsKey(key) && _cacheTimestamps.containsKey(key)) {
      final cacheTime = _cacheTimestamps[key]!;
      if (now.difference(cacheTime) < _cacheDuration) {
        print('响应缓存命中: $key');
        return _cache[key]!;
      } else {
        // 缓存过期
        _cache.remove(key);
        _cacheTimestamps.remove(key);
      }
    }
    
    // 执行请求并缓存响应
    final response = await super.execute();
    _cache[key] = response;
    _cacheTimestamps[key] = now;
    print('响应已缓存: $key');
    
    return response;
  }
}

// 请求重试装饰器
class RequestRetryDecorator extends HttpRequestDecorator {
  final int _maxRetries;
  final Duration _baseDelay;
  final double _backoffMultiplier;
  final bool Function(Exception)? _shouldRetry;
  
  RequestRetryDecorator(
    HttpRequest request,
    this._maxRetries, {
    Duration baseDelay = const Duration(milliseconds: 500),
    double backoffMultiplier = 2.0,
    bool Function(Exception)? shouldRetry,
  }) : _baseDelay = baseDelay,
       _backoffMultiplier = backoffMultiplier,
       _shouldRetry = shouldRetry,
       super(request);
  
  @override
  Future<Map<String, dynamic>> execute() async {
    Exception? lastException;
    
    for (int attempt = 0; attempt <= _maxRetries; attempt++) {
      try {
        return await super.execute();
      } catch (e) {
        lastException = e is Exception ? e : Exception(e.toString());
        
        if (attempt == _maxRetries) {
          print('请求重试失败，已达到最大重试次数: ${_maxRetries + 1}');
          rethrow;
        }
        
        if (_shouldRetry != null && !_shouldRetry!(lastException)) {
          print('不应重试的错误: $e');
          rethrow;
        }
        
        final delay = Duration(
          milliseconds: (_baseDelay.inMilliseconds * 
              pow(_backoffMultiplier, attempt)).round(),
        );
        
        print('请求重试 ${attempt + 1}/${_maxRetries + 1}，延迟 ${delay.inMilliseconds}ms: $e');
        await Future.delayed(delay);
      }
    }
    
    throw lastException!;
  }
}

// 请求日志装饰器
class RequestLoggingDecorator extends HttpRequestDecorator {
  final String _requestId;
  final bool _logRequest;
  final bool _logResponse;
  final bool _logTiming;
  
  RequestLoggingDecorator(
    HttpRequest request, {
    String? requestId,
    bool logRequest = true,
    bool logResponse = true,
    bool logTiming = true,
  }) : _requestId = requestId ?? _generateRequestId(),
       _logRequest = logRequest,
       _logResponse = logResponse,
       _logTiming = logTiming,
       super(request);
  
  static String _generateRequestId() {
    return 'req_${DateTime.now().millisecondsSinceEpoch}_${Random().nextInt(1000)}';
  }
  
  @override
  Future<Map<String, dynamic>> execute() async {
    final stopwatch = Stopwatch();
    
    if (_logRequest) {
      print('[$_requestId] 开始请求');
    }
    
    if (_logTiming) {
      stopwatch.start();
    }
    
    try {
      final response = await super.execute();
      
      if (_logTiming) {
        stopwatch.stop();
        print('[$_requestId] 请求完成，耗时: ${stopwatch.elapsedMilliseconds}ms');
      }
      
      if (_logResponse) {
        print('[$_requestId] 响应状态: ${response['status']}');
      }
      
      return response;
    } catch (e) {
      if (_logTiming) {
        stopwatch.stop();
        print('[$_requestId] 请求失败，耗时: ${stopwatch.elapsedMilliseconds}ms');
      }
      print('[$_requestId] 请求错误: $e');
      rethrow;
    }
  }
}

// HTTP请求构建器
class HttpRequestBuilder {
  HttpRequest _request;
  
  HttpRequestBuilder(this._request);
  
  HttpRequestBuilder withAuth(String token) {
    _request = AuthDecorator(_request, token);
    return this;
  }
  
  HttpRequestBuilder withRateLimit(
    int maxRequests,
    Duration timeWindow, {
    String? key,
  }) {
    _request = RateLimitDecorator(
      _request,
      maxRequests,
      timeWindow,
      key: key,
    );
    return this;
  }
  
  HttpRequestBuilder withTimeout(Duration timeout) {
    _request = TimeoutDecorator(_request, timeout);
    return this;
  }
  
  HttpRequestBuilder withCache(
    Duration cacheDuration, {
    String Function(HttpRequest)? keyGenerator,
  }) {
    _request = ResponseCacheDecorator(
      _request,
      cacheDuration,
      keyGenerator ?? (req) => req.toString(),
    );
    return this;
  }
  
  HttpRequestBuilder withRetry(
    int maxRetries, {
    Duration baseDelay = const Duration(milliseconds: 500),
    double backoffMultiplier = 2.0,
    bool Function(Exception)? shouldRetry,
  }) {
    _request = RequestRetryDecorator(
      _request,
      maxRetries,
      baseDelay: baseDelay,
      backoffMultiplier: backoffMultiplier,
      shouldRetry: shouldRetry,
    );
    return this;
  }
  
  HttpRequestBuilder withLogging({
    String? requestId,
    bool logRequest = true,
    bool logResponse = true,
    bool logTiming = true,
  }) {
    _request = RequestLoggingDecorator(
      _request,
      requestId: requestId,
      logRequest: logRequest,
      logResponse: logResponse,
      logTiming: logTiming,
    );
    return this;
  }
  
  HttpRequest build() => _request;
}

// 使用示例
class HttpRequestExample {
  static Future<void> example() async {
    // 创建装饰后的HTTP请求
    final request = HttpRequestBuilder(
      BaseHttpRequest(url: 'https://api.example.com/users'),
    )
        .withAuth('your-auth-token')
        .withRateLimit(5, const Duration(minutes: 1))
        .withTimeout(const Duration(seconds: 10))
        .withCache(
          const Duration(minutes: 5),
          keyGenerator: (req) => 'users_list',
        )
        .withRetry(
          3,
          shouldRetry: (e) => !e.toString().contains('认证'),
        )
        .withLogging()
        .build();
    
    try {
      print('=== HTTP请求示例 ===');
      
      // 第一次请求
      final response1 = await request.execute();
      print('响应1: ${response1['data']}');
      
      // 第二次请求（应该命中缓存）
      final response2 = await request.execute();
      print('响应2: ${response2['data']}');
      
      // 快速连续请求（测试限流）
      for (int i = 0; i < 3; i++) {
        try {
          final response = await request.execute();
          print('快速请求 ${i + 1}: ${response['data']}');
        } catch (e) {
          print('快速请求 ${i + 1} 失败: $e');
        }
      }
      
    } catch (e) {
      print('请求失败: $e');
    }
  }
}
```

## 🛠 测试和调试

### 1. 装饰器测试

```dart
import 'package:flutter_test/flutter_test.dart';
import 'package:mockito/mockito.dart';

// Mock类
class MockComponent extends Mock implements Component {}
class MockDataProcessor extends Mock implements DataProcessor<String> {}

void main() {
  group('Decorator Pattern Tests', () {
    test('ConcreteDecoratorA应该添加装饰A', () {
      final component = ConcreteComponent();
      final decorator = ConcreteDecoratorA(component);
      
      expect(decorator.operation(), equals('基础功能 + 装饰A'));
    });
    
    test('多个装饰器应该可以组合', () {
      Component component = ConcreteComponent();
      component = ConcreteDecoratorA(component);
      component = ConcreteDecoratorB(component);
      
      expect(component.operation(), equals('基础功能 + 装饰A + 装饰B'));
    });
    
    test('CacheDecorator应该缓存结果', () async {
      final mockProcessor = MockDataProcessor();
      when(mockProcessor.process('test')).thenAnswer((_) async => 'processed');
      
      final cacheDecorator = CacheDecorator(
        mockProcessor,
        (data) => data,
      );
      
      // 第一次调用
      final result1 = await cacheDecorator.process('test');
      expect(result1, equals('processed'));
      
      // 第二次调用应该命中缓存
      final result2 = await cacheDecorator.process('test');
      expect(result2, equals('processed'));
      
      // 验证原始处理器只被调用一次
      verify(mockProcessor.process('test')).called(1);
    });
    
    test('RetryDecorator应该重试失败的操作', () async {
      final mockProcessor = MockDataProcessor();
      when(mockProcessor.process('test'))
          .thenThrow(Exception('第一次失败'))
          .thenAnswer((_) async => 'success');
      
      final retryDecorator = RetryDecorator(mockProcessor, 2);
      
      final result = await retryDecorator.process('test');
      expect(result, equals('success'));
      
      // 验证处理器被调用两次
      verify(mockProcessor.process('test')).called(2);
    });
  });
}
```

### 2. 装饰器调试器

```dart
class DecoratorDebugger {
  static final Map<String, List<String>> _decorationLogs = {};
  static final Map<String, int> _decorationCounts = {};
  static final Map<String, Duration> _decorationTimes = {};
  
  // 记录装饰操作
  static void logDecoration(
    String decoratorType,
    String operation,
    String details, {
    Duration? duration,
  }) {
    final timestamp = DateTime.now().toIso8601String();
    final logEntry = '[$timestamp] $operation: $details';
    
    if (duration != null) {
      logEntry + ' (${duration.inMilliseconds}ms)';
    }
    
    _decorationLogs.putIfAbsent(decoratorType, () => []).add(logEntry);
    _decorationCounts[decoratorType] = (_decorationCounts[decoratorType] ?? 0) + 1;
    
    if (duration != null) {
      _decorationTimes[decoratorType] = 
          (_decorationTimes[decoratorType] ?? Duration.zero) + duration;
    }
  }
  
  // 获取装饰统计
  static Map<String, dynamic> getDecorationStats() {
    final stats = <String, dynamic>{};
    
    for (final decoratorType in _decorationCounts.keys) {
      stats[decoratorType] = {
        'count': _decorationCounts[decoratorType],
        'totalTime': _decorationTimes[decoratorType]?.inMilliseconds ?? 0,
        'averageTime': _decorationCounts[decoratorType]! > 0
            ? (_decorationTimes[decoratorType]?.inMilliseconds ?? 0) /
                _decorationCounts[decoratorType]!
            : 0,
        'logs': _decorationLogs[decoratorType],
      };
    }
    
    return stats;
  }
  
  // 打印调试信息
  static void printDebugInfo() {
    final stats = getDecorationStats();
    print('=== Decorator Debug Info ===');
    
    for (final entry in stats.entries) {
      print('${entry.key}:');
      print('  装饰次数: ${entry.value['count']}');
      print('  总耗时: ${entry.value['totalTime']}ms');
      print('  平均耗时: ${entry.value['averageTime'].toStringAsFixed(2)}ms');
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
    _decorationLogs.clear();
    _decorationCounts.clear();
    _decorationTimes.clear();
  }
}

// 带调试功能的装饰器基类
abstract class DebuggableDecorator<T> extends DataProcessorDecorator<T> {
  final String decoratorType;
  
  DebuggableDecorator(DataProcessor<T> processor, this.decoratorType)
      : super(processor);
  
  void logOperation(String operation, String details, {Duration? duration}) {
    DecoratorDebugger.logDecoration(
      decoratorType,
      operation,
      details,
      duration: duration,
    );
  }
  
  @override
  Future<T> process(T data) async {
    final stopwatch = Stopwatch()..start();
    
    try {
      logOperation('start', 'processing: $data');
      final result = await super.process(data);
      stopwatch.stop();
      logOperation('success', 'result: $result', duration: stopwatch.elapsed);
      return result;
    } catch (e) {
      stopwatch.stop();
      logOperation('error', 'failed: $e', duration: stopwatch.elapsed);
      rethrow;
    }
  }
}
```

## 🔒 最佳实践

### 1. 设计原则
- **单一职责**: 每个装饰器只负责一个功能
- **开闭原则**: 对扩展开放，对修改关闭
- **组合优于继承**: 使用组合而不是继承
- **接口一致性**: 保持装饰前后接口一致

### 2. 实现建议
- **装饰器顺序**: 注意装饰器的应用顺序
- **性能考虑**: 避免过度装饰影响性能
- **异常处理**: 确保异常能正确传播
- **资源管理**: 注意装饰器中的资源释放

### 3. 使用场景
- **功能增强**: 为现有对象添加新功能
- **横切关注点**: 日志、缓存、安全等
- **条件装饰**: 根据条件动态添加功能
- **管道处理**: 数据处理管道

### 4. 注意事项
- **避免过度装饰**: 不要创建过多的装饰层
- **性能影响**: 考虑装饰器对性能的影响
- **调试困难**: 多层装饰可能使调试变困难
- **内存使用**: 注意装饰器链的内存占用

装饰器模式是Flutter开发中非常有用的设计模式，它提供了一种灵活的方式来扩展对象的功能，特别适合处理横切关注点和功能增强的场景。