> Install nginx ingress controller and cert manager
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.1.1/deploy/static/provider/cloud/deploy.yaml
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.17.0/cert-manager.yaml
```

> Install Cert issuer and certificate
```
kubectl apply -f issuer.yaml  -n erpnext
kubectl apply -f cert.yaml -n erpnext
```

> Install nfs provisioner 
```
kubectl create namespace nfs
helm repo add nfs-ganesha-server-and-external-provisioner https://kubernetes-sigs.github.io/nfs-ganesha-server-and-external-provisioner
helm upgrade --install -n nfs in-cluster nfs-ganesha-server-and-external-provisioner/nfs-server-provisioner --set 'storageClass.mountOptions={vers=4.1}' --set persistence.enabled=true --set persistence.size=8Gi
```

> Install base erpnext
```
helm repo add frappe https://helm.erpnext.com
helm upgrade --install frappe-bench --namespace erpnext frappe/erpnext --set persistence.worker.storageClass=nfs --set createSite.enabled=true --set createSite.siteName=erpnext.ezy.dev --set ingress.enabled=true --set ingress.className=nginx --set ingress.hosts[0].host=erpnext.ezy.dev --set ingress.hosts[0].paths[0].path=/ --set ingress.hosts[0].paths[0].pathType=ImplementationSpecific --set ingress.tls[0].secretName=erpnext-tls --set ingress.tls[0].hosts[0]=erpnext.ezy.dev --set createSite.adminPassword="secret"
```

> Create new site
```
helm template frappe-bench -n erpnext frappe/erpnext -f custom-values.yaml -s templates/job-create-site.yaml > create-new-site-job.yaml
kubectl apply -f create-new-site-job.yaml -n erpnext
```

> Use
```
open https://erpnext.ezy.dev
user "Administrator/secret
```
