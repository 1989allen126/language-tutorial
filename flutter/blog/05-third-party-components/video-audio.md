# 视频音频组件

本文档详细介绍 Flutter 中视频和音频处理组件的使用，包括播放器、录制、流媒体等功能。

## 1. 视频播放组件

### 1.1 video_player

```yaml
# pubspec.yaml
dependencies:
  video_player: ^2.8.1
```

```dart
// 基础视频播放器
import 'package:video_player/video_player.dart';

class VideoPlayerWidget extends StatefulWidget {
  final String videoUrl;
  
  const VideoPlayerWidget({Key? key, required this.videoUrl}) : super(key: key);
  
  @override
  _VideoPlayerWidgetState createState() => _VideoPlayerWidgetState();
}

class _VideoPlayerWidgetState extends State<VideoPlayerWidget> {
  late VideoPlayerController _controller;
  bool _isPlaying = false;
  bool _isInitialized = false;
  
  @override
  void initState() {
    super.initState();
    _initializePlayer();
  }
  
  Future<void> _initializePlayer() async {
    try {
      _controller = VideoPlayerController.networkUrl(
        Uri.parse(widget.videoUrl),
      );
      
      await _controller.initialize();
      
      setState(() {
        _isInitialized = true;
      });
      
      _controller.addListener(() {
        if (_controller.value.hasError) {
          print('视频播放错误: ${_controller.value.errorDescription}');
        }
      });
    } catch (e) {
      print('初始化视频播放器失败: $e');
    }
  }
  
  @override
  Widget build(BuildContext context) {
    if (!_isInitialized) {
      return const Center(
        child: CircularProgressIndicator(),
      );
    }
    
    return Column(
      children: [
        AspectRatio(
          aspectRatio: _controller.value.aspectRatio,
          child: VideoPlayer(_controller),
        ),
        VideoProgressIndicator(
          _controller,
          allowScrubbing: true,
          colors: const VideoProgressColors(
            playedColor: Colors.blue,
            bufferedColor: Colors.grey,
            backgroundColor: Colors.black12,
          ),
        ),
        Row(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            IconButton(
              icon: Icon(
                _isPlaying ? Icons.pause : Icons.play_arrow,
              ),
              onPressed: () {
                setState(() {
                  if (_isPlaying) {
                    _controller.pause();
                  } else {
                    _controller.play();
                  }
                  _isPlaying = !_isPlaying;
                });
              },
            ),
            IconButton(
              icon: const Icon(Icons.stop),
              onPressed: () {
                _controller.seekTo(Duration.zero);
                _controller.pause();
                setState(() {
                  _isPlaying = false;
                });
              },
            ),
          ],
        ),
      ],
    );
  }
  
  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }
}
```

### 1.2 chewie - 高级视频播放器

```yaml
# pubspec.yaml
dependencies:
  chewie: ^1.7.4
  video_player: ^2.8.1
```

```dart
// 高级视频播放器
import 'package:chewie/chewie.dart';
import 'package:video_player/video_player.dart';

class ChewiePlayerWidget extends StatefulWidget {
  final String videoUrl;
  
  const ChewiePlayerWidget({Key? key, required this.videoUrl}) : super(key: key);
  
  @override
  _ChewiePlayerWidgetState createState() => _ChewiePlayerWidgetState();
}

class _ChewiePlayerWidgetState extends State<ChewiePlayerWidget> {
  late VideoPlayerController _videoPlayerController;
  ChewieController? _chewieController;
  
  @override
  void initState() {
    super.initState();
    _initializePlayer();
  }
  
  Future<void> _initializePlayer() async {
    _videoPlayerController = VideoPlayerController.networkUrl(
      Uri.parse(widget.videoUrl),
    );
    
    await _videoPlayerController.initialize();
    
    _chewieController = ChewieController(
      videoPlayerController: _videoPlayerController,
      autoPlay: false,
      looping: false,
      showControls: true,
      materialProgressColors: ChewieProgressColors(
        playedColor: Colors.blue,
        handleColor: Colors.blueAccent,
        backgroundColor: Colors.grey,
        bufferedColor: Colors.lightBlue,
      ),
      placeholder: Container(
        color: Colors.grey,
        child: const Center(
          child: CircularProgressIndicator(),
        ),
      ),
      autoInitialize: true,
    );
    
    setState(() {});
  }
  
  @override
  Widget build(BuildContext context) {
    return _chewieController != null
        ? Chewie(controller: _chewieController!)
        : const Center(child: CircularProgressIndicator());
  }
  
  @override
  void dispose() {
    _videoPlayerController.dispose();
    _chewieController?.dispose();
    super.dispose();
  }
}
```

