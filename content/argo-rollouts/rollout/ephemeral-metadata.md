---
weight: 5
title: "短暂元数据"
date: '2023-06-21T16:00:00+08:00'
type: book
---

🔔 重要提示：自 v0.10.0 起可用于金丝雀发布。

🔔 重要提示：自 v1.0 起可用于蓝绿发布。

一个用例是，在 Rollout 标记或注释所需/稳定的 Pod 时，用用户定义的标签/注释，*仅*在它们是所需或稳定的集合期间，并且在 ReplicaSet 切换角色（例如从所需到稳定）时更新/删除标签。此举使 prometheus、wavefront、datadog 等查询和仪表板可以依赖一致的标签，而不是不可预测且从版本到版本不断变化的 `rollouts-pod-template-hash`。

使用金丝雀策略的 Rollout 有能力使用 `stableMetadata` 和 `canaryMetadata` 字段将短暂元数据附加到稳定或金丝雀 Pod 上。

```yaml
spec:
  strategy:
    canary:
      stableMetadata:
        labels:
          role: stable
      canaryMetadata:
        labels:
          role: canary
```

使用蓝绿策略的 Rollout 有能力使用 `activeMetadata` 和 `previewMetadata` 字段将短暂元数据附加到活动或预览 Pod 上。

```yaml
spec:
  strategy:
    blueGreen:
      activeMetadata:
        labels:
          role: active
      previewMetadata:
        labels:
          role: preview
```

在更新期间，Rollout 将创建所需的 ReplicaSet，并将 `canaryMetadata`/`previewMetadata` 中定义的元数据合并到所需的 ReplicaSet 的 `spec.template.metadata` 中。这将导致 ReplicaSet 的所有 Pod 都使用所需的元数据创建。当 Rollout 完全升级时，所需的 ReplicaSet 变为稳定，更新为使用 `stableMetadata`/`activeMetadata` 下的标签和注释。ReplicaSet 的 Pod 然后将*原地*更新以使用稳定的元数据（而无需重新创建 Pod）。

🔔 重要提示：为了让工具能够利用这个功能，他们需要识别标签和/或注释在 Pod 启动后发生的变化。并非所有工具都能检测到这一点。
