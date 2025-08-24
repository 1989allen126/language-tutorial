# 物联网和边缘计算组件

本文档详细介绍 Flutter 中物联网 (IoT) 和边缘计算相关组件的使用，包括设备连接、数据采集、边缘计算、云端同步等功能。

## 1. 设备连接管理

### 1.1 基础配置

```yaml
# pubspec.yaml
dependencies:
  mqtt_client: ^10.0.0
  web_socket_channel: ^2.4.0
  http: ^1.1.0
  connectivity_plus: ^5.0.2
  network_info_plus: ^4.1.0
  device_info_plus: ^9.1.1
  permission_handler: ^11.1.0
  shared_preferences: ^2.2.2
  sqflite: ^2.3.0
  path_provider: ^2.1.1
```

### 1.2 MQTT 连接管理

```dart
// MQTT 连接管理器
import 'package:mqtt_client/mqtt_client.dart';
import 'package:mqtt_client/mqtt_server_client.dart';

class MQTTManager {
  static final MQTTManager _instance = MQTTManager._internal();
  factory MQTTManager() => _instance;
  MQTTManager._internal();
  
  MqttServerClient? _client;
  final StreamController<MQTTMessage> _messageController = StreamController.broadcast();
  final Map<String, List<Function(String)>> _subscriptions = {};
  
  bool get isConnected => _client?.connectionStatus?.state == MqttConnectionState.connected;
  Stream<MQTTMessage> get messageStream => _messageController.stream;
  
  // 连接到 MQTT 服务器
  Future<bool> connect({
    required String server,
    required int port,
    required String clientId,
    String? username,
    String? password,
    bool useSSL = false,
  }) async {
    try {
      _client = MqttServerClient.withPort(server, clientId, port);
      _client!.logging(on: true);
      _client!.onConnected = _onConnected;
      _client!.onDisconnected = _onDisconnected;
      _client!.onUnsubscribed = _onUnsubscribed;
      _client!.onSubscribed = _onSubscribed;
      _client!.onSubscribeFail = _onSubscribeFail;
      _client!.pongCallback = _onPong;
      
      if (useSSL) {
        _client!.secure = true;
      }
      
      final connMessage = MqttConnectMessage()
          .withClientIdentifier(clientId)
          .withWillTopic('will')
          .withWillMessage('Client disconnected')
          .startClean()
          .withWillQos(MqttQos.atLeastOnce);
      
      if (username != null && password != null) {
        connMessage.authenticateAs(username, password);
      }
      
      _client!.connectionMessage = connMessage;
      
      await _client!.connect();
      
      if (isConnected) {
        _client!.updates!.listen(_onMessage);
        print('MQTT 连接成功');
        return true;
      }
      
      return false;
    } catch (e) {
      print('MQTT 连接失败: $e');
      return false;
    }
  }
  
  // 订阅主题
  Future<void> subscribe(String topic, {MqttQos qos = MqttQos.atMostOnce}) async {
    if (!isConnected) return;
    
    _client!.subscribe(topic, qos);
    print('订阅主题: $topic');
  }
  
  // 取消订阅
  Future<void> unsubscribe(String topic) async {
    if (!isConnected) return;
    
    _client!.unsubscribe(topic);
    _subscriptions.remove(topic);
    print('取消订阅: $topic');
  }
  
  // 发布消息
  Future<void> publish(String topic, String message, {MqttQos qos = MqttQos.atMostOnce}) async {
    if (!isConnected) return;
    
    final builder = MqttClientPayloadBuilder();
    builder.addString(message);
    
    _client!.publishMessage(topic, qos, builder.payload!);
    print('发布消息到 $topic: $message');
  }
  
  // 发布 JSON 数据
  Future<void> publishJson(String topic, Map<String, dynamic> data, {MqttQos qos = MqttQos.atMostOnce}) async {
    final jsonString = json.encode(data);
    await publish(topic, jsonString, qos: qos);
  }
  
  // 添加主题监听器
  void addTopicListener(String topic, Function(String) callback) {
    if (!_subscriptions.containsKey(topic)) {
      _subscriptions[topic] = [];
      subscribe(topic);
    }
    _subscriptions[topic]!.add(callback);
  }
  
  // 移除主题监听器
  void removeTopicListener(String topic, Function(String) callback) {
    if (_subscriptions.containsKey(topic)) {
      _subscriptions[topic]!.remove(callback);
      if (_subscriptions[topic]!.isEmpty) {
        unsubscribe(topic);
      }
    }
  }
  
  // 断开连接
  Future<void> disconnect() async {
    if (_client != null) {
      _client!.disconnect();
      _client = null;
    }
    _subscriptions.clear();
  }
  
  // 连接成功回调
  void _onConnected() {
    print('MQTT 连接成功');
  }
  
  // 断开连接回调
  void _onDisconnected() {
    print('MQTT 连接断开');
  }
  
  // 订阅成功回调
  void _onSubscribed(String topic) {
    print('订阅成功: $topic');
  }
  
  // 取消订阅回调
  void _onUnsubscribed(String? topic) {
    print('取消订阅: $topic');
  }
  
  // 订阅失败回调
  void _onSubscribeFail(String topic) {
    print('订阅失败: $topic');
  }
  
  // Pong 回调
  void _onPong() {
    print('MQTT Pong 收到');
  }
  
  // 消息接收回调
  void _onMessage(List<MqttReceivedMessage<MqttMessage>> messages) {
    for (final message in messages) {
      final topic = message.topic;
      final payload = MqttPublishPayload.bytesToStringAsString(
        (message.payload as MqttPublishMessage).payload.message,
      );
      
      final mqttMessage = MQTTMessage(
        topic: topic,
        payload: payload,
        timestamp: DateTime.now(),
      );
      
      _messageController.add(mqttMessage);
      
      // 调用主题监听器
      if (_subscriptions.containsKey(topic)) {
        for (final callback in _subscriptions[topic]!) {
          callback(payload);
        }
      }
    }
  }
}

// MQTT 消息类
class MQTTMessage {
  final String topic;
  final String payload;
  final DateTime timestamp;
  
  MQTTMessage({
    required this.topic,
    required this.payload,
    required this.timestamp,
  });
  
  Map<String, dynamic> toJson() {
    return {
      'topic': topic,
      'payload': payload,
      'timestamp': timestamp.toIso8601String(),
    };
  }
  
  factory MQTTMessage.fromJson(Map<String, dynamic> json) {
    return MQTTMessage(
      topic: json['topic'],
      payload: json['payload'],
      timestamp: DateTime.parse(json['timestamp']),
    );
  }
}
```

### 1.3 WebSocket 连接管理

