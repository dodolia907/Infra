# Infra
このリポジトリでは、自宅のインフラのうち公開可能なものを管理します。

## Network
### ハードウェア
- Router: [NEC UNIVERGE IX3110](/network/config/ix3110.txt)
- Firewall: FortiGate 60D
- Server Switch: [Allied Telesis AT-x230-28GT](/network/config/atx230.txt)
- Desktop Switch: [Cisco Catalyst 2960-L](/network/config/c2960l.txt)
- Access Point: NEC Aterm WG2600HS2
### 物理構成
![Phisical design](/network/design/physical.drawio.png)
### 論理構成
![Logical design](/network/design/logical.drawio.png)

## Server
### 仮想基盤 (TYHDC)
- Hypervisor: Proxmox Virtual Environment
- Total Memory: 48GB
- Member
    - tyh01
        - Device: Dell OptiPlex 3020
        - CPU: Intel Core i5-4570 @ 3.20GHz
        - Memory: 16GB
        - Storage: 1.5TB
    - tyh02
        - Device: Dell OptiPlex 3020
        - CPU: Intel Core i5-3470 @ 3.20GHz
        - Memory: 16GB
        - Storage: 1.1TB
    - tyh03
        - Device: NEC Express5800/R110f-1E
        - CPU: Intel Xeon E3-1220 v3 @ 3.10GHz
        - Memory: 16GB
        - Storage: 1.9TB
### コンテナ基盤 (Kubernetes Cluster)
- Master
    - Device: Intel NUC Kit NUC6CAYS
    - OS: Debian 12
    - CPU: Intel Celeron J3455 @ 1.50GHz
    - Memory: 10GB
    - Storage: 32GB + 500GB
- Worker x 3
    - Device: Intel NUC Kit NUC6CAYS
    - OS: Debian 12
    - CPU: Intel Celeron J3455 @ 1.50GHz
    - Memory: 10GB
    - Storage: 32GB

### ハイパフォーマンスコンピューティング (HPC)
- Device: HITACHI HA8000/RS220-h
- OS: Red Hat Enterprise Linux 9
- CPU: Intel Xeon E5-2630 v2 @ 2.60GHz x 2
- Memory: 104GB
- Storage: 2.4TB

### 窓口サーバ
- Device: Apple iMac 14,3
- OS: macOS Sequoia
- CPU: Intel Core i5-4570S @ 2.90GHz
- GPU: NVIDIA GeForce GT 750M
- Memory: 8GB
- Storage: 1.1TB