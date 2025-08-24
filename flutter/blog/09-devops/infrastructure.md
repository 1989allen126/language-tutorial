# Flutter 基础设施管理

本文档详细介绍Flutter应用的基础设施管理，包括容器化部署、云原生实践、基础设施即代码和安全实践等核心内容。

## 📦 容器化部署

### 1. Docker 配置

#### Flutter Web 应用 Dockerfile

```dockerfile
# Dockerfile.web
# 多阶段构建 - 构建阶段
FROM cirrusci/flutter:stable AS build-env

# 设置工作目录
WORKDIR /app

# 复制 pubspec 文件
COPY pubspec.* ./

# 获取依赖
RUN flutter pub get

# 复制源代码
COPY . .

# 构建 Web 应用
RUN flutter build web --release

# 生产阶段 - 使用 Nginx 服务静态文件
FROM nginx:alpine

# 复制构建产物
COPY --from=build-env /app/build/web /usr/share/nginx/html

# 复制 Nginx 配置
COPY nginx.conf /etc/nginx/nginx.conf

# 暴露端口
EXPOSE 80

# 启动 Nginx
CMD ["nginx", "-g", "daemon off;"]
```

#### Nginx 配置文件

```nginx
# nginx.conf
events {
    worker_connections 1024;
}

http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;
    
    # 启用 gzip 压缩
    gzip on;
    gzip_vary on;
    gzip_min_length 1024;
    gzip_proxied any;
    gzip_comp_level 6;
    gzip_types
        text/plain
        text/css
        text/xml
        text/javascript
        application/json
        application/javascript
        application/xml+rss
        application/atom+xml
        image/svg+xml;
    
    server {
        listen 80;
        server_name localhost;
        
        # 设置根目录
        root /usr/share/nginx/html;
        index index.html;
        
        # 处理 Flutter Web 路由
        location / {
            try_files $uri $uri/ /index.html;
        }
        
        # 缓存静态资源
        location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg|woff|woff2|ttf|eot)$ {
            expires 1y;
            add_header Cache-Control "public, immutable";
        }
        
        # 安全头
        add_header X-Frame-Options "SAMEORIGIN" always;
        add_header X-Content-Type-Options "nosniff" always;
        add_header X-XSS-Protection "1; mode=block" always;
        add_header Referrer-Policy "no-referrer-when-downgrade" always;
        add_header Content-Security-Policy "default-src 'self' http: https: data: blob: 'unsafe-inline'" always;
    }
}
```

#### Flutter 服务端应用 Dockerfile

```dockerfile
# Dockerfile.server
FROM dart:stable AS build

# 设置工作目录
WORKDIR /app

# 复制 pubspec 文件
COPY pubspec.* ./

# 获取依赖
RUN dart pub get

# 复制源代码
COPY . .

# 编译应用
RUN dart compile exe bin/server.dart -o bin/server

# 运行时镜像
FROM scratch

# 复制编译后的可执行文件
COPY --from=build /runtime/ /
COPY --from=build /app/bin/server /app/bin/

# 暴露端口
EXPOSE 8080

# 启动应用
ENTRYPOINT ["/app/bin/server"]
```

### 2. Docker Compose 配置

```yaml
# docker-compose.yml
version: '3.8'

services:
  # Flutter Web 应用
  flutter-web:
    build:
      context: .
      dockerfile: Dockerfile.web
    ports:
      - "80:80"
    environment:
      - NODE_ENV=production
    restart: unless-stopped
    networks:
      - flutter-network
    
  # Flutter 服务端应用
  flutter-server:
    build:
      context: .
      dockerfile: Dockerfile.server
    ports:
      - "8080:8080"
    environment:
      - DATABASE_URL=postgresql://user:password@postgres:5432/flutter_db
      - REDIS_URL=redis://redis:6379
    depends_on:
      - postgres
      - redis
    restart: unless-stopped
    networks:
      - flutter-network
    
  # PostgreSQL 数据库
  postgres:
    image: postgres:13
    environment:
      - POSTGRES_DB=flutter_db
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=password
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql
    restart: unless-stopped
    networks:
      - flutter-network
    
  # Redis 缓存
  redis:
    image: redis:6-alpine
    restart: unless-stopped
    networks:
      - flutter-network
    
  # Nginx 负载均衡器
  nginx:
    image: nginx:alpine
    ports:
      - "443:443"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf
      - ./nginx/ssl:/etc/nginx/ssl
    depends_on:
      - flutter-web
      - flutter-server
    restart: unless-stopped
    networks:
      - flutter-network

volumes:
  postgres_data:

networks:
  flutter-network:
    driver: bridge
```

