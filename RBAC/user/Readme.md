# User Creation
## User Key Generation 
```bash
openssl genrsa -out mucahid.key 2048
```

## User Certificate Signing Request
```bash
openssl req -new -key mucahid.key -out mucahid.csr -subj "/CN=mucahid@k8s/O=senior"
```
* CN: User Name
* O: Group Name


## User Certificate Signing
```bash
cat <<EOF | kubectl apply -f -
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: mucahid-aydin
spec:
  groups:
  - system:authenticated
  request: $(cat mucahid.csr | base64 | tr -d "\n")
  signerName: kubernetes.io/kube-apiserver-client
  usages:
  - client auth
EOF
```
* `request`: CSR file content