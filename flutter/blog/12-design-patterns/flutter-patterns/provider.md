# Flutter Provider 模式 (Provider Pattern)

## 模式概述

### 定义
Provider 是基于 InheritedWidget 构建的状态管理库，它简化了数据在 Widget 树中的传递和管理。Provider 提供了更简洁的 API 和更强大的功能，是 Flutter 官方推荐的状态管理解决方案之一。

### 适用场景
- 中小型应用的状态管理
- 需要在多个页面间共享数据
- 实现响应式编程模式
- 依赖注入和服务定位
- 复杂的业务逻辑管理

### 优点
- 简单易学，API 友好
- 基于 InheritedWidget，性能优秀
- 支持多种 Provider 类型
- 良好的测试支持
- 官方维护，生态完善

### 缺点
- 对于大型应用可能不够灵活
- 嵌套过深时代码可读性下降
- 缺乏时间旅行调试
- 异步状态管理相对复杂

## 基础实现

### 依赖配置

```yaml
# pubspec.yaml
dependencies:
  flutter:
    sdk: flutter
  provider: ^6.0.5
  
dev_dependencies:
  flutter_test:
    sdk: flutter
```

### 简单 Provider 实现

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

// 计数器模型
class CounterModel extends ChangeNotifier {
  int _count = 0;
  
  int get count => _count;
  
  void increment() {
    _count++;
    notifyListeners();
  }
  
  void decrement() {
    _count--;
    notifyListeners();
  }
  
  void reset() {
    _count = 0;
    notifyListeners();
  }
}

