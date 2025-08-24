# Flutter 持续集成(CI)完整指南

## 📖 概述

持续集成(Continuous Integration, CI)是DevOps实践的核心组成部分，通过自动化构建、测试和代码质量检查，确保代码变更的质量和稳定性。本文档详细介绍Flutter应用的CI实践。

## 🎯 CI核心目标

- **快速反馈**: 代码提交后立即获得构建和测试结果
- **质量保证**: 自动化代码质量检查和测试
- **风险降低**: 早期发现和修复问题
- **效率提升**: 减少手动操作，提高开发效率
- **标准化**: 统一的构建和测试流程

## 🛠 主流CI平台

### 1. GitHub Actions

#### 基础配置
```yaml
# .github/workflows/ci.yml
name: Flutter CI

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

env:
  FLUTTER_VERSION: '3.16.0'
  JAVA_VERSION: '17'
  NODE_VERSION: '18'

jobs:
  # 代码质量检查
  code-quality:
    name: Code Quality
    runs-on: ubuntu-latest
    timeout-minutes: 15
    
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
        with:
          fetch-depth: 0  # 获取完整历史用于分析
          
      - name: Setup Flutter
        uses: subosito/flutter-action@v2
        with:
          flutter-version: ${{ env.FLUTTER_VERSION }}
          cache: true
          
      - name: Get dependencies
        run: flutter pub get
        
      - name: Verify formatting
        run: dart format --output=none --set-exit-if-changed .
        
      - name: Analyze project source
        run: flutter analyze --fatal-infos
        
      - name: Run custom lints
        run: flutter pub run dart_code_metrics:metrics analyze lib
        
      - name: Check for outdated dependencies
        run: flutter pub outdated --exit-if-outdated

  # 单元测试和Widget测试
  test:
    name: Unit & Widget Tests
    runs-on: ubuntu-latest
    timeout-minutes: 20
    needs: code-quality
    
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
        
      - name: Setup Flutter
        uses: subosito/flutter-action@v2
        with:
          flutter-version: ${{ env.FLUTTER_VERSION }}
          cache: true
          
      - name: Get dependencies
        run: flutter pub get
        
      - name: Run tests with coverage
        run: |
          flutter test --coverage --test-randomize-ordering-seed random
          
      - name: Generate coverage report
        run: |
          sudo apt-get update
          sudo apt-get install -y lcov
          genhtml coverage/lcov.info -o coverage/html
          
      - name: Upload coverage to Codecov
        uses: codecov/codecov-action@v3
        with:
          file: coverage/lcov.info
          flags: unittests
          name: codecov-umbrella
          
      - name: Upload coverage artifacts
        uses: actions/upload-artifact@v3
        with:
          name: coverage-report
          path: coverage/html/

  # Android构建
  build-android:
    name: Build Android
    runs-on: ubuntu-latest
    timeout-minutes: 30
    needs: test
    
    strategy:
      matrix:
        build-type: [debug, release]
        
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
        
      - name: Setup Java
        uses: actions/setup-java@v3
        with:
          distribution: 'zulu'
          java-version: ${{ env.JAVA_VERSION }}
          
      - name: Setup Flutter
        uses: subosito/flutter-action@v2
        with:
          flutter-version: ${{ env.FLUTTER_VERSION }}
          cache: true
          
      - name: Setup Android SDK
        uses: android-actions/setup-android@v2
        
      - name: Get dependencies
        run: flutter pub get
        
      - name: Build APK
        run: |
          if [ "${{ matrix.build-type }}" = "release" ]; then
            flutter build apk --release --split-per-abi
            flutter build appbundle --release
          else
            flutter build apk --debug
          fi
          
      - name: Upload Android artifacts
        uses: actions/upload-artifact@v3
        with:
          name: android-${{ matrix.build-type }}
          path: |
            build/app/outputs/flutter-apk/*.apk
            build/app/outputs/bundle/release/*.aab

  # iOS构建
  build-ios:
    name: Build iOS
    runs-on: macos-latest
    timeout-minutes: 45
    needs: test
    
    strategy:
      matrix:
        build-type: [debug, release]
        
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
        
      - name: Setup Flutter
        uses: subosito/flutter-action@v2
        with:
          flutter-version: ${{ env.FLUTTER_VERSION }}
          cache: true
          
      - name: Get dependencies
        run: flutter pub get
        
      - name: Build iOS
        run: |
          if [ "${{ matrix.build-type }}" = "release" ]; then
            flutter build ios --release --no-codesign
          else
            flutter build ios --debug --no-codesign
          fi
          
      - name: Archive iOS app
        if: matrix.build-type == 'release'
        run: |
          cd ios
          xcodebuild -workspace Runner.xcworkspace \
                     -scheme Runner \
                     -configuration Release \
                     -destination generic/platform=iOS \
                     -archivePath build/Runner.xcarchive \
                     archive
                     
      - name: Upload iOS artifacts
        uses: actions/upload-artifact@v3
        with:
          name: ios-${{ matrix.build-type }}
          path: |
            build/ios/iphoneos/Runner.app
            ios/build/Runner.xcarchive

  # 集成测试
  integration-test:
    name: Integration Tests
    runs-on: macos-latest
    timeout-minutes: 60
    needs: [build-android, build-ios]
    
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
        
      - name: Setup Flutter
        uses: subosito/flutter-action@v2
        with:
          flutter-version: ${{ env.FLUTTER_VERSION }}
          cache: true
          
      - name: Get dependencies
        run: flutter pub get
        
      - name: Start iOS Simulator
        run: |
          xcrun simctl list devices
          xcrun simctl boot "iPhone 14 Pro"
          
      - name: Run integration tests
        run: |
          flutter drive \
            --driver=test_driver/integration_test.dart \
            --target=integration_test/app_test.dart
            
      - name: Upload test results
        if: always()
        uses: actions/upload-artifact@v3
        with:
          name: integration-test-results
          path: |
            test_driver/screenshots/
            integration_test/reports/

  # 安全扫描
  security-scan:
    name: Security Scan
    runs-on: ubuntu-latest
    timeout-minutes: 15
    
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
        
      - name: Run Trivy vulnerability scanner
        uses: aquasecurity/trivy-action@master
        with:
          scan-type: 'fs'
          scan-ref: '.'
          format: 'sarif'
          output: 'trivy-results.sarif'
          
      - name: Upload Trivy scan results
        uses: github/codeql-action/upload-sarif@v2
        with:
          sarif_file: 'trivy-results.sarif'
          
      - name: Dependency vulnerability scan
        run: |
          flutter pub deps --json > deps.json
          # 使用自定义脚本检查已知漏洞
          python scripts/check_vulnerabilities.py deps.json

  # 性能基准测试
  performance-test:
    name: Performance Benchmarks
    runs-on: ubuntu-latest
    timeout-minutes: 30
    
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
        
      - name: Setup Flutter
        uses: subosito/flutter-action@v2
        with:
          flutter-version: ${{ env.FLUTTER_VERSION }}
          cache: true
          
      - name: Get dependencies
        run: flutter pub get
        
      - name: Run performance tests
        run: |
          flutter test test/performance/
          
      - name: Generate performance report
        run: |
          python scripts/generate_performance_report.py
          
      - name: Upload performance results
        uses: actions/upload-artifact@v3
        with:
          name: performance-report
          path: performance_report.html
```

