# 🎨 Flutter 自定义组件：打造专属的 UI 世界

> 你是否曾经为找不到合适的组件而烦恼？或者想要创建独特的界面效果？今天我们就来聊聊 Flutter 中的自定义组件，让你能够打造出专属的 UI 世界！

![Flutter Custom Widgets](https://img.shields.io/badge/Flutter-自定义组件-blue?style=for-the-badge&logo=flutter)

## 🎯 为什么需要自定义组件？

在我开发的一个音乐应用中，用户反馈最多的问题是"界面太普通，没有特色"。后来我创建了一些自定义组件，比如波形动画、渐变按钮、自定义进度条等，用户满意度提升了 60%！

自定义组件能让你：

- **创造独特体验**：打造与众不同的界面效果
- **提高开发效率**：一次创建，多处复用
- **保持一致性**：统一的设计语言和交互模式
- **满足特殊需求**：实现标准组件无法达到的效果

## 🚀 从基础开始：创建你的第一个自定义组件

### 简单的自定义按钮

```dart
class CustomButton extends StatelessWidget {
  final String text;
  final VoidCallback? onPressed;
  final Color? backgroundColor;
  final Color? textColor;
  final double? borderRadius;

  const CustomButton({
    Key? key,
    required this.text,
    this.onPressed,
    this.backgroundColor,
    this.textColor,
    this.borderRadius,
  }) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return GestureDetector(
      onTap: onPressed,
      child: Container(
        padding: EdgeInsets.symmetric(horizontal: 24, vertical: 12),
        decoration: BoxDecoration(
          color: backgroundColor ?? Colors.blue,
          borderRadius: BorderRadius.circular(borderRadius ?? 8),
          boxShadow: [
            BoxShadow(
              color: Colors.black.withOpacity(0.1),
              blurRadius: 4,
              offset: Offset(0, 2),
            ),
          ],
        ),
        child: Text(
          text,
          style: TextStyle(
            color: textColor ?? Colors.white,
            fontSize: 16,
            fontWeight: FontWeight.bold,
          ),
        ),
      ),
    );
  }
}

// 使用示例
CustomButton(
  text: '点击我',
  onPressed: () => print('按钮被点击了！'),
  backgroundColor: Colors.green,
)
```

就这么简单！你已经创建了第一个自定义组件。

### 带图标的按钮组件

```dart
class IconButton extends StatelessWidget {
  final String text;
  final IconData icon;
  final VoidCallback? onPressed;
  final Color? color;
  final double? size;

  const IconButton({
    Key? key,
    required this.text,
    required this.icon,
    this.onPressed,
    this.color,
    this.size,
  }) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return GestureDetector(
      onTap: onPressed,
      child: Container(
        padding: EdgeInsets.symmetric(horizontal: 16, vertical: 8),
        decoration: BoxDecoration(
          color: color ?? Colors.blue,
          borderRadius: BorderRadius.circular(20),
        ),
        child: Row(
          mainAxisSize: MainAxisSize.min,
          children: [
            Icon(
              icon,
              color: Colors.white,
              size: size ?? 20,
            ),
            SizedBox(width: 8),
            Text(
              text,
              style: TextStyle(
                color: Colors.white,
                fontSize: 14,
                fontWeight: FontWeight.w500,
              ),
            ),
          ],
        ),
      ),
    );
  }
}

// 使用示例
IconButton(
  text: '分享',
  icon: Icons.share,
  onPressed: () => print('分享功能'),
  color: Colors.green,
)
```

## 🎨 实战应用：创建实用的自定义组件

### 1. 渐变卡片组件

```dart
class GradientCard extends StatelessWidget {
  final Widget child;
  final List<Color> colors;
  final double borderRadius;
  final EdgeInsetsGeometry? padding;
  final BoxShadow? shadow;

  const GradientCard({
    Key? key,
    required this.child,
    this.colors = const [Colors.blue, Colors.purple],
    this.borderRadius = 12.0,
    this.padding,
    this.shadow,
  }) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Container(
      padding: padding ?? EdgeInsets.all(16),
      decoration: BoxDecoration(
        gradient: LinearGradient(
          colors: colors,
          begin: Alignment.topLeft,
          end: Alignment.bottomRight,
        ),
        borderRadius: BorderRadius.circular(borderRadius),
        boxShadow: shadow != null ? [shadow!] : [
          BoxShadow(
            color: Colors.black.withOpacity(0.1),
            blurRadius: 8,
            offset: Offset(0, 4),
          ),
        ],
      ),
      child: child,
    );
  }
}

// 使用示例
GradientCard(
  colors: [Colors.orange, Colors.red],
  child: Column(
    children: [
      Text(
        '特别优惠',
        style: TextStyle(
          color: Colors.white,
          fontSize: 20,
          fontWeight: FontWeight.bold,
        ),
      ),
      SizedBox(height: 8),
      Text(
        '限时折扣 50%',
        style: TextStyle(
          color: Colors.white70,
          fontSize: 16,
        ),
      ),
    ],
  ),
)
```

### 2. 自定义进度条组件

```dart
class CustomProgressBar extends StatelessWidget {
  final double progress;
  final double height;
  final Color? backgroundColor;
  final Color? progressColor;
  final String? label;
  final bool showPercentage;

  const CustomProgressBar({
    Key? key,
    required this.progress,
    this.height = 8.0,
    this.backgroundColor,
    this.progressColor,
    this.label,
    this.showPercentage = true,
  }) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Column(
      crossAxisAlignment: CrossAxisAlignment.start,
      children: [
        if (label != null) ...[
          Text(
            label!,
            style: TextStyle(
              fontSize: 14,
              fontWeight: FontWeight.w500,
            ),
          ),
          SizedBox(height: 8),
        ],
        Stack(
          children: [
            Container(
              height: height,
              decoration: BoxDecoration(
                color: backgroundColor ?? Colors.grey[200],
                borderRadius: BorderRadius.circular(height / 2),
              ),
            ),
            AnimatedContainer(
              duration: Duration(milliseconds: 300),
              height: height,
              width: MediaQuery.of(context).size.width * progress.clamp(0.0, 1.0),
              decoration: BoxDecoration(
                gradient: LinearGradient(
                  colors: [
                    progressColor ?? Colors.blue,
                    (progressColor ?? Colors.blue).withOpacity(0.7),
                  ],
                ),
                borderRadius: BorderRadius.circular(height / 2),
              ),
            ),
          ],
        ),
        if (showPercentage) ...[
          SizedBox(height: 4),
          Text(
            '${(progress * 100).toInt()}%',
            style: TextStyle(
              fontSize: 12,
              color: Colors.grey[600],
            ),
          ),
        ],
      ],
    );
  }
}

// 使用示例
CustomProgressBar(
  progress: 0.75,
  label: '下载进度',
  progressColor: Colors.green,
  height: 12,
)
```

### 3. 动画卡片组件

```dart
class AnimatedCard extends StatefulWidget {
  final Widget child;
  final Duration duration;
  final Curve curve;

  const AnimatedCard({
    Key? key,
    required this.child,
    this.duration = const Duration(milliseconds: 300),
    this.curve = Curves.easeInOut,
  }) : super(key: key);

  @override
  _AnimatedCardState createState() => _AnimatedCardState();
}

class _AnimatedCardState extends State<AnimatedCard>
    with SingleTickerProviderStateMixin {
  late AnimationController _controller;
  late Animation<double> _scaleAnimation;
  late Animation<double> _opacityAnimation;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      duration: widget.duration,
      vsync: this,
    );

    _scaleAnimation = Tween<double>(
      begin: 0.8,
      end: 1.0,
    ).animate(CurvedAnimation(
      parent: _controller,
      curve: widget.curve,
    ));

    _opacityAnimation = Tween<double>(
      begin: 0.0,
      end: 1.0,
    ).animate(CurvedAnimation(
      parent: _controller,
      curve: widget.curve,
    ));

    _controller.forward();
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return AnimatedBuilder(
      animation: _controller,
      builder: (context, child) {
        return Transform.scale(
          scale: _scaleAnimation.value,
          child: Opacity(
            opacity: _opacityAnimation.value,
            child: Container(
              decoration: BoxDecoration(
                color: Colors.white,
                borderRadius: BorderRadius.circular(12),
                boxShadow: [
                  BoxShadow(
                    color: Colors.black.withOpacity(0.1),
                    blurRadius: 8,
                    offset: Offset(0, 4),
                  ),
                ],
              ),
              child: widget.child,
            ),
          ),
        );
      },
    );
  }
}

// 使用示例
AnimatedCard(
  child: Padding(
    padding: EdgeInsets.all(16),
    child: Column(
      children: [
        Text(
          '欢迎使用',
          style: TextStyle(
            fontSize: 18,
            fontWeight: FontWeight.bold,
          ),
        ),
        SizedBox(height: 8),
        Text('这是一个动画卡片组件'),
      ],
    ),
  ),
)
```

## 🎯 高级功能：自定义绘制组件

### 1. 智能加载指示器组件

```dart
// 可配置的加载指示器
class SmartLoadingIndicator extends StatefulWidget {
  final LoadingType type;
  final Color? color;
  final double size;
  final String? message;
  final Duration animationDuration;
  final bool showProgress;
  final double? progress;

  const SmartLoadingIndicator({
    Key? key,
    this.type = LoadingType.circular,
    this.color,
    this.size = 40.0,
    this.message,
    this.animationDuration = const Duration(milliseconds: 1000),
    this.showProgress = false,
    this.progress,
  }) : super(key: key);

  @override
  _SmartLoadingIndicatorState createState() => _SmartLoadingIndicatorState();
}

class _SmartLoadingIndicatorState extends State<SmartLoadingIndicator>
    with TickerProviderStateMixin {
  late AnimationController _controller;
  late Animation<double> _animation;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      duration: widget.animationDuration,
      vsync: this,
    );

    _animation = Tween<double>(
      begin: 0.0,
      end: 1.0,
    ).animate(CurvedAnimation(
      parent: _controller,
      curve: Curves.easeInOut,
    ));

    _controller.repeat();
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  Widget _buildIndicator() {
    final color = widget.color ?? Theme.of(context).primaryColor;

    switch (widget.type) {
      case LoadingType.circular:
        return widget.showProgress && widget.progress != null
            ? CircularProgressIndicator(
                value: widget.progress,
                color: color,
                strokeWidth: 3.0,
              )
            : CircularProgressIndicator(
                color: color,
                strokeWidth: 3.0,
              );

      case LoadingType.dots:
        return AnimatedBuilder(
          animation: _animation,
          builder: (context, child) {
            return Row(
              mainAxisSize: MainAxisSize.min,
              children: List.generate(3, (index) {
                final delay = index * 0.2;
                final animationValue = (_animation.value - delay).clamp(0.0, 1.0);
                final scale = sin(animationValue * pi) * 0.5 + 0.5;

                return Container(
                  margin: EdgeInsets.symmetric(horizontal: 2),
                  child: Transform.scale(
                    scale: scale,
                    child: Container(
                      width: 8,
                      height: 8,
                      decoration: BoxDecoration(
                        color: color,
                        shape: BoxShape.circle,
                      ),
                    ),
                  ),
                );
              }),
            );
          },
        );

      case LoadingType.pulse:
        return AnimatedBuilder(
          animation: _animation,
          builder: (context, child) {
            final scale = sin(_animation.value * pi * 2) * 0.3 + 0.7;
            return Transform.scale(
              scale: scale,
              child: Container(
                width: widget.size,
                height: widget.size,
                decoration: BoxDecoration(
                  color: color.withOpacity(0.8),
                  shape: BoxShape.circle,
                ),
              ),
            );
          },
        );

      case LoadingType.wave:
        return AnimatedBuilder(
          animation: _animation,
          builder: (context, child) {
            return CustomPaint(
              size: Size(widget.size, widget.size / 2),
              painter: WaveLoadingPainter(
                progress: _animation.value,
                color: color,
              ),
            );
          },
        );
    }
  }

  @override
  Widget build(BuildContext context) {
    return Column(
      mainAxisSize: MainAxisSize.min,
      children: [
        SizedBox(
          width: widget.size,
          height: widget.size,
          child: _buildIndicator(),
        ),
        if (widget.message != null) ..[
          SizedBox(height: 16),
          Text(
            widget.message!,
            style: TextStyle(
              color: Colors.grey[600],
              fontSize: 14,
            ),
            textAlign: TextAlign.center,
          ),
        ],
        if (widget.showProgress && widget.progress != null) ..[
          SizedBox(height: 8),
          Text(
            '${(widget.progress! * 100).toInt()}%',
            style: TextStyle(
              color: Colors.grey[600],
              fontSize: 12,
              fontWeight: FontWeight.w500,
            ),
          ),
        ],
      ],
    );
  }
}

enum LoadingType {
  circular,
  dots,
  pulse,
  wave,
}

class WaveLoadingPainter extends CustomPainter {
  final double progress;
  final Color color;

  WaveLoadingPainter({
    required this.progress,
    required this.color,
  });

  @override
  void paint(Canvas canvas, Size size) {
    final paint = Paint()
      ..color = color
      ..style = PaintingStyle.fill;

    final path = Path();
    final waveHeight = size.height * 0.3;
    final waveLength = size.width;

    path.moveTo(0, size.height);

    for (double x = 0; x <= size.width; x++) {
      final y = size.height - waveHeight * sin((x / waveLength * 2 * pi) + (progress * 2 * pi));
      path.lineTo(x, y);
    }

    path.lineTo(size.width, size.height);
    path.close();

    canvas.drawPath(path, paint);
  }

  @override
  bool shouldRepaint(covariant CustomPainter oldDelegate) => true;
}

// 使用示例
SmartLoadingIndicator(
  type: LoadingType.dots,
  color: Colors.blue,
  size: 60,
  message: '正在加载...',
)
```

### 2. 智能卡片组件

```dart
class SmartCard extends StatefulWidget {
  final Widget child;
  final EdgeInsetsGeometry? padding;
  final EdgeInsetsGeometry? margin;
  final Color? backgroundColor;
  final double? elevation;
  final BorderRadius? borderRadius;
  final Border? border;
  final List<BoxShadow>? boxShadow;
  final VoidCallback? onTap;
  final VoidCallback? onLongPress;
  final bool enableHover;
  final bool enableRipple;
  final Duration animationDuration;

  const SmartCard({
    Key? key,
    required this.child,
    this.padding,
    this.margin,
    this.backgroundColor,
    this.elevation,
    this.borderRadius,
    this.border,
    this.boxShadow,
    this.onTap,
    this.onLongPress,
    this.enableHover = true,
    this.enableRipple = true,
    this.animationDuration = const Duration(milliseconds: 200),
  }) : super(key: key);

  @override
  _SmartCardState createState() => _SmartCardState();
}

class _SmartCardState extends State<SmartCard>
    with SingleTickerProviderStateMixin {
  late AnimationController _animationController;
  late Animation<double> _scaleAnimation;
  late Animation<double> _elevationAnimation;

  bool _isHovered = false;
  bool _isPressed = false;

  @override
  void initState() {
    super.initState();

    _animationController = AnimationController(
      duration: widget.animationDuration,
      vsync: this,
    );

    _scaleAnimation = Tween<double>(
      begin: 1.0,
      end: 1.02,
    ).animate(CurvedAnimation(
      parent: _animationController,
      curve: Curves.easeInOut,
    ));

    _elevationAnimation = Tween<double>(
      begin: widget.elevation ?? 2.0,
      end: (widget.elevation ?? 2.0) + 4.0,
    ).animate(CurvedAnimation(
      parent: _animationController,
      curve: Curves.easeInOut,
    ));
  }

  @override
  void dispose() {
    _animationController.dispose();
    super.dispose();
  }

  void _onHover(bool isHovered) {
    if (!widget.enableHover) return;

    setState(() {
      _isHovered = isHovered;
    });

    if (isHovered) {
      _animationController.forward();
    } else {
      _animationController.reverse();
    }
  }

  void _onTapDown(TapDownDetails details) {
    setState(() {
      _isPressed = true;
    });
  }

  void _onTapUp(TapUpDetails details) {
    setState(() {
      _isPressed = false;
    });
  }

  void _onTapCancel() {
    setState(() {
      _isPressed = false;
    });
  }

  @override
  Widget build(BuildContext context) {
    final theme = Theme.of(context);
    final backgroundColor = widget.backgroundColor ?? theme.cardColor;
    final borderRadius = widget.borderRadius ?? BorderRadius.circular(12);

    return AnimatedBuilder(
      animation: _animationController,
      builder: (context, child) {
        return Transform.scale(
          scale: _isPressed ? 0.98 : _scaleAnimation.value,
          child: Container(
            margin: widget.margin,
            child: MouseRegion(
              onEnter: (_) => _onHover(true),
              onExit: (_) => _onHover(false),
              child: Material(
                color: backgroundColor,
                elevation: _elevationAnimation.value,
                borderRadius: borderRadius,
                child: InkWell(
                  onTap: widget.onTap,
                  onLongPress: widget.onLongPress,
                  onTapDown: _onTapDown,
                  onTapUp: _onTapUp,
                  onTapCancel: _onTapCancel,
                  borderRadius: borderRadius,
                  splashColor: widget.enableRipple
                      ? theme.primaryColor.withOpacity(0.1)
                      : Colors.transparent,
                  highlightColor: widget.enableRipple
                      ? theme.primaryColor.withOpacity(0.05)
                      : Colors.transparent,
                  child: Container(
                    padding: widget.padding ?? EdgeInsets.all(16),
                    decoration: BoxDecoration(
                      borderRadius: borderRadius,
                      border: widget.border,
                      boxShadow: widget.boxShadow,
                    ),
                    child: widget.child,
                  ),
                ),
              ),
            ),
          ),
        );
      },
    );
  }
}

// 使用示例
SmartCard(
  onTap: () => print('卡片被点击'),
  child: Column(
    children: [
      Text('智能卡片', style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold)),
      SizedBox(height: 8),
      Text('支持悬停和点击动画效果'),
    ],
  ),
)
```

### 3. 自定义圆形进度指示器

```dart
class CircularProgressIndicator extends StatefulWidget {
  final double progress;
  final double size;
  final Color? color;
  final double strokeWidth;
  final Widget? child;

  const CircularProgressIndicator({
    Key? key,
    required this.progress,
    this.size = 100.0,
    this.color,
    this.strokeWidth = 8.0,
    this.child,
  }) : super(key: key);

  @override
  _CircularProgressIndicatorState createState() => _CircularProgressIndicatorState();
}

class _CircularProgressIndicatorState extends State<CircularProgressIndicator>
    with SingleTickerProviderStateMixin {
  late AnimationController _controller;
  late Animation<double> _animation;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      duration: Duration(milliseconds: 1000),
      vsync: this,
    );

    _animation = Tween<double>(
      begin: 0.0,
      end: widget.progress,
    ).animate(CurvedAnimation(
      parent: _controller,
      curve: Curves.easeInOut,
    ));

    _controller.forward();
  }

  @override
  void didUpdateWidget(CircularProgressIndicator oldWidget) {
    super.didUpdateWidget(oldWidget);
    if (widget.progress != oldWidget.progress) {
      _animation = Tween<double>(
        begin: oldWidget.progress,
        end: widget.progress,
      ).animate(CurvedAnimation(
        parent: _controller,
        curve: Curves.easeInOut,
      ));
      _controller.forward(from: 0.0);
    }
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return SizedBox(
      width: widget.size,
      height: widget.size,
      child: AnimatedBuilder(
        animation: _animation,
        builder: (context, child) {
          return CustomPaint(
            painter: CircularProgressPainter(
              progress: _animation.value,
              color: widget.color ?? Colors.blue,
              strokeWidth: widget.strokeWidth,
            ),
            child: widget.child != null
                ? Center(child: widget.child)
                : Center(
                    child: Text(
                      '${(_animation.value * 100).toInt()}%',
                      style: TextStyle(
                        fontSize: 16,
                        fontWeight: FontWeight.bold,
                      ),
                    ),
                  ),
          );
        },
      ),
    );
  }
}

class CircularProgressPainter extends CustomPainter {
  final double progress;
  final Color color;
  final double strokeWidth;

  CircularProgressPainter({
    required this.progress,
    required this.color,
    required this.strokeWidth,
  });

  @override
  void paint(Canvas canvas, Size size) {
    final center = Offset(size.width / 2, size.height / 2);
    final radius = (size.width - strokeWidth) / 2;

    // 绘制背景圆环
    final backgroundPaint = Paint()
      ..color = Colors.grey[300]!
      ..style = PaintingStyle.stroke
      ..strokeWidth = strokeWidth;

    canvas.drawCircle(center, radius, backgroundPaint);

    // 绘制进度圆环
    final progressPaint = Paint()
      ..color = color
      ..style = PaintingStyle.stroke
      ..strokeWidth = strokeWidth
      ..strokeCap = StrokeCap.round;

    canvas.drawArc(
      Rect.fromCircle(center: center, radius: radius),
      -90 * (3.14159 / 180), // 从顶部开始
      progress * 2 * 3.14159,
      false,
      progressPaint,
    );
  }

  @override
  bool shouldRepaint(CircularProgressPainter oldDelegate) {
    return oldDelegate.progress != progress ||
           oldDelegate.color != color ||
           oldDelegate.strokeWidth != strokeWidth;
  }
}

// 使用示例
CircularProgressIndicator(
  progress: 0.75,
  size: 120,
  color: Colors.green,
  child: Icon(Icons.check, color: Colors.green, size: 32),
)
```

### 2. 自定义波形动画组件

```dart
class WaveformAnimation extends StatefulWidget {
  final double height;
  final Color? color;
  final int barCount;

  const WaveformAnimation({
    Key? key,
    this.height = 60.0,
    this.color,
    this.barCount = 5,
  }) : super(key: key);

  @override
  _WaveformAnimationState createState() => _WaveformAnimationState();
}

class _WaveformAnimationState extends State<WaveformAnimation>
    with TickerProviderStateMixin {
  late List<AnimationController> _controllers;
  late List<Animation<double>> _animations;

  @override
  void initState() {
    super.initState();
    _controllers = List.generate(
      widget.barCount,
      (index) => AnimationController(
        duration: Duration(milliseconds: 600 + index * 100),
        vsync: this,
      ),
    );

    _animations = _controllers.map((controller) {
      return Tween<double>(
        begin: 0.1,
        end: 1.0,
      ).animate(CurvedAnimation(
        parent: controller,
        curve: Curves.easeInOut,
      ));
    }).toList();

    _startAnimation();
  }

  void _startAnimation() {
    for (var controller in _controllers) {
      controller.repeat(reverse: true);
    }
  }

  @override
  void dispose() {
    for (var controller in _controllers) {
      controller.dispose();
    }
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Row(
      mainAxisAlignment: MainAxisAlignment.center,
      children: List.generate(widget.barCount, (index) {
        return AnimatedBuilder(
          animation: _animations[index],
          builder: (context, child) {
            return Container(
              width: 4,
              height: widget.height * _animations[index].value,
              margin: EdgeInsets.symmetric(horizontal: 2),
              decoration: BoxDecoration(
                color: widget.color ?? Colors.blue,
                borderRadius: BorderRadius.circular(2),
              ),
            );
          },
        );
      }),
    );
  }
}

// 使用示例
WaveformAnimation(
  height: 80,
  color: Colors.green,
  barCount: 7,
)
```

## 💡 高级组件设计模式

### 1. Builder 模式组件

```dart
class FlexibleDialog {
  String? _title;
  String? _content;
  List<Widget> _actions = [];
  Color? _backgroundColor;
  BorderRadius? _borderRadius;
  bool _barrierDismissible = true;

  FlexibleDialog title(String title) {
    _title = title;
    return this;
  }

  FlexibleDialog content(String content) {
    _content = content;
    return this;
  }

  FlexibleDialog action(Widget action) {
    _actions.add(action);
    return this;
  }

  FlexibleDialog backgroundColor(Color color) {
    _backgroundColor = color;
    return this;
  }

  FlexibleDialog borderRadius(BorderRadius radius) {
    _borderRadius = radius;
    return this;
  }

  FlexibleDialog barrierDismissible(bool dismissible) {
    _barrierDismissible = dismissible;
    return this;
  }

  Future<T?> show<T>(BuildContext context) {
    return showDialog<T>(
      context: context,
      barrierDismissible: _barrierDismissible,
      builder: (context) => AlertDialog(
        backgroundColor: _backgroundColor,
        shape: RoundedRectangleBorder(
          borderRadius: _borderRadius ?? BorderRadius.circular(12),
        ),
        title: _title != null ? Text(_title!) : null,
        content: _content != null ? Text(_content!) : null,
        actions: _actions.isNotEmpty ? _actions : null,
      ),
    );
  }
}

// 使用示例
FlexibleDialog()
  .title('确认操作')
  .content('您确定要删除这个项目吗？')
  .action(TextButton(
    onPressed: () => Navigator.of(context).pop(false),
    child: Text('取消'),
  ))
  .action(ElevatedButton(
    onPressed: () => Navigator.of(context).pop(true),
    child: Text('确定'),
  ))
  .backgroundColor(Colors.white)
  .show(context);
```

### 2. 策略模式组件

```dart
abstract class ValidationStrategy {
  String? validate(String value);
}

class EmailValidationStrategy implements ValidationStrategy {
  @override
  String? validate(String value) {
    if (value.isEmpty) return '邮箱不能为空';
    if (!RegExp(r'^[\w-\.]+@([\w-]+\.)+[\w-]{2,4}$').hasMatch(value)) {
      return '请输入有效的邮箱地址';
    }
    return null;
  }
}

class PhoneValidationStrategy implements ValidationStrategy {
  @override
  String? validate(String value) {
    if (value.isEmpty) return '手机号不能为空';
    if (!RegExp(r'^1[3-9]\d{9}$').hasMatch(value)) {
      return '请输入有效的手机号';
    }
    return null;
  }
}

class PasswordValidationStrategy implements ValidationStrategy {
  @override
  String? validate(String value) {
    if (value.isEmpty) return '密码不能为空';
    if (value.length < 6) return '密码长度不能少于6位';
    if (!RegExp(r'^(?=.*[a-zA-Z])(?=.*\d)').hasMatch(value)) {
      return '密码必须包含字母和数字';
    }
    return null;
  }
}

class SmartTextField extends StatefulWidget {
  final String label;
  final String? hint;
  final ValidationStrategy? validationStrategy;
  final TextInputType? keyboardType;
  final bool obscureText;
  final ValueChanged<String>? onChanged;
  final TextEditingController? controller;

  const SmartTextField({
    Key? key,
    required this.label,
    this.hint,
    this.validationStrategy,
    this.keyboardType,
    this.obscureText = false,
    this.onChanged,
    this.controller,
  }) : super(key: key);

  @override
  _SmartTextFieldState createState() => _SmartTextFieldState();
}

class _SmartTextFieldState extends State<SmartTextField> {
  late TextEditingController _controller;
  String? _errorText;
  bool _isValid = true;

  @override
  void initState() {
    super.initState();
    _controller = widget.controller ?? TextEditingController();
    _controller.addListener(_validateInput);
  }

  @override
  void dispose() {
    if (widget.controller == null) {
      _controller.dispose();
    }
    super.dispose();
  }

  void _validateInput() {
    if (widget.validationStrategy != null) {
      final error = widget.validationStrategy!.validate(_controller.text);
      setState(() {
        _errorText = error;
        _isValid = error == null;
      });
    }
    widget.onChanged?.call(_controller.text);
  }

  @override
  Widget build(BuildContext context) {
    return Column(
      crossAxisAlignment: CrossAxisAlignment.start,
      children: [
        TextFormField(
          controller: _controller,
          keyboardType: widget.keyboardType,
          obscureText: widget.obscureText,
          decoration: InputDecoration(
            labelText: widget.label,
            hintText: widget.hint,
            errorText: _errorText,
            border: OutlineInputBorder(
              borderRadius: BorderRadius.circular(8),
            ),
            focusedBorder: OutlineInputBorder(
              borderRadius: BorderRadius.circular(8),
              borderSide: BorderSide(
                color: _isValid ? Colors.blue : Colors.red,
                width: 2,
              ),
            ),
            prefixIcon: _getIcon(),
            suffixIcon: _controller.text.isNotEmpty
                ? Icon(
                    _isValid ? Icons.check_circle : Icons.error,
                    color: _isValid ? Colors.green : Colors.red,
                  )
                : null,
          ),
        ),
      ],
    );
  }

  Widget? _getIcon() {
    if (widget.keyboardType == TextInputType.emailAddress) {
      return Icon(Icons.email);
    } else if (widget.keyboardType == TextInputType.phone) {
      return Icon(Icons.phone);
    } else if (widget.obscureText) {
      return Icon(Icons.lock);
    }
    return null;
  }
}

// 使用示例
SmartTextField(
  label: '邮箱',
  hint: '请输入邮箱地址',
  keyboardType: TextInputType.emailAddress,
  validationStrategy: EmailValidationStrategy(),
)
```

### 3. 性能优化技巧

```dart
// 使用 const 构造函数和静态数据
class OptimizedWidget extends StatelessWidget {
  static const List<String> _defaultItems = ['项目1', '项目2', '项目3'];

  const OptimizedWidget({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Column(
      children: _defaultItems.map((item) => const ListTile(
        title: Text('项目'),
      )).toList(),
    );
  }
}

// 使用 RepaintBoundary 隔离重绘
class EfficientListItem extends StatelessWidget {
  final String title;
  final String subtitle;
  final VoidCallback? onTap;

  const EfficientListItem({
    Key? key,
    required this.title,
    required this.subtitle,
    this.onTap,
  }) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return RepaintBoundary(
      child: ListTile(
        title: Text(title),
        subtitle: Text(subtitle),
        onTap: onTap,
      ),
    );
  }
}

// 避免在 build 方法中创建对象
class EfficientWidget extends StatelessWidget {
  final List<String> items;
  static const EdgeInsets _itemPadding = EdgeInsets.all(8.0);
  static const TextStyle _titleStyle = TextStyle(
    fontSize: 16,
    fontWeight: FontWeight.bold,
  );

  const EfficientWidget({
    Key? key,
    required this.items,
  }) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return ListView.builder(
      itemCount: items.length,
      itemBuilder: (context, index) {
        return Container(
          padding: _itemPadding,
          child: Text(
            items[index],
            style: _titleStyle,
          ),
        );
      },
    );
  }
}
```

### 4. 错误处理和容错机制

```dart
class SafeWidget extends StatelessWidget {
  final Widget child;
  final Widget? fallback;
  final Function(Object error, StackTrace stackTrace)? onError;

  const SafeWidget({
    Key? key,
    required this.child,
    this.fallback,
    this.onError,
  }) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return ErrorBoundary(
      onError: onError,
      fallback: fallback ?? Container(
        padding: EdgeInsets.all(16),
        child: Column(
          mainAxisSize: MainAxisSize.min,
          children: [
            Icon(Icons.error_outline, color: Colors.red, size: 48),
            SizedBox(height: 8),
            Text(
              '组件加载失败',
              style: TextStyle(color: Colors.red, fontSize: 16),
            ),
          ],
        ),
      ),
      child: child,
    );
  }
}

class ErrorBoundary extends StatefulWidget {
  final Widget child;
  final Widget fallback;
  final Function(Object error, StackTrace stackTrace)? onError;

  const ErrorBoundary({
    Key? key,
    required this.child,
    required this.fallback,
    this.onError,
  }) : super(key: key);

  @override
  _ErrorBoundaryState createState() => _ErrorBoundaryState();
}

class _ErrorBoundaryState extends State<ErrorBoundary> {
  bool _hasError = false;

  @override
  Widget build(BuildContext context) {
    if (_hasError) {
      return widget.fallback;
    }

    return ErrorWidget.builder = (FlutterErrorDetails details) {
      widget.onError?.call(details.exception, details.stack ?? StackTrace.empty);
      WidgetsBinding.instance.addPostFrameCallback((_) {
        if (mounted) {
          setState(() {
            _hasError = true;
          });
        }
      });
      return widget.fallback;
    };

    return widget.child;
  }
}
```

### 5. 无障碍支持

```dart
class AccessibleWidget extends StatelessWidget {
  final String label;
  final String? hint;
  final String? value;
  final VoidCallback? onTap;
  final Widget child;
  final bool isButton;
  final bool isSelected;

  const AccessibleWidget({
    Key? key,
    required this.label,
    this.hint,
    this.value,
    this.onTap,
    required this.child,
    this.isButton = false,
    this.isSelected = false,
  }) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Semantics(
      label: label,
      hint: hint,
      value: value,
      button: isButton,
      selected: isSelected,
      onTap: onTap,
      child: GestureDetector(
        onTap: onTap,
        child: child,
      ),
    );
  }
}

// 可访问的自定义按钮
class AccessibleButton extends StatelessWidget {
  final String text;
  final VoidCallback? onPressed;
  final Color? backgroundColor;
  final Color? textColor;
  final IconData? icon;
  final bool isLoading;

  const AccessibleButton({
    Key? key,
    required this.text,
    this.onPressed,
    this.backgroundColor,
    this.textColor,
    this.icon,
    this.isLoading = false,
  }) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Semantics(
      label: text,
      hint: isLoading ? '正在加载' : '点击执行操作',
      button: true,
      enabled: onPressed != null && !isLoading,
      child: Material(
        color: backgroundColor ?? Theme.of(context).primaryColor,
        borderRadius: BorderRadius.circular(8),
        child: InkWell(
          onTap: isLoading ? null : onPressed,
          borderRadius: BorderRadius.circular(8),
          child: Container(
            padding: EdgeInsets.symmetric(horizontal: 24, vertical: 12),
            child: Row(
              mainAxisSize: MainAxisSize.min,
              children: [
                if (isLoading)
                  SizedBox(
                    width: 16,
                    height: 16,
                    child: CircularProgressIndicator(
                      strokeWidth: 2,
                      valueColor: AlwaysStoppedAnimation<Color>(
                        textColor ?? Colors.white,
                      ),
                    ),
                  )
                else if (icon != null)
                  Icon(
                    icon,
                    color: textColor ?? Colors.white,
                    size: 18,
                  ),
                if ((isLoading || icon != null) && text.isNotEmpty)
                  SizedBox(width: 8),
                if (text.isNotEmpty)
                  Text(
                    text,
                    style: TextStyle(
                      color: textColor ?? Colors.white,
                      fontSize: 16,
                      fontWeight: FontWeight.w500,
                    ),
                  ),
              ],
            ),
          ),
        ),
      ),
    );
  }
}
```

## 📚 总结与最佳实践

自定义组件是 Flutter 开发的核心技能，通过本文的深入学习，你应该能够：

### 核心技能掌握

1. **StatelessWidget**：熟练创建高效的无状态组件
2. **StatefulWidget**：掌握复杂的状态管理和生命周期
3. **组件组合**：通过组合模式创建强大的复合组件
4. **性能优化**：合理使用 const 构造函数和优化策略
5. **高级技巧**：实现动画、手势和复杂交互效果

### 设计原则与最佳实践

1. **单一职责原则**：每个组件只负责一个明确的功能
2. **开闭原则**：对扩展开放，对修改封闭
3. **可复用性**：设计通用的组件接口和配置选项
4. **可测试性**：确保组件易于单元测试和集成测试
5. **可维护性**：清晰的代码结构和完善的文档

### 组件优化策略

```dart
// 组件性能优化清单
class PerformanceOptimizedWidget extends StatelessWidget {
  // 1. 使用 const 构造函数
  const PerformanceOptimizedWidget({Key? key}) : super(key: key);

  // 2. 静态常量避免重复创建
  static const EdgeInsets _padding = EdgeInsets.all(16.0);
  static const TextStyle _textStyle = TextStyle(fontSize: 16);

  @override
  Widget build(BuildContext context) {
    return RepaintBoundary( // 3. 使用 RepaintBoundary 隔离重绘
      child: Container(
        padding: _padding,
        child: const Text(
          '优化的组件',
          style: _textStyle,
        ),
      ),
    );
  }
}
```

### 组件设计模式

1. **Builder 模式**：灵活构建复杂组件
2. **Factory 模式**：根据条件创建不同组件
3. **Observer 模式**：实现组件间通信
4. **Strategy 模式**：支持多种行为策略
5. **Decorator 模式**：动态添加组件功能

### 测试策略

```dart
// 组件测试示例
void main() {
  group('CustomButton Tests', () {
    testWidgets('should display text correctly', (WidgetTester tester) async {
      await tester.pumpWidget(
        MaterialApp(
          home: CustomButton(
            text: 'Test Button',
            onPressed: () {},
          ),
        ),
      );

      expect(find.text('Test Button'), findsOneWidget);
    });

    testWidgets('should call onPressed when tapped', (WidgetTester tester) async {
      bool wasPressed = false;

      await tester.pumpWidget(
        MaterialApp(
          home: CustomButton(
            text: 'Test Button',
            onPressed: () => wasPressed = true,
          ),
        ),
      );

      await tester.tap(find.text('Test Button'));
      expect(wasPressed, isTrue);
    });
  });
}
```

### 推荐工具和库

- **flutter_test**：组件单元测试框架
- **golden_toolkit**：UI 回归测试工具
- **flutter_driver**：端到端集成测试
- **mockito**：模拟依赖进行测试
- **flutter_hooks**：简化状态管理
- **provider**：依赖注入和状态管理

### 组件发布与维护

```yaml
# pubspec.yaml 示例
name: my_custom_widgets
description: A collection of custom Flutter widgets
version: 1.0.0

dependencies:
  flutter:
    sdk: flutter

dev_dependencies:
  flutter_test:
    sdk: flutter
  flutter_lints: ^2.0.0
```

### 关键要点总结

- **遵循设计原则**：使用 SOLID 原则设计组件
- **注重性能优化**：避免不必要的重建和计算
- **支持无障碍访问**：为所有用户提供良好的体验
- **错误处理**：提供友好的错误提示和降级方案
- **完善测试**：确保组件的稳定性和可靠性
- **文档完整**：提供清晰的使用说明和示例

### 下一步学习

掌握了自定义组件的基础后，你可以继续学习：

- [动画组件](animation-widgets.md) - 学习高级动画组件开发
- [状态管理](state-management.md) - 深入状态管理模式
- [性能优化](performance.md) - 组件性能优化技巧
- [测试策略](testing.md) - 组件测试最佳实践
- [架构设计](architecture.md) - 大型应用组件架构

记住，优秀的自定义组件不仅功能强大，更重要的是易于使用、维护和扩展。在设计组件时，始终站在使用者的角度思考，追求简洁的 API 和卓越的用户体验。通过不断实践和优化，你一定能创造出真正有价值的组件库！

---

<div align="center">

**🌟 如果这篇文章对你有帮助，请给个 Star 支持一下！ 🌟**

[![GitHub stars](https://img.shields.io/github/stars/1989allen126/language-tutorial?style=social)](https://github.com/1989allen126/language-tutorial)
[![GitHub forks](https://img.shields.io/github/forks/1989allen126/language-tutorial?style=social)](https://github.com/1989allen126/language-tutorial)

</div>