### 3. 多环境配置

```yaml
# docker-compose.dev.yml
version: '3.8'

services:
  flutter-web:
    build:
      context: .
      dockerfile: Dockerfile.dev
    volumes:
      - .:/app
      - /app/node_modules
    environment:
      - NODE_ENV=development
      - FLUTTER_WEB_AUTO_DETECT=true
    command: flutter run -d web-server --web-port 3000
    
  flutter-server:
    volumes:
      - .:/app
    environment:
      - NODE_ENV=development
      - DEBUG=true
    command: dart run bin/server.dart --debug
```

```yaml
# docker-compose.prod.yml
version: '3.8'

services:
  flutter-web:
    environment:
      - NODE_ENV=production
      - FLUTTER_WEB_RENDERER=html
    deploy:
      replicas: 3
      resources:
        limits:
          cpus: '0.5'
          memory: 512M
        reservations:
          cpus: '0.25'
          memory: 256M
      restart_policy:
        condition: on-failure
        delay: 5s
        max_attempts: 3
        window: 120s
    
  flutter-server:
    environment:
      - NODE_ENV=production
    deploy:
      replicas: 2
      resources:
        limits:
          cpus: '1.0'
          memory: 1G
        reservations:
          cpus: '0.5'
          memory: 512M
```

## ☁️ 云原生实践

### 1. Kubernetes 部署

#### Deployment 配置

```yaml
# k8s/flutter-web-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: flutter-web
  labels:
    app: flutter-web
spec:
  replicas: 3
  selector:
    matchLabels:
      app: flutter-web
  template:
    metadata:
      labels:
        app: flutter-web
    spec:
      containers:
      - name: flutter-web
        image: flutter-web:latest
        ports:
        - containerPort: 80
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5
        env:
        - name: NODE_ENV
          value: "production"
---
apiVersion: v1
kind: Service
metadata:
  name: flutter-web-service
spec:
  selector:
    app: flutter-web
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: ClusterIP
```

#### Ingress 配置

```yaml
# k8s/ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: flutter-ingress
  annotations:
    kubernetes.io/ingress.class: nginx
    cert-manager.io/cluster-issuer: letsencrypt-prod
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    nginx.ingress.kubernetes.io/force-ssl-redirect: "true"
spec:
  tls:
  - hosts:
    - flutter-app.example.com
    secretName: flutter-tls
  rules:
  - host: flutter-app.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: flutter-web-service
            port:
              number: 80
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: flutter-server-service
            port:
              number: 8080
```

#### ConfigMap 和 Secret

```yaml
# k8s/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: flutter-config
data:
  app.properties: |
    environment=production
    log.level=info
    cache.ttl=3600
  nginx.conf: |
    server {
        listen 80;
        location / {
            try_files $uri $uri/ /index.html;
        }
    }
---
apiVersion: v1
kind: Secret
metadata:
  name: flutter-secrets
type: Opaque
data:
  database-url: cG9zdGdyZXNxbDovL3VzZXI6cGFzc3dvcmRAcG9zdGdyZXM6NTQzMi9mbHV0dGVyX2Ri
  api-key: eW91ci1hcGkta2V5LWhlcmU=
```

### 2. Helm Charts

#### Chart.yaml

```yaml
# helm/flutter-app/Chart.yaml
apiVersion: v2
name: flutter-app
description: A Helm chart for Flutter application
type: application
version: 0.1.0
appVersion: "1.0.0"

dependencies:
  - name: postgresql
    version: 11.6.12
    repository: https://charts.bitnami.com/bitnami
    condition: postgresql.enabled
  - name: redis
    version: 16.13.2
    repository: https://charts.bitnami.com/bitnami
    condition: redis.enabled
```

