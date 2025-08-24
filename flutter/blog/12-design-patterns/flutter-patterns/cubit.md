# Flutter Cubit 模式

## 模式概述

### 定义
Cubit 是 BLoC 库的一个简化版本，它是一个可以管理任何类型状态的类。Cubit 只需要一个函数来输出新状态，相比 BLoC 模式更加简单直接，适用于不需要复杂事件处理的场景。

### 适用场景
- 简单的状态管理需求
- 不需要复杂事件流的应用
- 快速原型开发
- 学习状态管理的入门场景
- 局部组件状态管理

### 优点
- 学习曲线平缓
- 代码量少，实现简单
- 性能开销小
- 易于理解和维护
- 支持时间旅行调试
- 与 BLoC 生态系统兼容

### 缺点
- 功能相对简单
- 不适合复杂的业务逻辑
- 缺少事件驱动的优势
- 对于大型应用可能不够灵活

## 基础实现

### 依赖配置

```yaml
# pubspec.yaml
dependencies:
  flutter:
    sdk: flutter
  flutter_bloc: ^8.1.3
  equatable: ^2.0.5
  
dev_dependencies:
  flutter_test:
    sdk: flutter
  bloc_test: ^9.1.4
```

### 简单 Cubit 实现

```dart
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import 'package:equatable/equatable.dart';

// 计数器状态
class CounterState extends Equatable {
  final int count;
  
  const CounterState({required this.count});
  
  @override
  List<Object> get props => [count];
  
  CounterState copyWith({int? count}) {
    return CounterState(count: count ?? this.count);
  }
}

// 计数器 Cubit
class CounterCubit extends Cubit<CounterState> {
  CounterCubit() : super(const CounterState(count: 0));
  
  void increment() {
    emit(state.copyWith(count: state.count + 1));
  }
  
  void decrement() {
    emit(state.copyWith(count: state.count - 1));
  }
  
  void reset() {
    emit(const CounterState(count: 0));
  }
  
  void setValue(int value) {
    emit(CounterState(count: value));
  }
}

// 应用入口
class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Cubit Demo',
      home: BlocProvider(
        create: (context) => CounterCubit(),
        child: CounterPage(),
      ),
    );
  }
}

// 计数器页面
class CounterPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Counter Cubit'),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Text(
              'Count:',
              style: TextStyle(fontSize: 20),
            ),
            BlocBuilder<CounterCubit, CounterState>(
              builder: (context, state) {
                return Text(
                  '${state.count}',
                  style: TextStyle(fontSize: 48, fontWeight: FontWeight.bold),
                );
              },
            ),
            SizedBox(height: 20),
            Row(
              mainAxisAlignment: MainAxisAlignment.spaceEvenly,
              children: [
                ElevatedButton(
                  onPressed: () {
                    context.read<CounterCubit>().decrement();
                  },
                  child: Text('-'),
                ),
                ElevatedButton(
                  onPressed: () {
                    context.read<CounterCubit>().reset();
                  },
                  child: Text('Reset'),
                ),
                ElevatedButton(
                  onPressed: () {
                    context.read<CounterCubit>().increment();
                  },
                  child: Text('+'),
                ),
              ],
            ),
          ],
        ),
      ),
    );
  }
}
```

### 复杂 Cubit 实现