```dart
// WebSocket 连接管理器
import 'package:web_socket_channel/web_socket_channel.dart';
import 'package:web_socket_channel/status.dart' as status;

class WebSocketManager {
  static final WebSocketManager _instance = WebSocketManager._internal();
  factory WebSocketManager() => _instance;
  WebSocketManager._internal();
  
  WebSocketChannel? _channel;
  final StreamController<WebSocketMessage> _messageController = StreamController.broadcast();
  Timer? _heartbeatTimer;
  Timer? _reconnectTimer;
  
  String? _url;
  Map<String, String>? _headers;
  bool _autoReconnect = true;
  int _reconnectAttempts = 0;
  final int _maxReconnectAttempts = 5;
  
  bool get isConnected => _channel != null;
  Stream<WebSocketMessage> get messageStream => _messageController.stream;
  
  // 连接 WebSocket
  Future<bool> connect(String url, {Map<String, String>? headers, bool autoReconnect = true}) async {
    try {
      _url = url;
      _headers = headers;
      _autoReconnect = autoReconnect;
      
      _channel = WebSocketChannel.connect(
        Uri.parse(url),
        protocols: headers,
      );
      
      _channel!.stream.listen(
        _onMessage,
        onError: _onError,
        onDone: _onDone,
      );
      
      _startHeartbeat();
      _reconnectAttempts = 0;
      
      print('WebSocket 连接成功: $url');
      return true;
    } catch (e) {
      print('WebSocket 连接失败: $e');
      _scheduleReconnect();
      return false;
    }
  }
  
  // 发送消息
  void send(String message) {
    if (isConnected) {
      _channel!.sink.add(message);
    }
  }
  
  // 发送 JSON 数据
  void sendJson(Map<String, dynamic> data) {
    final jsonString = json.encode(data);
    send(jsonString);
  }
  
  // 发送二进制数据
  void sendBytes(List<int> bytes) {
    if (isConnected) {
      _channel!.sink.add(bytes);
    }
  }
  
  // 断开连接
  Future<void> disconnect() async {
    _autoReconnect = false;
    _stopHeartbeat();
    _stopReconnectTimer();
    
    if (_channel != null) {
      await _channel!.sink.close(status.goingAway);
      _channel = null;
    }
  }
  
  // 消息接收回调
  void _onMessage(dynamic message) {
    final wsMessage = WebSocketMessage(
      data: message,
      timestamp: DateTime.now(),
      type: message is String ? MessageType.text : MessageType.binary,
    );
    
    _messageController.add(wsMessage);
  }
  
  // 错误回调
  void _onError(error) {
    print('WebSocket 错误: $error');
    _scheduleReconnect();
  }
  
  // 连接关闭回调
  void _onDone() {
    print('WebSocket 连接关闭');
    _channel = null;
    _scheduleReconnect();
  }
  
  // 开始心跳
  void _startHeartbeat() {
    _heartbeatTimer = Timer.periodic(const Duration(seconds: 30), (timer) {
      if (isConnected) {
        send(json.encode({'type': 'ping', 'timestamp': DateTime.now().millisecondsSinceEpoch}));
      }
    });
  }
  
  // 停止心跳
  void _stopHeartbeat() {
    _heartbeatTimer?.cancel();
    _heartbeatTimer = null;
  }
  
  // 安排重连
  void _scheduleReconnect() {
    if (!_autoReconnect || _reconnectAttempts >= _maxReconnectAttempts) {
      return;
    }
    
    _stopReconnectTimer();
    
    final delay = Duration(seconds: math.pow(2, _reconnectAttempts).toInt());
    _reconnectTimer = Timer(delay, () {
      _reconnectAttempts++;
      print('尝试重连 WebSocket (第 $_reconnectAttempts 次)');
      
      if (_url != null) {
        connect(_url!, headers: _headers, autoReconnect: _autoReconnect);
      }
    });
  }
  
  // 停止重连定时器
  void _stopReconnectTimer() {
    _reconnectTimer?.cancel();
    _reconnectTimer = null;
  }
}

// WebSocket 消息类
class WebSocketMessage {
  final dynamic data;
  final DateTime timestamp;
  final MessageType type;
  
  WebSocketMessage({
    required this.data,
    required this.timestamp,
    required this.type,
  });
}

enum MessageType { text, binary }
```

## 2. 设备数据采集

### 2.1 传感器数据管理