#### values.yaml

```yaml
# helm/flutter-app/values.yaml
# 默认配置
replicaCount: 3

image:
  repository: flutter-app
  pullPolicy: IfNotPresent
  tag: "latest"

service:
  type: ClusterIP
  port: 80

ingress:
  enabled: true
  className: "nginx"
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
  hosts:
    - host: flutter-app.local
      paths:
        - path: /
          pathType: Prefix
  tls:
    - secretName: flutter-tls
      hosts:
        - flutter-app.local

resources:
  limits:
    cpu: 500m
    memory: 512Mi
  requests:
    cpu: 250m
    memory: 256Mi

autoscaling:
  enabled: true
  minReplicas: 2
  maxReplicas: 10
  targetCPUUtilizationPercentage: 80
  targetMemoryUtilizationPercentage: 80

# PostgreSQL 配置
postgresql:
  enabled: true
  auth:
    postgresPassword: "password"
    username: "flutter"
    password: "password"
    database: "flutter_db"

# Redis 配置
redis:
  enabled: true
  auth:
    enabled: false
```

#### 部署模板

```yaml
# helm/flutter-app/templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "flutter-app.fullname" . }}
  labels:
    {{- include "flutter-app.labels" . | nindent 4 }}
spec:
  {{- if not .Values.autoscaling.enabled }}
  replicas: {{ .Values.replicaCount }}
  {{- end }}
  selector:
    matchLabels:
      {{- include "flutter-app.selectorLabels" . | nindent 6 }}
  template:
    metadata:
      labels:
        {{- include "flutter-app.selectorLabels" . | nindent 8 }}
    spec:
      containers:
        - name: {{ .Chart.Name }}
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag | default .Chart.AppVersion }}"
          imagePullPolicy: {{ .Values.image.pullPolicy }}
          ports:
            - name: http
              containerPort: 80
              protocol: TCP
          livenessProbe:
            httpGet:
              path: /
              port: http
            initialDelaySeconds: 30
            periodSeconds: 10
          readinessProbe:
            httpGet:
              path: /
              port: http
            initialDelaySeconds: 5
            periodSeconds: 5
          resources:
            {{- toYaml .Values.resources | nindent 12 }}
          env:
            - name: DATABASE_URL
              valueFrom:
                secretKeyRef:
                  name: {{ include "flutter-app.fullname" . }}-secrets
                  key: database-url
```

### 3. 服务网格 (Istio)

#### VirtualService 配置

```yaml
# istio/virtual-service.yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: flutter-app
spec:
  hosts:
  - flutter-app.example.com
  gateways:
  - flutter-gateway
  http:
  - match:
    - uri:
        prefix: /api
    route:
    - destination:
        host: flutter-server-service
        port:
          number: 8080
    fault:
      delay:
        percentage:
          value: 0.1
        fixedDelay: 5s
    retries:
      attempts: 3
      perTryTimeout: 2s
  - match:
    - uri:
        prefix: /
    route:
    - destination:
        host: flutter-web-service
        port:
          number: 80
```

#### DestinationRule 配置

```yaml
# istio/destination-rule.yaml
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: flutter-app
spec:
  host: flutter-web-service
  trafficPolicy:
    loadBalancer:
      simple: LEAST_CONN
    connectionPool:
      tcp:
        maxConnections: 100
      http:
        http1MaxPendingRequests: 50
        maxRequestsPerConnection: 10
    circuitBreaker:
      consecutiveErrors: 3
      interval: 30s
      baseEjectionTime: 30s
      maxEjectionPercent: 50
  subsets:
  - name: v1
    labels:
      version: v1
  - name: v2
    labels:
      version: v2
```

## 🏗️ 基础设施即代码 (IaC)

### 1. Terraform 配置

#### 主配置文件