## 2. 音频播放组件

### 2.1 audioplayers

```yaml
# pubspec.yaml
dependencies:
  audioplayers: ^5.2.1
```

```dart
// 音频播放器
import 'package:audioplayers/audioplayers.dart';

class AudioPlayerWidget extends StatefulWidget {
  final String audioUrl;
  
  const AudioPlayerWidget({Key? key, required this.audioUrl}) : super(key: key);
  
  @override
  _AudioPlayerWidgetState createState() => _AudioPlayerWidgetState();
}

class _AudioPlayerWidgetState extends State<AudioPlayerWidget> {
  late AudioPlayer _audioPlayer;
  bool _isPlaying = false;
  Duration _duration = Duration.zero;
  Duration _position = Duration.zero;
  
  @override
  void initState() {
    super.initState();
    _audioPlayer = AudioPlayer();
    _setupAudioPlayer();
  }
  
  void _setupAudioPlayer() {
    _audioPlayer.onDurationChanged.listen((duration) {
      setState(() {
        _duration = duration;
      });
    });
    
    _audioPlayer.onPositionChanged.listen((position) {
      setState(() {
        _position = position;
      });
    });
    
    _audioPlayer.onPlayerStateChanged.listen((state) {
      setState(() {
        _isPlaying = state == PlayerState.playing;
      });
    });
  }
  
  Future<void> _playPause() async {
    if (_isPlaying) {
      await _audioPlayer.pause();
    } else {
      await _audioPlayer.play(UrlSource(widget.audioUrl));
    }
  }
  
  Future<void> _stop() async {
    await _audioPlayer.stop();
  }
  
  String _formatDuration(Duration duration) {
    String twoDigits(int n) => n.toString().padLeft(2, '0');
    final minutes = twoDigits(duration.inMinutes.remainder(60));
    final seconds = twoDigits(duration.inSeconds.remainder(60));
    return '$minutes:$seconds';
  }
  
  @override
  Widget build(BuildContext context) {
    return Card(
      child: Padding(
        padding: const EdgeInsets.all(16.0),
        child: Column(
          mainAxisSize: MainAxisSize.min,
          children: [
            Row(
              children: [
                IconButton(
                  icon: Icon(
                    _isPlaying ? Icons.pause : Icons.play_arrow,
                    size: 32,
                  ),
                  onPressed: _playPause,
                ),
                IconButton(
                  icon: const Icon(Icons.stop, size: 32),
                  onPressed: _stop,
                ),
                Expanded(
                  child: Slider(
                    value: _position.inSeconds.toDouble(),
                    max: _duration.inSeconds.toDouble(),
                    onChanged: (value) async {
                      await _audioPlayer.seek(
                        Duration(seconds: value.toInt()),
                      );
                    },
                  ),
                ),
              ],
            ),
            Row(
              mainAxisAlignment: MainAxisAlignment.spaceBetween,
              children: [
                Text(_formatDuration(_position)),
                Text(_formatDuration(_duration)),
              ],
            ),
          ],
        ),
      ),
    );
  }
  
  @override
  void dispose() {
    _audioPlayer.dispose();
    super.dispose();
  }
}
```

### 2.2 just_audio - 高级音频播放

```yaml
# pubspec.yaml
dependencies:
  just_audio: ^0.9.36
  audio_session: ^0.1.16
```

