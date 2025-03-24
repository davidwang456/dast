# dast
Acunetix automatically creates a list of all your websites, applications, and APIs
https://www.acunetix.com/product/#
# Acunetix 网络应用安全扫描工具

Acunetix 是一款专业的网络应用和 API 安全扫描工具，旨在帮助企业自动化应用安全流程，通过简单的 6 个步骤保护网络应用和 API 安全。

## 核心功能与优势

1. **发现与爬取**
   - 自动创建并更新所有网站、应用和 API 的列表
   - 扫描单页应用(SPA)、脚本密集型网站以及 HTML5 和 JavaScript 构建的应用
   - 通过宏录制功能自动扫描受密码保护和难以访问的区域
   - 扫描其他扫描器无法看到的未链接文件和 API 端点

2. **风险评估**
   - 利用专有的机器学习模型进行预测性风险评分
   - 在漏洞扫描开始前快速准确地估计网络应用的风险级别
   - 基于超过 200 个网站和应用的外部特征计算风险分数
   - 提供更广泛且准确的攻击面视图，改进攻击面管理(ASM)

3. **漏洞检测**
   - 检测超过 12,000 种漏洞，包括零日漏洞
   - 使用世界上最准确的漏洞扫描器查找安全缺陷
   - 运行快速扫描，即时发现漏洞
   - 同时扫描多个环境
   - 通过混合 DAST + IAST 扫描提供更全面的覆盖

4. **快速修复**
   - 消除耗时的误报，提供漏洞利用证明
   - 精确定位需要修复的代码行
   - 使开发人员能够自行解决安全问题

5. **集成安全到开发流程**
   - 一键向开发人员发送工单
   - 帮助开发人员编写更安全的代码以预防漏洞
   - 连接到 CI/CD、问题跟踪器、WAF 和其他工具

6. **持续安全**
   - 安排定期漏洞扫描
   - 通过趋势图表观察应用安全性的改进情况
   - 集成 WAF 以在修复漏洞期间保持安全

## 客户优势

Acunetix 已被全球 2,300 多家各种规模的公司使用，客户反馈包括：
- 大幅减少识别网络威胁所需的时间
- 快速轻松地识别和修复网络应用中的漏洞
- 提高应用安全测试的信心
- 在开发流程中集成安全测试，降低安全风险

Acunetix 由 Invicti 提供支持，是企业确保网络应用和 API 安全的理想解决方案。 

# Acunetix 部署指南：Docker、Docker-Compose 和 Kubernetes

## 一、Docker 部署方式

### 1. 获取 Acunetix 镜像

Acunetix 提供两种方式获取 Docker 镜像：

#### 方式一：从 Invicti 注册表获取

```bash
# 登录 Invicti 注册表
docker login registry.invicti.com

# 登录时使用以下凭据：
# 用户名：您的 Acunetix 登录邮箱
# 密码：您的 Acunetix 许可证密钥（可在 Settings > Subscription 中找到）

# 拉取最新镜像
docker pull registry.invicti.com/acunetix/wvs:24.10
```

#### 方式二：从 Repo One 获取（适用于政府/军方用户）

```bash
# 登录 Repo One 注册表
docker login registry1.dso.mil

# 拉取镜像
docker pull registry1.dso.mil/ironbank/invicti/acunetix/wvs:latest
```

### 2. 运行 Docker 容器

```bash
# 创建用于存储数据的目录
mkdir -p /path/to/acunetix-data

# 运行 PostgreSQL 数据库容器
docker run -d --name acunetix-database \
  -e POSTGRES_USER=acunetix \
  -e POSTGRES_PASSWORD=your_secure_password \
  -e POSTGRES_DB=wvs \
  -v acunetix-db-data:/data \
  -p 5432:5432 \
  postgres:13

# 运行 Acunetix 主容器
docker run -d --name acunetix-main \
  --link acunetix-database:acunetix-database \
  -e acunetix_user_data=/home/acunetix/user-data \
  -e acunetix_database=postgresql://acunetix:your_secure_password@acunetix-database:5432/wvs \
  -e acunetix_user=your_email@example.com \
  -e acunetix_password=your_secure_password \
  -e acunetix_logging_console_level=DEBUG \
  -v /path/to/acunetix-data:/home/acunetix/user-data \
  -p 3500:3443 \
  registry.invicti.com/acunetix/wvs:24.10
```

### 3. 环境变量说明

主要环境变量：
- `acunetix_user_data`：指定后端存储用户数据的位置
- `acunetix_database`：PostgreSQL 数据库连接字符串
- `acunetix_user`：主用户的电子邮件地址
- `acunetix_password`：主用户的密码
- `acunetix_logging_console_level`：控制台输出的日志级别（可选，默认为 DEBUG）
- `acunetix_ssl_certificate`：SSL 证书的位置（可选）
- `acunetix_ssl_private_key`：SSL 证书私钥的位置（可选）
- `acunetix_engineonly`：设置为 1 表示实例作为工作节点运行（可选）
- `acunetix_host`：Acunetix 安装的 IP 地址或 FQDN（可选）
- `acunetix_port`：Acunetix 安装的端口号，必须大于 1024（可选）