// 应用入口
class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return ChangeNotifierProvider(
      create: (context) => CounterModel(),
      child: MaterialApp(
        title: 'Provider Demo',
        home: CounterPage(),
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
        title: Text('Counter'),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Text(
              'Count:',
              style: TextStyle(fontSize: 20),
            ),
            Consumer<CounterModel>(
              builder: (context, counter, child) {
                return Text(
                  '${counter.count}',
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
                    context.read<CounterModel>().decrement();
                  },
                  child: Text('-'),
                ),
                ElevatedButton(
                  onPressed: () {
                    context.read<CounterModel>().reset();
                  },
                  child: Text('Reset'),
                ),
                ElevatedButton(
                  onPressed: () {
                    context.read<CounterModel>().increment();
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

// 使用 Selector 优化性能
class OptimizedCounterDisplay extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Selector<CounterModel, int>(
      selector: (context, counter) => counter.count,
      builder: (context, count, child) {
        print('CounterDisplay rebuilt'); // 只在 count 变化时重建
        return Text(
          '$count',
          style: TextStyle(fontSize: 48),
        );
      },
    );
  }
}
```

### 多 Provider 实现

```dart
// 用户模型
class UserModel extends ChangeNotifier {
  User? _user;
  bool _isLoading = false;
  String? _error;
  
  User? get user => _user;
  bool get isLoading => _isLoading;
  String? get error => _error;
  bool get isLoggedIn => _user != null;
  
  Future<void> login(String email, String password) async {
    _setLoading(true);
    _setError(null);
    
    try {
      final user = await AuthService.login(email, password);
      _user = user;
      notifyListeners();
    } catch (e) {
      _setError(e.toString());
    } finally {
      _setLoading(false);
    }
  }
  
  Future<void> logout() async {
    _setLoading(true);
    
    try {
      await AuthService.logout();
      _user = null;
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
  
  void _setError(String? error) {
    _error = error;
    notifyListeners();
  }
}

// 购物车模型
class CartModel extends ChangeNotifier {
  final List<CartItem> _items = [];
  
  List<CartItem> get items => List.unmodifiable(_items);
  
  int get itemCount => _items.fold(0, (sum, item) => sum + item.quantity);
  
  double get totalPrice => _items.fold(0.0, (sum, item) => sum + item.totalPrice);
  
  void addItem(Product product, {int quantity = 1}) {
    final existingIndex = _items.indexWhere((item) => item.product.id == product.id);
    
    if (existingIndex >= 0) {
      _items[existingIndex] = _items[existingIndex].copyWith(
        quantity: _items[existingIndex].quantity + quantity,
      );
    } else {
      _items.add(CartItem(product: product, quantity: quantity));
    }
    
    notifyListeners();
  }
  
  void removeItem(String productId) {
    _items.removeWhere((item) => item.product.id == productId);
    notifyListeners();
  }
  
  void updateQuantity(String productId, int quantity) {
    if (quantity <= 0) {
      removeItem(productId);
      return;
    }
    
    final index = _items.indexWhere((item) => item.product.id == productId);
    if (index >= 0) {
      _items[index] = _items[index].copyWith(quantity: quantity);
      notifyListeners();
    }
  }
  
  void clear() {
    _items.clear();
    notifyListeners();
  }
}

// 主题模型
class ThemeModel extends ChangeNotifier {
  ThemeMode _themeMode = ThemeMode.system;
  Color _primaryColor = Colors.blue;
  
  ThemeMode get themeMode => _themeMode;
  Color get primaryColor => _primaryColor;
  
  bool get isDarkMode {
    switch (_themeMode) {
      case ThemeMode.dark:
        return true;
      case ThemeMode.light:
        return false;
      case ThemeMode.system:
        return WidgetsBinding.instance.window.platformBrightness == Brightness.dark;
    }
  }
  
  void setThemeMode(ThemeMode mode) {
    _themeMode = mode;
    notifyListeners();
    _saveThemeMode();
  }
  
  void setPrimaryColor(Color color) {
    _primaryColor = color;
    notifyListeners();
    _savePrimaryColor();
  }
  
  void toggleTheme() {
    switch (_themeMode) {
      case ThemeMode.light:
        setThemeMode(ThemeMode.dark);
        break;
      case ThemeMode.dark:
        setThemeMode(ThemeMode.light);
        break;
      case ThemeMode.system:
        setThemeMode(ThemeMode.light);
        break;
    }
  }
  
  Future<void> _saveThemeMode() async {
    final prefs = await SharedPreferences.getInstance();
    await prefs.setInt('theme_mode', _themeMode.index);
  }
  
  Future<void> _savePrimaryColor() async {
    final prefs = await SharedPreferences.getInstance();
    await prefs.setInt('primary_color', _primaryColor.value);
  }
  
  Future<void> loadSettings() async {
    final prefs = await SharedPreferences.getInstance();
    
    final themeModeIndex = prefs.getInt('theme_mode');
    if (themeModeIndex != null) {
      _themeMode = ThemeMode.values[themeModeIndex];
    }
    
    final primaryColorValue = prefs.getInt('primary_color');
    if (primaryColorValue != null) {
      _primaryColor = Color(primaryColorValue);
    }
    
    notifyListeners();
  }
}

// 多 Provider 应用
class MultiProviderApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MultiProvider(
      providers: [
        ChangeNotifierProvider(create: (context) => UserModel()),
        ChangeNotifierProvider(create: (context) => CartModel()),
        ChangeNotifierProvider(create: (context) => ThemeModel()..loadSettings()),
        ChangeNotifierProvider(create: (context) => CounterModel()),
      ],
      child: Consumer<ThemeModel>(
        builder: (context, themeModel, child) {
          return MaterialApp(
            title: 'Multi Provider Demo',
            themeMode: themeModel.themeMode,
            theme: ThemeData(
              primarySwatch: MaterialColor(
                themeModel.primaryColor.value,
                _generateMaterialColor(themeModel.primaryColor),
              ),
              brightness: Brightness.light,
            ),
            darkTheme: ThemeData(
              primarySwatch: MaterialColor(
                themeModel.primaryColor.value,
                _generateMaterialColor(themeModel.primaryColor),
              ),
              brightness: Brightness.dark,
            ),
            home: HomePage(),
          );
        },
      ),
    );
  }
  
  Map<int, Color> _generateMaterialColor(Color color) {
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
```

## Flutter 应用实例

### 购物应用状态管理

```dart
// 产品模型
class ProductModel extends ChangeNotifier {
  List<Product> _products = [];
  List<Product> _filteredProducts = [];
  bool _isLoading = false;
  String? _error;
  String _searchQuery = '';
  ProductCategory? _selectedCategory;
  
  List<Product> get products => List.unmodifiable(_filteredProducts);
  bool get isLoading => _isLoading;
  String? get error => _error;
  String get searchQuery => _searchQuery;
  ProductCategory? get selectedCategory => _selectedCategory;
  
  Future<void> loadProducts() async {
    _setLoading(true);
    _setError(null);
    
    try {
      final products = await ProductService.getProducts();
      _products = products;
      _applyFilters();
    } catch (e) {
      _setError(e.toString());
    } finally {
      _setLoading(false);
    }
  }
  
  void searchProducts(String query) {
    _searchQuery = query;
    _applyFilters();
  }
  
  void filterByCategory(ProductCategory? category) {
    _selectedCategory = category;
    _applyFilters();
  }
  
  void _applyFilters() {
    _filteredProducts = _products.where((product) {
      final matchesSearch = _searchQuery.isEmpty ||
          product.name.toLowerCase().contains(_searchQuery.toLowerCase()) ||
          product.description.toLowerCase().contains(_searchQuery.toLowerCase());
      
      final matchesCategory = _selectedCategory == null ||
          product.category == _selectedCategory;
      
      return matchesSearch && matchesCategory;
    }).toList();
    
    notifyListeners();
  }
  
  void _setLoading(bool loading) {
    _isLoading = loading;
    notifyListeners();
  }
  
  void _setError(String? error) {
    _error = error;
    notifyListeners();
  }
}

// 订单模型
class OrderModel extends ChangeNotifier {
  List<Order> _orders = [];
  bool _isLoading = false;
  String? _error;
  
  List<Order> get orders => List.unmodifiable(_orders);
  bool get isLoading => _isLoading;
  String? get error => _error;
  
  Future<void> loadOrders() async {
    _setLoading(true);
    _setError(null);
    
    try {
      final orders = await OrderService.getOrders();
      _orders = orders;
    } catch (e) {
      _setError(e.toString());
    } finally {
      _setLoading(false);
    }
  }
  
  Future<Order> createOrder(List<CartItem> items, Address address) async {
    _setLoading(true);
    _setError(null);
    
    try {
      final order = await OrderService.createOrder(items, address);
      _orders.insert(0, order);
      notifyListeners();
      return order;
    } catch (e) {
      _setError(e.toString());
      rethrow;
    } finally {
      _setLoading(false);
    }
  }
  
  Future<void> cancelOrder(String orderId) async {
    try {
      await OrderService.cancelOrder(orderId);
      final index = _orders.indexWhere((order) => order.id == orderId);
      if (index >= 0) {
        _orders[index] = _orders[index].copyWith(status: OrderStatus.cancelled);
        notifyListeners();
      }
    } catch (e) {
      _setError(e.toString());
    }
  }
  
  void _setLoading(bool loading) {
    _isLoading = loading;
    notifyListeners();
  }
  
  void _setError(String? error) {
    _error = error;
    notifyListeners();
  }
}

// 购物应用主页
class ShoppingHomePage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Shopping App'),
        actions: [
          Consumer<CartModel>(
            builder: (context, cart, child) {
              return Stack(
                children: [
                  IconButton(
                    icon: Icon(Icons.shopping_cart),
                    onPressed: () {
                      Navigator.push(
                        context,
                        MaterialPageRoute(builder: (context) => CartPage()),
                      );
                    },
                  ),
                  if (cart.itemCount > 0)
                    Positioned(
                      right: 8,
                      top: 8,
                      child: Container(
                        padding: EdgeInsets.all(2),
                        decoration: BoxDecoration(
                          color: Colors.red,
                          borderRadius: BorderRadius.circular(10),
                        ),
                        constraints: BoxConstraints(
                          minWidth: 16,
                          minHeight: 16,
                        ),
                        child: Text(
                          '${cart.itemCount}',
                          style: TextStyle(
                            color: Colors.white,
                            fontSize: 12,
                          ),
                          textAlign: TextAlign.center,
                        ),
                      ),
                    ),
                ],
              );
            },
          ),
          Consumer<ThemeModel>(
            builder: (context, theme, child) {
              return IconButton(
                icon: Icon(
                  theme.isDarkMode ? Icons.light_mode : Icons.dark_mode,
                ),
                onPressed: theme.toggleTheme,
              );
            },
          ),
        ],
      ),
      body: Column(
        children: [
          // 搜索栏
          Padding(
            padding: EdgeInsets.all(16),
            child: Consumer<ProductModel>(
              builder: (context, productModel, child) {
                return TextField(
                  decoration: InputDecoration(
                    hintText: 'Search products...',
                    prefixIcon: Icon(Icons.search),
                    border: OutlineInputBorder(
                      borderRadius: BorderRadius.circular(8),
                    ),
                  ),
                  onChanged: productModel.searchProducts,
                );
              },
            ),
          ),
          // 分类筛选
          Container(
            height: 50,
            child: Consumer<ProductModel>(
              builder: (context, productModel, child) {
                return ListView.builder(
                  scrollDirection: Axis.horizontal,
                  padding: EdgeInsets.symmetric(horizontal: 16),
                  itemCount: ProductCategory.values.length + 1,
                  itemBuilder: (context, index) {
                    if (index == 0) {
                      return Padding(
                        padding: EdgeInsets.only(right: 8),
                        child: FilterChip(
                          label: Text('All'),
                          selected: productModel.selectedCategory == null,
                          onSelected: (selected) {
                            productModel.filterByCategory(null);
                          },
                        ),
                      );
                    }
                    
                    final category = ProductCategory.values[index - 1];
                    return Padding(
                      padding: EdgeInsets.only(right: 8),
                      child: FilterChip(
                        label: Text(category.name),
                        selected: productModel.selectedCategory == category,
                        onSelected: (selected) {
                          productModel.filterByCategory(selected ? category : null);
                        },
                      ),
                    );
                  },
                );
              },
            ),
          ),
          // 产品列表
          Expanded(
            child: Consumer<ProductModel>(
              builder: (context, productModel, child) {
                if (productModel.isLoading) {
                  return Center(child: CircularProgressIndicator());
                }
                
                if (productModel.error != null) {
                  return Center(
                    child: Column(
                      mainAxisAlignment: MainAxisAlignment.center,
                      children: [
                        Text('Error: ${productModel.error}'),
                        ElevatedButton(
                          onPressed: productModel.loadProducts,
                          child: Text('Retry'),
                        ),
                      ],
                    ),
                  );
                }
                
                if (productModel.products.isEmpty) {
                  return Center(
                    child: Text('No products found'),
                  );
                }
                
                return GridView.builder(
                  padding: EdgeInsets.all(16),
                  gridDelegate: SliverGridDelegateWithFixedCrossAxisCount(
                    crossAxisCount: 2,
                    childAspectRatio: 0.8,
                    crossAxisSpacing: 16,
                    mainAxisSpacing: 16,
                  ),
                  itemCount: productModel.products.length,
                  itemBuilder: (context, index) {
                    final product = productModel.products[index];
                    return ProductCard(product: product);
                  },
                );
              },
            ),
          ),
        ],
      ),
    );
  }
}

// 产品卡片组件
class ProductCard extends StatelessWidget {
  final Product product;
  
  const ProductCard({Key? key, required this.product}) : super(key: key);
  
  @override
  Widget build(BuildContext context) {
    return Card(
      elevation: 4,
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          Expanded(
            child: Container(
              width: double.infinity,
              decoration: BoxDecoration(
                borderRadius: BorderRadius.vertical(top: Radius.circular(4)),
                image: DecorationImage(
                  image: NetworkImage(product.imageUrl),
                  fit: BoxFit.cover,
                ),
              ),
            ),
          ),
          Padding(
            padding: EdgeInsets.all(8),
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: [
                Text(
                  product.name,
                  style: TextStyle(fontWeight: FontWeight.bold),
                  maxLines: 1,
                  overflow: TextOverflow.ellipsis,
                ),
                SizedBox(height: 4),
                Text(
                  '\$${product.price.toStringAsFixed(2)}',
                  style: TextStyle(
                    color: Theme.of(context).primaryColor,
                    fontWeight: FontWeight.bold,
                  ),
                ),
                SizedBox(height: 8),
                SizedBox(
                  width: double.infinity,
                  child: ElevatedButton(
                    onPressed: () {
                      context.read<CartModel>().addItem(product);
                      ScaffoldMessenger.of(context).showSnackBar(
                        SnackBar(
                          content: Text('${product.name} added to cart'),
                          duration: Duration(seconds: 1),
                        ),
                      );
                    },
                    child: Text('Add to Cart'),
                  ),
                ),
              ],
            ),
          ),
        ],
      ),
    );
  }
}
```

### 社交应用状态管理

```dart
// 帖子模型
class PostModel extends ChangeNotifier {
  List<Post> _posts = [];
  bool _isLoading = false;
  bool _hasMore = true;
  String? _error;
  int _currentPage = 0;
  