```dart
// 高级音频播放器
import 'package:just_audio/just_audio.dart';
import 'package:audio_session/audio_session.dart';

class JustAudioPlayerWidget extends StatefulWidget {
  final List<String> playlist;
  
  const JustAudioPlayerWidget({Key? key, required this.playlist}) : super(key: key);
  
  @override
  _JustAudioPlayerWidgetState createState() => _JustAudioPlayerWidgetState();
}

class _JustAudioPlayerWidgetState extends State<JustAudioPlayerWidget> {
  late AudioPlayer _player;
  late ConcatenatingAudioSource _playlist;
  
  @override
  void initState() {
    super.initState();
    _initializePlayer();
  }
  
  Future<void> _initializePlayer() async {
    _player = AudioPlayer();
    
    // 配置音频会话
    final session = await AudioSession.instance;
    await session.configure(const AudioSessionConfiguration.music());
    
    // 创建播放列表
    _playlist = ConcatenatingAudioSource(
      children: widget.playlist.map((url) => AudioSource.uri(Uri.parse(url))).toList(),
    );
    
    try {
      await _player.setAudioSource(_playlist);
    } catch (e) {
      print('加载音频失败: $e');
    }
  }
  
  @override
  Widget build(BuildContext context) {
    return StreamBuilder<PlayerState>(
      stream: _player.playerStateStream,
      builder: (context, snapshot) {
        final playerState = snapshot.data;
        final processingState = playerState?.processingState;
        final playing = playerState?.playing;
        
        return Column(
          children: [
            // 播放控制
            Row(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                IconButton(
                  icon: const Icon(Icons.skip_previous),
                  onPressed: _player.hasPrevious ? _player.seekToPrevious : null,
                ),
                if (processingState == ProcessingState.loading ||
                    processingState == ProcessingState.buffering)
                  const CircularProgressIndicator()
                else if (playing != true)
                  IconButton(
                    icon: const Icon(Icons.play_arrow, size: 64),
                    onPressed: _player.play,
                  )
                else
                  IconButton(
                    icon: const Icon(Icons.pause, size: 64),
                    onPressed: _player.pause,
                  ),
                IconButton(
                  icon: const Icon(Icons.skip_next),
                  onPressed: _player.hasNext ? _player.seekToNext : null,
                ),
              ],
            ),
            
            // 进度条
            StreamBuilder<Duration>(
              stream: _player.positionStream,
              builder: (context, snapshot) {
                final position = snapshot.data ?? Duration.zero;
                final duration = _player.duration ?? Duration.zero;
                
                return Column(
                  children: [
                    Slider(
                      value: position.inMilliseconds.toDouble(),
                      max: duration.inMilliseconds.toDouble(),
                      onChanged: (value) {
                        _player.seek(Duration(milliseconds: value.toInt()));
                      },
                    ),
                    Row(
                      mainAxisAlignment: MainAxisAlignment.spaceBetween,
                      children: [
                        Text(_formatDuration(position)),
                        Text(_formatDuration(duration)),
                      ],
                    ),
                  ],
                );
              },
            ),
            
            // 播放模式控制
            Row(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                StreamBuilder<LoopMode>(
                  stream: _player.loopModeStream,
                  builder: (context, snapshot) {
                    final loopMode = snapshot.data ?? LoopMode.off;
                    const icons = [
                      Icon(Icons.repeat, color: Colors.grey),
                      Icon(Icons.repeat, color: Colors.orange),
                      Icon(Icons.repeat_one, color: Colors.orange),
                    ];
                    const cycleModes = [
                      LoopMode.off,
                      LoopMode.all,
                      LoopMode.one,
                    ];
                    final index = cycleModes.indexOf(loopMode);
                    return IconButton(
                      icon: icons[index],
                      onPressed: () {
                        _player.setLoopMode(
                          cycleModes[(index + 1) % cycleModes.length],
                        );
                      },
                    );
                  },
                ),
                StreamBuilder<bool>(
                  stream: _player.shuffleModeEnabledStream,
                  builder: (context, snapshot) {
                    final shuffleModeEnabled = snapshot.data ?? false;
                    return IconButton(
                      icon: Icon(
                        Icons.shuffle,
                        color: shuffleModeEnabled ? Colors.orange : Colors.grey,
                      ),
                      onPressed: () {
                        _player.setShuffleModeEnabled(!shuffleModeEnabled);
                      },
                    );
                  },
                ),
              ],
            ),
          ],
        );
      },
    );
  }
  
  String _formatDuration(Duration duration) {
    String twoDigits(int n) => n.toString().padLeft(2, '0');
    final minutes = twoDigits(duration.inMinutes.remainder(60));
    final seconds = twoDigits(duration.inSeconds.remainder(60));
    return '$minutes:$seconds';
  }
  
  @override
  void dispose() {
    _player.dispose();
    super.dispose();
  }
}
```

