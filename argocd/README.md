# Minecraft ArgoCD GitOps

这套清单使用 App of Apps，把 `minecraft-backend` 和 `minecraft-operator` 交给 ArgoCD 管理。

## 目录

- `bootstrap/root-app.yaml`: 引导用 Root Application
- `apps/`: ArgoCD `AppProject` 和两个子 `Application`
- `workloads/backend/`: backend 的 Kustomize 清单，包含 Istio mTLS Gateway
- `workloads/minecraft-operator/`: operator overlay，复用仓库内现有 `config/default`

## 引导

```bash
kubectl apply -n argocd -f gitops/argocd/bootstrap/root-app.yaml
```

## Backend mTLS

backend 默认暴露在 `minecraft-api.charlie-cloud.me`，Istio `Gateway` 已配置为 `MUTUAL`。

在流量真正可用前，你还需要先在 `mc-infra` 创建名为 `minecraft-backend-mtls-cert` 的 secret，至少包含：

- `tls.crt`
- `tls.key`
- `ca.crt`

示例模板在 `workloads/backend/samples/istio-mtls-secret.example.yaml`，默认没有被 Kustomize 引入，避免把示例证书误同步到集群。

## 说明

- backend 运行在 `mc-infra`
- backend 通过 RoleBinding 访问 `minecraft` 命名空间中的 `Server` CR、Pod、Service 和日志
- operator 运行在 `minecraft-operator-system`
- operator overlay 额外补上了它实际需要的 `ConfigMap`、`Service`、`StatefulSet`、`PVC` 权限
