# 物联网和硬件集成组件

本文档详细介绍 Flutter 中物联网 (IoT) 和硬件集成相关组件的使用，包括蓝牙连接、WiFi 配置、传感器数据、串口通信等。

## 1. 蓝牙集成

### 1.1 基础配置

```yaml
# pubspec.yaml
dependencies:
  flutter_bluetooth_serial: ^0.4.0
  flutter_blue_plus: ^1.14.6
  permission_handler: ^11.0.1
```

### 1.2 蓝牙经典连接

```dart
// 蓝牙经典连接管理器
import 'package:flutter_bluetooth_serial/flutter_bluetooth_serial.dart';
import 'dart:typed_data';
import 'dart:convert';

class BluetoothClassicManager {
  static final BluetoothClassicManager _instance = BluetoothClassicManager._internal();
  factory BluetoothClassicManager() => _instance;
  BluetoothClassicManager._internal();
  
  BluetoothConnection? _connection;
  bool _isConnected = false;
  StreamSubscription<Uint8List>? _dataSubscription;
  
  final StreamController<String> _dataController = StreamController<String>.broadcast();
  Stream<String> get dataStream => _dataController.stream;
  
  bool get isConnected => _isConnected;
  
  // 获取已配对设备
  Future<List<BluetoothDevice>> getPairedDevices() async {
    try {
      return await FlutterBluetoothSerial.instance.getBondedDevices();
    } catch (e) {
      print('获取已配对设备失败: $e');
      return [];
    }
  }
  
  // 连接设备
  Future<bool> connectToDevice(BluetoothDevice device) async {
    try {
      _connection = await BluetoothConnection.toAddress(device.address);
      _isConnected = true;
      
      // 监听数据
      _dataSubscription = _connection!.input!.listen(
        (Uint8List data) {
          final message = utf8.decode(data);
          _dataController.add(message);
        },
        onError: (error) {
          print('蓝牙数据接收错误: $error');
          disconnect();
        },
        onDone: () {
          print('蓝牙连接断开');
          disconnect();
        },
      );
      
      print('蓝牙连接成功: ${device.name}');
      return true;
    } catch (e) {
      print('蓝牙连接失败: $e');
      _isConnected = false;
      return false;
    }
  }
  
  // 发送数据
  Future<bool> sendData(String data) async {
    if (!_isConnected || _connection == null) {
      print('蓝牙未连接');
      return false;
    }
    
    try {
      _connection!.output.add(Uint8List.fromList(utf8.encode(data)));
      await _connection!.output.allSent;
      return true;
    } catch (e) {
      print('发送数据失败: $e');
      return false;
    }
  }
  
  // 断开连接
  void disconnect() {
    _dataSubscription?.cancel();
    _connection?.dispose();
    _connection = null;
    _isConnected = false;
    print('蓝牙连接已断开');
  }
  
  void dispose() {
    disconnect();
    _dataController.close();
  }
}

// 蓝牙设备扫描和连接界面
class BluetoothDeviceScreen extends StatefulWidget {
  @override
  _BluetoothDeviceScreenState createState() => _BluetoothDeviceScreenState();
}

class _BluetoothDeviceScreenState extends State<BluetoothDeviceScreen> {
  final BluetoothClassicManager _bluetoothManager = BluetoothClassicManager();
  List<BluetoothDevice> _devices = [];
  bool _isLoading = false;
  String _receivedData = '';
  final TextEditingController _messageController = TextEditingController();
  
  @override
  void initState() {
    super.initState();
    _loadPairedDevices();
    _listenToData();
  }
  
  Future<void> _loadPairedDevices() async {
    setState(() {
      _isLoading = true;
    });
    
    try {
      final devices = await _bluetoothManager.getPairedDevices();
      setState(() {
        _devices = devices;
      });
    } catch (e) {
      print('加载设备失败: $e');
    } finally {
      setState(() {
        _isLoading = false;
      });
    }
  }
  
  void _listenToData() {
    _bluetoothManager.dataStream.listen((data) {
      setState(() {
        _receivedData += data;
      });
    });
  }
  
  Future<void> _connectToDevice(BluetoothDevice device) async {
    final success = await _bluetoothManager.connectToDevice(device);
    if (success) {
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(content: Text('已连接到 ${device.name}')),
      );
      setState(() {});
    } else {
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(content: Text('连接 ${device.name} 失败')),
      );
    }
  }
  
  Future<void> _sendMessage() async {
    if (_messageController.text.isNotEmpty) {
      final success = await _bluetoothManager.sendData(_messageController.text + '\n');
      if (success) {
        _messageController.clear();
      } else {
        ScaffoldMessenger.of(context).showSnackBar(
          const SnackBar(content: Text('发送消息失败')),
        );
      }
    }
  }
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('蓝牙设备'),
        actions: [
          IconButton(
            icon: const Icon(Icons.refresh),
            onPressed: _loadPairedDevices,
          ),
          if (_bluetoothManager.isConnected)
            IconButton(
              icon: const Icon(Icons.bluetooth_disabled),
              onPressed: () {
                _bluetoothManager.disconnect();
                setState(() {});
              },
            ),
        ],
      ),
      body: Column(
        children: [
          // 设备列表
          Expanded(
            flex: 1,
            child: _isLoading
                ? const Center(child: CircularProgressIndicator())
                : ListView.builder(
                    itemCount: _devices.length,
                    itemBuilder: (context, index) {
                      final device = _devices[index];
                      return ListTile(
                        leading: const Icon(Icons.bluetooth),
                        title: Text(device.name ?? '未知设备'),
                        subtitle: Text(device.address),
                        trailing: _bluetoothManager.isConnected
                            ? const Icon(Icons.check, color: Colors.green)
                            : const Icon(Icons.bluetooth_disabled),
                        onTap: _bluetoothManager.isConnected
                            ? null
                            : () => _connectToDevice(device),
                      );
                    },
                  ),
          ),
          
          const Divider(),
          
          // 数据接收区域
          Expanded(
            flex: 1,
            child: Container(
              padding: const EdgeInsets.all(16),
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  const Text(
                    '接收数据:',
                    style: TextStyle(
                      fontSize: 16,
                      fontWeight: FontWeight.bold,
                    ),
                  ),
                  const SizedBox(height: 8),
                  Expanded(
                    child: Container(
                      width: double.infinity,
                      padding: const EdgeInsets.all(8),
                      decoration: BoxDecoration(
                        border: Border.all(color: Colors.grey),
                        borderRadius: BorderRadius.circular(4),
                        color: Colors.grey[50],
                      ),
                      child: SingleChildScrollView(
                        child: Text(
                          _receivedData.isEmpty ? '暂无数据' : _receivedData,
                          style: const TextStyle(
                            fontFamily: 'monospace',
                            fontSize: 12,
                          ),
                        ),
                      ),
                    ),
                  ),
                  const SizedBox(height: 8),
                  Row(
                    children: [
                      Expanded(
                        child: TextField(
                          controller: _messageController,
                          decoration: const InputDecoration(
                            hintText: '输入要发送的命令',
                            border: OutlineInputBorder(),
                          ),
                          enabled: _serialManager.isConnected,
                        ),
                      ),
                      const SizedBox(width: 8),
                      ElevatedButton(
                        onPressed: _serialManager.isConnected ? _sendMessage : null,
                        child: const Text('发送'),
                      ),
                    ],
                  ),
                ],
              ),
            ),
          ),
        ],
      ),
    );
  }
  
  @override
  void dispose() {
    _messageController.dispose();
    _serialManager.dispose();
    super.dispose();
  }
}
```

