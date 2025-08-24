# 策略模式 (Strategy Pattern)

策略模式定义了一系列算法，把它们一个个封装起来，并且使它们可相互替换。策略模式让算法的变化独立于使用算法的客户。在Flutter中，策略模式广泛应用于主题切换、支付方式选择、数据处理算法等场景。

## 📋 模式概述

### 定义
策略模式是一种行为型设计模式，它定义了一系列算法，将每个算法封装起来，并使它们可以相互替换。策略模式使得算法可以在不影响客户端的情况下发生变化。

### 适用场景
- 多种算法实现同一功能
- 需要在运行时选择算法
- 避免使用多重条件判断
- 算法需要独立变化
- 支付方式选择
- 主题切换
- 数据处理策略

### 优点
- 算法可以自由切换
- 避免使用多重条件判断
- 扩展性良好
- 符合开闭原则

### 缺点
- 策略类数量增多
- 客户端必须知道所有策略
- 增加了对象的数目

## 🏗 基础实现

### 1. 传统策略模式

```dart
// 策略接口
abstract class Strategy {
  String execute(String data);
}

// 具体策略A - 大写转换
class UpperCaseStrategy implements Strategy {
  @override
  String execute(String data) {
    print('使用大写转换策略');
    return data.toUpperCase();
  }
}

// 具体策略B - 小写转换
class LowerCaseStrategy implements Strategy {
  @override
  String execute(String data) {
    print('使用小写转换策略');
    return data.toLowerCase();
  }
}

// 具体策略C - 反转字符串
class ReverseStrategy implements Strategy {
  @override
  String execute(String data) {
    print('使用字符串反转策略');
    return data.split('').reversed.join('');
  }
}

// 上下文类
class TextProcessor {
  Strategy? _strategy;
  
  void setStrategy(Strategy strategy) {
    _strategy = strategy;
  }
  
  String processText(String text) {
    if (_strategy == null) {
      throw StateError('策略未设置');
    }
    return _strategy!.execute(text);
  }
  
  String getCurrentStrategyType() {
    return _strategy?.runtimeType.toString() ?? 'None';
  }
}

// 使用示例
void main() {
  final processor = TextProcessor();
  const text = 'Hello World';
  
  // 使用大写策略
  processor.setStrategy(UpperCaseStrategy());
  print('大写结果: ${processor.processText(text)}');
  
  // 使用小写策略
  processor.setStrategy(LowerCaseStrategy());
  print('小写结果: ${processor.processText(text)}');
  
  // 使用反转策略
  processor.setStrategy(ReverseStrategy());
  print('反转结果: ${processor.processText(text)}');
}
```

### 2. 函数式策略模式

```dart
// 函数类型定义
typedef ProcessingStrategy = String Function(String data);

// 策略函数
String upperCaseStrategy(String data) => data.toUpperCase();
String lowerCaseStrategy(String data) => data.toLowerCase();
String reverseStrategy(String data) => data.split('').reversed.join('');
String capitalizeStrategy(String data) => 
    data.split(' ').map((word) => 
        word.isNotEmpty ? word[0].toUpperCase() + word.substring(1).toLowerCase() : ''
    ).join(' ');

// 函数式处理器
class FunctionalTextProcessor {
  ProcessingStrategy? _strategy;
  
  void setStrategy(ProcessingStrategy strategy) {
    _strategy = strategy;
  }
  
  String processText(String text) {
    if (_strategy == null) {
      throw StateError('策略未设置');
    }
    return _strategy!(text);
  }
}

// 策略注册表
class StrategyRegistry {
  static final Map<String, ProcessingStrategy> _strategies = {
    'uppercase': upperCaseStrategy,
    'lowercase': lowerCaseStrategy,
    'reverse': reverseStrategy,
    'capitalize': capitalizeStrategy,
  };
  
  static ProcessingStrategy? getStrategy(String name) {
    return _strategies[name];
  }
  
  static List<String> getAvailableStrategies() {
    return _strategies.keys.toList();
  }
  
  static void registerStrategy(String name, ProcessingStrategy strategy) {
    _strategies[name] = strategy;
  }
}
```

