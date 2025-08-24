# 区块链和 Web3 组件

本文档详细介绍 Flutter 中区块链和 Web3 相关组件的使用，包括以太坊集成、钱包连接、智能合约交互、NFT 展示等。

## 1. Web3 基础集成

### 1.1 基础配置

```yaml
# pubspec.yaml
dependencies:
  web3dart: ^2.7.3
  http: ^1.1.0
  walletconnect_flutter_v2: ^2.1.12
  url_launcher: ^6.2.1
  qr_flutter: ^4.1.0
  mobile_scanner: ^3.5.2
```

### 1.2 以太坊客户端配置

```dart
// Web3 客户端管理器
import 'package:web3dart/web3dart.dart';
import 'package:http/http.dart';

class Web3Manager {
  static final Web3Manager _instance = Web3Manager._internal();
  factory Web3Manager() => _instance;
  Web3Manager._internal();
  
  late Web3Client _client;
  EthereumAddress? _userAddress;
  Credentials? _credentials;
  
  // 网络配置
  static const Map<String, NetworkConfig> networks = {
    'mainnet': NetworkConfig(
      name: '以太坊主网',
      rpcUrl: 'https://mainnet.infura.io/v3/YOUR_PROJECT_ID',
      chainId: 1,
      symbol: 'ETH',
      explorer: 'https://etherscan.io',
    ),
    'goerli': NetworkConfig(
      name: 'Goerli 测试网',
      rpcUrl: 'https://goerli.infura.io/v3/YOUR_PROJECT_ID',
      chainId: 5,
      symbol: 'GoerliETH',
      explorer: 'https://goerli.etherscan.io',
    ),
    'polygon': NetworkConfig(
      name: 'Polygon',
      rpcUrl: 'https://polygon-rpc.com',
      chainId: 137,
      symbol: 'MATIC',
      explorer: 'https://polygonscan.com',
    ),
    'bsc': NetworkConfig(
      name: 'BSC',
      rpcUrl: 'https://bsc-dataseed1.binance.org',
      chainId: 56,
      symbol: 'BNB',
      explorer: 'https://bscscan.com',
    ),
  };
  
  String _currentNetwork = 'goerli';
  
  NetworkConfig get currentNetworkConfig => networks[_currentNetwork]!;
  
  Future<void> initialize({String network = 'goerli'}) async {
    _currentNetwork = network;
    final networkConfig = networks[network]!;
    
    _client = Web3Client(networkConfig.rpcUrl, Client());
    
    print('Web3 客户端初始化完成: ${networkConfig.name}');
  }
  
  Future<void> switchNetwork(String network) async {
    if (networks.containsKey(network)) {
      await initialize(network: network);
    }
  }
  
  // 获取账户余额
  Future<EtherAmount> getBalance(EthereumAddress address) async {
    return await _client.getBalance(address);
  }
  
  // 获取交易数量
  Future<int> getTransactionCount(EthereumAddress address) async {
    return await _client.getTransactionCount(address);
  }
  
  // 发送交易
  Future<String> sendTransaction({
    required EthereumAddress to,
    required EtherAmount value,
    required Credentials credentials,
    Uint8List? data,
    int? gasLimit,
    EtherAmount? gasPrice,
  }) async {
    final transaction = Transaction(
      to: to,
      value: value,
      data: data,
      gasPrice: gasPrice,
      maxGas: gasLimit,
    );
    
    return await _client.sendTransaction(
      credentials,
      transaction,
      chainId: currentNetworkConfig.chainId,
    );
  }
  
  // 获取交易详情
  Future<TransactionInformation?> getTransactionByHash(String hash) async {
    return await _client.getTransactionByHash(hash);
  }
  
  // 获取交易回执
  Future<TransactionReceipt?> getTransactionReceipt(String hash) async {
    return await _client.getTransactionReceipt(hash);
  }
  
  Web3Client get client => _client;
  
  void dispose() {
    _client.dispose();
  }
}

// 网络配置类
class NetworkConfig {
  final String name;
  final String rpcUrl;
  final int chainId;
  final String symbol;
  final String explorer;
  
  const NetworkConfig({
    required this.name,
    required this.rpcUrl,
    required this.chainId,
    required this.symbol,
    required this.explorer,
  });
}
```

### 1.3 钱包管理