```dart
// 主题状态
enum ThemeMode { light, dark, system }

class ThemeState extends Equatable {
  final ThemeMode mode;
  final Color primaryColor;
  final bool isDarkMode;
  
  const ThemeState({
    required this.mode,
    required this.primaryColor,
    required this.isDarkMode,
  });
  
  factory ThemeState.initial() {
    return const ThemeState(
      mode: ThemeMode.system,
      primaryColor: Colors.blue,
      isDarkMode: false,
    );
  }
  
  ThemeState copyWith({
    ThemeMode? mode,
    Color? primaryColor,
    bool? isDarkMode,
  }) {
    return ThemeState(
      mode: mode ?? this.mode,
      primaryColor: primaryColor ?? this.primaryColor,
      isDarkMode: isDarkMode ?? this.isDarkMode,
    );
  }
  
  @override
  List<Object> get props => [mode, primaryColor, isDarkMode];
}

// 主题 Cubit
class ThemeCubit extends Cubit<ThemeState> {
  final SharedPreferences _prefs;
  
  ThemeCubit({required SharedPreferences prefs})
      : _prefs = prefs,
        super(ThemeState.initial()) {
    _loadTheme();
  }
  
  void _loadTheme() {
    final modeIndex = _prefs.getInt('theme_mode') ?? 0;
    final colorValue = _prefs.getInt('primary_color') ?? Colors.blue.value;
    final isDark = _prefs.getBool('is_dark_mode') ?? false;
    
    emit(ThemeState(
      mode: ThemeMode.values[modeIndex],
      primaryColor: Color(colorValue),
      isDarkMode: isDark,
    ));
  }
  
  Future<void> setThemeMode(ThemeMode mode) async {
    await _prefs.setInt('theme_mode', mode.index);
    
    bool isDark;
    switch (mode) {
      case ThemeMode.light:
        isDark = false;
        break;
      case ThemeMode.dark:
        isDark = true;
        break;
      case ThemeMode.system:
        isDark = WidgetsBinding.instance.window.platformBrightness == Brightness.dark;
        break;
    }
    
    await _prefs.setBool('is_dark_mode', isDark);
    
    emit(state.copyWith(mode: mode, isDarkMode: isDark));
  }
  
  Future<void> setPrimaryColor(Color color) async {
    await _prefs.setInt('primary_color', color.value);
    emit(state.copyWith(primaryColor: color));
  }
  
  Future<void> toggleDarkMode() async {
    final newMode = state.isDarkMode ? ThemeMode.light : ThemeMode.dark;
    await setThemeMode(newMode);
  }
  
  ThemeData get lightTheme {
    return ThemeData(
      primarySwatch: MaterialColor(
        state.primaryColor.value,
        _generateMaterialColorSwatch(state.primaryColor),
      ),
      brightness: Brightness.light,
      visualDensity: VisualDensity.adaptivePlatformDensity,
    );
  }
  
  ThemeData get darkTheme {
    return ThemeData(
      primarySwatch: MaterialColor(
        state.primaryColor.value,
        _generateMaterialColorSwatch(state.primaryColor),
      ),
      brightness: Brightness.dark,
      visualDensity: VisualDensity.adaptivePlatformDensity,
    );
  }
  
  Map<int, Color> _generateMaterialColorSwatch(Color color) {
    return {
      50: color.withOpacity(0.1),
      100: color.withOpacity(0.2),
      200: color.withOpacity(0.3),
      300: color.withOpacity(0.4),
      400: color.withOpacity(0.5),
      500: color.withOpacity(0.6),
      600: color.withOpacity(0.7),
      700: color.withOpacity(0.8),
      800: color.withOpacity(0.9),
      900: color.withOpacity(1.0),
    };
  }
}

// 主题设置页面
class ThemeSettingsPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Theme Settings'),
      ),
      body: BlocBuilder<ThemeCubit, ThemeState>(
        builder: (context, state) {
          return ListView(
            padding: EdgeInsets.all(16),
            children: [
              // 主题模式选择
              Card(
                child: Padding(
                  padding: EdgeInsets.all(16),
                  child: Column(
                    crossAxisAlignment: CrossAxisAlignment.start,
                    children: [
                      Text(
                        'Theme Mode',
                        style: TextStyle(
                          fontSize: 18,
                          fontWeight: FontWeight.bold,
                        ),
                      ),
                      SizedBox(height: 16),
                      ...ThemeMode.values.map((mode) {
                        return RadioListTile<ThemeMode>(
                          title: Text(_getThemeModeTitle(mode)),
                          value: mode,
                          groupValue: state.mode,
                          onChanged: (value) {
                            if (value != null) {
                              context.read<ThemeCubit>().setThemeMode(value);
                            }
                          },
                        );
                      }).toList(),
                    ],
                  ),
                ),
              ),
              SizedBox(height: 16),
              // 主色调选择
              Card(
                child: Padding(
                  padding: EdgeInsets.all(16),
                  child: Column(
                    crossAxisAlignment: CrossAxisAlignment.start,
                    children: [
                      Text(
                        'Primary Color',
                        style: TextStyle(
                          fontSize: 18,
                          fontWeight: FontWeight.bold,
                        ),
                      ),
                      SizedBox(height: 16),
                      Wrap(
                        spacing: 8,
                        runSpacing: 8,
                        children: _predefinedColors.map((color) {
                          final isSelected = color.value == state.primaryColor.value;
                          return GestureDetector(
                            onTap: () {
                              context.read<ThemeCubit>().setPrimaryColor(color);
                            },
                            child: Container(
                              width: 50,
                              height: 50,
                              decoration: BoxDecoration(
                                color: color,
                                shape: BoxShape.circle,
                                border: isSelected
                                    ? Border.all(color: Colors.white, width: 3)
                                    : null,
                                boxShadow: isSelected
                                    ? [
                                        BoxShadow(
                                          color: Colors.black26,
                                          blurRadius: 4,
                                          offset: Offset(0, 2),
                                        ),
                                      ]
                                    : null,
                              ),
                              child: isSelected
                                  ? Icon(
                                      Icons.check,
                                      color: Colors.white,
                                    )
                                  : null,
                            ),
                          );
                        }).toList(),
                      ),
                    ],
                  ),
                ),
              ),
              SizedBox(height: 16),
              // 快速切换
              Card(
                child: Padding(
                  padding: EdgeInsets.all(16),
                  child: Column(
                    crossAxisAlignment: CrossAxisAlignment.start,
                    children: [
                      Text(
                        'Quick Actions',
                        style: TextStyle(
                          fontSize: 18,
                          fontWeight: FontWeight.bold,
                        ),
                      ),
                      SizedBox(height: 16),
                      SwitchListTile(
                        title: Text('Dark Mode'),
                        subtitle: Text('Toggle between light and dark theme'),
                        value: state.isDarkMode,
                        onChanged: (value) {
                          context.read<ThemeCubit>().toggleDarkMode();
                        },
                      ),
                    ],
                  ),
                ),
              ),
            ],
          );
        },
      ),
    );
  }
  
  String _getThemeModeTitle(ThemeMode mode) {
    switch (mode) {
      case ThemeMode.light:
        return 'Light';
      case ThemeMode.dark:
        return 'Dark';
      case ThemeMode.system:
        return 'System';
    }
  }
  
  static const List<Color> _predefinedColors = [
    Colors.blue,
    Colors.red,
    Colors.green,
    Colors.purple,
    Colors.orange,
    Colors.teal,
    Colors.pink,
    Colors.indigo,
  ];
}
```