```hcl
# terraform/main.tf
terraform {
  required_version = ">= 1.0"
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
    kubernetes = {
      source  = "hashicorp/kubernetes"
      version = "~> 2.0"
    }
  }
  
  backend "s3" {
    bucket = "flutter-terraform-state"
    key    = "flutter-app/terraform.tfstate"
    region = "us-west-2"
  }
}

provider "aws" {
  region = var.aws_region
}

provider "kubernetes" {
  host                   = module.eks.cluster_endpoint
  cluster_ca_certificate = base64decode(module.eks.cluster_certificate_authority_data)
  token                  = data.aws_eks_cluster_auth.cluster.token
}

data "aws_eks_cluster_auth" "cluster" {
  name = module.eks.cluster_id
}
```

#### VPC 模块

```hcl
# terraform/modules/vpc/main.tf
resource "aws_vpc" "main" {
  cidr_block           = var.vpc_cidr
  enable_dns_hostnames = true
  enable_dns_support   = true
  
  tags = {
    Name = "${var.project_name}-vpc"
    Environment = var.environment
  }
}

resource "aws_subnet" "private" {
  count = length(var.private_subnet_cidrs)
  
  vpc_id            = aws_vpc.main.id
  cidr_block        = var.private_subnet_cidrs[count.index]
  availability_zone = var.availability_zones[count.index]
  
  tags = {
    Name = "${var.project_name}-private-${count.index + 1}"
    Environment = var.environment
    Type = "private"
  }
}

resource "aws_subnet" "public" {
  count = length(var.public_subnet_cidrs)
  
  vpc_id                  = aws_vpc.main.id
  cidr_block              = var.public_subnet_cidrs[count.index]
  availability_zone       = var.availability_zones[count.index]
  map_public_ip_on_launch = true
  
  tags = {
    Name = "${var.project_name}-public-${count.index + 1}"
    Environment = var.environment
    Type = "public"
  }
}

resource "aws_internet_gateway" "main" {
  vpc_id = aws_vpc.main.id
  
  tags = {
    Name = "${var.project_name}-igw"
    Environment = var.environment
  }
}

resource "aws_nat_gateway" "main" {
  count = length(aws_subnet.public)
  
  allocation_id = aws_eip.nat[count.index].id
  subnet_id     = aws_subnet.public[count.index].id
  
  tags = {
    Name = "${var.project_name}-nat-${count.index + 1}"
    Environment = var.environment
  }
  
  depends_on = [aws_internet_gateway.main]
}

resource "aws_eip" "nat" {
  count = length(aws_subnet.public)
  
  domain = "vpc"
  
  tags = {
    Name = "${var.project_name}-eip-${count.index + 1}"
    Environment = var.environment
  }
}
```

#### EKS 集群配置

```hcl
# terraform/modules/eks/main.tf
resource "aws_eks_cluster" "main" {
  name     = var.cluster_name
  role_arn = aws_iam_role.cluster.arn
  version  = var.kubernetes_version
  
  vpc_config {
    subnet_ids              = var.subnet_ids
    endpoint_private_access = true
    endpoint_public_access  = true
    public_access_cidrs     = var.public_access_cidrs
  }
  
  encryption_config {
    provider {
      key_arn = aws_kms_key.eks.arn
    }
    resources = ["secrets"]
  }
  
  enabled_cluster_log_types = ["api", "audit", "authenticator", "controllerManager", "scheduler"]
  
  depends_on = [
    aws_iam_role_policy_attachment.cluster_AmazonEKSClusterPolicy,
    aws_cloudwatch_log_group.cluster,
  ]
  
  tags = {
    Name = var.cluster_name
    Environment = var.environment
  }
}

resource "aws_eks_node_group" "main" {
  cluster_name    = aws_eks_cluster.main.name
  node_group_name = "${var.cluster_name}-nodes"
  node_role_arn   = aws_iam_role.node_group.arn
  subnet_ids      = var.private_subnet_ids
  
  capacity_type  = "ON_DEMAND"
  instance_types = var.node_instance_types
  
  scaling_config {
    desired_size = var.node_desired_size
    max_size     = var.node_max_size
    min_size     = var.node_min_size
  }
  
  update_config {
    max_unavailable = 1
  }
  
  depends_on = [
    aws_iam_role_policy_attachment.node_group_AmazonEKSWorkerNodePolicy,
    aws_iam_role_policy_attachment.node_group_AmazonEKS_CNI_Policy,
    aws_iam_role_policy_attachment.node_group_AmazonEC2ContainerRegistryReadOnly,
  ]
  
  tags = {
    Name = "${var.cluster_name}-nodes"
    Environment = var.environment
  }
}
```

