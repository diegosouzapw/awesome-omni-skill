---
name: check-ceph-health
description: Check Ceph storage health on OpenShift OCS/ODF clusters. Use when PVCs are stuck in Pending, storage provisioning fails, Ceph is degraded, OSDs are full, or cluster storage needs diagnosis.
---

# Check Ceph Health

Use this guide to diagnose and remediate Ceph storage issues on OpenShift clusters running OCS/ODF (OpenShift Data Foundation).

## 1. Ceph Cluster Health

```bash
# Quick health status
kubectl -n openshift-storage get cephcluster -o jsonpath='{.items[*].status.ceph.health}'

# Detailed health with error messages
kubectl -n openshift-storage get cephcluster -o jsonpath='{.items[*].status.ceph.details}' | python3 -m json.tool

# Capacity overview (bytesAvailable, bytesUsed, bytesTotal)
kubectl -n openshift-storage get cephcluster -o jsonpath='{.items[*].status.ceph.capacity}' | python3 -m json.tool
```

Health states:
- `HEALTH_OK` -- cluster is healthy
- `HEALTH_WARN` -- degraded but functional (backfillfull, nearfull, degraded PGs)
- `HEALTH_ERR` -- critical, writes may be blocked (full OSDs, too few OSDs, down PGs)

## 2. Running Ceph Commands

OCS/ODF clusters may not have a rook-ceph-tools pod deployed. Use a mon pod to run ceph commands directly.

```bash
# Find the mon pod and its service address
MON_POD=$(kubectl -n openshift-storage get pods -l app=rook-ceph-mon -o jsonpath='{.items[0].metadata.name}')
MON_ADDR=$(kubectl -n openshift-storage get pod $MON_POD -o jsonpath='{.spec.containers[0].env[?(@.name=="ROOK_CEPH_MON_HOST")].value}' | sed 's/\[//;s/\]//')

# Run any ceph command via the mon pod
kubectl -n openshift-storage exec $MON_POD -c mon -- \
  ceph -m $MON_ADDR --keyring /etc/ceph/keyring-store/keyring status
```

Useful ceph commands to run this way:
- `status` -- overall cluster status
- `osd df` -- per-OSD disk usage
- `osd pool ls detail` -- pool details
- `df` -- pool-level capacity
- `health detail` -- verbose health messages

## 3. OSD Status

```bash
# OSD pods
kubectl -n openshift-storage get pods -l app=rook-ceph-osd

# OSD prepare jobs (should be Completed, not stuck)
kubectl -n openshift-storage get pods | grep osd-prepare

# Storage device sets (backing PVCs for OSDs)
kubectl -n openshift-storage get pvc -l app=rook-ceph-osd
```

## 4. CSI Provisioner Pods

PVC provisioning is handled by CSI driver pods. If these are unhealthy, no volumes can be created.

```bash
# RBD CSI controller (provisions rbd volumes)
kubectl -n openshift-storage get pods | grep rbd.*ctrlplugin

# CephFS CSI controller (provisions cephfs volumes)
kubectl -n openshift-storage get pods | grep cephfs.*ctrlplugin

# RBD node plugins (mount volumes on nodes)
kubectl -n openshift-storage get pods | grep rbd.*nodeplugin

# Check for CSI provisioner errors in logs
kubectl -n openshift-storage logs <rbd-ctrlplugin-pod> -c csi-rbdplugin --tail=50
```

## 5. PVC and PV Diagnosis

```bash
# Find stuck PVCs
kubectl get pvc --all-namespaces --field-selector status.phase=Pending

# Describe a pending PVC to see provisioning errors
kubectl describe pvc <pvc-name> -n <namespace>

# Find Released PVs (consume space but no longer bound to a PVC)
kubectl get pv --field-selector status.phase=Released

# Check StorageClasses
kubectl get storageclass
```

## 6. Common Problems and Remediation

### OSDs Full (HEALTH_ERR: full osd(s))

**Symptoms**: PVCs stuck in Pending, provisioning errors with `DeadlineExceeded` or `operation already exists`.

**Diagnosis**:
```bash
kubectl -n openshift-storage get cephcluster -o jsonpath='{.items[*].status.ceph.details}' | python3 -m json.tool
```
Look for `OSD_FULL` and `POOL_FULL` messages.