## Flutter 应用实例

### 购物车应用

```dart
// 购物车商品模型
class CartItem extends Equatable {
  final String id;
  final String name;
  final double price;
  final int quantity;
  final String imageUrl;
  
  const CartItem({
    required this.id,
    required this.name,
    required this.price,
    required this.quantity,
    required this.imageUrl,
  });
  
  CartItem copyWith({
    String? id,
    String? name,
    double? price,
    int? quantity,
    String? imageUrl,
  }) {
    return CartItem(
      id: id ?? this.id,
      name: name ?? this.name,
      price: price ?? this.price,
      quantity: quantity ?? this.quantity,
      imageUrl: imageUrl ?? this.imageUrl,
    );
  }
  
  double get totalPrice => price * quantity;
  
  @override
  List<Object> get props => [id, name, price, quantity, imageUrl];
}

// 购物车状态
class CartState extends Equatable {
  final List<CartItem> items;
  final double discount;
  final double shippingCost;
  
  const CartState({
    required this.items,
    required this.discount,
    required this.shippingCost,
  });
  
  factory CartState.empty() {
    return const CartState(
      items: [],
      discount: 0.0,
      shippingCost: 0.0,
    );
  }
  
  double get subtotal => items.fold(0.0, (sum, item) => sum + item.totalPrice);
  double get total => subtotal - discount + shippingCost;
  int get itemCount => items.fold(0, (sum, item) => sum + item.quantity);
  
  CartState copyWith({
    List<CartItem>? items,
    double? discount,
    double? shippingCost,
  }) {
    return CartState(
      items: items ?? this.items,
      discount: discount ?? this.discount,
      shippingCost: shippingCost ?? this.shippingCost,
    );
  }
  
  @override
  List<Object> get props => [items, discount, shippingCost];
}

// 购物车 Cubit
class CartCubit extends Cubit<CartState> {
  CartCubit() : super(CartState.empty());
  
  void addItem(CartItem item) {
    final existingIndex = state.items.indexWhere((i) => i.id == item.id);
    
    if (existingIndex >= 0) {
      // 如果商品已存在，增加数量
      final existingItem = state.items[existingIndex];
      final updatedItem = existingItem.copyWith(
        quantity: existingItem.quantity + item.quantity,
      );
      
      final updatedItems = List<CartItem>.from(state.items);
      updatedItems[existingIndex] = updatedItem;
      
      emit(state.copyWith(items: updatedItems));
    } else {
      // 添加新商品
      final updatedItems = List<CartItem>.from(state.items)..add(item);
      emit(state.copyWith(items: updatedItems));
    }
  }
  
  void removeItem(String itemId) {
    final updatedItems = state.items.where((item) => item.id != itemId).toList();
    emit(state.copyWith(items: updatedItems));
  }
  
  void updateQuantity(String itemId, int quantity) {
    if (quantity <= 0) {
      removeItem(itemId);
      return;
    }
    
    final updatedItems = state.items.map((item) {
      if (item.id == itemId) {
        return item.copyWith(quantity: quantity);
      }
      return item;
    }).toList();
    
    emit(state.copyWith(items: updatedItems));
  }
  
  void clearCart() {
    emit(CartState.empty());
  }
  
  void applyDiscount(double discount) {
    emit(state.copyWith(discount: discount));
  }
  
  void setShippingCost(double cost) {
    emit(state.copyWith(shippingCost: cost));
  }
  
  void incrementQuantity(String itemId) {
    final item = state.items.firstWhere((item) => item.id == itemId);
    updateQuantity(itemId, item.quantity + 1);
  }
  
  void decrementQuantity(String itemId) {
    final item = state.items.firstWhere((item) => item.id == itemId);
    updateQuantity(itemId, item.quantity - 1);
  }
}

// 购物车页面
class CartPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Shopping Cart'),
        actions: [
          BlocBuilder<CartCubit, CartState>(
            builder: (context, state) {
              if (state.items.isNotEmpty) {
                return IconButton(
                  icon: Icon(Icons.clear_all),
                  onPressed: () {
                    _showClearCartDialog(context);
                  },
                );
              }
              return SizedBox.shrink();
            },
          ),
        ],
      ),
      body: BlocBuilder<CartCubit, CartState>(
        builder: (context, state) {
          if (state.items.isEmpty) {
            return Center(
              child: Column(
                mainAxisAlignment: MainAxisAlignment.center,
                children: [
                  Icon(
                    Icons.shopping_cart_outlined,
                    size: 64,
                    color: Colors.grey,
                  ),
                  SizedBox(height: 16),
                  Text(
                    'Your cart is empty',
                    style: TextStyle(
                      fontSize: 18,
                      color: Colors.grey,
                    ),
                  ),
                  SizedBox(height: 16),
                  ElevatedButton(
                    onPressed: () {
                      Navigator.of(context).pop();
                    },
                    child: Text('Continue Shopping'),
                  ),
                ],
              ),
            );
          }
          
          return Column(
            children: [
              // 购物车商品列表
              Expanded(
                child: ListView.builder(
                  itemCount: state.items.length,
                  itemBuilder: (context, index) {
                    final item = state.items[index];
                    return CartItemWidget(item: item);
                  },
                ),
              ),
              // 购物车摘要
              CartSummary(),
            ],
          );
        },
      ),
    );
  }
  
  void _showClearCartDialog(BuildContext context) {
    showDialog(
      context: context,
      builder: (dialogContext) => AlertDialog(
        title: Text('Clear Cart'),
        content: Text('Are you sure you want to remove all items from your cart?'),
        actions: [
          TextButton(
            onPressed: () => Navigator.of(dialogContext).pop(),
            child: Text('Cancel'),
          ),
          ElevatedButton(
            onPressed: () {
              context.read<CartCubit>().clearCart();
              Navigator.of(dialogContext).pop();
            },
            child: Text('Clear'),
          ),
        ],
      ),
    );
  }
}

// 购物车商品组件
class CartItemWidget extends StatelessWidget {
  final CartItem item;
  
  const CartItemWidget({Key? key, required this.item}) : super(key: key);
  
  @override
  Widget build(BuildContext context) {
    return Card(
      margin: EdgeInsets.symmetric(horizontal: 16, vertical: 4),
      child: Padding(
        padding: EdgeInsets.all(12),
        child: Row(
          children: [
            // 商品图片
            ClipRRect(
              borderRadius: BorderRadius.circular(8),
              child: Image.network(
                item.imageUrl,
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
            // 商品信息
            Expanded(
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  Text(
                    item.name,
                    style: TextStyle(
                      fontSize: 16,
                      fontWeight: FontWeight.bold,
                    ),
                    maxLines: 2,
                    overflow: TextOverflow.ellipsis,
                  ),
                  SizedBox(height: 4),
                  Text(
                    '\$${item.price.toStringAsFixed(2)}',
                    style: TextStyle(
                      fontSize: 14,
                      color: Colors.grey[600],
                    ),
                  ),
                  SizedBox(height: 8),
                  Text(
                    'Total: \$${item.totalPrice.toStringAsFixed(2)}',
                    style: TextStyle(
                      fontSize: 16,
                      fontWeight: FontWeight.bold,
                      color: Theme.of(context).primaryColor,
                    ),
                  ),
                ],
              ),
            ),
            // 数量控制
            Column(
              children: [
                Row(
                  mainAxisSize: MainAxisSize.min,
                  children: [
                    IconButton(
                      onPressed: () {
                        context.read<CartCubit>().decrementQuantity(item.id);
                      },
                      icon: Icon(Icons.remove_circle_outline),
                      constraints: BoxConstraints(minWidth: 32, minHeight: 32),
                    ),
                    Container(
                      padding: EdgeInsets.symmetric(horizontal: 12, vertical: 4),
                      decoration: BoxDecoration(
                        border: Border.all(color: Colors.grey[300]!),
                        borderRadius: BorderRadius.circular(4),
                      ),
                      child: Text(
                        '${item.quantity}',
                        style: TextStyle(
                          fontSize: 16,
                          fontWeight: FontWeight.bold,
                        ),
                      ),
                    ),
                    IconButton(
                      onPressed: () {
                        context.read<CartCubit>().incrementQuantity(item.id);
                      },
                      icon: Icon(Icons.add_circle_outline),
                      constraints: BoxConstraints(minWidth: 32, minHeight: 32),
                    ),
                  ],
                ),
                SizedBox(height: 8),
                TextButton(
                  onPressed: () {
                    context.read<CartCubit>().removeItem(item.id);
                  },
                  child: Text(
                    'Remove',
                    style: TextStyle(color: Colors.red),
                  ),
                ),
              ],
            ),
          ],
        ),
      ),
    );
  }
}

// 购物车摘要组件
class CartSummary extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return BlocBuilder<CartCubit, CartState>(
      builder: (context, state) {
        return Container(
          padding: EdgeInsets.all(16),
          decoration: BoxDecoration(
            color: Theme.of(context).cardColor,
            boxShadow: [
              BoxShadow(
                color: Colors.black12,
                blurRadius: 4,
                offset: Offset(0, -2),
              ),
            ],
          ),
          child: Column(
            children: [
              Row(
                mainAxisAlignment: MainAxisAlignment.spaceBetween,
                children: [
                  Text('Subtotal (${state.itemCount} items):'),
                  Text('\$${state.subtotal.toStringAsFixed(2)}'),
                ],
              ),
              if (state.discount > 0) ..[
                SizedBox(height: 8),
                Row(
                  mainAxisAlignment: MainAxisAlignment.spaceBetween,
                  children: [
                    Text('Discount:'),
                    Text(
                      '-\$${state.discount.toStringAsFixed(2)}',
                      style: TextStyle(color: Colors.green),
                    ),
                  ],
                ),
              ],
              if (state.shippingCost > 0) ..[
                SizedBox(height: 8),
                Row(
                  mainAxisAlignment: MainAxisAlignment.spaceBetween,
                  children: [
                    Text('Shipping:'),
                    Text('\$${state.shippingCost.toStringAsFixed(2)}'),
                  ],
                ),
              ],
              Divider(),
              Row(
                mainAxisAlignment: MainAxisAlignment.spaceBetween,
                children: [
                  Text(
                    'Total:',
                    style: TextStyle(
                      fontSize: 18,
                      fontWeight: FontWeight.bold,
                    ),
                  ),
                  Text(
                    '\$${state.total.toStringAsFixed(2)}',
                    style: TextStyle(
                      fontSize: 18,
                      fontWeight: FontWeight.bold,
                      color: Theme.of(context).primaryColor,
                    ),
                  ),
                ],
              ),
              SizedBox(height: 16),
              SizedBox(
                width: double.infinity,
                child: ElevatedButton(
                  onPressed: state.items.isNotEmpty
                      ? () {
                          _proceedToCheckout(context);
                        }
                      : null,
                  child: Text('Proceed to Checkout'),
                ),
              ),
            ],
          ),
        );
      },
    );
  }
  
  void _proceedToCheckout(BuildContext context) {
    // 实现结账逻辑
    ScaffoldMessenger.of(context).showSnackBar(
      SnackBar(
        content: Text('Proceeding to checkout...'),
        backgroundColor: Colors.green,
      ),
    );
  }
}
```

