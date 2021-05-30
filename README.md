# Add-TLS-to-a-Kubernetes-Service-with-Ingress

##Introduction
- Your company, SecuriCorp, is using Kubernetes to run a variety of applications. Recently, hackers have been trying various techniques to break into the Kubernetes cluster and steal data.

- Your developers have built a Service called accounts-svc that provides user account data, but the Service itself uses an unencrypted HTTP protocol. This makes communications with that service vulnerable to various forms of attack.

- Use an Ingress to implement TLS termination for the Service.

**Note:** The cluster does not have any Ingress controllers set up. However, for now, all you need to do is implement the Ingress configuration

##Solution

### Generate Self-Signed Certificates for the Service and Store Them in a Secret

1. List the Services on the accounts namespace:
```
kubectl get service -n accounts
```
2. Create a self-signed certificate and key for the accounts-svc Service:
```
openssl req -nodes -new -x509 -keyout accounts.key -out accounts.crt -subj "/CN=accounts.svc"
# check the files after you created them. 
```
3. Create a Secret file to store the certificate and key:
```
vi accounts-tls-certs-secret.yml
```
4. Paste in the following YAML:
```
apiVersion: v1
kind: Secret
type: kubernetes.io/tls
metadata:
  name: accounts-tls-certs
  namespace: accounts
data:
  tls.crt: |
    <base64-encoded cert data from accounts.crt>
    # We will change 
  tls.key: |
    <base64-encoded key data from accounts.key>
     # We will change 
```

5. To save and exit the file, press Escape and enter :wq.

6. Get a Base64-encoded version of the certificate:

```
base64 accounts.crt
```
7. Copy the Base64-encoded string output.

8. Edit the manifest file:
```
vi accounts-tls-certs-secret.yml
```
9. Enter the command ```:set paste``` and ```i``` to enter insert mode.

10. Under tls.crt:, replace the placeholder text with the copied Base64-encoded string output.

11. Press Escape and enter ```:wq```
12. Get a Base64-encoded version of the key:
```
base64 accounts.key
```
13. Copy the Base64-encoded string output.

14. Edit the manifest file again:
```
vi accounts-tls-certs-secret.yml
```
15. Enter the command ```:set paste``` and ```i ```to enter insert mode.

16. Under tls.key:, replace the placeholder text with the copied Base64-encoded string output.

17. Press Escape and enter ```:wq```.

18. Create the Secret:
```
kubectl create -f accounts-tls-certs-secret.yml
```
### Create an Ingress on Top of the Service That Configures TLS Termination
1. Create a YAML manifest for the Ingress:

vi accounts-tls-ingress.yml

2. Paste in the following YAML:

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: accounts-tls
  namespace: accounts
spec:
  tls:
  - hosts:
      - accounts.svc
    secretName: accounts-tls-certs
  rules:
  - host: accounts.svc
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: accounts-svc
            port:
              number: 80
              Press Escape and enter :wq.

3. Create the Ingress:

kubectl create -f accounts-tls-ingress.yml

4. Verify that the Ingress is appropriately mapping to the backend:

kubectl describe ingress accounts-tls -n accounts