## 3. 录制功能

### 3.1 camera - 相机录制

```yaml
# pubspec.yaml
dependencies:
  camera: ^0.10.5+5
  path_provider: ^2.1.1
```

```dart
// 相机录制
import 'package:camera/camera.dart';
import 'package:path_provider/path_provider.dart';
import 'dart:io';

class CameraRecorderWidget extends StatefulWidget {
  @override
  _CameraRecorderWidgetState createState() => _CameraRecorderWidgetState();
}

class _CameraRecorderWidgetState extends State<CameraRecorderWidget> {
  CameraController? _controller;
  List<CameraDescription>? _cameras;
  bool _isRecording = false;
  String? _videoPath;
  
  @override
  void initState() {
    super.initState();
    _initializeCamera();
  }
  
  Future<void> _initializeCamera() async {
    _cameras = await availableCameras();
    if (_cameras!.isNotEmpty) {
      _controller = CameraController(
        _cameras![0],
        ResolutionPreset.high,
        enableAudio: true,
      );
      
      await _controller!.initialize();
      setState(() {});
    }
  }
  
  Future<void> _startRecording() async {
    if (_controller == null || !_controller!.value.isInitialized) return;
    
    try {
      await _controller!.startVideoRecording();
      setState(() {
        _isRecording = true;
      });
    } catch (e) {
      print('开始录制失败: $e');
    }
  }
  
  Future<void> _stopRecording() async {
    if (_controller == null || !_controller!.value.isRecordingVideo) return;
    
    try {
      final XFile videoFile = await _controller!.stopVideoRecording();
      setState(() {
        _isRecording = false;
        _videoPath = videoFile.path;
      });
      
      print('视频保存到: ${videoFile.path}');
    } catch (e) {
      print('停止录制失败: $e');
    }
  }
  
  @override
  Widget build(BuildContext context) {
    if (_controller == null || !_controller!.value.isInitialized) {
      return const Center(child: CircularProgressIndicator());
    }
    
    return Scaffold(
      body: Stack(
        children: [
          CameraPreview(_controller!),
          Positioned(
            bottom: 50,
            left: 0,
            right: 0,
            child: Row(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                FloatingActionButton(
                  onPressed: _isRecording ? _stopRecording : _startRecording,
                  backgroundColor: _isRecording ? Colors.red : Colors.blue,
                  child: Icon(
                    _isRecording ? Icons.stop : Icons.videocam,
                  ),
                ),
              ],
            ),
          ),
          if (_videoPath != null)
            Positioned(
              top: 50,
              right: 20,
              child: Container(
                padding: const EdgeInsets.all(8),
                decoration: BoxDecoration(
                  color: Colors.black54,
                  borderRadius: BorderRadius.circular(8),
                ),
                child: Text(
                  '视频已保存',
                  style: const TextStyle(color: Colors.white),
                ),
              ),
            ),
        ],
      ),
    );
  }
  
  @override
  void dispose() {
    _controller?.dispose();
    super.dispose();
  }
}
```

### 3.2 record - 音频录制

```yaml
# pubspec.yaml
dependencies:
  record: ^5.0.4
  permission_handler: ^11.0.1
```

