# Flutter BLoC 模式 (Business Logic Component Pattern)

## 模式概述

### 定义
BLoC (Business Logic Component) 是一种基于流 (Stream) 和响应式编程的状态管理模式。它将业务逻辑与 UI 完全分离，通过事件 (Event) 和状态 (State) 的方式管理应用状态，提供了可预测、可测试的状态管理解决方案。

### 适用场景
- 大型复杂应用的状态管理
- 需要严格分离业务逻辑和 UI
- 跨平台应用开发 (Flutter Web、Desktop)
- 需要时间旅行调试的应用
- 团队协作开发的项目

### 优点
- 完全分离业务逻辑和 UI
- 基于流的响应式编程
- 优秀的可测试性
- 支持时间旅行调试
- 强类型安全
- 丰富的开发工具支持

### 缺点
- 学习曲线较陡峭
- 样板代码较多
- 对于简单应用可能过于复杂
- 需要理解响应式编程概念

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

### 简单 BLoC 实现

```dart
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import 'package:equatable/equatable.dart';

// 计数器事件
abstract class CounterEvent extends Equatable {
  const CounterEvent();
  
  @override
  List<Object> get props => [];
}

class CounterIncrement extends CounterEvent {}

class CounterDecrement extends CounterEvent {}

class CounterReset extends CounterEvent {}

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

// 计数器 BLoC
class CounterBloc extends Bloc<CounterEvent, CounterState> {
  CounterBloc() : super(const CounterState(count: 0)) {
    on<CounterIncrement>(_onIncrement);
    on<CounterDecrement>(_onDecrement);
    on<CounterReset>(_onReset);
  }
  
  void _onIncrement(CounterIncrement event, Emitter<CounterState> emit) {
    emit(state.copyWith(count: state.count + 1));
  }
  
  void _onDecrement(CounterDecrement event, Emitter<CounterState> emit) {
    emit(state.copyWith(count: state.count - 1));
  }
  
  void _onReset(CounterReset event, Emitter<CounterState> emit) {
    emit(const CounterState(count: 0));
  }
}

// 应用入口
class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'BLoC Demo',
      home: BlocProvider(
        create: (context) => CounterBloc(),
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
        title: Text('Counter BLoC'),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Text(
              'Count:',
              style: TextStyle(fontSize: 20),
            ),
            BlocBuilder<CounterBloc, CounterState>(
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
                    context.read<CounterBloc>().add(CounterDecrement());
                  },
                  child: Text('-'),
                ),
                ElevatedButton(
                  onPressed: () {
                    context.read<CounterBloc>().add(CounterReset());
                  },
                  child: Text('Reset'),
                ),
                ElevatedButton(
                  onPressed: () {
                    context.read<CounterBloc>().add(CounterIncrement());
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

// 使用 BlocListener 处理副作用
class CounterPageWithListener extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return BlocListener<CounterBloc, CounterState>(
      listener: (context, state) {
        if (state.count == 10) {
          ScaffoldMessenger.of(context).showSnackBar(
            SnackBar(content: Text('Congratulations! You reached 10!')),
          );
        }
      },
      child: CounterPage(),
    );
  }
}
```

### 复杂 BLoC 实现