### 2. Ansible 配置管理

#### Playbook 主文件

```yaml
# ansible/site.yml
---
- name: Deploy Flutter Application
  hosts: all
  become: yes
  vars:
    app_name: flutter-app
    app_version: "{{ version | default('latest') }}"
    environment: "{{ env | default('production') }}"
  
  roles:
    - common
    - docker
    - kubernetes
    - monitoring
    - security
  
  tasks:
    - name: Deploy application
      include_tasks: tasks/deploy.yml
      tags: deploy
```

#### Docker 角色

```yaml
# ansible/roles/docker/tasks/main.yml
---
- name: Install Docker dependencies
  apt:
    name:
      - apt-transport-https
      - ca-certificates
      - curl
      - gnupg
      - lsb-release
    state: present
    update_cache: yes

- name: Add Docker GPG key
  apt_key:
    url: https://download.docker.com/linux/ubuntu/gpg
    state: present

- name: Add Docker repository
  apt_repository:
    repo: "deb [arch=amd64] https://download.docker.com/linux/ubuntu {{ ansible_distribution_release }} stable"
    state: present

- name: Install Docker
  apt:
    name:
      - docker-ce
      - docker-ce-cli
      - containerd.io
    state: present
    update_cache: yes

- name: Start and enable Docker service
  systemd:
    name: docker
    state: started
    enabled: yes

- name: Add user to docker group
  user:
    name: "{{ ansible_user }}"
    groups: docker
    append: yes
```

#### 部署任务

```yaml
# ansible/tasks/deploy.yml
---
- name: Create application directory
  file:
    path: "/opt/{{ app_name }}"
    state: directory
    owner: "{{ ansible_user }}"
    group: "{{ ansible_user }}"
    mode: '0755'

- name: Copy docker-compose file
  template:
    src: docker-compose.yml.j2
    dest: "/opt/{{ app_name }}/docker-compose.yml"
    owner: "{{ ansible_user }}"
    group: "{{ ansible_user }}"
    mode: '0644'
  notify: restart application

- name: Copy environment file
  template:
    src: .env.j2
    dest: "/opt/{{ app_name }}/.env"
    owner: "{{ ansible_user }}"
    group: "{{ ansible_user }}"
    mode: '0600'
  notify: restart application

- name: Pull latest images
  docker_compose:
    project_src: "/opt/{{ app_name }}"
    pull: yes
  tags: deploy

- name: Start application
  docker_compose:
    project_src: "/opt/{{ app_name }}"
    state: present
  tags: deploy
```

## 🔒 安全实践

### 1. 容器安全

#### 安全 Dockerfile

```dockerfile
# Dockerfile.secure
FROM flutter:3.10-alpine AS build

# 创建非 root 用户
RUN addgroup -g 1001 -S flutter && \
    adduser -S flutter -u 1001 -G flutter

# 设置工作目录
WORKDIR /app

# 复制文件并设置权限
COPY --chown=flutter:flutter pubspec.* ./
RUN flutter pub get

COPY --chown=flutter:flutter . .
RUN flutter build web --release

# 生产镜像
FROM nginx:alpine

# 安装安全更新
RUN apk update && apk upgrade && \
    apk add --no-cache dumb-init && \
    rm -rf /var/cache/apk/*

# 创建非 root 用户
RUN addgroup -g 1001 -S nginx && \
    adduser -S nginx -u 1001 -G nginx

# 复制应用文件
COPY --from=build --chown=nginx:nginx /app/build/web /usr/share/nginx/html
COPY --chown=nginx:nginx nginx.conf /etc/nginx/nginx.conf

# 设置正确的权限
RUN chown -R nginx:nginx /usr/share/nginx/html && \
    chown -R nginx:nginx /var/cache/nginx && \
    chown -R nginx:nginx /var/log/nginx && \
    chown -R nginx:nginx /etc/nginx/conf.d

# 切换到非 root 用户
USER nginx

# 使用 dumb-init
ENTRYPOINT ["/usr/bin/dumb-init", "--"]
CMD ["nginx", "-g", "daemon off;"]
```

