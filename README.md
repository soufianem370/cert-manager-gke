# cert-manager-
```
kubectl apply --validate=false \
  -f https://raw.githubusercontent.com/jetstack/cert-manager/v0.13.1/deploy/manifests/00-crds.yaml
 
 kubectl create namespace cert-manager
 helm repo add jetstack https://charts.jetstack.io
 helm repo update
 ```
 ```
 helm install \
    cert-manager jetstack/cert-manager \
    --namespace cert-manager \
    --version v0.13.1
 ```
 ```
 cat <<EOF > test-resources.yaml
  apiVersion: v1
  kind: Namespace
  metadata:
    name: cert-manager-test
  ---
  apiVersion: cert-manager.io/v1alpha2
  kind: Issuer
  metadata:
    name: test-selfsigned
    namespace: cert-manager-test
  spec:
    selfSigned: {}
  ---
  apiVersion: cert-manager.io/v1alpha2
  kind: Certificate
  metadata:
    name: selfsigned-cert
    namespace: cert-manager-test
  spec:
    dnsNames:
      - example.com
    secretName: selfsigned-cert-tls
    issuerRef:
      name: test-selfsigned
  EOF
  ```
  ```
  kubectl apply -f test-resources.yaml
  ```
  ```
  cat <<EOF > ingress.yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: sampleApp-ingress
  annotations:
    # specify the name of the global IP address resource to be associated with the HTTP(S) Load Balancer.
    kubernetes.io/ingress.global-static-ip-name: sampleApp-ip
    # add an annotation indicating the issuer to use.
    cert-manager.io/cluster-issuer: letsencrypt-staging
    # controls whether the ingress is modified ‘in-place’,
    # or a new one is created specifically for the HTTP01 challenge.
    acme.cert-manager.io/http01-edit-in-place: "true"
  labels:
    app: sampleApp
spec:
  tls: # < placing a host in the TLS config will indicate a certificate should be created
  - hosts:
    - example.example.com
    secretName: sampleApp-cert-secret # < cert-manager will store the created certificate in this secret
  rules:
  - host: example.example.com
    http:
      paths:
      - path: sample/app/path/*
        backend:
          serviceName: sampleApp-service
          servicePort: 8080
EOF
```
```
kubectl apply -f ingress.yaml
```
## Verify
```
kubectl get certificate
kubectl describe certificate sampleApp-cert-secret
kubectl describe secret sampleApp-cert-secret

  
