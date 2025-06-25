### 1. Coté client: générer clé privée et CSR
```bash
# clé privée
openssl genrsa -out alice.key 2048
# CSR
openssl req -new -key alice.key -out alice.csr -subj "/CN=alice/O=developers"
```
Envoyer la CSR à l'administrateur du cluster kubernetes

### 2. Administrateur signe la CSR avec le CA du cluster
```bash
openssl x509 -req -in alice.csr \
  -CA /etc/kubernetes/pki/ca.crt \
  -CAkey /etc/kubernetes/pki/ca.key \
  -CAcreateserial \
  -out alice.crt \
  -days 30
```
Envoyer `alice.crt` et `ca.crt` au client

### 3. Création du kubeconfig coté client
```yaml
apiVersion: v1
kind: Config
clusters:
- name: my-cluster
  cluster:
    certificate-authority: /chemin/vers/ca.crt
    server: https://<cluster-endpoint>:6443
users:
- name: alice
  user:
    client-certificate: /chemin/vers/alice.crt
    client-key: /chemin/vers/alice.key
contexts:
- name: alice@my-cluster
  context:
    cluster: my-cluster
    user: alice
current-context: alice@my-cluster
```

### 4. Administrateur créer un role binding pour le client
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: dev
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: alice-binding
  namespace: dev
subjects:
- kind: User
  name: alice
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```