### 2. Kubernetes 安全策略

#### Pod Security Policy

```yaml
# k8s/security/pod-security-policy.yaml
apiVersion: policy/v1beta1
kind: PodSecurityPolicy
metadata:
  name: flutter-app-psp
spec:
  privileged: false
  allowPrivilegeEscalation: false
  requiredDropCapabilities:
    - ALL
  volumes:
    - 'configMap'
    - 'emptyDir'
    - 'projected'
    - 'secret'
    - 'downwardAPI'
    - 'persistentVolumeClaim'
  runAsUser:
    rule: 'MustRunAsNonRoot'
  seLinux:
    rule: 'RunAsAny'
  fsGroup:
    rule: 'RunAsAny'
  readOnlyRootFilesystem: true
```

#### Network Policy

```yaml
# k8s/security/network-policy.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: flutter-app-netpol
spec:
  podSelector:
    matchLabels:
      app: flutter-app
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: ingress-nginx
    ports:
    - protocol: TCP
      port: 80
  egress:
  - to:
    - namespaceSelector:
        matchLabels:
          name: kube-system
    ports:
    - protocol: TCP
      port: 53
    - protocol: UDP
      port: 53
  - to:
    - podSelector:
        matchLabels:
          app: postgres
    ports:
    - protocol: TCP
      port: 5432
```

### 3. 密钥管理

#### Vault 集成

```dart
// lib/security/vault_client.dart
class VaultClient {
  static const String _baseUrl = 'https://vault.example.com';
  static const String _namespace = 'flutter-app';
  
  static Future<String> getSecret(String path) async {
    final token = await _getVaultToken();
    
    final response = await http.get(
      Uri.parse('$_baseUrl/v1/secret/data/$path'),
      headers: {
        'X-Vault-Token': token,
        'X-Vault-Namespace': _namespace,
      },
    );
    
    if (response.statusCode == 200) {
      final data = json.decode(response.body);
      return data['data']['data']['value'];
    } else {
      throw Exception('Failed to retrieve secret: ${response.statusCode}');
    }
  }
  
  static Future<String> _getVaultToken() async {
    // 使用 Kubernetes Service Account 认证
    final jwt = await File('/var/run/secrets/kubernetes.io/serviceaccount/token')
        .readAsString();
    
    final response = await http.post(
      Uri.parse('$_baseUrl/v1/auth/kubernetes/login'),
      headers: {'Content-Type': 'application/json'},
      body: json.encode({
        'role': 'flutter-app',
        'jwt': jwt,
      }),
    );
    
    if (response.statusCode == 200) {
      final data = json.decode(response.body);
      return data['auth']['client_token'];
    } else {
      throw Exception('Failed to authenticate with Vault');
    }
  }
}
```

## 🚀 最佳实践

### 1. 容器化最佳实践

- **多阶段构建**: 减少镜像大小，提高安全性
- **非 root 用户**: 避免使用 root 用户运行容器
- **最小化镜像**: 使用 Alpine 等轻量级基础镜像
- **安全扫描**: 定期扫描镜像漏洞

### 2. Kubernetes 最佳实践

- **资源限制**: 设置合理的 CPU 和内存限制
- **健康检查**: 配置 liveness 和 readiness 探针
- **滚动更新**: 使用滚动更新策略确保零停机部署
- **命名空间隔离**: 使用命名空间隔离不同环境

### 3. 安全最佳实践

- **最小权限原则**: 只授予必要的权限
- **网络隔离**: 使用网络策略限制流量
- **密钥轮换**: 定期轮换密钥和证书
- **审计日志**: 启用审计日志记录

### 4. 监控和可观测性

- **指标收集**: 收集应用和基础设施指标
- **日志聚合**: 集中收集和分析日志
- **分布式追踪**: 实现请求链路追踪
- **告警机制**: 设置合理的告警规则

通过完善的基础设施管理，可以实现Flutter应用的自动化部署、弹性伸缩、安全防护和高可用性，为应用的稳定运行提供坚实的基础。