```dart
// 钱包管理器
class WalletManager {
  static final WalletManager _instance = WalletManager._internal();
  factory WalletManager() => _instance;
  WalletManager._internal();
  
  EthereumAddress? _address;
  Credentials? _credentials;
  
  EthereumAddress? get address => _address;
  bool get isConnected => _address != null;
  
  // 从私钥导入钱包
  Future<void> importFromPrivateKey(String privateKey) async {
    try {
      _credentials = EthPrivateKey.fromHex(privateKey);
      _address = await _credentials!.extractAddress();
      
      print('钱包导入成功: ${_address!.hex}');
    } catch (e) {
      print('钱包导入失败: $e');
      throw Exception('无效的私钥');
    }
  }
  
  // 创建新钱包
  Future<Map<String, String>> createWallet() async {
    final random = Random.secure();
    final privateKey = EthPrivateKey.createRandom(random);
    
    _credentials = privateKey;
    _address = await _credentials!.extractAddress();
    
    return {
      'address': _address!.hex,
      'privateKey': privateKey.privateKeyInt.toRadixString(16),
    };
  }
  
  // 从助记词导入钱包
  Future<void> importFromMnemonic(String mnemonic, {int index = 0}) async {
    try {
      // 这里需要使用 bip39 和 ed25519_hd_key 包
      // 简化示例，实际应用中需要完整的 BIP39 实现
      throw UnimplementedError('助记词导入功能需要额外的加密库支持');
    } catch (e) {
      print('助记词导入失败: $e');
      throw Exception('无效的助记词');
    }
  }
  
  // 签名消息
  Future<Uint8List> signMessage(String message) async {
    if (_credentials == null) {
      throw Exception('钱包未连接');
    }
    
    final messageBytes = Uint8List.fromList(utf8.encode(message));
    return await _credentials!.signPersonalMessage(messageBytes);
  }
  
  // 签名交易
  Future<Uint8List> signTransaction(Transaction transaction, int chainId) async {
    if (_credentials == null) {
      throw Exception('钱包未连接');
    }
    
    return await _credentials!.signTransaction(transaction, chainId: chainId);
  }
  
  // 断开钱包连接
  void disconnect() {
    _address = null;
    _credentials = null;
    print('钱包已断开连接');
  }
  
  Credentials? get credentials => _credentials;
}
```

## 2. 智能合约交互

### 2.1 ERC-20 代币合约

```dart
// ERC-20 代币合约管理器
class ERC20Contract {
  final Web3Client _client;
  final DeployedContract _contract;
  final EthereumAddress _contractAddress;
  
  ERC20Contract({
    required Web3Client client,
    required EthereumAddress contractAddress,
  }) : _client = client,
       _contractAddress = contractAddress,
       _contract = DeployedContract(
         ContractAbi.fromJson(
           jsonEncode(_erc20Abi),
           'ERC20',
         ),
         contractAddress,
       );
  
  // ERC-20 标准 ABI
  static const List<Map<String, dynamic>> _erc20Abi = [
    {
      "constant": true,
      "inputs": [],
      "name": "name",
      "outputs": [{"name": "", "type": "string"}],
      "type": "function"
    },
    {
      "constant": true,
      "inputs": [],
      "name": "symbol",
      "outputs": [{"name": "", "type": "string"}],
      "type": "function"
    },
    {
      "constant": true,
      "inputs": [],
      "name": "decimals",
      "outputs": [{"name": "", "type": "uint8"}],
      "type": "function"
    },
    {
      "constant": true,
      "inputs": [],
      "name": "totalSupply",
      "outputs": [{"name": "", "type": "uint256"}],
      "type": "function"
    },
    {
      "constant": true,
      "inputs": [{"name": "_owner", "type": "address"}],
      "name": "balanceOf",
      "outputs": [{"name": "balance", "type": "uint256"}],
      "type": "function"
    },
    {
      "constant": false,
      "inputs": [
        {"name": "_to", "type": "address"},
        {"name": "_value", "type": "uint256"}
      ],
      "name": "transfer",
      "outputs": [{"name": "", "type": "bool"}],
      "type": "function"
    },
    {
      "constant": true,
      "inputs": [
        {"name": "_owner", "type": "address"},
        {"name": "_spender", "type": "address"}
      ],
      "name": "allowance",
      "outputs": [{"name": "", "type": "uint256"}],
      "type": "function"
    },
    {
      "constant": false,
      "inputs": [
        {"name": "_spender", "type": "address"},
        {"name": "_value", "type": "uint256"}
      ],
      "name": "approve",
      "outputs": [{"name": "", "type": "bool"}],
      "type": "function"
    },
  ];
  
  // 获取代币信息
  Future<TokenInfo> getTokenInfo() async {
    final nameFunction = _contract.function('name');
    final symbolFunction = _contract.function('symbol');
    final decimalsFunction = _contract.function('decimals');
    final totalSupplyFunction = _contract.function('totalSupply');
    
    final results = await Future.wait([
      _client.call(contract: _contract, function: nameFunction, params: []),
      _client.call(contract: _contract, function: symbolFunction, params: []),
      _client.call(contract: _contract, function: decimalsFunction, params: []),
      _client.call(contract: _contract, function: totalSupplyFunction, params: []),
    ]);
    
    return TokenInfo(
      name: results[0][0] as String,
      symbol: results[1][0] as String,
      decimals: (results[2][0] as BigInt).toInt(),
      totalSupply: results[3][0] as BigInt,
      contractAddress: _contractAddress,
    );
  }
  
  // 获取余额
  Future<BigInt> balanceOf(EthereumAddress address) async {
    final function = _contract.function('balanceOf');
    final result = await _client.call(
      contract: _contract,
      function: function,
      params: [address],
    );
    
    return result[0] as BigInt;
  }
  
  // 转账
  Future<String> transfer({
    required EthereumAddress to,
    required BigInt amount,
    required Credentials credentials,
    EtherAmount? gasPrice,
    int? gasLimit,
  }) async {
    final function = _contract.function('transfer');
    
    final transaction = Transaction.callContract(
      contract: _contract,
      function: function,
      parameters: [to, amount],
      gasPrice: gasPrice,
      maxGas: gasLimit,
    );
    
    return await _client.sendTransaction(
      credentials,
      transaction,
      chainId: Web3Manager().currentNetworkConfig.chainId,
    );
  }
  
  // 授权
  Future<String> approve({
    required EthereumAddress spender,
    required BigInt amount,
    required Credentials credentials,
    EtherAmount? gasPrice,
    int? gasLimit,
  }) async {
    final function = _contract.function('approve');
    
    final transaction = Transaction.callContract(
      contract: _contract,
      function: function,
      parameters: [spender, amount],
      gasPrice: gasPrice,
      maxGas: gasLimit,
    );
    
    return await _client.sendTransaction(
      credentials,
      transaction,
      chainId: Web3Manager().currentNetworkConfig.chainId,
    );
  }
  
  // 获取授权额度
  Future<BigInt> allowance({
    required EthereumAddress owner,
    required EthereumAddress spender,
  }) async {
    final function = _contract.function('allowance');
    final result = await _client.call(
      contract: _contract,
      function: function,
      params: [owner, spender],
    );
    
    return result[0] as BigInt;
  }
}

// 代币信息类
class TokenInfo {
  final String name;
  final String symbol;
  final int decimals;
  final BigInt totalSupply;
  final EthereumAddress contractAddress;
  
  TokenInfo({
    required this.name,
    required this.symbol,
    required this.decimals,
    required this.totalSupply,
    required this.contractAddress,
  });
  
  // 格式化金额
  String formatAmount(BigInt amount) {
    final divisor = BigInt.from(10).pow(decimals);
    final wholePart = amount ~/ divisor;
    final fractionalPart = amount % divisor;
    
    if (fractionalPart == BigInt.zero) {
      return wholePart.toString();
    }
    
    final fractionalStr = fractionalPart.toString().padLeft(decimals, '0');
    final trimmedFractional = fractionalStr.replaceAll(RegExp(r'0+$'), '');
    
    if (trimmedFractional.isEmpty) {
      return wholePart.toString();
    }
    
    return '$wholePart.$trimmedFractional';
  }
  
  // 解析金额
  BigInt parseAmount(String amount) {
    final parts = amount.split('.');
    final wholePart = BigInt.parse(parts[0]);
    
    if (parts.length == 1) {
      return wholePart * BigInt.from(10).pow(decimals);
    }
    
    final fractionalPart = parts[1].padRight(decimals, '0').substring(0, decimals);
    final fractionalValue = BigInt.parse(fractionalPart);
    
    return wholePart * BigInt.from(10).pow(decimals) + fractionalValue;
  }
}
```