## 5. 平台配置

### 5.1 Android 配置

```xml
<!-- android/app/src/main/AndroidManifest.xml -->
<uses-permission android:name="android.permission.BLUETOOTH" />
<uses-permission android:name="android.permission.BLUETOOTH_ADMIN" />
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
<uses-permission android:name="android.permission.BLUETOOTH_SCAN" />
<uses-permission android:name="android.permission.BLUETOOTH_CONNECT" />
<uses-permission android:name="android.permission.BLUETOOTH_ADVERTISE" />
<uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
<uses-permission android:name="android.permission.CHANGE_WIFI_STATE" />
<uses-permission android:name="android.permission.CHANGE_NETWORK_STATE" />
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.CAMERA" />
<uses-permission android:name="android.permission.RECORD_AUDIO" />
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
```

### 5.2 iOS 配置

```xml
<!-- ios/Runner/Info.plist -->
<key>NSBluetoothAlwaysUsageDescription</key>
<string>此应用需要蓝牙权限来连接IoT设备</string>
<key>NSBluetoothPeripheralUsageDescription</key>
<string>此应用需要蓝牙权限来连接IoT设备</string>
<key>NSLocationWhenInUseUsageDescription</key>
<string>此应用需要位置权限来获取传感器数据</string>
<key>NSLocationAlwaysAndWhenInUseUsageDescription</key>
<string>此应用需要位置权限来获取传感器数据</string>
<key>NSCameraUsageDescription</key>
<string>此应用需要相机权限来扫描设备二维码</string>
<key>NSMicrophoneUsageDescription</key>
<string>此应用需要麦克风权限来录制音频</string>
```

## 6. 最佳实践

### 6.1 连接管理

```dart
// 连接状态管理
class ConnectionStateManager {
  static const int maxRetryAttempts = 3;
  static const Duration retryDelay = Duration(seconds: 2);
  
  static Future<bool> connectWithRetry(
    Future<bool> Function() connectFunction,
  ) async {
    for (int attempt = 1; attempt <= maxRetryAttempts; attempt++) {
      try {
        final success = await connectFunction();
        if (success) {
          return true;
        }
      } catch (e) {
        print('连接尝试 $attempt 失败: $e');
      }
      
      if (attempt < maxRetryAttempts) {
        await Future.delayed(retryDelay);
      }
    }
    
    return false;
  }
  
  static Future<void> handleConnectionLoss(
    VoidCallback onReconnect,
  ) async {
    print('检测到连接丢失，尝试重连...');
    
    // 等待一段时间后重连
    await Future.delayed(const Duration(seconds: 5));
    onReconnect();
  }
}
```

### 6.2 数据缓存

```dart
// 传感器数据缓存
class SensorDataCache {
  static const int maxCacheSize = 1000;
  final Queue<SensorData> _cache = Queue<SensorData>();
  
  void addData(SensorData data) {
    _cache.add(data);
    
    // 保持缓存大小
    while (_cache.length > maxCacheSize) {
      _cache.removeFirst();
    }
  }
  
  List<SensorData> getDataByType(SensorType type) {
    return _cache.where((data) => data.type == type).toList();
  }
  
  List<SensorData> getDataInTimeRange(
    DateTime start,
    DateTime end,
  ) {
    return _cache
        .where((data) =>
            data.timestamp.isAfter(start) && data.timestamp.isBefore(end))
        .toList();
  }
  
  void clearCache() {
    _cache.clear();
  }
  
  int get cacheSize => _cache.length;
}
```

### 6.3 错误处理

```dart
// IoT 错误处理
class IoTErrorHandler {
  static void handleBluetoothError(dynamic error) {
    if (error.toString().contains('GATT_ERROR')) {
      print('蓝牙GATT错误，建议重启蓝牙');
    } else if (error.toString().contains('CONNECTION_TIMEOUT')) {
      print('连接超时，检查设备距离');
    } else {
      print('蓝牙未知错误: $error');
    }
  }
  
  static void handleWiFiError(dynamic error) {
    if (error.toString().contains('AUTHENTICATION_FAILED')) {
      print('WiFi认证失败，检查密码');
    } else if (error.toString().contains('NETWORK_NOT_FOUND')) {
      print('网络未找到，检查SSID');
    } else {
      print('WiFi未知错误: $error');
    }
  }
  
  static void handleSensorError(dynamic error) {
    if (error.toString().contains('PERMISSION_DENIED')) {
      print('传感器权限被拒绝');
    } else if (error.toString().contains('SENSOR_NOT_AVAILABLE')) {
      print('传感器不可用');
    } else {
      print('传感器未知错误: $error');
    }
  }
}
```

### 6.4 性能优化

```dart
// 数据采样优化
class DataSamplingOptimizer {
  static const Map<SensorType, Duration> samplingIntervals = {
    SensorType.accelerometer: Duration(milliseconds: 100),
    SensorType.gyroscope: Duration(milliseconds: 100),
    SensorType.magnetometer: Duration(milliseconds: 200),
    SensorType.light: Duration(seconds: 1),
    SensorType.location: Duration(seconds: 5),
    SensorType.battery: Duration(seconds: 30),
  };
  
  static bool shouldSample(SensorType type, DateTime lastSample) {
    final interval = samplingIntervals[type] ?? const Duration(seconds: 1);
    return DateTime.now().difference(lastSample) >= interval;
  }
}

// 内存管理
class MemoryManager {
  static void optimizeForIoT() {
    // 定期清理缓存
    Timer.periodic(const Duration(minutes: 5), (timer) {
      // 清理过期数据
      _cleanupExpiredData();
      
      // 强制垃圾回收
      _forceGarbageCollection();
    });
  }
  
  static void _cleanupExpiredData() {
    final cutoffTime = DateTime.now().subtract(const Duration(hours: 1));
    // 清理一小时前的数据
  }
  
  static void _forceGarbageCollection() {
    // 在Dart中，垃圾回收是自动的
    // 这里可以清理不必要的引用
  }
}
```

