# The Full Site Development Project of the E-commerce Mall

基于 Google Cloud 微服务架构的全栈电商商城项目。

## 项目简介

本项目是 **Online Boutique** 的完整部署版本，包含 11 个微服务，使用多种编程语言开发，通过 gRPC 进行服务间通信。

## 技术栈

- **容器编排**: Kubernetes (GKE)
- **服务网格**: Cloud Service Mesh
- **通信协议**: gRPC
- **监控**: Cloud Operations
- **数据库**: Redis, Spanner, AlloyDB

## 微服务架构

| 服务 | 语言 | 说明 |
|------|------|------|
| frontend | Go | 前端 HTTP 服务 |
| checkoutservice | Go | 订单结算服务 |
| cartservice | C# | 购物车服务 (Redis) |
| productcatalogservice | Go | 商品目录服务 |
| currencyservice | Node.js | 货币转换服务 |
| paymentservice | Node.js | 支付服务 |
| shippingservice | Go | 物流服务 |
| emailservice | Python | 邮件服务 |
| recommendationservice | Python | 推荐服务 |
| adservice | Java | 广告服务 |
| loadgenerator | Python/Locust | 负载生成器 |

## 快速开始

### 前置要求

- Google Cloud 项目
- `gcloud`, `git`, `kubectl` 命令行工具

### 部署到 GKE

```bash
# 设置项目和区域
export PROJECT_ID=<your-project-id>
export REGION=us-central1

# 启用 GKE API
gcloud services enable container.googleapis.com --project=${PROJECT_ID}

# 创建 GKE 集群
gcloud container clusters create-auto online-boutique \
  --project=${PROJECT_ID} --region=${REGION}

# 部署应用
kubectl apply -f microservices-demo/release/kubernetes-manifests.yaml

# 等待 Pod 就绪
kubectl get pods

# 访问应用
kubectl port-forward svc/frontend 8080:80
# 浏览器访问 http://localhost:8080
```

### 本地开发 (Skaffold)

```bash
cd microservices-demo
skaffold run          # 构建并部署
skaffold dev          # 开发模式，自动重载
skaffold delete       # 清理资源
```

## 项目结构

```
├── microservices-demo/        # 主项目目录
│   ├── src/                   # 源代码
│   ├── kubernetes-manifests/  # K8s 配置
│   ├── terraform/             # 基础设施配置
│   ├── helm-chart/            # Helm 图表
│   └── docs/                  # 文档
└── README.md
```

## 相关文档

- [技术说明文档](microservices-demo/技术说明文档.md)
- [交互式启动教程](microservices-demo/交互式启动教程.html)

## 许可证

Apache License 2.0
