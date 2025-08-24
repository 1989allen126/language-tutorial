# Flutter 网络检测实现

本文档详细介绍 Flutter 应用中网络检测的实现方案，包括网络状态监控、连接质量检测、网络速度测试等功能。

## 📋 目录

- [网络检测基础](#网络检测基础)
- [网络状态监控](#网络状态监控)
- [网络质量检测](#网络质量检测)
- [网络速度测试](#网络速度测试)
- [网络诊断工具](#网络诊断工具)
- [实际应用案例](#实际应用案例)
- [最佳实践](#最佳实践)

## 网络检测基础

### 架构图

```

## 网络质量检测

### 网络速度测试

```dart
class NetworkSpeedTest {
  static final Logger _logger = Logger();
  static final Dio _dio = Dio();

  // 下载速度测试
  static Future<SpeedTestResult> testDownloadSpeed({
    String? testUrl,
    Duration timeout = const Duration(seconds: 30),
    int fileSizeBytes = 10485760, // 10MB
  }) async {
    testUrl ??= 'https://httpbin.org/bytes/$fileSizeBytes';

    final stopwatch = Stopwatch();
    int downloadedBytes = 0;

    try {
      stopwatch.start();

      final response = await _dio.get(
        testUrl,
        options: Options(
          receiveTimeout: timeout,
          responseType: ResponseType.stream,
        ),
        onReceiveProgress: (received, total) {
          downloadedBytes = received;
        },
      );

      // 确保完全下载
      if (response.data is ResponseBody) {
        await (response.data as ResponseBody).stream.drain();
      }

      stopwatch.stop();

      final elapsedSeconds = stopwatch.elapsedMilliseconds / 1000.0;
      final speedMbps = (downloadedBytes * 8) / (elapsedSeconds * 1000000);

      return SpeedTestResult(
        type: SpeedTestType.download,
        speedMbps: speedMbps,
        bytesTransferred: downloadedBytes,
        elapsedTime: Duration(milliseconds: stopwatch.elapsedMilliseconds),
        success: true,
        timestamp: DateTime.now(),
      );
    } catch (e) {
      stopwatch.stop();
      _logger.e('Download speed test failed: $e');

      return SpeedTestResult(
        type: SpeedTestType.download,
        speedMbps: 0,
        bytesTransferred: downloadedBytes,
        elapsedTime: Duration(milliseconds: stopwatch.elapsedMilliseconds),
        success: false,
        error: e.toString(),
        timestamp: DateTime.now(),
      );
    }
  }

  // 上传速度测试
  static Future<SpeedTestResult> testUploadSpeed({
    String? testUrl,
    Duration timeout = const Duration(seconds: 30),
    int fileSizeBytes = 5242880, // 5MB
  }) async {
    testUrl ??= 'https://httpbin.org/post';

    // 生成测试数据
    final testData = List.generate(fileSizeBytes, (index) => index % 256);

    final stopwatch = Stopwatch();
    int uploadedBytes = 0;

    try {
      stopwatch.start();

      final formData = FormData.fromMap({
        'file': MultipartFile.fromBytes(
          testData,
          filename: 'speedtest.bin',
        ),
      });

      await _dio.post(
        testUrl,
        data: formData,
        options: Options(
          sendTimeout: timeout,
          receiveTimeout: timeout,
        ),
        onSendProgress: (sent, total) {
          uploadedBytes = sent;
        },
      );

      stopwatch.stop();

      final elapsedSeconds = stopwatch.elapsedMilliseconds / 1000.0;
      final speedMbps = (uploadedBytes * 8) / (elapsedSeconds * 1000000);

      return SpeedTestResult(
        type: SpeedTestType.upload,
        speedMbps: speedMbps,
        bytesTransferred: uploadedBytes,
        elapsedTime: Duration(milliseconds: stopwatch.elapsedMilliseconds),
        success: true,
        timestamp: DateTime.now(),
      );
    } catch (e) {
      stopwatch.stop();
      _logger.e('Upload speed test failed: $e');

      return SpeedTestResult(
        type: SpeedTestType.upload,
        speedMbps: 0,
        bytesTransferred: uploadedBytes,
        elapsedTime: Duration(milliseconds: stopwatch.elapsedMilliseconds),
        success: false,
        error: e.toString(),
        timestamp: DateTime.now(),
      );
    }
  }

  // 综合网络速度测试
  static Future<ComprehensiveSpeedTest> runComprehensiveTest({
    Duration timeout = const Duration(seconds: 30),
  }) async {
    final results = <SpeedTestResult>[];

    // 下载速度测试
    final downloadResult = await testDownloadSpeed(timeout: timeout);
    results.add(downloadResult);

    // 上传速度测试
    final uploadResult = await testUploadSpeed(timeout: timeout);
    results.add(uploadResult);

    return ComprehensiveSpeedTest(
      speedTestResults: results,
      timestamp: DateTime.now(),
    );
  }
}

// 速度测试类型
enum SpeedTestType {
  download,
  upload,
}

// 速度测试结果
class SpeedTestResult {
  final SpeedTestType type;
  final double speedMbps;
  final int bytesTransferred;
  final Duration elapsedTime;
  final bool success;
  final String? error;
  final DateTime timestamp;

  SpeedTestResult({
    required this.type,
    required this.speedMbps,
    required this.bytesTransferred,
    required this.elapsedTime,
    required this.success,
    this.error,
    required this.timestamp,
  });

  String get formattedSpeed {
    if (speedMbps < 1) {
      return '${(speedMbps * 1000).toStringAsFixed(0)} Kbps';
    } else {
      return '${speedMbps.toStringAsFixed(2)} Mbps';
    }
  }

  String get typeDisplayName {
    switch (type) {
      case SpeedTestType.download:
        return '下载';
      case SpeedTestType.upload:
        return '上传';
    }
  }
}

// 综合速度测试结果
class ComprehensiveSpeedTest {
  final List<SpeedTestResult> speedTestResults;
  final DateTime timestamp;

  ComprehensiveSpeedTest({
    required this.speedTestResults,
    required this.timestamp,
  });

  SpeedTestResult? get downloadResult {
    return speedTestResults
        .where((r) => r.type == SpeedTestType.download)
        .firstOrNull;
  }

  SpeedTestResult? get uploadResult {
    return speedTestResults
        .where((r) => r.type == SpeedTestType.upload)
        .firstOrNull;
  }

  String get networkQualityAssessment {
    final download = downloadResult?.speedMbps ?? 0;
    final upload = uploadResult?.speedMbps ?? 0;

    if (download > 25 && upload > 5) {
      return '网络质量优秀';
    } else if (download > 10 && upload > 2) {
      return '网络质量良好';
    } else if (download > 5 && upload > 1) {
      return '网络质量一般';
    } else {
      return '网络质量较差';
    }
  }
}
```

## 网络诊断工具

### DNS 解析测试

```dart
class DNSResolver {
  static final Logger _logger = Logger();

  // DNS 解析测试
  static Future<DNSResolveResult> resolveDomain(
    String domain, {
    List<String>? dnsServers,
    Duration timeout = const Duration(seconds: 5),
  }) async {
    dnsServers ??= [
      '8.8.8.8',      // Google DNS
      '1.1.1.1',      // Cloudflare DNS
      '208.67.222.222', // OpenDNS
    ];

    final results = <String, DNSServerResult>{};

    for (final dnsServer in dnsServers) {
      final stopwatch = Stopwatch()..start();

      try {
        final addresses = await InternetAddress.lookup(
          domain,
          type: InternetAddressType.any,
        ).timeout(timeout);

        stopwatch.stop();

        results[dnsServer] = DNSServerResult(
          dnsServer: dnsServer,
          domain: domain,
          resolvedAddresses: addresses.map((addr) => addr.address).toList(),
          resolveTime: stopwatch.elapsedMilliseconds,
          success: true,
          timestamp: DateTime.now(),
        );
      } catch (e) {
        stopwatch.stop();

        results[dnsServer] = DNSServerResult(
          dnsServer: dnsServer,
          domain: domain,
          resolvedAddresses: [],
          resolveTime: stopwatch.elapsedMilliseconds,
          success: false,
          error: e.toString(),
          timestamp: DateTime.now(),
        );
      }
    }

    return DNSResolveResult(
      domain: domain,
      serverResults: results,
      timestamp: DateTime.now(),
    );
  }

  // 批量域名解析测试
  static Future<Map<String, DNSResolveResult>> resolveMultipleDomains(
    List<String> domains, {
    List<String>? dnsServers,
    Duration timeout = const Duration(seconds: 5),
  }) async {
    final results = <String, DNSResolveResult>{};

    final futures = domains.map((domain) => resolveDomain(
      domain,
      dnsServers: dnsServers,
      timeout: timeout,
    ));

    final resolveResults = await Future.wait(futures);

    for (int i = 0; i < domains.length; i++) {
      results[domains[i]] = resolveResults[i];
    }

    return results;
  }
}

// DNS 服务器结果
class DNSServerResult {
  final String dnsServer;
  final String domain;
  final List<String> resolvedAddresses;
  final int resolveTime; // 毫秒
  final bool success;
  final String? error;
  final DateTime timestamp;

  DNSServerResult({
    required this.dnsServer,
    required this.domain,
    required this.resolvedAddresses,
    required this.resolveTime,
    required this.success,
    this.error,
    required this.timestamp,
  });
}

// DNS 解析结果
class DNSResolveResult {
  final String domain;
  final Map<String, DNSServerResult> serverResults;
  final DateTime timestamp;

  DNSResolveResult({
    required this.domain,
    required this.serverResults,
    required this.timestamp,
  });

  List<DNSServerResult> get successfulResults {
    return serverResults.values.where((r) => r.success).toList();
  }

  double get averageResolveTime {
    final successful = successfulResults;
    if (successful.isEmpty) return -1;

    final totalTime = successful
        .map((r) => r.resolveTime)
        .reduce((a, b) => a + b);

    return totalTime / successful.length;
  }

  Set<String> get allResolvedAddresses {
    final addresses = <String>{};
    for (final result in successfulResults) {
      addresses.addAll(result.resolvedAddresses);
    }
    return addresses;
  }
}
```

### 路由跟踪工具

```dart
class TraceRoute {
  static final Logger _logger = Logger();

  // 路由跟踪（简化版本，使用递增TTL的ping）
  static Future<TraceRouteResult> traceRoute(
    String destination, {
    int maxHops = 30,
    Duration timeout = const Duration(seconds: 5),
  }) async {
    final hops = <TraceRouteHop>[];

    for (int ttl = 1; ttl <= maxHops; ttl++) {
      final hop = await _performHop(destination, ttl, timeout);
      hops.add(hop);

      // 如果到达目标主机，停止跟踪
      if (hop.isDestination) {
        break;
      }

      // 如果连续失败太多次，停止跟踪
      if (hops.length >= 5) {
        final recentHops = hops.takeLast(5);
        if (recentHops.every((h) => !h.success)) {
          break;
        }
      }
    }

    return TraceRouteResult(
      destination: destination,
      hops: hops,
      timestamp: DateTime.now(),
    );
  }

  // 执行单跳测试
  static Future<TraceRouteHop> _performHop(
    String destination,
    int ttl,
    Duration timeout,
  ) async {
    final stopwatch = Stopwatch()..start();

    try {
      // 简化实现：直接ping目标，实际应该设置TTL
      final socket = await Socket.connect(
        destination,
        80,
        timeout: timeout,
      );

      await socket.close();
      stopwatch.stop();

      return TraceRouteHop(
        hopNumber: ttl,
        address: destination,
        hostname: destination,
        latency: stopwatch.elapsedMilliseconds.toDouble(),
        success: true,
        isDestination: true,
        timestamp: DateTime.now(),
      );
    } catch (e) {
      stopwatch.stop();

      return TraceRouteHop(
        hopNumber: ttl,
        address: '*',
        hostname: '*',
        latency: -1,
        success: false,
        isDestination: false,
        error: e.toString(),
        timestamp: DateTime.now(),
      );
    }
  }
}

// 路由跟踪跳点
class TraceRouteHop {
  final int hopNumber;
  final String address;
  final String hostname;
  final double latency; // 毫秒，-1表示失败
  final bool success;
  final bool isDestination;
  final String? error;
  final DateTime timestamp;

  TraceRouteHop({
    required this.hopNumber,
    required this.address,
    required this.hostname,
    required this.latency,
    required this.success,
    required this.isDestination,
    this.error,
    required this.timestamp,
  });

  String get displayText {
    if (!success) {
      return '$hopNumber\t*\t*\t*';
    }

    return '$hopNumber\t$address\t$hostname\t${latency.toStringAsFixed(1)}ms';
  }
}

// 路由跟踪结果
class TraceRouteResult {
  final String destination;
  final List<TraceRouteHop> hops;
  final DateTime timestamp;

  TraceRouteResult({
    required this.destination,
    required this.hops,
    required this.timestamp,
  });

  bool get reachedDestination {
    return hops.any((hop) => hop.isDestination && hop.success);
  }

  int get totalHops => hops.length;

  double get averageLatency {
    final successfulHops = hops.where((h) => h.success).toList();
    if (successfulHops.isEmpty) return -1;

    final totalLatency = successfulHops
        .map((h) => h.latency)
        .reduce((a, b) => a + b);

    return totalLatency / successfulHops.length;
  }
}
```

## 实际应用案例

### 网络监控仪表板

```dart
class NetworkMonitoringDashboard extends StatefulWidget {
  @override
  State<NetworkMonitoringDashboard> createState() =>
      _NetworkMonitoringDashboardState();
}

class _NetworkMonitoringDashboardState
    extends State<NetworkMonitoringDashboard> {
  final NetworkDetectionManager _networkManager =
      NetworkDetectionManager.instance;

  NetworkStatus? _networkStatus;
  NetworkQuality? _networkQuality;
  ComprehensiveSpeedTest? _speedTest;
  DNSResolveResult? _dnsResult;

  bool _isRunningTests = false;

  @override
  void initState() {
    super.initState();
    _initializeMonitoring();
  }

  void _initializeMonitoring() {
    // 监听网络状态变化
    _networkManager.networkStatusStream.listen((status) {
      setState(() {
        _networkStatus = status;
      });
    });

    _networkManager.networkQualityStream.listen((quality) {
      setState(() {
        _networkQuality = quality;
      });
    });

    // 运行初始测试
    _runNetworkTests();
  }

  Future<void> _runNetworkTests() async {
    if (_isRunningTests) return;

    setState(() {
      _isRunningTests = true;
    });

    try {
      // 并行运行多个测试
      final futures = await Future.wait([
        NetworkSpeedTest.runComprehensiveTest(),
        DNSResolver.resolveDomain('www.google.com'),
      ]);

      setState(() {
        _speedTest = futures[0] as ComprehensiveSpeedTest;
        _dnsResult = futures[1] as DNSResolveResult;
      });
    } catch (e) {
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(content: Text('网络测试失败: $e')),
      );
    } finally {
      setState(() {
        _isRunningTests = false;
      });
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('网络监控仪表板'),
        actions: [
          IconButton(
            icon: _isRunningTests
                ? const CircularProgressIndicator(color: Colors.white)
                : const Icon(Icons.refresh),
            onPressed: _isRunningTests ? null : _runNetworkTests,
          ),
        ],
      ),
      body: RefreshIndicator(
        onRefresh: _runNetworkTests,
        child: ListView(
          padding: const EdgeInsets.all(16),
          children: [
            _buildNetworkStatusCard(),
            const SizedBox(height: 16),
            _buildSpeedTestCard(),
            const SizedBox(height: 16),
            _buildDNSResultCard(),
          ],
        ),
      ),
    );
  }

  Widget _buildNetworkStatusCard() {
    return Card(
      child: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Row(
              children: [
                Icon(
                  Icons.network_check,
                  color: _networkQuality?.color ?? Colors.grey,
                ),
                const SizedBox(width: 8),
                const Text(
                  '网络状态',
                  style: TextStyle(
                    fontSize: 18,
                    fontWeight: FontWeight.bold,
                  ),
                ),
              ],
            ),
            const SizedBox(height: 12),
            if (_networkStatus != null) ..[
              _buildStatusRow('连接类型', _networkStatus!.connectionType),
              _buildStatusRow('连接状态',
                  _networkStatus!.isConnected ? '已连接' : '未连接'),
              if (_networkStatus!.wifiName != null)
                _buildStatusRow('WiFi名称', _networkStatus!.wifiName!),
              if (_networkStatus!.wifiIP != null)
                _buildStatusRow('IP地址', _networkStatus!.wifiIP!),
              _buildStatusRow('网络质量',
                  _networkQuality?.displayName ?? '检测中...'),
            ] else
              const Text('正在检测网络状态...'),
          ],
        ),
      ),
    );
  }



  Widget _buildSpeedTestCard() {
    return Card(
      child: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            const Row(
              children: [
                Icon(Icons.speed_outlined, color: Colors.green),
                SizedBox(width: 8),
                Text(
                  '速度测试',
                  style: TextStyle(
                    fontSize: 18,
                    fontWeight: FontWeight.bold,
                  ),
                ),
              ],
            ),
            const SizedBox(height: 12),
            if (_speedTest != null) ..[
              if (_speedTest!.downloadResult != null)
                _buildStatusRow('下载速度',
                    _speedTest!.downloadResult!.formattedSpeed),
              if (_speedTest!.uploadResult != null)
                _buildStatusRow('上传速度',
                    _speedTest!.uploadResult!.formattedSpeed),
              _buildStatusRow('网络评估',
                  _speedTest!.networkQualityAssessment),
            ] else
              const Text('正在进行速度测试...'),
          ],
        ),
      ),
    );
  }

  Widget _buildDNSResultCard() {
    return Card(
      child: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            const Row(
              children: [
                Icon(Icons.dns, color: Colors.orange),
                SizedBox(width: 8),
                Text(
                  'DNS 解析',
                  style: TextStyle(
                    fontSize: 18,
                    fontWeight: FontWeight.bold,
                  ),
                ),
              ],
            ),
            const SizedBox(height: 12),
            if (_dnsResult != null) ..[
              _buildStatusRow('测试域名', _dnsResult!.domain),
              _buildStatusRow('解析时间',
                  '${_dnsResult!.averageResolveTime.toStringAsFixed(1)} ms'),
              _buildStatusRow('解析地址',
                  _dnsResult!.allResolvedAddresses.join(', ')),
            ] else
              const Text('正在进行 DNS 解析测试...'),
          ],
        ),
      ),
    );
  }

  Widget _buildStatusRow(String label, String value) {
    return Padding(
      padding: const EdgeInsets.symmetric(vertical: 4),
      child: Row(
        mainAxisAlignment: MainAxisAlignment.spaceBetween,
        children: [
          Text(
            label,
            style: const TextStyle(fontWeight: FontWeight.w500),
          ),
          Text(
            value,
            style: const TextStyle(color: Colors.grey),
          ),
        ],
      ),
    );
  }
}
```

## 最佳实践

### 设计原则

1. **非阻塞检测**
   - 使用异步操作避免阻塞UI
   - 合理设置超时时间
   - 提供取消机制

2. **智能检测频率**
   - 根据网络状态调整检测频率
   - 避免过度消耗电量和流量
   - 在网络状态变化时立即检测

3. **用户体验优化**
   - 提供清晰的状态指示
   - 支持手动刷新
   - 显示详细的错误信息

### 性能优化

```dart
class NetworkDetectionOptimizer {
  // 智能检测策略
  static Duration getOptimalCheckInterval(NetworkQuality quality) {
    switch (quality) {
      case NetworkQuality.excellent:
        return const Duration(minutes: 5);
      case NetworkQuality.good:
        return const Duration(minutes: 3);
      case NetworkQuality.fair:
        return const Duration(minutes: 2);
      case NetworkQuality.poor:
        return const Duration(minutes: 1);
      case NetworkQuality.disconnected:
        return const Duration(seconds: 30);
      default:
        return const Duration(minutes: 2);
    }
  }