## 7. 故障排除

### 7.1 常见问题

1. **蓝牙连接失败**
   - 检查设备是否已配对
   - 确认蓝牙权限已授予
   - 重启蓝牙服务

2. **WiFi连接问题**
   - 验证网络密码
   - 检查网络安全类型
   - 确认设备支持的频段

3. **传感器数据异常**
   - 检查设备权限
   - 验证传感器可用性
   - 校准传感器数据

4. **串口通信中断**
   - 检查USB连接
   - 验证波特率设置
   - 确认数据格式

### 7.2 调试技巧

```dart
// IoT 调试工具
class IoTDebugger {
  static bool debugMode = false;
  
  static void log(String message) {
    if (debugMode) {
      print('[IoT Debug] ${DateTime.now()}: $message');
    }
  }
  
  static void logSensorData(SensorData data) {
    if (debugMode) {
      print('[Sensor] ${data.type}: ${data.data}');
    }
  }
  
  static void logConnectionEvent(String event, String details) {
    if (debugMode) {
      print('[Connection] $event: $details');
    }
  }
}
```

通过本文档，您可以在 Flutter 应用中集成各种物联网和硬件设备，实现丰富的IoT功能。记住要根据具体的硬件设备调整通信协议和数据格式。: 'monospace',
                            fontSize: 12,
                          ),
                        ),
                      ),
                    ),
                  ),
                  Row(
                    children: [
                      Expanded(
                        child: TextField(
                          controller: _messageController,
                          decoration: const InputDecoration(
                            hintText: '输入要发送的消息',
                            border: OutlineInputBorder(),
                          ),
                          enabled: _bluetoothManager.isConnected,
                        ),
                      ),
                      const SizedBox(width: 8),
                      ElevatedButton(
                        onPressed: _bluetoothManager.isConnected ? _sendMessage : null,
                        child: const Text('发送'),
                      ),
                    ],
                  ),
                ],
              ),
            ),
          ),
        ],
      ),
    );
  }
  
  @override
  void dispose() {
    _messageController.dispose();
    super.dispose();
  }
}
```

### 1.3 蓝牙低功耗 (BLE)

```dart
// BLE 设备管理器
import 'package:flutter_blue_plus/flutter_blue_plus.dart';

class BLEManager {
  static final BLEManager _instance = BLEManager._internal();
  factory BLEManager() => _instance;
  BLEManager._internal();
  
  BluetoothDevice? _connectedDevice;
  BluetoothCharacteristic? _writeCharacteristic;
  BluetoothCharacteristic? _notifyCharacteristic;
  
  final StreamController<String> _dataController = StreamController<String>.broadcast();
  Stream<String> get dataStream => _dataController.stream;
  
  bool get isConnected => _connectedDevice != null;
  
  // 扫描设备
  Stream<ScanResult> scanForDevices({Duration timeout = const Duration(seconds: 10)}) {
    FlutterBluePlus.startScan(timeout: timeout);
    return FlutterBluePlus.scanResults;
  }
  
  // 停止扫描
  void stopScan() {
    FlutterBluePlus.stopScan();
  }
  
  // 连接设备
  Future<bool> connectToDevice(BluetoothDevice device) async {
    try {
      await device.connect(timeout: const Duration(seconds: 10));
      _connectedDevice = device;
      
      // 发现服务
      final services = await device.discoverServices();
      
      // 查找特征值
      for (final service in services) {
        for (final characteristic in service.characteristics) {
          // 查找写特征值
          if (characteristic.properties.write) {
            _writeCharacteristic = characteristic;
          }
          
          // 查找通知特征值
          if (characteristic.properties.notify) {
            _notifyCharacteristic = characteristic;
            
            // 启用通知
            await characteristic.setNotifyValue(true);
            
            // 监听数据
            characteristic.value.listen((value) {
              if (value.isNotEmpty) {
                final data = utf8.decode(value);
                _dataController.add(data);
              }
            });
          }
        }
      }
      
      // 监听连接状态
      device.connectionState.listen((state) {
        if (state == BluetoothConnectionState.disconnected) {
          _handleDisconnection();
        }
      });
      
      print('BLE 设备连接成功: ${device.platformName}');
      return true;
    } catch (e) {
      print('BLE 连接失败: $e');
      return false;
    }
  }
  
  // 发送数据
  Future<bool> sendData(String data) async {
    if (_writeCharacteristic == null) {
      print('写特征值不可用');
      return false;
    }
    
    try {
      await _writeCharacteristic!.write(utf8.encode(data));
      return true;
    } catch (e) {
      print('发送数据失败: $e');
      return false;
    }
  }
  
  // 断开连接
  Future<void> disconnect() async {
    if (_connectedDevice != null) {
      await _connectedDevice!.disconnect();
      _handleDisconnection();
    }
  }
  
  void _handleDisconnection() {
    _connectedDevice = null;
    _writeCharacteristic = null;
    _notifyCharacteristic = null;
    print('BLE 设备已断开连接');
  }
  
  void dispose() {
    disconnect();
    _dataController.close();
  }
}

// BLE 设备扫描界面
class BLEDeviceScreen extends StatefulWidget {
  @override
  _BLEDeviceScreenState createState() => _BLEDeviceScreenState();
}

class _BLEDeviceScreenState extends State<BLEDeviceScreen> {
  final BLEManager _bleManager = BLEManager();
  List<ScanResult> _scanResults = [];
  bool _isScanning = false;
  String _receivedData = '';
  final TextEditingController _messageController = TextEditingController();
  
  @override
  void initState() {
    super.initState();
    _listenToData();
  }
  
  void _listenToData() {
    _bleManager.dataStream.listen((data) {
      setState(() {
        _receivedData += data;
      });
    });
  }
  
  void _startScan() {
    setState(() {
      _isScanning = true;
      _scanResults.clear();
    });
    
    _bleManager.scanForDevices().listen((result) {
      setState(() {
        // 避免重复添加
        final existingIndex = _scanResults.indexWhere(
          (r) => r.device.remoteId == result.device.remoteId,
        );
        
        if (existingIndex >= 0) {
          _scanResults[existingIndex] = result;
        } else {
          _scanResults.add(result);
        }
      });
    });
    
    // 10秒后停止扫描
    Timer(const Duration(seconds: 10), () {
      _stopScan();
    });
  }
  