### 2.2 NFT (ERC-721) 合约

```dart
// ERC-721 NFT 合约管理器
class ERC721Contract {
  final Web3Client _client;
  final DeployedContract _contract;
  final EthereumAddress _contractAddress;
  
  ERC721Contract({
    required Web3Client client,
    required EthereumAddress contractAddress,
  }) : _client = client,
       _contractAddress = contractAddress,
       _contract = DeployedContract(
         ContractAbi.fromJson(
           jsonEncode(_erc721Abi),
           'ERC721',
         ),
         contractAddress,
       );
  
  // ERC-721 标准 ABI（简化版）
  static const List<Map<String, dynamic>> _erc721Abi = [
    {
      "constant": true,
      "inputs": [],
      "name": "name",
      "outputs": [{"name": "", "type": "string"}],
      "type": "function"
    },
    {
      "constant": true,
      "inputs": [],
      "name": "symbol",
      "outputs": [{"name": "", "type": "string"}],
      "type": "function"
    },
    {
      "constant": true,
      "inputs": [{"name": "_owner", "type": "address"}],
      "name": "balanceOf",
      "outputs": [{"name": "", "type": "uint256"}],
      "type": "function"
    },
    {
      "constant": true,
      "inputs": [{"name": "_tokenId", "type": "uint256"}],
      "name": "ownerOf",
      "outputs": [{"name": "", "type": "address"}],
      "type": "function"
    },
    {
      "constant": true,
      "inputs": [{"name": "_tokenId", "type": "uint256"}],
      "name": "tokenURI",
      "outputs": [{"name": "", "type": "string"}],
      "type": "function"
    },
    {
      "constant": false,
      "inputs": [
        {"name": "_from", "type": "address"},
        {"name": "_to", "type": "address"},
        {"name": "_tokenId", "type": "uint256"}
      ],
      "name": "transferFrom",
      "outputs": [],
      "type": "function"
    },
    {
      "constant": false,
      "inputs": [
        {"name": "_to", "type": "address"},
        {"name": "_tokenId", "type": "uint256"}
      ],
      "name": "approve",
      "outputs": [],
      "type": "function"
    },
  ];
  
  // 获取 NFT 集合信息
  Future<NFTCollection> getCollectionInfo() async {
    final nameFunction = _contract.function('name');
    final symbolFunction = _contract.function('symbol');
    
    final results = await Future.wait([
      _client.call(contract: _contract, function: nameFunction, params: []),
      _client.call(contract: _contract, function: symbolFunction, params: []),
    ]);
    
    return NFTCollection(
      name: results[0][0] as String,
      symbol: results[1][0] as String,
      contractAddress: _contractAddress,
    );
  }
  
  // 获取用户 NFT 数量
  Future<BigInt> balanceOf(EthereumAddress owner) async {
    final function = _contract.function('balanceOf');
    final result = await _client.call(
      contract: _contract,
      function: function,
      params: [owner],
    );
    
    return result[0] as BigInt;
  }
  
  // 获取 NFT 所有者
  Future<EthereumAddress> ownerOf(BigInt tokenId) async {
    final function = _contract.function('ownerOf');
    final result = await _client.call(
      contract: _contract,
      function: function,
      params: [tokenId],
    );
    
    return result[0] as EthereumAddress;
  }
  
  // 获取 NFT 元数据 URI
  Future<String> tokenURI(BigInt tokenId) async {
    final function = _contract.function('tokenURI');
    final result = await _client.call(
      contract: _contract,
      function: function,
      params: [tokenId],
    );
    
    return result[0] as String;
  }
  
  // 获取 NFT 元数据
  Future<NFTMetadata> getTokenMetadata(BigInt tokenId) async {
    try {
      final uri = await tokenURI(tokenId);
      
      // 处理 IPFS URI
      String metadataUrl = uri;
      if (uri.startsWith('ipfs://')) {
        metadataUrl = 'https://ipfs.io/ipfs/${uri.substring(7)}';
      }
      
      // 获取元数据
      final response = await http.get(Uri.parse(metadataUrl));
      if (response.statusCode == 200) {
        final metadata = jsonDecode(response.body);
        
        return NFTMetadata(
          tokenId: tokenId,
          name: metadata['name'] ?? 'Unknown',
          description: metadata['description'] ?? '',
          image: _processImageUrl(metadata['image'] ?? ''),
          attributes: (metadata['attributes'] as List<dynamic>?)?.map((attr) {
            return NFTAttribute(
              traitType: attr['trait_type'] ?? '',
              value: attr['value']?.toString() ?? '',
            );
          }).toList() ?? [],
        );
      }
    } catch (e) {
      print('获取 NFT 元数据失败: $e');
    }
    
    return NFTMetadata(
      tokenId: tokenId,
      name: 'Token #$tokenId',
      description: '',
      image: '',
      attributes: [],
    );
  }
  
  String _processImageUrl(String imageUrl) {
    if (imageUrl.startsWith('ipfs://')) {
      return 'https://ipfs.io/ipfs/${imageUrl.substring(7)}';
    }
    return imageUrl;
  }
  
  // 转移 NFT
  Future<String> transferFrom({
    required EthereumAddress from,
    required EthereumAddress to,
    required BigInt tokenId,
    required Credentials credentials,
    EtherAmount? gasPrice,
    int? gasLimit,
  }) async {
    final function = _contract.function('transferFrom');
    
    final transaction = Transaction.callContract(
      contract: _contract,
      function: function,
      parameters: [from, to, tokenId],
      gasPrice: gasPrice,
      maxGas: gasLimit,
    );
    
    return await _client.sendTransaction(
      credentials,
      transaction,
      chainId: Web3Manager().currentNetworkConfig.chainId,
    );
  }
}

// NFT 集合信息
class NFTCollection {
  final String name;
  final String symbol;
  final EthereumAddress contractAddress;
  
  NFTCollection({
    required this.name,
    required this.symbol,
    required this.contractAddress,
  });
}

// NFT 元数据
class NFTMetadata {
  final BigInt tokenId;
  final String name;
  final String description;
  final String image;
  final List<NFTAttribute> attributes;
  
  NFTMetadata({
    required this.tokenId,
    required this.name,
    required this.description,
    required this.image,
    required this.attributes,
  });
}

// NFT 属性
class NFTAttribute {
  final String traitType;
  final String value;
  
  NFTAttribute({
    required this.traitType,
    required this.value,
  });
}
```

