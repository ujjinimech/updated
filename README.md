# GitOps
- Bootstrap both the clusters with flux
#### For Production
~~~
flux bootstrap github \
 --owner=sathyap0104 \
 --repository=GitOps \
 --branch=main \
 --read-write-key=true \
 --private=true \
 --path=cluster/production \
 --personal=true    
~~~
#### For Staging
~~~
flux bootstrap github \
--owner=sathyap0104 \
--repository=GitOps \
--branch=main \
--read-write-key=true \
--private=true \
--path=cluster/staging \
--personal=true    
~~~
***
### apps
**In base dir**
- create namespeace
- create source for the helmcharts
~~~
flux create source helm bitnami --interval=30s --url=https://charts.bitnami.com/bitnami
~~~
- create helmrelease for nginx charts
~~~
flux create helmrelease nginx  \ 
--interval=30s  \ 
--release-name=nginx  \
--source=HelmRepository/bitnami  \ 
--chart=nginx \
--export   | tee apps/base/nginx/release.yaml
~~~
- Now create kustomization for both files in the dir
~~~
kubectl kustomize <dirname>
~~~
**In production**
- create a helmrelease for value 
~~~
flux create helmrelease nginx   \
--release-name=nginx   \
--source=HelmRepository/bitnami   \
--chart=nginx  \ 
--chart-version=">9.6.2"   \
--export   | tee apps/production/nginx-values.yaml
~~~
- create kustomization file for production and nginx dir

**In Staging**
- create a helmrelease for value 
~~~
flux create helmrelease nginx   \
--release-name=nginx   \
--source=HelmRepository/bitnami   \
--chart=nginx  \ 
--chart-version=">9.6.2"   \
--export   | tee apps/staging/nginx-values.yaml
~~~
- create kustomization file for staging and nginx dir
***
### cluster
**In production**
- create a source for our git repo 
~~~
flux create source git nginx-repo \
--url=https://github.com/sathyap0104/GitOps.git \
--branch=main \
--username=<username> \
--password=<token>
~~~
- create a kustomization for the production to communicate with nginx
~~~
flux create kustomization nginx   \
--source=nginx-repo \
--path="./apps/production"   \
--prune=true   \
--interval=30s   \
--health-check="HelmRelease/nginx.nginx"   \
--health-check-timeout=1m   \
--export | tee cluster/production/apps.yaml
~~~
**In staging**
- create a source for our git repo 
~~~
flux create source git nginx-repo \
--url=https://github.com/sathyap0104/GitOps.git \
--branch=main \
--username=<username> \
--password=<token>
~~~
- create a kustomization for the staging to communicate with nginx
~~~
flux create kustomization nginx   \
--source=nginx-repo \
--path="./apps/staging"   \
--prune=true   \
--interval=30s   \
--health-check="HelmRelease/nginx.nginx"   \
--health-check-timeout=1m   \
--export | tee cluster/staging/apps.yaml
~~~

## Testing

**now to test it** 
- For source
~~~
watch flux get sources git
~~~
- For Kustomization
~~~
watch flux get kustomization
~~~
- For helmrelaese
~~~
watch flux get helmreleases -n nginx
~~~
- For pods
~~~
kubectl get pods -n flux-system
~~~
- check output
~~~
kubectl describe pod <podname>
~~~
- check with the IP address