### 表单验证应用

```dart
// 表单字段状态
class FormFieldState extends Equatable {
  final String value;
  final String? error;
  final bool isValid;
  
  const FormFieldState({
    required this.value,
    this.error,
    required this.isValid,
  });
  
  factory FormFieldState.initial() {
    return const FormFieldState(
      value: '',
      isValid: false,
    );
  }
  
  FormFieldState copyWith({
    String? value,
    String? error,
    bool? isValid,
  }) {
    return FormFieldState(
      value: value ?? this.value,
      error: error,
      isValid: isValid ?? this.isValid,
    );
  }
  
  @override
  List<Object?> get props => [value, error, isValid];
}

// 注册表单状态
class RegistrationFormState extends Equatable {
  final FormFieldState email;
  final FormFieldState password;
  final FormFieldState confirmPassword;
  final FormFieldState name;
  final bool isSubmitting;
  final String? submitError;
  
  const RegistrationFormState({
    required this.email,
    required this.password,
    required this.confirmPassword,
    required this.name,
    required this.isSubmitting,
    this.submitError,
  });
  
  factory RegistrationFormState.initial() {
    return RegistrationFormState(
      email: FormFieldState.initial(),
      password: FormFieldState.initial(),
      confirmPassword: FormFieldState.initial(),
      name: FormFieldState.initial(),
      isSubmitting: false,
    );
  }
  
  bool get isValid =>
      email.isValid &&
      password.isValid &&
      confirmPassword.isValid &&
      name.isValid;
  
  RegistrationFormState copyWith({
    FormFieldState? email,
    FormFieldState? password,
    FormFieldState? confirmPassword,
    FormFieldState? name,
    bool? isSubmitting,
    String? submitError,
  }) {
    return RegistrationFormState(
      email: email ?? this.email,
      password: password ?? this.password,
      confirmPassword: confirmPassword ?? this.confirmPassword,
      name: name ?? this.name,
      isSubmitting: isSubmitting ?? this.isSubmitting,
      submitError: submitError,
    );
  }
  
  @override
  List<Object?> get props => [
    email,
    password,
    confirmPassword,
    name,
    isSubmitting,
    submitError,
  ];
}

// 注册表单 Cubit
class RegistrationFormCubit extends Cubit<RegistrationFormState> {
  final AuthRepository _authRepository;
  
  RegistrationFormCubit({required AuthRepository authRepository})
      : _authRepository = authRepository,
        super(RegistrationFormState.initial());
  
  void updateEmail(String value) {
    final error = _validateEmail(value);
    final fieldState = FormFieldState(
      value: value,
      error: error,
      isValid: error == null,
    );
    
    emit(state.copyWith(email: fieldState));
  }
  
  void updatePassword(String value) {
    final error = _validatePassword(value);
    final fieldState = FormFieldState(
      value: value,
      error: error,
      isValid: error == null,
    );
    
    emit(state.copyWith(password: fieldState));
    
    // 重新验证确认密码
    if (state.confirmPassword.value.isNotEmpty) {
      updateConfirmPassword(state.confirmPassword.value);
    }
  }
  
  void updateConfirmPassword(String value) {
    final error = _validateConfirmPassword(value, state.password.value);
    final fieldState = FormFieldState(
      value: value,
      error: error,
      isValid: error == null,
    );
    
    emit(state.copyWith(confirmPassword: fieldState));
  }
  
  void updateName(String value) {
    final error = _validateName(value);
    final fieldState = FormFieldState(
      value: value,
      error: error,
      isValid: error == null,
    );
    
    emit(state.copyWith(name: fieldState));
  }
  
  Future<void> submitForm() async {
    if (!state.isValid) return;
    
    emit(state.copyWith(isSubmitting: true, submitError: null));
    
    try {
      await _authRepository.register(
        email: state.email.value,
        password: state.password.value,
        name: state.name.value,
      );
      
      emit(state.copyWith(isSubmitting: false));
      // 注册成功，可以导航到其他页面
    } catch (e) {
      emit(state.copyWith(
        isSubmitting: false,
        submitError: e.toString(),
      ));
    }
  }
  
  String? _validateEmail(String value) {
    if (value.isEmpty) {
      return 'Email is required';
    }
    
    final emailRegex = RegExp(r'^[\w-\.]+@([\w-]+\.)+[\w-]{2,4}$');
    if (!emailRegex.hasMatch(value)) {
      return 'Please enter a valid email address';
    }
    
    return null;
  }
  
  String? _validatePassword(String value) {
    if (value.isEmpty) {
      return 'Password is required';
    }
    
    if (value.length < 8) {
      return 'Password must be at least 8 characters long';
    }
    
    if (!RegExp(r'(?=.*[a-z])').hasMatch(value)) {
      return 'Password must contain at least one lowercase letter';
    }
    
    if (!RegExp(r'(?=.*[A-Z])').hasMatch(value)) {
      return 'Password must contain at least one uppercase letter';
    }
    
    if (!RegExp(r'(?=.*\d)').hasMatch(value)) {
      return 'Password must contain at least one number';
    }
    
    return null;
  }
  
  String? _validateConfirmPassword(String value, String password) {
    if (value.isEmpty) {
      return 'Please confirm your password';
    }
    
    if (value != password) {
      return 'Passwords do not match';
    }
    
    return null;
  }
  
  String? _validateName(String value) {
    if (value.isEmpty) {
      return 'Name is required';
    }
    
    if (value.length < 2) {
      return 'Name must be at least 2 characters long';
    }
    
    return null;
  }
}

// 注册表单页面
class RegistrationFormPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Create Account'),
      ),
      body: BlocProvider(
        create: (context) => RegistrationFormCubit(
          authRepository: context.read<AuthRepository>(),
        ),
        child: RegistrationFormView(),
      ),
    );
  }
}

class RegistrationFormView extends StatefulWidget {
  @override
  _RegistrationFormViewState createState() => _RegistrationFormViewState();
}

class _RegistrationFormViewState extends State<RegistrationFormView> {
  final _formKey = GlobalKey<FormState>();
  final _nameController = TextEditingController();
  final _emailController = TextEditingController();
  final _passwordController = TextEditingController();
  final _confirmPasswordController = TextEditingController();
  
  @override
  Widget build(BuildContext context) {
    return BlocConsumer<RegistrationFormCubit, RegistrationFormState>(
      listener: (context, state) {
        if (state.submitError != null) {
          ScaffoldMessenger.of(context).showSnackBar(
            SnackBar(
              content: Text(state.submitError!),
              backgroundColor: Colors.red,
            ),
          );
        }
      },
      builder: (context, state) {
        return Padding(
          padding: EdgeInsets.all(16),
          child: Form(
            key: _formKey,
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.stretch,
              children: [
                // 姓名字段
                TextFormField(
                  controller: _nameController,
                  decoration: InputDecoration(
                    labelText: 'Full Name',
                    border: OutlineInputBorder(),
                    errorText: state.name.error,
                  ),
                  onChanged: (value) {
                    context.read<RegistrationFormCubit>().updateName(value);
                  },
                ),
                SizedBox(height: 16),
                // 邮箱字段
                TextFormField(
                  controller: _emailController,
                  decoration: InputDecoration(
                    labelText: 'Email',
                    border: OutlineInputBorder(),
                    errorText: state.email.error,
                  ),
                  keyboardType: TextInputType.emailAddress,
                  onChanged: (value) {
                    context.read<RegistrationFormCubit>().updateEmail(value);
                  },
                ),
                SizedBox(height: 16),
                // 密码字段
                TextFormField(
                  controller: _passwordController,
                  decoration: InputDecoration(
                    labelText: 'Password',
                    border: OutlineInputBorder(),
                    errorText: state.password.error,
                  ),
                  obscureText: true,
                  onChanged: (value) {
                    context.read<RegistrationFormCubit>().updatePassword(value);
                  },
                ),
                SizedBox(height: 16),
                // 确认密码字段
                TextFormField(
                  controller: _confirmPasswordController,
                  decoration: InputDecoration(
                    labelText: 'Confirm Password',
                    border: OutlineInputBorder(),
                    errorText: state.confirmPassword.error,
                  ),
                  obscureText: true,
                  onChanged: (value) {
                    context.read<RegistrationFormCubit>().updateConfirmPassword(value);
                  },
                ),
                SizedBox(height: 24),
                // 提交按钮
                ElevatedButton(
                  onPressed: state.isValid && !state.isSubmitting
                      ? () {
                          context.read<RegistrationFormCubit>().submitForm();
                        }
                      : null,
                  child: state.isSubmitting
                      ? SizedBox(
                          height: 20,
                          width: 20,
                          child: CircularProgressIndicator(
                            strokeWidth: 2,
                            valueColor: AlwaysStoppedAnimation<Color>(Colors.white),
                          ),
                        )
                      : Text('Create Account'),
                ),
                SizedBox(height: 16),
                // 表单状态指示器
                if (kDebugMode)
                  Card(
                    child: Padding(
                      padding: EdgeInsets.all(8),
                      child: Column(
                        crossAxisAlignment: CrossAxisAlignment.start,
                        children: [
                          Text('Form Status:', style: TextStyle(fontWeight: FontWeight.bold)),
                          Text('Valid: ${state.isValid}'),
                          Text('Submitting: ${state.isSubmitting}'),
                          Text('Name Valid: ${state.name.isValid}'),
                          Text('Email Valid: ${state.email.isValid}'),
                          Text('Password Valid: ${state.password.isValid}'),
                          Text('Confirm Password Valid: ${state.confirmPassword.isValid}'),
                        ],
                      ),
                    ),
                  ),
              ],
            ),
          ),
        );
      },
    );
  }
  
  @override
  void dispose() {
    _nameController.dispose();
    _emailController.dispose();
    _passwordController.dispose();
    _confirmPasswordController.dispose();
    super.dispose();
  }
}
```