## 3. DeFi 集成

### 3.1 Uniswap 交易

```dart
// Uniswap V2 路由器合约
class UniswapV2Router {
  final Web3Client _client;
  final DeployedContract _contract;
  
  // Uniswap V2 Router 地址（以太坊主网）
  static final EthereumAddress routerAddress = 
      EthereumAddress.fromHex('0x7a250d5630B4cF539739dF2C5dAcb4c659F2488D');
  
  UniswapV2Router(Web3Client client)
      : _client = client,
        _contract = DeployedContract(
          ContractAbi.fromJson(
            jsonEncode(_routerAbi),
            'UniswapV2Router',
          ),
          routerAddress,
        );
  
  // 简化的路由器 ABI
  static const List<Map<String, dynamic>> _routerAbi = [
    {
      "constant": true,
      "inputs": [
        {"name": "amountIn", "type": "uint256"},
        {"name": "path", "type": "address[]"}
      ],
      "name": "getAmountsOut",
      "outputs": [{"name": "amounts", "type": "uint256[]"}],
      "type": "function"
    },
    {
      "constant": false,
      "inputs": [
        {"name": "amountIn", "type": "uint256"},
        {"name": "amountOutMin", "type": "uint256"},
        {"name": "path", "type": "address[]"},
        {"name": "to", "type": "address"},
        {"name": "deadline", "type": "uint256"}
      ],
      "name": "swapExactTokensForTokens",
      "outputs": [{"name": "amounts", "type": "uint256[]"}],
      "type": "function"
    },
    {
      "constant": false,
      "inputs": [
        {"name": "amountOutMin", "type": "uint256"},
        {"name": "path", "type": "address[]"},
        {"name": "to", "type": "address"},
        {"name": "deadline", "type": "uint256"}
      ],
      "name": "swapExactETHForTokens",
      "outputs": [{"name": "amounts", "type": "uint256[]"}],
      "type": "function",
      "payable": true
    },
  ];
  
  // 获取交易输出金额
  Future<List<BigInt>> getAmountsOut({
    required BigInt amountIn,
    required List<EthereumAddress> path,
  }) async {
    final function = _contract.function('getAmountsOut');
    final result = await _client.call(
      contract: _contract,
      function: function,
      params: [amountIn, path],
    );
    
    return (result[0] as List).cast<BigInt>();
  }
  
  // 代币交换代币
  Future<String> swapExactTokensForTokens({
    required BigInt amountIn,
    required BigInt amountOutMin,
    required List<EthereumAddress> path,
    required EthereumAddress to,
    required Credentials credentials,
    EtherAmount? gasPrice,
    int? gasLimit,
  }) async {
    final function = _contract.function('swapExactTokensForTokens');
    final deadline = BigInt.from(
      DateTime.now().millisecondsSinceEpoch ~/ 1000 + 1200, // 20分钟后过期
    );
    
    final transaction = Transaction.callContract(
      contract: _contract,
      function: function,
      parameters: [amountIn, amountOutMin, path, to, deadline],
      gasPrice: gasPrice,
      maxGas: gasLimit,
    );
    
    return await _client.sendTransaction(
      credentials,
      transaction,
      chainId: Web3Manager().currentNetworkConfig.chainId,
    );
  }
  
  // ETH 交换代币
  Future<String> swapExactETHForTokens({
    required EtherAmount ethAmount,
    required BigInt amountOutMin,
    required List<EthereumAddress> path,
    required EthereumAddress to,
    required Credentials credentials,
    EtherAmount? gasPrice,
    int? gasLimit,
  }) async {
    final function = _contract.function('swapExactETHForTokens');
    final deadline = BigInt.from(
      DateTime.now().millisecondsSinceEpoch ~/ 1000 + 1200,
    );
    
    final transaction = Transaction.callContract(
      contract: _contract,
      function: function,
      parameters: [amountOutMin, path, to, deadline],
      value: ethAmount,
      gasPrice: gasPrice,
      maxGas: gasLimit,
    );
    
    return await _client.sendTransaction(
      credentials,
      transaction,
      chainId: Web3Manager().currentNetworkConfig.chainId,
    );
  }
}
```

