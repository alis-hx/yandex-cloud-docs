## Решения для известных проблем {{ managed-k8s-name }} {#known-issues-resolving}

* [Хосты группы узлов {{ managed-k8s-name }} долго находятся в статусе `RECONCILING`](node-group-hosts-are-in-reconciling-status-for-a-long-time.md)
* [Устранение ошибки `Timed out waiting for the condition` при монтировании тома с большим количеством файлов](unable-to-mount-volume-with-a-large-number-of-files.md)
* [Устранение ошибки `Timed out waiting for the condition` при монтировании PVC](error-unable-to-attach-or-mount-volumes-timed-out-waiting-for-the-condition.md)
* [Устранение ошибки `Multi-Attach error for volume`](multi-attach-error-for-volume.md)
* [Под `kube-dns-autoscaler` постоянно перезапускается](pod-kube-dns-autoscaler-constantly-restarting.md)
* [Не уменьшается количество узлов в группе {{ managed-k8s-name }}](cannot-reduce-the-number-of-nodes.md)
* [Устранение ошибки `DEADLINE_EXCEEDED`](error-504.md)
* [Не увеличивается количество узлов в группе {{ managed-k8s-name }}](autoscaler-node-enlarging-issues.md)
* [Устранение ошибки `Can't use allocation_policy.locations.subnet_id together with node_template.network_interface_specs`](cant-use-allocation-policy-locations-subnet-id-together-with-node-template-network-interface-specs.md)
* [Кластер слишком долго находится в состоянии `STARTING`](cluster-stuck-in-starting-state.md)
* [Устранение проблем с разрешением DNS-имен в {{ managed-k8s-name }}](dns-name-resolving-issues.md)
* [Устранение проблем с работой HPA в {{ managed-k8s-name }}](hpa-and-metrics-server-issues.md)
* [Устранение ошибок с синхронизацией времени на узлах кластера {{ managed-k8s-name }}](ntp-time-sync-issues.md)
* [Поды кластера слишком долго находятся в состоянии `PENDING`](pods-stuck-in-pending-state.md)
* [Устранение проблем с монтированием томов к подам кластера {{ managed-k8s-name }} средствами Container Storage Interface для S3](unable-to-mount-volume-to-pods-via-csi-s3.md)
* [Устранение ошибки `0/10 nodes are available - node(s) had untolerated taint`](untolerated-taints-issue.md)
* [Кластер {{ managed-k8s-name }} слишком долго находится в состоянии `STARTING` после переименования](cluster-stuck-in-starting-after-renaming.md)