```dart
// 传感器数据管理器
import 'package:sensors_plus/sensors_plus.dart';
import 'package:geolocator/geolocator.dart';
import 'package:battery_plus/battery_plus.dart';
import 'package:device_info_plus/device_info_plus.dart';

class SensorDataManager {
  static final SensorDataManager _instance = SensorDataManager._internal();
  factory SensorDataManager() => _instance;
  SensorDataManager._internal();
  
  final StreamController<SensorData> _dataController = StreamController.broadcast();
  final List<StreamSubscription> _subscriptions = [];
  final Battery _battery = Battery();
  final DeviceInfoPlugin _deviceInfo = DeviceInfoPlugin();
  
  bool _isCollecting = false;
  Timer? _collectionTimer;
  
  Stream<SensorData> get dataStream => _dataController.stream;
  bool get isCollecting => _isCollecting;
  
  // 开始数据采集
  Future<void> startCollection({
    Duration interval = const Duration(seconds: 1),
    bool includeAccelerometer = true,
    bool includeGyroscope = true,
    bool includeMagnetometer = true,
    bool includeLocation = true,
    bool includeBattery = true,
  }) async {
    if (_isCollecting) return;
    
    _isCollecting = true;
    
    // 加速度计
    if (includeAccelerometer) {
      _subscriptions.add(
        accelerometerEvents.listen((event) {
          _addSensorData(SensorData(
            type: SensorType.accelerometer,
            values: [event.x, event.y, event.z],
            timestamp: DateTime.now(),
          ));
        }),
      );
    }
    
    // 陀螺仪
    if (includeGyroscope) {
      _subscriptions.add(
        gyroscopeEvents.listen((event) {
          _addSensorData(SensorData(
            type: SensorType.gyroscope,
            values: [event.x, event.y, event.z],
            timestamp: DateTime.now(),
          ));
        }),
      );
    }
    
    // 磁力计
    if (includeMagnetometer) {
      _subscriptions.add(
        magnetometerEvents.listen((event) {
          _addSensorData(SensorData(
            type: SensorType.magnetometer,
            values: [event.x, event.y, event.z],
            timestamp: DateTime.now(),
          ));
        }),
      );
    }
    
    // 定时采集其他数据
    _collectionTimer = Timer.periodic(interval, (timer) async {
      // 位置信息
      if (includeLocation) {
        try {
          final position = await Geolocator.getCurrentPosition();
          _addSensorData(SensorData(
            type: SensorType.location,
            values: [position.latitude, position.longitude, position.altitude],
            timestamp: DateTime.now(),
            metadata: {
              'accuracy': position.accuracy,
              'speed': position.speed,
              'heading': position.heading,
            },
          ));
        } catch (e) {
          print('获取位置信息失败: $e');
        }
      }
      
      // 电池信息
      if (includeBattery) {
        try {
          final batteryLevel = await _battery.batteryLevel;
          final batteryState = await _battery.batteryState;
          
          _addSensorData(SensorData(
            type: SensorType.battery,
            values: [batteryLevel.toDouble()],
            timestamp: DateTime.now(),
            metadata: {
              'state': batteryState.toString(),
            },
          ));
        } catch (e) {
          print('获取电池信息失败: $e');
        }
      }
    });
    
    print('传感器数据采集已开始');
  }
  
  // 停止数据采集
  void stopCollection() {
    if (!_isCollecting) return;
    
    _isCollecting = false;
    
    for (final subscription in _subscriptions) {
      subscription.cancel();
    }
    _subscriptions.clear();
    
    _collectionTimer?.cancel();
    _collectionTimer = null;
    
    print('传感器数据采集已停止');
  }
  
  // 添加传感器数据
  void _addSensorData(SensorData data) {
    _dataController.add(data);
  }
  
  // 获取设备信息
  Future<Map<String, dynamic>> getDeviceInfo() async {
    try {
      if (Platform.isAndroid) {
        final androidInfo = await _deviceInfo.androidInfo;
        return {
          'platform': 'Android',
          'model': androidInfo.model,
          'manufacturer': androidInfo.manufacturer,
          'version': androidInfo.version.release,
          'sdkInt': androidInfo.version.sdkInt,
        };
      } else if (Platform.isIOS) {
        final iosInfo = await _deviceInfo.iosInfo;
        return {
          'platform': 'iOS',
          'model': iosInfo.model,
          'name': iosInfo.name,
          'systemVersion': iosInfo.systemVersion,
        };
      }
    } catch (e) {
      print('获取设备信息失败: $e');
    }
    
    return {};
  }
  
  void dispose() {
    stopCollection();
    _dataController.close();
  }
}

// 传感器数据类
class SensorData {
  final SensorType type;
  final List<double> values;
  final DateTime timestamp;
  final Map<String, dynamic>? metadata;
  
  SensorData({
    required this.type,
    required this.values,
    required this.timestamp,
    this.metadata,
  });
  
  Map<String, dynamic> toJson() {
    return {
      'type': type.toString(),
      'values': values,
      'timestamp': timestamp.toIso8601String(),
      'metadata': metadata,
    };
  }
  
  factory SensorData.fromJson(Map<String, dynamic> json) {
    return SensorData(
      type: SensorType.values.firstWhere(
        (e) => e.toString() == json['type'],
        orElse: () => SensorType.unknown,
      ),
      values: List<double>.from(json['values']),
      timestamp: DateTime.parse(json['timestamp']),
      metadata: json['metadata'],
    );
  }
}

enum SensorType {
  accelerometer,
  gyroscope,
  magnetometer,
  location,
  battery,
  temperature,
  humidity,
  pressure,
  light,
  proximity,
  unknown,
}
```

### 2.2 数据缓存和同步

