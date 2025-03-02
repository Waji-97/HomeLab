# Default TLS for Ingress Controller
I miserably failed with cert-manager. Deploying the helm chart with argocd always gave me an tls bad cert error so I opted for the default SSL cert.

Anyways, there is a `default-ssl-certificate` argument in the nginx ingress controller which is enabled and it refers to `ingress-tls` secret inside `argocd` namespace. 

Firstly, the TLS cert is needed. Let's Encrypt is good but Cloudflare offers 15 years free certificate. From the cloudflare dashboard, create a TLS cert.

![tls](assets/img/tls.png)

It will give us a `.key` and `.crt` file which will be used to create the secret file as below

```bash
kubectl create secret tls ingress-tls --key=homek8s.site.key --cert=homek8s.site.crt -n ingress-nginx --dry-run=client  -o yaml
```
<br>

After that sops will be used once again to encrypt the secret before pushing to Github (Of course it will be applied manually)

```bash
➜  argocd git:(main) ✗ sops -e -i ingress-tls-secret.yaml 
➜  argocd git:(main) ✗ sops -d ingress-tls-secret.yaml| kubectl apply -f -
secret/ingress-tls created

➜  argocd git:(main) ✗ k get secret -n argocd | grep ingress
ingress-tls                   kubernetes.io/tls   2      6s
```

### Ingress Object Config
Each ingress object needs to have the following annotation
```bash
nginx.ingress.kubernetes.io/force-ssl-redirect: "true"
```