```dart
// 音频录制
import 'package:record/record.dart';
import 'package:permission_handler/permission_handler.dart';
import 'package:path_provider/path_provider.dart';
import 'dart:io';

class AudioRecorderWidget extends StatefulWidget {
  @override
  _AudioRecorderWidgetState createState() => _AudioRecorderWidgetState();
}

class _AudioRecorderWidgetState extends State<AudioRecorderWidget> {
  late AudioRecorder _recorder;
  bool _isRecording = false;
  String? _audioPath;
  Duration _recordDuration = Duration.zero;
  Timer? _timer;
  
  @override
  void initState() {
    super.initState();
    _recorder = AudioRecorder();
  }
  
  Future<void> _startRecording() async {
    // 检查权限
    if (await Permission.microphone.request() != PermissionStatus.granted) {
      print('麦克风权限被拒绝');
      return;
    }
    
    try {
      final directory = await getApplicationDocumentsDirectory();
      final path = '${directory.path}/audio_${DateTime.now().millisecondsSinceEpoch}.m4a';
      
      await _recorder.start(
        const RecordConfig(
          encoder: AudioEncoder.aacLc,
          bitRate: 128000,
          sampleRate: 44100,
        ),
        path: path,
      );
      
      setState(() {
        _isRecording = true;
        _recordDuration = Duration.zero;
      });
      
      _startTimer();
    } catch (e) {
      print('开始录制失败: $e');
    }
  }
  
  Future<void> _stopRecording() async {
    try {
      final path = await _recorder.stop();
      
      setState(() {
        _isRecording = false;
        _audioPath = path;
      });
      
      _stopTimer();
      
      if (path != null) {
        print('音频保存到: $path');
      }
    } catch (e) {
      print('停止录制失败: $e');
    }
  }
  
  void _startTimer() {
    _timer = Timer.periodic(const Duration(seconds: 1), (timer) {
      setState(() {
        _recordDuration = Duration(seconds: timer.tick);
      });
    });
  }
  
  void _stopTimer() {
    _timer?.cancel();
    _timer = null;
  }
  
  String _formatDuration(Duration duration) {
    String twoDigits(int n) => n.toString().padLeft(2, '0');
    final minutes = twoDigits(duration.inMinutes.remainder(60));
    final seconds = twoDigits(duration.inSeconds.remainder(60));
    return '$minutes:$seconds';
  }
  
  @override
  Widget build(BuildContext context) {
    return Card(
      child: Padding(
        padding: const EdgeInsets.all(16.0),
        child: Column(
          mainAxisSize: MainAxisSize.min,
          children: [
            Icon(
              _isRecording ? Icons.mic : Icons.mic_none,
              size: 64,
              color: _isRecording ? Colors.red : Colors.grey,
            ),
            const SizedBox(height: 16),
            Text(
              _formatDuration(_recordDuration),
              style: Theme.of(context).textTheme.headlineSmall,
            ),
            const SizedBox(height: 16),
            Row(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                ElevatedButton.icon(
                  onPressed: _isRecording ? _stopRecording : _startRecording,
                  icon: Icon(
                    _isRecording ? Icons.stop : Icons.fiber_manual_record,
                  ),
                  label: Text(_isRecording ? '停止录制' : '开始录制'),
                  style: ElevatedButton.styleFrom(
                    backgroundColor: _isRecording ? Colors.red : Colors.blue,
                    foregroundColor: Colors.white,
                  ),
                ),
              ],
            ),
            if (_audioPath != null) ..[
              const SizedBox(height: 16),
              Container(
                padding: const EdgeInsets.all(8),
                decoration: BoxDecoration(
                  color: Colors.green.withOpacity(0.1),
                  borderRadius: BorderRadius.circular(8),
                  border: Border.all(color: Colors.green),
                ),
                child: Row(
                  children: [
                    const Icon(Icons.check_circle, color: Colors.green),
                    const SizedBox(width: 8),
                    const Expanded(
                      child: Text('录制完成'),
                    ),
                    TextButton(
                      onPressed: () {
                        // 播放录制的音频
                        print('播放音频: $_audioPath');
                      },
                      child: const Text('播放'),
                    ),
                  ],
                ),
              ),
            ],
          ],
        ),
      ),
    );
  }
  
  @override
  void dispose() {
    _recorder.dispose();
    _stopTimer();
    super.dispose();
  }
}
```

## 4. 流媒体组件

### 4.1 WebRTC 实时通信

```yaml
# pubspec.yaml
dependencies:
  flutter_webrtc: ^0.9.48
```

