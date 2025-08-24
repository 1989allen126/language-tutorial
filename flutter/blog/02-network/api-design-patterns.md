# Flutter API设计模式与RESTful最佳实践

本文档详细介绍API设计模式、RESTful架构原则以及在Flutter中的最佳实践。

## 目录

1. [RESTful API设计原则](#1-restful-api设计原则)
2. [API版本管理](#2-api版本管理)
3. [数据传输对象模式](#3-数据传输对象模式)
4. [API客户端设计模式](#4-api客户端设计模式)
5. [错误处理与状态码](#5-错误处理与状态码)
6. [API文档与测试](#6-api文档与测试)
7. [GraphQL集成](#7-graphql集成)

## 1. RESTful API设计原则

### 1.1 资源设计

```dart
// 用户资源API设计
abstract class UserApi {
  // GET /api/v1/users - 获取用户列表
  Future<PaginatedResponse<User>> getUsers({
    int page = 1,
    int limit = 20,
    String? search,
    UserStatus? status,
    String? sortBy,
    SortOrder sortOrder = SortOrder.asc,
  });
  
  // GET /api/v1/users/{id} - 获取单个用户
  Future<User> getUser(int id);
  
  // POST /api/v1/users - 创建用户
  Future<User> createUser(CreateUserRequest request);
  
  // PUT /api/v1/users/{id} - 更新用户
  Future<User> updateUser(int id, UpdateUserRequest request);
  
  // PATCH /api/v1/users/{id} - 部分更新用户
  Future<User> patchUser(int id, PatchUserRequest request);
  
  // DELETE /api/v1/users/{id} - 删除用户
  Future<void> deleteUser(int id);
  
  // GET /api/v1/users/{id}/posts - 获取用户的文章
  Future<PaginatedResponse<Post>> getUserPosts(
    int userId, {
    int page = 1,
    int limit = 20,
  });
  
  // POST /api/v1/users/{id}/avatar - 上传用户头像
  Future<User> uploadAvatar(int id, File avatarFile);
}

// 分页响应模型
class PaginatedResponse<T> {
  final List<T> data;
  final PaginationMeta meta;
  final List<Link> links;
  
  const PaginatedResponse({
    required this.data,
    required this.meta,
    required this.links,
  });
  
  factory PaginatedResponse.fromJson(
    Map<String, dynamic> json,
    T Function(Map<String, dynamic>) fromJsonT,
  ) {
    return PaginatedResponse(
      data: (json['data'] as List)
          .map((item) => fromJsonT(item as Map<String, dynamic>))
          .toList(),
      meta: PaginationMeta.fromJson(json['meta']),
      links: (json['links'] as List)
          .map((link) => Link.fromJson(link as Map<String, dynamic>))
          .toList(),
    );
  }
}

class PaginationMeta {
  final int currentPage;
  final int lastPage;
  final int perPage;
  final int total;
  final int from;
  final int to;
  
  const PaginationMeta({
    required this.currentPage,
    required this.lastPage,
    required this.perPage,
    required this.total,
    required this.from,
    required this.to,
  });
  
  factory PaginationMeta.fromJson(Map<String, dynamic> json) {
    return PaginationMeta(
      currentPage: json['current_page'],
      lastPage: json['last_page'],
      perPage: json['per_page'],
      total: json['total'],
      from: json['from'],
      to: json['to'],
    );
  }
}

class Link {
  final String? url;
  final String label;
  final bool active;
  
  const Link({
    this.url,
    required this.label,
    required this.active,
  });
  
  factory Link.fromJson(Map<String, dynamic> json) {
    return Link(
      url: json['url'],
      label: json['label'],
      active: json['active'],
    );
  }
}
```

### 1.2 HTTP方法使用规范

```dart
class HttpMethodGuide {
  // GET - 获取资源（幂等、安全）
  static const String get = 'GET';
  
  // POST - 创建资源（非幂等）
  static const String post = 'POST';
  
  // PUT - 完整更新资源（幂等）
  static const String put = 'PUT';
  
  // PATCH - 部分更新资源（非幂等）
  static const String patch = 'PATCH';
  
  // DELETE - 删除资源（幂等）
  static const String delete = 'DELETE';
  
  // HEAD - 获取资源头信息（幂等、安全）
  static const String head = 'HEAD';
  
  // OPTIONS - 获取资源支持的方法（幂等、安全）
  static const String options = 'OPTIONS';
}

// HTTP状态码规范
class HttpStatusCodes {
  // 2xx 成功
  static const int ok = 200;                    // 请求成功
  static const int created = 201;               // 资源创建成功
  static const int accepted = 202;              // 请求已接受，正在处理
  static const int noContent = 204;             // 请求成功，无返回内容
  
  // 3xx 重定向
  static const int notModified = 304;           // 资源未修改
  
  // 4xx 客户端错误
  static const int badRequest = 400;            // 请求参数错误
  static const int unauthorized = 401;          // 未认证
  static const int forbidden = 403;             // 无权限
  static const int notFound = 404;              // 资源不存在
  static const int methodNotAllowed = 405;      // 方法不允许
  static const int conflict = 409;              // 资源冲突
  static const int unprocessableEntity = 422;   // 请求格式正确但语义错误
  static const int tooManyRequests = 429;       // 请求过于频繁
  
  // 5xx 服务器错误
  static const int internalServerError = 500;   // 服务器内部错误
  static const int badGateway = 502;            // 网关错误
  static const int serviceUnavailable = 503;    // 服务不可用
  static const int gatewayTimeout = 504;        // 网关超时
}
```

### 1.3 URL设计规范

```dart
class UrlDesignPatterns {
  // 资源命名规范
  static const Map<String, String> resourceNaming = {
    '集合资源': '/api/v1/users',              // 复数形式
    '单个资源': '/api/v1/users/123',          // 使用ID标识
    '嵌套资源': '/api/v1/users/123/posts',    // 表示关联关系
    '过滤查询': '/api/v1/users?status=active&role=admin',
    '排序': '/api/v1/users?sort=created_at&order=desc',
    '分页': '/api/v1/users?page=2&limit=20',
    '字段选择': '/api/v1/users?fields=id,name,email',
    '搜索': '/api/v1/users?q=john&search_fields=name,email',
  };
  
  // URL构建器
  static String buildUrl({
    required String baseUrl,
    required String resource,
    String? id,
    String? subResource,
    Map<String, dynamic>? queryParams,
  }) {
    final buffer = StringBuffer(baseUrl);
    
    if (!baseUrl.endsWith('/')) {
      buffer.write('/');
    }
    
    buffer.write(resource);
    
    if (id != null) {
      buffer.write('/$id');
    }
    
    if (subResource != null) {
      buffer.write('/$subResource');
    }
    
    if (queryParams != null && queryParams.isNotEmpty) {
      final queryString = queryParams.entries
          .where((entry) => entry.value != null)
          .map((entry) => '${entry.key}=${Uri.encodeComponent(entry.value.toString())}')
          .join('&');
      
      if (queryString.isNotEmpty) {
        buffer.write('?$queryString');
      }
    }
    
    return buffer.toString();
  }
}

// 使用示例
class ApiUrlBuilder {
  static const String baseUrl = 'https://api.example.com/v1';
  
  static String users({Map<String, dynamic>? params}) {
    return UrlDesignPatterns.buildUrl(
      baseUrl: baseUrl,
      resource: 'users',
      queryParams: params,
    );
  }
  
  static String user(int id) {
    return UrlDesignPatterns.buildUrl(
      baseUrl: baseUrl,
      resource: 'users',
      id: id.toString(),
    );
  }
  
  static String userPosts(int userId, {Map<String, dynamic>? params}) {
    return UrlDesignPatterns.buildUrl(
      baseUrl: baseUrl,
      resource: 'users',
      id: userId.toString(),
      subResource: 'posts',
      queryParams: params,
    );
  }
}
```

## 2. API版本管理

### 2.1 版本控制策略

```dart
enum ApiVersionStrategy {
  url,      // /api/v1/users
  header,   // Accept: application/vnd.api+json;version=1
  query,    // /api/users?version=1
}

class ApiVersionManager {
  final ApiVersionStrategy strategy;
  final String currentVersion;
  final Map<String, String> supportedVersions;
  
  ApiVersionManager({
    required this.strategy,
    required this.currentVersion,
    required this.supportedVersions,
  });
  
  String buildVersionedUrl(String baseUrl, String endpoint) {
    switch (strategy) {
      case ApiVersionStrategy.url:
        return '$baseUrl/v$currentVersion$endpoint';
      case ApiVersionStrategy.query:
        final separator = endpoint.contains('?') ? '&' : '?';
        return '$baseUrl$endpoint${separator}version=$currentVersion';
      case ApiVersionStrategy.header:
        return '$baseUrl$endpoint';
    }
  }
  
  Map<String, String> getVersionHeaders() {
    switch (strategy) {
      case ApiVersionStrategy.header:
        return {
          'Accept': 'application/vnd.api+json;version=$currentVersion',
          'API-Version': currentVersion,
        };
      default:
        return {};
    }
  }
  
  bool isVersionSupported(String version) {
    return supportedVersions.containsKey(version);
  }
  
  String? getVersionDescription(String version) {
    return supportedVersions[version];
  }
}

// 版本兼容性处理
class ApiCompatibilityHandler {
  final Map<String, ApiVersionAdapter> adapters = {};
  
  void registerAdapter(String version, ApiVersionAdapter adapter) {
    adapters[version] = adapter;
  }
  
  T adaptResponse<T>(
    String version,
    Map<String, dynamic> response,
    T Function(Map<String, dynamic>) parser,
  ) {
    final adapter = adapters[version];
    if (adapter != null) {
      final adaptedResponse = adapter.adaptResponse(response);
      return parser(adaptedResponse);
    }
    return parser(response);
  }
  
  Map<String, dynamic> adaptRequest(
    String version,
    Map<String, dynamic> request,
  ) {
    final adapter = adapters[version];
    if (adapter != null) {
      return adapter.adaptRequest(request);
    }
    return request;
  }
}

abstract class ApiVersionAdapter {
  Map<String, dynamic> adaptRequest(Map<String, dynamic> request);
  Map<String, dynamic> adaptResponse(Map<String, dynamic> response);
}

class V1ToV2Adapter extends ApiVersionAdapter {
  @override
  Map<String, dynamic> adaptRequest(Map<String, dynamic> request) {
    // 将v1请求格式转换为v2格式
    final adapted = Map<String, dynamic>.from(request);
    
    // 示例：v2中字段名变更
    if (adapted.containsKey('user_name')) {
      adapted['username'] = adapted.remove('user_name');
    }
    
    return adapted;
  }
  
  @override
  Map<String, dynamic> adaptResponse(Map<String, dynamic> response) {
    // 将v2响应格式转换为v1格式
    final adapted = Map<String, dynamic>.from(response);
    
    // 示例：v1中使用不同的字段名
    if (adapted.containsKey('username')) {
      adapted['user_name'] = adapted.remove('username');
    }
    
    return adapted;
  }
}
```

### 2.2 版本化API客户端

```dart
abstract class VersionedApiClient {
  String get version;
  Dio get dio;
  
  Future<T> request<T>(
    String method,
    String endpoint, {
    Map<String, dynamic>? data,
    Map<String, dynamic>? queryParameters,
    T Function(Map<String, dynamic>)? parser,
  });
}

class V1ApiClient extends VersionedApiClient {
  @override
  String get version => '1';
  
  @override
  final Dio dio;
  
  V1ApiClient({required this.dio});
  
  @override
  Future<T> request<T>(
    String method,
    String endpoint, {
    Map<String, dynamic>? data,
    Map<String, dynamic>? queryParameters,
    T Function(Map<String, dynamic>)? parser,
  }) async {
    final response = await dio.request(
      '/api/v1$endpoint',
      options: Options(method: method),
      data: data,
      queryParameters: queryParameters,
    );
    
    if (parser != null) {
      return parser(response.data);
    }
    
    return response.data as T;
  }
  
  // V1特有的方法
  Future<User> getUser(int id) async {
    return await request(
      'GET',
      '/users/$id',
      parser: (data) => User.fromJsonV1(data),
    );
  }
}

class V2ApiClient extends VersionedApiClient {
  @override
  String get version => '2';
  
  @override
  final Dio dio;
  
  V2ApiClient({required this.dio});
  
  @override
  Future<T> request<T>(
    String method,
    String endpoint, {
    Map<String, dynamic>? data,
    Map<String, dynamic>? queryParameters,
    T Function(Map<String, dynamic>)? parser,
  }) async {
    final response = await dio.request(
      '/api/v2$endpoint',
      options: Options(
        method: method,
        headers: {
          'Accept': 'application/vnd.api+json;version=2',
        },
      ),
      data: data,
      queryParameters: queryParameters,
    );
    
    if (parser != null) {
      return parser(response.data);
    }
    
    return response.data as T;
  }
  
  // V2特有的方法
  Future<UserV2> getUser(int id) async {
    return await request(
      'GET',
      '/users/$id',
      parser: (data) => UserV2.fromJson(data),
    );
  }
  
  // V2新增的批量操作
  Future<List<UserV2>> batchGetUsers(List<int> ids) async {
    return await request(
      'POST',
      '/users/batch',
      data: {'ids': ids},
      parser: (data) => (data['users'] as List)
          .map((user) => UserV2.fromJson(user))
          .toList(),
    );
  }
}

// API客户端工厂
class ApiClientFactory {
  static VersionedApiClient create(String version, Dio dio) {
    switch (version) {
      case '1':
        return V1ApiClient(dio: dio);
      case '2':
        return V2ApiClient(dio: dio);
      default:
        throw UnsupportedError('API version $version is not supported');
    }
  }
  
  static List<String> get supportedVersions => ['1', '2'];
  
  static String get latestVersion => '2';
  
  static String get deprecatedVersion => '1';
}
```

## 3. 数据传输对象模式

### 3.1 DTO设计模式

```dart
// 基础DTO接口
abstract class DataTransferObject {
  Map<String, dynamic> toJson();
}

// 请求DTO基类
abstract class RequestDto extends DataTransferObject {
  void validate();
}

// 响应DTO基类
abstract class ResponseDto extends DataTransferObject {
  factory ResponseDto.fromJson(Map<String, dynamic> json);
}

// 用户创建请求DTO
class CreateUserRequest extends RequestDto {
  final String username;
  final String email;
  final String password;
  final String? firstName;
  final String? lastName;
  final UserRole role;
  final Map<String, dynamic>? metadata;
  
  CreateUserRequest({
    required this.username,
    required this.email,
    required this.password,
    this.firstName,
    this.lastName,
    this.role = UserRole.user,
    this.metadata,
  });
  
  @override
  void validate() {
    if (username.isEmpty) {
      throw ValidationException('Username cannot be empty');
    }
    
    if (!_isValidEmail(email)) {
      throw ValidationException('Invalid email format');
    }
    
    if (password.length < 8) {
      throw ValidationException('Password must be at least 8 characters');
    }
    
    if (password.length > 128) {
      throw ValidationException('Password cannot exceed 128 characters');
    }
    
    // 密码强度验证
    if (!_isStrongPassword(password)) {
      throw ValidationException(
        'Password must contain at least one uppercase letter, '
        'one lowercase letter, one digit, and one special character'
      );
    }
  }
  
  bool _isValidEmail(String email) {
    return RegExp(r'^[\w-\.]+@([\w-]+\.)+[\w-]{2,4}$').hasMatch(email);
  }
  
  bool _isStrongPassword(String password) {
    return RegExp(r'^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]').hasMatch(password);
  }
  
  @override
  Map<String, dynamic> toJson() {
    return {
      'username': username,
      'email': email,
      'password': password,
      if (firstName != null) 'first_name': firstName,
      if (lastName != null) 'last_name': lastName,
      'role': role.name,
      if (metadata != null) 'metadata': metadata,
    };
  }
}

// 用户更新请求DTO
class UpdateUserRequest extends RequestDto {
  final String? username;
  final String? email;
  final String? firstName;
  final String? lastName;
  final UserRole? role;
  final UserStatus? status;
  final Map<String, dynamic>? metadata;
  
  UpdateUserRequest({
    this.username,
    this.email,
    this.firstName,
    this.lastName,
    this.role,
    this.status,
    this.metadata,
  });
  
  @override
  void validate() {
    if (username != null && username!.isEmpty) {
      throw ValidationException('Username cannot be empty');
    }
    
    if (email != null && !_isValidEmail(email!)) {
      throw ValidationException('Invalid email format');
    }
  }
  
  bool _isValidEmail(String email) {
    return RegExp(r'^[\w-\.]+@([\w-]+\.)+[\w-]{2,4}$').hasMatch(email);
  }
  
  @override
  Map<String, dynamic> toJson() {
    final json = <String, dynamic>{};
    
    if (username != null) json['username'] = username;
    if (email != null) json['email'] = email;
    if (firstName != null) json['first_name'] = firstName;
    if (lastName != null) json['last_name'] = lastName;
    if (role != null) json['role'] = role!.name;
    if (status != null) json['status'] = status!.name;
    if (metadata != null) json['metadata'] = metadata;
    
    return json;
  }
}

// 用户响应DTO
class UserResponse extends ResponseDto {
  final int id;
  final String username;
  final String email;
  final String? firstName;
  final String? lastName;
  final String? fullName;
  final UserRole role;
  final UserStatus status;
  final String? avatarUrl;
  final DateTime createdAt;
  final DateTime updatedAt;
  final DateTime? lastLoginAt;
  final Map<String, dynamic> metadata;
  
  UserResponse({
    required this.id,
    required this.username,
    required this.email,
    this.firstName,
    this.lastName,
    required this.role,
    required this.status,
    this.avatarUrl,
    required this.createdAt,
    required this.updatedAt,
    this.lastLoginAt,
    this.metadata = const {},
  }) : fullName = _buildFullName(firstName, lastName);
  
  static String? _buildFullName(String? firstName, String? lastName) {
    if (firstName == null && lastName == null) return null;
    return [firstName, lastName].where((name) => name != null).join(' ');
  }
  
  factory UserResponse.fromJson(Map<String, dynamic> json) {
    return UserResponse(
      id: json['id'],
      username: json['username'],
      email: json['email'],
      firstName: json['first_name'],
      lastName: json['last_name'],
      role: UserRole.values.byName(json['role']),
      status: UserStatus.values.byName(json['status']),
      avatarUrl: json['avatar_url'],
      createdAt: DateTime.parse(json['created_at']),
      updatedAt: DateTime.parse(json['updated_at']),
      lastLoginAt: json['last_login_at'] != null 
          ? DateTime.parse(json['last_login_at']) 
          : null,
      metadata: Map<String, dynamic>.from(json['metadata'] ?? {}),
    );
  }
  
  @override
  Map<String, dynamic> toJson() {
    return {
      'id': id,
      'username': username,
      'email': email,
      'first_name': firstName,
      'last_name': lastName,
      'full_name': fullName,
      'role': role.name,
      'status': status.name,
      'avatar_url': avatarUrl,
      'created_at': createdAt.toIso8601String(),
      'updated_at': updatedAt.toIso8601String(),
      'last_login_at': lastLoginAt?.toIso8601String(),
      'metadata': metadata,
    };
  }
}

enum UserRole { admin, moderator, user, guest }
enum UserStatus { active, inactive, suspended, pending }

class ValidationException implements Exception {
  final String message;
  ValidationException(this.message);
  
  @override
  String toString() => 'ValidationException: $message';
}
```

### 3.2 DTO转换器

```dart
abstract class DtoConverter<TEntity, TDto> {
  TDto toDto(TEntity entity);
  TEntity toEntity(TDto dto);
  List<TDto> toDtoList(List<TEntity> entities);
  List<TEntity> toEntityList(List<TDto> dtos);
}

class UserDtoConverter extends DtoConverter<User, UserResponse> {
  @override
  UserResponse toDto(User entity) {
    return UserResponse(
      id: entity.id,
      username: entity.username,
      email: entity.email,
      firstName: entity.firstName,
      lastName: entity.lastName,
      role: entity.role,
      status: entity.status,
      avatarUrl: entity.avatarUrl,
      createdAt: entity.createdAt,
      updatedAt: entity.updatedAt,
      lastLoginAt: entity.lastLoginAt,
      metadata: entity.metadata,
    );
  }
  
  @override
  User toEntity(UserResponse dto) {
    return User(
      id: dto.id,
      username: dto.username,
      email: dto.email,
      firstName: dto.firstName,
      lastName: dto.lastName,
      role: dto.role,
      status: dto.status,
      avatarUrl: dto.avatarUrl,
      createdAt: dto.createdAt,
      updatedAt: dto.updatedAt,
      lastLoginAt: dto.lastLoginAt,
      metadata: dto.metadata,
    );
  }
  
  @override
  List<UserResponse> toDtoList(List<User> entities) {
    return entities.map(toDto).toList();
  }
  
  @override
  List<User> toEntityList(List<UserResponse> dtos) {
    return dtos.map(toEntity).toList();
  }
  
  // 特殊转换方法
  User fromCreateRequest(CreateUserRequest request) {
    return User(
      id: 0, // 将由服务器分配
      username: request.username,
      email: request.email,
      firstName: request.firstName,
      lastName: request.lastName,
      role: request.role,
      status: UserStatus.pending,
      createdAt: DateTime.now(),
      updatedAt: DateTime.now(),
      metadata: request.metadata ?? {},
    );
  }
  
  User applyUpdateRequest(User user, UpdateUserRequest request) {
    return user.copyWith(
      username: request.username ?? user.username,
      email: request.email ?? user.email,
      firstName: request.firstName ?? user.firstName,
      lastName: request.lastName ?? user.lastName,
      role: request.role ?? user.role,
      status: request.status ?? user.status,
      updatedAt: DateTime.now(),
      metadata: request.metadata ?? user.metadata,
    );
  }
}

// DTO转换器注册表
class DtoConverterRegistry {
  final Map<Type, DtoConverter> _converters = {};
  
  void register<TEntity, TDto>(DtoConverter<TEntity, TDto> converter) {
    _converters[TEntity] = converter;
  }
  
  DtoConverter<TEntity, TDto> get<TEntity, TDto>() {
    final converter = _converters[TEntity];
    if (converter == null) {
      throw Exception('No converter registered for type $TEntity');
    }
    return converter as DtoConverter<TEntity, TDto>;
  }
  
  bool isRegistered<TEntity>() {
    return _converters.containsKey(TEntity);
  }
}

// 使用示例
class DtoConverterExample {
  static void setupConverters() {
    final registry = DtoConverterRegistry();
    registry.register<User, UserResponse>(UserDtoConverter());
    
    // 使用转换器
    final converter = registry.get<User, UserResponse>();
    final user = User(/* ... */);
    final userDto = converter.toDto(user);
  }
}
```

## 4. API客户端设计模式

### 4.1 Repository模式

```dart
abstract class Repository<T, ID> {
  Future<T?> findById(ID id);
  Future<List<T>> findAll();
  Future<T> save(T entity);
  Future<T> update(ID id, T entity);
  Future<void> delete(ID id);
  Future<bool> exists(ID id);
}

abstract class UserRepository extends Repository<User, int> {
  Future<User?> findByUsername(String username);
  Future<User?> findByEmail(String email);
  Future<List<User>> findByRole(UserRole role);
  Future<List<User>> findByStatus(UserStatus status);
  Future<PaginatedResponse<User>> findWithPagination({
    int page = 1,
    int limit = 20,
    String? search,
    UserStatus? status,
    UserRole? role,
  });
}

class ApiUserRepository implements UserRepository {
  final UserApi userApi;
  final DtoConverter<User, UserResponse> converter;
  final CacheManager? cacheManager;
  
  ApiUserRepository({
    required this.userApi,
    required this.converter,
    this.cacheManager,
  });
  
  @override
  Future<User?> findById(int id) async {
    try {
      // 先检查缓存
      if (cacheManager != null) {
        final cached = await cacheManager!.get<User>('user_$id');
        if (cached != null) {
          return cached;
        }
      }
      
      final userDto = await userApi.getUser(id);
      final user = converter.toEntity(userDto);
      
      // 缓存结果
      if (cacheManager != null) {
        await cacheManager!.set('user_$id', user, duration: Duration(minutes: 15));
      }
      
      return user;
    } on DioException catch (e) {
      if (e.response?.statusCode == 404) {
        return null;
      }
      rethrow;
    }
  }
  
  @override
  Future<List<User>> findAll() async {
    final response = await userApi.getUsers();
    return converter.toEntityList(response.data);
  }
  
  @override
  Future<User> save(User entity) async {
    final request = CreateUserRequest(
      username: entity.username,
      email: entity.email,
      password: 'temp_password', // 实际应用中需要处理密码
      firstName: entity.firstName,
      lastName: entity.lastName,
      role: entity.role,
      metadata: entity.metadata,
    );
    
    final userDto = await userApi.createUser(request);
    final user = converter.toEntity(userDto);
    
    // 清除相关缓存
    if (cacheManager != null) {
      await cacheManager!.delete('users_list');
    }
    
    return user;
  }
  
  @override
  Future<User> update(int id, User entity) async {
    final request = UpdateUserRequest(
      username: entity.username,
      email: entity.email,
      firstName: entity.firstName,
      lastName: entity.lastName,
      role: entity.role,
      status: entity.status,
      metadata: entity.metadata,
    );
    
    final userDto = await userApi.updateUser(id, request);
    final user = converter.toEntity(userDto);
    
    // 更新缓存
    if (cacheManager != null) {
      await cacheManager!.set('user_$id', user, duration: Duration(minutes: 15));
      await cacheManager!.delete('users_list');
    }
    
    return user;
  }
  
  @override
  Future<void> delete(int id) async {
    await userApi.deleteUser(id);
    
    // 清除缓存
    if (cacheManager != null) {
      await cacheManager!.delete('user_$id');
      await cacheManager!.delete('users_list');
    }
  }
  
  @override
  Future<bool> exists(int id) async {
    try {
      await findById(id);
      return true;
    } catch (e) {
      return false;
    }
  }
  
  @override
  Future<User?> findByUsername(String username) async {
    final response = await userApi.getUsers(search: username);
    final users = converter.toEntityList(response.data);
    return users.firstWhere(
      (user) => user.username == username,
      orElse: () => throw StateError('User not found'),
    );
  }
  
  @override
  Future<User?> findByEmail(String email) async {
    final response = await userApi.getUsers(search: email);
    final users = converter.toEntityList(response.data);
    return users.firstWhere(
      (user) => user.email == email,
      orElse: () => throw StateError('User not found'),
    );
  }
  
  @override
  Future<List<User>> findByRole(UserRole role) async {
    final response = await userApi.getUsers();
    final users = converter.toEntityList(response.data);
    return users.where((user) => user.role == role).toList();
  }
  
  @override
  Future<List<User>> findByStatus(UserStatus status) async {
    final response = await userApi.getUsers(status: status);
    return converter.toEntityList(response.data);
  }
  
  @override
  Future<PaginatedResponse<User>> findWithPagination({
    int page = 1,
    int limit = 20,
    String? search,
    UserStatus? status,
    UserRole? role,
  }) async {
    final response = await userApi.getUsers(
      page: page,
      limit: limit,
      search: search,
      status: status,
    );
    
    return PaginatedResponse(
      data: converter.toEntityList(response.data),
      meta: response.meta,
      links: response.links,
    );
  }
}
```

### 4.2 Service层模式

```dart
abstract class UserService {
  Future<User> createUser(CreateUserRequest request);
  Future<User> getUserById(int id);
  Future<User> updateUser(int id, UpdateUserRequest request);
  Future<void> deleteUser(int id);
  Future<List<User>> searchUsers(String query);
  Future<bool> changePassword(int id, String oldPassword, String newPassword);
  Future<void> resetPassword(String email);
  Future<User> uploadAvatar(int id, File avatarFile);
}

class UserServiceImpl implements UserService {
  final UserRepository userRepository;
  final PasswordService passwordService;
  final EmailService emailService;
  final FileUploadService fileUploadService;
  final EventBus eventBus;
  
  UserServiceImpl({
    required this.userRepository,
    required this.passwordService,
    required this.emailService,
    required this.fileUploadService,
    required this.eventBus,
  });
  
  @override
  Future<User> createUser(CreateUserRequest request) async {
    // 验证请求
    request.validate();
    
    // 检查用户名是否已存在
    final existingUser = await userRepository.findByUsername(request.username);
    if (existingUser != null) {
      throw ConflictException('Username already exists');
    }
    
    // 检查邮箱是否已存在
    final existingEmail = await userRepository.findByEmail(request.email);
    if (existingEmail != null) {
      throw ConflictException('Email already exists');
    }
    
    // 加密密码
    final hashedPassword = await passwordService.hashPassword(request.password);
    
    // 创建用户实体
    final user = User(
      id: 0,
      username: request.username,
      email: request.email,
      firstName: request.firstName,
      lastName: request.lastName,
      role: request.role,
      status: UserStatus.pending,
      passwordHash: hashedPassword,
      createdAt: DateTime.now(),
      updatedAt: DateTime.now(),
      metadata: request.metadata ?? {},
    );
    
    // 保存用户
    final savedUser = await userRepository.save(user);
    
    // 发送欢迎邮件
    await emailService.sendWelcomeEmail(savedUser.email, savedUser.username);
    
    // 发布用户创建事件
    eventBus.fire(UserCreatedEvent(savedUser));
    
    return savedUser;
  }
  
  @override
  Future<User> getUserById(int id) async {
    final user = await userRepository.findById(id);
    if (user == null) {
      throw NotFoundException('User not found');
    }
    return user;
  }
  
  @override
  Future<User> updateUser(int id, UpdateUserRequest request) async {
    // 验证请求
    request.validate();
    
    // 获取现有用户
    final existingUser = await getUserById(id);
    
    // 检查用户名冲突
    if (request.username != null && request.username != existingUser.username) {
      final conflictUser = await userRepository.findByUsername(request.username!);
      if (conflictUser != null && conflictUser.id != id) {
        throw ConflictException('Username already exists');
      }
    }
    
    // 检查邮箱冲突
    if (request.email != null && request.email != existingUser.email) {
      final conflictUser = await userRepository.findByEmail(request.email!);
      if (conflictUser != null && conflictUser.id != id) {
        throw ConflictException('Email already exists');
      }
    }
    
    // 更新用户
    final updatedUser = existingUser.copyWith(
      username: request.username ?? existingUser.username,
      email: request.email ?? existingUser.email,
      firstName: request.firstName ?? existingUser.firstName,
      lastName: request.lastName ?? existingUser.lastName,
      role: request.role ?? existingUser.role,
      status: request.status ?? existingUser.status,
      updatedAt: DateTime.now(),
      metadata: request.metadata ?? existingUser.metadata,
    );
    
    final savedUser = await userRepository.update(id, updatedUser);
    
    // 发布用户更新事件
    eventBus.fire(UserUpdatedEvent(savedUser, existingUser));
    
    return savedUser;
  }
  
  @override
  Future<void> deleteUser(int id) async {
    final user = await getUserById(id);
    
    // 软删除：更新状态而不是物理删除
    await userRepository.update(id, user.copyWith(
      status: UserStatus.inactive,
      updatedAt: DateTime.now(),
    ));
    
    // 发布用户删除事件
    eventBus.fire(UserDeletedEvent(user));
  }
  
  @override
  Future<List<User>> searchUsers(String query) async {
    if (query.trim().isEmpty) {
      return [];
    }
    
    final response = await userRepository.findWithPagination(
      search: query,
      limit: 50,
    );
    
    return response.data;
  }
  
  @override
  Future<bool> changePassword(
    int id,
    String oldPassword,
    String newPassword,
  ) async {
    final user = await getUserById(id);
    
    // 验证旧密码
    final isOldPasswordValid = await passwordService.verifyPassword(
      oldPassword,
      user.passwordHash,
    );
    
    if (!isOldPasswordValid) {
      throw UnauthorizedException('Invalid old password');
    }
    
    // 验证新密码强度
    if (!passwordService.isStrongPassword(newPassword)) {
      throw ValidationException('New password does not meet requirements');
    }
    
    // 加密新密码
    final newPasswordHash = await passwordService.hashPassword(newPassword);
    
    // 更新密码
    await userRepository.update(id, user.copyWith(
      passwordHash: newPasswordHash,
      updatedAt: DateTime.now(),
    ));
    
    // 发送密码更改通知
    await emailService.sendPasswordChangedNotification(user.email);
    
    return true;
  }
  
  @override
  Future<void> resetPassword(String email) async {
    final user = await userRepository.findByEmail(email);
    if (user == null) {
      // 为了安全，不透露用户是否存在
      return;
    }
    
    // 生成重置令牌
    final resetToken = passwordService.generateResetToken();
    
    // 保存重置令牌（实际应用中应该有专门的表存储）
    await userRepository.update(user.id, user.copyWith(
      metadata: {
        ...user.metadata,
        'password_reset_token': resetToken,
        'password_reset_expires': DateTime.now()
            .add(Duration(hours: 1))
            .toIso8601String(),
      },
    ));
    
    // 发送重置邮件
    await emailService.sendPasswordResetEmail(email, resetToken);
  }
  
  @override
  Future<User> uploadAvatar(int id, File avatarFile) async {
    final user = await getUserById(id);
    
    // 上传文件
    final avatarUrl = await fileUploadService.uploadImage(
      avatarFile,
      path: 'avatars/$id',
    );
    
    // 更新用户头像
    final updatedUser = await userRepository.update(id, user.copyWith(
      avatarUrl: avatarUrl,
      updatedAt: DateTime.now(),
    ));
    
    return updatedUser;
  }
}

// 事件定义
abstract class UserEvent {
  final User user;
  UserEvent(this.user);
}

class UserCreatedEvent extends UserEvent {
  UserCreatedEvent(super.user);
}

class UserUpdatedEvent extends UserEvent {
  final User oldUser;
  UserUpdatedEvent(super.user, this.oldUser);
}

class UserDeletedEvent extends UserEvent {
  UserDeletedEvent(super.user);
}

// 异常定义
class ConflictException implements Exception {
  final String message;
  ConflictException(this.message);
}

class NotFoundException implements Exception {
  final String message;
  NotFoundException(this.message);
}

class UnauthorizedException implements Exception {
  final String message;
  UnauthorizedException(this.message);
}
```

## 📚 总结

### 核心设计原则
- **RESTful设计**: 资源导向、统一接口、无状态通信
- **版本管理**: 向后兼容、渐进升级、清晰文档
- **数据传输**: DTO模式、验证机制、类型安全
- **客户端架构**: Repository模式、Service层、事件驱动

### 最佳实践
- **API设计**: 一致性、可预测性、易用性
- **错误处理**: 标准化状态码、详细错误信息
- **性能优化**: 缓存策略、分页加载、批量操作
- **安全考虑**: 输入验证、权限控制、敏感数据保护

### 推荐工具
- **API文档**: OpenAPI/Swagger、Postman
- **代码生成**: json_annotation、retrofit
- **测试工具**: Mockito、HTTP Mock Server
- **监控工具**: Sentry、Firebase Analytics