#### 高级配置特性

```yaml
# 矩阵构建策略
strategy:
  matrix:
    os: [ubuntu-latest, macos-latest, windows-latest]
    flutter-version: ['3.13.0', '3.16.0', 'stable']
    include:
      - os: ubuntu-latest
        flutter-version: 'beta'
    exclude:
      - os: windows-latest
        flutter-version: '3.13.0'
  fail-fast: false
  max-parallel: 3

# 条件执行
if: |
  github.event_name == 'push' ||
  (github.event_name == 'pull_request' && 
   contains(github.event.pull_request.labels.*.name, 'run-ci'))

# 缓存优化
- name: Cache Flutter dependencies
  uses: actions/cache@v3
  with:
    path: |
      ~/.pub-cache
      ${{ runner.tool_cache }}/flutter
    key: ${{ runner.os }}-flutter-${{ hashFiles('**/pubspec.lock') }}
    restore-keys: |
      ${{ runner.os }}-flutter-

# 并行作业依赖
needs: [code-quality, test]
if: success() || failure()  # 即使前置作业失败也执行
```

### 2. GitLab CI/CD

#### 基础配置
```yaml
# .gitlab-ci.yml
stages:
  - prepare
  - quality
  - test
  - build
  - deploy
  - cleanup

variables:
  FLUTTER_VERSION: "3.16.0"
  PUB_CACHE: ".pub-cache"
  GRADLE_OPTS: "-Dorg.gradle.daemon=false"
  GRADLE_USER_HOME: ".gradle"

# 缓存配置
cache:
  key: "$CI_COMMIT_REF_SLUG"
  paths:
    - .pub-cache/
    - .gradle/
    - android/.gradle/

# 准备阶段
prepare:dependencies:
  stage: prepare
  image: cirrusci/flutter:$FLUTTER_VERSION
  script:
    - flutter --version
    - flutter pub get
    - flutter pub deps
  artifacts:
    paths:
      - .pub-cache/
    expire_in: 1 hour
  only:
    - merge_requests
    - main
    - develop

# 代码质量检查
code:format:
  stage: quality
  image: cirrusci/flutter:$FLUTTER_VERSION
  dependencies:
    - prepare:dependencies
  script:
    - dart format --output=none --set-exit-if-changed .
  only:
    - merge_requests
    - main
    - develop

code:analyze:
  stage: quality
  image: cirrusci/flutter:$FLUTTER_VERSION
  dependencies:
    - prepare:dependencies
  script:
    - flutter analyze --fatal-infos
  artifacts:
    reports:
      junit: analysis_results.xml
  only:
    - merge_requests
    - main
    - develop

code:metrics:
  stage: quality
  image: cirrusci/flutter:$FLUTTER_VERSION
  dependencies:
    - prepare:dependencies
  script:
    - flutter pub run dart_code_metrics:metrics analyze lib --reporter=gitlab > code_metrics.json
  artifacts:
    reports:
      codequality: code_metrics.json
    expire_in: 1 week
  only:
    - merge_requests
    - main
    - develop

# 测试阶段
test:unit:
  stage: test
  image: cirrusci/flutter:$FLUTTER_VERSION
  dependencies:
    - prepare:dependencies
  script:
    - flutter test --coverage --machine > test_results.json
    - genhtml coverage/lcov.info -o coverage/html
  coverage: '/lines......: \d+\.\d+%/'
  artifacts:
    reports:
      junit: test_results.xml
      coverage_report:
        coverage_format: cobertura
        path: coverage/cobertura.xml
    paths:
      - coverage/html/
    expire_in: 1 week
  only:
    - merge_requests
    - main
    - develop

test:integration:
  stage: test
  image: cirrusci/flutter:$FLUTTER_VERSION
  services:
    - name: selenium/standalone-chrome:latest
      alias: selenium
  dependencies:
    - prepare:dependencies
  script:
    - flutter drive --target=test_driver/app.dart
  artifacts:
    paths:
      - test_driver/screenshots/
    expire_in: 1 week
  only:
    - merge_requests
    - main
    - develop

# Android构建
build:android:debug:
  stage: build
  image: cirrusci/flutter:$FLUTTER_VERSION
  dependencies:
    - prepare:dependencies
  script:
    - flutter build apk --debug
  artifacts:
    paths:
      - build/app/outputs/flutter-apk/app-debug.apk
    expire_in: 1 week
  only:
    - merge_requests
    - develop

build:android:release:
  stage: build
  image: cirrusci/flutter:$FLUTTER_VERSION
  dependencies:
    - prepare:dependencies
  script:
    - flutter build apk --release --split-per-abi
    - flutter build appbundle --release
  artifacts:
    paths:
      - build/app/outputs/flutter-apk/*.apk
      - build/app/outputs/bundle/release/app-release.aab
    expire_in: 1 month
  only:
    - main
    - tags

# iOS构建
build:ios:debug:
  stage: build
  tags:
    - macos
  dependencies:
    - prepare:dependencies
  script:
    - flutter build ios --debug --no-codesign
  artifacts:
    paths:
      - build/ios/iphoneos/Runner.app
    expire_in: 1 week
  only:
    - merge_requests
    - develop

build:ios:release:
  stage: build
  tags:
    - macos
  dependencies:
    - prepare:dependencies
  script:
    - flutter build ios --release --no-codesign
    - cd ios && xcodebuild -workspace Runner.xcworkspace -scheme Runner -configuration Release -destination generic/platform=iOS -archivePath build/Runner.xcarchive archive
  artifacts:
    paths:
      - ios/build/Runner.xcarchive
    expire_in: 1 month
  only:
    - main
    - tags

# 安全扫描
security:scan:
  stage: test
  image: aquasec/trivy:latest
  script:
    - trivy fs --format template --template "@contrib/gitlab.tpl" -o gl-sast-report.json .
  artifacts:
    reports:
      sast: gl-sast-report.json
  only:
    - merge_requests
    - main
    - develop

# 清理阶段
cleanup:cache:
  stage: cleanup
  script:
    - rm -rf .pub-cache/
    - rm -rf .gradle/
  when: always
  only:
    - schedules
```