  // 批量测试优化
  static Future<List<T>> runBatchTests<T>(
    List<Future<T> Function()> testFunctions, {
    int maxConcurrency = 3,
  }) async {
    final results = <T>[];

    for (int i = 0; i < testFunctions.length; i += maxConcurrency) {
      final batch = testFunctions
          .skip(i)
          .take(maxConcurrency)
          .map((fn) => fn())
          .toList();

      final batchResults = await Future.wait(batch);
      results.addAll(batchResults);
    }

    return results;
  }

  // 缓存机制
  static final Map<String, CachedResult> _cache = {};

  static Future<T> getCachedResult<T>(
    String key,
    Future<T> Function() computation, {
    Duration cacheDuration = const Duration(minutes: 5),
  }) async {
    final cached = _cache[key];

    if (cached != null &&
        DateTime.now().difference(cached.timestamp) < cacheDuration) {
      return cached.result as T;
    }

    final result = await computation();
    _cache[key] = CachedResult(result, DateTime.now());

    return result;
  }
}

class CachedResult {
  final dynamic result;
  final DateTime timestamp;

  CachedResult(this.result, this.timestamp);
}
```

### 错误处理

```dart
class NetworkDetectionErrorHandler {
  static final Logger _logger = Logger();

  // 统一错误处理
  static Future<T> handleNetworkOperation<T>(
    Future<T> Function() operation,
    T defaultValue, {
    String? operationName,
    bool logError = true,
  }) async {
    try {
      return await operation();
    } on SocketException catch (e) {
      if (logError) {
        _logger.w('Socket error in ${operationName ?? 'network operation'}: $e');
      }
      return defaultValue;
    } on TimeoutException catch (e) {
      if (logError) {
        _logger.w('Timeout in ${operationName ?? 'network operation'}: $e');
      }
      return defaultValue;
    } on DioException catch (e) {
      if (logError) {
        _logger.w('HTTP error in ${operationName ?? 'network operation'}: ${e.message}');
      }
      return defaultValue;
    } catch (e) {
      if (logError) {
        _logger.e('Unexpected error in ${operationName ?? 'network operation'}: $e');
      }
      return defaultValue;
    }
  }

