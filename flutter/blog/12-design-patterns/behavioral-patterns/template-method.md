# Flutter 模板方法模式 (Template Method Pattern)

## 模式概述

### 定义
模板方法模式定义一个操作中的算法骨架，而将一些步骤延迟到子类中。模板方法使得子类可以不改变一个算法的结构即可重定义该算法的某些特定步骤。

### 适用场景
- 一次性实现一个算法的不变的部分，并将可变的行为留给子类来实现
- 各子类中公共的行为应被提取出来并集中到一个公共父类中以避免代码重复
- 控制子类扩展，模板方法只在特定点调用钩子操作，这样就只允许在这些点进行扩展
- 需要通过子类来决定如何实现算法中的某些步骤

### 优点
- 封装不变部分，扩展可变部分
- 提取公共代码，便于维护
- 行为由父类控制，子类实现
- 符合开闭原则

### 缺点
- 每一个不同的实现都需要一个子类来实现，导致类的个数增加
- 类数量的增加，间接地增加了系统实现的复杂度
- 继承关系自身缺点，如果父类添加新的抽象方法，所有子类都要改一遍

## 基础实现

### 传统模板方法模式

```dart
// 抽象模板类
abstract class AbstractTemplate {
  // 模板方法 - 定义算法骨架
  void templateMethod() {
    step1();
    step2();
    if (hook()) {
      step3();
    }
    step4();
  }
  
  // 具体方法 - 已实现的步骤
  void step1() {
    print('执行步骤1：通用操作');
  }
  
  // 抽象方法 - 子类必须实现
  void step2();
  void step4();
  
  // 可选步骤 - 子类可以选择实现
  void step3() {
    print('执行步骤3：默认实现');
  }
  
  // 钩子方法 - 子类可以控制算法流程
  bool hook() {
    return true;
  }
}

// 具体实现类A
class ConcreteTemplateA extends AbstractTemplate {
  @override
  void step2() {
    print('执行步骤2：实现A的特定操作');
  }
  
  @override
  void step4() {
    print('执行步骤4：实现A的结束操作');
  }
  
  @override
  bool hook() {
    return false; // 跳过步骤3
  }
}

// 具体实现类B
class ConcreteTemplateB extends AbstractTemplate {
  @override
  void step2() {
    print('执行步骤2：实现B的特定操作');
  }
  
  @override
  void step3() {
    print('执行步骤3：实现B的自定义操作');
  }
  
  @override
  void step4() {
    print('执行步骤4：实现B的结束操作');
  }
}
```

### 函数式模板方法

```dart
// 函数式模板方法
class FunctionalTemplate {
  final void Function() step1;
  final void Function() step2;
  final void Function()? step3;
  final void Function() step4;
  final bool Function() shouldExecuteStep3;
  
  FunctionalTemplate({
    required this.step1,
    required this.step2,
    this.step3,
    required this.step4,
    this.shouldExecuteStep3 = _defaultHook,
  });
  
  static bool _defaultHook() => true;
  
  void execute() {
    step1();
    step2();
    if (shouldExecuteStep3() && step3 != null) {
      step3!();
    }
    step4();
  }
}

// 使用示例
void useFunctionalTemplate() {
  final template = FunctionalTemplate(
    step1: () => print('通用步骤1'),
    step2: () => print('自定义步骤2'),
    step3: () => print('可选步骤3'),
    step4: () => print('结束步骤4'),
    shouldExecuteStep3: () => true,
  );
  
  template.execute();
}
```

## Flutter 应用实例

### 数据加载模板