## 🎯 Flutter应用实例

### 1. 主题策略

```dart
// 主题策略接口
abstract class ThemeStrategy {
  ThemeData getTheme();
  String get name;
  IconData get icon;
}

// 亮色主题策略
class LightThemeStrategy implements ThemeStrategy {
  @override
  String get name => '亮色主题';
  
  @override
  IconData get icon => Icons.light_mode;
  
  @override
  ThemeData getTheme() {
    return ThemeData(
      brightness: Brightness.light,
      primarySwatch: Colors.blue,
      scaffoldBackgroundColor: Colors.white,
      appBarTheme: const AppBarTheme(
        backgroundColor: Colors.blue,
        foregroundColor: Colors.white,
      ),
      cardTheme: CardTheme(
        color: Colors.white,
        elevation: 2,
        shape: RoundedRectangleBorder(
          borderRadius: BorderRadius.circular(8),
        ),
      ),
    );
  }
}

// 暗色主题策略
class DarkThemeStrategy implements ThemeStrategy {
  @override
  String get name => '暗色主题';
  
  @override
  IconData get icon => Icons.dark_mode;
  
  @override
  ThemeData getTheme() {
    return ThemeData(
      brightness: Brightness.dark,
      primarySwatch: Colors.blue,
      scaffoldBackgroundColor: Colors.grey[900],
      appBarTheme: AppBarTheme(
        backgroundColor: Colors.grey[850],
        foregroundColor: Colors.white,
      ),
      cardTheme: CardTheme(
        color: Colors.grey[800],
        elevation: 2,
        shape: RoundedRectangleBorder(
          borderRadius: BorderRadius.circular(8),
        ),
      ),
    );
  }
}

// 自定义主题策略
class CustomThemeStrategy implements ThemeStrategy {
  final Color primaryColor;
  final String themeName;
  
  const CustomThemeStrategy({
    required this.primaryColor,
    required this.themeName,
  });
  
  @override
  String get name => themeName;
  
  @override
  IconData get icon => Icons.palette;
  
  @override
  ThemeData getTheme() {
    return ThemeData(
      primarySwatch: MaterialColor(
        primaryColor.value,
        _generateMaterialColorSwatch(primaryColor),
      ),
      scaffoldBackgroundColor: Colors.white,
      appBarTheme: AppBarTheme(
        backgroundColor: primaryColor,
        foregroundColor: Colors.white,
      ),
    );
  }
  
  Map<int, Color> _generateMaterialColorSwatch(Color color) {
    return {
      50: color.withOpacity(0.1),
      100: color.withOpacity(0.2),
      200: color.withOpacity(0.3),
      300: color.withOpacity(0.4),
      400: color.withOpacity(0.6),
      500: color.withOpacity(0.8),
      600: color.withOpacity(0.9),
      700: color,
      800: color,
      900: color,
    };
  }
}

// 主题管理器
class ThemeManager extends ChangeNotifier {
  ThemeStrategy _currentStrategy = LightThemeStrategy();
  
  final List<ThemeStrategy> _availableStrategies = [
    LightThemeStrategy(),
    DarkThemeStrategy(),
    CustomThemeStrategy(
      primaryColor: Colors.green,
      themeName: '绿色主题',
    ),
    CustomThemeStrategy(
      primaryColor: Colors.purple,
      themeName: '紫色主题',
    ),
  ];
  
  ThemeData get currentTheme => _currentStrategy.getTheme();
  ThemeStrategy get currentStrategy => _currentStrategy;
  List<ThemeStrategy> get availableStrategies => _availableStrategies;
  
  void setThemeStrategy(ThemeStrategy strategy) {
    if (_currentStrategy != strategy) {
      _currentStrategy = strategy;
      notifyListeners();
    }
  }
  
  void addCustomStrategy(ThemeStrategy strategy) {
    _availableStrategies.add(strategy);
    notifyListeners();
  }
}
```

### 2. 支付策略