```dart
// 数据缓存管理器
class DataCacheManager {
  static final DataCacheManager _instance = DataCacheManager._internal();
  factory DataCacheManager() => _instance;
  DataCacheManager._internal();
  
  Database? _database;
  final Queue<SensorData> _dataQueue = Queue();
  Timer? _syncTimer;
  
  static const String _tableName = 'sensor_data';
  static const int _maxCacheSize = 1000;
  static const Duration _syncInterval = Duration(minutes: 5);
  
  // 初始化数据库
  Future<void> initialize() async {
    final documentsDirectory = await getApplicationDocumentsDirectory();
    final path = join(documentsDirectory.path, 'sensor_cache.db');
    
    _database = await openDatabase(
      path,
      version: 1,
      onCreate: (db, version) {
        return db.execute(
          'CREATE TABLE $_tableName('
          'id INTEGER PRIMARY KEY AUTOINCREMENT, '
          'type TEXT NOT NULL, '
          'values TEXT NOT NULL, '
          'timestamp TEXT NOT NULL, '
          'metadata TEXT, '
          'synced INTEGER DEFAULT 0'
          ')',
        );
      },
    );
    
    _startAutoSync();
  }
  
  // 缓存传感器数据
  Future<void> cacheSensorData(SensorData data) async {
    if (_database == null) return;
    
    try {
      await _database!.insert(
        _tableName,
        {
          'type': data.type.toString(),
          'values': json.encode(data.values),
          'timestamp': data.timestamp.toIso8601String(),
          'metadata': data.metadata != null ? json.encode(data.metadata) : null,
          'synced': 0,
        },
      );
      
      // 检查缓存大小
      await _checkCacheSize();
    } catch (e) {
      print('缓存数据失败: $e');
    }
  }
  
  // 批量缓存数据
  Future<void> batchCacheData(List<SensorData> dataList) async {
    if (_database == null) return;
    
    final batch = _database!.batch();
    
    for (final data in dataList) {
      batch.insert(
        _tableName,
        {
          'type': data.type.toString(),
          'values': json.encode(data.values),
          'timestamp': data.timestamp.toIso8601String(),
          'metadata': data.metadata != null ? json.encode(data.metadata) : null,
          'synced': 0,
        },
      );
    }
    
    try {
      await batch.commit();
      await _checkCacheSize();
    } catch (e) {
      print('批量缓存数据失败: $e');
    }
  }
  
  // 获取未同步的数据
  Future<List<SensorData>> getUnsyncedData({int limit = 100}) async {
    if (_database == null) return [];
    
    try {
      final List<Map<String, dynamic>> maps = await _database!.query(
        _tableName,
        where: 'synced = ?',
        whereArgs: [0],
        orderBy: 'timestamp ASC',
        limit: limit,
      );
      
      return maps.map((map) {
        return SensorData(
          type: SensorType.values.firstWhere(
            (e) => e.toString() == map['type'],
            orElse: () => SensorType.unknown,
          ),
          values: List<double>.from(json.decode(map['values'])),
          timestamp: DateTime.parse(map['timestamp']),
          metadata: map['metadata'] != null ? json.decode(map['metadata']) : null,
        );
      }).toList();
    } catch (e) {
      print('获取未同步数据失败: $e');
      return [];
    }
  }
  
  // 标记数据为已同步
  Future<void> markAsSynced(List<SensorData> dataList) async {
    if (_database == null) return;
    
    final batch = _database!.batch();
    
    for (final data in dataList) {
      batch.update(
        _tableName,
        {'synced': 1},
        where: 'timestamp = ? AND type = ?',
        whereArgs: [data.timestamp.toIso8601String(), data.type.toString()],
      );
    }
    
    try {
      await batch.commit();
    } catch (e) {
      print('标记同步状态失败: $e');
    }
  }
  
  // 检查缓存大小
  Future<void> _checkCacheSize() async {
    if (_database == null) return;
    
    try {
      final count = Sqflite.firstIntValue(
        await _database!.rawQuery('SELECT COUNT(*) FROM $_tableName'),
      ) ?? 0;
      
      if (count > _maxCacheSize) {
        // 删除最旧的已同步数据
        await _database!.delete(
          _tableName,
          where: 'synced = 1',
          orderBy: 'timestamp ASC',
          limit: count - _maxCacheSize,
        );
      }
    } catch (e) {
      print('检查缓存大小失败: $e');
    }
  }
  
  // 开始自动同步
  void _startAutoSync() {
    _syncTimer = Timer.periodic(_syncInterval, (timer) {
      _syncData();
    });
  }
  
  // 同步数据到云端
  Future<void> _syncData() async {
    try {
      final unsyncedData = await getUnsyncedData();
      if (unsyncedData.isEmpty) return;
      
      // 检查网络连接
      final connectivityResult = await Connectivity().checkConnectivity();
      if (connectivityResult == ConnectivityResult.none) {
        print('无网络连接，跳过同步');
        return;
      }
      
      // 发送数据到云端
      final success = await _uploadToCloud(unsyncedData);
      if (success) {
        await markAsSynced(unsyncedData);
        print('数据同步成功: ${unsyncedData.length} 条');
      }
    } catch (e) {
      print('数据同步失败: $e');
    }
  }
  
  // 上传数据到云端
  Future<bool> _uploadToCloud(List<SensorData> dataList) async {
    try {
      final response = await http.post(
        Uri.parse('https://your-iot-api.com/sensor-data'),
        headers: {
          'Content-Type': 'application/json',
          'Authorization': 'Bearer your-token',
        },
        body: json.encode({
          'data': dataList.map((data) => data.toJson()).toList(),
          'device_id': await _getDeviceId(),
          'timestamp': DateTime.now().toIso8601String(),
        }),
      );
      
      return response.statusCode == 200;
    } catch (e) {
      print('上传数据失败: $e');
      return false;
    }
  }
  
  // 获取设备 ID
  Future<String> _getDeviceId() async {
    final prefs = await SharedPreferences.getInstance();
    String? deviceId = prefs.getString('device_id');
    
    if (deviceId == null) {
      deviceId = const Uuid().v4();
      await prefs.setString('device_id', deviceId);
    }
    
    return deviceId;
  }
  
  // 手动同步
  Future<bool> manualSync() async {
    try {
      await _syncData();
      return true;
    } catch (e) {
      print('手动同步失败: $e');
      return false;
    }
  }
  
  // 清除所有缓存
  Future<void> clearCache() async {
    if (_database == null) return;
    
    try {
      await _database!.delete(_tableName);
      print('缓存已清除');
    } catch (e) {
      print('清除缓存失败: $e');
    }
  }
  
  void dispose() {
    _syncTimer?.cancel();
    _database?.close();
  }
}
```

## 3. 边缘计算处理

### 3.1 本地数据处理

