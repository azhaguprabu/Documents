### Configuration User Certificate
```
openssl genrsa -out user.key 2048
openssl req -new -key user.key -subj "/CN=user" -out user.csr
```

### Encrypt base64 string
```
cat user.csr | base64 -w 0
<< note the output line >>
```

### Create Role YAML
```
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: user-role
  namespace: ns
rules:
- apiGroups: [""]
  resources: [ "pods","pods/log" ]
  verbs: [ "get", "watch", "list" ]
```
### Create RoleBinding YAML
```
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: user-rolebinding
  namespace: ns
subjects:
- kind: User
  name: user
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: user-role
  apiGroup: rbac.authorization.k8s.io
```
### Certificate Request YAML
```
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
    name: user-csr
spec:
  request: << csr_base64_encrypt_data >>
  usages:
    - client auth
  signerName: kubernetes.io/kube-apiserver-client
  groups:
    - system: authenticated
```

### Upload Certifiate into K8S and approve
```
kubectl apply -f user-csr.yaml
kubectl certificate approve user-csr

<<< one Year validity default >>>
```

### Create kubeconfig file 
```
kubectl certificate get user -o yaml
<<< note the certificate value >>>

echo "certificate_value..." | base64 --decode 

mkdir ~/.kube

vi ~/.kube/config
apiVersion: v1
kind: Config
clusters:
- cluster:
    certificate-authority-data: << ca base64 decryption data >>
    server: https://<<kube_api | control-plane >>:6443
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    user: user
    namespace: ns
  name: user@kubernetes
current-context: user@kubernetes
preferences: {}
users:
- name: user
  user:
    client-certificate-data: << certificate_base64_encrypt_data >>
    client-key-data: << key_base64_encrypt_data >>
```