# Online Boutique 微服务电商项目 技术文档

## 1. 项目概述

本项目是 Google 官方提供的云原生微服务电商演示应用，包含 11 个独立微服务，使用 5 种编程语言开发，通过 gRPC 进行服务间通信，部署在 Kubernetes 集群上。

- **仓库地址**: https://github.com/GoogleCloudPlatform/microservices-demo
- **许可证**: Apache License 2.0
- **部署路径**: `/Users/kaisery/凯kk/AI-agent/Agent工具/first-cc/实战项目/SDE后端/全栈开发项目/microservices-demo`

---

## 2. 项目结构

```
microservices-demo/
├── src/                            # 微服务源代码
│   ├── frontend/                   # Go - 前端 HTTP 服务
│   ├── checkoutservice/            # Go - 订单结算服务
│   ├── cartservice/                # C# - 购物车服务 (Redis)
│   ├── productcatalogservice/      # Go - 商品目录服务
│   ├── currencyservice/            # Node.js - 货币转换服务
│   ├── paymentservice/             # Node.js - 支付服务
│   ├── shippingservice/            # Go - 物流服务
│   ├── emailservice/               # Python - 邮件服务
│   ├── recommendationservice/      # Python - 推荐服务
│   ├── adservice/                  # Java - 广告服务
│   └── loadgenerator/              # Python/Locust - 负载生成器
├── kubernetes-manifests/           # Kubernetes 部署配置
├── release/                        # 生产级部署清单
├── terraform/                      # 基础设施即代码
├── helm-chart/                     # Helm 图表
├── kustomize/                      # Kustomize 配置
├── protos/                         # gRPC Protocol Buffers 定义
├── docs/                           # 项目文档
└── skaffold.yaml                   # Skaffold 开发配置
```

---

## 3. 技术栈

### 3.1 微服务

| 服务 | 语言 | 说明 | 关键依赖 |
|------|------|------|----------|
| frontend | Go | HTTP 服务器，提供 Web 界面 | HTML Templates |
| checkoutservice | Go | 订单编排，协调多个服务 | gRPC |
| cartservice | C# | 购物车存储 | Redis |
| productcatalogservice | Go | 商品目录管理 | JSON 配置 |
| currencyservice | Node.js | 实时汇率转换 | ECB API |
| paymentservice | Node.js | 支付处理（模拟） | - |
| shippingservice | Go | 物流估算（模拟） | - |
| emailservice | Python | 邮件发送（模拟） | - |
| recommendationservice | Python | 商品推荐 | - |
| adservice | Java | 广告投放 | - |
| loadgenerator | Python/Locust | 压力测试 | Locust |

### 3.2 基础设施

| 组件 | 技术 |
|------|------|
| 容器化 | Docker |
| 编排 | Kubernetes (GKE) |
| 服务网格 | Cloud Service Mesh |
| 通信协议 | gRPC + Protocol Buffers |
| 缓存 | Redis |
| 监控 | Cloud Operations (Tracing, Profiling) |
| CI/CD | GitHub Actions |
| IaC | Terraform |
| 包管理 | Helm, Kustomize |

---

## 4. 核心架构：微服务协作流程

### 4.1 请求流转图

```
用户浏览器
    │
    ▼
┌─────────────────┐
│   frontend      │  ← HTTP 服务器，提供 Web 界面
│   (Go)          │
└────────┬────────┘
         │ gRPC
         ▼
┌─────────────────┐
│ checkoutservice │  ← 订单编排，协调以下服务
│   (Go)          │
└────────┬────────┘
         │
    ┌────┴────┬──────────┬──────────┬──────────┐
    ▼         ▼          ▼          ▼          ▼
┌───────┐ ┌───────┐ ┌───────┐ ┌───────┐ ┌───────┐
│ cart  │ │payment│ │ship   │ │email  │ │currency│
│service│ │service│ │service│ │service│ │service │
│ (C#)  │ │(Node) │ │ (Go)  │ │(Py)   │ │(Node)  │
└───┬───┘ └───────┘ └───────┘ └───────┘ └───────┘
    │
    ▼
┌───────┐
│ Redis │  ← 购物车数据存储
└───────┘
```