```dart
// 边缘计算处理器
class EdgeComputingProcessor {
  static final EdgeComputingProcessor _instance = EdgeComputingProcessor._internal();
  factory EdgeComputingProcessor() => _instance;
  EdgeComputingProcessor._internal();
  
  final Map<String, EdgeTask> _tasks = {};
  final StreamController<EdgeResult> _resultController = StreamController.broadcast();
  
  Stream<EdgeResult> get resultStream => _resultController.stream;
  
  // 注册边缘计算任务
  void registerTask(EdgeTask task) {
    _tasks[task.id] = task;
    print('注册边缘计算任务: ${task.name}');
  }
  
  // 移除任务
  void removeTask(String taskId) {
    _tasks.remove(taskId);
    print('移除边缘计算任务: $taskId');
  }
  
  // 处理传感器数据
  Future<void> processSensorData(SensorData data) async {
    for (final task in _tasks.values) {
      if (task.canProcess(data)) {
        try {
          final result = await task.process(data);
          if (result != null) {
            _resultController.add(result);
          }
        } catch (e) {
          print('任务处理失败 ${task.name}: $e');
        }
      }
    }
  }
  
  // 批量处理数据
  Future<void> batchProcessData(List<SensorData> dataList) async {
    for (final task in _tasks.values) {
      if (task.supportsBatchProcessing) {
        try {
          final results = await task.batchProcess(dataList);
          for (final result in results) {
            _resultController.add(result);
          }
        } catch (e) {
          print('批量处理失败 ${task.name}: $e');
        }
      } else {
        for (final data in dataList) {
          await processSensorData(data);
        }
      }
    }
  }
  
  // 获取任务状态
  Map<String, dynamic> getTaskStatus() {
    return {
      'total_tasks': _tasks.length,
      'tasks': _tasks.values.map((task) => {
        'id': task.id,
        'name': task.name,
        'type': task.type.toString(),
        'enabled': task.enabled,
        'processed_count': task.processedCount,
      }).toList(),
    };
  }
}

// 边缘计算任务基类
abstract class EdgeTask {
  final String id;
  final String name;
  final EdgeTaskType type;
  bool enabled;
  int processedCount = 0;
  
  EdgeTask({
    required this.id,
    required this.name,
    required this.type,
    this.enabled = true,
  });
  
  bool get supportsBatchProcessing => false;
  
  bool canProcess(SensorData data);
  Future<EdgeResult?> process(SensorData data);
  
  Future<List<EdgeResult>> batchProcess(List<SensorData> dataList) async {
    final results = <EdgeResult>[];
    for (final data in dataList) {
      final result = await process(data);
      if (result != null) {
        results.add(result);
      }
    }
    return results;
  }
}

// 异常检测任务
class AnomalyDetectionTask extends EdgeTask {
  final double threshold;
  final List<double> _history = [];
  final int _historySize;
  
  AnomalyDetectionTask({
    required String id,
    required String name,
    required this.threshold,
    this.historySize = 10,
  }) : _historySize = historySize, super(
    id: id,
    name: name,
    type: EdgeTaskType.anomalyDetection,
  );
  
  @override
  bool canProcess(SensorData data) {
    return data.type == SensorType.accelerometer ||
           data.type == SensorType.gyroscope ||
           data.type == SensorType.temperature;
  }
  
  @override
  Future<EdgeResult?> process(SensorData data) async {
    processedCount++;
    
    final magnitude = _calculateMagnitude(data.values);
    _history.add(magnitude);
    
    if (_history.length > _historySize) {
      _history.removeAt(0);
    }
    
    if (_history.length >= 3) {
      final average = _history.reduce((a, b) => a + b) / _history.length;
      final deviation = (magnitude - average).abs();
      
      if (deviation > threshold) {
        return EdgeResult(
          taskId: id,
          type: EdgeResultType.anomaly,
          data: {
            'sensor_type': data.type.toString(),
            'magnitude': magnitude,
            'average': average,
            'deviation': deviation,
            'threshold': threshold,
          },
          timestamp: DateTime.now(),
        );
      }
    }
    
    return null;
  }
  
  double _calculateMagnitude(List<double> values) {
    return math.sqrt(values.map((v) => v * v).reduce((a, b) => a + b));
  }
}

// 数据聚合任务
class DataAggregationTask extends EdgeTask {
  final Duration _windowSize;
  final List<SensorData> _buffer = [];
  Timer? _flushTimer;
  
  DataAggregationTask({
    required String id,
    required String name,
    Duration windowSize = const Duration(minutes: 1),
  }) : _windowSize = windowSize, super(
    id: id,
    name: name,
    type: EdgeTaskType.aggregation,
  ) {
    _startFlushTimer();
  }
  
  @override
  bool get supportsBatchProcessing => true;
  
  @override
  bool canProcess(SensorData data) => true;
  
  @override
  Future<EdgeResult?> process(SensorData data) async {
    processedCount++;
    _buffer.add(data);
    return null;
  }
  
  void _startFlushTimer() {
    _flushTimer = Timer.periodic(_windowSize, (timer) {
      _flushBuffer();
    });
  }
  
  void _flushBuffer() {
    if (_buffer.isEmpty) return;
    
    final groupedData = <SensorType, List<SensorData>>{};
    
    for (final data in _buffer) {
      groupedData.putIfAbsent(data.type, () => []).add(data);
    }
    
    for (final entry in groupedData.entries) {
      final sensorType = entry.key;
      final dataList = entry.value;
      
      final aggregatedResult = _aggregateData(sensorType, dataList);
      
      EdgeComputingProcessor()._resultController.add(EdgeResult(
        taskId: id,
        type: EdgeResultType.aggregation,
        data: aggregatedResult,
        timestamp: DateTime.now(),
      ));
    }
    
    _buffer.clear();
  }
  
  Map<String, dynamic> _aggregateData(SensorType sensorType, List<SensorData> dataList) {
    final allValues = dataList.expand((data) => data.values).toList();
    
    if (allValues.isEmpty) {
      return {
        'sensor_type': sensorType.toString(),
        'count': 0,
      };
    }
    
    allValues.sort();
    
    return {
      'sensor_type': sensorType.toString(),
      'count': dataList.length,
      'min': allValues.first,
      'max': allValues.last,
      'average': allValues.reduce((a, b) => a + b) / allValues.length,
      'median': allValues[allValues.length ~/ 2],
      'start_time': dataList.first.timestamp.toIso8601String(),
      'end_time': dataList.last.timestamp.toIso8601String(),
    };
  }
  
  void dispose() {
    _flushTimer?.cancel();
  }
}

// 边缘计算结果
class EdgeResult {
  final String taskId;
  final EdgeResultType type;
  final Map<String, dynamic> data;
  final DateTime timestamp;
  
  EdgeResult({
    required this.taskId,
    required this.type,
    required this.data,
    required this.timestamp,
  });
  
  Map<String, dynamic> toJson() {
    return {
      'task_id': taskId,
      'type': type.toString(),
      'data': data,
      'timestamp': timestamp.toIso8601String(),
    };
  }
}

enum EdgeTaskType {
  anomalyDetection,
  aggregation,
  filtering,
  transformation,
  prediction,
}

enum EdgeResultType {
  anomaly,
  aggregation,
  alert,
  prediction,
  insight,
}
```

## 4. IoT 应用示例

### 4.1 智能家居控制应用