  void _stopScan() {
    _bleManager.stopScan();
    setState(() {
      _isScanning = false;
    });
  }
  
  Future<void> _connectToDevice(BluetoothDevice device) async {
    final success = await _bleManager.connectToDevice(device);
    if (success) {
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(content: Text('已连接到 ${device.platformName}')),
      );
      setState(() {});
    } else {
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(content: Text('连接 ${device.platformName} 失败')),
      );
    }
  }
  
  Future<void> _sendMessage() async {
    if (_messageController.text.isNotEmpty) {
      final success = await _bleManager.sendData(_messageController.text);
      if (success) {
        _messageController.clear();
      } else {
        ScaffoldMessenger.of(context).showSnackBar(
          const SnackBar(content: Text('发送消息失败')),
        );
      }
    }
  }
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('BLE 设备'),
        actions: [
          if (_isScanning)
            const Padding(
              padding: EdgeInsets.all(16),
              child: SizedBox(
                width: 20,
                height: 20,
                child: CircularProgressIndicator(strokeWidth: 2),
              ),
            )
          else
            IconButton(
              icon: const Icon(Icons.search),
              onPressed: _startScan,
            ),
          if (_bleManager.isConnected)
            IconButton(
              icon: const Icon(Icons.bluetooth_disabled),
              onPressed: () {
                _bleManager.disconnect();
                setState(() {});
              },
            ),
        ],
      ),
      body: Column(
        children: [
          // 设备列表
          Expanded(
            flex: 1,
            child: ListView.builder(
              itemCount: _scanResults.length,
              itemBuilder: (context, index) {
                final result = _scanResults[index];
                final device = result.device;
                return ListTile(
                  leading: Icon(
                    Icons.bluetooth,
                    color: _bleManager.isConnected ? Colors.green : Colors.grey,
                  ),
                  title: Text(device.platformName.isNotEmpty ? device.platformName : '未知设备'),
                  subtitle: Column(
                    crossAxisAlignment: CrossAxisAlignment.start,
                    children: [
                      Text(device.remoteId.toString()),
                      Text('RSSI: ${result.rssi} dBm'),
                    ],
                  ),
                  trailing: _bleManager.isConnected
                      ? const Icon(Icons.check, color: Colors.green)
                      : null,
                  onTap: _bleManager.isConnected
                      ? null
                      : () => _connectToDevice(device),
                );
              },
            ),
          ),
          
          const Divider(),
          
          // 数据交互区域
          Expanded(
            flex: 1,
            child: Container(
              padding: const EdgeInsets.all(16),
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  const Text(
                    '接收数据:',
                    style: TextStyle(
                      fontSize: 16,
                      fontWeight: FontWeight.bold,
                    ),
                  ),
                  const SizedBox(height: 8),
                  Expanded(
                    child: Container(
                      width: double.infinity,
                      padding: const EdgeInsets.all(8),
                      decoration: BoxDecoration(
                        border: Border.all(color: Colors.grey),
                        borderRadius: BorderRadius.circular(4),
                        color: Colors.grey[50],
                      ),
                      child: SingleChildScrollView(
                        child: Text(
                          _receivedData.isEmpty ? '暂无数据' : _receivedData,
                          style: const TextStyle(
                            fontFamily: 'monospace',
                            fontSize: 12,
                          ),
                        ),
                      ),
                    ),
                  ),
                  const SizedBox(height: 8),
                  Row(
                    children: [
                      Expanded(
                        child: TextField(
                          controller: _messageController,
                          decoration: const InputDecoration(
                            hintText: '输入要发送的消息',
                            border: OutlineInputBorder(),
                          ),
                          enabled: _bleManager.isConnected,
                        ),
                      ),
                      const SizedBox(width: 8),
                      ElevatedButton(
                        onPressed: _bleManager.isConnected ? _sendMessage : null,
                        child: const Text('发送'),
                      ),
                    ],
                  ),
                ],
              ),
            ),
          ),
        ],
      ),
    );
  }
  
  @override
  void dispose() {
    _messageController.dispose();
    _bleManager.dispose();
    super.dispose();
  }
}
```

## 2. WiFi 和网络配置

### 2.1 WiFi 管理

```yaml
# pubspec.yaml
dependencies:
  wifi_iot: ^0.3.18
  connectivity_plus: ^5.0.1
  network_info_plus: ^4.1.0
```

```dart
// WiFi 管理器
import 'package:wifi_iot/wifi_iot.dart';
import 'package:connectivity_plus/connectivity_plus.dart';
import 'package:network_info_plus/network_info_plus.dart';

class WiFiManager {
  static final WiFiManager _instance = WiFiManager._internal();
  factory WiFiManager() => _instance;
  WiFiManager._internal();
  
  final Connectivity _connectivity = Connectivity();
  final NetworkInfo _networkInfo = NetworkInfo();
  
  // 获取当前连接状态
  Future<ConnectivityResult> getCurrentConnectivity() async {
    return await _connectivity.checkConnectivity();
  }
  
  // 获取当前 WiFi 信息
  Future<WiFiInfo?> getCurrentWiFiInfo() async {
    try {
      final ssid = await _networkInfo.getWifiName();
      final bssid = await _networkInfo.getWifiBSSID();
      final ip = await _networkInfo.getWifiIP();
      final gateway = await _networkInfo.getWifiGatewayIP();
      final subnet = await _networkInfo.getWifiSubmask();
      
      if (ssid != null) {
        return WiFiInfo(
          ssid: ssid.replaceAll('"', ''), // 移除引号
          bssid: bssid ?? '',
          ipAddress: ip ?? '',
          gateway: gateway ?? '',
          subnet: subnet ?? '',
        );
      }
    } catch (e) {
      print('获取 WiFi 信息失败: $e');
    }
    return null;
  }
  
  // 扫描 WiFi 网络
  Future<List<WifiNetwork>> scanWiFiNetworks() async {
    try {
      final networks = await WiFiForIoTPlugin.loadWifiList();
      return networks;
    } catch (e) {
      print('扫描 WiFi 网络失败: $e');
      return [];
    }
  }
  
  // 连接到 WiFi 网络
  Future<bool> connectToWiFi(String ssid, String password) async {
    try {
      final result = await WiFiForIoTPlugin.connect(
        ssid,
        password: password,
        security: NetworkSecurity.WPA,
      );
      return result;
    } catch (e) {
      print('连接 WiFi 失败: $e');
      return false;
    }
  }
  