### 3. Jenkins Pipeline

#### Declarative Pipeline
```groovy
// Jenkinsfile
pipeline {
    agent {
        kubernetes {
            yaml """
                apiVersion: v1
                kind: Pod
                spec:
                  containers:
                  - name: flutter
                    image: cirrusci/flutter:3.16.0
                    command:
                    - cat
                    tty: true
                    volumeMounts:
                    - name: docker-sock
                      mountPath: /var/run/docker.sock
                  - name: android
                    image: androidsdk/android-30
                    command:
                    - cat
                    tty: true
                  volumes:
                  - name: docker-sock
                    hostPath:
                      path: /var/run/docker.sock
            """
        }
    }
    
    environment {
        FLUTTER_VERSION = '3.16.0'
        PUB_CACHE = "${WORKSPACE}/.pub-cache"
        GRADLE_USER_HOME = "${WORKSPACE}/.gradle"
    }
    
    options {
        buildDiscarder(logRotator(numToKeepStr: '10'))
        timeout(time: 60, unit: 'MINUTES')
        timestamps()
        ansiColor('xterm')
    }
    
    triggers {
        pollSCM('H/5 * * * *')  // 每5分钟检查一次代码变更
        cron('H 2 * * *')       // 每天凌晨2点执行
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
                script {
                    env.GIT_COMMIT_SHORT = sh(
                        script: "git rev-parse --short HEAD",
                        returnStdout: true
                    ).trim()
                }
            }
        }
        
        stage('Setup') {
            parallel {
                stage('Flutter Dependencies') {
                    steps {
                        container('flutter') {
                            sh '''
                                flutter --version
                                flutter pub get
                                flutter pub deps
                            '''
                        }
                    }
                }
                
                stage('Cache Restore') {
                    steps {
                        script {
                            if (fileExists('.pub-cache')) {
                                echo 'Restoring pub cache'
                            }
                        }
                    }
                }
            }
        }
        
        stage('Code Quality') {
            parallel {
                stage('Format Check') {
                    steps {
                        container('flutter') {
                            sh 'dart format --output=none --set-exit-if-changed .'
                        }
                    }
                }
                
                stage('Static Analysis') {
                    steps {
                        container('flutter') {
                            sh 'flutter analyze --fatal-infos'
                        }
                    }
                    post {
                        always {
                            publishHTML([
                                allowMissing: false,
                                alwaysLinkToLastBuild: true,
                                keepAll: true,
                                reportDir: 'analysis_results',
                                reportFiles: 'index.html',
                                reportName: 'Analysis Report'
                            ])
                        }
                    }
                }
                
                stage('Code Metrics') {
                    steps {
                        container('flutter') {
                            sh 'flutter pub run dart_code_metrics:metrics analyze lib'
                        }
                    }
                }
            }
        }
        
        stage('Testing') {
            parallel {
                stage('Unit Tests') {
                    steps {
                        container('flutter') {
                            sh '''
                                flutter test --coverage --machine > test_results.json
                                lcov --summary coverage/lcov.info
                            '''
                        }
                    }
                    post {
                        always {
                            publishTestResults(
                                testResultsPattern: 'test_results.xml'
                            )
                            publishCoverage(
                                adapters: [coberturaAdapter('coverage/cobertura.xml')],
                                sourceFileResolver: sourceFiles('STORE_LAST_BUILD')
                            )
                        }
                    }
                }
                
                stage('Widget Tests') {
                    steps {
                        container('flutter') {
                            sh 'flutter test test/widget/'
                        }
                    }
                }
            }
        }
        
        stage('Build') {
            parallel {
                stage('Android Debug') {
                    when {
                        anyOf {
                            branch 'develop'
                            changeRequest()
                        }
                    }
                    steps {
                        container('android') {
                            sh '''
                                flutter build apk --debug
                                ls -la build/app/outputs/flutter-apk/
                            '''
                        }
                    }
                    post {
                        success {
                            archiveArtifacts(
                                artifacts: 'build/app/outputs/flutter-apk/app-debug.apk',
                                fingerprint: true
                            )
                        }
                    }
                }
                
                stage('Android Release') {
                    when {
                        anyOf {
                            branch 'main'
                            tag pattern: 'v\\d+\\.\\d+\\.\\d+', comparator: 'REGEXP'
                        }
                    }
                    steps {
                        container('android') {
                            sh '''
                                flutter build apk --release --split-per-abi
                                flutter build appbundle --release
                            '''
                        }
                    }
                    post {
                        success {
                            archiveArtifacts(
                                artifacts: 'build/app/outputs/flutter-apk/*.apk,build/app/outputs/bundle/release/*.aab',
                                fingerprint: true
                            )
                        }
                    }
                }
                
                stage('iOS Build') {
                    agent {
                        label 'macos'
                    }
                    when {
                        anyOf {
                            branch 'main'
                            branch 'develop'
                        }
                    }
                    steps {
                        sh '''
                            flutter build ios --release --no-codesign
                            cd ios
                            xcodebuild -workspace Runner.xcworkspace \
                                       -scheme Runner \
                                       -configuration Release \
                                       -destination generic/platform=iOS \
                                       -archivePath build/Runner.xcarchive \
                                       archive
                        '''
                    }
                    post {
                        success {
                            archiveArtifacts(
                                artifacts: 'ios/build/Runner.xcarchive/**',
                                fingerprint: true
                            )
                        }
                    }
                }
            }
        }
        
        stage('Integration Tests') {
            when {
                anyOf {
                    branch 'main'
                    branch 'develop'
                }
            }
            steps {
                container('flutter') {
                    sh '''
                        flutter drive \
                            --driver=test_driver/integration_test.dart \
                            --target=integration_test/app_test.dart
                    '''
                }
            }
            post {
                always {
                    publishHTML([
                        allowMissing: false,
                        alwaysLinkToLastBuild: true,
                        keepAll: true,
                        reportDir: 'integration_test/reports',
                        reportFiles: 'index.html',
                        reportName: 'Integration Test Report'
                    ])
                }
            }
        }
        
        stage('Security Scan') {
            steps {
                script {
                    def scanResult = sh(
                        script: 'trivy fs --format json --output trivy-report.json .',
                        returnStatus: true
                    )
                    
                    if (scanResult != 0) {
                        currentBuild.result = 'UNSTABLE'
                        echo 'Security vulnerabilities found'
                    }
                }
            }
            post {
                always {
                    archiveArtifacts(
                        artifacts: 'trivy-report.json',
                        allowEmptyArchive: true
                    )
                }
            }
        }
        
        stage('Deploy to Staging') {
            when {
                branch 'develop'
            }
            steps {
                script {
                    // 部署到测试环境
                    sh '''
                        echo "Deploying to staging environment"
                        # 实际部署脚本
                    '''
                }
            }
        }
    }
    
    post {
        always {
            // 清理工作空间
            cleanWs()
        }
        
        success {
            script {
                if (env.BRANCH_NAME == 'main') {
                    // 发送成功通知
                    slackSend(
                        channel: '#ci-cd',
                        color: 'good',
                        message: "✅ Build succeeded for ${env.JOB_NAME} - ${env.BUILD_NUMBER}"
                    )
                }
            }
        }
        
        failure {
            // 发送失败通知
            slackSend(
                channel: '#ci-cd',
                color: 'danger',
                message: "❌ Build failed for ${env.JOB_NAME} - ${env.BUILD_NUMBER}\nCheck: ${env.BUILD_URL}"
            )
            
            // 发送邮件通知
            emailext(
                subject: "Build Failed: ${env.JOB_NAME} - ${env.BUILD_NUMBER}",
                body: "Build failed. Check console output at ${env.BUILD_URL}",
                to: "${env.CHANGE_AUTHOR_EMAIL}"
            )
        }
        
        unstable {
            slackSend(
                channel: '#ci-cd',
                color: 'warning',
                message: "⚠️ Build unstable for ${env.JOB_NAME} - ${env.BUILD_NUMBER}"
            )
        }
    }
}
```

