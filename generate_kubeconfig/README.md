### 1. Création d'un service account
```bash
kubectl create namespace dev
kubectl create serviceaccount dev-user --namespace dev
```

### 2. Création d'un role 

Commande : `kubectl apply -f dev-custom-role.yaml`

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: dev
  name: dev-custom-role
rules:
- apiGroups: [""]
  resources: ["pods", "pods/log", "services", "configmaps"]
  verbs: ["get", "watch", "list", "create", "update", "patch", "delete"]
- apiGroups: ["apps"]
  resources: ["deployments", "statefulsets", "replicasets"]
  verbs: ["get", "watch", "list", "create", "update", "patch", "delete"]
```

### 3. Role binding

```bash
kubectl create rolebinding dev-user-binding \
  --role=dev-custom-role \
  --serviceaccount=dev:dev-user \
  --namespace=dev
```

### 4. Générer un token lié au ServiceAccount

Commande : `kubectl apply -f dev-user-token.yaml`

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: dev-user-token
  namespace: dev
  annotations:
    kubernetes.io/service-account.name: dev-user
type: kubernetes.io/service-account-token
```

### 5. Token et certificat CA

```bash
SECRET_NAME="dev-user-token"

# Récupération du token
USER_TOKEN=$(kubectl get secret $SECRET_NAME -n dev -o jsonpath='{.data.token}' | base64 --decode)

# Récupération du certificat CA (en base64, pour kubeconfig)
CA_CERT=$(kubectl get secret $SECRET_NAME -n dev -o jsonpath='{.data.ca\.crt}')
```

### 6. URL de l'API server

```bash
APISERVER=$(kubectl config view --minify -o jsonpath='{.clusters[0].cluster.server}')
```

### 7. Générer le kubeconfig
```bash
cat <<EOF > dev-kubeconfig.yaml
apiVersion: v1
kind: Config
clusters:
- cluster:
    certificate-authority-data: ${CA_CERT}
    server: ${APISERVER}
  name: dev-cluster
contexts:
- context:
    cluster: dev-cluster
    user: dev-user
  name: dev-context
current-context: dev-context
users:
- name: dev-user
  user:
    token: ${USER_TOKEN}
EOF
```