---
name: add-k8s-service
description: 往 Kubernetes 集群添加普通服务。当用户请求部署新应用、添加新服务到集群时使用此技能。
---

# 添加 Kubernetes 服务

## 概述

在 home-ops 集群中部署新的普通服务，使用 GitOps (Flux) 模式管理。

## 目录结构

新服务应创建在 `k8s/apps/common/<app-name>/` 下：

```
k8s/apps/common/<app-name>/
├── ks.yaml                    # Flux Kustomization (入口)
└── app/
    ├── kustomization.yaml     # Kustomize 配置
    ├── helmrelease.yaml       # HelmRelease (使用 app-template，路由也在此定义)
    └── externalsecret.yaml    # 可选：外部密钥
```

## 步骤

### 1. 创建 ks.yaml (Flux Kustomization)

```yaml
---
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: &app <app-name>
  namespace: &namespace default
spec:
  targetNamespace: *namespace
  dependsOn:
    - name: cilium
      namespace: kube-system
  interval: 1h
  path: ./k8s/apps/common/<app-name>/app
  prune: true
  retryInterval: 2m
  sourceRef:
    kind: GitRepository
    name: flux-system
    namespace: flux-system
  timeout: 5m
  wait: false
```

### 2. 创建 app/helmrelease.yaml

使用 `app-template` chart：

```yaml
---
apiVersion: helm.toolkit.fluxcd.io/v2
kind: HelmRelease
metadata:
  name: &app <app-name>
spec:
  interval: 1h
  chartRef:
    kind: OCIRepository
    name: app-template
    namespace: flux-system
  values:
    controllers:
      <app-name>:
        annotations:
          reloader.stakater.com/auto: "true"
        containers:
          app:
            image:
              repository: <image-repo>
              tag: <image-tag>
            env:
              TZ: ${TIMEZONE}
            probes:
              liveness: &probes
                enabled: true
                custom: true
                spec:
                  httpGet:
                    path: /health
                    port: &port 80
                  initialDelaySeconds: 0
                  periodSeconds: 10
                  timeoutSeconds: 1
                  failureThreshold: 10
              readiness: *probes
            securityContext:
              allowPrivilegeEscalation: false
              readOnlyRootFilesystem: true
              capabilities: { drop: ["ALL"] }
            resources:
              requests:
                cpu: 10m
                memory: 16Mi
              limits:
                memory: 100Mi
    defaultPodOptions:
      securityContext:
        runAsNonRoot: true
        runAsUser: 1000
        runAsGroup: 1000
        fsGroup: 1000
        fsGroupChangePolicy: OnRootMismatch
    service:
      app:
        ports:
          http:
            port: *port
    route:
      app:
        annotations:
          gethomepage.dev/enabled: "true"
          gethomepage.dev/group: external
          gethomepage.dev/name: <app-name>
          gethomepage.dev/icon: <icon>.svg
        hostnames: ["{{ .Release.Name }}.${MAIN_DOMAIN}"]
        parentRefs:
          - name: envoy-external
            namespace: network
        rules:
          - backendRefs:
              - identifier: app
                port: *port
```

### 3. 创建 app/kustomization.yaml

```yaml
---
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - ./helmrelease.yaml
  # - ./externalsecret.yaml  # 如果需要密钥
```

### 4. (可选) 创建 app/externalsecret.yaml

如果服务需要密钥：

```yaml
---
# yaml-language-server: $schema=https://kubernetes-schemas.pages.dev/external-secrets.io/externalsecret_v1.json
apiVersion: external-secrets.io/v1
kind: ExternalSecret
metadata:
  name: &secret <app-name>-secret
spec:
  secretStoreRef:
    kind: ClusterSecretStore
    name: azure-store
  target:
    name: *secret
    template:
      data:
        KEY_NAME: "{{ .KEY_NAME }}"
  dataFrom:
    - extract:
        key: <azure-keyvault-secret-name>
```

### 5. 注册服务到集群

在 `k8s/clusters/staging/apps.yaml` 或相关 kustomization 中引入新服务。

## 原则

- **Chart**: 除非特殊情况，始终使用 `app-template` 作为 Helm chart
- **密钥**: 使用 ExternalSecret 从 Azure KeyVault 同步，不要硬编码
- **组件**: 如需复用组件，从 `k8s/components/` 引入
- **安全**: 遵循最小权限原则，设置 securityContext
- **资源**: 始终设置 resources requests/limits

## 验证

1. 检查文件语法: `kubectl kustomize k8s/apps/common/<app-name>/app`
2. 提交后等待 Flux 自动同步
3. 检查状态: `flux get kustomization <app-name>`
4. 检查 Pod: `kubectl get pods -l app.kubernetes.io/name=<app-name>`

## 参考模板

- 简单服务: `k8s/apps/common/echo/`
- 带密钥服务: `k8s/apps/common/vaultwarden/`
- 带数据库服务: `k8s/apps/common/affine/`