```dart
// 抽象数据加载器
abstract class DataLoader<T> {
  // 模板方法 - 定义数据加载流程
  Future<T?> loadData() async {
    try {
      onLoadStart();
      
      // 检查缓存
      final cachedData = await getCachedData();
      if (cachedData != null && isCacheValid(cachedData)) {
        onLoadSuccess(cachedData);
        return cachedData;
      }
      
      // 从网络加载
      final networkData = await fetchFromNetwork();
      if (networkData != null) {
        await cacheData(networkData);
        onLoadSuccess(networkData);
        return networkData;
      }
      
      onLoadEmpty();
      return null;
    } catch (error) {
      onLoadError(error);
      return null;
    } finally {
      onLoadComplete();
    }
  }
  
  // 具体方法 - 通用实现
  void onLoadStart() {
    print('开始加载数据...');
  }
  
  void onLoadComplete() {
    print('数据加载完成');
  }
  
  // 抽象方法 - 子类必须实现
  Future<T?> getCachedData();
  Future<T?> fetchFromNetwork();
  Future<void> cacheData(T data);
  
  // 钩子方法 - 子类可以重写
  bool isCacheValid(T cachedData) {
    return true; // 默认认为缓存有效
  }
  
  void onLoadSuccess(T data) {
    print('数据加载成功');
  }
  
  void onLoadError(dynamic error) {
    print('数据加载失败: $error');
  }
  
  void onLoadEmpty() {
    print('没有数据');
  }
}

// 用户数据加载器
class UserDataLoader extends DataLoader<User> {
  final ApiService _apiService;
  final CacheService _cacheService;
  
  UserDataLoader(this._apiService, this._cacheService);
  
  @override
  Future<User?> getCachedData() async {
    return await _cacheService.getUser();
  }
  
  @override
  Future<User?> fetchFromNetwork() async {
    return await _apiService.fetchUser();
  }
  
  @override
  Future<void> cacheData(User data) async {
    await _cacheService.saveUser(data);
  }
  
  @override
  bool isCacheValid(User cachedData) {
    // 检查缓存是否在1小时内
    final now = DateTime.now();
    final cacheTime = cachedData.lastUpdated;
    return now.difference(cacheTime).inHours < 1;
  }
  
  @override
  void onLoadSuccess(User data) {
    super.onLoadSuccess(data);
    print('用户数据加载成功: ${data.name}');
  }
}

// 文章数据加载器
class ArticleDataLoader extends DataLoader<List<Article>> {
  final ApiService _apiService;
  final CacheService _cacheService;
  final int page;
  
  ArticleDataLoader(this._apiService, this._cacheService, this.page);
  
  @override
  Future<List<Article>?> getCachedData() async {
    return await _cacheService.getArticles(page);
  }
  
  @override
  Future<List<Article>?> fetchFromNetwork() async {
    return await _apiService.fetchArticles(page);
  }
  
  @override
  Future<void> cacheData(List<Article> data) async {
    await _cacheService.saveArticles(page, data);
  }
  
  @override
  bool isCacheValid(List<Article> cachedData) {
    // 文章列表缓存5分钟
    return DateTime.now().difference(
      cachedData.first.lastUpdated
    ).inMinutes < 5;
  }
  
  @override
  void onLoadEmpty() {
    super.onLoadEmpty();
    print('第$page页没有更多文章');
  }
}
```

### Widget 构建模板