```dart
// 智能家居控制应用
class SmartHomeApp extends StatefulWidget {
  @override
  _SmartHomeAppState createState() => _SmartHomeAppState();
}

class _SmartHomeAppState extends State<SmartHomeApp> {
  final MQTTManager _mqttManager = MQTTManager();
  final Map<String, SmartDevice> _devices = {};
  bool _isConnected = false;
  
  @override
  void initState() {
    super.initState();
    _initializeSmartHome();
  }
  
  Future<void> _initializeSmartHome() async {
    // 连接 MQTT 服务器
    final connected = await _mqttManager.connect(
      server: 'your-mqtt-broker.com',
      port: 1883,
      clientId: 'flutter_smart_home_${DateTime.now().millisecondsSinceEpoch}',
      username: 'your_username',
      password: 'your_password',
    );
    
    if (connected) {
      setState(() {
        _isConnected = true;
      });
      
      _setupDevices();
      _subscribeToDeviceUpdates();
    }
  }
  
  void _setupDevices() {
    _devices.addAll({
      'living_room_light': SmartDevice(
        id: 'living_room_light',
        name: '客厅灯',
        type: DeviceType.light,
        topic: 'home/living_room/light',
        icon: Icons.lightbulb,
      ),
      'bedroom_ac': SmartDevice(
        id: 'bedroom_ac',
        name: '卧室空调',
        type: DeviceType.airConditioner,
        topic: 'home/bedroom/ac',
        icon: Icons.ac_unit,
      ),
      'kitchen_fan': SmartDevice(
        id: 'kitchen_fan',
        name: '厨房风扇',
        type: DeviceType.fan,
        topic: 'home/kitchen/fan',
        icon: Icons.air,
      ),
      'door_lock': SmartDevice(
        id: 'door_lock',
        name: '门锁',
        type: DeviceType.lock,
        topic: 'home/entrance/lock',
        icon: Icons.lock,
      ),
    });
  }
  
  void _subscribeToDeviceUpdates() {
    for (final device in _devices.values) {
      _mqttManager.addTopicListener(
        '${device.topic}/status',
        (message) => _handleDeviceUpdate(device.id, message),
      );
    }
  }
  
  void _handleDeviceUpdate(String deviceId, String message) {
    try {
      final data = json.decode(message);
      final device = _devices[deviceId];
      
      if (device != null) {
        setState(() {
          device.isOn = data['state'] == 'on';
          device.value = data['value']?.toDouble() ?? 0.0;
          device.lastUpdate = DateTime.now();
        });
      }
    } catch (e) {
      print('解析设备状态失败: $e');
    }
  }
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('智能家居'),
        actions: [
          Icon(
            _isConnected ? Icons.cloud_done : Icons.cloud_off,
            color: _isConnected ? Colors.green : Colors.red,
          ),
          const SizedBox(width: 16),
        ],
      ),
      body: _isConnected ? _buildDeviceGrid() : _buildConnectionStatus(),
    );
  }
  
  Widget _buildConnectionStatus() {
    return const Center(
      child: Column(
        mainAxisAlignment: MainAxisAlignment.center,
        children: [
          CircularProgressIndicator(),
          SizedBox(height: 16),
          Text('正在连接智能家居系统...'),
        ],
      ),
    );
  }
  
  Widget _buildDeviceGrid() {
    return Padding(
      padding: const EdgeInsets.all(16),
      child: GridView.builder(
        gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
          crossAxisCount: 2,
          crossAxisSpacing: 16,
          mainAxisSpacing: 16,
          childAspectRatio: 1.2,
        ),
        itemCount: _devices.length,
        itemBuilder: (context, index) {
          final device = _devices.values.elementAt(index);
          return _buildDeviceCard(device);
        },
      ),
    );
  }
  
  Widget _buildDeviceCard(SmartDevice device) {
    return Card(
      elevation: 4,
      child: InkWell(
        onTap: () => _toggleDevice(device),
        child: Padding(
          padding: const EdgeInsets.all(16),
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              Icon(
                device.icon,
                size: 48,
                color: device.isOn ? Colors.blue : Colors.grey,
              ),
              const SizedBox(height: 8),
              Text(
                device.name,
                style: const TextStyle(
                  fontSize: 16,
                  fontWeight: FontWeight.bold,
                ),
                textAlign: TextAlign.center,
              ),
              const SizedBox(height: 4),
              Text(
                device.isOn ? '开启' : '关闭',
                style: TextStyle(
                  color: device.isOn ? Colors.green : Colors.red,
                  fontSize: 12,
                ),
              ),
              if (device.type == DeviceType.airConditioner && device.isOn) ..[
                const SizedBox(height: 8),
                Text(
                  '${device.value.toInt()}°C',
                  style: const TextStyle(
                    fontSize: 14,
                    fontWeight: FontWeight.bold,
                  ),
                ),
              ],
            ],
          ),
        ),
      ),
    );
  }
  
  void _toggleDevice(SmartDevice device) {
    final newState = !device.isOn;
    
    _mqttManager.publishJson(
      '${device.topic}/command',
      {
        'action': newState ? 'turn_on' : 'turn_off',
        'timestamp': DateTime.now().toIso8601String(),
      },
    );
    
    // 乐观更新 UI
    setState(() {
      device.isOn = newState;
    });
  }
  
  @override
  void dispose() {
    _mqttManager.disconnect();
    super.dispose();
  }
}

// 智能设备类
class SmartDevice {
  final String id;
  final String name;
  final DeviceType type;
  final String topic;
  final IconData icon;
  
  bool isOn;
  double value;
  DateTime? lastUpdate;
  
  SmartDevice({
    required this.id,
    required this.name,
    required this.type,
    required this.topic,
    required this.icon,
    this.isOn = false,
    this.value = 0.0,
  });
}

enum DeviceType {
  light,
  airConditioner,
  fan,
  lock,
  sensor,
  camera,
}
```

### 4.2 环境监测应用