```dart
// 用户认证事件
abstract class AuthEvent extends Equatable {
  const AuthEvent();
  
  @override
  List<Object?> get props => [];
}

class AuthLoginRequested extends AuthEvent {
  final String email;
  final String password;
  
  const AuthLoginRequested({required this.email, required this.password});
  
  @override
  List<Object> get props => [email, password];
}

class AuthLogoutRequested extends AuthEvent {}

class AuthStatusChanged extends AuthEvent {
  final User? user;
  
  const AuthStatusChanged({this.user});
  
  @override
  List<Object?> get props => [user];
}

class AuthRegisterRequested extends AuthEvent {
  final String email;
  final String password;
  final String name;
  
  const AuthRegisterRequested({
    required this.email,
    required this.password,
    required this.name,
  });
  
  @override
  List<Object> get props => [email, password, name];
}

// 用户认证状态
abstract class AuthState extends Equatable {
  const AuthState();
  
  @override
  List<Object?> get props => [];
}

class AuthInitial extends AuthState {}

class AuthLoading extends AuthState {}

class AuthAuthenticated extends AuthState {
  final User user;
  
  const AuthAuthenticated({required this.user});
  
  @override
  List<Object> get props => [user];
}

class AuthUnauthenticated extends AuthState {}

class AuthError extends AuthState {
  final String message;
  
  const AuthError({required this.message});
  
  @override
  List<Object> get props => [message];
}

// 用户认证 BLoC
class AuthBloc extends Bloc<AuthEvent, AuthState> {
  final AuthRepository _authRepository;
  late final StreamSubscription<User?> _userSubscription;
  
  AuthBloc({required AuthRepository authRepository})
      : _authRepository = authRepository,
        super(AuthInitial()) {
    on<AuthLoginRequested>(_onLoginRequested);
    on<AuthLogoutRequested>(_onLogoutRequested);
    on<AuthStatusChanged>(_onStatusChanged);
    on<AuthRegisterRequested>(_onRegisterRequested);
    
    // 监听用户状态变化
    _userSubscription = _authRepository.userStream.listen(
      (user) => add(AuthStatusChanged(user: user)),
    );
  }
  
  Future<void> _onLoginRequested(
    AuthLoginRequested event,
    Emitter<AuthState> emit,
  ) async {
    emit(AuthLoading());
    
    try {
      await _authRepository.login(
        email: event.email,
        password: event.password,
      );
    } catch (e) {
      emit(AuthError(message: e.toString()));
    }
  }
  
  Future<void> _onLogoutRequested(
    AuthLogoutRequested event,
    Emitter<AuthState> emit,
  ) async {
    emit(AuthLoading());
    
    try {
      await _authRepository.logout();
    } catch (e) {
      emit(AuthError(message: e.toString()));
    }
  }
  
  void _onStatusChanged(
    AuthStatusChanged event,
    Emitter<AuthState> emit,
  ) {
    if (event.user != null) {
      emit(AuthAuthenticated(user: event.user!));
    } else {
      emit(AuthUnauthenticated());
    }
  }
  
  Future<void> _onRegisterRequested(
    AuthRegisterRequested event,
    Emitter<AuthState> emit,
  ) async {
    emit(AuthLoading());
    
    try {
      await _authRepository.register(
        email: event.email,
        password: event.password,
        name: event.name,
      );
    } catch (e) {
      emit(AuthError(message: e.toString()));
    }
  }
  
  @override
  Future<void> close() {
    _userSubscription.cancel();
    return super.close();
  }
}

// 认证页面
class AuthPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Authentication')),
      body: BlocConsumer<AuthBloc, AuthState>(
        listener: (context, state) {
          if (state is AuthError) {
            ScaffoldMessenger.of(context).showSnackBar(
              SnackBar(
                content: Text(state.message),
                backgroundColor: Colors.red,
              ),
            );
          }
        },
        builder: (context, state) {
          if (state is AuthLoading) {
            return Center(child: CircularProgressIndicator());
          }
          
          if (state is AuthAuthenticated) {
            return Center(
              child: Column(
                mainAxisAlignment: MainAxisAlignment.center,
                children: [
                  Text('Welcome, ${state.user.name}!'),
                  SizedBox(height: 20),
                  ElevatedButton(
                    onPressed: () {
                      context.read<AuthBloc>().add(AuthLogoutRequested());
                    },
                    child: Text('Logout'),
                  ),
                ],
              ),
            );
          }
          
          return LoginForm();
        },
      ),
    );
  }
}

// 登录表单
class LoginForm extends StatefulWidget {
  @override
  _LoginFormState createState() => _LoginFormState();
}

class _LoginFormState extends State<LoginForm> {
  final _emailController = TextEditingController();
  final _passwordController = TextEditingController();
  final _formKey = GlobalKey<FormState>();
  
  @override
  Widget build(BuildContext context) {
    return Padding(
      padding: EdgeInsets.all(16),
      child: Form(
        key: _formKey,
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            TextFormField(
              controller: _emailController,
              decoration: InputDecoration(
                labelText: 'Email',
                border: OutlineInputBorder(),
              ),
              validator: (value) {
                if (value == null || value.isEmpty) {
                  return 'Please enter your email';
                }
                if (!value.contains('@')) {
                  return 'Please enter a valid email';
                }
                return null;
              },
            ),
            SizedBox(height: 16),
            TextFormField(
              controller: _passwordController,
              decoration: InputDecoration(
                labelText: 'Password',
                border: OutlineInputBorder(),
              ),
              obscureText: true,
              validator: (value) {
                if (value == null || value.isEmpty) {
                  return 'Please enter your password';
                }
                if (value.length < 6) {
                  return 'Password must be at least 6 characters';
                }
                return null;
              },
            ),
            SizedBox(height: 24),
            SizedBox(
              width: double.infinity,
              child: ElevatedButton(
                onPressed: () {
                  if (_formKey.currentState!.validate()) {
                    context.read<AuthBloc>().add(
                      AuthLoginRequested(
                        email: _emailController.text,
                        password: _passwordController.text,
                      ),
                    );
                  }
                },
                child: Text('Login'),
              ),
            ),
          ],
        ),
      ),
    );
  }
  
  @override
  void dispose() {
    _emailController.dispose();
    _passwordController.dispose();
    super.dispose();
  }
}
```

## Flutter 应用实例

### 待办事项应用