  // 重试机制
  static Future<T> retryOperation<T>(
    Future<T> Function() operation, {
    int maxRetries = 3,
    Duration delay = const Duration(seconds: 1),
    bool Function(dynamic error)? shouldRetry,
  }) async {
    int attempts = 0;

    while (attempts < maxRetries) {
      try {
        return await operation();
      } catch (e) {
        attempts++;

        if (attempts >= maxRetries ||
            (shouldRetry != null && !shouldRetry(e))) {
          rethrow;
        }

        await Future.delayed(delay * attempts);
      }
    }

    throw StateError('Max retries exceeded');
  }
}
```

### 测试策略

```dart
// 网络检测测试
class NetworkDetectionTest {
  static Future<void> runAllTests() async {
    await testSpeedTest();
    await testDNSResolution();
    await testNetworkMonitoring();
  }

  static Future<void> testSpeedTest() async {
    print('Testing Speed Test functionality...');

    final speedTest = await NetworkSpeedTest.runComprehensiveTest(
      timeout: const Duration(seconds: 10),
    );

    assert(speedTest.speedTestResults.isNotEmpty, 'Speed test results should not be empty');

    print('Speed test completed');
  }

  static Future<void> testDNSResolution() async {
    print('Testing DNS Resolution...');

    final dnsResult = await DNSResolver.resolveDomain('www.google.com');
    assert(dnsResult.successfulResults.isNotEmpty, 'DNS resolution should succeed');
    assert(dnsResult.allResolvedAddresses.isNotEmpty, 'Should resolve to at least one address');

    print('DNS resolution test completed');
  }