## 测试和调试

### Cubit 测试

```dart
import 'package:bloc_test/bloc_test.dart';
import 'package:flutter_test/flutter_test.dart';

void main() {
  group('CounterCubit', () {
    late CounterCubit counterCubit;
    
    setUp(() {
      counterCubit = CounterCubit();
    });
    
    tearDown(() {
      counterCubit.close();
    });
    
    test('初始状态应该是 CounterState(count: 0)', () {
      expect(counterCubit.state, const CounterState(count: 0));
    });
    
    blocTest<CounterCubit, CounterState>(
      '调用 increment 应该发出 count + 1',
      build: () => counterCubit,
      act: (cubit) => cubit.increment(),
      expect: () => [const CounterState(count: 1)],
    );
    
    blocTest<CounterCubit, CounterState>(
      '调用 decrement 应该发出 count - 1',
      build: () => counterCubit,
      seed: () => const CounterState(count: 1),
      act: (cubit) => cubit.decrement(),
      expect: () => [const CounterState(count: 0)],
    );
    
    blocTest<CounterCubit, CounterState>(
      '调用 reset 应该发出 count = 0',
      build: () => counterCubit,
      seed: () => const CounterState(count: 10),
      act: (cubit) => cubit.reset(),
      expect: () => [const CounterState(count: 0)],
    );
    
    blocTest<CounterCubit, CounterState>(
      '调用 setValue 应该设置指定值',
      build: () => counterCubit,
      act: (cubit) => cubit.setValue(42),
      expect: () => [const CounterState(count: 42)],
    );
  });
  
  group('CartCubit', () {
    late CartCubit cartCubit;
    
    setUp(() {
      cartCubit = CartCubit();
    });
    
    tearDown(() {
      cartCubit.close();
    });
    
    test('初始状态应该是空购物车', () {
      expect(cartCubit.state, CartState.empty());
    });
    
    blocTest<CartCubit, CartState>(
      '添加商品应该更新购物车',
      build: () => cartCubit,
      act: (cubit) => cubit.addItem(
        const CartItem(
          id: '1',
          name: 'Test Item',
          price: 10.0,
          quantity: 1,
          imageUrl: 'test.jpg',
        ),
      ),
      expect: () => [
        predicate<CartState>((state) {
          return state.items.length == 1 &&
                 state.items.first.id == '1' &&
                 state.items.first.name == 'Test Item';
        }),
      ],
    );
    
    blocTest<CartCubit, CartState>(
      '添加相同商品应该增加数量',
      build: () => cartCubit,
      seed: () => CartState(
        items: [
          const CartItem(
            id: '1',
            name: 'Test Item',
            price: 10.0,
            quantity: 1,
            imageUrl: 'test.jpg',
          ),
        ],
        discount: 0.0,
        shippingCost: 0.0,
      ),
      act: (cubit) => cubit.addItem(
        const CartItem(
          id: '1',
          name: 'Test Item',
          price: 10.0,
          quantity: 2,
          imageUrl: 'test.jpg',
        ),
      ),
      expect: () => [
        predicate<CartState>((state) {
          return state.items.length == 1 &&
                 state.items.first.quantity == 3;
        }),
      ],
    );
    
    blocTest<CartCubit, CartState>(
      '清空购物车应该移除所有商品',
      build: () => cartCubit,
      seed: () => CartState(
        items: [
          const CartItem(
            id: '1',
            name: 'Test Item',
            price: 10.0,
            quantity: 1,
            imageUrl: 'test.jpg',
          ),
        ],
        discount: 0.0,
        shippingCost: 0.0,
      ),
      act: (cubit) => cubit.clearCart(),
      expect: () => [CartState.empty()],
    );
  });
}
```