### 4.2 服务详解

#### frontend（前端服务）
- **语言**: Go
- **端口**: 8080
- **功能**: 提供 HTML 页面，处理用户请求
- **依赖**: productcatalogservice, cartservice, currencyservice, recommendationservice

#### checkoutservice（订单结算服务）
- **语言**: Go
- **功能**: 编排整个下单流程
- **流程**: 查购物车 → 算总价 → 扣款 → 发货 → 清空购物车 → 发邮件
- **依赖**: cartservice, paymentservice, shippingservice, emailservice, currencyservice

#### cartservice（购物车服务）
- **语言**: C#
- **存储**: Redis
- **内存需求**: ≥256Mi（默认 64Mi 会导致 OOM）
- **功能**: 管理用户购物车数据

#### currencyservice（货币服务）
- **语言**: Node.js
- **数据源**: 欧洲央行实时汇率
- **特性**: 最高 QPS 服务

---

## 5. 快速开始

### 5.1 前置要求

- Google Cloud 项目
- `gcloud`, `git`, `kubectl` 命令行工具

### 5.2 部署到 GKE

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
kubectl apply -f release/kubernetes-manifests.yaml

# 等待 Pod 就绪
kubectl get pods

# 访问应用
kubectl port-forward svc/frontend 8080:80
# 浏览器访问 http://localhost:8080
```

### 5.3 本地开发 (Skaffold)

```bash
cd microservices-demo
skaffold run          # 构建并部署（首次约 20 分钟）
skaffold dev          # 开发模式，自动重载
skaffold delete       # 清理资源
```

### 5.4 常用命令

```bash
# 查看服务状态
kubectl get pods
kubectl get services

# 查看日志
kubectl logs -l app=frontend

# 扩缩容
kubectl scale deployment frontend --replicas=3

# 更新某个服务
kubectl rollout restart deployment cartservice
```

---

## 6. 关键配置

### 6.1 环境变量

| 变量名 | 说明 | 默认值 |
|--------|------|--------|
| `CART_SERVICE_ADDR` | 购物车服务地址 | cartservice:7070 |
| `PRODUCT_CATALOG_SERVICE_ADDR` | 商品目录服务地址 | productcatalogservice:3550 |
| `CURRENCY_SERVICE_ADDR` | 货币服务地址 | currencyservice:7000 |
| `EMAIL_SERVICE_ADDR` | 邮件服务地址 | emailservice:8080 |
| `PAYMENT_SERVICE_ADDR` | 支付服务地址 | paymentservice:50051 |
| `SHIPPING_SERVICE_ADDR` | 物流服务地址 | shippingservice:50051 |
| `CYMBAL_BRANDING` | 启用 Cymbal 品牌 | false |

### 6.2 注意事项

- **cartservice 内存**: 默认配置只有 64Mi，需要至少 256Mi
  ```bash
  kubectl patch deployment cartservice -p '{"spec":{"template":{"spec":{"containers":[{"name":"server","resources":{"limits":{"memory":"256Mi"}}}]}}}}'
  ```

- **frontend-external**: LoadBalancer 类型在 kind/minikube 会一直 Pending，使用 `kubectl port-forward` 代替

- **模板更新**: 修改 HTML 模板需要重启 Pod，CSS 修改只需刷新浏览器

---

## 7. 学习资源

- [技术说明文档](microservices-demo/技术说明文档.md) - 从零到精通的完整学习指南
- [交互式启动教程](microservices-demo/交互式启动教程.html) - 可视化学习教程
- [Google Cloud 官方文档](https://cloud.google.com/kubernetes-engine/docs/deploy-apps-overview)