```dart
// WebRTC 视频通话
import 'package:flutter_webrtc/flutter_webrtc.dart';

class WebRTCWidget extends StatefulWidget {
  @override
  _WebRTCWidgetState createState() => _WebRTCWidgetState();
}

class _WebRTCWidgetState extends State<WebRTCWidget> {
  RTCPeerConnection? _peerConnection;
  MediaStream? _localStream;
  RTCVideoRenderer _localRenderer = RTCVideoRenderer();
  RTCVideoRenderer _remoteRenderer = RTCVideoRenderer();
  bool _inCalling = false;
  
  @override
  void initState() {
    super.initState();
    _initRenderers();
  }
  
  Future<void> _initRenderers() async {
    await _localRenderer.initialize();
    await _remoteRenderer.initialize();
  }
  
  Future<void> _makeCall() async {
    final configuration = {
      'iceServers': [
        {'urls': 'stun:stun.l.google.com:19302'},
      ]
    };
    
    _peerConnection = await createPeerConnection(configuration);
    
    _peerConnection!.onIceCandidate = (candidate) {
      // 发送 ICE candidate 到远程端
      print('ICE candidate: ${candidate.toMap()}');
    };
    
    _peerConnection!.onAddStream = (stream) {
      _remoteRenderer.srcObject = stream;
      setState(() {});
    };
    
    // 获取本地媒体流
    _localStream = await navigator.mediaDevices.getUserMedia({
      'video': true,
      'audio': true,
    });
    
    _localRenderer.srcObject = _localStream;
    _peerConnection!.addStream(_localStream!);
    
    // 创建 offer
    RTCSessionDescription offer = await _peerConnection!.createOffer();
    await _peerConnection!.setLocalDescription(offer);
    
    setState(() {
      _inCalling = true;
    });
    
    // 发送 offer 到远程端
    print('Offer: ${offer.toMap()}');
  }
  
  Future<void> _hangUp() async {
    await _localStream?.dispose();
    await _peerConnection?.close();
    
    _localRenderer.srcObject = null;
    _remoteRenderer.srcObject = null;
    
    setState(() {
      _inCalling = false;
    });
  }
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('WebRTC 视频通话')),
      body: Column(
        children: [
          Expanded(
            child: Row(
              children: [
                Expanded(
                  child: Container(
                    decoration: BoxDecoration(
                      border: Border.all(color: Colors.grey),
                    ),
                    child: RTCVideoView(_localRenderer, mirror: true),
                  ),
                ),
                Expanded(
                  child: Container(
                    decoration: BoxDecoration(
                      border: Border.all(color: Colors.grey),
                    ),
                    child: RTCVideoView(_remoteRenderer),
                  ),
                ),
              ],
            ),
          ),
          Padding(
            padding: const EdgeInsets.all(16.0),
            child: Row(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                FloatingActionButton(
                  onPressed: _inCalling ? _hangUp : _makeCall,
                  backgroundColor: _inCalling ? Colors.red : Colors.green,
                  child: Icon(
                    _inCalling ? Icons.call_end : Icons.call,
                  ),
                ),
              ],
            ),
          ),
        ],
      ),
    );
  }
  
  @override
  void dispose() {
    _localRenderer.dispose();
    _remoteRenderer.dispose();
    _localStream?.dispose();
    _peerConnection?.dispose();
    super.dispose();
  }
}
```

## 5. 最佳实践

### 5.1 性能优化

```dart
// 视频播放器优化
class OptimizedVideoPlayer extends StatefulWidget {
  final String videoUrl;
  
  const OptimizedVideoPlayer({Key? key, required this.videoUrl}) : super(key: key);
  
  @override
  _OptimizedVideoPlayerState createState() => _OptimizedVideoPlayerState();
}

class _OptimizedVideoPlayerState extends State<OptimizedVideoPlayer>
    with AutomaticKeepAliveClientMixin {
  late VideoPlayerController _controller;
  bool _isInitialized = false;
  
  @override
  bool get wantKeepAlive => true;
  
  @override
  void initState() {
    super.initState();
    _initializePlayer();
  }
  
  Future<void> _initializePlayer() async {
    _controller = VideoPlayerController.networkUrl(
      Uri.parse(widget.videoUrl),
      videoPlayerOptions: VideoPlayerOptions(
        mixWithOthers: true, // 允许与其他音频混合
        allowBackgroundPlayback: false, // 禁止后台播放
      ),
    );
    
    await _controller.initialize();
    
    setState(() {
      _isInitialized = true;
    });
  }
  
  @override
  Widget build(BuildContext context) {
    super.build(context);
    
    if (!_isInitialized) {
      return const Center(child: CircularProgressIndicator());
    }
    
    return AspectRatio(
      aspectRatio: _controller.value.aspectRatio,
      child: VideoPlayer(_controller),
    );
  }
  
  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }
}
```