  List<Post> get posts => List.unmodifiable(_posts);
  bool get isLoading => _isLoading;
  bool get hasMore => _hasMore;
  String? get error => _error;
  
  Future<void> loadPosts({bool refresh = false}) async {
    if (_isLoading) return;
    
    if (refresh) {
      _currentPage = 0;
      _posts.clear();
      _hasMore = true;
    }
    
    _setLoading(true);
    _setError(null);
    
    try {
      final newPosts = await PostService.getPosts(
        page: _currentPage,
        limit: 20,
      );
      
      if (refresh) {
        _posts = newPosts;
      } else {
        _posts.addAll(newPosts);
      }
      
      _currentPage++;
      _hasMore = newPosts.length == 20;
      
    } catch (e) {
      _setError(e.toString());
    } finally {
      _setLoading(false);
    }
  }
  
  Future<void> likePost(String postId) async {
    final index = _posts.indexWhere((post) => post.id == postId);
    if (index < 0) return;
    
    final post = _posts[index];
    final wasLiked = post.isLiked;
    
    // 乐观更新
    _posts[index] = post.copyWith(
      isLiked: !wasLiked,
      likeCount: wasLiked ? post.likeCount - 1 : post.likeCount + 1,
    );
    notifyListeners();
    
    try {
      if (wasLiked) {
        await PostService.unlikePost(postId);
      } else {
        await PostService.likePost(postId);
      }
    } catch (e) {
      // 回滚更新
      _posts[index] = post;
      notifyListeners();
      _setError(e.toString());
    }
  }
  