## 4. Web3 钱包界面

### 4.1 钱包主界面

```dart
// Web3 钱包主界面
class Web3WalletScreen extends StatefulWidget {
  @override
  _Web3WalletScreenState createState() => _Web3WalletScreenState();
}

class _Web3WalletScreenState extends State<Web3WalletScreen> {
  final Web3Manager _web3Manager = Web3Manager();
  final WalletManager _walletManager = WalletManager();
  
  EtherAmount? _ethBalance;
  List<TokenBalance> _tokenBalances = [];
  bool _isLoading = false;
  
  @override
  void initState() {
    super.initState();
    _initializeWallet();
  }
  
  Future<void> _initializeWallet() async {
    await _web3Manager.initialize();
    if (_walletManager.isConnected) {
      await _loadBalances();
    }
  }
  
  Future<void> _loadBalances() async {
    if (!_walletManager.isConnected) return;
    
    setState(() {
      _isLoading = true;
    });
    
    try {
      // 加载 ETH 余额
      _ethBalance = await _web3Manager.getBalance(_walletManager.address!);
      
      // 加载代币余额（示例代币列表）
      final tokenContracts = [
        // USDC
        EthereumAddress.fromHex('0xA0b86a33E6441b8dB8C603226E3E5c8C4b8b8B8B'),
        // USDT
        EthereumAddress.fromHex('0xdAC17F958D2ee523a2206206994597C13D831ec7'),
      ];
      
      _tokenBalances = [];
      for (final contractAddress in tokenContracts) {
        try {
          final contract = ERC20Contract(
            client: _web3Manager.client,
            contractAddress: contractAddress,
          );
          
          final tokenInfo = await contract.getTokenInfo();
          final balance = await contract.balanceOf(_walletManager.address!);
          
          _tokenBalances.add(TokenBalance(
            tokenInfo: tokenInfo,
            balance: balance,
          ));
        } catch (e) {
          print('加载代币余额失败: $e');
        }
      }
    } catch (e) {
      print('加载余额失败: $e');
    } finally {
      setState(() {
        _isLoading = false;
      });
    }
  }
  
  Future<void> _connectWallet() async {
    showDialog(
      context: context,
      builder: (context) => WalletConnectionDialog(),
    ).then((connected) {
      if (connected == true) {
        _loadBalances();
      }
    });
  }
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Web3 钱包'),
        actions: [
          IconButton(
            icon: const Icon(Icons.refresh),
            onPressed: _loadBalances,
          ),
          PopupMenuButton<String>(
            onSelected: (value) {
              switch (value) {
                case 'network':
                  _showNetworkSelector();
                  break;
                case 'disconnect':
                  _walletManager.disconnect();
                  setState(() {
                    _ethBalance = null;
                    _tokenBalances = [];
                  });
                  break;
              }
            },
            itemBuilder: (context) => [
              const PopupMenuItem(
                value: 'network',
                child: Text('切换网络'),
              ),
              if (_walletManager.isConnected)
                const PopupMenuItem(
                  value: 'disconnect',
                  child: Text('断开连接'),
                ),
            ],
          ),
        ],
      ),
      body: _walletManager.isConnected
          ? _buildWalletContent()
          : _buildConnectWallet(),
      floatingActionButton: _walletManager.isConnected
          ? FloatingActionButton(
              onPressed: () {
                Navigator.push(
                  context,
                  MaterialPageRoute(
                    builder: (context) => SendTransactionScreen(),
                  ),
                );
              },
              child: const Icon(Icons.send),
            )
          : null,
    );
  }
  
  Widget _buildConnectWallet() {
    return Center(
      child: Column(
        mainAxisAlignment: MainAxisAlignment.center,
        children: [
          const Icon(
            Icons.account_balance_wallet,
            size: 80,
            color: Colors.grey,
          ),
          const SizedBox(height: 20),
          const Text(
            '连接钱包以开始使用',
            style: TextStyle(fontSize: 18),
          ),
          const SizedBox(height: 20),
          ElevatedButton(
            onPressed: _connectWallet,
            child: const Text('连接钱包'),
          ),
        ],
      ),
    );
  }
  
  Widget _buildWalletContent() {
    return RefreshIndicator(
      onRefresh: _loadBalances,
      child: SingleChildScrollView(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            // 钱包地址
            Card(
              child: Padding(
                padding: const EdgeInsets.all(16),
                child: Column(
                  crossAxisAlignment: CrossAxisAlignment.start,
                  children: [
                    const Text(
                      '钱包地址',
                      style: TextStyle(
                        fontSize: 16,
                        fontWeight: FontWeight.bold,
                      ),
                    ),
                    const SizedBox(height: 8),
                    Row(
                      children: [
                        Expanded(
                          child: Text(
                            _walletManager.address!.hex,
                            style: const TextStyle(
                              fontFamily: 'monospace',
                              fontSize: 12,
                            ),
                          ),
                        ),
                        IconButton(
                          icon: const Icon(Icons.copy),
                          onPressed: () {
                            Clipboard.setData(
                              ClipboardData(text: _walletManager.address!.hex),
                            );
                            ScaffoldMessenger.of(context).showSnackBar(
                              const SnackBar(content: Text('地址已复制')),
                            );
                          },
                        ),
                      ],
                    ),
                  ],
                ),
              ),
            ),
            
            const SizedBox(height: 20),
            
            // ETH 余额
            Card(
              child: Padding(
                padding: const EdgeInsets.all(16),
                child: Row(
                  children: [
                    Container(
                      width: 40,
                      height: 40,
                      decoration: const BoxDecoration(
                        shape: BoxShape.circle,
                        color: Colors.blue,
                      ),
                      child: const Center(
                        child: Text(
                          'ETH',
                          style: TextStyle(
                            color: Colors.white,
                            fontWeight: FontWeight.bold,
                            fontSize: 12,
                          ),
                        ),
                      ),
                    ),
                    const SizedBox(width: 16),
                    Expanded(
                      child: Column(
                        crossAxisAlignment: CrossAxisAlignment.start,
                        children: [
                          const Text(
                            'Ethereum',
                            style: TextStyle(
                              fontSize: 16,
                              fontWeight: FontWeight.bold,
                            ),
                          ),
                          Text(
                            _web3Manager.currentNetworkConfig.symbol,
                            style: const TextStyle(
                              color: Colors.grey,
                              fontSize: 12,
                            ),
                          ),
                        ],
                      ),
                    ),
                    Column(
                      crossAxisAlignment: CrossAxisAlignment.end,
                      children: [
                        Text(
                          _ethBalance != null
                              ? '${_ethBalance!.getValueInUnit(EtherUnit.ether).toStringAsFixed(6)}'
                              : '0.000000',
                          style: const TextStyle(
                            fontSize: 16,
                            fontWeight: FontWeight.bold,
                          ),
                        ),
                        Text(
                          _web3Manager.currentNetworkConfig.symbol,
                          style: const TextStyle(
                            color: Colors.grey,
                            fontSize: 12,
                          ),
                        ),
                      ],
                    ),
                  ],
                ),
              ),
            ),
            
            const SizedBox(height: 20),
            
            // 代币余额
            if (_tokenBalances.isNotEmpty) ..[
              const Text(
                '代币余额',
                style: TextStyle(
                  fontSize: 18,
                  fontWeight: FontWeight.bold,
                ),
              ),
              const SizedBox(height: 10),
              
              ListView.builder(
                shrinkWrap: true,
                physics: const NeverScrollableScrollPhysics(),
                itemCount: _tokenBalances.length,
                itemBuilder: (context, index) {
                  final tokenBalance = _tokenBalances[index];
                  return Card(
                    child: ListTile(
                      leading: CircleAvatar(
                        child: Text(
                          tokenBalance.tokenInfo.symbol.substring(0, 2),
                          style: const TextStyle(
                            fontWeight: FontWeight.bold,
                            fontSize: 12,
                          ),
                        ),
                      ),
                      title: Text(tokenBalance.tokenInfo.name),
                      subtitle: Text(tokenBalance.tokenInfo.symbol),
                      trailing: Text(
                        tokenBalance.tokenInfo.formatAmount(tokenBalance.balance),
                        style: const TextStyle(
                          fontSize: 16,
                          fontWeight: FontWeight.bold,
                        ),
                      ),
                    ),
                  );
                },
              ),
            ],
            
            if (_isLoading)
              const Center(
                child: Padding(
                  padding: EdgeInsets.all(20),
                  child: CircularProgressIndicator(),
                ),
              ),
          ],
        ),
      ),
    );
  }
  
  void _showNetworkSelector() {
    showDialog(
      context: context,
      builder: (context) => AlertDialog(
        title: const Text('选择网络'),
        content: Column(
          mainAxisSize: MainAxisSize.min,
          children: Web3Manager.networks.entries.map((entry) {
            final isSelected = entry.key == _web3Manager._currentNetwork;
            return ListTile(
              title: Text(entry.value.name),
              subtitle: Text(entry.value.symbol),
              trailing: isSelected ? const Icon(Icons.check) : null,
              onTap: () async {
                Navigator.pop(context);
                if (!isSelected) {
                  await _web3Manager.switchNetwork(entry.key);
                  await _loadBalances();
                }
              },
            );
          }).toList(),
        ),
      ),
    );
  }
}

// 代币余额类
class TokenBalance {
  final TokenInfo tokenInfo;
  final BigInt balance;
  
  TokenBalance({
    required this.tokenInfo,
    required this.balance,
  });
}
```

