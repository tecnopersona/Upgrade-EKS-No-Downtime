# 🚀 Guía de Actualización de EKS sin Downtime

## 📋 Tabla de Contenidos
- [Descripción](#descripción)
- [Prerrequisitos](#prerrequisitos)
- [Preparación](#preparación)
- [Proceso de Actualización](#proceso-de-actualización)
- [Post-Actualización](#post-actualización)
- [Comandos de Emergencia](#comandos-de-emergencia)
- [Checklist](#checklist-final)
- [Recursos Adicionales](#recursos-adicionales)

---

## 📖 Descripción
Guía completa para actualizar un cluster de Amazon EKS de la versión 1.28 a la última versión disponible, manteniendo **zero downtime** y garantizando la estabilidad del entorno de producción.

⏰ **Frecuencia Recomendada:** Cada 3 meses (Kubernetes solo soporta las últimas 3 versiones)

---

## ✅ Prerrequisitos

### 🔧 Requisitos Técnicos
- Cluster EKS existente en versión 1.28
- AWS CLI configurado con permisos adecuados
- kubectl instalado y configurado
- Velero instalado para backups
- Acceso a la consola de AWS

### 📊 Verificaciones Iniciales
```bash
# Verificar estado actual del cluster
kubectl get nodes
kubectl version --short

# Verificar recursos del sistema
kubectl top nodes
kubectl get pods -n kube-system
```

---

## 🛠 Preparación (Días/Semanas Antes)

### 1. 📚 Investigación y Release Notes
```bash
# Consultar versiones disponibles de EKS
aws eks describe-addon-versions --kubernetes-version 1.28

# Verificar APIs obsoletas
kubectl api-resources --verbs=list -o name | xargs -n 1 kubectl get --show-kind --ignore-not-found --all-namespaces
```
**Enlaces importantes:**
- [Release Notes EKS](https://docs.aws.amazon.com/eks/latest/userguide/release-notes.html)
- [Kubernetes Release Notes](https://github.com/kubernetes/kubernetes/releases)

### 2. 💾 Backup del Cluster
```bash
# Backup completo con Velero
velero backup create pre-upgrade-backup-$(date +%Y%m%d-%H%M%S) --include-namespaces="*" --wait

# Verificar backup
velero backup describe pre-upgrade-backup-YYYYMMDD-HHMMSS
```

### 3. 🔍 Verificación de Compatibilidad
- Cluster Autoscaler compatible con nueva versión
- Add-ons (kube-proxy, CoreDNS, VPC-CNI) compatibles
- Al menos 5 IPs disponibles en subred
- Recursos YAML usando APIs compatibles

### 4. 🧪 Pruebas en Entornos Inferiores
- Actualización probada en desarrollo
- Actualización probada en staging
- Período de prueba: 1-2 semanas
- Validación de aplicaciones críticas

---

## ⚡ Proceso de Actualización

### Fase 1: 📈 Actualizar Plano de Control (30 min)
```bash
# Iniciar actualización del plano de control
aws eks update-cluster-version \
    --region <your-region> \
    --name <your-cluster-name> \
    --kubernetes-version 1.29

# Monitorear progreso
aws eks describe-update \
    --region <your-region> \
    --name <your-cluster-name> \
    --update-id <update-id>
```

### Fase 2: 🖥 Actualizar Grupos de Nodos (30 min - 2 hrs)

**Para EKS Managed Node Groups:**
```bash
aws eks update-nodegroup-version \
    --cluster-name <your-cluster-name> \
    --nodegroup-name <your-nodegroup-name> \
    --region <your-region>
```

**Para Node Groups Personalizados:**
```bash
# Proceso por nodo
kubectl cordon <node-name>
kubectl drain <node-name> \
    --ignore-daemonsets \
    --delete-emptydir-data \
    --force \
    --timeout=600s

# Actualizar nodo (modificar ASG/Launch Template)
# Esperar nuevo nodo
kubectl uncordon <new-node-name>
```

### Fase 3: 🔌 Actualizar Add-ons
```bash
# Actualizar add-ons principales
aws eks update-addon --cluster-name <cluster> --addon-name kube-proxy
aws eks update-addon --cluster-name <cluster> --addon-name coredns
aws eks update-addon --cluster-name <cluster> --addon-name vpc-cni
```

---

## 🔍 Post-Actualización

### Verificación Inmediata
```bash
# Estado del cluster
kubectl get nodes
kubectl get pods -A | grep -v Running

# Componentes del sistema
kubectl get pods -n kube-system
kubectl top nodes
```

### Pruebas de Validación
```bash
# Pod de prueba
kubectl run test-pod --image=nginx:alpine --restart=Never
kubectl exec test-pod -- nslookup kubernetes.default.svc.cluster.local
kubectl delete pod test-pod

# Verificar aplicaciones
kubectl get deployments,services,ingress --all-namespaces
```

### Monitoreo Continuo
- Métricas de aplicaciones (24-48 horas)
- Logs de componentes del sistema
- Rendimiento del cluster
- Alertas y notificaciones

---

## 🚨 Comandos de Emergencia
```bash
# Cancelar drenado problemático
kubectl drain <node> --disable-eviction --force

# Forzar eliminación de pod
kubectl delete pod <pod> --force --grace-period=0

# Verificar eventos del cluster
kubectl get events --all-namespaces --sort-by='.lastTimestamp'

# Diagnosticar problemas
kubectl describe node <problem-node>
kubectl logs -n kube-system <problem-pod>
```

---

## ✅ Checklist Final

### Pre-Actualización
- Release notes revisadas y entendidas
- APIs obsoletas identificadas y migradas
- Backup completo del cluster
- Compatibilidad de add-ons verificada
- Pruebas en staging exitosas
- Equipos notificados
- Ventana de mantenimiento coordinada

### Durante Actualización
- Plano de control actualizado exitosamente
- Nodos actualizados con rolling update
- Add-ons actualizados a versión compatible
- Aplicaciones sin interrupciones

### Post-Actualización
- Todos los nodos en versión correcta y estado "Ready"
- Todos los pods del sistema en "Running"
- Aplicaciones respondiendo correctamente
- Pruebas de regresión exitosas
- Métricas estables por 24 horas
- Comunicación de éxito a equipos

---

## 📚 Recursos Adicionales

### Documentación Oficial
- [EKS Update Documentation](https://docs.aws.amazon.com/eks/latest/userguide/update-cluster.html)
- [Kubernetes Version Skew Policy](https://kubernetes.io/docs/setup/release/version-skew-policy/)
- [EKS Release Calendar](https://docs.aws.amazon.com/eks/latest/userguide/kubernetes-versions.html)

### Herramientas
- [Velero Backup Tool](https://velero.io/)
- [kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

### Monitoreo
```bash
# Comandos útiles para monitoreo continuo
kubectl get nodes -w
kubectl top pods -A --containers
kubectl get events --field-selector type=Warning
```

---

> 📝 **Nota:** Esta guía asume mejores prácticas de Kubernetes y EKS. Adaptar según las necesidades específicas de tu entorno y aplicaciones.

⏱ **Tiempo Total Estimado:** 2-4 horas (dependiendo del tamaño del cluster)

🎯 **Objetivo:** Zero downtime durante toda la actualización