### Cubit 调试器

```dart
class CubitDebugger {
  static final Map<String, CubitStats> _stats = {};
  static final List<CubitStateChange> _stateChanges = [];
  
  static void logStateChange(String cubitName, dynamic previousState, dynamic currentState) {
    final stats = _stats.putIfAbsent(cubitName, () => CubitStats(cubitName));
    stats.incrementStateChange();
    
    final stateChange = CubitStateChange(
      cubitName: cubitName,
      previousState: previousState,
      currentState: currentState,
      timestamp: DateTime.now(),
    );
    _stateChanges.add(stateChange);
    
    if (kDebugMode) {
      print('Cubit状态变化: $cubitName');
      print('  从: ${previousState.runtimeType}');
      print('  到: ${currentState.runtimeType}');
    }
  }
  
  static Map<String, CubitStats> getStats() => Map.unmodifiable(_stats);
  
  static List<CubitStateChange> getStateChanges() => List.unmodifiable(_stateChanges);
  
  static void clearHistory() {
    _stats.clear();
    _stateChanges.clear();
  }
  
  static void printStatistics() {
    print('=== Cubit 统计信息 ===');
    print('Cubit数量: ${_stats.length}');
    print('总状态变化: ${_stateChanges.length}');
    print('\n各Cubit统计:');
    _stats.values.forEach((stats) {
      print('  ${stats.cubitName}:');
      print('    状态变化: ${stats.stateChanges}');
    });
  }
}

class CubitStats {
  final String cubitName;
  int stateChanges = 0;
  
  CubitStats(this.cubitName);
  
  void incrementStateChange() {
    stateChanges++;
  }
}

class CubitStateChange {
  final String cubitName;
  final dynamic previousState;
  final dynamic currentState;
  final DateTime timestamp;
  
  CubitStateChange({
    required this.cubitName,
    required this.previousState,
    required this.currentState,
    required this.timestamp,
  });
}

// 带调试功能的 Cubit 基类
abstract class DebuggableCubit<State> extends Cubit<State> {
  DebuggableCubit(State initialState) : super(initialState);
  
  @override
  void emit(State state) {
    if (kDebugMode) {
      CubitDebugger.logStateChange(
        runtimeType.toString(),
        this.state,
        state,
      );
    }
    super.emit(state);
  }
}
```