```dart
// 支付结果
class PaymentResult {
  final bool success;
  final String? transactionId;
  final String? error;
  final DateTime timestamp;
  
  PaymentResult({
    required this.success,
    this.transactionId,
    this.error,
    DateTime? timestamp,
  }) : timestamp = timestamp ?? DateTime.now();
}

// 支付策略接口
abstract class PaymentStrategy {
  String get name;
  String get displayName;
  IconData get icon;
  bool get isAvailable;
  
  Future<PaymentResult> processPayment(
    double amount,
    Map<String, dynamic> paymentData,
  );
  
  bool validatePaymentData(Map<String, dynamic> paymentData);
}

// 支付宝策略
class AlipayStrategy implements PaymentStrategy {
  @override
  String get name => 'alipay';
  
  @override
  String get displayName => '支付宝';
  
  @override
  IconData get icon => Icons.account_balance_wallet;
  
  @override
  bool get isAvailable => true;
  
  @override
  bool validatePaymentData(Map<String, dynamic> paymentData) {
    return paymentData.containsKey('account') && 
           paymentData['account'] != null &&
           paymentData['account'].toString().isNotEmpty;
  }
  
  @override
  Future<PaymentResult> processPayment(
    double amount,
    Map<String, dynamic> paymentData,
  ) async {
    if (!validatePaymentData(paymentData)) {
      return PaymentResult(
        success: false,
        error: '支付宝账户信息无效',
      );
    }
    
    try {
      // 模拟支付宝支付处理
      await Future.delayed(const Duration(seconds: 2));
      
      // 模拟支付成功
      if (amount > 0 && amount <= 10000) {
        return PaymentResult(
          success: true,
          transactionId: 'alipay_${DateTime.now().millisecondsSinceEpoch}',
        );
      } else {
        return PaymentResult(
          success: false,
          error: '支付金额超出限制',
        );
      }
    } catch (e) {
      return PaymentResult(
        success: false,
        error: '支付宝支付失败: $e',
      );
    }
  }
}

// 微信支付策略
class WechatPayStrategy implements PaymentStrategy {
  @override
  String get name => 'wechat';
  
  @override
  String get displayName => '微信支付';
  
  @override
  IconData get icon => Icons.chat;
  
  @override
  bool get isAvailable => true;
  
  @override
  bool validatePaymentData(Map<String, dynamic> paymentData) {
    return paymentData.containsKey('openid') && 
           paymentData['openid'] != null &&
           paymentData['openid'].toString().isNotEmpty;
  }
  
  @override
  Future<PaymentResult> processPayment(
    double amount,
    Map<String, dynamic> paymentData,
  ) async {
    if (!validatePaymentData(paymentData)) {
      return PaymentResult(
        success: false,
        error: '微信用户信息无效',
      );
    }
    
    try {
      // 模拟微信支付处理
      await Future.delayed(const Duration(seconds: 1, milliseconds: 500));
      
      // 模拟支付成功
      if (amount > 0 && amount <= 5000) {
        return PaymentResult(
          success: true,
          transactionId: 'wechat_${DateTime.now().millisecondsSinceEpoch}',
        );
      } else {
        return PaymentResult(
          success: false,
          error: '支付金额超出限制',
        );
      }
    } catch (e) {
      return PaymentResult(
        success: false,
        error: '微信支付失败: $e',
      );
    }
  }
}

// 银行卡支付策略
class BankCardStrategy implements PaymentStrategy {
  @override
  String get name => 'bankcard';
  
  @override
  String get displayName => '银行卡';
  
  @override
  IconData get icon => Icons.credit_card;
  
  @override
  bool get isAvailable => true;
  
  @override
  bool validatePaymentData(Map<String, dynamic> paymentData) {
    return paymentData.containsKey('cardNumber') && 
           paymentData.containsKey('cvv') &&
           paymentData.containsKey('expiryDate') &&
           paymentData['cardNumber'] != null &&
           paymentData['cvv'] != null &&
           paymentData['expiryDate'] != null;
  }
  
  @override
  Future<PaymentResult> processPayment(
    double amount,
    Map<String, dynamic> paymentData,
  ) async {
    if (!validatePaymentData(paymentData)) {
      return PaymentResult(
        success: false,
        error: '银行卡信息不完整',
      );
    }
    
    try {
      // 模拟银行卡支付处理
      await Future.delayed(const Duration(seconds: 3));
      
      // 模拟支付成功
      if (amount > 0) {
        return PaymentResult(
          success: true,
          transactionId: 'bank_${DateTime.now().millisecondsSinceEpoch}',
        );
      } else {
        return PaymentResult(
          success: false,
          error: '支付金额无效',
        );
      }
    } catch (e) {
      return PaymentResult(
        success: false,
        error: '银行卡支付失败: $e',
      );
    }
  }
}

// 支付管理器
class PaymentManager {
  PaymentStrategy? _currentStrategy;
  
  final Map<String, PaymentStrategy> _strategies = {
    'alipay': AlipayStrategy(),
    'wechat': WechatPayStrategy(),
    'bankcard': BankCardStrategy(),
  };
  
  List<PaymentStrategy> get availableStrategies => 
      _strategies.values.where((s) => s.isAvailable).toList();
  
  PaymentStrategy? get currentStrategy => _currentStrategy;
  
  void setPaymentStrategy(String strategyName) {
    final strategy = _strategies[strategyName];
    if (strategy != null && strategy.isAvailable) {
      _currentStrategy = strategy;
    } else {
      throw ArgumentError('支付策略不可用: $strategyName');
    }
  }
  
  Future<PaymentResult> processPayment(
    double amount,
    Map<String, dynamic> paymentData,
  ) async {
    if (_currentStrategy == null) {
      return PaymentResult(
        success: false,
        error: '未选择支付方式',
      );
    }
    
    return await _currentStrategy!.processPayment(amount, paymentData);
  }
  
  void addPaymentStrategy(String name, PaymentStrategy strategy) {
    _strategies[name] = strategy;
  }
}
```

