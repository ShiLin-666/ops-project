# ops-project · K8s 云原生运维平台

三节点 Kubernetes 运维实战项目：从零搭建集群，到应用部署、数据备份、监控告警全链路落地。

## 架构总览

| 模块 | 技术选型 | 说明 |
|---|---|---|
| 集群 | kubeadm 1.28 + containerd | 三节点集群（1 master + 2 node，阿里云 ECS） |
| 网络 | Calico | Pod 网络与网络策略 |
| 应用 | Nginx / myweb（自制镜像）/ MySQL | 业务应用全生命周期 |
| 存储 | local-path-provisioner | 动态 PV 供给 |
| 数据备份 | Kubernetes CronJob | MySQL 每 5 分钟定时备份，保留 7 份，恢复演练验证 |
| 监控告警 | Prometheus + Grafana + Alertmanager | 指标采集、可视化看板、邮件告警闭环 |
| 自动化 | Ansible（3 剧本 + site.yaml） | 节点初始化、组件安装标准化，支持新节点一键接入 |

## 目录结构

```
ops-project/
├── ansible/            # 自动化剧本：init-nodes / install-containerd / install-kubeadm + site.yaml
├── apps/               # 应用部署清单
│   ├── nginx/          # 静态应用 + NodePort 暴露
│   ├── web/            # 自制 myweb 镜像（Dockerfile + 页面）
│   ├── mysql/          # Deployment + PVC + 备份 CronJob + 恢复 Job
│   └── monitoring/     # Prometheus + Grafana + Alertmanager + Node Exporter
├── k8s/                # Calico、存储组件等集群级清单
├── docs/               # 部署手册
└── scripts/            # 运维脚本
```

## 快速部署

1. 节点初始化（新节点一键标准化）

```bash
ansible-playbook -i ansible/inventory.ini ansible/site.yaml
```

2. 部署业务应用

```bash
kubectl apply -f apps/nginx/
kubectl apply -f apps/mysql/
kubectl apply -f apps/web/
```

3. 部署监控告警栈

```bash
kubectl apply -f apps/monitoring/
```

## 安全实践

- 真实凭据不入库：`secret.yaml`、`alertmanager-config.yaml` 由 `.gitignore` 排除，仓库仅保留 `*.example` 模板，部署时在服务器上填入真实值
- 数据库密码通过 Secret 环境变量注入，不出现在命令行（`mysqldump` 读取 `MYSQL_PWD`）
- SSH 密钥与服务器密钥用途隔离

## 项目亮点

- **备份恢复闭环**：定时备份 + 保留策略 + 恢复演练（8 秒内完成恢复验证），保证数据可回滚
- **告警闭环**：Node/Pod 异常 → Prometheus 采集 → Alertmanager 路由 → 邮件通知，全链路可验证
- **标准化交付**：Ansible 剧本实现节点交付标准化，新节点接入无需手工步骤
- **镜像自制**：myweb 应用从 Dockerfile 到部署全流程实践
