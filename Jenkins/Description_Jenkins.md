# Jenkins on Kubernetes

Testing Continous Integration and Continous Delivery on Kubernetes via jenkins.
Code used for deployment found in this [GitRepo](https://github.com/StraysWonderland/simple-java-maven-app/blob/master/jenkins/scripts/deliver.sh)
## Workflow

start by either starting minikube cluster with additional ressources
```bash
minikube start -memory 8192
```
 or creating a kind cluster with 2 workers using the following yaml configuration file
```yaml 
#three node cluster config
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
    - role: control-plane
    - role: worker
    - role: worker
```
and following bash command
```bash
kind create cluster --config kind-config.yaml
```

create namespace for jenkins
```bash
kubectl create namespace devops-tools
```

create serviceAccount
```yaml
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: jenkins-admin
rules:
  - apiGroups: [""]
    resources: ["*"]
    verbs: ["*"]

---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: jenkins-admin
  namespace: devops-tools

---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: jenkins-admin
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: jenkins-admin
subjects:
- kind: ServiceAccount
  name: jenkins-admin
  namespace: devops-tools
```

create volume for persistent storage
```yaml
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: local-storage
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer

---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: jenkins-pv-volume
  labels:
    type: local
spec:
  storageClassName: local-storage
  claimRef:
    name: jenkins-pv-claim
    namespace: devops-tools
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  local:
    path: /mnt
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - worker-node01

---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: jenkins-pv-claim
  namespace: devops-tools
spec:
  storageClassName: local-storage
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 3Gi
```

create deployment
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: jenkins
  namespace: devops-tools
spec:
  replicas: 1
  selector:
    matchLabels:
      app: jenkins-server
  template:
    metadata:
      labels:
        app: jenkins-server
    spec:
      securityContext:
            fsGroup: 1000 
            runAsUser: 1000
      serviceAccountName: jenkins-admin
      containers:
        - name: jenkins
          image: jenkins/jenkins:lts
          resources:
            limits:
              memory: "2Gi"
              cpu: "1000m"
            requests:
              memory: "500Mi"
              cpu: "500m"
          ports:
            - name: httpport
              containerPort: 8080
            - name: jnlpport
              containerPort: 50000
          livenessProbe:
            httpGet:
              path: "/login"
              port: 8080
            initialDelaySeconds: 90
            periodSeconds: 10
            timeoutSeconds: 5
            failureThreshold: 5
          readinessProbe:
            httpGet:
              path: "/login"
              port: 8080
            initialDelaySeconds: 60
            periodSeconds: 10
            timeoutSeconds: 5
            failureThreshold: 3
          volumeMounts:
            - name: jenkins-data
              mountPath: /var/jenkins_home         
      volumes:
        - name: jenkins-data
          persistentVolumeClaim:
              claimName: jenkins-pv-claim

```
bash commands to apply serviceAccount, Volume and deployment
```bash
kubectl apply -f FILE-PATH.yaml

kubectl apply -f serviceAccount.yaml
kubectl apply -f jenkins-volume.yaml
kubectl apply -f deployment.yaml
```

### helm workflow

```bash
kind create cluster

kubectl create ns jenkins
```
```bash
helm install <name> jenkins/jenkins -n devops-tools
```

### access jenkins

- get admin password for jenkins
```bash
printf $(kubectl get secret --namespace devops-tools jenkins -o jsonpath="{.data.jenkins-admin-password}" | base64 --decode);echo

NTZoblFhOWdqa2pMWlBxQ3NLMkVQNA==

OR TO COPY DIRECTLY TO CLIPBOARD

echo $(kubectl get secret --namespace devops-tools jenkins -o jsonpath="{.data.jenkins-admin-password}" | base64 --decode) | clipboard

```
- forward jenkins ports
```bash
 kubectl -n devops-tools port-forward jenkins-0 8080:8080
```


## pipeline templating
## build agent setup
## pipeline trigger
## hashicorp vault/azure keyvault integration
Jenkins provides a hashicorp vault plugin, as well as a azure key vault plugion to set environment variables from a HashiCorp/Azure Key Vault secret.
- configurable via jenkinsfile
```yaml
    node {
        // define the secrets and the env variables
        // engine version can be defined on secret, job, folder or global.
        // the default is engine version 2 unless otherwise specified globally.
        def secrets = [
            [path: 'secret/testing', engineVersion: 1, secretValues: [
                [envVar: 'testing', vaultKey: 'value_one'],
                [envVar: 'testing_again', vaultKey: 'value_two']]],
            [path: 'secret/another_test', engineVersion: 2, secretValues: [
                [vaultKey: 'another_test']]]
        ]

        // optional configuration, if you do not provide this the next higher configuration
        // (e.g. folder or global) will be used
        def configuration = [vaultUrl: 'http://my-very-other-vault-url.com',
                            vaultCredentialId: 'my-vault-cred-id',
                            engineVersion: 1]
        // inside this block your credentials will be available as env variables
        withVault([configuration: configuration, vaultSecrets: secrets]) {
            sh 'echo $testing'
            sh 'echo $testing_again'
            sh 'echo $another_test'
        }
    }
```
## deployment pipeline
- set up multiple pipelines via ui or groovy script
```yaml
pipeline {
    agent any

    stages {
        stage ('Deployment Stage') {
            steps {
                withMaven(maven : 'maven_3_5_0') {
                    sh 'mvn deploy'
                }
            }
        }
    }
}
```

## Runtime parameter mit values und freitext input feld


# Jenkins and Job Builder Plugin