## 🔧 CI优化策略

### 1. 构建缓存优化

```yaml
# GitHub Actions缓存策略
- name: Cache Flutter dependencies
  uses: actions/cache@v3
  with:
    path: |
      ~/.pub-cache
      ${{ runner.tool_cache }}/flutter
      ~/.gradle/caches
      ~/.gradle/wrapper
    key: ${{ runner.os }}-flutter-${{ hashFiles('**/pubspec.lock', '**/gradle-wrapper.properties') }}
    restore-keys: |
      ${{ runner.os }}-flutter-
      ${{ runner.os }}-

# 分层缓存策略
- name: Cache pub dependencies
  uses: actions/cache@v3
  with:
    path: ~/.pub-cache
    key: pub-${{ hashFiles('**/pubspec.lock') }}
    
- name: Cache build outputs
  uses: actions/cache@v3
  with:
    path: |
      build/
      .dart_tool/
    key: build-${{ github.sha }}
    restore-keys: |
      build-${{ github.ref }}-
      build-
```

### 2. 并行化策略

```yaml
# 作业级并行
jobs:
  test-unit:
    strategy:
      matrix:
        test-suite: [core, ui, network, storage]
    steps:
      - run: flutter test test/${{ matrix.test-suite }}/

  test-platform:
    strategy:
      matrix:
        os: [ubuntu-latest, macos-latest, windows-latest]
    runs-on: ${{ matrix.os }}
    
# 步骤级并行
- name: Parallel linting
  run: |
    flutter analyze &
    dart format --set-exit-if-changed . &
    flutter pub run dart_code_metrics:metrics analyze lib &
    wait
```