  // 断开 WiFi 连接
  Future<bool> disconnectWiFi() async {
    try {
      return await WiFiForIoTPlugin.disconnect();
    } catch (e) {
      print('断开 WiFi 失败: $e');
      return false;
    }
  }
  
  // 创建热点
  Future<bool> createHotspot(String ssid, String password) async {
    try {
      return await WiFiForIoTPlugin.setWiFiAPEnabled(
        ssid,
        password,
        NetworkSecurity.WPA,
      );
    } catch (e) {
      print('创建热点失败: $e');
      return false;
    }
  }
  
  // 监听连接状态变化
  Stream<ConnectivityResult> get connectivityStream {
    return _connectivity.onConnectivityChanged;
  }
}

// WiFi 信息类
class WiFiInfo {
  final String ssid;
  final String bssid;
  final String ipAddress;
  final String gateway;
  final String subnet;
  
  WiFiInfo({
    required this.ssid,
    required this.bssid,
    required this.ipAddress,
    required this.gateway,
    required this.subnet,
  });
}

// WiFi 管理界面
class WiFiManagerScreen extends StatefulWidget {
  @override
  _WiFiManagerScreenState createState() => _WiFiManagerScreenState();
}

class _WiFiManagerScreenState extends State<WiFiManagerScreen> {
  final WiFiManager _wifiManager = WiFiManager();
  WiFiInfo? _currentWiFi;
  List<WifiNetwork> _availableNetworks = [];
  bool _isScanning = false;
  ConnectivityResult _connectivityStatus = ConnectivityResult.none;
  
  @override
  void initState() {
    super.initState();
    _loadCurrentWiFiInfo();
    _checkConnectivity();
    _listenToConnectivityChanges();
  }
  
  Future<void> _loadCurrentWiFiInfo() async {
    final wifiInfo = await _wifiManager.getCurrentWiFiInfo();
    setState(() {
      _currentWiFi = wifiInfo;
    });
  }
  
  Future<void> _checkConnectivity() async {
    final status = await _wifiManager.getCurrentConnectivity();
    setState(() {
      _connectivityStatus = status;
    });
  }
  
  void _listenToConnectivityChanges() {
    _wifiManager.connectivityStream.listen((status) {
      setState(() {
        _connectivityStatus = status;
      });
      
      if (status == ConnectivityResult.wifi) {
        _loadCurrentWiFiInfo();
      } else {
        setState(() {
          _currentWiFi = null;
        });
      }
    });
  }
  
  Future<void> _scanWiFiNetworks() async {
    setState(() {
      _isScanning = true;
    });
    
    try {
      final networks = await _wifiManager.scanWiFiNetworks();
      setState(() {
        _availableNetworks = networks;
      });
    } catch (e) {
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(content: Text('扫描失败: $e')),
      );
    } finally {
      setState(() {
        _isScanning = false;
      });
    }
  }
  
  void _showConnectDialog(WifiNetwork network) {
    final passwordController = TextEditingController();
    
    showDialog(
      context: context,
      builder: (context) => AlertDialog(
        title: Text('连接到 ${network.ssid}'),
        content: TextField(
          controller: passwordController,
          decoration: const InputDecoration(
            labelText: '密码',
            border: OutlineInputBorder(),
          ),
          obscureText: true,
        ),
        actions: [
          TextButton(
            onPressed: () => Navigator.pop(context),
            child: const Text('取消'),
          ),
          ElevatedButton(
            onPressed: () async {
              Navigator.pop(context);
              await _connectToWiFi(network.ssid!, passwordController.text);
            },
            child: const Text('连接'),
          ),
        ],
      ),
    );
  }
  
  Future<void> _connectToWiFi(String ssid, String password) async {
    final success = await _wifiManager.connectToWiFi(ssid, password);
    
    ScaffoldMessenger.of(context).showSnackBar(
      SnackBar(
        content: Text(success ? '连接成功' : '连接失败'),
        backgroundColor: success ? Colors.green : Colors.red,
      ),
    );
    
    if (success) {
      await Future.delayed(const Duration(seconds: 2));
      await _loadCurrentWiFiInfo();
    }
  }
  
  String _getConnectivityStatusText() {
    switch (_connectivityStatus) {
      case ConnectivityResult.wifi:
        return 'WiFi 已连接';
      case ConnectivityResult.mobile:
        return '移动网络已连接';
      case ConnectivityResult.ethernet:
        return '以太网已连接';
      case ConnectivityResult.none:
        return '无网络连接';
      default:
        return '未知状态';
    }
  }
  
  Color _getConnectivityStatusColor() {
    switch (_connectivityStatus) {
      case ConnectivityResult.wifi:
      case ConnectivityResult.mobile:
      case ConnectivityResult.ethernet:
        return Colors.green;
      case ConnectivityResult.none:
        return Colors.red;
      default:
        return Colors.grey;
    }
  }
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('WiFi 管理'),
        actions: [
          IconButton(
            icon: const Icon(Icons.refresh),
            onPressed: _scanWiFiNetworks,
          ),
        ],
      ),
      body: Column(
        children: [
          // 连接状态
          Container(
            width: double.infinity,
            padding: const EdgeInsets.all(16),
            color: _getConnectivityStatusColor().withOpacity(0.1),
            child: Row(
              children: [
                Icon(
                  _connectivityStatus == ConnectivityResult.wifi
                      ? Icons.wifi
                      : _connectivityStatus == ConnectivityResult.mobile
                          ? Icons.signal_cellular_4_bar
                          : Icons.signal_wifi_off,
                  color: _getConnectivityStatusColor(),
                ),
                const SizedBox(width: 8),
                Text(
                  _getConnectivityStatusText(),
                  style: TextStyle(
                    color: _getConnectivityStatusColor(),
                    fontWeight: FontWeight.bold,
                  ),
                ),
              ],
            ),
          ),
          
          // 当前 WiFi 信息
          if (_currentWiFi != null) ..[
            const Padding(
              padding: EdgeInsets.all(16),
              child: Align(
                alignment: Alignment.centerLeft,
                child: Text(
                  '当前连接:',
                  style: TextStyle(
                    fontSize: 18,
                    fontWeight: FontWeight.bold,
                  ),
                ),
              ),
            ),
            Card(
              margin: const EdgeInsets.symmetric(horizontal: 16),
              child: Padding(
                padding: const EdgeInsets.all(16),
                child: Column(
                  crossAxisAlignment: CrossAxisAlignment.start,
                  children: [
                    _buildInfoRow('SSID', _currentWiFi!.ssid),
                    _buildInfoRow('BSSID', _currentWiFi!.bssid),
                    _buildInfoRow('IP 地址', _currentWiFi!.ipAddress),
                    _buildInfoRow('网关', _currentWiFi!.gateway),
                    _buildInfoRow('子网掩码', _currentWiFi!.subnet),
                  ],
                ),
              ),
            ),
          ],
          
          const SizedBox(height: 20),
          
          // 可用网络
          Padding(
            padding: const EdgeInsets.symmetric(horizontal: 16),
            child: Row(
              mainAxisAlignment: MainAxisAlignment.spaceBetween,
              children: [
                const Text(
                  '可用网络:',
                  style: TextStyle(
                    fontSize: 18,
                    fontWeight: FontWeight.bold,
                  ),
                ),
                if (_isScanning)
                  const SizedBox(
                    width: 20,
                    height: 20,
                    child: CircularProgressIndicator(strokeWidth: 2),
                  ),
              ],
            ),
          ),
          
          const SizedBox(height: 10),
          
          Expanded(
            child: ListView.builder(
              itemCount: _availableNetworks.length,
              itemBuilder: (context, index) {
                final network = _availableNetworks[index];
                return ListTile(
                  leading: Icon(
                    _getWiFiIcon(network.level ?? 0),
                    color: _getSignalColor(network.level ?? 0),
                  ),
                  title: Text(network.ssid ?? '未知网络'),
                  subtitle: Text(
                    '${network.capabilities ?? ''} • ${network.level ?? 0} dBm',
                  ),
                  trailing: _currentWiFi?.ssid == network.ssid
                      ? const Icon(Icons.check, color: Colors.green)
                      : null,
                  onTap: _currentWiFi?.ssid == network.ssid
                      ? null
                      : () => _showConnectDialog(network),
                );
              },
            ),
          ),
        ],
      ),
    );
  }
  
  Widget _buildInfoRow(String label, String value) {
    return Padding(
      padding: const EdgeInsets.symmetric(vertical: 4),
      child: Row(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          SizedBox(
            width: 80,
            child: Text(
              '$label:',
              style: const TextStyle(fontWeight: FontWeight.bold),
            ),
          ),
          Expanded(
            child: Text(
              value.isEmpty ? '未知' : value,
              style: const TextStyle(fontFamily: 'monospace'),
            ),
          ),
        ],
      ),
    );
  }
  
  IconData _getWiFiIcon(int level) {
    if (level >= -50) return Icons.wifi;
    if (level >= -60) return Icons.wifi_2_bar;
    if (level >= -70) return Icons.wifi_1_bar;
    return Icons.wifi_off;
  }
  
  Color _getSignalColor(int level) {
    if (level >= -50) return Colors.green;
    if (level >= -60) return Colors.orange;
    if (level >= -70) return Colors.red;
    return Colors.grey;
  }
}
```

## 3. 传感器数据采集

### 3.1 设备传感器

```yaml
# pubspec.yaml
dependencies:
  sensors_plus: ^4.0.2
  geolocator: ^10.1.0
  light: ^3.0.1
  battery_plus: ^5.0.2