```dart
// 待办事项模型
class Todo extends Equatable {
  final String id;
  final String title;
  final String description;
  final bool isCompleted;
  final DateTime createdAt;
  final DateTime? completedAt;
  
  const Todo({
    required this.id,
    required this.title,
    required this.description,
    required this.isCompleted,
    required this.createdAt,
    this.completedAt,
  });
  
  Todo copyWith({
    String? id,
    String? title,
    String? description,
    bool? isCompleted,
    DateTime? createdAt,
    DateTime? completedAt,
  }) {
    return Todo(
      id: id ?? this.id,
      title: title ?? this.title,
      description: description ?? this.description,
      isCompleted: isCompleted ?? this.isCompleted,
      createdAt: createdAt ?? this.createdAt,
      completedAt: completedAt ?? this.completedAt,
    );
  }
  
  @override
  List<Object?> get props => [
    id,
    title,
    description,
    isCompleted,
    createdAt,
    completedAt,
  ];
}

// 待办事项事件
abstract class TodoEvent extends Equatable {
  const TodoEvent();
  
  @override
  List<Object?> get props => [];
}

class TodosLoaded extends TodoEvent {}

class TodoAdded extends TodoEvent {
  final String title;
  final String description;
  
  const TodoAdded({required this.title, required this.description});
  
  @override
  List<Object> get props => [title, description];
}

class TodoUpdated extends TodoEvent {
  final Todo todo;
  
  const TodoUpdated({required this.todo});
  
  @override
  List<Object> get props => [todo];
}

class TodoDeleted extends TodoEvent {
  final String id;
  
  const TodoDeleted({required this.id});
  
  @override
  List<Object> get props => [id];
}

class TodoToggled extends TodoEvent {
  final String id;
  
  const TodoToggled({required this.id});
  
  @override
  List<Object> get props => [id];
}

class TodosFiltered extends TodoEvent {
  final TodoFilter filter;
  
  const TodosFiltered({required this.filter});
  
  @override
  List<Object> get props => [filter];
}

// 待办事项过滤器
enum TodoFilter { all, active, completed }

// 待办事项状态
class TodoState extends Equatable {
  final List<Todo> todos;
  final TodoFilter filter;
  final bool isLoading;
  final String? error;
  
  const TodoState({
    required this.todos,
    required this.filter,
    required this.isLoading,
    this.error,
  });
  
  factory TodoState.initial() {
    return const TodoState(
      todos: [],
      filter: TodoFilter.all,
      isLoading: false,
    );
  }
  
  List<Todo> get filteredTodos {
    switch (filter) {
      case TodoFilter.all:
        return todos;
      case TodoFilter.active:
        return todos.where((todo) => !todo.isCompleted).toList();
      case TodoFilter.completed:
        return todos.where((todo) => todo.isCompleted).toList();
    }
  }
  
  int get activeCount => todos.where((todo) => !todo.isCompleted).length;
  int get completedCount => todos.where((todo) => todo.isCompleted).length;
  
  TodoState copyWith({
    List<Todo>? todos,
    TodoFilter? filter,
    bool? isLoading,
    String? error,
  }) {
    return TodoState(
      todos: todos ?? this.todos,
      filter: filter ?? this.filter,
      isLoading: isLoading ?? this.isLoading,
      error: error ?? this.error,
    );
  }
  
  @override
  List<Object?> get props => [todos, filter, isLoading, error];
}

// 待办事项 BLoC
class TodoBloc extends Bloc<TodoEvent, TodoState> {
  final TodoRepository _todoRepository;
  
  TodoBloc({required TodoRepository todoRepository})
      : _todoRepository = todoRepository,
        super(TodoState.initial()) {
    on<TodosLoaded>(_onTodosLoaded);
    on<TodoAdded>(_onTodoAdded);
    on<TodoUpdated>(_onTodoUpdated);
    on<TodoDeleted>(_onTodoDeleted);
    on<TodoToggled>(_onTodoToggled);
    on<TodosFiltered>(_onTodosFiltered);
  }
  
  Future<void> _onTodosLoaded(
    TodosLoaded event,
    Emitter<TodoState> emit,
  ) async {
    emit(state.copyWith(isLoading: true, error: null));
    
    try {
      final todos = await _todoRepository.getTodos();
      emit(state.copyWith(todos: todos, isLoading: false));
    } catch (e) {
      emit(state.copyWith(
        isLoading: false,
        error: 'Failed to load todos: ${e.toString()}',
      ));
    }
  }
  
  Future<void> _onTodoAdded(
    TodoAdded event,
    Emitter<TodoState> emit,
  ) async {
    try {
      final todo = Todo(
        id: DateTime.now().millisecondsSinceEpoch.toString(),
        title: event.title,
        description: event.description,
        isCompleted: false,
        createdAt: DateTime.now(),
      );
      
      await _todoRepository.addTodo(todo);
      
      final updatedTodos = List<Todo>.from(state.todos)..add(todo);
      emit(state.copyWith(todos: updatedTodos));
    } catch (e) {
      emit(state.copyWith(
        error: 'Failed to add todo: ${e.toString()}',
      ));
    }
  }
  
  Future<void> _onTodoUpdated(
    TodoUpdated event,
    Emitter<TodoState> emit,
  ) async {
    try {
      await _todoRepository.updateTodo(event.todo);
      
      final updatedTodos = state.todos.map((todo) {
        return todo.id == event.todo.id ? event.todo : todo;
      }).toList();
      
      emit(state.copyWith(todos: updatedTodos));
    } catch (e) {
      emit(state.copyWith(
        error: 'Failed to update todo: ${e.toString()}',
      ));
    }
  }
  
  Future<void> _onTodoDeleted(
    TodoDeleted event,
    Emitter<TodoState> emit,
  ) async {
    try {
      await _todoRepository.deleteTodo(event.id);
      
      final updatedTodos = state.todos
          .where((todo) => todo.id != event.id)
          .toList();
      
      emit(state.copyWith(todos: updatedTodos));
    } catch (e) {
      emit(state.copyWith(
        error: 'Failed to delete todo: ${e.toString()}',
      ));
    }
  }
  
  Future<void> _onTodoToggled(
    TodoToggled event,
    Emitter<TodoState> emit,
  ) async {
    final todoIndex = state.todos.indexWhere((todo) => todo.id == event.id);
    if (todoIndex == -1) return;
    
    final todo = state.todos[todoIndex];
    final updatedTodo = todo.copyWith(
      isCompleted: !todo.isCompleted,
      completedAt: !todo.isCompleted ? DateTime.now() : null,
    );
    
    add(TodoUpdated(todo: updatedTodo));
  }
  
  void _onTodosFiltered(
    TodosFiltered event,
    Emitter<TodoState> emit,
  ) {
    emit(state.copyWith(filter: event.filter));
  }
}

// 待办事项页面
class TodoPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Todo List'),
        actions: [
          PopupMenuButton<TodoFilter>(
            onSelected: (filter) {
              context.read<TodoBloc>().add(TodosFiltered(filter: filter));
            },
            itemBuilder: (context) => [
              PopupMenuItem(
                value: TodoFilter.all,
                child: Text('All'),
              ),
              PopupMenuItem(
                value: TodoFilter.active,
                child: Text('Active'),
              ),
              PopupMenuItem(
                value: TodoFilter.completed,
                child: Text('Completed'),
              ),
            ],
          ),
        ],
      ),
      body: BlocConsumer<TodoBloc, TodoState>(
        listener: (context, state) {
          if (state.error != null) {
            ScaffoldMessenger.of(context).showSnackBar(
              SnackBar(
                content: Text(state.error!),
                backgroundColor: Colors.red,
              ),
            );
          }
        },
        builder: (context, state) {
          if (state.isLoading && state.todos.isEmpty) {
            return Center(child: CircularProgressIndicator());
          }
          
          return Column(
            children: [
              // 统计信息
              Container(
                padding: EdgeInsets.all(16),
                child: Row(
                  mainAxisAlignment: MainAxisAlignment.spaceAround,
                  children: [
                    _buildStatCard('Total', state.todos.length),
                    _buildStatCard('Active', state.activeCount),
                    _buildStatCard('Completed', state.completedCount),
                  ],
                ),
              ),
              // 待办事项列表
              Expanded(
                child: state.filteredTodos.isEmpty
                    ? Center(
                        child: Text(
                          'No todos found',
                          style: TextStyle(fontSize: 18, color: Colors.grey),
                        ),
                      )
                    : ListView.builder(
                        itemCount: state.filteredTodos.length,
                        itemBuilder: (context, index) {
                          final todo = state.filteredTodos[index];
                          return TodoItem(todo: todo);
                        },
                      ),
              ),
            ],
          );
        },
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () {
          _showAddTodoDialog(context);
        },
        child: Icon(Icons.add),
      ),
    );
  }
  
  Widget _buildStatCard(String label, int count) {
    return Card(
      child: Padding(
        padding: EdgeInsets.all(16),
        child: Column(
          children: [
            Text(
              '$count',
              style: TextStyle(fontSize: 24, fontWeight: FontWeight.bold),
            ),
            Text(label),
          ],
        ),
      ),
    );
  }
  
  void _showAddTodoDialog(BuildContext context) {
    showDialog(
      context: context,
      builder: (dialogContext) => AddTodoDialog(),
    );
  }
}

// 待办事项条目
class TodoItem extends StatelessWidget {
  final Todo todo;
  
  const TodoItem({Key? key, required this.todo}) : super(key: key);
  
  @override
  Widget build(BuildContext context) {
    return Card(
      margin: EdgeInsets.symmetric(horizontal: 16, vertical: 4),
      child: ListTile(
        leading: Checkbox(
          value: todo.isCompleted,
          onChanged: (value) {
            context.read<TodoBloc>().add(TodoToggled(id: todo.id));
          },
        ),
        title: Text(
          todo.title,
          style: TextStyle(
            decoration: todo.isCompleted
                ? TextDecoration.lineThrough
                : TextDecoration.none,
          ),
        ),
        subtitle: todo.description.isNotEmpty
            ? Text(
                todo.description,
                style: TextStyle(
                  decoration: todo.isCompleted
                      ? TextDecoration.lineThrough
                      : TextDecoration.none,
                ),
              )
            : null,
        trailing: PopupMenuButton(
          itemBuilder: (context) => [
            PopupMenuItem(
              value: 'edit',
              child: Text('Edit'),
            ),
            PopupMenuItem(
              value: 'delete',
              child: Text('Delete'),
            ),
          ],
          onSelected: (value) {
            if (value == 'edit') {
              _showEditTodoDialog(context, todo);
            } else if (value == 'delete') {
              context.read<TodoBloc>().add(TodoDeleted(id: todo.id));
            }
          },
        ),
      ),
    );
  }
  
  void _showEditTodoDialog(BuildContext context, Todo todo) {
    showDialog(
      context: context,
      builder: (dialogContext) => EditTodoDialog(todo: todo),
    );
  }
}

// 添加待办事项对话框
class AddTodoDialog extends StatefulWidget {
  @override
  _AddTodoDialogState createState() => _AddTodoDialogState();
}

class _AddTodoDialogState extends State<AddTodoDialog> {
  final _titleController = TextEditingController();
  final _descriptionController = TextEditingController();
  final _formKey = GlobalKey<FormState>();
  
  @override
  Widget build(BuildContext context) {
    return AlertDialog(
      title: Text('Add Todo'),
      content: Form(
        key: _formKey,
        child: Column(
          mainAxisSize: MainAxisSize.min,
          children: [
            TextFormField(
              controller: _titleController,
              decoration: InputDecoration(
                labelText: 'Title',
                border: OutlineInputBorder(),
              ),
              validator: (value) {
                if (value == null || value.trim().isEmpty) {
                  return 'Please enter a title';
                }
                return null;
              },
            ),
            SizedBox(height: 16),
            TextFormField(
              controller: _descriptionController,
              decoration: InputDecoration(
                labelText: 'Description (optional)',
                border: OutlineInputBorder(),
              ),
              maxLines: 3,
            ),
          ],
        ),
      ),
      actions: [
        TextButton(
          onPressed: () => Navigator.of(context).pop(),
          child: Text('Cancel'),
        ),
        ElevatedButton(
          onPressed: () {
            if (_formKey.currentState!.validate()) {
              context.read<TodoBloc>().add(
                TodoAdded(
                  title: _titleController.text.trim(),
                  description: _descriptionController.text.trim(),
                ),
              );
              Navigator.of(context).pop();
            }
          },
          child: Text('Add'),
        ),
      ],
    );
  }
  
  @override
  void dispose() {
    _titleController.dispose();
    _descriptionController.dispose();
    super.dispose();
  }
}

// 编辑待办事项对话框
class EditTodoDialog extends StatefulWidget {
  final Todo todo;
  
  const EditTodoDialog({Key? key, required this.todo}) : super(key: key);
  
  @override
  _EditTodoDialogState createState() => _EditTodoDialogState();
}

class _EditTodoDialogState extends State<EditTodoDialog> {
  late final TextEditingController _titleController;
  late final TextEditingController _descriptionController;
  final _formKey = GlobalKey<FormState>();
  
  @override
  void initState() {
    super.initState();
    _titleController = TextEditingController(text: widget.todo.title);
    _descriptionController = TextEditingController(text: widget.todo.description);
  }
  
  @override
  Widget build(BuildContext context) {
    return AlertDialog(
      title: Text('Edit Todo'),
      content: Form(
        key: _formKey,
        child: Column(
          mainAxisSize: MainAxisSize.min,
          children: [
            TextFormField(
              controller: _titleController,
              decoration: InputDecoration(
                labelText: 'Title',
                border: OutlineInputBorder(),
              ),
              validator: (value) {
                if (value == null || value.trim().isEmpty) {
                  return 'Please enter a title';
                }
                return null;
              },
            ),
            SizedBox(height: 16),
            TextFormField(
              controller: _descriptionController,
              decoration: InputDecoration(
                labelText: 'Description (optional)',
                border: OutlineInputBorder(),
              ),
              maxLines: 3,
            ),
          ],
        ),
      ),
      actions: [
        TextButton(
          onPressed: () => Navigator.of(context).pop(),
          child: Text('Cancel'),
        ),
        ElevatedButton(
          onPressed: () {
            if (_formKey.currentState!.validate()) {
              final updatedTodo = widget.todo.copyWith(
                title: _titleController.text.trim(),
                description: _descriptionController.text.trim(),
              );
              
              context.read<TodoBloc>().add(TodoUpdated(todo: updatedTodo));
              Navigator.of(context).pop();
            }
          },
          child: Text('Update'),
        ),
      ],
    );
  }
  
  @override
  void dispose() {
    _titleController.dispose();
    _descriptionController.dispose();
    super.dispose();
  }
}
```