  Future<void> addComment(String postId, String content) async {
    try {
      final comment = await PostService.addComment(postId, content);
      final index = _posts.indexWhere((post) => post.id == postId);
      if (index >= 0) {
        final post = _posts[index];
        _posts[index] = post.copyWith(
          commentCount: post.commentCount + 1,
        );
        notifyListeners();
      }
    } catch (e) {
      _setError(e.toString());
    }
  }
  
  void _setLoading(bool loading) {
    _isLoading = loading;
    notifyListeners();
  }
  
  void _setError(String? error) {
    _error = error;
    notifyListeners();
  }
}

// 通知模型
class NotificationModel extends ChangeNotifier {
  List<AppNotification> _notifications = [];
  int _unreadCount = 0;
  bool _isLoading = false;
  
  List<AppNotification> get notifications => List.unmodifiable(_notifications);
  int get unreadCount => _unreadCount;
  bool get isLoading => _isLoading;
  
  Future<void> loadNotifications() async {
    _isLoading = true;
    notifyListeners();
    
    try {
      final notifications = await NotificationService.getNotifications();
      _notifications = notifications;
      _unreadCount = notifications.where((n) => !n.isRead).length;
    } catch (e) {
      print('Failed to load notifications: $e');
    } finally {
      _isLoading = false;
      notifyListeners();
    }
  }
  
  Future<void> markAsRead(String notificationId) async {
    final index = _notifications.indexWhere((n) => n.id == notificationId);
    if (index < 0 || _notifications[index].isRead) return;
    
    try {
      await NotificationService.markAsRead(notificationId);
      _notifications[index] = _notifications[index].copyWith(isRead: true);
      _unreadCount--;
      notifyListeners();
    } catch (e) {
      print('Failed to mark notification as read: $e');
    }
  }
  