```

```dart
// 传感器数据管理器
import 'package:sensors_plus/sensors_plus.dart';
import 'package:geolocator/geolocator.dart';
import 'package:light/light.dart';
import 'package:battery_plus/battery_plus.dart';

class SensorDataManager {
  static final SensorDataManager _instance = SensorDataManager._internal();
  factory SensorDataManager() => _instance;
  SensorDataManager._internal();
  
  // 传感器数据流
  final StreamController<SensorData> _sensorDataController = 
      StreamController<SensorData>.broadcast();
  Stream<SensorData> get sensorDataStream => _sensorDataController.stream;
  
  // 传感器订阅
  StreamSubscription<AccelerometerEvent>? _accelerometerSubscription;
  StreamSubscription<GyroscopeEvent>? _gyroscopeSubscription;
  StreamSubscription<MagnetometerEvent>? _magnetometerSubscription;
  StreamSubscription<int>? _lightSubscription;
  
  final Battery _battery = Battery();
  Light? _light;
  
  bool _isListening = false;
  
  // 开始监听传感器
  Future<void> startListening() async {
    if (_isListening) return;
    
    _isListening = true;
    
    // 加速度计
    _accelerometerSubscription = accelerometerEvents.listen((event) {
      _sensorDataController.add(SensorData(
        type: SensorType.accelerometer,
        timestamp: DateTime.now(),
        data: {'x': event.x, 'y': event.y, 'z': event.z},
      ));
    });
    
    // 陀螺仪
    _gyroscopeSubscription = gyroscopeEvents.listen((event) {
      _sensorDataController.add(SensorData(
        type: SensorType.gyroscope,
        timestamp: DateTime.now(),
        data: {'x': event.x, 'y': event.y, 'z': event.z},
      ));
    });
    
    // 磁力计
    _magnetometerSubscription = magnetometerEvents.listen((event) {
      _sensorDataController.add(SensorData(
        type: SensorType.magnetometer,
        timestamp: DateTime.now(),
        data: {'x': event.x, 'y': event.y, 'z': event.z},
      ));
    });
    
    // 光线传感器
    try {
      _light = Light();
      _lightSubscription = _light!.lightSensorStream.listen((luxValue) {
        _sensorDataController.add(SensorData(
          type: SensorType.light,
          timestamp: DateTime.now(),
          data: {'lux': luxValue},
        ));
      });
    } catch (e) {
      print('光线传感器不可用: $e');
    }
    
    // 定期获取位置和电池信息
    Timer.periodic(const Duration(seconds: 5), (timer) {
      if (!_isListening) {
        timer.cancel();
        return;
      }
      
      _updateLocationData();
      _updateBatteryData();
    });
    
    print('传感器监听已启动');
  }
  
  // 停止监听传感器
  void stopListening() {
    _isListening = false;
    
    _accelerometerSubscription?.cancel();
    _gyroscopeSubscription?.cancel();
    _magnetometerSubscription?.cancel();
    _lightSubscription?.cancel();
    
    _accelerometerSubscription = null;
    _gyroscopeSubscription = null;
    _magnetometerSubscription = null;
    _lightSubscription = null;
    
    print('传感器监听已停止');
  }
  
  // 更新位置数据
  Future<void> _updateLocationData() async {
    try {
      final permission = await Geolocator.checkPermission();
      if (permission == LocationPermission.denied) {
        await Geolocator.requestPermission();
      }
      
      final position = await Geolocator.getCurrentPosition(
        desiredAccuracy: LocationAccuracy.high,
        timeLimit: const Duration(seconds: 5),
      );
      
      _sensorDataController.add(SensorData(
        type: SensorType.location,
        timestamp: DateTime.now(),
        data: {
          'latitude': position.latitude,
          'longitude': position.longitude,
          'altitude': position.altitude,
          'accuracy': position.accuracy,
          'speed': position.speed,
        },
      ));
    } catch (e) {
      print('获取位置失败: $e');
    }
  }
  