### 3. 条件执行优化

```yaml
# 路径过滤
on:
  push:
    paths:
      - 'lib/**'
      - 'test/**'
      - 'pubspec.yaml'
      - '.github/workflows/**'
  pull_request:
    paths-ignore:
      - 'docs/**'
      - '*.md'
      - 'assets/**'

# 分支策略
if: |
  github.event_name == 'push' && 
  (github.ref == 'refs/heads/main' || github.ref == 'refs/heads/develop') ||
  github.event_name == 'pull_request'

# 变更检测
- name: Check for changes
  id: changes
  uses: dorny/paths-filter@v2
  with:
    filters: |
      src:
        - 'lib/**'
        - 'test/**'
      docs:
        - 'docs/**'
        - '*.md'
        
- name: Run tests
  if: steps.changes.outputs.src == 'true'
  run: flutter test
```

## 📊 CI监控和分析

### 1. 构建时间分析

```python
# scripts/analyze_build_times.py
import json
import requests
from datetime import datetime, timedelta

def analyze_build_performance():
    # 获取最近30天的构建数据
    builds = get_recent_builds(days=30)
    
    # 分析构建时间趋势
    build_times = []
    for build in builds:
        duration = build['duration']
        timestamp = build['timestamp']
        build_times.append({
            'date': timestamp,
            'duration': duration,
            'branch': build['branch'],
            'status': build['status']
        })
    
    # 生成报告
    report = {
        'average_duration': sum(b['duration'] for b in build_times) / len(build_times),
        'success_rate': len([b for b in build_times if b['status'] == 'success']) / len(build_times),
        'slowest_builds': sorted(build_times, key=lambda x: x['duration'], reverse=True)[:10],
        'trends': analyze_trends(build_times)
    }
    
    return report

def generate_performance_dashboard():
    report = analyze_build_performance()
    
    html_template = """
    <!DOCTYPE html>
    <html>
    <head>
        <title>CI Performance Dashboard</title>
        <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    </head>
    <body>
        <h1>CI Performance Dashboard</h1>
        
        <div class="metrics">
            <div class="metric">
                <h3>Average Build Time</h3>
                <p>{avg_duration:.2f} minutes</p>
            </div>
            <div class="metric">
                <h3>Success Rate</h3>
                <p>{success_rate:.1%}</p>
            </div>
        </div>
        
        <canvas id="buildTrendChart"></canvas>
        
        <script>
            // 构建趋势图表
            const ctx = document.getElementById('buildTrendChart').getContext('2d');
            new Chart(ctx, {{
                type: 'line',
                data: {{
                    labels: {labels},
                    datasets: [{{
                        label: 'Build Duration (minutes)',
                        data: {durations},
                        borderColor: 'rgb(75, 192, 192)',
                        tension: 0.1
                    }}]
                }},
                options: {{
                    responsive: true,
                    scales: {{
                        y: {{
                            beginAtZero: true
                        }}
                    }}
                }}
            }});
        </script>
    </body>
    </html>
    """.format(
        avg_duration=report['average_duration'],
        success_rate=report['success_rate'],
        labels=json.dumps([b['date'] for b in report['trends']]),
        durations=json.dumps([b['duration'] for b in report['trends']])
    )
    
    with open('ci_performance_dashboard.html', 'w') as f:
        f.write(html_template)

if __name__ == '__main__':
    generate_performance_dashboard()
```