### 3. 数据处理策略

```dart
// 数据处理结果
class ProcessingResult {
  final bool success;
  final dynamic data;
  final String? error;
  final Duration processingTime;
  
  ProcessingResult({
    required this.success,
    this.data,
    this.error,
    required this.processingTime,
  });
}

// 数据处理策略接口
abstract class DataProcessingStrategy {
  String get name;
  String get description;
  
  Future<ProcessingResult> process(List<dynamic> data);
  bool canProcess(List<dynamic> data);
}

// 排序策略
class SortingStrategy implements DataProcessingStrategy {
  @override
  String get name => '排序处理';
  
  @override
  String get description => '对数据进行排序处理';
  
  @override
  bool canProcess(List<dynamic> data) {
    return data.isNotEmpty && data.every((item) => item is Comparable);
  }
  
  @override
  Future<ProcessingResult> process(List<dynamic> data) async {
    final stopwatch = Stopwatch()..start();
    
    try {
      if (!canProcess(data)) {
        return ProcessingResult(
          success: false,
          error: '数据不支持排序操作',
          processingTime: stopwatch.elapsed,
        );
      }
      
      // 模拟处理时间
      await Future.delayed(const Duration(milliseconds: 500));
      
      final sortedData = List.from(data)..sort();
      
      return ProcessingResult(
        success: true,
        data: sortedData,
        processingTime: stopwatch.elapsed,
      );
    } catch (e) {
      return ProcessingResult(
        success: false,
        error: '排序处理失败: $e',
        processingTime: stopwatch.elapsed,
      );
    } finally {
      stopwatch.stop();
    }
  }
}

// 过滤策略
class FilteringStrategy implements DataProcessingStrategy {
  final bool Function(dynamic) filterFunction;
  
  FilteringStrategy(this.filterFunction);
  
  @override
  String get name => '过滤处理';
  
  @override
  String get description => '根据条件过滤数据';
  
  @override
  bool canProcess(List<dynamic> data) {
    return data.isNotEmpty;
  }
  
  @override
  Future<ProcessingResult> process(List<dynamic> data) async {
    final stopwatch = Stopwatch()..start();
    
    try {
      if (!canProcess(data)) {
        return ProcessingResult(
          success: false,
          error: '数据为空，无法进行过滤',
          processingTime: stopwatch.elapsed,
        );
      }
      
      // 模拟处理时间
      await Future.delayed(const Duration(milliseconds: 300));
      
      final filteredData = data.where(filterFunction).toList();
      
      return ProcessingResult(
        success: true,
        data: filteredData,
        processingTime: stopwatch.elapsed,
      );
    } catch (e) {
      return ProcessingResult(
        success: false,
        error: '过滤处理失败: $e',
        processingTime: stopwatch.elapsed,
      );
    } finally {
      stopwatch.stop();
    }
  }
}

// 聚合策略
class AggregationStrategy implements DataProcessingStrategy {
  @override
  String get name => '聚合处理';
  
  @override
  String get description => '对数据进行聚合统计';
  
  @override
  bool canProcess(List<dynamic> data) {
    return data.isNotEmpty && data.every((item) => item is num);
  }
  
  @override
  Future<ProcessingResult> process(List<dynamic> data) async {
    final stopwatch = Stopwatch()..start();
    
    try {
      if (!canProcess(data)) {
        return ProcessingResult(
          success: false,
          error: '数据不支持聚合操作（需要数字类型）',
          processingTime: stopwatch.elapsed,
        );
      }
      
      // 模拟处理时间
      await Future.delayed(const Duration(milliseconds: 200));
      
      final numbers = data.cast<num>();
      final result = {
        'count': numbers.length,
        'sum': numbers.reduce((a, b) => a + b),
        'average': numbers.reduce((a, b) => a + b) / numbers.length,
        'min': numbers.reduce((a, b) => a < b ? a : b),
        'max': numbers.reduce((a, b) => a > b ? a : b),
      };
      
      return ProcessingResult(
        success: true,
        data: result,
        processingTime: stopwatch.elapsed,
      );
    } catch (e) {
      return ProcessingResult(
        success: false,
        error: '聚合处理失败: $e',
        processingTime: stopwatch.elapsed,
      );
    } finally {
      stopwatch.stop();
    }
  }
}

// 数据处理管理器
class DataProcessingManager {
  DataProcessingStrategy? _currentStrategy;
  
  final List<DataProcessingStrategy> _strategies = [
    SortingStrategy(),
    FilteringStrategy((item) => item is num && item > 0),
    AggregationStrategy(),
  ];
  
  List<DataProcessingStrategy> get availableStrategies => _strategies;
  DataProcessingStrategy? get currentStrategy => _currentStrategy;
  
  void setProcessingStrategy(DataProcessingStrategy strategy) {
    _currentStrategy = strategy;
  }
  
  Future<ProcessingResult> processData(List<dynamic> data) async {
    if (_currentStrategy == null) {
      return ProcessingResult(
        success: false,
        error: '未选择处理策略',
        processingTime: Duration.zero,
      );
    }
    
    return await _currentStrategy!.process(data);
  }
  
  void addStrategy(DataProcessingStrategy strategy) {
    _strategies.add(strategy);
  }
}

// 策略演示页面
class StrategyDemoPage extends StatefulWidget {
  const StrategyDemoPage({Key? key}) : super(key: key);
  
  @override
  State<StrategyDemoPage> createState() => _StrategyDemoPageState();
}

class _StrategyDemoPageState extends State<StrategyDemoPage> {
  final DataProcessingManager _manager = DataProcessingManager();
  final TextEditingController _dataController = TextEditingController();
  
  DataProcessingStrategy? _selectedStrategy;
  ProcessingResult? _lastResult;
  bool _isProcessing = false;
  
  final List<dynamic> _sampleData = [3, 1, 4, 1, 5, 9, 2, 6, 5, 3, 5];
  
  @override
  void initState() {
    super.initState();
    _dataController.text = _sampleData.toString();
  }
  
  @override
  void dispose() {
    _dataController.dispose();
    super.dispose();
  }
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('策略模式演示'),
        elevation: 0,
      ),
      body: SingleChildScrollView(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            _buildStrategySelector(),
            const SizedBox(height: 16),
            _buildDataInput(),
            const SizedBox(height: 16),
            _buildProcessButton(),
            if (_lastResult != null) ..[
              const SizedBox(height: 16),
              _buildResult(),
            ],
          ],
        ),
      ),
    );
  }
  
  Widget _buildStrategySelector() {
    return Card(
      child: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Text(
              '选择处理策略',
              style: Theme.of(context).textTheme.titleLarge,
            ),
            const SizedBox(height: 16),
            ...(_manager.availableStrategies.map((strategy) {
              return RadioListTile<DataProcessingStrategy>(
                title: Text(strategy.name),
                subtitle: Text(strategy.description),
                value: strategy,
                groupValue: _selectedStrategy,
                onChanged: (value) {
                  setState(() {
                    _selectedStrategy = value;
                    _lastResult = null;
                  });
                  if (value != null) {
                    _manager.setProcessingStrategy(value);
                  }
                },
              );
            }).toList()),
          ],
        ),
      ),
    );
  }
  
  Widget _buildDataInput() {
    return Card(
      child: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Text(
              '输入数据',
              style: Theme.of(context).textTheme.titleLarge,
            ),
            const SizedBox(height: 16),
            TextField(
              controller: _dataController,
              maxLines: 8,
              decoration: const InputDecoration(
                hintText: '请输入要处理的数据...',
                border: OutlineInputBorder(),
              ),
            ),
            const SizedBox(height: 8),
            Row(
              children: [
                TextButton(
                  onPressed: () {
                    _dataController.text = _sampleData.toString();
                  },
                  child: const Text('使用示例数据'),
                ),
                const SizedBox(width: 8),
                TextButton(
                  onPressed: () {
                    _dataController.clear();
                  },
                  child: const Text('清空'),
                ),
              ],
            ),
          ],
        ),
      ),
    );
  }
  
  Widget _buildProcessButton() {
    return SizedBox(
      width: double.infinity,
      child: ElevatedButton(
        onPressed: (_selectedStrategy != null && !_isProcessing)
            ? _processData
            : null,
        child: _isProcessing
            ? const Row(
                mainAxisAlignment: MainAxisAlignment.center,
                children: [
                  SizedBox(
                    width: 20,
                    height: 20,
                    child: CircularProgressIndicator(strokeWidth: 2),
                  ),
                  SizedBox(width: 8),
                  Text('处理中...'),
                ],
              )
            : const Text('开始处理'),
      ),
    );
  }
  
  Widget _buildResult() {
    return Card(
      child: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Row(
              children: [
                Icon(
                  _lastResult!.success ? Icons.check_circle : Icons.error,
                  color: _lastResult!.success ? Colors.green : Colors.red,
                ),
                const SizedBox(width: 8),
                Text(
                  _lastResult!.success ? '处理成功' : '处理失败',
                  style: Theme.of(context).textTheme.titleLarge?.copyWith(
                    color: _lastResult!.success ? Colors.green : Colors.red,
                  ),
                ),
              ],
            ),
            const SizedBox(height: 16),
            if (_lastResult!.success && _lastResult!.data != null) ..[
              Text(
                '处理结果:',
                style: Theme.of(context).textTheme.titleMedium,
              ),
              const SizedBox(height: 8),
              Container(
                width: double.infinity,
                padding: const EdgeInsets.all(12),
                decoration: BoxDecoration(
                  color: Colors.grey.shade100,
                  borderRadius: BorderRadius.circular(8),
                  border: Border.all(color: Colors.grey.shade300),
                ),
                child: SingleChildScrollView(
                  scrollDirection: Axis.horizontal,
                  child: Text(
                    _lastResult!.data.toString(),
                    style: const TextStyle(fontFamily: 'monospace'),
                  ),
                ),
              ),
            ],
            if (!_lastResult!.success && _lastResult!.error != null) ..[
              Text(
                '错误信息:',
                style: Theme.of(context).textTheme.titleMedium?.copyWith(
                  color: Colors.red,
                ),
              ),
              const SizedBox(height: 8),
              Text(
                _lastResult!.error!,
                style: const TextStyle(color: Colors.red),
              ),
            ],
            const SizedBox(height: 16),
            Text(
              '处理时间: ${_lastResult!.processingTime.inMilliseconds}ms',
              style: Theme.of(context).textTheme.bodyMedium,
            ),
          ],
        ),
      ),
    );
  }
  
  Future<void> _processData() async {
    setState(() {
      _isProcessing = true;
      _lastResult = null;
    });
    
    try {
      // 解析输入数据
      final inputText = _dataController.text.trim();
      List<dynamic> rawData;
      
      if (inputText.isEmpty) {
        rawData = _sampleData;
      } else {
        // 简单的数据解析
        try {
          rawData = _parseData(inputText);
        } catch (e) {
          rawData = [inputText];
        }
      }
      
      final result = await _manager.processData(rawData);
      
      setState(() {
        _lastResult = result;
      });
    } catch (e) {
      setState(() {
        _lastResult = ProcessingResult(
          success: false,
          error: '处理异常: $e',
          processingTime: Duration.zero,
        );
      });
    } finally {
      setState(() {
        _isProcessing = false;
      });
    }
  }
  
  List<dynamic> _parseData(String input) {
    // 简化的数据解析
    if (input.startsWith('[') && input.endsWith(']')) {
      final content = input.substring(1, input.length - 1);
      return content.split(',').map((s) {
        final trimmed = s.trim();
        if (int.tryParse(trimmed) != null) {
          return int.parse(trimmed);
        } else if (double.tryParse(trimmed) != null) {
          return double.parse(trimmed);
        }
        return trimmed;
      }).toList();
    }
    return [input];
  }
}
```