  // 更新电池数据
  Future<void> _updateBatteryData() async {
    try {
      final batteryLevel = await _battery.batteryLevel;
      final batteryState = await _battery.batteryState;
      
      _sensorDataController.add(SensorData(
        type: SensorType.battery,
        timestamp: DateTime.now(),
        data: {
          'level': batteryLevel,
          'state': batteryState.toString(),
        },
      ));
    } catch (e) {
      print('获取电池信息失败: $e');
    }
  }
  
  void dispose() {
    stopListening();
    _sensorDataController.close();
  }
}

// 传感器数据类
class SensorData {
  final SensorType type;
  final DateTime timestamp;
  final Map<String, dynamic> data;
  
  SensorData({
    required this.type,
    required this.timestamp,
    required this.data,
  });
  
  @override
  String toString() {
    return 'SensorData(type: $type, timestamp: $timestamp, data: $data)';
  }
}

// 传感器类型枚举
enum SensorType {
  accelerometer,
  gyroscope,
  magnetometer,
  light,
  location,
  battery,
}

// 传感器数据显示界面
class SensorDataScreen extends StatefulWidget {
  @override
  _SensorDataScreenState createState() => _SensorDataScreenState();
}

class _SensorDataScreenState extends State<SensorDataScreen> {
  final SensorDataManager _sensorManager = SensorDataManager();
  final Map<SensorType, SensorData> _latestData = {};
  bool _isListening = false;
  
  @override
  void initState() {
    super.initState();
    _listenToSensorData();
  }
  
  void _listenToSensorData() {
    _sensorManager.sensorDataStream.listen((data) {
      setState(() {
        _latestData[data.type] = data;
      });
    });
  }
  
  void _toggleListening() {
    if (_isListening) {
      _sensorManager.stopListening();
    } else {
      _sensorManager.startListening();
    }
    
    setState(() {
      _isListening = !_isListening;
    });
  }
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('传感器数据'),
        actions: [
          IconButton(
            icon: Icon(_isListening ? Icons.stop : Icons.play_arrow),
            onPressed: _toggleListening,
          ),
        ],
      ),
      body: _isListening
          ? ListView(
              padding: const EdgeInsets.all(16),
              children: [
                _buildSensorCard(
                  '加速度计',
                  Icons.speed,
                  SensorType.accelerometer,
                  (data) => [
                    'X: ${data['x']?.toStringAsFixed(3) ?? 'N/A'} m/s²',
                    'Y: ${data['y']?.toStringAsFixed(3) ?? 'N/A'} m/s²',
                    'Z: ${data['z']?.toStringAsFixed(3) ?? 'N/A'} m/s²',
                  ],
                ),
                
                _buildSensorCard(
                  '陀螺仪',
                  Icons.rotate_right,
                  SensorType.gyroscope,
                  (data) => [
                    'X: ${data['x']?.toStringAsFixed(3) ?? 'N/A'} rad/s',
                    'Y: ${data['y']?.toStringAsFixed(3) ?? 'N/A'} rad/s',
                    'Z: ${data['z']?.toStringAsFixed(3) ?? 'N/A'} rad/s',
                  ],
                ),
                
                _buildSensorCard(
                  '磁力计',
                  Icons.explore,
                  SensorType.magnetometer,
                  (data) => [
                    'X: ${data['x']?.toStringAsFixed(3) ?? 'N/A'} µT',
                    'Y: ${data['y']?.toStringAsFixed(3) ?? 'N/A'} µT',
                    'Z: ${data['z']?.toStringAsFixed(3) ?? 'N/A'} µT',
                  ],
                ),
                
                _buildSensorCard(
                  '光线传感器',
                  Icons.wb_sunny,
                  SensorType.light,
                  (data) => [
                    '亮度: ${data['lux']?.toString() ?? 'N/A'} lux',
                  ],
                ),
                
                _buildSensorCard(
                  '位置信息',
                  Icons.location_on,
                  SensorType.location,
                  (data) => [
                    '纬度: ${data['latitude']?.toStringAsFixed(6) ?? 'N/A'}°',
                    '经度: ${data['longitude']?.toStringAsFixed(6) ?? 'N/A'}°',
                    '海拔: ${data['altitude']?.toStringAsFixed(1) ?? 'N/A'} m',
                    '精度: ${data['accuracy']?.toStringAsFixed(1) ?? 'N/A'} m',
                    '速度: ${data['speed']?.toStringAsFixed(1) ?? 'N/A'} m/s',
                  ],
                ),
                
                _buildSensorCard(
                  '电池信息',
                  Icons.battery_full,
                  SensorType.battery,
                  (data) => [
                    '电量: ${data['level']?.toString() ?? 'N/A'}%',
                    '状态: ${data['state']?.toString().split('.').last ?? 'N/A'}',
                  ],
                ),
              ],
            )
          : const Center(
              child: Column(
                mainAxisAlignment: MainAxisAlignment.center,
                children: [
                  Icon(
                    Icons.sensors,
                    size: 80,
                    color: Colors.grey,
                  ),
                  SizedBox(height: 20),
                  Text(
                    '点击播放按钮开始监听传感器数据',
                    style: TextStyle(fontSize: 16),
                  ),
                ],
              ),
            ),
    );
  }
  
  Widget _buildSensorCard(
    String title,
    IconData icon,
    SensorType sensorType,
    List<String> Function(Map<String, dynamic>) dataFormatter,
  ) {
    final sensorData = _latestData[sensorType];
    final isActive = sensorData != null &&
        DateTime.now().difference(sensorData.timestamp).inSeconds < 10;
    
    return Card(
      margin: const EdgeInsets.only(bottom: 16),
      child: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Row(
              children: [
                Icon(
                  icon,
                  color: isActive ? Colors.green : Colors.grey,
                ),
                const SizedBox(width: 8),
                Text(
                  title,
                  style: const TextStyle(
                    fontSize: 18,
                    fontWeight: FontWeight.bold,
                  ),
                ),
                const Spacer(),
                Container(
                  width: 12,
                  height: 12,
                  decoration: BoxDecoration(
                    shape: BoxShape.circle,
                    color: isActive ? Colors.green : Colors.grey,
                  ),
                ),
              ],
            ),
            const SizedBox(height: 12),
            
            if (sensorData != null) ..[
              ...dataFormatter(sensorData.data).map((text) => Padding(
                padding: const EdgeInsets.symmetric(vertical: 2),
                child: Text(
                  text,
                  style: const TextStyle(
                    fontFamily: 'monospace',
                    fontSize: 14,
                  ),
                ),
              )),
              const SizedBox(height: 8),
              Text(
                '更新时间: ${DateFormat('HH:mm:ss').format(sensorData.timestamp)}',
                style: const TextStyle(
                  color: Colors.grey,
                  fontSize: 12,
                ),
              ),
            ] else
              const Text(
                '暂无数据',
                style: TextStyle(color: Colors.grey),
              ),
          ],
        ),
      ),
    );
  }
  
  @override
  void dispose() {
    _sensorManager.dispose();
    super.dispose();
  }
}
```

## 4. 串口通信

### 4.1 USB 串口通信

```yaml
# pubspec.yaml
dependencies:
  usb_serial: ^0.5.0