  Future<void> markAllAsRead() async {
    try {
      await NotificationService.markAllAsRead();
      _notifications = _notifications.map((n) => n.copyWith(isRead: true)).toList();
      _unreadCount = 0;
      notifyListeners();
    } catch (e) {
      print('Failed to mark all notifications as read: $e');
    }
  }
  
  void addNotification(AppNotification notification) {
    _notifications.insert(0, notification);
    if (!notification.isRead) {
      _unreadCount++;
    }
    notifyListeners();
  }
}

// 社交应用主页
class SocialHomePage extends StatefulWidget {
  @override
  _SocialHomePageState createState() => _SocialHomePageState();
}

class _SocialHomePageState extends State<SocialHomePage> {
  final ScrollController _scrollController = ScrollController();
  
  @override
  void initState() {
    super.initState();
    _scrollController.addListener(_onScroll);
    WidgetsBinding.instance.addPostFrameCallback((_) {
      context.read<PostModel>().loadPosts(refresh: true);
      context.read<NotificationModel>().loadNotifications();
    });
  }
  
  void _onScroll() {
    if (_scrollController.position.pixels >=
        _scrollController.position.maxScrollExtent - 200) {
      final postModel = context.read<PostModel>();
      if (postModel.hasMore && !postModel.isLoading) {
        postModel.loadPosts();
      }
    }
  }
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Social Feed'),
        actions: [
          Consumer<NotificationModel>(
            builder: (context, notificationModel, child) {
              return Stack(
                children: [
                  IconButton(
                    icon: Icon(Icons.notifications),
                    onPressed: () {
                      Navigator.push(
                        context,
                        MaterialPageRoute(
                          builder: (context) => NotificationPage(),
                        ),
                      );
                    },
                  ),
                  if (notificationModel.unreadCount > 0)
                    Positioned(
                      right: 8,
                      top: 8,
                      child: Container(
                        padding: EdgeInsets.all(2),
                        decoration: BoxDecoration(
                          color: Colors.red,
                          borderRadius: BorderRadius.circular(10),
                        ),
                        constraints: BoxConstraints(
                          minWidth: 16,
                          minHeight: 16,
                        ),
                        child: Text(
                          '${notificationModel.unreadCount}',
                          style: TextStyle(
                            color: Colors.white,
                            fontSize: 12,
                          ),
                          textAlign: TextAlign.center,
                        ),
                      ),
                    ),
                ],
              );
            },
          ),
        ],
      ),
      body: Consumer<PostModel>(
        builder: (context, postModel, child) {
          if (postModel.posts.isEmpty && postModel.isLoading) {
            return Center(child: CircularProgressIndicator());
          }
          
          if (postModel.error != null && postModel.posts.isEmpty) {
            return Center(
              child: Column(
                mainAxisAlignment: MainAxisAlignment.center,
                children: [
                  Text('Error: ${postModel.error}'),
                  ElevatedButton(
                    onPressed: () => postModel.loadPosts(refresh: true),
                    child: Text('Retry'),
                  ),
                ],
              ),
            );
          }
          
          return RefreshIndicator(
            onRefresh: () => postModel.loadPosts(refresh: true),
            child: ListView.builder(
              controller: _scrollController,
              itemCount: postModel.posts.length + (postModel.hasMore ? 1 : 0),
              itemBuilder: (context, index) {
                if (index == postModel.posts.length) {
                  return Center(
                    child: Padding(
                      padding: EdgeInsets.all(16),
                      child: CircularProgressIndicator(),
                    ),
                  );
                }
                
                final post = postModel.posts[index];
                return PostCard(post: post);
              },
            ),
          );
        },
      ),
    );
  }
  
  @override
  void dispose() {
    _scrollController.dispose();
    super.dispose();
  }
}

// 帖子卡片组件
class PostCard extends StatelessWidget {
  final Post post;
  
  const PostCard({Key? key, required this.post}) : super(key: key);
  