## 🛠 测试和调试

### 1. 策略模式测试

```dart
import 'package:flutter_test/flutter_test.dart';
import 'package:mockito/mockito.dart';

// Mock策略
class MockStrategy extends Mock implements Strategy {}

void main() {
  group('Strategy Pattern Tests', () {
    late TextProcessor processor;
    late MockStrategy mockStrategy;
    
    setUp(() {
      processor = TextProcessor();
      mockStrategy = MockStrategy();
    });
    
    test('应该能够设置策略', () {
      processor.setStrategy(mockStrategy);
      
      expect(processor.getCurrentStrategyType(), equals('MockStrategy'));
    });
    
    test('应该能够执行策略', () {
      when(mockStrategy.execute('test')).thenReturn('PROCESSED');
      
      processor.setStrategy(mockStrategy);
      final result = processor.processText('test');
      
      expect(result, equals('PROCESSED'));
      verify(mockStrategy.execute('test')).called(1);
    });
    
    test('未设置策略时应该抛出异常', () {
      expect(
        () => processor.processText('test'),
        throwsA(isA<StateError>()),
      );
    });
    
    test('不同策略应该产生不同结果', () {
      final upperStrategy = UpperCaseStrategy();
      final lowerStrategy = LowerCaseStrategy();
      const text = 'Hello World';
      
      processor.setStrategy(upperStrategy);
      final upperResult = processor.processText(text);
      
      processor.setStrategy(lowerStrategy);
      final lowerResult = processor.processText(text);
      
      expect(upperResult, equals('HELLO WORLD'));
      expect(lowerResult, equals('hello world'));
      expect(upperResult, isNot(equals(lowerResult)));
    });
  });
  
  group('Payment Strategy Tests', () {
    late PaymentManager paymentManager;
    
    setUp(() {
      paymentManager = PaymentManager();
    });
    
    test('应该能够获取可用支付策略', () {
      final strategies = paymentManager.availableStrategies;
      
      expect(strategies.length, greaterThan(0));
      expect(strategies.every((s) => s.isAvailable), isTrue);
    });
    
    test('应该能够设置支付策略', () {
      paymentManager.setPaymentStrategy('alipay');
      
      expect(paymentManager.currentStrategy, isA<AlipayStrategy>());
    });
    
    test('支付处理应该返回结果', () async {
      paymentManager.setPaymentStrategy('alipay');
      
      final result = await paymentManager.processPayment(
        100.0,
        {'account': 'test@example.com'},
      );
      
      expect(result, isA<PaymentResult>());
      expect(result.success, isA<bool>());
    });
  });
}
```

