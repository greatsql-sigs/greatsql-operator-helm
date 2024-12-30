# GreatSQL Operator Helm Chart
This Helm Chart is designed to deploy the GreatSQL Operator. It simplifies the process of deploying and managing MySQL-based databases in Kubernetes clusters, supporting both single-instance and group replication (cluster) modes.

## Prerequisites
Kubernetes 1.22+</br>
Helm 3.0+</br>
A compatible StorageClass for PersistentVolumes (required for persistence)</br>

## Installation
### Add the Helm Repository
Before installing the chart, add the official repository:

```sh
helm repo add greatsql-operator https://greatsql-sigs.github.io/greatsql-operator-helm
helm repo update
```

### Install the Chart
To install the chart, use the following command

```sh
helm install greatsql greatsql-operator/greatsql-operator
```

This will install the chart with default values. You can customize the installation by specifying parameters (see the `Configuration` section below).


## Uninstallation
To uninstall the deployed chart, use the following command

```sh
helm delete greatsql
```
This command removes all resources associated with the chart but retains PersistentVolumeClaims (PVCs) by default to avoid accidental data loss.

## Configuration
### Default Values
You can customize the installation by modifying the default values.yaml file or passing custom values through the --set flag.

#### Key Configuration Options:
| Parameter                  | Description                                                   | Default                                                     |
|----------------------------|---------------------------------------------------------------|-------------------------------------------------------------|
| `replicaCount`             | Number of replicas for single-instance deployments            | `1`                                                           |
| `type`                     | Deployment type: `SingleInstance` or `GroupReplicationCluster`    | `SingleInstance`                                              |
| `image.repository`         | Image repository                                              | `registry.cn-chengdu.aliyuncs.com/greatsql/greatsql-operator` |
| `image.tag`                | Image tag                                                     | `latest`                                                      |
| `service.type`             | Kubernetes service type                                       | `ClusterIP`                                                   |
| `service.port`             | MySQL service port                                            | `3306`                                                        |
| `storage.enabled`          | Enable persistent storage                                     | `true`                                                        |
| `storage.size`             | Requested storage size for PersistentVolumeClaim              | `10Gi`                                                        |
| `configFile.enabled`       | Enable custom configuration file mounting                     | `false`                                                       |
| `configFile.configMapName` | Name of the ConfigMap for configuration                       | `greatsql-config`                                            |

## Example Configuration
### 1. Deploy a SingleInstance
```sh
helm install greatsql greatsql-operator/greatsql-operator \
  --set type=SingleInstance \
  --set replicaCount=1 \
  --set storage.size=20Gi
```

### 2. Deploy a GroupReplicationCluster
```sh
helm install greatsql-cluster greatsql-operator/greatsql-operator \
  --set type=GroupReplicationCluster \
  --set cluster.replicas=3 \
  --set storage.size=50Gi \
  --set configFile.enabled=true \
  --set configFile.files.my.cnf="[mysqld]\nserver-id=1\nlog-bin=mysql-bin\nbinlog-format=ROW"
```

### 3. Enable Custom ConfigMap
If you want to mount a custom my.cnf configuration, create a ConfigMap:
```sh
kubectl create configmap greatsql-config --from-literal=my.cnf="[mysqld]\nmax_connections=200"
```

Then install the chart with:
```sh
helm install greatsql greatsql-operator/greatsql-operator \
  --set configFile.enabled=true \
  --set configFile.configMapName=greatsql-config
```
## Persistence
By default, persistent storage is enabled. To disable it, set storage.enabled to false:
```sh
helm install greatsql greatsql-operator/greatsql-operator \
  --set storage.enabled=false
```

When enabled, PersistentVolumeClaims are created to store MySQL data, ensuring data durability even if the Pod is deleted or restarted.

### Upgrading
To upgrade the chart to a newer version or update values, use the following command:
```sh
helm upgrade greatsql greatsql-operator/greatsql-operator --set <parameters>
```

## Monitoring
GreatSQL exposes basic liveness and readiness probes to ensure proper functioning. You can also integrate monitoring tools such as Prometheus and Grafana for advanced metrics.

## Support
For support, please visit the [GreatSQL SIGs GitHub Repository or raise an issue in the repository](https://github.com/greatsql-sigs/greatsql-operator-helm).

By using this chart, you can easily deploy and manage MySQL instances in Kubernetes clusters, whether for development, testing, or production environments.