### 5.2 错误处理

```dart
// 音频播放错误处理
class RobustAudioPlayer {
  late AudioPlayer _player;
  StreamSubscription? _playerStateSubscription;
  
  Future<void> initialize() async {
    _player = AudioPlayer();
    
    _playerStateSubscription = _player.playerStateStream.listen(
      (state) {
        if (state.processingState == ProcessingState.error) {
          _handleError('播放错误');
        }
      },
      onError: (error) {
        _handleError('流错误: $error');
      },
    );
  }
  
  Future<void> playWithRetry(String url, {int maxRetries = 3}) async {
    for (int i = 0; i < maxRetries; i++) {
      try {
        await _player.setUrl(url);
        await _player.play();
        return;
      } catch (e) {
        print('播放尝试 ${i + 1} 失败: $e');
        if (i == maxRetries - 1) {
          _handleError('播放失败，已达到最大重试次数');
        }
        await Future.delayed(Duration(seconds: 1 << i)); // 指数退避
      }
    }
  }
  
  void _handleError(String message) {
    print('音频播放器错误: $message');
    // 可以在这里添加用户通知逻辑
  }
  
  void dispose() {
    _playerStateSubscription?.cancel();
    _player.dispose();
  }
}
```

### 5.3 内存管理

```dart
// 视频缓存管理
class VideoCache {
  static final Map<String, VideoPlayerController> _cache = {};
  static const int maxCacheSize = 5;
  
  static VideoPlayerController? getController(String url) {
    return _cache[url];
  }
  
  static Future<VideoPlayerController> getOrCreateController(String url) async {
    if (_cache.containsKey(url)) {
      return _cache[url]!;
    }
    
    // 如果缓存已满，移除最旧的控制器
    if (_cache.length >= maxCacheSize) {
      final oldestKey = _cache.keys.first;
      final oldestController = _cache.remove(oldestKey);
      await oldestController?.dispose();
    }
    
    final controller = VideoPlayerController.networkUrl(Uri.parse(url));
    await controller.initialize();
    _cache[url] = controller;
    
    return controller;
  }
  
  static Future<void> clearCache() async {
    for (final controller in _cache.values) {
      await controller.dispose();
    }
    _cache.clear();
  }
}
```

## 6. 平台配置

### 6.1 Android 配置

```xml
<!-- android/app/src/main/AndroidManifest.xml -->
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.RECORD_AUDIO" />
<uses-permission android:name="android.permission.CAMERA" />
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />

<application
    android:usesCleartextTraffic="true">
    <!-- 其他配置 -->
</application>
```

### 6.2 iOS 配置

```xml
<!-- ios/Runner/Info.plist -->
<key>NSCameraUsageDescription</key>
<string>此应用需要访问相机来录制视频</string>
<key>NSMicrophoneUsageDescription</key>
<string>此应用需要访问麦克风来录制音频</string>
<key>NSPhotoLibraryUsageDescription</key>
<string>此应用需要访问相册来保存媒体文件</string>
```

## 7. 总结

视频音频组件是现代移动应用的重要组成部分，本文档涵盖了：

- **视频播放**: video_player 和 chewie 的使用
- **音频播放**: audioplayers 和 just_audio 的集成
- **录制功能**: 相机录制和音频录制的实现
- **流媒体**: WebRTC 实时通信的基础用法
- **最佳实践**: 性能优化、错误处理和内存管理

选择合适的组件库，并根据项目需求进行定制化开发，可以为用户提供优秀的多媒体体验。