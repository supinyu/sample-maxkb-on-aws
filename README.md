# maxkb-on-aws 

## 1、AWS 部署架构说明

本项目采用微服务架构，在AWS上部署MaxKB知识库系统。以下是各组件的部署顺序和说明：

## 2、部署架构图

![部署架构图](images/maxkb.png)

## 3、架构组件部署顺序

### 3-1、VPC 网络层 (`vpc-stack.ts`)
- **创建或使用现有VPC**：支持3个可用区的高可用部署
- **子网配置**：
  - 公有子网（Public Subnet）：部署ALB负载均衡器
  - 私有子网（Private Subnet）：部署ECS任务、RDS数据库、Redis缓存
- **NAT网关**：为私有子网提供互联网出站访问

### 3-2. 安全组层(`security-group-stack.ts`)
- **ALB安全组**：允许来自互联网的HTTP(80)/HTTPS(443)访问
- **ECS安全组**：只允许来自ALB的8080端口访问
- **RDS安全组**：只允许来自ECS的5432端口（PostgreSQL）访问
- **Redis安全组**：只允许来自ECS的6379端口访问

### 3-3. 数据存储层 (`rds-stack.ts` & `redis-stack.ts`)
- **PostgreSQL RDS**：
  - 引擎版本：PostgreSQL 15
  - 实例类型：r6g.large
  - 多可用区部署（高可用）
  - 启用pgvector扩展支持向量检索
  - 自动备份和性能监控
- **Valkey/Redis集群**：
  - 引擎：Valkey 8.0（Redis兼容）
  - 1主2从的高可用配置
  - 自动故障转移

### 3-4. 应用服务层 (`ecs-stack.ts`)
- **ECS Fargate集群**：无服务器容器服务
- **应用负载均衡器（ALB）**：面向互联网的负载均衡
- **ECS任务定义**：
  - CPU：4vCPU，内存：8GB
  - 临时存储：100GB
  - 期望实例数：2（高可用）
- **健康检查**：监控应用状态
- **自动扩展**：基于负载自动调整实例数

### 3-5. 密钥管理
- **AWS Secrets Manager**：安全存储数据库连接信息
- **环境变量注入**：通过Secrets Manager向容器注入敏感配置

## 4、数据流向

```
用户请求 → ALB（公有子网）→ ECS任务（私有子网）→ RDS/Redis（私有子网）
```



## 5、部署命令

- 安装docker

```
sudo yum install docker python3-pip git npm -y

# Configure Docker Components
sudo systemctl enable docker

sudo systemctl start docker

sudo usermod -aG docker $USER

newgrp docker
```

- 安装aws cdk

```
sudo npm install -g aws-cdk
```

- 安装相关的npm包

```
cd maxkb-cdk

npm install aws-cdk-lib

```

- 将您的账号信息以及需要部署的Region ID导入到环境变量中

```
export AWS_ACCOUNT_ID=XXXXXXXXXXXX
export AWS_REGION=us-west-2
```

- 进行CDK部署

```
cdk bootstrap aws://$AWS_ACCOUNT_ID/$AWS_REGION 

cdk deploy MaxKBCdkStack --require-approval never
```


## 6、如何更新MaxKB的版本

修改Dockerfile文件，中的镜像版本