  static Future<void> testNetworkMonitoring() async {
    print('Testing Network Monitoring...');

    final networkManager = NetworkDetectionManager.instance;

    // 测试网络状态获取
    final networkStatus = networkManager.currentNetworkStatus;
    assert(networkStatus != null, 'Network status should be available');

    // 测试网络质量获取
    final networkQuality = networkManager.currentNetworkQuality;
    assert(networkQuality != null, 'Network quality should be available');

    print('Network monitoring test completed');
  }
}
```

## 总结

网络检测是 Flutter 应用中重要的网络诊断功能，本文档涵盖了：

### 关键要点

1. **全面的网络检测**：包括连接状态、网络质量、速度测试等
2. **网络速度测试**：支持下载和上传速度检测
3. **网络诊断工具**：DNS 解析、路由跟踪等高级功能
4. **实时监控**：提供流式数据和事件通知
5. **用户友好界面**：直观的监控仪表板

### 最佳实践建议

1. **合理的检测频率**：避免过度消耗资源
2. **智能缓存策略**：提高性能和用户体验
3. **完善的错误处理**：提供重试和降级机制
4. **全面的测试覆盖**：确保功能的可靠性
5. **性能优化**：批量处理和并发控制

### 相关文档链接

- [connectivity_plus 官方文档](https://pub.dev/packages/connectivity_plus)
- [network_info_plus 官方文档](https://pub.dev/packages/network_info_plus)
- [Flutter 网络编程指南](https://flutter.dev/docs/cookbook/networking)
- [Dart Socket 编程](https://dart.dev/tutorials/server/httpserver)
- [网络安全最佳实践](./README.md)mermaid
graph TB
    subgraph "网络检测系统架构"
        A[网络检测管理器] --> B[连接状态监控]
        A --> C[网络速度测试]
        A --> D[网络质量检测]
        A --> E[网络诊断]

        B --> F[WiFi 检测]
        B --> G[移动网络检测]
        B --> H[以太网检测]

        C --> I[下载速度测试]
        C --> J[上传速度测试]
        C --> K[综合速度测试]

        D --> L[连接质量检测]
        D --> M[带宽测试]
        D --> N[网络稳定性测试]

        E --> O[路由跟踪]
        E --> P[DNS 解析测试]
        E --> Q[端口扫描]

        subgraph "监控指标"
            R[连接状态]
            S[信号强度]
            T[网络类型]
            U[IP 地址]
            V[下载速度]
            W[上传速度]
        end

        B --> R
        B --> S
        B --> T
        B --> U
        C --> V
        C --> W

        subgraph "事件通知"
            X[连接状态变化]
            Y[网络质量变化]
            Z[网络异常告警]
        end

        A --> X
        A --> Y
        A --> Z
    end
```