  @override
  Widget build(BuildContext context) {
    return Card(
      margin: EdgeInsets.symmetric(horizontal: 16, vertical: 8),
      child: Padding(
        padding: EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            // 用户信息
            Row(
              children: [
                CircleAvatar(
                  radius: 20,
                  backgroundImage: NetworkImage(post.author.avatarUrl),
                ),
                SizedBox(width: 12),
                Expanded(
                  child: Column(
                    crossAxisAlignment: CrossAxisAlignment.start,
                    children: [
                      Text(
                        post.author.name,
                        style: TextStyle(fontWeight: FontWeight.bold),
                      ),
                      Text(
                        _formatTime(post.createdAt),
                        style: TextStyle(
                          color: Colors.grey[600],
                          fontSize: 12,
                        ),
                      ),
                    ],
                  ),
                ),
                IconButton(
                  icon: Icon(Icons.more_vert),
                  onPressed: () {
                    // 显示更多选项
                  },
                ),
              ],
            ),
            SizedBox(height: 12),
            // 帖子内容
            Text(post.content),
            if (post.imageUrls.isNotEmpty) ..[
              SizedBox(height: 12),
              Container(
                height: 200,
                child: PageView.builder(
                  itemCount: post.imageUrls.length,
                  itemBuilder: (context, index) {
                    return Container(
                      margin: EdgeInsets.only(right: 8),
                      decoration: BoxDecoration(
                        borderRadius: BorderRadius.circular(8),
                        image: DecorationImage(
                          image: NetworkImage(post.imageUrls[index]),
                          fit: BoxFit.cover,
                        ),
                      ),
                    );
                  },
                ),
              ),
            ],
            SizedBox(height: 12),
            // 操作按钮
            Row(
              children: [
                Consumer<PostModel>(
                  builder: (context, postModel, child) {
                    return TextButton.icon(
                      icon: Icon(
                        post.isLiked ? Icons.favorite : Icons.favorite_border,
                        color: post.isLiked ? Colors.red : null,
                      ),
                      label: Text('${post.likeCount}'),
                      onPressed: () {
                        postModel.likePost(post.id);
                      },
                    );
                  },
                ),
                TextButton.icon(
                  icon: Icon(Icons.comment),
                  label: Text('${post.commentCount}'),
                  onPressed: () {
                    Navigator.push(
                      context,
                      MaterialPageRoute(
                        builder: (context) => PostDetailPage(post: post),
                      ),
                    );
                  },
                ),
                TextButton.icon(
                  icon: Icon(Icons.share),
                  label: Text('Share'),
                  onPressed: () {
                    // 分享功能
                  },
                ),
              ],
            ),
          ],
        ),
      ),
    );
  }
  
  String _formatTime(DateTime dateTime) {
    final now = DateTime.now();
    final difference = now.difference(dateTime);
    
    if (difference.inMinutes < 1) {
      return 'Just now';
    } else if (difference.inHours < 1) {
      return '${difference.inMinutes}m ago';
    } else if (difference.inDays < 1) {
      return '${difference.inHours}h ago';
    } else {
      return '${difference.inDays}d ago';
    }
  }
}
```

## 高级用法

### ProxyProvider 实现

```dart
// 依赖其他 Provider 的 Provider
class CartSummaryModel {
  final CartModel cart;
  final UserModel user;
  
  CartSummaryModel({
    required this.cart,
    required this.user,
  });
  
  double get subtotal => cart.totalPrice;
  
  double get tax => subtotal * 0.08; // 8% 税率
  
  double get shipping {
    if (user.user?.isPremium == true) {
      return 0.0; // 会员免运费
    }
    return subtotal > 50 ? 0.0 : 5.99; // 满50免运费
  }
  
  double get total => subtotal + tax + shipping;
  
  double get discount {
    if (user.user?.isPremium == true) {
      return subtotal * 0.1; // 会员10%折扣
    }
    return 0.0;
  }
  
  double get finalTotal => total - discount;
}

// 使用 ProxyProvider
class ShoppingApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MultiProvider(
      providers: [
        ChangeNotifierProvider(create: (context) => UserModel()),
        ChangeNotifierProvider(create: (context) => CartModel()),
        ProxyProvider2<CartModel, UserModel, CartSummaryModel>(
          update: (context, cart, user, previous) => CartSummaryModel(
            cart: cart,
            user: user,
          ),
        ),
      ],
      child: MaterialApp(
        home: ShoppingHomePage(),
      ),
    );
  }
}

// 使用 CartSummaryModel
class CheckoutPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Checkout')),
      body: Consumer<CartSummaryModel>(
        builder: (context, summary, child) {
          return Padding(
            padding: EdgeInsets.all(16),
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: [
                Text('Order Summary', style: TextStyle(fontSize: 20, fontWeight: FontWeight.bold)),
                SizedBox(height: 16),
                _buildSummaryRow('Subtotal', summary.subtotal),
                _buildSummaryRow('Tax', summary.tax),
                _buildSummaryRow('Shipping', summary.shipping),
                if (summary.discount > 0)
                  _buildSummaryRow('Discount', -summary.discount, color: Colors.green),
                Divider(),
                _buildSummaryRow('Total', summary.finalTotal, isTotal: true),
              ],
            ),
          );
        },
      ),
    );
  }
  
  Widget _buildSummaryRow(String label, double amount, {Color? color, bool isTotal = false}) {
    return Padding(
      padding: EdgeInsets.symmetric(vertical: 4),
      child: Row(
        mainAxisAlignment: MainAxisAlignment.spaceBetween,
        children: [
          Text(
            label,
            style: TextStyle(
              fontSize: isTotal ? 18 : 16,
              fontWeight: isTotal ? FontWeight.bold : FontWeight.normal,
            ),
          ),
          Text(
            '\$${amount.abs().toStringAsFixed(2)}',
            style: TextStyle(
              fontSize: isTotal ? 18 : 16,
              fontWeight: isTotal ? FontWeight.bold : FontWeight.normal,
              color: color,
            ),
          ),
        ],
      ),
    );
  }
}
```

### FutureProvider 和 StreamProvider

```dart
// 使用 FutureProvider
class WeatherApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: FutureProvider<Weather?>(
        create: (context) => WeatherService.getCurrentWeather(),
        initialData: null,
        child: WeatherPage(),
      ),
    );
  }
}

class WeatherPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Weather')),
      body: Consumer<Weather?>(
        builder: (context, weather, child) {
          if (weather == null) {
            return Center(child: CircularProgressIndicator());
          }
          
          return Center(
            child: Column(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                Text(
                  weather.city,
                  style: TextStyle(fontSize: 24, fontWeight: FontWeight.bold),
                ),
                SizedBox(height: 16),
                Text(
                  '${weather.temperature}°C',
                  style: TextStyle(fontSize: 48),
                ),
                Text(
                  weather.description,
                  style: TextStyle(fontSize: 18),
                ),
              ],
            ),
          );
        },
      ),
    );
  }
}

// 使用 StreamProvider
class ChatApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: StreamProvider<List<Message>>(
        create: (context) => ChatService.getMessageStream(),
        initialData: [],
        child: ChatPage(),
      ),
    );
  }
}

class ChatPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Chat')),
      body: Consumer<List<Message>>(
        builder: (context, messages, child) {
          return ListView.builder(
            reverse: true,
            itemCount: messages.length,
            itemBuilder: (context, index) {
              final message = messages[messages.length - 1 - index];
              return MessageBubble(message: message);
            },
          );
        },
      ),
    );
  }
}
```

## 测试和调试

### Provider 测试

```dart
import 'package:flutter/material.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:provider/provider.dart';

void main() {
  group('Provider 测试', () {
    testWidgets('CounterModel 应该正确更新计数', (WidgetTester tester) async {
      final counterModel = CounterModel();
      
      await tester.pumpWidget(
        ChangeNotifierProvider.value(
          value: counterModel,
          child: MaterialApp(
            home: Consumer<CounterModel>(
              builder: (context, counter, child) {
                return Text('${counter.count}');
              },
            ),
          ),
        ),
      );
      
      expect(find.text('0'), findsOneWidget);
      
      counterModel.increment();
      await tester.pump();
      
      expect(find.text('1'), findsOneWidget);
    });
    
    testWidgets('CartModel 应该正确添加商品', (WidgetTester tester) async {
      final cartModel = CartModel();
      final product = Product(
        id: '1',
        name: 'Test Product',
        price: 10.0,
        imageUrl: 'test.jpg',
        category: ProductCategory.electronics,
        description: 'Test description',
      );
      
      await tester.pumpWidget(
        ChangeNotifierProvider.value(
          value: cartModel,
          child: MaterialApp(
            home: Consumer<CartModel>(
              builder: (context, cart, child) {
                return Column(
                  children: [
                    Text('Items: ${cart.itemCount}'),
                    Text('Total: \$${cart.totalPrice.toStringAsFixed(2)}'),
                    ElevatedButton(
                      onPressed: () => cart.addItem(product),
                      child: Text('Add Item'),
                    ),
                  ],
                );
              },
            ),
          ),
        ),
      );
      
      expect(find.text('Items: 0'), findsOneWidget);
      expect(find.text('Total: \$0.00'), findsOneWidget);
      
      await tester.tap(find.text('Add Item'));
      await tester.pump();
      
      expect(find.text('Items: 1'), findsOneWidget);
      expect(find.text('Total: \$10.00'), findsOneWidget);
    });
  });
  
  group('异步 Provider 测试', () {
    testWidgets('UserModel 应该处理登录流程', (WidgetTester tester) async {
      final userModel = UserModel();
      
      await tester.pumpWidget(
        ChangeNotifierProvider.value(
          value: userModel,
          child: MaterialApp(
            home: Consumer<UserModel>(
              builder: (context, user, child) {
                if (user.isLoading) {
                  return CircularProgressIndicator();
                }
                
                if (user.error != null) {
                  return Text('Error: ${user.error}');
                }
                
                if (user.isLoggedIn) {
                  return Text('Welcome ${user.user!.name}');
                }
                
                return ElevatedButton(
                  onPressed: () => user.login('test@example.com', 'password'),
                  child: Text('Login'),
                );
              },
            ),
          ),
        ),
      );
      
      expect(find.text('Login'), findsOneWidget);
      
      await tester.tap(find.text('Login'));
      await tester.pump();
      
      expect(find.byType(CircularProgressIndicator), findsOneWidget);
      
      // 等待异步操作完成
      await tester.pumpAndSettle();
      
      // 根据 mock 服务的返回结果验证
    });
  });
}
```

### Provider 调试器

```dart
class ProviderDebugger {
  static final Map<Type, ProviderStats> _stats = {};
  static final List<ProviderEvent> _events = [];
  
  static void logProviderAccess(Type providerType, String method) {
    final stats = _stats.putIfAbsent(providerType, () => ProviderStats(providerType));
    stats.incrementAccess(method);
    
    final event = ProviderEvent(
      providerType: providerType,
      method: method,
      timestamp: DateTime.now(),
    );
    _events.add(event);
    
    if (kDebugMode) {
      print('Provider访问: ${providerType.toString()}.$method');
    }
  }
  
  static void logProviderUpdate(Type providerType) {
    final stats = _stats.putIfAbsent(providerType, () => ProviderStats(providerType));
    stats.incrementUpdate();
    
    final event = ProviderEvent(
      providerType: providerType,
      method: 'notifyListeners',
      timestamp: DateTime.now(),
    );
    _events.add(event);
    
    if (kDebugMode) {
      print('Provider更新: ${providerType.toString()}');
    }
  }
  
