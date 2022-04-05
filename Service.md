# Service

Demonstrate K8s service to expose applications

*  create the deployment
``` 
kubectl create deploy nginx --image=nginx 
```
* scale the deployment to 3 replicas

``` 
kubectl scale deploy nginx --replicas=3

kubectl get pods -o wide
```

* expose the application by adding K8s service

```
kubectl expose deploy nginx --port=80
kubectl describe svc nginx
kubectl get svc
```

* fetch service IP address

```
SVCIP=$(kubectl describe svc nginx | grep '^IP' | awk '{ print $2 }')
```

* Test the application access via K8s service

```
curl ${SVCIP}:80
```

* Service uses the Selector label to connect to Pods , edit the selector label in the service definition

```
sed -i -e 's/app\: nginx/app\: Nginx/g' nginx.yaml
kubectl apply -f nginx.yaml
```

```
sed -i -e 's/app\: Nginx/app\: nginx/g' nginx.yaml
kubectl delete svc nginx
kubectl apply -f nginx.yaml
kubectl get endpoints
```


![img.png](service/svc.png)