```

```dart
// USB 串口管理器
import 'package:usb_serial/usb_serial.dart';

class USBSerialManager {
  static final USBSerialManager _instance = USBSerialManager._internal();
  factory USBSerialManager() => _instance;
  USBSerialManager._internal();
  
  UsbPort? _port;
  StreamSubscription<String>? _dataSubscription;
  
  final StreamController<String> _dataController = StreamController<String>.broadcast();
  Stream<String> get dataStream => _dataController.stream;
  
  bool get isConnected => _port != null;
  
  // 获取可用设备
  Future<List<UsbDevice>> getAvailableDevices() async {
    return await UsbSerial.listDevices();
  }
  
  // 连接设备
  Future<bool> connectToDevice(UsbDevice device) async {
    try {
      _port = await device.create();
      
      if (_port != null) {
        await _port!.open();
        await _port!.setDTR(true);
        await _port!.setRTS(true);
        await _port!.setPortParameters(
          115200, // 波特率
          UsbPort.DATABITS_8,
          UsbPort.STOPBITS_1,
          UsbPort.PARITY_NONE,
        );
        
        // 监听数据
        _dataSubscription = _port!.inputStream?.listen(
          (data) {
            _dataController.add(data);
          },
          onError: (error) {
            print('串口数据接收错误: $error');
            disconnect();
          },
        );
        
        print('USB 串口连接成功');
        return true;
      }
    } catch (e) {
      print('USB 串口连接失败: $e');
    }
    
    return false;
  }
  
  // 发送数据
  Future<bool> sendData(String data) async {
    if (_port == null) {
      print('串口未连接');
      return false;
    }
    
    try {
      await _port!.write(Uint8List.fromList(data.codeUnits));
      return true;
    } catch (e) {
      print('发送数据失败: $e');
      return false;
    }
  }
  
  // 断开连接
  Future<void> disconnect() async {
    _dataSubscription?.cancel();
    await _port?.close();
    _port = null;
    print('USB 串口连接已断开');
  }
  
  void dispose() {
    disconnect();
    _dataController.close();
  }
}

// USB 串口界面
class USBSerialScreen extends StatefulWidget {
  @override
  _USBSerialScreenState createState() => _USBSerialScreenState();
}

class _USBSerialScreenState extends State<USBSerialScreen> {
  final USBSerialManager _serialManager = USBSerialManager();
  List<UsbDevice> _devices = [];
  String _receivedData = '';
  final TextEditingController _messageController = TextEditingController();
  
  @override
  void initState() {
    super.initState();
    _loadDevices();
    _listenToData();
  }
  
  Future<void> _loadDevices() async {
    final devices = await _serialManager.getAvailableDevices();
    setState(() {
      _devices = devices;
    });
  }
  
  void _listenToData() {
    _serialManager.dataStream.listen((data) {
      setState(() {
        _receivedData += data;
      });
    });
  }
  
  Future<void> _connectToDevice(UsbDevice device) async {
    final success = await _serialManager.connectToDevice(device);
    
    ScaffoldMessenger.of(context).showSnackBar(
      SnackBar(
        content: Text(success ? '连接成功' : '连接失败'),
        backgroundColor: success ? Colors.green : Colors.red,
      ),
    );
    
    setState(() {});
  }
  
  Future<void> _sendMessage() async {
    if (_messageController.text.isNotEmpty) {
      final success = await _serialManager.sendData(_messageController.text + '\r\n');
      if (success) {
        _messageController.clear();
      } else {
        ScaffoldMessenger.of(context).showSnackBar(
          const SnackBar(content: Text('发送消息失败')),
        );
      }
    }
  }
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('USB 串口'),
        actions: [
          IconButton(
            icon: const Icon(Icons.refresh),
            onPressed: _loadDevices,
          ),
          if (_serialManager.isConnected)
            IconButton(
              icon: const Icon(Icons.close),
              onPressed: () {
                _serialManager.disconnect();
                setState(() {});
              },
            ),
        ],
      ),
      body: Column(
        children: [
          // 设备列表
          Expanded(
            flex: 1,
            child: ListView.builder(
              itemCount: _devices.length,
              itemBuilder: (context, index) {
                final device = _devices[index];
                return ListTile(
                  leading: const Icon(Icons.usb),
                  title: Text(device.productName ?? '未知设备'),
                  subtitle: Text(
                    'VID: ${device.vid?.toRadixString(16).toUpperCase()} '
                    'PID: ${device.pid?.toRadixString(16).toUpperCase()}',
                  ),
                  trailing: _serialManager.isConnected
                      ? const Icon(Icons.check, color: Colors.green)
                      : null,
                  onTap: _serialManager.isConnected
                      ? null
                      : () => _connectToDevice(device),
                );
              },
            ),
          ),
          
          const Divider(),
          
          // 数据交互区域
          Expanded(
            flex: 1,
            child: Container(
              padding: const EdgeInsets.all(16),
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  const Text(
                    '接收数据:',
                    style: TextStyle(
                      fontSize: 16,
                      fontWeight: FontWeight.bold,
                    ),
                  ),
                  const SizedBox(height: 8),
                  Expanded(
                    child: Container(
                      width: double.infinity,
                      padding: const EdgeInsets.all(8),
                      decoration: BoxDecoration(
                        border: Border.all(color: Colors.grey),
                        borderRadius: BorderRadius.circular(4),
                        color: Colors.grey[50],
                      ),
                      child: SingleChildScrollView(
                        child: Text(
                          _receivedData.isEmpty ? '暂无数据' : _receivedData,
                          style: const TextStyle(
                            fontFamily