  static Map<Type, ProviderStats> getStats() => Map.unmodifiable(_stats);
  
  static List<ProviderEvent> getEvents() => List.unmodifiable(_events);
  
  static void clearHistory() {
    _stats.clear();
    _events.clear();
  }
  
  static void printStatistics() {
    print('=== Provider 统计信息 ===');
    print('Provider类型数量: ${_stats.length}');
    print('总事件数量: ${_events.length}');
    print('\n各Provider统计:');
    _stats.values.forEach((stats) {
      print('  ${stats.providerType}:');
      print('    访问次数: ${stats.totalAccess}');
      print('    更新次数: ${stats.updateCount}');
      print('    访问方法: ${stats.accessMethods}');
    });
  }
}

class ProviderStats {
  final Type providerType;
  final Map<String, int> accessMethods = {};
  int updateCount = 0;
  
  ProviderStats(this.providerType);
  
  void incrementAccess(String method) {
    accessMethods[method] = (accessMethods[method] ?? 0) + 1;
  }
  
  void incrementUpdate() {
    updateCount++;
  }
  
  int get totalAccess => accessMethods.values.fold(0, (sum, count) => sum + count);
}

class ProviderEvent {
  final Type providerType;
  final String method;
  final DateTime timestamp;
  
  ProviderEvent({
    required this.providerType,
    required this.method,
    required this.timestamp,
  });
}

// 带调试功能的 ChangeNotifier
abstract class DebuggableChangeNotifier extends ChangeNotifier {
  @override
  void notifyListeners() {
    if (kDebugMode) {
      ProviderDebugger.logProviderUpdate(runtimeType);
    }
    super.notifyListeners();
  }
}

// 带调试功能的 Consumer
class DebuggableConsumer<T> extends StatelessWidget {
  final Widget Function(BuildContext context, T value, Widget? child) builder;
  final Widget? child;
  
  const DebuggableConsumer({
    Key? key,
    required this.builder,
    this.child,
  }) : super(key: key);
  
  @override
  Widget build(BuildContext context) {
    if (kDebugMode) {
      ProviderDebugger.logProviderAccess(T, 'Consumer');
    }
    
    return Consumer<T>(
      builder: builder,
      child: child,
    );
  }
}
```

## 最佳实践

### 设计原则

1. **单一职责原则**
   ```dart
   // 好的做法：每个 Model 负责单一功能
   class UserModel extends ChangeNotifier {
     // 只处理用户相关逻辑
   }
   
   class CartModel extends ChangeNotifier {
     // 只处理购物车相关逻辑
   }
   
   // 避免：一个 Model 处理多种功能
   class AppModel extends ChangeNotifier {
     // 用户、购物车、订单等所有逻辑混在一起
   }
   ```

2. **不可变数据**
   ```dart
   // 好的做法：使用不可变数据
   class Product {
     final String id;
     final String name;
     final double price;
     
     const Product({required this.id, required this.name, required this.price});
     
     Product copyWith({String? name, double? price}) {
       return Product(
         id: id,
         name: name ?? this.name,
         price: price ?? this.price,
       );
     }
   }
   ```

3. **合理的 Provider 层次结构**
   ```dart
   // 好的做法：根据作用域组织 Provider
   MultiProvider(
     providers: [
       // 全局 Provider
       ChangeNotifierProvider(create: (context) => ThemeModel()),
       ChangeNotifierProvider(create: (context) => UserModel()),
     ],
     child: MaterialApp(
       home: ChangeNotifierProvider(
         // 页面级 Provider
         create: (context) => HomePageModel(),
         child: HomePage(),
       ),
     ),
   )
   ```

### 实现建议

1. **使用 Selector 优化性能**
   ```dart
   // 只监听特定字段的变化
   Selector<UserModel, String?>(
     selector: (context, user) => user.user?.name,
     builder: (context, userName, child) {
       return Text(userName ?? 'Guest');
     },
   )
   ```

2. **合理使用 context.read() 和 context.watch()**
   ```dart
   // 在事件处理中使用 read
   onPressed: () {
     context.read<CartModel>().addItem(product);
   }
   
   // 在 build 方法中使用 watch
   Widget build(BuildContext context) {
     final cart = context.watch<CartModel>();
     return Text('Items: ${cart.itemCount}');
   }
   ```

3. **错误处理和加载状态**
   ```dart
   class ApiModel extends ChangeNotifier {
     bool _isLoading = false;
     String? _error;
     
     bool get isLoading => _isLoading;
     String? get error => _error;
     
     Future<void> fetchData() async {
       _setLoading(true);
       _setError(null);
       
       try {
         // API 调用
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
     
     void _setError(String? error) {
       _error = error;
       notifyListeners();
     }
   }
   ```

### 注意事项

1. **避免在 build 方法中调用 notifyListeners**
2. **合理管理 Provider 的生命周期**
3. **使用 ProxyProvider 处理依赖关系**
4. **在测试中使用 mock Provider**
5. **避免过度嵌套 Consumer**

Provider 是 Flutter 中最受欢迎的状态管理解决方案之一，它提供了简洁的 API 和强大的功能。通过遵循最佳实践，可以构建出高效、可维护的 Flutter 应用。