```dart
// 环境监测应用
class EnvironmentMonitorApp extends StatefulWidget {
  @override
  _EnvironmentMonitorAppState createState() => _EnvironmentMonitorAppState();
}

class _EnvironmentMonitorAppState extends State<EnvironmentMonitorApp> {
  final SensorDataManager _sensorManager = SensorDataManager();
  final DataCacheManager _cacheManager = DataCacheManager();
  final EdgeComputingProcessor _edgeProcessor = EdgeComputingProcessor();
  
  final Map<SensorType, double> _currentValues = {};
  final Map<SensorType, List<double>> _history = {};
  bool _isMonitoring = false;
  
  @override
  void initState() {
    super.initState();
    _initializeMonitoring();
  }
  
  Future<void> _initializeMonitoring() async {
    await _cacheManager.initialize();
    
    // 注册边缘计算任务
    _edgeProcessor.registerTask(
      AnomalyDetectionTask(
        id: 'temp_anomaly',
        name: '温度异常检测',
        threshold: 5.0,
      ),
    );
    
    _edgeProcessor.registerTask(
      DataAggregationTask(
        id: 'data_aggregation',
        name: '数据聚合',
        windowSize: const Duration(minutes: 5),
      ),
    );
    
    // 监听传感器数据
    _sensorManager.dataStream.listen((data) {
      _updateCurrentValue(data);
      _cacheManager.cacheSensorData(data);
      _edgeProcessor.processSensorData(data);
    });
    
    // 监听边缘计算结果
    _edgeProcessor.resultStream.listen((result) {
      _handleEdgeResult(result);
    });
  }
  
  void _updateCurrentValue(SensorData data) {
    setState(() {
      if (data.values.isNotEmpty) {
        _currentValues[data.type] = data.values.first;
        
        // 更新历史数据
        _history.putIfAbsent(data.type, () => []);
        _history[data.type]!.add(data.values.first);
        
        // 保持历史数据在合理范围内
        if (_history[data.type]!.length > 50) {
          _history[data.type]!.removeAt(0);
        }
      }
    });
  }
  
  void _handleEdgeResult(EdgeResult result) {
    if (result.type == EdgeResultType.anomaly) {
      _showAnomalyAlert(result);
    }
  }
  
  void _showAnomalyAlert(EdgeResult result) {
    showDialog(
      context: context,
      builder: (context) => AlertDialog(
        title: const Text('异常检测'),
        content: Text(
          '检测到 ${result.data['sensor_type']} 异常\n'
          '当前值: ${result.data['magnitude']?.toStringAsFixed(2)}\n'
          '平均值: ${result.data['average']?.toStringAsFixed(2)}\n'
          '偏差: ${result.data['deviation']?.toStringAsFixed(2)}',
        ),
        actions: [
          TextButton(
            onPressed: () => Navigator.pop(context),
            child: const Text('确定'),
          ),
        ],
      ),
    );
  }
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('环境监测'),
        actions: [
          IconButton(
            icon: Icon(_isMonitoring ? Icons.stop : Icons.play_arrow),
            onPressed: _toggleMonitoring,
          ),
        ],
      ),
      body: Column(
        children: [
          _buildStatusCard(),
          Expanded(
            child: _buildSensorGrid(),
          ),
        ],
      ),
    );
  }
  
  Widget _buildStatusCard() {
    return Card(
      margin: const EdgeInsets.all(16),
      child: Padding(
        padding: const EdgeInsets.all(16),
        child: Row(
          children: [
            Icon(
              _isMonitoring ? Icons.sensors : Icons.sensors_off,
              color: _isMonitoring ? Colors.green : Colors.grey,
              size: 32,
            ),
            const SizedBox(width: 16),
            Expanded(
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  Text(
                    _isMonitoring ? '监测中' : '已停止',
                    style: const TextStyle(
                      fontSize: 18,
                      fontWeight: FontWeight.bold,
                    ),
                  ),
                  Text(
                    '传感器数量: ${_currentValues.length}',
                    style: const TextStyle(color: Colors.grey),
                  ),
                ],
              ),
            ),
            ElevatedButton(
              onPressed: _cacheManager.manualSync,
              child: const Text('同步数据'),
            ),
          ],
        ),
      ),
    );
  }
  
  Widget _buildSensorGrid() {
    final sensorTypes = [
      SensorType.temperature,
      SensorType.humidity,
      SensorType.pressure,
      SensorType.light,
    ];
    
    return GridView.builder(
      padding: const EdgeInsets.all(16),
      gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
        crossAxisCount: 2,
        crossAxisSpacing: 16,
        mainAxisSpacing: 16,
        childAspectRatio: 1.0,
      ),
      itemCount: sensorTypes.length,
      itemBuilder: (context, index) {
        final sensorType = sensorTypes[index];
        return _buildSensorCard(sensorType);
      },
    );
  }
  
  Widget _buildSensorCard(SensorType sensorType) {
    final currentValue = _currentValues[sensorType] ?? 0.0;
    final history = _history[sensorType] ?? [];
    
    return Card(
      child: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Icon(
              _getSensorIcon(sensorType),
              size: 32,
              color: Colors.blue,
            ),
            const SizedBox(height: 8),
            Text(
              _getSensorName(sensorType),
              style: const TextStyle(
                fontSize: 14,
                fontWeight: FontWeight.bold,
              ),
            ),
            const SizedBox(height: 8),
            Text(
              '${currentValue.toStringAsFixed(1)} ${_getSensorUnit(sensorType)}',
              style: const TextStyle(
                fontSize: 18,
                fontWeight: FontWeight.bold,
                color: Colors.green,
              ),
            ),
            const SizedBox(height: 8),
            if (history.isNotEmpty)
              SizedBox(
                height: 40,
                child: _buildMiniChart(history),
              ),
          ],
        ),
      ),
    );
  }
  
  Widget _buildMiniChart(List<double> data) {
    if (data.length < 2) {
      return const SizedBox();
    }
    
    final minValue = data.reduce(math.min);
    final maxValue = data.reduce(math.max);
    final range = maxValue - minValue;
    
    if (range == 0) {
      return Container(
        decoration: BoxDecoration(
          border: Border.all(color: Colors.grey),
          borderRadius: BorderRadius.circular(4),
        ),
        child: const Center(
          child: Text('稳定', style: TextStyle(fontSize: 10)),
        ),
      );
    }
    
    return Container(
      decoration: BoxDecoration(
        border: Border.all(color: Colors.grey),
        borderRadius: BorderRadius.circular(4),
      ),
      child: CustomPaint(
        painter: MiniChartPainter(data, minValue, maxValue),
        size: const Size.fromHeight(40),
      ),
    );
  }
  
  IconData _getSensorIcon(SensorType type) {
    switch (type) {
      case SensorType.temperature:
        return Icons.thermostat;
      case SensorType.humidity:
        return Icons.water_drop;
      case SensorType.pressure:
        return Icons.compress;
      case SensorType.light:
        return Icons.wb_sunny;
      default:
        return Icons.sensors;
    }
  }
  
  String _getSensorName(SensorType type) {
    switch (type) {
      case SensorType.temperature:
        return '温度';
      case SensorType.humidity:
        return '湿度';
      case SensorType.pressure:
        return '气压';
      case SensorType.light:
        return '光照';
      default:
        return '传感器';
    }
  }
  
  String _getSensorUnit(SensorType type) {
    switch (type) {
      case SensorType.temperature:
        return '°C';
      case SensorType.humidity:
        return '%';
      case SensorType.pressure:
        return 'hPa';
      case SensorType.light:
        return 'lux';
      default:
        return '';
    }
  }
  
  void _toggleMonitoring() {
    if (_isMonitoring) {
      _sensorManager.stopCollection();
    } else {
      _sensorManager.startCollection(
        interval: const Duration(seconds: 2),
        includeAccelerometer: false,
        includeGyroscope: false,
        includeMagnetometer: false,
        includeLocation: false,
        includeBattery: true,
      );
    }
    
    setState(() {
      _isMonitoring = !_isMonitoring;
    });
  }
  
  @override
  void dispose() {
    _sensorManager.dispose();
    _cacheManager.dispose();
    super.dispose();
  }
}

// 迷你图表绘制器
class MiniChartPainter extends CustomPainter {
  final List<double> data;
  final double minValue;
  final double maxValue;
  
  MiniChartPainter(this.data, this.minValue, this.maxValue);
  
  @override
  void paint(Canvas canvas, Size size) {
    final paint = Paint()
      ..color = Colors.blue
      ..strokeWidth = 1.5
      ..style = PaintingStyle.stroke;
    
    final path = Path();
    final range = maxValue - minValue;
    
    for (int i = 0; i < data.length; i++) {
      final x = (i / (data.length - 1)) * size.width;
      final y = size.height - ((data[i] - minValue) / range) * size.height;
      
      if (i == 0) {
        path.moveTo(x, y);
      } else {
        path.lineTo(x, y);
      }
    }
    
    canvas.drawPath(path, paint);
  }
  
  @override
  bool shouldRepaint(covariant CustomPainter oldDelegate) => true;
}
```

## 5. 平台配置

### 5.1 Android 配置

```xml
<!-- android/app/src/main/AndroidManifest.xml -->
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
<uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
<uses-permission android:name="android.permission.CHANGE_WIFI_STATE" />
<uses-permission android:name="android.permission.BLUETOOTH" />
<uses-permission android:name="android.permission.BLUETOOTH_ADMIN" />
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
<uses-permission android:name="android.permission.WAKE_LOCK" />
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
```

### 5.2 iOS 配置