```dart
// 抽象页面构建器
abstract class PageBuilder extends StatelessWidget {
  const PageBuilder({Key? key}) : super(key: key);
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: buildAppBar(context),
      body: buildBody(context),
      floatingActionButton: buildFloatingActionButton(context),
      bottomNavigationBar: buildBottomNavigationBar(context),
      drawer: buildDrawer(context),
    );
  }
  
  // 抽象方法 - 子类必须实现
  Widget buildBody(BuildContext context);
  
  // 钩子方法 - 子类可以选择实现
  PreferredSizeWidget? buildAppBar(BuildContext context) {
    return AppBar(
      title: Text(getTitle()),
      backgroundColor: getAppBarColor(),
      actions: buildAppBarActions(context),
    );
  }
  
  Widget? buildFloatingActionButton(BuildContext context) {
    return null;
  }
  
  Widget? buildBottomNavigationBar(BuildContext context) {
    return null;
  }
  
  Widget? buildDrawer(BuildContext context) {
    return null;
  }
  
  List<Widget>? buildAppBarActions(BuildContext context) {
    return null;
  }
  
  // 默认实现方法
  String getTitle() {
    return '默认标题';
  }
  
  Color getAppBarColor() {
    return Theme.of(context).primaryColor;
  }
}

// 具体页面实现
class HomePage extends PageBuilder {
  const HomePage({Key? key}) : super(key: key);
  
  @override
  Widget buildBody(BuildContext context) {
    return Center(
      child: Column(
        mainAxisAlignment: MainAxisAlignment.center,
        children: [
          Text('欢迎来到首页'),
          ElevatedButton(
            onPressed: () {},
            child: Text('开始使用'),
          ),
        ],
      ),
    );
  }
  
  @override
  String getTitle() {
    return '首页';
  }
  
  @override
  Widget? buildFloatingActionButton(BuildContext context) {
    return FloatingActionButton(
      onPressed: () {},
      child: Icon(Icons.add),
    );
  }
  
  @override
  List<Widget>? buildAppBarActions(BuildContext context) {
    return [
      IconButton(
        icon: Icon(Icons.search),
        onPressed: () {},
      ),
      IconButton(
        icon: Icon(Icons.settings),
        onPressed: () {},
      ),
    ];
  }
}

// 设置页面实现
class SettingsPage extends PageBuilder {
  const SettingsPage({Key? key}) : super(key: key);
  
  @override
  Widget buildBody(BuildContext context) {
    return ListView(
      children: [
        ListTile(
          title: Text('账户设置'),
          leading: Icon(Icons.account_circle),
          onTap: () {},
        ),
        ListTile(
          title: Text('通知设置'),
          leading: Icon(Icons.notifications),
          onTap: () {},
        ),
        ListTile(
          title: Text('隐私设置'),
          leading: Icon(Icons.privacy_tip),
          onTap: () {},
        ),
      ],
    );
  }
  
  @override
  String getTitle() {
    return '设置';
  }
  
  @override
  Color getAppBarColor() {
    return Colors.grey[800]!;
  }
}
```

### 表单验证模板

```dart
// 抽象表单验证器
abstract class FormValidator {
  // 模板方法 - 定义验证流程
  ValidationResult validate(Map<String, dynamic> formData) {
    final errors = <String, String>{};
    
    // 预处理
    final processedData = preprocessData(formData);
    
    // 基础验证
    final basicErrors = performBasicValidation(processedData);
    errors.addAll(basicErrors);
    
    // 业务验证
    if (errors.isEmpty) {
      final businessErrors = performBusinessValidation(processedData);
      errors.addAll(businessErrors);
    }
    
    // 自定义验证
    if (errors.isEmpty && shouldPerformCustomValidation()) {
      final customErrors = performCustomValidation(processedData);
      errors.addAll(customErrors);
    }
    
    // 后处理
    postProcess(processedData, errors);
    
    return ValidationResult(
      isValid: errors.isEmpty,
      errors: errors,
      processedData: processedData,
    );
  }
  
  // 具体方法 - 通用实现
  Map<String, dynamic> preprocessData(Map<String, dynamic> data) {
    final processed = Map<String, dynamic>.from(data);
    // 去除空白字符
    processed.forEach((key, value) {
      if (value is String) {
        processed[key] = value.trim();
      }
    });
    return processed;
  }
  
  void postProcess(Map<String, dynamic> data, Map<String, String> errors) {
    // 记录验证日志
    print('表单验证完成，错误数量: ${errors.length}');
  }
  
  // 抽象方法 - 子类必须实现
  Map<String, String> performBasicValidation(Map<String, dynamic> data);
  Map<String, String> performBusinessValidation(Map<String, dynamic> data);
  
  // 钩子方法 - 子类可以重写
  bool shouldPerformCustomValidation() {
    return false;
  }
  
  Map<String, String> performCustomValidation(Map<String, dynamic> data) {
    return {};
  }
}

// 用户注册表单验证器
class UserRegistrationValidator extends FormValidator {
  @override
  Map<String, String> performBasicValidation(Map<String, dynamic> data) {
    final errors = <String, String>{};
    
    // 用户名验证
    final username = data['username'] as String?;
    if (username == null || username.isEmpty) {
      errors['username'] = '用户名不能为空';
    } else if (username.length < 3) {
      errors['username'] = '用户名至少3个字符';
    }
    
    // 邮箱验证
    final email = data['email'] as String?;
    if (email == null || email.isEmpty) {
      errors['email'] = '邮箱不能为空';
    } else if (!RegExp(r'^[\w-\.]+@([\w-]+\.)+[\w-]{2,4}$').hasMatch(email)) {
      errors['email'] = '邮箱格式不正确';
    }
    
    // 密码验证
    final password = data['password'] as String?;
    if (password == null || password.isEmpty) {
      errors['password'] = '密码不能为空';
    } else if (password.length < 6) {
      errors['password'] = '密码至少6个字符';
    }
    
    return errors;
  }
  
  @override
  Map<String, String> performBusinessValidation(Map<String, dynamic> data) {
    final errors = <String, String>{};
    
    // 检查用户名是否已存在
    final username = data['username'] as String;
    if (isUsernameExists(username)) {
      errors['username'] = '用户名已存在';
    }
    
    // 检查邮箱是否已注册
    final email = data['email'] as String;
    if (isEmailRegistered(email)) {
      errors['email'] = '邮箱已被注册';
    }
    
    return errors;
  }
  
  @override
  bool shouldPerformCustomValidation() {
    return true;
  }
  
  @override
  Map<String, String> performCustomValidation(Map<String, dynamic> data) {
    final errors = <String, String>{};
    
    // 密码强度验证
    final password = data['password'] as String;
    if (!hasUpperCase(password) || !hasLowerCase(password) || !hasDigit(password)) {
      errors['password'] = '密码必须包含大小写字母和数字';
    }
    
    return errors;
  }
  
  // 辅助方法
  bool isUsernameExists(String username) {
    // 模拟检查用户名是否存在
    return false;
  }
  
  bool isEmailRegistered(String email) {
    // 模拟检查邮箱是否已注册
    return false;
  }
  
  bool hasUpperCase(String str) => str.contains(RegExp(r'[A-Z]'));
  bool hasLowerCase(String str) => str.contains(RegExp(r'[a-z]'));
  bool hasDigit(String str) => str.contains(RegExp(r'[0-9]'));
}

// 验证结果类
class ValidationResult {
  final bool isValid;
  final Map<String, String> errors;
  final Map<String, dynamic> processedData;
  
  ValidationResult({
    required this.isValid,
    required this.errors,
    required this.processedData,
  });
}
```

