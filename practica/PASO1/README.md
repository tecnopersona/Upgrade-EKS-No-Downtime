# 🚀 Script de Despliegue del Entorno de Práctica

## Tabla de Contenidos

| | |
|:-:|---|
| 🚀 | [Creación del Cluster EKS Problemático](#creación-del-cluster-eks-problemático) |
| 🎯 | [Despliegue de Recursos Problemáticos](#despliegue-de-recursos-problemáticos) |
| 🔧 | [Configuración de Problemas de Infraestructura](#configuración-de-problemas-de-infraestructura) |

---

## Creación del Cluster EKS Problemático

Este script crea un cluster EKS 1.28 con configuraciones intencionalmente problemáticas para prácticas de troubleshooting y actualización.

```bash
#!/bin/bash
set -e

echo "🚀 CREANDO ENTORNO DE PRÁCTICA COMPLEJO EKS 1.28..."

# Configuración
CLUSTER_NAME="practice-cluster-$(date +%Y%m%d)"
REGION="us-east-1"
VERSION="1.28"

# 1. Crear cluster EKS 1.28 con problemas intencionales
echo "1. Creando cluster EKS 1.28 con configuraciones problemáticas..."

cat << EOF > cluster-config.yaml
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig

metadata:
  name: ${CLUSTER_NAME}
  region: ${REGION}
  version: "${VERSION}"

iam:
  withOIDC: true

addons:
- name: vpc-cni
- name: coredns
- name: kube-proxy

nodeGroups:
  - name: ng-al2-problematic
    instanceType: t3.medium
    desiredCapacity: 3
    minSize: 1
    maxSize: 5
    volumeSize: 20
    labels:
      os: amazon-linux-2
    tags:
      k8s.io/cluster-autosaster: owned
    # Configuración problemática intencional
    kubeletExtraConfig:
      keepTerminatedPodVolumes: true
    iam:
      withAddonPolicies:
        autoScaler: true
        ebs: true

  - name: ng-mixed-os
    instanceType: t3.small
    desiredCapacity: 2
    minSize: 1
    maxSize: 3
    volumeSize: 20
    labels:
      os: mixed-problems
    # Sin políticas de IAM para autoscaler
EOF

eksctl create cluster -f cluster-config.yaml

echo "✅ Cluster creado. Configurando problemas intencionales..."
```

---

## Despliegue de Recursos Problemáticos

Despliega recursos con APIs deprecadas, configuraciones inseguras y otros problemas comunes para simular un entorno realista de migración y troubleshooting.

```bash
#!/bin/bash
echo "🛠️ DESPLEGANDO RECURSOS CON PROBLEMAS COMUNES..."

# 2. Aplicar recursos con APIs deprecadas
echo "2.1 Desplegando FlowControl con APIs beta..."
cat << EOF | kubectl apply -f -
apiVersion: flowcontrol.apiserver.k8s.io/v1beta2
kind: FlowSchema
metadata:
  name: problematic-flowschema
spec:
  matchingPrecedence: 10000
  priorityLevelConfiguration:
    name: problematic-priority
  rules:
  - resourceRules:
    - apiGroups: ['*']
      clusterScope: true
      namespaces: ['*']
      resources: ['*']
      verbs: ['*']
    subjects:
    - kind: User
      user:
        name: test-user
---
apiVersion: flowcontrol.apiserver.k8s.io/v1beta2
kind: PriorityLevelConfiguration
metadata:
  name: problematic-priority
spec:
  type: Limited
  limited:
    assuredConcurrencyShares: 1
    limitResponse:
      type: Reject
EOF

# 2.2 Desplegar Ingress con API antigua
echo "2.2 Desplegando Ingress con API deprecada..."
cat << EOF | kubectl apply -f -
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: deprecated-ingress
  namespace: default
spec:
  backend:
    serviceName: test-service
    servicePort: 80
EOF

# 2.3 Desplegar recursos con AppArmor annotations
echo "2.3 Desplegando Pods con AppArmor annotations..."
cat << EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: apparmor-pod
  annotations:
    container.apparmor.security.beta.kubernetes.io/main: localhost/restricted
spec:
  containers:
  - name: main
    image: nginx:1.19
    ports:
    - containerPort: 80
EOF

# 2.4 Crear StorageClasses problemáticas
echo "2.4 Configurando StorageClasses problemáticas..."
cat << EOF | kubectl apply -f -
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: problematic-sc
  annotations:
    storageclass.kubernetes.io/is-default-class: "true"
provisioner: ebs.csi.aws.com
parameters:
  type: gp2
EOF

# 2.5 Desplegar aplicaciones con problemas de readiness
echo "2.5 Desplegando aplicaciones con probes problemáticos..."
cat << EOF | kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: problematic-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: problematic
  template:
    metadata:
      labels:
        app: problematic
    spec:
      containers:
      - name: main
        image: nginx:1.18  # Versión antigua
        ports:
        - containerPort: 80
        readinessProbe:
          httpGet:
            path: /health
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5
        livenessProbe:
          httpGet:
            path: /health  
            port: 80
          initialDelaySeconds: 15
          periodSeconds: 20
---
apiVersion: v1
kind: Service
metadata:
  name: problematic-service
spec:
  selector:
    app: problematic
  ports:
  - port: 80
    targetPort: 80
EOF

# 2.6 Desplegar HPA con métricas deprecated
echo "2.6 Desplegando HPA con métricas beta..."
cat << EOF | kubectl apply -f -
apiVersion: autoscaling/v2beta2
kind: HorizontalPodAutoscaler
metadata:
  name: deprecated-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: problematic-app
  minReplicas: 1
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
EOF

# 2.7 Desplegar NetworkPolicy con API antigua
echo "2.7 Desplegando NetworkPolicy antigua..."
cat << EOF | kubectl apply -f -
apiVersion: extensions/v1beta1
kind: NetworkPolicy
metadata:
  name: old-network-policy
spec:
  podSelector:
    matchLabels:
      app: problematic
  ingress:
  - from:
    - podSelector:
        matchLabels:
          role: frontend
EOF

# 2.8 Crear ServiceAccounts problemáticos
echo "2.8 Creando ServiceAccounts con annotations deprecated..."
cat << EOF | kubectl apply -f -
apiVersion: v1
kind: ServiceAccount
metadata:
  name: problematic-sa
  annotations:
    kubernetes.io/enforce-mountable-secrets: "true"
    eks.amazonaws.com/role-arn: arn:aws:iam::123456789012:role/NonExistentRole
EOF

echo "✅ Todos los recursos problemáticos desplegados!"
```

---

## Configuración de Problemas de Infraestructura

Simula problemas de IAM, versiones incompatibles de autoscaler y drivers, y otros errores comunes de infraestructura.

```bash
#!/bin/bash
echo "🏗️ CONFIGURANDO PROBLEMAS DE INFRAESTRUCTURA..."

# 3. Configurar problemas de IAM
echo "3.1 Verificando problemas de IAM..."
# Esto simulará problemas de permisos

# 3.2 Configurar Autoscaler con versión incompatible
echo "3.2 Desplegando Cluster Autoscaler versión incompatible..."
cat << EOF | kubectl apply -f -
apiVersion: v1
kind: ServiceAccount
metadata:
  labels:
    k8s-addon: cluster-autoscaler.addons.k8s.io
    k8s-app: cluster-autoscaler
  name: cluster-autoscaler
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cluster-autoscaler
  labels:
    k8s-addon: cluster-autoscaler.addons.k8s.io
    k8s-app: cluster-autoscaler
rules:
  - apiGroups: [""]
    resources: ["events", "endpoints"]
    verbs: ["create", "patch"]
  - apiGroups: [""]
    resources: ["pods/eviction"]
    verbs: ["create"]
  - apiGroups: [""]
    resources: ["pods/status"]
    verbs: ["update"]
  - apiGroups: [""]
    resources: ["endpoints"]
    resourceNames: ["cluster-autoscaler"]
    verbs: ["get", "update"]
  - apiGroups: [""]
    resources: ["nodes"]
    verbs: ["watch", "list", "get", "update"]
  - apiGroups: [""]
    resources: ["pods", "services", "replicationcontrollers", "persistentvolumeclaims", "persistentvolumes"]
    verbs: ["watch", "list", "get"]
  - apiGroups: ["extensions"]
    resources: ["replicasets", "daemonsets"]
    verbs: ["watch", "list", "get"]
  - apiGroups: ["policy"]
    resources: ["poddisruptionbudgets"]
    verbs: ["watch", "list"]
  - apiGroups: ["apps"]
    resources: ["statefulsets", "replicasets", "daemonsets"]
    verbs: ["watch", "list", "get"]
  - apiGroups: ["storage.k8s.io"]
    resources: ["storageclasses", "csinodes", "csidrivers", "csistoragecapacities"]
    verbs: ["watch", "list", "get"]
  - apiGroups: ["batch", "extensions"]
    resources: ["jobs"]
    verbs: ["get", "list", "watch", "patch"]
  - apiGroups: ["coordination.k8s.io"]
    resources: ["leases"]
    verbs: ["create"]
  - apiGroups: ["coordination.k8s.io"]
    resourceNames: ["cluster-autoscaler"]
    resources: ["leases"]
    verbs: ["get", "update"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: cluster-autoscaler
  namespace: kube-system
  labels:
    k8s-addon: cluster-autoscaler.addons.k8s.io
    k8s-app: cluster-autoscaler
rules:
  - apiGroups: [""]
    resources: ["configmaps"]
    verbs: ["create","list","watch"]
  - apiGroups: [""]
    resources: ["configmaps"]
    resourceNames: ["cluster-autoscaler-status", "cluster-autoscaler-priority-expander"]
    verbs: ["delete", "get", "update", "watch"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: cluster-autoscaler
  labels:
    k8s-addon: cluster-autoscaler.addons.k8s.io
    k8s-app: cluster-autoscaler
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-autoscaler
subjects:
  - kind: ServiceAccount
    name: cluster-autoscaler
    namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: cluster-autoscaler
  namespace: kube-system
  labels:
    k8s-addon: cluster-autoscaler.addons.k8s.io
    k8s-app: cluster-autoscaler
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: cluster-autoscaler
subjects:
  - kind: ServiceAccount
    name: cluster-autoscaler
    namespace: kube-system
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: cluster-autoscaler
  namespace: kube-system
  labels:
    app: cluster-autoscaler
spec:
  replicas: 1
  selector:
    matchLabels:
      app: cluster-autoscaler
  template:
    metadata:
      labels:
        app: cluster-autoscaler
      annotations:
        prometheus.io/scrape: 'true'
        prometheus.io/port: '8085'
    spec:
      serviceAccountName: cluster-autoscaler
      containers:
        - image: k8s.gcr.io/autoscaling/cluster-autoscaler:v1.22.0  # Versión incompatible
          name: cluster-autoscaler
          resources:
            limits:
              cpu: 100m
              memory: 300Mi
            requests:
              cpu: 100m
              memory: 300Mi
          command:
            - ./cluster-autoscaler
            - --v=4
            - --stderrthreshold=info
            - --cloud-provider=aws
            - --skip-nodes-with-local-storage=false
            - --expander=least-waste
            - --node-group-auto-discovery=asg:tag=k8s.io/cluster-autoscaler/enabled,k8s.io/cluster-autoscaler/${CLUSTER_NAME}
          env:
            - name: AWS_REGION
              value: ${REGION}
          volumeMounts:
            - name: ssl-certs
              mountPath: /etc/ssl/certs/ca-certificates.crt
              readOnly: true
          imagePullPolicy: "Always"
      volumes:
        - name: ssl-certs
          hostPath:
            path: "/etc/ssl/certs/ca-bundle.crt"
EOF

# 3.3 Configurar EBS CSI Driver con versión problemática
echo "3.3 Desplegando EBS CSI Driver versión antigua..."
kubectl apply -k "github.com/kubernetes-sigs/aws-ebs-csi-driver/deploy/kubernetes/overlays/stable/?ref=release-1.8"

# 3.4 Crear PVCs problemáticos
echo "3.4 Creando PVCs sin StorageClass..."
cat << EOF | kubectl apply -f -
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-no-storageclass
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
EOF

echo "✅ Problemas de infraestructura configurados!"
```

---

> **Nota:** Este entorno está diseñado para prácticas de troubleshooting, migración y actualización de EKS. Todos los problemas son intencionales para fines educativos.