```xml
<!-- ios/Runner/Info.plist -->
<key>NSLocationWhenInUseUsageDescription</key>
<string>此应用需要位置权限来进行环境监测</string>

<key>NSLocationAlwaysAndWhenInUseUsageDescription</key>
<string>此应用需要位置权限来进行环境监测</string>

<key>NSBluetoothAlwaysUsageDescription</key>
<string>此应用需要蓝牙权限来连接IoT设备</string>

<key>NSBluetoothPeripheralUsageDescription</key>
<string>此应用需要蓝牙权限来连接IoT设备</string>
```

## 6. 最佳实践

### 6.1 连接管理

```dart
// 连接管理最佳实践
class ConnectionManager {
  static const Duration _reconnectDelay = Duration(seconds: 5);
  static const int _maxRetries = 3;
  
  // 网络状态检查
  static Future<bool> isNetworkAvailable() async {
    final connectivityResult = await Connectivity().checkConnectivity();
    return connectivityResult != ConnectivityResult.none;
  }
  
  // 带重试的连接
  static Future<bool> connectWithRetry(
    Future<bool> Function() connectFunction,
    {int maxRetries = _maxRetries}
  ) async {
    for (int i = 0; i < maxRetries; i++) {
      try {
        if (await connectFunction()) {
          return true;
        }
      } catch (e) {
        print('连接尝试 ${i + 1} 失败: $e');
      }
      
      if (i < maxRetries - 1) {
        await Future.delayed(_reconnectDelay);
      }
    }
    
    return false;
  }
  
  // 连接健康检查
  static void startHealthCheck(
    bool Function() isConnected,
    Future<void> Function() reconnect,
  ) {
    Timer.periodic(const Duration(minutes: 1), (timer) async {
      if (!isConnected()) {
        print('连接丢失，尝试重连...');
        await reconnect();
      }
    });
  }
}
```

### 6.2 数据处理优化

```dart
// 数据处理优化
class DataProcessingOptimizer {
  // 数据压缩
  static List<int> compressData(String data) {
    return gzip.encode(utf8.encode(data));
  }
  
  // 数据解压
  static String decompressData(List<int> compressedData) {
    return utf8.decode(gzip.decode(compressedData));
  }
  
  // 批量处理
  static Future<void> batchProcess<T>(
    List<T> items,
    Future<void> Function(T) processor,
    {int batchSize = 10}
  ) async {
    for (int i = 0; i < items.length; i += batchSize) {
      final batch = items.skip(i).take(batchSize).toList();
      await Future.wait(batch.map(processor));
    }
  }
  
  // 数据去重
  static List<T> removeDuplicates<T>(
    List<T> items,
    bool Function(T, T) equals,
  ) {
    final result = <T>[];
    for (final item in items) {
      if (!result.any((existing) => equals(existing, item))) {
        result.add(item);
      }
    }
    return result;
  }
}
```

### 6.3 错误处理

```dart
// IoT 错误处理
class IoTErrorHandler {
  static void handleConnectionError(dynamic error) {
    if (error is SocketException) {
      print('网络连接错误: ${error.message}');
      // 显示网络错误提示
    } else if (error is TimeoutException) {
      print('连接超时: ${error.message}');
      // 显示超时错误提示
    } else {
      print('未知连接错误: $error');
      // 显示通用错误提示
    }
  }
  
  static void handleDataError(dynamic error) {
    if (error is FormatException) {
      print('数据格式错误: ${error.message}');
    } else if (error is TypeError) {
      print('数据类型错误: $error');
    } else {
      print('数据处理错误: $error');
    }
  }
  
  static Future<T?> safeExecute<T>(
    Future<T> Function() operation,
    {T? defaultValue}
  ) async {
    try {
      return await operation();
    } catch (e) {
      print('操作执行失败: $e');
      return defaultValue;
    }
  }
}
```

### 6.4 性能监控

```dart
// 性能监控
class PerformanceMonitor {
  static final Map<String, Stopwatch> _timers = {};
  static final Map<String, List<int>> _metrics = {};
  
  // 开始计时
  static void startTimer(String name) {
    _timers[name] = Stopwatch()..start();
  }
  
  // 停止计时
  static int stopTimer(String name) {
    final timer = _timers[name];
    if (timer != null) {
      timer.stop();
      final elapsed = timer.elapsedMilliseconds;
      
      _metrics.putIfAbsent(name, () => []).add(elapsed);
      _timers.remove(name);
      
      return elapsed;
    }
    return 0;
  }
  
  // 获取性能统计
  static Map<String, dynamic> getStats(String name) {
    final metrics = _metrics[name];
    if (metrics == null || metrics.isEmpty) {
      return {};
    }
    
    metrics.sort();
    final sum = metrics.reduce((a, b) => a + b);
    
    return {
      'count': metrics.length,
      'average': sum / metrics.length,
      'min': metrics.first,
      'max': metrics.last,
      'median': metrics[metrics.length ~/ 2],
    };
  }
  
  // 清除统计数据
  static void clearStats([String? name]) {
    if (name != null) {
      _metrics.remove(name);
    } else {
      _metrics.clear();
    }
  }
}
```

## 7. 故障排除

### 7.1 常见问题

1. **MQTT 连接失败**
   - 检查服务器地址和端口
   - 验证用户名和密码
   - 确认网络连接
   - 检查防火墙设置

2. **传感器数据异常**
   - 验证传感器权限
   - 检查设备兼容性
   - 确认传感器可用性
   - 重新校准传感器

3. **数据同步问题**
   - 检查网络状态
   - 验证 API 端点
   - 确认认证令牌
   - 检查数据格式

### 7.2 调试技巧

```dart
// 调试工具
class IoTDebugger {
  static bool _debugMode = false;
  
  static void enableDebug() {
    _debugMode = true;
  }
  
  static void log(String message, [String? tag]) {
    if (_debugMode) {
      final timestamp = DateTime.now().toIso8601String();
      print('[$timestamp] ${tag ?? 'IoT'}: $message');
    }
  }
  
  static void logData(String label, dynamic data) {
    if (_debugMode) {
      log('$label: ${json.encode(data)}', 'DATA');
    }
  }
  
  static void logError(String message, dynamic error, [StackTrace? stackTrace]) {
    if (_debugMode) {
      log('ERROR: $message - $error', 'ERROR');
      if (stackTrace != null) {
        log('Stack trace: $stackTrace', 'ERROR');
      }
    }
  }
}
```

通过本文档，您可以在 Flutter 应用中构建完整的物联网和边缘计算解决方案，实现设备连接、数据采集、本地处理和云端同步等功能。记住要根据具体需求选择合适的协议和架构，并注意性能优化和错误处理。