## 最佳实践

### 设计原则

1. **保持简单**
   ```dart
   // 好的做法：简单直接的状态管理
   class ToggleCubit extends Cubit<bool> {
     ToggleCubit() : super(false);
     
     void toggle() => emit(!state);
   }
   ```

2. **使用不可变状态**
   ```dart
   // 使用 Equatable 确保状态比较
   class MyState extends Equatable {
     final String value;
     
     const MyState({required this.value});
     
     @override
     List<Object> get props => [value];
   }
   ```

3. **合理的方法命名**
   ```dart
   class UserCubit extends Cubit<User?> {
     UserCubit() : super(null);
     
     void setUser(User user) => emit(user);
     void clearUser() => emit(null);
     void updateUserName(String name) {
       if (state != null) {
         emit(state!.copyWith(name: name));
       }
     }
   }
   ```

### 实现建议

1. **适当的状态粒度**
   ```dart
   // 避免过于复杂的状态
   class SimpleLoadingState extends Equatable {
     final bool isLoading;
     final String? error;
     
     const SimpleLoadingState({required this.isLoading, this.error});
   }
   ```

2. **使用 BlocListener 处理副作用**
   ```dart
   BlocListener<MyCubit, MyState>(
     listener: (context, state) {
       if (state.hasError) {
         ScaffoldMessenger.of(context).showSnackBar(
           SnackBar(content: Text(state.error!)),
         );
       }
     },
     child: MyWidget(),
   )
   ```

3. **依赖注入**
   ```dart
   class DataCubit extends Cubit<DataState> {
     final Repository repository;
     
     DataCubit({required this.repository}) : super(DataState.initial());
   }
   ```

### 注意事项

1. **避免在 Cubit 中直接访问 UI**
2. **合理管理 Cubit 的生命周期**
3. **使用 Repository 模式分离数据层**
4. **编写充分的单元测试**
5. **避免过度复杂的状态设计**
6. **考虑使用 BLoC 处理复杂的业务逻辑**

Cubit 模式提供了简单直接的状态管理方式，特别适合简单到中等复杂度的应用。通过遵循最佳实践，可以构建出清晰、可维护的 Flutter 应用。