### 天气应用

```dart
// 天气事件
abstract class WeatherEvent extends Equatable {
  const WeatherEvent();
  
  @override
  List<Object?> get props => [];
}

class WeatherRequested extends WeatherEvent {
  final String city;
  
  const WeatherRequested({required this.city});
  
  @override
  List<Object> get props => [city];
}

class WeatherRefreshed extends WeatherEvent {
  final String city;
  
  const WeatherRefreshed({required this.city});
  
  @override
  List<Object> get props => [city];
}

// 天气状态
abstract class WeatherState extends Equatable {
  const WeatherState();
  
  @override
  List<Object?> get props => [];
}

class WeatherInitial extends WeatherState {}

class WeatherLoading extends WeatherState {}

class WeatherLoaded extends WeatherState {
  final Weather weather;
  final DateTime lastUpdated;
  
  const WeatherLoaded({
    required this.weather,
    required this.lastUpdated,
  });
  
  @override
  List<Object> get props => [weather, lastUpdated];
}

class WeatherError extends WeatherState {
  final String message;
  
  const WeatherError({required this.message});
  
  @override
  List<Object> get props => [message];
}

// 天气 BLoC
class WeatherBloc extends Bloc<WeatherEvent, WeatherState> {
  final WeatherRepository _weatherRepository;
  
  WeatherBloc({required WeatherRepository weatherRepository})
      : _weatherRepository = weatherRepository,
        super(WeatherInitial()) {
    on<WeatherRequested>(_onWeatherRequested);
    on<WeatherRefreshed>(_onWeatherRefreshed);
  }
  
  Future<void> _onWeatherRequested(
    WeatherRequested event,
    Emitter<WeatherState> emit,
  ) async {
    emit(WeatherLoading());
    await _fetchWeather(event.city, emit);
  }
  
  Future<void> _onWeatherRefreshed(
    WeatherRefreshed event,
    Emitter<WeatherState> emit,
  ) async {
    if (state is! WeatherLoaded) return;
    
    try {
      final weather = await _weatherRepository.getWeather(event.city);
      emit(WeatherLoaded(
        weather: weather,
        lastUpdated: DateTime.now(),
      ));
    } catch (e) {
      emit(WeatherError(message: e.toString()));
    }
  }
  
  Future<void> _fetchWeather(
    String city,
    Emitter<WeatherState> emit,
  ) async {
    try {
      final weather = await _weatherRepository.getWeather(city);
      emit(WeatherLoaded(
        weather: weather,
        lastUpdated: DateTime.now(),
      ));
    } catch (e) {
      emit(WeatherError(message: e.toString()));
    }
  }
}

// 天气页面
class WeatherPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Weather'),
        actions: [
          BlocBuilder<WeatherBloc, WeatherState>(
            builder: (context, state) {
              if (state is WeatherLoaded) {
                return IconButton(
                  icon: Icon(Icons.refresh),
                  onPressed: () {
                    context.read<WeatherBloc>().add(
                      WeatherRefreshed(city: state.weather.city),
                    );
                  },
                );
              }
              return SizedBox.shrink();
            },
          ),
        ],
      ),
      body: BlocBuilder<WeatherBloc, WeatherState>(
        builder: (context, state) {
          if (state is WeatherInitial) {
            return WeatherSearch();
          }
          
          if (state is WeatherLoading) {
            return Center(child: CircularProgressIndicator());
          }
          
          if (state is WeatherLoaded) {
            return WeatherDisplay(weather: state.weather);
          }
          
          if (state is WeatherError) {
            return WeatherError(message: state.message);
          }
          
          return SizedBox.shrink();
        },
      ),
    );
  }
}

// 天气搜索组件
class WeatherSearch extends StatefulWidget {
  @override
  _WeatherSearchState createState() => _WeatherSearchState();
}

class _WeatherSearchState extends State<WeatherSearch> {
  final _cityController = TextEditingController();
  
  @override
  Widget build(BuildContext context) {
    return Center(
      child: Padding(
        padding: EdgeInsets.all(16),
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            TextField(
              controller: _cityController,
              decoration: InputDecoration(
                labelText: 'Enter city name',
                border: OutlineInputBorder(),
                suffixIcon: IconButton(
                  icon: Icon(Icons.search),
                  onPressed: _searchWeather,
                ),
              ),
              onSubmitted: (_) => _searchWeather(),
            ),
            SizedBox(height: 16),
            ElevatedButton(
              onPressed: _searchWeather,
              child: Text('Get Weather'),
            ),
          ],
        ),
      ),
    );
  }
  
  void _searchWeather() {
    final city = _cityController.text.trim();
    if (city.isNotEmpty) {
      context.read<WeatherBloc>().add(WeatherRequested(city: city));
    }
  }
  
  @override
  void dispose() {
    _cityController.dispose();
    super.dispose();
  }
}

// 天气显示组件
class WeatherDisplay extends StatelessWidget {
  final Weather weather;
  
  const WeatherDisplay({Key? key, required this.weather}) : super(key: key);
  
  @override
  Widget build(BuildContext context) {
    return Center(
      child: Column(
        mainAxisAlignment: MainAxisAlignment.center,
        children: [
          Text(
            weather.city,
            style: TextStyle(fontSize: 32, fontWeight: FontWeight.bold),
          ),
          SizedBox(height: 16),
          Text(
            '${weather.temperature.round()}°C',
            style: TextStyle(fontSize: 64),
          ),
          Text(
            weather.description,
            style: TextStyle(fontSize: 20),
          ),
          SizedBox(height: 32),
          Card(
            child: Padding(
              padding: EdgeInsets.all(16),
              child: Column(
                children: [
                  _buildWeatherDetail('Feels like', '${weather.feelsLike.round()}°C'),
                  _buildWeatherDetail('Humidity', '${weather.humidity}%'),
                  _buildWeatherDetail('Wind Speed', '${weather.windSpeed} m/s'),
                  _buildWeatherDetail('Pressure', '${weather.pressure} hPa'),
                ],
              ),
            ),
          ),
        ],
      ),
    );
  }
  
  Widget _buildWeatherDetail(String label, String value) {
    return Padding(
      padding: EdgeInsets.symmetric(vertical: 4),
      child: Row(
        mainAxisAlignment: MainAxisAlignment.spaceBetween,
        children: [
          Text(label),
          Text(value, style: TextStyle(fontWeight: FontWeight.bold)),
        ],
      ),
    );
  }
}

// 天气错误组件
class WeatherErrorWidget extends StatelessWidget {
  final String message;
  
  const WeatherErrorWidget({Key? key, required this.message}) : super(key: key);
  
  @override
  Widget build(BuildContext context) {
    return Center(
      child: Column(
        mainAxisAlignment: MainAxisAlignment.center,
        children: [
          Icon(
            Icons.error_outline,
            size: 64,
            color: Colors.red,
          ),
          SizedBox(height: 16),
          Text(
            'Error',
            style: TextStyle(fontSize: 24, fontWeight: FontWeight.bold),
          ),
          SizedBox(height: 8),
          Text(
            message,
            textAlign: TextAlign.center,
            style: TextStyle(fontSize: 16),
          ),
          SizedBox(height: 16),
          ElevatedButton(
            onPressed: () {
              // 返回搜索页面
              context.read<WeatherBloc>().add(WeatherInitial());
            },
            child: Text('Try Again'),
          ),
        ],
      ),
    );
  }
}
```