**Remediation**:

1. **Delete Released PVs** to reclaim space from orphaned volumes:
   ```bash
   kubectl get pv --field-selector status.phase=Released
   kubectl delete pv <released-pv-names>
   ```

2. **Temporarily raise the full ratio** if Ceph is blocking all writes (including deletes):
   ```bash
   # Raise to 0.92 to unblock writes temporarily
   kubectl -n openshift-storage exec $MON_POD -c mon -- \
     ceph -m $MON_ADDR --keyring /etc/ceph/keyring-store/keyring \
     osd set-full-ratio 0.92
   ```
   Once space is freed and health improves, **reset to default**:
   ```bash
   kubectl -n openshift-storage exec $MON_POD -c mon -- \
     ceph -m $MON_ADDR --keyring /etc/ceph/keyring-store/keyring \
     osd set-full-ratio 0.85
   ```

3. **Add more storage** by expanding OSD count or disk size if cleanup is insufficient.

### OSDs Nearfull / Backfillfull (HEALTH_WARN)

**Symptoms**: Cluster functional but approaching full. Warnings about `nearfull` or `backfillfull` OSDs.

**Remediation**:
- Clean up unused PVCs and Released PVs
- Delete completed migration data no longer needed
- Plan capacity expansion before reaching full threshold (85%)

### Degraded PGs

**Symptoms**: `HEALTH_WARN` with messages about degraded or undersized placement groups.

**Diagnosis**:
```bash
# Via mon pod:
ceph health detail
ceph pg stat
```

**Remediation**:
- If an OSD is down, check the OSD pod and its node
- If a node is down, Ceph will self-heal once the node returns
- If an OSD is permanently lost, Ceph will rebalance automatically (may take time)

### CSI Provisioner Not Responding

**Symptoms**: PVC events say "waiting for external provisioner" but no `ProvisioningFailed` errors.

**Diagnosis**:
```bash
kubectl -n openshift-storage get pods | grep ctrlplugin
kubectl -n openshift-storage logs <rbd-ctrlplugin-pod> -c csi-rbdplugin --tail=100
```

**Remediation**:
- Restart the CSI controller pod if it's stuck
- Check if the Ceph cluster is reachable from the CSI pod
- Verify the StorageClass references a valid pool and secret

### Pools Full but OSDs Not Full

**Symptoms**: `POOL_FULL` warning but individual OSDs have space.

**Diagnosis**:
```bash
# Via mon pod:
ceph osd pool ls detail
ceph df detail
```

**Remediation**:
- A pool may have a quota set -- check and raise it
- Rebalance may be needed if data is unevenly distributed

## 7. Operator Health

```bash
# OCS/ODF operator pods
kubectl -n openshift-storage get pods | grep -E 'ocs-operator|odf-operator|rook-ceph-operator'

# Rook operator logs (manages Ceph cluster lifecycle)
kubectl -n openshift-storage logs deployment/rook-ceph-operator --tail=50

# Check for CrashLoopBackOff or restarts
kubectl -n openshift-storage get pods -o custom-columns=\
'NAME:.metadata.name,STATUS:.status.phase,RESTARTS:.status.containerStatuses[0].restartCount' \
  | sort -t' ' -k3 -rn | head -10
```

## 8. Preventive Checks

Run these periodically to avoid surprise outages:

```bash
# Capacity usage percentage
kubectl -n openshift-storage get cephcluster -o jsonpath='{.items[*].status.ceph.capacity}' | \
  python3 -c "import json,sys; d=json.load(sys.stdin); pct=d['bytesUsed']/d['bytesTotal']*100; print(f'Used: {pct:.1f}%  ({d[\"bytesUsed\"]//2**30} GiB / {d[\"bytesTotal\"]//2**30} GiB)')"

# Released PVs consuming space
kubectl get pv --field-selector status.phase=Released --no-headers | wc -l

# PVCs stuck in Pending
kubectl get pvc --all-namespaces --field-selector status.phase=Pending --no-headers | wc -l
```

Act when usage exceeds 70% -- start cleaning up or expanding capacity before hitting the 85% full threshold.