## 二、Docker-Compose 部署方式

### 1. 创建 docker-compose.yml 文件

创建一个包含以下内容的 `docker-compose.yml` 文件：

```yaml
version: "3"
services:
  adjust-permissions:
    image: busybox
    entrypoint: 'sh -c "chown -R 9900:9900 /user-data && chown -R 9900:9900 /worker-data"'
    restart: 'no'
    volumes:
      - acunetix-user-data:/user-data
      - acunetix-worker-data:/worker-data

  acunetix-database:
    image: postgres:13
    restart: unless-stopped
    environment:
      POSTGRES_USER: acunetix
      POSTGRES_PASSWORD: your_secure_password
      POSTGRES_DB: wvs
      PGDATA: /data/postgres
    volumes:
      - acunetix-db-data:/data
    ports:
      - "5432:5432"
  
  acunetix-main:
    restart: unless-stopped
    depends_on:
      - adjust-permissions
      - acunetix-database
    image: registry.invicti.com/acunetix/wvs:24.10
    environment:
      acunetix_user_data: /home/acunetix/user-data
      acunetix_database: postgresql://acunetix:your_secure_password@acunetix-database:5432/wvs
      acunetix_user: your_email@example.com
      acunetix_password: your_secure_password
      acunetix_logging_console_level: DEBUG
      acunetix_ssl_certificate: /home/acunetix/user-data/certs/server.cer
      acunetix_ssl_private_key: /home/acunetix/user-data/certs/server.key
    volumes:
      - acunetix-user-data:/home/acunetix/user-data
    ports:
      - "0.0.0.0:3500:3443"
      - "0.0.0.0:7900:7880"

  acunetix-worker:
    restart: unless-stopped
    depends_on:
      - adjust-permissions
      - acunetix-database
      - acunetix-main
    image: registry.invicti.com/acunetix/wvs:24.10
    environment:
      acunetix_user_data: /home/acunetix/worker-data
      acunetix_ssl_certificate: /home/acunetix/user-data/certs/server.cer
      acunetix_ssl_private_key: /home/acunetix/user-data/certs/server.key
      acunetix_engineonly: 1
      acunetix_logging_console_level: DEBUG
      acunetix_main_backend_url: https://acunetix-main:3443
    volumes:
      - acunetix-worker-data:/home/acunetix/worker-data
    ports:
      - "0.0.0.0:3501:3443"

volumes:
  acunetix-db-data:
  acunetix-user-data:
  acunetix-worker-data:
```

### 2. 启动服务

```bash
# 启动所有服务
docker-compose up -d

# 查看服务状态
docker-compose ps
```

### 3. 工作节点配置说明

在此设置中，工作节点和主容器通过 Docker 网络进行通信：

- 内部通信：
  - 使用主机名 acunetix-worker 和 acunetix-main，端口 3443
- 外部访问：
  - 主容器：使用转发端口 3500
  - 工作节点：使用转发端口 3501

## 三、Kubernetes 部署方式

### 1. 准备 Kubernetes 配置文件

需要创建以下文件：

#### `acunetix-namespace.yaml`
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: dast
```

#### `secret.yaml`
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: acunetix-secret
  namespace: dast
type: Opaque
data:
  db-password: cm9vdA==  
  user-email: bXRsQGhrZXguY29tLmhk  
  user-password: YWRtaW4=  
```

#### `db/database-service.yaml`
```yaml
apiVersion: v1
kind: Service
metadata:
  name: acunetix-database
  namespace: dast
spec:
  ports:
  - port: 5432
    targetPort: 5432
    protocol: TCP
  selector:
    app: acunetix-database
```

#### `db/database-vc.yaml`
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: acunetix-db-data
  namespace: dast
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
```

#### `db/database-deployment.yaml`
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: acunetix-database
  namespace: dast
spec:
  replicas: 1
  selector:
    matchLabels:
      app: acunetix-database
  template:
    metadata:
      labels:
        app: acunetix-database
    spec:
      containers:
      - name: postgres
        image: postgres:13
        env:
        - name: POSTGRES_USER
          value: acunetix
        - name: POSTGRES_PASSWORD
          valueFrom:
            secretKeyRef:
              name: acunetix-secret
              key: db-password
        - name: POSTGRES_DB
          value: wvs
        - name: PGDATA
          value: /data/postgres
        ports:
        - containerPort: 5432
        volumeMounts:
        - name: acunetix-db-data
          mountPath: /data
      volumes:
      - name: acunetix-db-data
        persistentVolumeClaim:
          claimName: acunetix-db-data
```

