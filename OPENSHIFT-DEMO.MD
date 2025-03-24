# Prepare
## login to openshift
`minikube start`

# Show Environment
```
sudo virsh list

kubectl version

kubectl create ns guestbook
kubectl apply -f guestbook/all-in-one/guestbook-all-in-one.yaml -n guestbook
```

## show running pods (redis-master won’t be up)
`kubectl get pods -n guestbook`

## show that redis-master is not up because of Docker Image Format
`kubectl describe pod -l role=master -n guestbook`


## Switch to Wordpress
```
kubectl create ns wordpress
kubectl apply -k ./mysql-wordpress-pd/ -n wordpress
kubectl get pods -n wordpress
```

## Switch to OpenShift
```
kubectl create ns wordpress
kubectl apply -k ./mysql-wordpress-pd/ -n wordpress
kubectl get pods -n wordpress

kubectl delete -k ./mysql-wordpress-pd/ -n wordpress

```
### Switch to bitnami helm chart (Tanzu Application Catalog)
```
helm install my-release oci://registry-1.docker.io/bitnamicharts/wordpress --namespace wordpress --set ingress.enabled=true --set ingress.hostname=wordpress-bwi.apps.ocp4.pleasedonotuse.win

curl http://wordpress-bwi.apps.ocp4.pleasedonotuse.win/


oc new-project guestbook-upstream
oc apply -f guestbook/all-in-one/guestbook-all-in-one.yaml
oc logs -l tier=frontend
```

### Solution 1: Add SCC which allows binding of local ports
```
oc new-project guestbook-insecure
oc adm policy add-scc-to-user anyuid -z  default -n  guestbook-insecure
diff guestbook/all-in-one/guestbook-all-in-one.yaml guestbook/all-in-one/guestbook-all-in-one-openshift-insecure.yaml
oc apply -f guestbook/all-in-one/guestbook-all-in-one-openshift-insecure.yaml
oc logs -l tier=frontend

oc expose service frontend
```

### Solution 2: Rebuild Container-Image to bind/expose port 8080 instead of 80
```
oc new-project guestbook-secure
oc apply -f guestbook/php-redis/build-config.yaml
oc start-build php-redis
oc logs php-redis-1-build -f

diff guestbook/all-in-one/guestbook-all-in-one.yaml guestbook/all-in-one/guestbook-all-in-one-openshift-secure.yaml
oc apply -f guestbook/all-in-one/guestbook-all-in-one-openshift-secure.yaml
oc expose service frontend
curl frontend-guestbook-secure.apps.ocp4.pleasedonotuse.win
```
## Clean up
```
oc delete project guestbook-upstream
oc delete project guestbook-insecure
oc delete project guestbook-secure
oc delete project wordpress

minikube delete
```