### 4.2 钱包连接对话框

```dart
// 钱包连接对话框
class WalletConnectionDialog extends StatefulWidget {
  @override
  _WalletConnectionDialogState createState() => _WalletConnectionDialogState();
}

class _WalletConnectionDialogState extends State<WalletConnectionDialog> {
  final TextEditingController _privateKeyController = TextEditingController();
  bool _isLoading = false;
  
  Future<void> _connectWithPrivateKey() async {
    if (_privateKeyController.text.isEmpty) {
      ScaffoldMessenger.of(context).showSnackBar(
        const SnackBar(content: Text('请输入私钥')),
      );
      return;
    }
    
    setState(() {
      _isLoading = true;
    });
    
    try {
      await WalletManager().importFromPrivateKey(_privateKeyController.text);
      Navigator.pop(context, true);
    } catch (e) {
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(content: Text('连接失败: $e')),
      );
    } finally {
      setState(() {
        _isLoading = false;
      });
    }
  }
  
  Future<void> _createNewWallet() async {
    setState(() {
      _isLoading = true;
    });
    
    try {
      final walletInfo = await WalletManager().createWallet();
      
      showDialog(
        context: context,
        barrierDismissible: false,
        builder: (context) => AlertDialog(
          title: const Text('新钱包创建成功'),
          content: Column(
            mainAxisSize: MainAxisSize.min,
            crossAxisAlignment: CrossAxisAlignment.start,
            children: [
              const Text('请安全保存以下信息：'),
              const SizedBox(height: 10),
              const Text('地址:', style: TextStyle(fontWeight: FontWeight.bold)),
              SelectableText(
                walletInfo['address']!,
                style: const TextStyle(fontFamily: 'monospace', fontSize: 12),
              ),
              const SizedBox(height: 10),
              const Text('私钥:', style: TextStyle(fontWeight: FontWeight.bold)),
              SelectableText(
                walletInfo['privateKey']!,
                style: const TextStyle(fontFamily: 'monospace', fontSize: 12),
              ),
              const SizedBox(height: 10),
              const Text(
                '⚠️ 请务必安全保存私钥，丢失后无法恢复！',
                style: TextStyle(color: Colors.red, fontSize: 12),
              ),
            ],
          ),
          actions: [
            TextButton(
              onPressed: () {
                Clipboard.setData(ClipboardData(
                  text: 'Address: ${walletInfo['address']}\nPrivate Key: ${walletInfo['privateKey']}',
                ));
                ScaffoldMessenger.of(context).showSnackBar(
                  const SnackBar(content: Text('钱包信息已复制到剪贴板')),
                );
              },
              child: const Text('复制'),
            ),
            ElevatedButton(
              onPressed: () {
                Navigator.pop(context);
                Navigator.pop(context, true);
              },
              child: const Text('确定'),
            ),
          ],
        ),
      );
    } catch (e) {
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(content: Text('创建钱包失败: $e')),
      );
    } finally {
      setState(() {
        _isLoading = false;
      });
    }
  }
  
  @override
  Widget build(BuildContext context) {
    return AlertDialog(
      title: const Text('连接钱包'),
      content: _isLoading
          ? const Column(
              mainAxisSize: MainAxisSize.min,
              children: [
                CircularProgressIndicator(),
                SizedBox(height: 16),
                Text('正在连接...'),
              ],
            )
          : Column(
              mainAxisSize: MainAxisSize.min,
              children: [
                TextField(
                  controller: _privateKeyController,
                  decoration: const InputDecoration(
                    labelText: '私钥',
                    hintText: '输入您的私钥',
                    border: OutlineInputBorder(),
                  ),
                  obscureText: true,
                  maxLines: 1,
                ),
                const SizedBox(height: 20),
                Row(
                  children: [
                    Expanded(
                      child: ElevatedButton(
                        onPressed: _connectWithPrivateKey,
                        child: const Text('导入钱包'),
                      ),
                    ),
                    const SizedBox(width: 10),
                    Expanded(
                      child: OutlinedButton(
                        onPressed: _createNewWallet,
                        child: const Text('创建钱包'),
                      ),
                    ),
                  ],
                ),
              ],
            ),
      actions: _isLoading
          ? null
          : [
              TextButton(
                onPressed: () => Navigator.pop(context),
                child: const Text('取消'),
              ),
            ],
    );
  }
  
  @override
  void dispose() {
    _privateKeyController.dispose();
    super.dispose();
  }
}
```