### 基础依赖配置

```yaml
# pubspec.yaml
dependencies:
  flutter:
    sdk: flutter

  # 网络连接检测
  connectivity_plus: ^4.0.2

  # 网络信息获取
  network_info_plus: ^4.0.2

  # HTTP 请求
  dio: ^5.3.2
  http: ^1.1.0

  # 权限管理
  permission_handler: ^11.0.1

  # 设备信息
  device_info_plus: ^9.1.0

  # 原生平台交互
  flutter/services.dart

  # 异步处理
  rxdart: ^0.27.7

  # 日志记录
  logger: ^2.0.2
```

### 网络检测管理器

```dart
import 'dart:async';
import 'dart:io';
import 'dart:math';
import 'package:connectivity_plus/connectivity_plus.dart';
import 'package:network_info_plus/network_info_plus.dart';
import 'package:dio/dio.dart';
import 'package:rxdart/rxdart.dart';
import 'package:logger/logger.dart';

class NetworkDetectionManager {
  static final NetworkDetectionManager _instance = NetworkDetectionManager._internal();
  static NetworkDetectionManager get instance => _instance;

  NetworkDetectionManager._internal() {
    _initializeNetworkMonitoring();
  }

  final Connectivity _connectivity = Connectivity();
  final NetworkInfo _networkInfo = NetworkInfo();
  final Dio _dio = Dio();
  final Logger _logger = Logger();

  // 网络状态流
  final BehaviorSubject<NetworkStatus> _networkStatusSubject =
      BehaviorSubject<NetworkStatus>.seeded(NetworkStatus.unknown);

  // 网络质量流
  final BehaviorSubject<NetworkQuality> _networkQualitySubject =
      BehaviorSubject<NetworkQuality>.seeded(NetworkQuality.unknown);

  // 订阅管理
  StreamSubscription<ConnectivityResult>? _connectivitySubscription;
  Timer? _qualityCheckTimer;

  // 公共流
  Stream<NetworkStatus> get networkStatusStream => _networkStatusSubject.stream;
  Stream<NetworkQuality> get networkQualityStream => _networkQualitySubject.stream;

  // 当前状态
  NetworkStatus get currentNetworkStatus => _networkStatusSubject.value;
  NetworkQuality get currentNetworkQuality => _networkQualitySubject.value;

  // 初始化网络监控
  void _initializeNetworkMonitoring() {
    _connectivitySubscription = _connectivity.onConnectivityChanged.listen(
      _onConnectivityChanged,
      onError: (error) {
        _logger.e('Connectivity monitoring error: $error');
      },
    );

    // 定期检查网络质量
    _qualityCheckTimer = Timer.periodic(
      const Duration(seconds: 30),
      (_) => _checkNetworkQuality(),
    );

    // 初始检查
    _checkInitialNetworkStatus();
  }

  // 连接状态变化处理
  Future<void> _onConnectivityChanged(ConnectivityResult result) async {
    final networkStatus = await _buildNetworkStatus(result);
    _networkStatusSubject.add(networkStatus);

    _logger.i('Network connectivity changed: ${result.name}');

    // 连接状态变化时立即检查网络质量
    if (networkStatus.isConnected) {
      await _checkNetworkQuality();
    } else {
      _networkQualitySubject.add(NetworkQuality.disconnected);
    }
  }

  // 检查初始网络状态
  Future<void> _checkInitialNetworkStatus() async {
    try {
      final result = await _connectivity.checkConnectivity();
      final networkStatus = await _buildNetworkStatus(result);
      _networkStatusSubject.add(networkStatus);

      if (networkStatus.isConnected) {
        await _checkNetworkQuality();
      }
    } catch (e) {
      _logger.e('Failed to check initial network status: $e');
    }
  }

  // 构建网络状态
  Future<NetworkStatus> _buildNetworkStatus(ConnectivityResult result) async {
    try {
      String? wifiName;
      String? wifiBSSID;
      String? wifiIP;
      String? wifiIPv6;
      String? wifiSubmask;
      String? wifiGateway;
      String? wifiBroadcast;

      if (result == ConnectivityResult.wifi) {
        wifiName = await _networkInfo.getWifiName();
        wifiBSSID = await _networkInfo.getWifiBSSID();
        wifiIP = await _networkInfo.getWifiIP();
        wifiIPv6 = await _networkInfo.getWifiIPv6();
        wifiSubmask = await _networkInfo.getWifiSubmask();
        wifiGateway = await _networkInfo.getWifiGatewayIP();
        wifiBroadcast = await _networkInfo.getWifiBroadcast();
      }

      return NetworkStatus(
        connectivityResult: result,
        isConnected: result != ConnectivityResult.none,
        wifiName: wifiName,
        wifiBSSID: wifiBSSID,
        wifiIP: wifiIP,
        wifiIPv6: wifiIPv6,
        wifiSubmask: wifiSubmask,
        wifiGateway: wifiGateway,
        wifiBroadcast: wifiBroadcast,
        timestamp: DateTime.now(),
      );
    } catch (e) {
      _logger.e('Failed to build network status: $e');
      return NetworkStatus(
        connectivityResult: result,
        isConnected: result != ConnectivityResult.none,
        timestamp: DateTime.now(),
      );
    }
  }

  // 检查网络质量
  Future<void> _checkNetworkQuality() async {
    if (!currentNetworkStatus.isConnected) {
      _networkQualitySubject.add(NetworkQuality.disconnected);
      return;
    }

    try {
      final downloadSpeed = await _measureDownloadSpeed();

      final quality = _calculateNetworkQuality(
        downloadSpeed: downloadSpeed,
      );

      _networkQualitySubject.add(quality);
    } catch (e) {
      _logger.e('Failed to check network quality: $e');
      _networkQualitySubject.add(NetworkQuality.poor);
    }
  }

  // 计算网络质量
  NetworkQuality _calculateNetworkQuality({
    required double latency,
    required double packetLoss,
    required double downloadSpeed,
  }) {
    // 基于延迟、丢包率和下载速度计算网络质量
    int score = 100;

    // 延迟评分 (0-40分)
    if (latency > 200) {
      score -= 40;
    } else if (latency > 100) {
      score -= 20;
    } else if (latency > 50) {
      score -= 10;
    }

    // 丢包率评分 (0-30分)
    if (packetLoss > 10) {
      score -= 30;
    } else if (packetLoss > 5) {
      score -= 20;
    } else if (packetLoss > 1) {
      score -= 10;
    }

    // 下载速度评分 (0-30分)
    if (downloadSpeed < 1) { // 1 Mbps
      score -= 30;
    } else if (downloadSpeed < 5) { // 5 Mbps
      score -= 20;
    } else if (downloadSpeed < 10) { // 10 Mbps
      score -= 10;
    }

    if (score >= 80) {
      return NetworkQuality.excellent;
    } else if (score >= 60) {
      return NetworkQuality.good;
    } else if (score >= 40) {
      return NetworkQuality.fair;
    } else {
      return NetworkQuality.poor;
    }
  }

  // 测量下载速度
  Future<double> _measureDownloadSpeed() async {
    try {
      const testUrl = 'https://httpbin.org/bytes/1048576'; // 1MB 测试文件
      const timeout = Duration(seconds: 10);

      final stopwatch = Stopwatch()..start();

      final response = await _dio.get(
        testUrl,
        options: Options(
          receiveTimeout: timeout,
          sendTimeout: timeout,
        ),
      );

      stopwatch.stop();

      if (response.statusCode == 200) {
        final bytes = response.data.toString().length;
        final seconds = stopwatch.elapsedMilliseconds / 1000.0;
        final mbps = (bytes * 8) / (seconds * 1000000); // 转换为 Mbps

        return mbps;
      }

      return 0.0;
    } catch (e) {
      _logger.w('Failed to measure download speed: $e');
      return 0.0;
    }
  }

  // 手动刷新网络状态
  Future<void> refreshNetworkStatus() async {
    await _checkInitialNetworkStatus();
  }

  // 手动检查网络质量
  Future<void> refreshNetworkQuality() async {
    await _checkNetworkQuality();
  }

  // 释放资源
  void dispose() {
    _connectivitySubscription?.cancel();
    _qualityCheckTimer?.cancel();
    _networkStatusSubject.close();
    _networkQualitySubject.close();
  }
}

// 网络状态模型
class NetworkStatus {
  final ConnectivityResult connectivityResult;
  final bool isConnected;
  final String? wifiName;
  final String? wifiBSSID;
  final String? wifiIP;
  final String? wifiIPv6;
  final String? wifiSubmask;
  final String? wifiGateway;
  final String? wifiBroadcast;
  final DateTime timestamp;

  NetworkStatus({
    required this.connectivityResult,
    required this.isConnected,
    this.wifiName,
    this.wifiBSSID,
    this.wifiIP,
    this.wifiIPv6,
    this.wifiSubmask,
    this.wifiGateway,
    this.wifiBroadcast,
    required this.timestamp,
  });

  String get connectionType {
    switch (connectivityResult) {
      case ConnectivityResult.wifi:
        return 'WiFi';
      case ConnectivityResult.mobile:
        return 'Mobile';
      case ConnectivityResult.ethernet:
        return 'Ethernet';
      case ConnectivityResult.bluetooth:
        return 'Bluetooth';
      case ConnectivityResult.vpn:
        return 'VPN';
      case ConnectivityResult.other:
        return 'Other';
      case ConnectivityResult.none:
      default:
        return 'None';
    }
  }

  Map<String, dynamic> toJson() {
    return {
      'connectivityResult': connectivityResult.name,
      'isConnected': isConnected,
      'connectionType': connectionType,
      'wifiName': wifiName,
      'wifiBSSID': wifiBSSID,
      'wifiIP': wifiIP,
      'wifiIPv6': wifiIPv6,
      'wifiSubmask': wifiSubmask,
      'wifiGateway': wifiGateway,
      'wifiBroadcast': wifiBroadcast,
      'timestamp': timestamp.toIso8601String(),
    };
  }
}

// 网络质量枚举
enum NetworkQuality {
  unknown,
  disconnected,
  poor,
  fair,
  good,
  excellent,
}

extension NetworkQualityExtension on NetworkQuality {
  String get displayName {
    switch (this) {
      case NetworkQuality.unknown:
        return '未知';
      case NetworkQuality.disconnected:
        return '未连接';
      case NetworkQuality.poor:
        return '差';
      case NetworkQuality.fair:
        return '一般';
      case NetworkQuality.good:
        return '良好';
      case NetworkQuality.excellent:
        return '优秀';
    }
  }

  Color get color {
    switch (this) {
      case NetworkQuality.unknown:
        return Colors.grey;
      case NetworkQuality.disconnected:
        return Colors.red;
      case NetworkQuality.poor:
        return Colors.red;
      case NetworkQuality.fair:
        return Colors.orange;
      case NetworkQuality.good:
        return Colors.green;
      case NetworkQuality.excellent:
        return Colors.blue;
    }
  }
}
```

