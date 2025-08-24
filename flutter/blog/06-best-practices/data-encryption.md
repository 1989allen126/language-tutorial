# Flutter 数据加密工具使用详解

本文档详细介绍 Flutter 应用中常用的数据加密工具和实现方案，包括对称加密、非对称加密、哈希算法等。

## 📋 目录

- [加密基础](#加密基础)
- [对称加密](#对称加密)
- [非对称加密](#非对称加密)
- [哈希算法](#哈希算法)
- [数字签名](#数字签名)
- [密钥管理](#密钥管理)
- [实际应用案例](#实际应用案例)
- [最佳实践](#最佳实践)

## 加密基础

### 架构图

```mermaid
graph TB
    subgraph "加密系统架构"
        A[原始数据] --> B[加密算法选择]
        B --> C{加密类型}

        C -->|对称加密| D[AES/ChaCha20]
        C -->|非对称加密| E[RSA/ECC]
        C -->|哈希算法| F[SHA-256/Blake2b]

        D --> G[密钥管理]
        E --> G
        F --> H[数据完整性验证]

        G --> I[安全存储]
        H --> I

        subgraph "密钥管理"
            J[密钥生成]
            K[密钥存储]
            L[密钥轮换]
            M[密钥销毁]
        end

        G --> J
        G --> K
        G --> L
        G --> M

        subgraph "安全存储"
            N[Keychain/Keystore]
            O[Secure Enclave]
            P[Hardware Security]
        end

        I --> N
        I --> O
        I --> P
    end
```

### 基础依赖配置

```yaml
# pubspec.yaml
dependencies:
  flutter:
    sdk: flutter

  # 加密库
  crypto: ^3.0.3
  encrypt: ^5.0.1
  pointycastle: ^3.7.3

  # 安全存储
  flutter_secure_storage: ^9.0.0

  # 密钥管理
  flutter_keychain: ^2.3.0

  # 生物识别
  local_auth: ^2.1.6

  # 随机数生成
  dart:math
  dart:typed_data
  dart:convert
```

### 加密工具管理器

```dart
import 'dart:convert';
import 'dart:math';
import 'dart:typed_data';
import 'package:crypto/crypto.dart';
import 'package:encrypt/encrypt.dart';
import 'package:pointycastle/export.dart';
import 'package:flutter_secure_storage/flutter_secure_storage.dart';

class CryptographyManager {
  static final CryptographyManager _instance = CryptographyManager._internal();
  static CryptographyManager get instance => _instance;

  CryptographyManager._internal();

  final FlutterSecureStorage _secureStorage = const FlutterSecureStorage(
    aOptions: AndroidOptions(
      encryptedSharedPreferences: true,
    ),
    iOptions: IOSOptions(
      accessibility: IOSAccessibility.first_unlock_this_device,
    ),
  );

  final Random _random = Random.secure();

  // 生成安全随机字节
  Uint8List generateRandomBytes(int length) {
    final bytes = Uint8List(length);
    for (int i = 0; i < length; i++) {
      bytes[i] = _random.nextInt(256);
    }
    return bytes;
  }

  // 生成安全随机字符串
  String generateRandomString(int length) {
    const chars = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789';
    return String.fromCharCodes(
      Iterable.generate(
        length,
        (_) => chars.codeUnitAt(_random.nextInt(chars.length)),
      ),
    );
  }

  // 安全存储密钥
  Future<void> storeKey(String keyId, String key) async {
    await _secureStorage.write(key: keyId, value: key);
  }

  // 读取密钥
  Future<String?> getKey(String keyId) async {
    return await _secureStorage.read(key: keyId);
  }

  // 删除密钥
  Future<void> deleteKey(String keyId) async {
    await _secureStorage.delete(key: keyId);
  }

  // 清除所有密钥
  Future<void> clearAllKeys() async {
    await _secureStorage.deleteAll();
  }
}
```

## 对称加密

### AES 加密实现

```dart
class AESEncryption {
  static const String _keyId = 'aes_encryption_key';
  static const int _keyLength = 32; // 256 bits
  static const int _ivLength = 16;  // 128 bits

  final CryptographyManager _cryptoManager = CryptographyManager.instance;

  // 生成 AES 密钥
  Future<String> generateKey() async {
    final keyBytes = _cryptoManager.generateRandomBytes(_keyLength);
    final keyBase64 = base64Encode(keyBytes);

    await _cryptoManager.storeKey(_keyId, keyBase64);
    return keyBase64;
  }

  // 获取或生成密钥
  Future<Key> _getOrCreateKey() async {
    String? keyBase64 = await _cryptoManager.getKey(_keyId);

    if (keyBase64 == null) {
      keyBase64 = await generateKey();
    }

    final keyBytes = base64Decode(keyBase64);
    return Key(keyBytes);
  }

  // AES-GCM 加密（推荐）
  Future<EncryptedData> encryptGCM(String plaintext) async {
    try {
      final key = await _getOrCreateKey();
      final iv = IV.fromSecureRandom(_ivLength);

      final encrypter = Encrypter(AES(key, mode: AESMode.gcm));
      final encrypted = encrypter.encrypt(plaintext, iv: iv);

      return EncryptedData(
        ciphertext: encrypted.base64,
        iv: iv.base64,
        tag: encrypted.bytes.sublist(encrypted.bytes.length - 16),
        algorithm: 'AES-GCM',
      );
    } catch (e) {
      throw EncryptionException('AES-GCM encryption failed: $e');
    }
  }

  // AES-GCM 解密
  Future<String> decryptGCM(EncryptedData encryptedData) async {
    try {
      final key = await _getOrCreateKey();
      final iv = IV.fromBase64(encryptedData.iv);

      final encrypter = Encrypter(AES(key, mode: AESMode.gcm));
      final encrypted = Encrypted.fromBase64(encryptedData.ciphertext);

      return encrypter.decrypt(encrypted, iv: iv);
    } catch (e) {
      throw DecryptionException('AES-GCM decryption failed: $e');
    }
  }

  // AES-CBC 加密（兼容性）
  Future<EncryptedData> encryptCBC(String plaintext) async {
    try {
      final key = await _getOrCreateKey();
      final iv = IV.fromSecureRandom(_ivLength);

      final encrypter = Encrypter(AES(key, mode: AESMode.cbc));
      final encrypted = encrypter.encrypt(plaintext, iv: iv);

      return EncryptedData(
        ciphertext: encrypted.base64,
        iv: iv.base64,
        algorithm: 'AES-CBC',
      );
    } catch (e) {
      throw EncryptionException('AES-CBC encryption failed: $e');
    }
  }

  // AES-CBC 解密
  Future<String> decryptCBC(EncryptedData encryptedData) async {
    try {
      final key = await _getOrCreateKey();
      final iv = IV.fromBase64(encryptedData.iv);

      final encrypter = Encrypter(AES(key, mode: AESMode.cbc));
      final encrypted = Encrypted.fromBase64(encryptedData.ciphertext);

      return encrypter.decrypt(encrypted, iv: iv);
    } catch (e) {
      throw DecryptionException('AES-CBC decryption failed: $e');
    }
  }

  // 文件加密
  Future<void> encryptFile(String inputPath, String outputPath) async {
    try {
      final file = File(inputPath);
      final bytes = await file.readAsBytes();

      final plaintext = base64Encode(bytes);
      final encryptedData = await encryptGCM(plaintext);

      final outputFile = File(outputPath);
      await outputFile.writeAsString(json.encode(encryptedData.toJson()));
    } catch (e) {
      throw EncryptionException('File encryption failed: $e');
    }
  }

  // 文件解密
  Future<void> decryptFile(String inputPath, String outputPath) async {
    try {
      final file = File(inputPath);
      final encryptedJson = await file.readAsString();

      final encryptedData = EncryptedData.fromJson(
        Map<String, dynamic>.from(json.decode(encryptedJson) as Map),
      );

      final decryptedBase64 = await decryptGCM(encryptedData);
      final decryptedBytes = base64Decode(decryptedBase64);

      final outputFile = File(outputPath);
      await outputFile.writeAsBytes(decryptedBytes);
    } catch (e) {
      throw DecryptionException('File decryption failed: $e');
    }
  }
}

// ChaCha20 加密实现
class ChaCha20Encryption {
  static const String _keyId = 'chacha20_encryption_key';
  static const int _keyLength = 32; // 256 bits
  static const int _nonceLength = 12; // 96 bits

  final CryptographyManager _cryptoManager = CryptographyManager.instance;

  // 生成 ChaCha20 密钥
  Future<String> generateKey() async {
    final keyBytes = _cryptoManager.generateRandomBytes(_keyLength);
    final keyBase64 = base64Encode(keyBytes);

    await _cryptoManager.storeKey(_keyId, keyBase64);
    return keyBase64;
  }

  // ChaCha20-Poly1305 加密
  Future<EncryptedData> encrypt(String plaintext) async {
    try {
      String? keyBase64 = await _cryptoManager.getKey(_keyId);

      if (keyBase64 == null) {
        keyBase64 = await generateKey();
      }

      final keyBytes = base64Decode(keyBase64);
      final nonce = _cryptoManager.generateRandomBytes(_nonceLength);

      // 使用 PointyCastle 实现 ChaCha20-Poly1305
      final cipher = ChaCha20Poly1305();
      final params = AEADParameters(
        KeyParameter(keyBytes),
        128, // tag length in bits
        nonce,
        Uint8List(0), // additional data
      );

      cipher.init(true, params);

      final plaintextBytes = utf8.encode(plaintext);
      final ciphertext = cipher.process(plaintextBytes);

      return EncryptedData(
        ciphertext: base64Encode(ciphertext),
        iv: base64Encode(nonce),
        algorithm: 'ChaCha20-Poly1305',
      );
    } catch (e) {
      throw EncryptionException('ChaCha20 encryption failed: $e');
    }
  }

  // ChaCha20-Poly1305 解密
  Future<String> decrypt(EncryptedData encryptedData) async {
    try {
      final keyBase64 = await _cryptoManager.getKey(_keyId);
      if (keyBase64 == null) {
        throw DecryptionException('Encryption key not found');
      }

      final keyBytes = base64Decode(keyBase64);
      final nonce = base64Decode(encryptedData.iv);
      final ciphertext = base64Decode(encryptedData.ciphertext);

      final cipher = ChaCha20Poly1305();
      final params = AEADParameters(
        KeyParameter(keyBytes),
        128,
        nonce,
        Uint8List(0),
      );

      cipher.init(false, params);

      final decryptedBytes = cipher.process(ciphertext);
      return utf8.decode(decryptedBytes);
    } catch (e) {
      throw DecryptionException('ChaCha20 decryption failed: $e');
    }
  }
}

// 加密数据模型
class EncryptedData {
  final String ciphertext;
  final String iv;
  final String algorithm;
  final Uint8List? tag;
  final DateTime? timestamp;

  EncryptedData({
    required this.ciphertext,
    required this.iv,
    required this.algorithm,
    this.tag,
    this.timestamp,
  });

  Map<String, dynamic> toJson() {
    return {
      'ciphertext': ciphertext,
      'iv': iv,
      'algorithm': algorithm,
      'tag': tag != null ? base64Encode(tag!) : null,
      'timestamp': timestamp?.toIso8601String(),
    };
  }

  factory EncryptedData.fromJson(Map<String, dynamic> json) {
    return EncryptedData(
      ciphertext: json['ciphertext'] as String,
      iv: json['iv'] as String,
      algorithm: json['algorithm'] as String,
      tag: json['tag'] != null ? base64Decode(json['tag'] as String) : null,
      timestamp: json['timestamp'] != null
          ? DateTime.parse(json['timestamp'] as String)
          : null,
    );
  }
}
```

## 非对称加密

### RSA 加密实现

```dart
class RSAEncryption {
  static const String _publicKeyId = 'rsa_public_key';
  static const String _privateKeyId = 'rsa_private_key';
  static const int _keySize = 2048; // bits

  final CryptographyManager _cryptoManager = CryptographyManager.instance;

  // 生成 RSA 密钥对
  Future<RSAKeyPair> generateKeyPair() async {
    final keyGen = RSAKeyGenerator();

    final params = RSAKeyGeneratorParameters(
      BigInt.parse('65537'), // public exponent
      _keySize,
      64, // certainty for prime generation
    );

    keyGen.init(ParametersWithRandom(
      params,
      SecureRandom('Fortuna')..seed(
        KeyParameter(_cryptoManager.generateRandomBytes(32)),
      ),
    ));

    final keyPair = keyGen.generateKeyPair();

    final publicKey = keyPair.publicKey as RSAPublicKey;
    final privateKey = keyPair.privateKey as RSAPrivateKey;

    // 存储密钥
    await _storeKeyPair(publicKey, privateKey);

    return RSAKeyPair(publicKey: publicKey, privateKey: privateKey);
  }

  // 存储密钥对
  Future<void> _storeKeyPair(RSAPublicKey publicKey, RSAPrivateKey privateKey) async {
    final publicKeyPem = _encodePublicKeyToPem(publicKey);
    final privateKeyPem = _encodePrivateKeyToPem(privateKey);

    await _cryptoManager.storeKey(_publicKeyId, publicKeyPem);
    await _cryptoManager.storeKey(_privateKeyId, privateKeyPem);
  }

  // 获取公钥
  Future<RSAPublicKey?> getPublicKey() async {
    final publicKeyPem = await _cryptoManager.getKey(_publicKeyId);
    if (publicKeyPem == null) return null;

    return _parsePublicKeyFromPem(publicKeyPem);
  }

  // 获取私钥
  Future<RSAPrivateKey?> getPrivateKey() async {
    final privateKeyPem = await _cryptoManager.getKey(_privateKeyId);
    if (privateKeyPem == null) return null;

    return _parsePrivateKeyFromPem(privateKeyPem);
  }

  // RSA 加密
  Future<String> encrypt(String plaintext, {RSAPublicKey? publicKey}) async {
    try {
      publicKey ??= await getPublicKey();
      if (publicKey == null) {
        await generateKeyPair();
        publicKey = await getPublicKey();
      }

      final cipher = OAEPEncoding(RSAEngine());
      cipher.init(
        true,
        PublicKeyParameter<RSAPublicKey>(publicKey!),
      );

      final plaintextBytes = utf8.encode(plaintext);
      final ciphertext = cipher.process(plaintextBytes);

      return base64Encode(ciphertext);
    } catch (e) {
      throw EncryptionException('RSA encryption failed: $e');
    }
  }

  // RSA 解密
  Future<String> decrypt(String ciphertext) async {
    try {
      final privateKey = await getPrivateKey();
      if (privateKey == null) {
        throw DecryptionException('Private key not found');
      }

      final cipher = OAEPEncoding(RSAEngine());
      cipher.init(
        false,
        PrivateKeyParameter<RSAPrivateKey>(privateKey),
      );

      final ciphertextBytes = base64Decode(ciphertext);
      final decryptedBytes = cipher.process(ciphertextBytes);

      return utf8.decode(decryptedBytes);
    } catch (e) {
      throw DecryptionException('RSA decryption failed: $e');
    }
  }

  // 混合加密（RSA + AES）
  Future<HybridEncryptedData> hybridEncrypt(String plaintext) async {
    try {
      // 生成随机 AES 密钥
      final aesKey = _cryptoManager.generateRandomBytes(32);
      final iv = _cryptoManager.generateRandomBytes(16);

      // 使用 AES 加密数据
      final aesEncrypter = Encrypter(AES(Key(aesKey)));
      final encryptedData = aesEncrypter.encrypt(plaintext, iv: IV(iv));

      // 使用 RSA 加密 AES 密钥
      final encryptedKey = await encrypt(base64Encode(aesKey));

      return HybridEncryptedData(
        encryptedData: encryptedData.base64,
        encryptedKey: encryptedKey,
        iv: base64Encode(iv),
      );
    } catch (e) {
      throw EncryptionException('Hybrid encryption failed: $e');
    }
  }

  // 混合解密
  Future<String> hybridDecrypt(HybridEncryptedData hybridData) async {
    try {
      // 使用 RSA 解密 AES 密钥
      final aesKeyBase64 = await decrypt(hybridData.encryptedKey);
      final aesKey = base64Decode(aesKeyBase64);

      // 使用 AES 解密数据
      final aesEncrypter = Encrypter(AES(Key(aesKey)));
      final encrypted = Encrypted.fromBase64(hybridData.encryptedData);
      final iv = IV.fromBase64(hybridData.iv);

      return aesEncrypter.decrypt(encrypted, iv: iv);
    } catch (e) {
      throw DecryptionException('Hybrid decryption failed: $e');
    }
  }

  // PEM 编码/解码方法
  String _encodePublicKeyToPem(RSAPublicKey publicKey) {
    final algorithmSeq = ASN1Sequence();
    final algorithmAsn1Obj = ASN1Object.fromBytes(Uint8List.fromList([0x30, 0x0d, 0x06, 0x09, 0x2a, 0x86, 0x48, 0x86, 0xf7, 0x0d, 0x01, 0x01, 0x01, 0x05, 0x00]));
    algorithmSeq.add(algorithmAsn1Obj);

    final publicKeySeq = ASN1Sequence();
    publicKeySeq.add(ASN1Integer(publicKey.modulus!));
    publicKeySeq.add(ASN1Integer(publicKey.exponent!));
    final publicKeySeqBitString = ASN1BitString(publicKeySeq.encodedBytes);

    final topLevelSeq = ASN1Sequence();
    topLevelSeq.add(algorithmSeq);
    topLevelSeq.add(publicKeySeqBitString);

    final dataBase64 = base64Encode(topLevelSeq.encodedBytes);
    return '-----BEGIN PUBLIC KEY-----\n$dataBase64\n-----END PUBLIC KEY-----';
  }

  String _encodePrivateKeyToPem(RSAPrivateKey privateKey) {
    final version = ASN1Integer(BigInt.from(0));
    final modulus = ASN1Integer(privateKey.modulus!);
    final publicExponent = ASN1Integer(privateKey.exponent!);
    final privateExponent = ASN1Integer(privateKey.privateExponent!);
    final p = ASN1Integer(privateKey.p!);
    final q = ASN1Integer(privateKey.q!);
    final dP = ASN1Integer(privateKey.privateExponent! % (privateKey.p! - BigInt.one));
    final dQ = ASN1Integer(privateKey.privateExponent! % (privateKey.q! - BigInt.one));
    final qInv = ASN1Integer(privateKey.q!.modInverse(privateKey.p!));

    final seq = ASN1Sequence();
    seq.add(version);
    seq.add(modulus);
    seq.add(publicExponent);
    seq.add(privateExponent);
    seq.add(p);
    seq.add(q);
    seq.add(dP);
    seq.add(dQ);
    seq.add(qInv);

    final dataBase64 = base64Encode(seq.encodedBytes);
    return '-----BEGIN PRIVATE KEY-----\n$dataBase64\n-----END PRIVATE KEY-----';
  }

  RSAPublicKey _parsePublicKeyFromPem(String pem) {
    // 实现 PEM 解析逻辑
    throw UnimplementedError('PEM parsing not implemented');
  }

  RSAPrivateKey _parsePrivateKeyFromPem(String pem) {
    // 实现 PEM 解析逻辑
    throw UnimplementedError('PEM parsing not implemented');
  }
}

// RSA 密钥对
class RSAKeyPair {
  final RSAPublicKey publicKey;
  final RSAPrivateKey privateKey;

  RSAKeyPair({
    required this.publicKey,
    required this.privateKey,
  });
}

// 混合加密数据
class HybridEncryptedData {
  final String encryptedData;
  final String encryptedKey;
  final String iv;

  HybridEncryptedData({
    required this.encryptedData,
    required this.encryptedKey,
    required this.iv,
  });

  Map<String, dynamic> toJson() {
    return {
      'encryptedData': encryptedData,
      'encryptedKey': encryptedKey,
      'iv': iv,
    };
  }

  factory HybridEncryptedData.fromJson(Map<String, dynamic> json) {
    return HybridEncryptedData(
      encryptedData: json['encryptedData'] as String,
      encryptedKey: json['encryptedKey'] as String,
      iv: json['iv'] as String,
    );
  }
}
```

## 哈希算法

### 哈希工具实现

```dart
class HashingUtils {
  // SHA-256 哈希
  static String sha256Hash(String input) {
    final bytes = utf8.encode(input);
    final digest = sha256.convert(bytes);
    return digest.toString();
  }

  // SHA-512 哈希
  static String sha512Hash(String input) {
    final bytes = utf8.encode(input);
    final digest = sha512.convert(bytes);
    return digest.toString();
  }

  // HMAC-SHA256
  static String hmacSha256(String message, String key) {
    final keyBytes = utf8.encode(key);
    final messageBytes = utf8.encode(message);

    final hmac = Hmac(sha256, keyBytes);
    final digest = hmac.convert(messageBytes);

    return digest.toString();
  }

  // PBKDF2 密钥派生
  static Uint8List pbkdf2(String password, Uint8List salt, int iterations, int keyLength) {
    final pbkdf2 = PBKDF2KeyDerivator(HMac(SHA256Digest(), 64));

    pbkdf2.init(Pbkdf2Parameters(salt, iterations, keyLength));

    return pbkdf2.process(utf8.encode(password));
  }

  // Argon2 密钥派生（推荐用于密码哈希）
  static Future<String> argon2Hash(String password, {String? salt}) async {
    salt ??= CryptographyManager.instance.generateRandomString(16);

    // 注意：这里需要使用 argon2 包或原生实现
    // 这是一个简化的示例
    final saltBytes = utf8.encode(salt);
    final passwordBytes = utf8.encode(password);

    // 使用 PBKDF2 作为 Argon2 的替代（实际项目中应使用真正的 Argon2）
    final hash = pbkdf2(password, saltBytes, 100000, 32);

    return '\$argon2id\$v=19\$m=65536,t=3,p=4\$${base64Encode(saltBytes)}\$${base64Encode(hash)}';
  }

  // 验证 Argon2 哈希
  static Future<bool> verifyArgon2Hash(String password, String hash) async {
    try {
      // 解析哈希字符串
      final parts = hash.split('\$');
      if (parts.length != 6) return false;

      final saltBase64 = parts[4];
      final hashBase64 = parts[5];

      final salt = base64Decode(saltBase64);
      final expectedHash = base64Decode(hashBase64);

      // 重新计算哈希
      final computedHash = pbkdf2(password, salt, 100000, 32);

      // 常量时间比较
      return _constantTimeEquals(expectedHash, computedHash);
    } catch (e) {
      return false;
    }
  }

  // 常量时间比较（防止时序攻击）
  static bool _constantTimeEquals(Uint8List a, Uint8List b) {
    if (a.length != b.length) return false;

    int result = 0;
    for (int i = 0; i < a.length; i++) {
      result |= a[i] ^ b[i];
    }

    return result == 0;
  }

  // 文件哈希
  static Future<String> hashFile(String filePath, {String algorithm = 'sha256'}) async {
    final file = File(filePath);
    final bytes = await file.readAsBytes();

    switch (algorithm.toLowerCase()) {
      case 'sha256':
        return sha256.convert(bytes).toString();
      case 'sha512':
        return sha512.convert(bytes).toString();
      case 'md5':
        return md5.convert(bytes).toString();
      default:
        throw ArgumentError('Unsupported hash algorithm: $algorithm');
    }
  }

  // 流式哈希（大文件）
  static Future<String> hashFileStream(String filePath, {String algorithm = 'sha256'}) async {
    final file = File(filePath);
    final stream = file.openRead();

    late Digest digest;
    switch (algorithm.toLowerCase()) {
      case 'sha256':
        digest = sha256;
        break;
      case 'sha512':
        digest = sha512;
        break;
      case 'md5':
        digest = md5;
        break;
      default:
        throw ArgumentError('Unsupported hash algorithm: $algorithm');
    }

    final output = AccumulatorSink<Digest>();
    final input = digest.startChunkedConversion(output);

    await for (final chunk in stream) {
      input.add(chunk);
    }

    input.close();
    return output.events.single.toString();
  }
}
```

## 数字签名

### 数字签名实现

```dart
class DigitalSignature {
  static const String _signingKeyId = 'digital_signature_key';

  final CryptographyManager _cryptoManager = CryptographyManager.instance;

  // 生成签名密钥对
  Future<AsymmetricKeyPair> generateSigningKeyPair() async {
    final keyGen = RSAKeyGenerator();

    final params = RSAKeyGeneratorParameters(
      BigInt.parse('65537'),
      2048,
      64,
    );

    keyGen.init(ParametersWithRandom(
      params,
      SecureRandom('Fortuna')..seed(
        KeyParameter(_cryptoManager.generateRandomBytes(32)),
      ),
    ));

    final keyPair = keyGen.generateKeyPair();

    // 存储密钥对
    await _storeSigningKeyPair(keyPair);

    return keyPair;
  }

  // 存储签名密钥对
  Future<void> _storeSigningKeyPair(AsymmetricKeyPair keyPair) async {
    final publicKey = keyPair.publicKey as RSAPublicKey;
    final privateKey = keyPair.privateKey as RSAPrivateKey;

    // 这里应该使用适当的序列化方法
    // 简化示例
    await _cryptoManager.storeKey(
      '${_signingKeyId}_public',
      publicKey.modulus.toString(),
    );
    await _cryptoManager.storeKey(
      '${_signingKeyId}_private',
      privateKey.privateExponent.toString(),
    );
  }

  // RSA-PSS 签名
  Future<String> signRSAPSS(String message) async {
    try {
      final privateKeyData = await _cryptoManager.getKey('${_signingKeyId}_private');
      if (privateKeyData == null) {
        await generateSigningKeyPair();
      }

      // 这里需要重新构造私钥
      // 简化示例，实际实现需要完整的密钥重构

      final messageBytes = utf8.encode(message);
      final messageHash = sha256.convert(messageBytes).bytes;

      final signer = PSSSigner(RSAEngine(), SHA256Digest(), SHA256Digest());
      // 初始化签名器
      // signer.init(true, privateKeyParam);

      // 生成签名
      // final signature = signer.generateSignature(messageHash);

      // 返回 Base64 编码的签名
      // return base64Encode(signature);

      // 简化返回
      return base64Encode(messageHash);
    } catch (e) {
      throw SignatureException('RSA-PSS signing failed: $e');
    }
  }

  // RSA-PSS 验证
  Future<bool> verifyRSAPSS(String message, String signature) async {
    try {
      final publicKeyData = await _cryptoManager.getKey('${_signingKeyId}_public');
      if (publicKeyData == null) {
        return false;
      }

      final messageBytes = utf8.encode(message);
      final messageHash = sha256.convert(messageBytes).bytes;
      final signatureBytes = base64Decode(signature);

      final verifier = PSSSigner(RSAEngine(), SHA256Digest(), SHA256Digest());
      // 初始化验证器
      // verifier.init(false, publicKeyParam);

      // 验证签名
      // return verifier.verifySignature(messageHash, signatureBytes);

      // 简化验证
      return _constantTimeEquals(messageHash, signatureBytes);
    } catch (e) {
      return false;
    }
  }

  // ECDSA 签名（椭圆曲线）
  Future<String> signECDSA(String message) async {
    try {
      // 生成 ECDSA 密钥对
      final keyGen = ECKeyGenerator();
      final ecDomainParams = ECDomainParameters('secp256r1');

      keyGen.init(ParametersWithRandom(
        ECKeyGeneratorParameters(ecDomainParams),
        SecureRandom('Fortuna')..seed(
          KeyParameter(_cryptoManager.generateRandomBytes(32)),
        ),
      ));

      final keyPair = keyGen.generateKeyPair();
      final privateKey = keyPair.privateKey as ECPrivateKey;

      final messageBytes = utf8.encode(message);
      final messageHash = sha256.convert(messageBytes).bytes;

      final signer = ECDSASigner(SHA256Digest());
      signer.init(true, PrivateKeyParameter(privateKey));

      final signature = signer.generateSignature(messageHash);

      // 编码签名（DER 格式）
      final r = (signature as ECSignature).r;
      final s = signature.s;

      final rBytes = _bigIntToBytes(r);
      final sBytes = _bigIntToBytes(s);

      final signatureBytes = Uint8List(rBytes.length + sBytes.length);
      signatureBytes.setRange(0, rBytes.length, rBytes);
      signatureBytes.setRange(rBytes.length, signatureBytes.length, sBytes);

      return base64Encode(signatureBytes);
    } catch (e) {
      throw SignatureException('ECDSA signing failed: $e');
    }
  }

  // 消息认证码 (HMAC)
  String generateHMAC(String message, String key) {
    return HashingUtils.hmacSha256(message, key);
  }

  // 验证 HMAC
  bool verifyHMAC(String message, String key, String expectedHmac) {
    final computedHmac = generateHMAC(message, key);
    return _constantTimeEquals(
      utf8.encode(computedHmac),
      utf8.encode(expectedHmac),
    );
  }

  // 时间戳签名
  Future<TimestampedSignature> signWithTimestamp(String message) async {
    final timestamp = DateTime.now();
    final messageWithTimestamp = '$message|${timestamp.toIso8601String()}';

    final signature = await signRSAPSS(messageWithTimestamp);

    return TimestampedSignature(
      message: message,
      signature: signature,
      timestamp: timestamp,
    );
  }

  // 验证时间戳签名
  Future<bool> verifyTimestampedSignature(
    TimestampedSignature timestampedSignature,
    {Duration? maxAge}
  ) async {
    // 检查时间戳是否过期
    if (maxAge != null) {
      final age = DateTime.now().difference(timestampedSignature.timestamp);
      if (age > maxAge) {
        return false;
      }
    }

    final messageWithTimestamp =
        '${timestampedSignature.message}|${timestampedSignature.timestamp.toIso8601String()}';

    return await verifyRSAPSS(messageWithTimestamp, timestampedSignature.signature);
  }

  Uint8List _bigIntToBytes(BigInt bigInt) {
    final bytes = <int>[];
    var value = bigInt;

    while (value > BigInt.zero) {
      bytes.insert(0, (value & BigInt.from(0xff)).toInt());
      value = value >> 8;
    }

    return Uint8List.fromList(bytes);
  }

  bool _constantTimeEquals(List<int> a, List<int> b) {
    if (a.length != b.length) return false;

    int result = 0;
    for (int i = 0; i < a.length; i++) {
      result |= a[i] ^ b[i];
    }

    return result == 0;
  }
}

// 时间戳签名
class TimestampedSignature {
  final String message;
  final String signature;
  final DateTime timestamp;

  TimestampedSignature({
    required this.message,
    required this.signature,
    required this.timestamp,
  });

  Map<String, dynamic> toJson() {
    return {
      'message': message,
      'signature': signature,
      'timestamp': timestamp.toIso8601String(),
    };
  }

  factory TimestampedSignature.fromJson(Map<String, dynamic> json) {
    return TimestampedSignature(
      message: json['message'] as String,
      signature: json['signature'] as String,
      timestamp: DateTime.parse(json['timestamp'] as String),
    );
  }
}

// 签名异常
class SignatureException implements Exception {
  final String message;

  SignatureException(this.message);

  @override
  String toString() => 'SignatureException: $message';
}

// 加密异常
class EncryptionException implements Exception {
  final String message;

  EncryptionException(this.message);

  @override
  String toString() => 'EncryptionException: $message';
}

// 解密异常
class DecryptionException implements Exception {
  final String message;

  DecryptionException(this.message);

  @override
  String toString() => 'DecryptionException: $message';
}
```

## 密钥管理

### 密钥生命周期管理

```dart
class KeyLifecycleManager {
  static const String _keyMetadataPrefix = 'key_metadata_';

  final CryptographyManager _cryptoManager = CryptographyManager.instance;

  // 密钥元数据
  class KeyMetadata {
    final String keyId;
    final String algorithm;
    final DateTime createdAt;
    final DateTime? expiresAt;
    final int usageCount;
    final int maxUsageCount;
    final KeyStatus status;

    KeyMetadata({
      required this.keyId,
      required this.algorithm,
      required this.createdAt,
      this.expiresAt,
      this.usageCount = 0,
      this.maxUsageCount = -1, // -1 表示无限制
      this.status = KeyStatus.active,
    });

    Map<String, dynamic> toJson() {
      return {
        'keyId': keyId,
        'algorithm': algorithm,
        'createdAt': createdAt.toIso8601String(),
        'expiresAt': expiresAt?.toIso8601String(),
        'usageCount': usageCount,
        'maxUsageCount': maxUsageCount,
        'status': status.toString(),
      };
    }

    factory KeyMetadata.fromJson(Map<String, dynamic> json) {
      return KeyMetadata(
        keyId: json['keyId'] as String,
        algorithm: json['algorithm'] as String,
        createdAt: DateTime.parse(json['createdAt'] as String),
        expiresAt: json['expiresAt'] != null
            ? DateTime.parse(json['expiresAt'] as String)
            : null,
        usageCount: json['usageCount'] as int,
        maxUsageCount: json['maxUsageCount'] as int,
        status: KeyStatus.values.firstWhere(
          (e) => e.toString() == json['status'],
          orElse: () => KeyStatus.active,
        ),
      );
    }

    KeyMetadata copyWith({
      String? keyId,
      String? algorithm,
      DateTime? createdAt,
      DateTime? expiresAt,
      int? usageCount,
      int? maxUsageCount,
      KeyStatus? status,
    }) {
      return KeyMetadata(
        keyId: keyId ?? this.keyId,
        algorithm: algorithm ?? this.algorithm,
        createdAt: createdAt ?? this.createdAt,
        expiresAt: expiresAt ?? this.expiresAt,
        usageCount: usageCount ?? this.usageCount,
        maxUsageCount: maxUsageCount ?? this.maxUsageCount,
        status: status ?? this.status,
      );
    }
  }

  enum KeyStatus {
    active,
    expired,
    revoked,
    compromised,
  }

  // 创建密钥
  Future<String> createKey({
    required String algorithm,
    Duration? validityPeriod,
    int? maxUsageCount,
  }) async {
    final keyId = _generateKeyId();
    final now = DateTime.now();

    // 生成实际密钥
    String keyData;
    switch (algorithm.toLowerCase()) {
      case 'aes-256':
        keyData = base64Encode(_cryptoManager.generateRandomBytes(32));
        break;
      case 'chacha20':
        keyData = base64Encode(_cryptoManager.generateRandomBytes(32));
        break;
      case 'hmac-sha256':
        keyData = base64Encode(_cryptoManager.generateRandomBytes(32));
        break;
      default:
        throw ArgumentError('Unsupported algorithm: $algorithm');
    }

    // 存储密钥
    await _cryptoManager.storeKey(keyId, keyData);

    // 创建元数据
    final metadata = KeyMetadata(
      keyId: keyId,
      algorithm: algorithm,
      createdAt: now,
      expiresAt: validityPeriod != null ? now.add(validityPeriod) : null,
      maxUsageCount: maxUsageCount ?? -1,
    );

    await _storeKeyMetadata(metadata);

    return keyId;
  }

  // 获取密钥
  Future<String?> getKey(String keyId) async {
    final metadata = await _getKeyMetadata(keyId);
    if (metadata == null) return null;

    // 检查密钥状态
    if (!_isKeyValid(metadata)) {
      return null;
    }

    // 更新使用计数
    await _incrementUsageCount(keyId);

    return await _cryptoManager.getKey(keyId);
  }

  // 轮换密钥
  Future<String> rotateKey(String oldKeyId) async {
    final oldMetadata = await _getKeyMetadata(oldKeyId);
    if (oldMetadata == null) {
      throw ArgumentError('Key not found: $oldKeyId');
    }

    // 创建新密钥
    final newKeyId = await createKey(
      algorithm: oldMetadata.algorithm,
      validityPeriod: oldMetadata.expiresAt != null
          ? oldMetadata.expiresAt!.difference(oldMetadata.createdAt)
          : null,
      maxUsageCount: oldMetadata.maxUsageCount > 0
          ? oldMetadata.maxUsageCount
          : null,
    );

    // 标记旧密钥为已撤销
    await _updateKeyStatus(oldKeyId, KeyStatus.revoked);

    return newKeyId;
  }

  // 撤销密钥
  Future<void> revokeKey(String keyId) async {
    await _updateKeyStatus(keyId, KeyStatus.revoked);
  }

  // 标记密钥为已泄露
  Future<void> markKeyCompromised(String keyId) async {
    await _updateKeyStatus(keyId, KeyStatus.compromised);
  }

  // 清理过期密钥
  Future<void> cleanupExpiredKeys() async {
    final allKeys = await _getAllKeyMetadata();
    final now = DateTime.now();

    for (final metadata in allKeys) {
      if (metadata.expiresAt != null && metadata.expiresAt!.isBefore(now)) {
        await _updateKeyStatus(metadata.keyId, KeyStatus.expired);

        // 可选：删除过期密钥
        // await _cryptoManager.deleteKey(metadata.keyId);
        // await _deleteKeyMetadata(metadata.keyId);
      }
    }
  }

  // 获取密钥统计信息
  Future<KeyStatistics> getKeyStatistics() async {
    final allKeys = await _getAllKeyMetadata();

    int activeCount = 0;
    int expiredCount = 0;
    int revokedCount = 0;
    int compromisedCount = 0;

    for (final metadata in allKeys) {
      switch (metadata.status) {
        case KeyStatus.active:
          activeCount++;
          break;
        case KeyStatus.expired:
          expiredCount++;
          break;
        case KeyStatus.revoked:
          revokedCount++;
          break;
        case KeyStatus.compromised:
          compromisedCount++;
          break;
      }
    }

    return KeyStatistics(
      totalKeys: allKeys.length,
      activeKeys: activeCount,
      expiredKeys: expiredCount,
      revokedKeys: revokedCount,
      compromisedKeys: compromisedCount,
    );
  }

  // 私有方法
  String _generateKeyId() {
    return 'key_${DateTime.now().millisecondsSinceEpoch}_${_cryptoManager.generateRandomString(8)}';
  }

  bool _isKeyValid(KeyMetadata metadata) {
    final now = DateTime.now();

    // 检查状态
    if (metadata.status != KeyStatus.active) {
      return false;
    }

    // 检查过期时间
    if (metadata.expiresAt != null && metadata.expiresAt!.isBefore(now)) {
      return false;
    }

    // 检查使用次数
    if (metadata.maxUsageCount > 0 && metadata.usageCount >= metadata.maxUsageCount) {
      return false;
    }

    return true;
  }

  Future<void> _storeKeyMetadata(KeyMetadata metadata) async {
    final metadataJson = json.encode(metadata.toJson());
    await _cryptoManager.storeKey('$_keyMetadataPrefix${metadata.keyId}', metadataJson);
  }

  Future<KeyMetadata?> _getKeyMetadata(String keyId) async {
    final metadataJson = await _cryptoManager.getKey('$_keyMetadataPrefix$keyId');
    if (metadataJson == null) return null;

    final metadataMap = Map<String, dynamic>.from(json.decode(metadataJson) as Map);
    return KeyMetadata.fromJson(metadataMap);
  }

  Future<void> _updateKeyStatus(String keyId, KeyStatus status) async {
    final metadata = await _getKeyMetadata(keyId);
    if (metadata == null) return;

    final updatedMetadata = metadata.copyWith(status: status);
    await _storeKeyMetadata(updatedMetadata);
  }

  Future<void> _incrementUsageCount(String keyId) async {
    final metadata = await _getKeyMetadata(keyId);
    if (metadata == null) return;

    final updatedMetadata = metadata.copyWith(
      usageCount: metadata.usageCount + 1,
    );
    await _storeKeyMetadata(updatedMetadata);
  }

  Future<List<KeyMetadata>> _getAllKeyMetadata() async {
    // 这里需要实现获取所有密钥元数据的逻辑
    // 由于 FlutterSecureStorage 不支持列出所有键，
    // 实际实现中可能需要维护一个密钥索引
    return [];
  }

  Future<void> _deleteKeyMetadata(String keyId) async {
    await _cryptoManager.deleteKey('$_keyMetadataPrefix$keyId');
  }
}

// 密钥统计信息
class KeyStatistics {
  final int totalKeys;
  final int activeKeys;
  final int expiredKeys;
  final int revokedKeys;
  final int compromisedKeys;

  KeyStatistics({
    required this.totalKeys,
    required this.activeKeys,
    required this.expiredKeys,
    required this.revokedKeys,
    required this.compromisedKeys,
  });

  Map<String, dynamic> toJson() {
    return {
      'totalKeys': totalKeys,
      'activeKeys': activeKeys,
      'expiredKeys': expiredKeys,
      'revokedKeys': revokedKeys,
      'compromisedKeys': compromisedKeys,
    };
  }
}
```

## 实际应用案例

### 用户数据加密服务

```dart
class UserDataEncryptionService {
  final AESEncryption _aesEncryption = AESEncryption();
  final RSAEncryption _rsaEncryption = RSAEncryption();
  final KeyLifecycleManager _keyManager = KeyLifecycleManager();

  // 加密用户敏感信息
  Future<EncryptedUserData> encryptUserData(UserData userData) async {
    try {
      // 为每个用户创建专用密钥
      final userKeyId = await _keyManager.createKey(
        algorithm: 'aes-256',
        validityPeriod: const Duration(days: 365),
      );

      // 加密个人信息
      final encryptedPersonalInfo = await _aesEncryption.encryptGCM(
        json.encode(userData.personalInfo.toJson()),
      );

      // 加密财务信息（使用更强的加密）
      final encryptedFinancialInfo = await _rsaEncryption.hybridEncrypt(
        json.encode(userData.financialInfo.toJson()),
      );

      // 加密健康信息
      final encryptedHealthInfo = await _aesEncryption.encryptGCM(
        json.encode(userData.healthInfo.toJson()),
      );

      return EncryptedUserData(
        userId: userData.userId,
        encryptedPersonalInfo: encryptedPersonalInfo,
        encryptedFinancialInfo: encryptedFinancialInfo,
        encryptedHealthInfo: encryptedHealthInfo,
        keyId: userKeyId,
        encryptedAt: DateTime.now(),
      );
    } catch (e) {
      throw EncryptionException('Failed to encrypt user data: $e');
    }
  }

  // 解密用户数据
  Future<UserData> decryptUserData(EncryptedUserData encryptedData) async {
    try {
      // 解密个人信息
      final personalInfoJson = await _aesEncryption.decryptGCM(
        encryptedData.encryptedPersonalInfo,
      );
      final personalInfo = PersonalInfo.fromJson(
        Map<String, dynamic>.from(json.decode(personalInfoJson) as Map),
      );

      // 解密财务信息
      final financialInfoJson = await _rsaEncryption.hybridDecrypt(
        encryptedData.encryptedFinancialInfo,
      );
      final financialInfo = FinancialInfo.fromJson(
        Map<String, dynamic>.from(json.decode(financialInfoJson) as Map),
      );

      // 解密健康信息
      final healthInfoJson = await _aesEncryption.decryptGCM(
        encryptedData.encryptedHealthInfo,
      );
      final healthInfo = HealthInfo.fromJson(
        Map<String, dynamic>.from(json.decode(healthInfoJson) as Map),
      );

      return UserData(
        userId: encryptedData.userId,
        personalInfo: personalInfo,
        financialInfo: financialInfo,
        healthInfo: healthInfo,
      );
    } catch (e) {
      throw DecryptionException('Failed to decrypt user data: $e');
    }
  }

  // 数据脱敏
  UserData maskSensitiveData(UserData userData) {
    return UserData(
      userId: userData.userId,
      personalInfo: PersonalInfo(
        name: _maskName(userData.personalInfo.name),
        email: _maskEmail(userData.personalInfo.email),
        phone: _maskPhone(userData.personalInfo.phone),
        address: _maskAddress(userData.personalInfo.address),
      ),
      financialInfo: FinancialInfo(
        creditCardNumber: _maskCreditCard(userData.financialInfo.creditCardNumber),
        bankAccount: _maskBankAccount(userData.financialInfo.bankAccount),
        income: userData.financialInfo.income, // 保留收入信息用于分析
      ),
      healthInfo: HealthInfo(
        medicalHistory: ['***'], // 完全隐藏医疗历史
        allergies: userData.healthInfo.allergies.map((a) => '***').toList(),
        medications: ['***'],
      ),
    );
  }

  String _maskName(String name) {
    if (name.length <= 2) return '*' * name.length;
    return '${name[0]}${'*' * (name.length - 2)}${name[name.length - 1]}';
  }

  String _maskEmail(String email) {
    final parts = email.split('@');
    if (parts.length != 2) return '***@***.***';

    final username = parts[0];
    final domain = parts[1];

    final maskedUsername = username.length > 2
        ? '${username.substring(0, 2)}${'*' * (username.length - 2)}'
        : '*' * username.length;

    return '$maskedUsername@$domain';
  }

  String _maskPhone(String phone) {
    if (phone.length < 4) return '*' * phone.length;
    return '${phone.substring(0, 3)}${'*' * (phone.length - 6)}${phone.substring(phone.length - 3)}';
  }

  String _maskAddress(String address) {
    final words = address.split(' ');
    if (words.length <= 2) return '*** ***';

    return '${words[0]} *** ${words[words.length - 1]}';
  }

  String _maskCreditCard(String cardNumber) {
    if (cardNumber.length < 4) return '*' * cardNumber.length;
    return '**** **** **** ${cardNumber.substring(cardNumber.length - 4)}';
  }

  String _maskBankAccount(String account) {
    if (account.length < 4) return '*' * account.length;
    return '${'*' * (account.length - 4)}${account.substring(account.length - 4)}';
  }
}

// 数据模型
class UserData {
  final String userId;
  final PersonalInfo personalInfo;
  final FinancialInfo financialInfo;
  final HealthInfo healthInfo;

  UserData({
    required this.userId,
    required this.personalInfo,
    required this.financialInfo,
    required this.healthInfo,
  });
}

class PersonalInfo {
  final String name;
  final String email;
  final String phone;
  final String address;

  PersonalInfo({
    required this.name,
    required this.email,
    required this.phone,
    required this.address,
  });

  Map<String, dynamic> toJson() {
    return {
      'name': name,
      'email': email,
      'phone': phone,
      'address': address,
    };
  }

  factory PersonalInfo.fromJson(Map<String, dynamic> json) {
    return PersonalInfo(
      name: json['name'] as String,
      email: json['email'] as String,
      phone: json['phone'] as String,
      address: json['address'] as String,
    );
  }
}

class FinancialInfo {
  final String creditCardNumber;
  final String bankAccount;
  final double income;

  FinancialInfo({
    required this.creditCardNumber,
    required this.bankAccount,
    required this.income,
  });

  Map<String, dynamic> toJson() {
    return {
      'creditCardNumber': creditCardNumber,
      'bankAccount': bankAccount,
      'income': income,
    };
  }

  factory FinancialInfo.fromJson(Map<String, dynamic> json) {
    return FinancialInfo(
      creditCardNumber: json['creditCardNumber'] as String,
      bankAccount: json['bankAccount'] as String,
      income: (json['income'] as num).toDouble(),
    );
  }
}

class HealthInfo {
  final List<String> medicalHistory;
  final List<String> allergies;
  final List<String> medications;

  HealthInfo({
    required this.medicalHistory,
    required this.allergies,
    required this.medications,
  });

  Map<String, dynamic> toJson() {
    return {
      'medicalHistory': medicalHistory,
      'allergies': allergies,
      'medications': medications,
    };
  }

  factory HealthInfo.fromJson(Map<String, dynamic> json) {
    return HealthInfo(
      medicalHistory: List<String>.from(json['medicalHistory'] as List),
      allergies: List<String>.from(json['allergies'] as List),
      medications: List<String>.from(json['medications'] as List),
    );
  }
}

class EncryptedUserData {
  final String userId;
  final EncryptedData encryptedPersonalInfo;
  final HybridEncryptedData encryptedFinancialInfo;
  final EncryptedData encryptedHealthInfo;
  final String keyId;
  final DateTime encryptedAt;

  EncryptedUserData({
    required this.userId,
    required this.encryptedPersonalInfo,
    required this.encryptedFinancialInfo,
    required this.encryptedHealthInfo,
    required this.keyId,
    required this.encryptedAt,
  });
}
```

### 文件加密管理器

```dart
class FileEncryptionManager {
  final AESEncryption _aesEncryption = AESEncryption();
  final ChaCha20Encryption _chaCha20Encryption = ChaCha20Encryption();

  // 加密文件夹
  Future<void> encryptDirectory(String inputDir, String outputDir) async {
    final inputDirectory = Directory(inputDir);
    final outputDirectory = Directory(outputDir);

    if (!await outputDirectory.exists()) {
      await outputDirectory.create(recursive: true);
    }

    await for (final entity in inputDirectory.list(recursive: true)) {
      if (entity is File) {
        final relativePath = path.relative(entity.path, from: inputDir);
        final outputPath = path.join(outputDir, '$relativePath.enc');

        await _encryptSingleFile(entity.path, outputPath);
      }
    }
  }

  // 解密文件夹
  Future<void> decryptDirectory(String inputDir, String outputDir) async {
    final inputDirectory = Directory(inputDir);
    final outputDirectory = Directory(outputDir);

    if (!await outputDirectory.exists()) {
      await outputDirectory.create(recursive: true);
    }

    await for (final entity in inputDirectory.list(recursive: true)) {
      if (entity is File && entity.path.endsWith('.enc')) {
        final relativePath = path.relative(entity.path, from: inputDir);
        final outputPath = path.join(
          outputDir,
          relativePath.substring(0, relativePath.length - 4), // 移除 .enc 后缀
        );

        await _decryptSingleFile(entity.path, outputPath);
      }
    }
  }

  // 加密单个文件
  Future<void> _encryptSingleFile(String inputPath, String outputPath) async {
    final file = File(inputPath);
    final fileSize = await file.length();

    // 根据文件大小选择加密算法
    if (fileSize > 100 * 1024 * 1024) { // 100MB 以上使用 ChaCha20
      await _encryptLargeFile(inputPath, outputPath);
    } else {
      await _aesEncryption.encryptFile(inputPath, outputPath);
    }
  }

  // 解密单个文件
  Future<void> _decryptSingleFile(String inputPath, String outputPath) async {
    try {
      // 首先尝试 AES 解密
      await _aesEncryption.decryptFile(inputPath, outputPath);
    } catch (e) {
      // 如果 AES 失败，尝试 ChaCha20
      await _decryptLargeFile(inputPath, outputPath);
    }
  }

  // 加密大文件（流式处理）
  Future<void> _encryptLargeFile(String inputPath, String outputPath) async {
    final inputFile = File(inputPath);
    final outputFile = File(outputPath);

    final inputStream = inputFile.openRead();
    final outputSink = outputFile.openWrite();

    const chunkSize = 64 * 1024; // 64KB 块
    final buffer = <int>[];

    try {
      await for (final chunk in inputStream) {
        buffer.addAll(chunk);

        while (buffer.length >= chunkSize) {
          final chunkData = buffer.take(chunkSize).toList();
          buffer.removeRange(0, chunkSize);

          final encryptedChunk = await _chaCha20Encryption.encrypt(
            base64Encode(chunkData),
          );

          outputSink.writeln(json.encode(encryptedChunk.toJson()));
        }
      }

      // 处理剩余数据
      if (buffer.isNotEmpty) {
        final encryptedChunk = await _chaCha20Encryption.encrypt(
          base64Encode(buffer),
        );

        outputSink.writeln(json.encode(encryptedChunk.toJson()));
      }
    } finally {
      await outputSink.close();
    }
  }

  // 解密大文件
  Future<void> _decryptLargeFile(String inputPath, String outputPath) async {
    final inputFile = File(inputPath);
    final outputFile = File(outputPath);

    final lines = await inputFile.readAsLines();
    final outputSink = outputFile.openWrite();

    try {
      for (final line in lines) {
        if (line.trim().isEmpty) continue;

        final encryptedData = EncryptedData.fromJson(
          Map<String, dynamic>.from(json.decode(line) as Map),
        );

        final decryptedBase64 = await _chaCha20Encryption.decrypt(encryptedData);
        final decryptedBytes = base64Decode(decryptedBase64);

        outputSink.add(decryptedBytes);
      }
    } finally {
      await outputSink.close();
    }
  }

  // 计算文件完整性
  Future<String> calculateFileIntegrity(String filePath) async {
    return await HashingUtils.hashFileStream(filePath, algorithm: 'sha256');
  }

  // 验证文件完整性
  Future<bool> verifyFileIntegrity(String filePath, String expectedHash) async {
    final actualHash = await calculateFileIntegrity(filePath);
    return actualHash == expectedHash;
  }
}
```

## 最佳实践

### 设计原则

1. **最小权限原则**
   - 只授予必要的加密权限
   - 分离密钥管理和数据处理
   - 实施角色基础的访问控制

2. **深度防御**
   - 多层加密保护
   - 密钥分离存储
   - 定期安全审计

3. **密钥管理**
   - 使用硬件安全模块（HSM）
   - 实施密钥轮换策略
   - 安全的密钥备份和恢复

### 性能优化

```dart
class EncryptionPerformanceOptimizer {
  // 批量加密优化
  static Future<List<EncryptedData>> batchEncrypt(
    List<String> plaintexts,
    AESEncryption encryption,
  ) async {
    final results = <EncryptedData>[];

    // 并行处理小批量
    const batchSize = 10;
    for (int i = 0; i < plaintexts.length; i += batchSize) {
      final batch = plaintexts.skip(i).take(batchSize);

      final futures = batch.map((plaintext) => encryption.encryptGCM(plaintext));
      final batchResults = await Future.wait(futures);

      results.addAll(batchResults);
    }

    return results;
  }

  // 内存优化的流式加密
  static Stream<EncryptedData> streamEncrypt(
    Stream<String> plaintextStream,
    AESEncryption encryption,
  ) async* {
    await for (final plaintext in plaintextStream) {
      yield await encryption.encryptGCM(plaintext);
    }
  }

  // 缓存优化
  static final Map<String, EncryptedData> _encryptionCache = {};

  static Future<EncryptedData> cachedEncrypt(
    String plaintext,
    AESEncryption encryption,
  ) async {
    final hash = HashingUtils.sha256Hash(plaintext);

    if (_encryptionCache.containsKey(hash)) {
      return _encryptionCache[hash]!;
    }

    final encrypted = await encryption.encryptGCM(plaintext);
    _encryptionCache[hash] = encrypted;

    return encrypted;
  }

  static void clearCache() {
    _encryptionCache.clear();
  }
}
```

### 错误处理

```dart
class EncryptionErrorHandler {
  static Future<T> handleEncryptionOperation<T>(
    Future<T> Function() operation,
    {
      int maxRetries = 3,
      Duration retryDelay = const Duration(seconds: 1),
      void Function(Exception)? onError,
    }
  ) async {
    int attempts = 0;

    while (attempts < maxRetries) {
      try {
        return await operation();
      } catch (e) {
        attempts++;

        if (e is EncryptionException || e is DecryptionException) {
          onError?.call(e as Exception);

          if (attempts >= maxRetries) {
            rethrow;
          }

          await Future.delayed(retryDelay * attempts);
        } else {
          rethrow;
        }
      }
    }

    throw Exception('Max retries exceeded');
  }

  static void logEncryptionError(Exception error, StackTrace stackTrace) {
    // 记录加密错误（注意不要记录敏感信息）
    print('Encryption error: ${error.runtimeType}');
    print('Stack trace: $stackTrace');
  }
}
```

### 测试策略

```dart
// 加密测试工具
class EncryptionTestUtils {
  static Future<void> testEncryptionRoundTrip(
    String plaintext,
    Future<EncryptedData> Function(String) encrypt,
    Future<String> Function(EncryptedData) decrypt,
  ) async {
    final encrypted = await encrypt(plaintext);
    final decrypted = await decrypt(encrypted);

    assert(decrypted == plaintext, 'Round-trip encryption failed');
  }

  static Future<void> testPerformance(
    String testName,
    Future<void> Function() operation,
    {int iterations = 1000}
  ) async {
    final stopwatch = Stopwatch()..start();

    for (int i = 0; i < iterations; i++) {
      await operation();
    }

    stopwatch.stop();

    final avgTime = stopwatch.elapsedMicroseconds / iterations;
    print('$testName: ${avgTime.toStringAsFixed(2)} μs/operation');
  }

  static void testKeyGeneration() {
    final cryptoManager = CryptographyManager.instance;

    // 测试随机性
    final keys = <String>{};
    for (int i = 0; i < 1000; i++) {
      final key = cryptoManager.generateRandomString(32);
      assert(!keys.contains(key), 'Duplicate key generated');
      keys.add(key);
    }

    print('Key generation test passed: 1000 unique keys generated');
  }
}
```

## 总结

### 关键要点

1. **算法选择**
   - AES-GCM：通用对称加密，提供认证
   - ChaCha20-Poly1305：高性能流加密
   - RSA-OAEP：非对称加密，适合密钥交换
   - ECDSA：高效数字签名

2. **密钥管理**
   - 使用硬件安全存储
   - 实施密钥轮换策略
   - 监控密钥使用情况
   - 安全的密钥销毁

3. **性能优化**
   - 选择合适的加密算法
   - 实施批量处理
   - 使用流式处理大文件
   - 合理使用缓存

4. **安全考虑**
   - 防止时序攻击
   - 安全的随机数生成
   - 完整性验证
   - 错误处理不泄露信息

### 最佳实践建议

- 始终使用经过验证的加密库
- 定期更新加密算法和密钥
- 实施全面的安全测试
- 遵循相关法规和标准
- 建立安全事件响应机制

### 相关文档链接

- [Flutter Secure Storage](https://pub.dev/packages/flutter_secure_storage)
- [Encrypt Package](https://pub.dev/packages/encrypt)
- [PointyCastle](https://pub.dev/packages/pointycastle)
- [Crypto Package](https://pub.dev/packages/crypto)
- [NIST Cryptographic Standards](https://csrc.nist.gov/projects/cryptographic-standards-and-guidelines)
```