### 2. 质量指标追踪

```dart
// lib/ci/quality_metrics.dart
class QualityMetrics {
  final double testCoverage;
  final int codeComplexity;
  final int technicalDebt;
  final List<String> codeSmells;
  final int vulnerabilities;
  
  QualityMetrics({
    required this.testCoverage,
    required this.codeComplexity,
    required this.technicalDebt,
    required this.codeSmells,
    required this.vulnerabilities,
  });
  
  bool get passesQualityGate {
    return testCoverage >= 80.0 &&
           codeComplexity <= 10 &&
           technicalDebt <= 5 &&
           vulnerabilities == 0;
  }
  
  Map<String, dynamic> toJson() {
    return {
      'test_coverage': testCoverage,
      'code_complexity': codeComplexity,
      'technical_debt': technicalDebt,
      'code_smells': codeSmells,
      'vulnerabilities': vulnerabilities,
      'passes_quality_gate': passesQualityGate,
    };
  }
}

class QualityGate {
  static Future<QualityMetrics> analyze() async {
    // 收集各种质量指标
    final coverage = await _getCoverageMetrics();
    final complexity = await _getComplexityMetrics();
    final debt = await _getTechnicalDebtMetrics();
    final smells = await _getCodeSmells();
    final vulnerabilities = await _getSecurityVulnerabilities();
    
    return QualityMetrics(
      testCoverage: coverage,
      codeComplexity: complexity,
      technicalDebt: debt,
      codeSmells: smells,
      vulnerabilities: vulnerabilities,
    );
  }
  
  static Future<double> _getCoverageMetrics() async {
    // 解析lcov.info文件
    final lcovFile = File('coverage/lcov.info');
    if (!await lcovFile.exists()) return 0.0;
    
    final content = await lcovFile.readAsString();
    final lines = content.split('\n');
    
    int totalLines = 0;
    int coveredLines = 0;
    
    for (final line in lines) {
      if (line.startsWith('LF:')) {
        totalLines += int.parse(line.substring(3));
      } else if (line.startsWith('LH:')) {
        coveredLines += int.parse(line.substring(3));
      }
    }
    
    return totalLines > 0 ? (coveredLines / totalLines) * 100 : 0.0;
  }
  
  static Future<int> _getComplexityMetrics() async {
    // 使用dart_code_metrics分析复杂度
    final result = await Process.run(
      'flutter',
      ['pub', 'run', 'dart_code_metrics:metrics', 'analyze', 'lib', '--reporter=json'],
    );
    
    if (result.exitCode == 0) {
      final json = jsonDecode(result.stdout);
      return json['averageComplexity'] ?? 0;
    }
    
    return 0;
  }
}
```

## 🚀 最佳实践

### 1. CI配置最佳实践

- **快速反馈**: 优先执行快速检查(格式化、静态分析)
- **并行执行**: 合理使用并行策略减少总体时间
- **缓存策略**: 有效利用缓存减少重复工作
- **条件执行**: 根据变更内容智能执行相关检查
- **资源优化**: 合理分配CI资源，避免资源浪费

### 2. 质量门禁设置

- **代码覆盖率**: 不低于80%
- **静态分析**: 零警告和错误
- **安全扫描**: 零高危漏洞
- **性能测试**: 关键指标不退化
- **代码审查**: 必须通过同行评审

### 3. 监控和告警

- **构建失败率**: 监控并及时处理
- **构建时间**: 设置合理阈值和告警
- **资源使用**: 监控CI资源消耗
- **质量趋势**: 跟踪代码质量变化
- **依赖安全**: 监控依赖漏洞

通过系统的CI实践，可以显著提升Flutter应用的开发效率和代码质量，确保每次代码变更都经过严格的自动化验证。