## 测试和调试

### BLoC 测试

```dart
import 'package:bloc_test/bloc_test.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:mocktail/mocktail.dart';

// Mock 类
class MockTodoRepository extends Mock implements TodoRepository {}

void main() {
  group('TodoBloc', () {
    late TodoRepository todoRepository;
    late TodoBloc todoBloc;
    
    setUp(() {
      todoRepository = MockTodoRepository();
      todoBloc = TodoBloc(todoRepository: todoRepository);
    });
    
    tearDown(() {
      todoBloc.close();
    });
    
    test('初始状态应该是 TodoState.initial()', () {
      expect(todoBloc.state, TodoState.initial());
    });
    
    group('TodosLoaded', () {
      final todos = [
        Todo(
          id: '1',
          title: 'Test Todo',
          description: 'Test Description',
          isCompleted: false,
          createdAt: DateTime.now(),
        ),
      ];
      
      blocTest<TodoBloc, TodoState>(
        '成功加载待办事项时应该发出 [loading, loaded] 状态',
        build: () {
          when(() => todoRepository.getTodos())
              .thenAnswer((_) async => todos);
          return todoBloc;
        },
        act: (bloc) => bloc.add(TodosLoaded()),
        expect: () => [
          TodoState.initial().copyWith(isLoading: true),
          TodoState.initial().copyWith(todos: todos, isLoading: false),
        ],
        verify: (_) {
          verify(() => todoRepository.getTodos()).called(1);
        },
      );
      
      blocTest<TodoBloc, TodoState>(
        '加载失败时应该发出 [loading, error] 状态',
        build: () {
          when(() => todoRepository.getTodos())
              .thenThrow(Exception('Failed to load'));
          return todoBloc;
        },
        act: (bloc) => bloc.add(TodosLoaded()),
        expect: () => [
          TodoState.initial().copyWith(isLoading: true),
          TodoState.initial().copyWith(
            isLoading: false,
            error: 'Failed to load todos: Exception: Failed to load',
          ),
        ],
      );
    });
    
    group('TodoAdded', () {
      const title = 'New Todo';
      const description = 'New Description';
      
      blocTest<TodoBloc, TodoState>(
        '成功添加待办事项时应该更新状态',
        build: () {
          when(() => todoRepository.addTodo(any()))
              .thenAnswer((_) async {});
          return todoBloc;
        },
        act: (bloc) => bloc.add(TodoAdded(
          title: title,
          description: description,
        )),
        expect: () => [
          predicate<TodoState>((state) {
            return state.todos.length == 1 &&
                   state.todos.first.title == title &&
                   state.todos.first.description == description &&
                   !state.todos.first.isCompleted;
          }),
        ],
        verify: (_) {
          verify(() => todoRepository.addTodo(any())).called(1);
        },
      );
    });
    
    group('TodoToggled', () {
      final todo = Todo(
        id: '1',
        title: 'Test Todo',
        description: 'Test Description',
        isCompleted: false,
        createdAt: DateTime.now(),
      );
      
      blocTest<TodoBloc, TodoState>(
        '切换待办事项状态时应该更新完成状态',
        build: () {
          when(() => todoRepository.updateTodo(any()))
              .thenAnswer((_) async {});
          return todoBloc;
        },
        seed: () => TodoState.initial().copyWith(todos: [todo]),
        act: (bloc) => bloc.add(TodoToggled(id: todo.id)),
        expect: () => [
          predicate<TodoState>((state) {
            return state.todos.first.isCompleted == true &&
                   state.todos.first.completedAt != null;
          }),
        ],
      );
    });
  });
  
  group('CounterBloc', () {
    late CounterBloc counterBloc;
    
    setUp(() {
      counterBloc = CounterBloc();
    });
    
    tearDown(() {
      counterBloc.close();
    });
    
    test('初始状态应该是 CounterState(count: 0)', () {
      expect(counterBloc.state, const CounterState(count: 0));
    });
    
    blocTest<CounterBloc, CounterState>(
      '增加计数时应该发出 count + 1',
      build: () => counterBloc,
      act: (bloc) => bloc.add(CounterIncrement()),
      expect: () => [const CounterState(count: 1)],
    );
    
    blocTest<CounterBloc, CounterState>(
      '减少计数时应该发出 count - 1',
      build: () => counterBloc,
      seed: () => const CounterState(count: 1),
      act: (bloc) => bloc.add(CounterDecrement()),
      expect: () => [const CounterState(count: 0)],
    );
    
    blocTest<CounterBloc, CounterState>(
      '重置计数时应该发出 count = 0',
      build: () => counterBloc,
      seed: () => const CounterState(count: 10),
      act: (bloc) => bloc.add(CounterReset()),
      expect: () => [const CounterState(count: 0)],
    );
    
    blocTest<CounterBloc, CounterState>(
      '多个事件应该按顺序处理',
      build: () => counterBloc,
      act: (bloc) => bloc
        ..add(CounterIncrement())
        ..add(CounterIncrement())
        ..add(CounterDecrement()),
      expect: () => [
        const CounterState(count: 1),
        const CounterState(count: 2),
        const CounterState(count: 1),
      ],
    );
  });
}
```