#### `backend/user-data-vc.yaml`
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: acunetix-user-data
  namespace: dast
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 20Gi
```

#### `backend/main-backend-service.yaml`
```yaml
apiVersion: v1
kind: Service
metadata:
  name: acunetix-main
  namespace: dast
spec:
  type: NodePort
  ports:
  - port: 3443
    targetPort: 3443
    nodePort: 30443
    protocol: TCP
  selector:
    app: acunetix-main
```

#### `backend/main-backend-deployment.yaml`
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: acunetix-main
  namespace: dast
spec:
  replicas: 1
  selector:
    matchLabels:
      app: acunetix-main
  template:
    metadata:
      labels:
        app: acunetix-main
    spec:
      initContainers:
      - name: adjust-permissions
        image: busybox
        command: ['sh', '-c', 'chown -R 9900:9900 /home/acunetix/user-data']
        volumeMounts:
        - name: acunetix-user-data
          mountPath: /home/acunetix/user-data
      containers:
      - name: acunetix-main
        image: registry.invicti.com/acunetix/wvs:24.10
        env:
        - name: acunetix_user_data
          value: /home/acunetix/user-data
        - name: acunetix_database
          value: postgresql://acunetix:$(DB_PASSWORD)@acunetix-database:5432/wvs
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: acunetix-secret
              key: db-password
        - name: acunetix_user
          valueFrom:
            secretKeyRef:
              name: acunetix-secret
              key: user-email
        - name: acunetix_password
          valueFrom:
            secretKeyRef:
              name: acunetix-secret
              key: user-password
        - name: acunetix_logging_console_level
          value: DEBUG
        ports:
        - containerPort: 3443
        volumeMounts:
        - name: acunetix-user-data
          mountPath: /home/acunetix/user-data
      volumes:
      - name: acunetix-user-data
        persistentVolumeClaim:
          claimName: acunetix-user-data
```

#### `backend/worker-deployment.yaml`
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: acunetix-worker
  namespace: dast
spec:
  replicas: 1
  selector:
    matchLabels:
      app: acunetix-worker
  template:
    metadata:
      labels:
        app: acunetix-worker
    spec:
      initContainers:
      - name: adjust-permissions
        image: busybox
        command: ['sh', '-c', 'mkdir -p /home/acunetix/worker-data && chown -R 9900:9900 /home/acunetix/worker-data']
        volumeMounts:
        - name: acunetix-worker-data
          mountPath: /home/acunetix/worker-data
      containers:
      - name: acunetix-worker
        image: registry.invicti.com/acunetix/wvs:24.10
        env:
        - name: acunetix_user_data
          value: /home/acunetix/worker-data
        - name: acunetix_engineonly
          value: "1"
        - name: acunetix_logging_console_level
          value: DEBUG
        - name: acunetix_main_backend_url
          value: https://acunetix-main:3443
        volumeMounts:
        - name: acunetix-worker-data
          mountPath: /home/acunetix/worker-data
      volumes:
      - name: acunetix-worker-data
        emptyDir: {}
```

### 2. 部署 Acunetix 到 Kubernetes

```bash
# 创建命名空间
kubectl apply -f acunetix-namespace.yaml

# 创建 Secret
kubectl apply -f secret.yaml

# 部署数据库
kubectl apply -f db/database-vc.yaml
kubectl apply -f db/database-service.yaml
kubectl apply -f db/database-deployment.yaml

# 等待数据库启动
kubectl -n dast wait --for=condition=ready pod -l app=acunetix-database --timeout=90s

# 部署主后端
kubectl apply -f backend/user-data-vc.yaml
kubectl apply -f backend/main-backend-service.yaml
kubectl apply -f backend/main-backend-deployment.yaml

# 等待主后端启动
kubectl -n dast wait --for=condition=ready pod -l app=acunetix-main --timeout=120s

# 部署工作节点
kubectl apply -f backend/worker-deployment.yaml
```

### 3. 访问 Acunetix 服务

通过以下命令获取访问 URL：

```bash
# 获取节点 IP
NODE_IP=$(kubectl get nodes -o jsonpath='{.items[0].status.addresses[?(@.type=="ExternalIP")].address}')

# 服务 URL
echo "Acunetix 服务访问地址: https://${NODE_IP}:30443"
```

## 四、自动更新

Acunetix Docker 镜像不支持自动更新，因为所有版本都是固定的。如需定期更新服务，可以使用 Watchtower 等工具：

```bash
# 运行 Watchtower 以自动更新容器
docker run -d \
  --name watchtower \
  -v /var/run/docker.sock:/var/run/docker.sock \
  containrrr/watchtower \
  --interval 86400 \
  acunetix-main acunetix-worker
```

## 注意事项

1. 在生产环境中，请确保使用强密码并妥善保管
2. 建议配置 SSL 证书以确保安全通信
3. 定期备份数据库和用户数据卷
4. Kubernetes 部署建议在生产环境中使用更完善的存储解决方案 