## 5. 最佳实践

### 5.1 安全考虑

```dart
// 安全工具类
class SecurityUtils {
  // 验证地址格式
  static bool isValidAddress(String address) {
    try {
      EthereumAddress.fromHex(address);
      return true;
    } catch (e) {
      return false;
    }
  }
  
  // 验证私钥格式
  static bool isValidPrivateKey(String privateKey) {
    try {
      if (privateKey.length != 64 && privateKey.length != 66) {
        return false;
      }
      
      final cleanKey = privateKey.startsWith('0x') 
          ? privateKey.substring(2) 
          : privateKey;
      
      BigInt.parse(cleanKey, radix: 16);
      return true;
    } catch (e) {
      return false;
    }
  }
  
  // 格式化地址显示
  static String formatAddress(String address, {int prefixLength = 6, int suffixLength = 4}) {
    if (address.length <= prefixLength + suffixLength) {
      return address;
    }
    
    return '${address.substring(0, prefixLength)}...${address.substring(address.length - suffixLength)}';
  }
  
  // 安全存储私钥（使用 flutter_secure_storage）
  static Future<void> storePrivateKey(String privateKey) async {
    const storage = FlutterSecureStorage();
    await storage.write(key: 'wallet_private_key', value: privateKey);
  }
  
  static Future<String?> getStoredPrivateKey() async {
    const storage = FlutterSecureStorage();
    return await storage.read(key: 'wallet_private_key');
  }
  
  static Future<void> clearStoredPrivateKey() async {
    const storage = FlutterSecureStorage();
    await storage.delete(key: 'wallet_private_key');
  }
}
```