## 网络状态监控

### 网络状态监控组件

```dart
class NetworkStatusWidget extends StatefulWidget {
  final Widget child;
  final bool showNetworkIndicator;
  final VoidCallback? onNetworkChanged;

  const NetworkStatusWidget({
    Key? key,
    required this.child,
    this.showNetworkIndicator = true,
    this.onNetworkChanged,
  }) : super(key: key);

  @override
  State<NetworkStatusWidget> createState() => _NetworkStatusWidgetState();
}

class _NetworkStatusWidgetState extends State<NetworkStatusWidget> {
  final NetworkDetectionManager _networkManager = NetworkDetectionManager.instance;

  StreamSubscription<NetworkStatus>? _networkStatusSubscription;
  StreamSubscription<NetworkQuality>? _networkQualitySubscription;

  NetworkStatus? _currentStatus;
  NetworkQuality? _currentQuality;

  @override
  void initState() {
    super.initState();
    _initializeNetworkListeners();
  }

  void _initializeNetworkListeners() {
    _networkStatusSubscription = _networkManager.networkStatusStream.listen(
      (status) {
        setState(() {
          _currentStatus = status;
        });
        widget.onNetworkChanged?.call();
      },
    );

    _networkQualitySubscription = _networkManager.networkQualityStream.listen(
      (quality) {
        setState(() {
          _currentQuality = quality;
        });
      },
    );
  }

  @override
  void dispose() {
    _networkStatusSubscription?.cancel();
    _networkQualitySubscription?.cancel();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Stack(
      children: [
        widget.child,
        if (widget.showNetworkIndicator && _currentStatus != null)
          _buildNetworkIndicator(),
        if (_currentStatus?.isConnected == false)
          _buildOfflineOverlay(),
      ],
    );
  }

  Widget _buildNetworkIndicator() {
    return Positioned(
      top: MediaQuery.of(context).padding.top + 10,
      right: 16,
      child: Container(
        padding: const EdgeInsets.symmetric(horizontal: 8, vertical: 4),
        decoration: BoxDecoration(
          color: _currentQuality?.color.withOpacity(0.8) ?? Colors.grey,
          borderRadius: BorderRadius.circular(12),
        ),
        child: Row(
          mainAxisSize: MainAxisSize.min,
          children: [
            Icon(
              _getConnectionIcon(),
              size: 16,
              color: Colors.white,
            ),
            const SizedBox(width: 4),
            Text(
              _currentStatus?.connectionType ?? 'Unknown',
              style: const TextStyle(
                color: Colors.white,
                fontSize: 12,
                fontWeight: FontWeight.w500,
              ),
            ),
          ],
        ),
      ),
    );
  }

  Widget _buildOfflineOverlay() {
    return Positioned.fill(
      child: Container(
        color: Colors.black54,
        child: Center(
          child: Card(
            margin: const EdgeInsets.all(32),
            child: Padding(
              padding: const EdgeInsets.all(24),
              child: Column(
                mainAxisSize: MainAxisSize.min,
                children: [
                  const Icon(
                    Icons.wifi_off,
                    size: 64,
                    color: Colors.red,
                  ),
                  const SizedBox(height: 16),
                  const Text(
                    '网络连接已断开',
                    style: TextStyle(
                      fontSize: 18,
                      fontWeight: FontWeight.bold,
                    ),
                  ),
                  const SizedBox(height: 8),
                  const Text(
                    '请检查您的网络连接并重试',
                    textAlign: TextAlign.center,
                    style: TextStyle(
                      color: Colors.grey,
                    ),
                  ),
                  const SizedBox(height: 16),
                  ElevatedButton(
                    onPressed: () {
                      _networkManager.refreshNetworkStatus();
                    },
                    child: const Text('重试'),
                  ),
                ],
              ),
            ),
          ),
        ),
      ),
    );
  }

  IconData _getConnectionIcon() {
    switch (_currentStatus?.connectivityResult) {
      case ConnectivityResult.wifi:
        return Icons.wifi;
      case ConnectivityResult.mobile:
        return Icons.signal_cellular_4_bar;
      case ConnectivityResult.ethernet:
        return Icons.ethernet;
      case ConnectivityResult.bluetooth:
        return Icons.bluetooth;
      case ConnectivityResult.vpn:
        return Icons.vpn_lock;
      default:
        return Icons.signal_wifi_off;
    }
  }
}
```