## 测试和调试

### 模板方法测试

```dart
import 'package:flutter_test/flutter_test.dart';

void main() {
  group('模板方法模式测试', () {
    test('具体模板A应该跳过步骤3', () {
      final template = ConcreteTemplateA();
      // 由于无法直接测试控制台输出，这里测试钩子方法
      expect(template.hook(), false);
    });
    
    test('具体模板B应该执行步骤3', () {
      final template = ConcreteTemplateB();
      expect(template.hook(), true);
    });
  });
  
  group('表单验证器测试', () {
    late UserRegistrationValidator validator;
    
    setUp(() {
      validator = UserRegistrationValidator();
    });
    
    test('空表单应该验证失败', () {
      final result = validator.validate({});
      expect(result.isValid, false);
      expect(result.errors.length, greaterThan(0));
    });
    
    test('有效表单应该验证成功', () {
      final formData = {
        'username': 'testuser',
        'email': 'test@example.com',
        'password': 'Password123',
      };
      
      final result = validator.validate(formData);
      expect(result.isValid, true);
      expect(result.errors.isEmpty, true);
    });
    
    test('无效邮箱应该验证失败', () {
      final formData = {
        'username': 'testuser',
        'email': 'invalid-email',
        'password': 'Password123',
      };
      
      final result = validator.validate(formData);
      expect(result.isValid, false);
      expect(result.errors['email'], '邮箱格式不正确');
    });
    
    test('弱密码应该验证失败', () {
      final formData = {
        'username': 'testuser',
        'email': 'test@example.com',
        'password': 'weak',
      };
      
      final result = validator.validate(formData);
      expect(result.isValid, false);
      expect(result.errors.containsKey('password'), true);
    });
  });
}
```

### 模板方法调试器