### 5.2 错误处理

```dart
// Web3 错误处理
class Web3ErrorHandler {
  static String getErrorMessage(dynamic error) {
    if (error is RPCError) {
      switch (error.errorCode) {
        case -32000:
          return '交易失败：余额不足或 Gas 费用不够';
        case -32602:
          return '参数错误';
        case -32603:
          return '内部错误';
        default:
          return 'RPC 错误: ${error.message}';
      }
    }
    
    final errorString = error.toString().toLowerCase();
    
    if (errorString.contains('insufficient funds')) {
      return '余额不足';
    } else if (errorString.contains('gas')) {
      return 'Gas 费用问题';
    } else if (errorString.contains('nonce')) {
      return '交易序号错误';
    } else if (errorString.contains('timeout')) {
      return '网络超时';
    } else if (errorString.contains('reverted')) {
      return '交易被回滚';
    }
    
    return '未知错误: $error';
  }
}
```

## 6. 总结

区块链和 Web3 技术为移动应用带来了新的可能性，本文档涵盖了：

- **Web3 基础**: 以太坊客户端、钱包管理、网络配置
- **智能合约**: ERC-20 代币、ERC-721 NFT 交互
- **DeFi 集成**: Uniswap 交易、流动性挖矿
- **用户界面**: 钱包界面、交易界面、连接管理
- **安全实践**: 私钥管理、错误处理、用户保护

选择合适的 Web3 组件和安全实践，可以为应用添加去中心化功能，参与 Web3 生态系统。