### BLoC 调试器

```dart
class BlocDebugger {
  static final Map<String, BlocStats> _stats = {};
  static final List<BlocEvent> _events = [];
  
  static void logEvent(String blocName, dynamic event) {
    final stats = _stats.putIfAbsent(blocName, () => BlocStats(blocName));
    stats.incrementEvent(event.runtimeType.toString());
    
    final blocEvent = BlocEvent(
      blocName: blocName,
      eventType: event.runtimeType.toString(),
      timestamp: DateTime.now(),
      event: event,
    );
    _events.add(blocEvent);
    
    if (kDebugMode) {
      print('BLoC事件: $blocName -> ${event.runtimeType}');
    }
  }
  
  static void logStateChange(String blocName, dynamic previousState, dynamic currentState) {
    final stats = _stats.putIfAbsent(blocName, () => BlocStats(blocName));
    stats.incrementStateChange();
    
    if (kDebugMode) {
      print('BLoC状态变化: $blocName');
      print('  从: ${previousState.runtimeType}');
      print('  到: ${currentState.runtimeType}');
    }
  }
  
  static Map<String, BlocStats> getStats() => Map.unmodifiable(_stats);
  
  static List<BlocEvent> getEvents() => List.unmodifiable(_events);
  
  static void clearHistory() {
    _stats.clear();
    _events.clear();
  }
  
  static void printStatistics() {
    print('=== BLoC 统计信息 ===');
    print('BLoC数量: ${_stats.length}');
    print('总事件数量: ${_events.length}');
    print('\n各BLoC统计:');
    _stats.values.forEach((stats) {
      print('  ${stats.blocName}:');
      print('    事件数量: ${stats.totalEvents}');
      print('    状态变化: ${stats.stateChanges}');
      print('    事件类型: ${stats.eventTypes}');
    });
  }
}

class BlocStats {
  final String blocName;
  final Map<String, int> eventTypes = {};
  int stateChanges = 0;
  
  BlocStats(this.blocName);
  
  void incrementEvent(String eventType) {
    eventTypes[eventType] = (eventTypes[eventType] ?? 0) + 1;
  }
  
  void incrementStateChange() {
    stateChanges++;
  }
  
  int get totalEvents => eventTypes.values.fold(0, (sum, count) => sum + count);
}

class BlocEvent {
  final String blocName;
  final String eventType;
  final DateTime timestamp;
  final dynamic event;
  
  BlocEvent({
    required this.blocName,
    required this.eventType,
    required this.timestamp,
    required this.event,
  });
}

// 带调试功能的 BLoC 基类
abstract class DebuggableBloc<Event, State> extends Bloc<Event, State> {
  DebuggableBloc(State initialState) : super(initialState);
  
  @override
  void add(Event event) {
    if (kDebugMode) {
      BlocDebugger.logEvent(runtimeType.toString(), event);
    }
    super.add(event);
  }
  
  @override
  void emit(State state) {
    if (kDebugMode) {
      BlocDebugger.logStateChange(
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

1. **单一职责原则**
   ```dart
   // 好的做法：每个 BLoC 负责单一功能
   class UserBloc extends Bloc<UserEvent, UserState> {
     // 只处理用户相关逻辑
   }
   
   class CartBloc extends Bloc<CartEvent, CartState> {
     // 只处理购物车相关逻辑
   }
   ```

2. **使用 Equatable**
   ```dart
   // 确保状态和事件的相等性比较
   class MyState extends Equatable {
     final String value;
     
     const MyState({required this.value});
     
     @override
     List<Object> get props => [value];
   }
   ```

3. **不可变状态**
   ```dart
   // 使用 copyWith 方法创建新状态
   class MyState extends Equatable {
     final String value;
     final bool isLoading;
     
     const MyState({required this.value, required this.isLoading});
     
     MyState copyWith({String? value, bool? isLoading}) {
       return MyState(
         value: value ?? this.value,
         isLoading: isLoading ?? this.isLoading,
       );
     }
     
     @override
     List<Object> get props => [value, isLoading];
   }
   ```

### 实现建议

1. **合理的状态设计**
   ```dart
   // 包含加载、错误、数据状态
   class DataState extends Equatable {
     final List<Item> items;
     final bool isLoading;
     final String? error;
     
     const DataState({
       required this.items,
       required this.isLoading,
       this.error,
     });
   }
   ```

2. **使用 BlocConsumer 处理副作用**
   ```dart
   BlocConsumer<MyBloc, MyState>(
     listener: (context, state) {
       if (state is ErrorState) {
         ScaffoldMessenger.of(context).showSnackBar(
           SnackBar(content: Text(state.message)),
         );
       }
     },
     builder: (context, state) {
       // UI 构建逻辑
     },
   )
   ```

3. **依赖注入**
   ```dart
   // 通过构造函数注入依赖
   class MyBloc extends Bloc<MyEvent, MyState> {
     final Repository repository;
     
     MyBloc({required this.repository}) : super(InitialState());
   }
   ```

### 注意事项

1. **避免在 BLoC 中直接访问 UI**
2. **合理管理 BLoC 的生命周期**
3. **使用 Repository 模式分离数据层**
4. **编写充分的单元测试**
5. **避免过度复杂的状态设计**

BLoC 模式提供了强大的状态管理能力，特别适合大型应用。通过遵循最佳实践，可以构建出可维护、可测试的 Flutter 应用。