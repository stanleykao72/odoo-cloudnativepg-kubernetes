# Install odoo and CloudNativePG on Kubernetes

## Requirements

- Kubernetes: I use Rancher Desktop(1.14.0), Kubernetes(1.29.5)
- kubectl: 1.30.1
- helm: 3.15.0

## Infrastructure reference
* [OCA Days 2023: Adrien Peiffer - Odoo DevOps with Kubernetes](https://www.youtube.com/watch?v=1KV3nwq2iVM&t=1161s)
* [Odoo OCA Runboat](https://github.com/sbidoul/runboat/tree/main)
* [Odoo 16 en Kubernetes](https://www.youtube.com/watch?v=_Ab6KQq5qzY) -- [github: GrupoYACCK/odoo-kubernetes](https://github.com/GrupoYACCK/odoo-kubernetes)
* [How to build a scaled odoo v16 - multiple odoo servers](https://www.reddit.com/r/Odoo/comments/17jv9p8/how_to_build_a_scaled_odoo_v16_multiple_odoo/)

## Longhorn (version 1.6.2)
Reference: 
* [Longhorn: Install with Kubectl](https://longhorn.io/docs/1.6.2/deploy/install/install-with-kubectl/)
* [Longhorn — 雲原生儲存系統試玩（上）](https://www.geminiopencloud.com/zh-tw/blog/longhorn1/)
* [Trying Out CloudNative-PG: A Novice in a Kubernetes World](https://www.enterprisedb.com/blog/Trying-Out-CloudNative-PG-Novice-Kubernetes-World)

1. Download Longhorn manifest file
   ```
   wget https://raw.githubusercontent.com/longhorn/longhorn/v1.6.2/deploy/longhorn.yaml -O 01-longhorn.yaml
   ```
2. Change to longhron folder and user kubectl to apply 01-longhorn.yaml
   ```
   kubectl apply -f 01-longhron.yaml
   ```

   for note: 01-longhron.yaml is modified below block to create StorageClass `longhorn-1r` and set numberOfReplicas=1

   ```
    apiVersion: v1
    kind: ConfigMap
    metadata:
    name: longhorn-storageclass
    namespace: longhorn-system
    labels:
        app.kubernetes.io/name: longhorn
        app.kubernetes.io/instance: longhorn
        app.kubernetes.io/version: v1.6.2
    data:
    storageclass.yaml: |
        kind: StorageClass
        apiVersion: storage.k8s.io/v1
        metadata:
        name: longhorn-1r
        annotations:
            storageclass.kubernetes.io/is-default-class: "true"
        provisioner: driver.longhorn.io
        allowVolumeExpansion: true
        reclaimPolicy: "Delete"
        volumeBindingMode: Immediate
        parameters:
            numberOfReplicas: "1"
            staleReplicaTimeout: "30"
            fromBackup: ""
            fsType: "ext4"
            dataLocality: "disabled"
            unmapMarkSnapChainRemoved: "ignored"
   ```
3. To confirm that the deployment succeeded, run:
   ```
   kubectl -n longhorn-system get pod
   ```

4. When your node doesn't install open-iscsi, the pod will get error. [Rancher Desktop with Rancher Longhorn](https://medium.com/@thizaom/rancher-desktop-with-rancher-longhorn-280e687f5022)
   
   4-1 edit .ssh/config at Macos
    ```
        Host rancher-desktop
            HostName 127.0.0.1
            Port 56015 # rg -Fw localPort ~/Library/Application\ Support/rancher-desktop/lima/0/lima.yaml | awk -F': ' '{ print $2 }'
            IdentityFile "~/Library/Application Support/rancher-desktop/lima/_config/user"
            NoHostAuthenticationForLocalhost yes
    ```
   4-2 ssh to Node: rancher-desktop

   ```
        ssh rancher-desktop
   ```
   4-3 In the Node Shell terminal run the following commands:
   ```
        sudo apk add open-iscsi
        sudo rc-update add iscsid
        sudo rc-service iscsid start
   ```
## CloudNativePG
Reference: 
* [CloudNativePG: Run PostgreSQL ® in Kubernetes!](https://github.com/cloudnative-pg)
* [Trying Out CloudNative-PG: A Novice in a Kubernetes World](https://www.enterprisedb.com/blog/Trying-Out-CloudNative-PG-Novice-Kubernetes-World)
* [Grafana Dashboards](https://github.com/cloudnative-pg/grafana-dashboards/tree/main?tab=readme-ov-file)
## Odoo
## Ingress