```dart
class TemplateMethodDebugger {
  static final List<MethodCall> _methodCalls = [];
  
  static void logMethodCall(String className, String methodName, [Map<String, dynamic>? params]) {
    final call = MethodCall(
      className: className,
      methodName: methodName,
      params: params ?? {},
      timestamp: DateTime.now(),
    );
    _methodCalls.add(call);
    print('调用方法: $className.$methodName');
  }
  
  static List<MethodCall> getMethodCalls() => List.unmodifiable(_methodCalls);
  
  static void clearMethodCalls() => _methodCalls.clear();
  
  static void printExecutionFlow() {
    print('=== 模板方法执行流程 ===');
    for (final call in _methodCalls) {
      print('${call.timestamp}: ${call.className}.${call.methodName}');
      if (call.params.isNotEmpty) {
        print('  参数: ${call.params}');
      }
    }
  }
}

class MethodCall {
  final String className;
  final String methodName;
  final Map<String, dynamic> params;
  final DateTime timestamp;
  
  MethodCall({
    required this.className,
    required this.methodName,
    required this.params,
    required this.timestamp,
  });
}

// 带调试功能的模板类
abstract class DebuggableTemplate {
  void templateMethod() {
    TemplateMethodDebugger.logMethodCall(runtimeType.toString(), 'templateMethod');
    
    step1();
    step2();
    if (hook()) {
      step3();
    }
    step4();
  }
  
  void step1() {
    TemplateMethodDebugger.logMethodCall(runtimeType.toString(), 'step1');
    print('执行步骤1');
  }
  
  void step2();
  void step3() {
    TemplateMethodDebugger.logMethodCall(runtimeType.toString(), 'step3');
    print('执行步骤3');
  }
  void step4();
  
  bool hook() {
    TemplateMethodDebugger.logMethodCall(runtimeType.toString(), 'hook');
    return true;
  }
}
```

## 最佳实践

### 设计原则

1. **单一职责原则**
   - 模板方法只负责定义算法骨架
   - 每个步骤方法只负责特定的功能

2. **开闭原则**
   - 对扩展开放：可以通过子类扩展新的实现
   - 对修改封闭：不需要修改模板方法的结构

3. **里氏替换原则**
   - 所有子类都可以替换父类
   - 子类的实现不应该破坏模板方法的约定

### 实现建议

1. **合理设计模板方法**
   ```dart
   // 好的做法：清晰的步骤划分
   abstract class DataProcessor {
     Future<void> processData() async {
       await validateInput();
       await preprocessData();
       await executeProcessing();
       await postprocessData();
       await saveResults();
     }
   }
   
   // 避免：步骤过于细碎或过于粗糙
   abstract class BadDataProcessor {
     Future<void> processData() async {
       await doEverything(); // 太粗糙
     }
   }
   ```

2. **使用钩子方法**
   ```dart
   abstract class FlexibleTemplate {
     void templateMethod() {
       step1();
       if (shouldExecuteStep2()) {
         step2();
       }
       step3();
     }
     
     // 钩子方法提供灵活性
     bool shouldExecuteStep2() => true;
   }
   ```

3. **提供默认实现**
   ```dart
   abstract class UserFriendlyTemplate {
     void templateMethod() {
       requiredStep();
       optionalStep();
     }
     
     // 必须实现的方法
     void requiredStep();
     
     // 提供默认实现的可选方法
     void optionalStep() {
       // 默认什么都不做
     }
   }
   ```

### 使用场景

1. **适合使用模板方法的场景**
   - 数据处理流程
   - 表单验证流程
   - 页面渲染流程
   - 文件处理流程
   - 网络请求流程

2. **不适合使用模板方法的场景**
   - 算法步骤经常变化
   - 子类需要完全不同的算法
   - 简单的一次性操作

### 注意事项

1. **避免过度设计**
   - 不要为了使用模式而使用模式
   - 确保真的有公共的算法骨架

2. **控制继承层次**
   - 避免过深的继承层次
   - 考虑使用组合替代继承

3. **文档化约定**
   - 清楚地文档化每个步骤的职责
   - 说明钩子方法的用途

4. **测试覆盖**
   - 测试模板方法的完整流程
   - 测试各个步骤的独立功能

模板方法模式在 Flutter 开发中非常有用，特别是在处理有固定流程但具体实现可能不同的场景。通过合理使用模板方法模式，可以减少代码重复，提高代码的可维护性和可扩展性。