### 2. 策略调试器

```dart
class StrategyDebugger {
  static final Map<String, List<String>> _strategyLogs = {};
  static final Map<String, int> _executionCounts = {};
  static final Map<String, Duration> _totalExecutionTimes = {};
  
  // 记录策略执行
  static void logStrategyExecution(
    String strategyName,
    String operation,
    Duration executionTime,
    [String? details]
  ) {
    final timestamp = DateTime.now().toIso8601String();
    final logEntry = '[$timestamp] $operation (${executionTime.inMilliseconds}ms)${details != null ? ': $details' : ''}';
    
    _strategyLogs.putIfAbsent(strategyName, () => []).add(logEntry);
    _executionCounts[strategyName] = (_executionCounts[strategyName] ?? 0) + 1;
    _totalExecutionTimes[strategyName] = 
        (_totalExecutionTimes[strategyName] ?? Duration.zero) + executionTime;
  }
  
  // 获取策略统计
  static Map<String, dynamic> getStrategyStats() {
    final stats = <String, dynamic>{};
    
    for (final strategyName in _strategyLogs.keys) {
      final executionCount = _executionCounts[strategyName] ?? 0;
      final totalTime = _totalExecutionTimes[strategyName] ?? Duration.zero;
      
      stats[strategyName] = {
        'executionCount': executionCount,
        'totalExecutionTime': totalTime.inMilliseconds,
        'averageExecutionTime': executionCount > 0 
            ? totalTime.inMilliseconds / executionCount 
            : 0,
        'logs': _strategyLogs[strategyName],
      };
    }
    
    return stats;
  }
  
  // 打印调试信息
  static void printDebugInfo() {
    final stats = getStrategyStats();
    print('=== Strategy Debug Info ===');
    
    for (final entry in stats.entries) {
      print('${entry.key}:');
      print('  执行次数: ${entry.value['executionCount']}');
      print('  总执行时间: ${entry.value['totalExecutionTime']}ms');
      print('  平均执行时间: ${entry.value['averageExecutionTime'].toStringAsFixed(2)}ms');
      print('  最近执行记录:');
      
      final logs = entry.value['logs'] as List<String>;
      for (final log in logs.take(3)) {
        print('    $log');
      }
      print('');
    }
  }
  
  // 重置统计
  static void reset() {
    _strategyLogs.clear();
    _executionCounts.clear();
    _totalExecutionTimes.clear();
  }
}

// 带调试功能的策略基类
abstract class DebuggableStrategy implements Strategy {
  @override
  String execute(String data) {
    final stopwatch = Stopwatch()..start();
    
    try {
      final result = doExecute(data);
      stopwatch.stop();
      
      StrategyDebugger.logStrategyExecution(
        runtimeType.toString(),
        'execute',
        stopwatch.elapsed,
        'input: "$data", output: "$result"',
      );
      
      return result;
    } catch (e) {
      stopwatch.stop();
      
      StrategyDebugger.logStrategyExecution(
        runtimeType.toString(),
        'execute_error',
        stopwatch.elapsed,
        'input: "$data", error: $e',
      );
      
      rethrow;
    }
  }
  
  // 子类需要实现的实际执行方法
  String doExecute(String data);
}
```

## 🔒 最佳实践

### 1. 设计原则
- **开闭原则**: 对扩展开放，对修改关闭
- **单一职责**: 每个策略只负责一种算法
- **依赖倒置**: 依赖抽象而不是具体实现
- **里氏替换**: 策略可以相互替换

### 2. 实现建议
- **策略接口设计**: 保持接口简洁明确
- **策略注册**: 使用注册机制管理策略
- **策略选择**: 提供灵活的策略选择机制
- **错误处理**: 处理策略执行中的异常

### 3. 使用场景
- **算法选择**: 根据条件选择不同算法
- **支付处理**: 多种支付方式选择
- **主题切换**: 应用主题动态切换
- **数据处理**: 不同格式的数据处理

### 4. 注意事项
- **策略数量**: 避免策略类过多
- **客户端复杂度**: 客户端需要了解策略差异
- **性能考虑**: 策略切换的性能开销
- **状态管理**: 策略可能需要维护状态

策略模式是Flutter开发中非常实用的设计模式，它提供了灵活的算法选择机制，使应用能够根据不同情况采用不同的处理策略，提高了代码的可扩展性和可维护性。