### Домашнее задание к занятию «Конфигурация приложений» Баранов Сергей

### Цель задания

В тестовой среде Kubernetes необходимо создать конфигурацию и продемонстрировать работу приложения.

------

### Чеклист готовности к домашнему заданию

1. Установленное K8s-решение (например, MicroK8s).
2. Установленный локальный kubectl.
3. Редактор YAML-файлов с подключённым GitHub-репозиторием.

------

### Инструменты и дополнительные материалы, которые пригодятся для выполнения задания

1. [Описание](https://kubernetes.io/docs/concepts/configuration/secret/) Secret.
2. [Описание](https://kubernetes.io/docs/concepts/configuration/configmap/) ConfigMap.
3. [Описание](https://github.com/wbitt/Network-MultiTool) Multitool.

------

### Задание 1. Создать Deployment приложения и решить возникшую проблему с помощью ConfigMap. Добавить веб-страницу

1. Создать Deployment приложения, состоящего из контейнеров nginx и multitool.

[deployment.yaml](https://github.com/12sergey12/12.8-Kubernetes_Application-Configuration/blob/main/deployment.yaml)

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  replicas: 1
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
        - image: wbitt/network-multitool
          imagePullPolicy: IfNotPresent
          volumeMounts:
            - name: nginx-index-file
              mountPath: /usr/share/nginx/html/
          name: network-multitool
          resources:
            limits:
              cpu: 200m
              memory: 512Mi
            requests:
              cpu: 100m
              memory: 256Mi
      volumes:
      - name: nginx-index-file
        configMap:
          name: index-html-configmap
      initContainers:
      - name: init-myservice
        image: busybox:1.28
        command: ['sh', '-c', "until nslookup myservice.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for myservice; sleep 2; done"]
```

```
root@baranov:/home/baranovsa/kube_2.3# kubectl apply -f deployment.yaml
deployment.apps/myapp-pod created
root@baranov:/home/baranovsa/kube_2.3# kubectl get pod
NAME                    	READY   STATUS	RESTARTS   AGE
myapp-pod-76c587bb79-jpbgw     	1/1 	Running   0        10s
root@baranov:/home/baranovsa/kube_2.3#
```

2. Решить возникшую проблему с помощью ConfigMap.

```
kubectl apply -f myservice.yaml
service/myservice created
root@baranov:/home/baranovsa/kube_2.3#
root@baranov:/home/baranovsa/kube_2.3# kubectl get svc
NAME     	TYPE    	CLUSTER-IP   	EXTERNAL-IP   PORT(S)    	AGE
kubernetes      ClusterIP       10.96.0.1    	<none>        443/TCP    	32d
myservice	NodePort	10.105.150.123  <none>        80:32000/TCP      39s
root@baranov:/home/baranovsa/kube_2.3# kubectl apply -f index-html-configmap.yaml
configmap/index-html-configmap created
root@baranov:/home/baranovsa/kube_2.3# kubectl get cm
NAME                     	DATA    AGE
index-html-configmap   	        1  	16s
ingress-nginx-controller        1  	20d
kube-root-ca.crt          	1  	32d
root@baranov:/home/baranovsa/kube_2.3
```

3. Продемонстрировать, что pod стартовал и оба конейнера работают.

```
root@baranov:/home/baranovsa/kube_2.3# kubectl get pod
NAME                            READY   STATUS  RESTARTS   AGE
myapp-pod-76c587bb79-jpbgw      1/1     Running   0        10s
root@baranov:/home/baranovsa/kube_2.3#
```



4. Сделать простую веб-страницу и подключить её к Nginx с помощью ConfigMap. Подключить Service и показать вывод curl или в браузере.

[index-html-configmap.yaml](https://github.com/12sergey12/12.8-Kubernetes_Application-Configuration/blob/main/index-html-configmap.yaml)

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: index-html-configmap
  namespace: default
data:
  index.html: |
    <html>
    <h1>Welcome</h1>
    </br>
    <h1>Hi! This is a configmap Index file </h1>
    </html
```

[myservice.yaml](https://github.com/12sergey12/12.8-Kubernetes_Application-Configuration/blob/main/myservice.yaml)

```
apiVersion: v1
kind: Service
metadata:
  name: myservice
spec:
  selector:
    app: myapp
  type: NodePort
  ports:
    - name: web
      port: 80
      nodePort: 32000
      targetPort: 80
```

```
root@baranov:/home/baranovsa/kube_2.3# kubectl describe pod myapp-pod-76c587bb79-jpbgw
Name:         	myapp-pod-76c587bb79-jpbgw
Namespace:    	default
Priority:     	0
Service Account:  default
Node:         	minikube/192.168.49.2
Start Time:   	Tue, 02 Apr 2024 20:32:43 +0700
Labels:       	app=myapp
              	pod-template-hash=76c587bb79
Annotations:  	<none>
Status:       	Running
IP:           	10.244.0.100
IPs:
  IP:       	10.244.0.100
Controlled By:  ReplicaSet/myapp-pod-76c587bb79
Init Containers:
  init-myservice:
	Container ID:  docker://816cceb55f6ea89bd48a930e722b87657489f3323de074c0d1070a0ea21879fa
	Image:     	busybox:1.28
	Image ID:  	docker-pullable://busybox@sha256:141c253bc4c3fd0a201d32dc1f493bcf3fff003b6df416dea4f41046e0f37d47
	Port:      	<none>
	Host Port: 	<none>
	Command:
  	sh
  	-c
  	until nslookup myservice.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for myservice; sleep 2; done
	State:      	Terminated
  	Reason:   	Completed
  	Exit Code:	0
  	Started:  	Tue, 02 Apr 2024 20:32:49 +0700
  	Finished: 	Tue, 02 Apr 2024 20:32:49 +0700
	Ready:      	True
	Restart Count:  0
	Environment:	<none>
	Mounts:
  	/var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-zpr7l (ro)
Containers:
  network-multitool:
	Container ID:   docker://bb2f419d7fc172b479da487bf881f7b6280bbcb3849bc62f4a40b42eb591e267
	Image:      	wbitt/network-multitool
	Image ID:   	docker-pullable://wbitt/network-multitool@sha256:d1137e87af76ee15cd0b3d4c7e2fcd111ffbd510ccd0af076fc98dddfc50a735
	Port:       	<none>
	Host Port:  	<none>
	State:      	Running
  	Started:  	Tue, 02 Apr 2024 20:32:51 +0700
	Ready:      	True
	Restart Count:  0
	Limits:
  	cpu: 	200m
  	memory:  512Mi
	Requests:
  	cpu:    	100m
  	memory: 	256Mi
	Environment:  <none>
	Mounts:
  	/usr/share/nginx/html/ from nginx-index-file (rw)
  	/var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-zpr7l (ro)
Conditions:
  Type          	Status
  Initialized   	True
  Ready         	True
  ContainersReady   True
  PodScheduled  	True
Volumes:
  nginx-index-file:
	Type:  	ConfigMap (a volume populated by a ConfigMap)
	Name:  	index-html-configmap
	Optional:  false
  kube-api-access-zpr7l:
	Type:                	Projected (a volume that contains injected data from multiple sources)
	TokenExpirationSeconds:  3607
	ConfigMapName:       	kube-root-ca.crt
	ConfigMapOptional:   	<nil>
	DownwardAPI:         	true
QoS Class:               	Burstable
Node-Selectors:          	<none>
Tolerations:             	node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                         	node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type	Reason 	Age   From           	Message
  ----	------ 	----  ----           	-------
  Normal  Scheduled  33s   default-scheduler  Successfully assigned default/myapp-pod-76c587bb79-jpbgw to minikube
  Normal  Pulled 	29s   kubelet        	Container image "busybox:1.28" already present on machine
  Normal  Created	28s   kubelet        	Created container init-myservice
  Normal  Started	27s   kubelet        	Started container init-myservice
  Normal  Pulled 	26s   kubelet        	Container image "wbitt/network-multitool" already present on machine
  Normal  Created	25s   kubelet        	Created container network-multitool
  Normal  Started	25s   kubelet        	Started container network-multitool
root@baranov:/home/baranovsa/kube_2.3#

```
Проверяем curl

```
root@baranov:/home/baranovsa/kube_2.3# curl 192.168.49.2:32000
<html>
<h1>Welcome</h1>
</br>
<h1>Hi! This is a configmap Index file </h1>
</html
root@baranov:/home/baranovsa/kube_2.3#
```

5. Предоставить манифесты, а также скриншоты или вывод необходимых команд.

Deployment [deployment.yaml](https://github.com/12sergey12/12.8-Kubernetes_Application-Configuration/blob/main/deployment.yaml)

ConfigMap [index-html-configmap.yaml](https://github.com/12sergey12/12.8-Kubernetes_Application-Configuration/blob/main/index-html-configmap.yaml)

Service [myservice.yaml](https://github.com/12sergey12/12.8-Kubernetes_Application-Configuration/blob/main/myservice.yaml)

Проверяем curl

```
root@baranov:/home/baranovsa/kube_2.3# curl 192.168.49.2:32000
<html>
<h1>Welcome</h1>
</br>
<h1>Hi! This is a configmap Index file </h1>
</html
root@baranov:/home/baranovsa/kube_2.3#
```

И через браузер

![monitoring](https://github.com/12sergey12/12.8-Kubernetes_Application-Configuration/blob/main/images/12.8_kube_2.3_1.png)


------


### Задание 2. Создать приложение с вашей веб-страницей, доступной по HTTPS 

1. Создать Deployment приложения, состоящего из Nginx.

[ConfigMapDep2.yaml](https://github.com/12sergey12/12.8-Kubernetes_Application-Configuration/blob/main/ConfigMapDep2.yaml)

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  replicas: 1
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
        - image: wbitt/network-multitool
          imagePullPolicy: IfNotPresent
          env:
            - name: HTTP_PORT
              value: "80"
            - name: HTTPS_PORT
              value: "443"
          ports:
            - containerPort: 80
              name: http-port
            - containerPort: 443
              name: https-port
          volumeMounts:
            - name: nginx-index-file
              mountPath: /usr/share/nginx/html/
          name: network-multitool
          resources:
            limits:
              cpu: 200m
              memory: 512Mi
            requests:
              cpu: 100m
              memory: 256Mi
      volumes:
      - name: nginx-index-file
        configMap:
          name: index-html-configmap
      initContainers:
      - name: init-myservice
        image: busybox:1.28
        command: ['sh', '-c', "until nslookup myservice.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for myservice; sleep 2; done"]
```


2. Создать собственную веб-страницу и подключить её как ConfigMap к приложению.

[index-html-configmap.yaml](https://github.com/12sergey12/12.8-Kubernetes_Application-Configuration/blob/main/index-html-configmap.yaml)

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: index-html-configmap
  namespace: default
data:
  index.html: |
    <html>
    <h1>Welcome</h1>
    </br>
    <h1>Hi! This is a configmap Index file </h1>
    </html
```


3. Выпустить самоподписной сертификат SSL. Создать Secret для использования сертификата.

```
root@baranov:/home/baranovsa/kube_2.3# openssl req -x509 -newkey rsa:4096 -sha256 -nodes -keyout tls.key -out tls.crt -subj "/CN=my-app.com" -days 365
Generating a RSA private key
.......................................++++
..................................................++++
writing new private key to 'tls.key'
-----
root@baranov:/home/baranovsa/kube_2.3#

```
[tls.crt](https://github.com/12sergey12/12.8-Kubernetes_Application-Configuration/blob/main/tls.crt)

[tls.key](https://github.com/12sergey12/12.8-Kubernetes_Application-Configuration/blob/main/tls.key)


4. Создать Ingress и необходимый Service, подключить к нему SSL в вид. Продемонстировать доступ к приложению по HTTPS. 

Создаем ингресс и сервис

ingress [ingress.yaml](https://github.com/12sergey12/12.8-Kubernetes_Application-Configuration/blob/main/ingress.yaml)

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
spec:
  rules:
  - host: my-app.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: myservice
            port:
              number: 80
  tls:
    - hosts:
      - my-app.com
      secretName: my-app-secret-tls
```

myservice2 [myservice.yaml](https://github.com/12sergey12/12.8-Kubernetes_Application-Configuration/blob/main/myservice.yaml)

```
apiVersion: v1
kind: Service
metadata:
  name: myservice
spec:
  selector:
    app: myapp
  type: NodePort
  ports:
    - name: web
      port: 80
      nodePort: 32000
      targetPort: 80
```

```
root@baranov:/home/baranovsa/kube_2.3# kubectl apply -f ingress.yaml
ingress.networking.k8s.io/my-ingress created
root@baranov:/home/baranovsa/kube_2.3# kubectl get ingress
NAME     	CLASS   HOSTS    	ADDRESS PORTS   AGE
my-ingress      nginx   my-app.com             	80, 443 23s
root@baranov:/home/baranovsa/kube_2.3#

```


Создаем secret


```
root@baranov:/home/baranovsa/kube_2.3# kubectl create secret tls my-app-secret-tls --cert=./tls.crt --key=./tls.key
secret/my-app-secret-tls created
root@baranov:/home/baranovsa/kube_2.3# kubectl get secret -o yaml
apiVersion: v1
items:
- apiVersion: v1
  data:
	tls.crt: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUZDekNDQXZPZ0F3SUJBZ0lVQnh1bHlGUlJKenBQUDFZcmRVRGd4NGkraFVjd0RRWUpLb1pJaHZjTkFRRUwKQlFBd0ZURVRNQkVHQTFVRUF3d0tiWGt0WVhCd0xtTnZiVEFlRncweU5EQTBNREl4TlRBek5UTmFGdzB5TlRBMApNREl4TlRBek5UTmFNQlV4RXpBUkJnTlZCQU1NQ20xNUxXRndjQzVqYjIwd2dnSWlNQTBHQ1NxR1NJYjNEUUVCCkFRVUFBNElDRHdBd2dnSUtBb0lDQVFESk56MUp3NDhVTisyZFNnWlZvR0tTbWtVK0RIQ3NFL2Rhc3hOU2RmdzUKMElPc0pSeHU4b1dhYlBwQmRsK3J0YTJCZ3FSNVBBTkhaQVlwSjNuUDB4cnJrNmo3UVk0K3dQRi91bUk1eWJUMQp6cHZvQ3JxMmM5Y0JsQnNJTGVkaGZheGFrdGh1RjRHNnZMbnhOOXBDa3A0M1kxS3JheWd3MUcvem4wdXJQUUJvClA4NExvc2xuQ0s2TWpCSitaUjI3ZTdhSUpRdk9wV0kveHJnYXUzeG9zd25IeVhnWnlIZThncVZsMjVRa1ZjQlUKTS9rRHZvWGQxMVZCUjExcjFSWXVkTGRIaFlwVDhrZlhUVmZ3bUNtTjFHcFNhZ0ZweldyNnBWY0h5WTBKVm1YbgphVGcyTmFzNzEzVTVrN01qZ2cwV09sV0Qra3B6N2dma08xTGcvZXFOYU85TldSTG4xK2FvazRkc0VSVGNBREFvCmpwZEhYS1NUMkpYd0JQV2d4bTBHbVJ4cTU2M1h6blpySkZNN2VjcDliSFZYMklRRi96WlNQbWszb1l0TVRadmEKdnp5WXRoTDFtbUxVQ2tLSVZZNWVVSTNuZ3lvQWkzeGxISHgrZ200bDFwVGV3ZjA0TkdHMjFJZjdVOTl6eWJycQpSaDBXY3FTdUhtTFJCVXdGV1JyOWVaSFR3RXVmV1JMSXkzOTlMTDZoQTFJbEtmZWMwVHpsWHVTZUlMMlZZbXhoCmsxcEJ1Y1VQU2dJanErSWQxM2RZaVVXTDJiOW52bGRFNDJSc1lkd3lFMTlqU0FyYzR1My9neGhrN0JrOFkvYmUKNVNkWVZCRUt6a0wwWm5uOGUrNkVadkU1ZDdPZWhqdTFCQmZtcTh4Z2U0RDRBcGMxcUdOeHl2OEJOZHZhazhFQQpzUUlEQVFBQm8xTXdVVEFkQmdOVkhRNEVGZ1FVUmhsQ292dFlkSzdRWm1wb1ZSMjlOY2VPMGc4d0h3WURWUjBqCkJCZ3dGb0FVUmhsQ292dFlkSzdRWm1wb1ZSMjlOY2VPMGc4d0R3WURWUjBUQVFIL0JBVXdBd0VCL3pBTkJna3EKaGtpRzl3MEJBUXNGQUFPQ0FnRUFaekVDNW5jWTJITzJlK0NEM2d1MjNFNVdOZnRFakE0WGpHOURndTJBam81RwpUVldnSGFUcjBrWWI3NjdGeSt3MDQ2RkJTbTJtbExYcy9lbFZvWTFuUU5EOGt4VnI3c1dCYjQ2Y0NiWHNOdTBFCnZvcVBTaCtyU1UrSURFMEVjMmN6K2ZVMzVtUnVZdVpQWFZsQXc5NDgwYS9NUTdtNnM3ODVpNnFBVzZ3SEY1cUMKMkNuN0dNQTNlYzBFZytlTFZSa05EemN3SXcvWHBxNWNRWm5mS1pvSWJheEVtQmF2bE5rV0Fld3VpUlZoWmVXdQpsOWp6SEF4Sm1kb25wa1ZZNC9HUS9CeE1tYmdNU3I1MUNCeDJHMnJaUXQyTnhHY2lvZzNTV0V4dEtnb3l5ZUdyCmlQZFFySm5XM2t2TnRjM2tmU1N5WlVJNHhsdUFtbGM0ZVlLSCtkaERMUlhNUlVBMkc3d3Q1clZJU0xnSldWdGYKbmJOQWJjQmhDWjg0NGw5MWdoVVlva3JOYTNvTTJzSURodEUxYlhVME15OS9XRkphV3dTVW4yR2d6bWlLazI1WgpIcUdVVVJ0SjYrVlRIWXhpQmRCc3d0bC95OEt5eU00MjErMWtGSlNEeVRTb3V2ZUVZR2RTWkx2UDVueThjWHJFCkwrb0txakJac1B3ZUVDMzNROTl0MUJwckZDZ0VzL01JcXdzczZHUHo0NFE3K1RPM3ZlNkUxQkNoVnJXc1F4T2gKREUwNUh1L21tclR5REFCOHlOWVNWZTJTTEwvK2JsditnYkhiVXFxVFFlVktHZmc5ZEg5ZHhVckdqU3pCTXNCVwpNRW51NUtoZ0xSakNCTzVsR0tnRWxoZGlEUGUyd24xRlVyRHdhSkRkbzJhOHlFSlk0eGVhTGxXTlppcGN4cFE9Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K
	tls.key: LS0tLS1CRUdJTiBQUklWQVRFIEtFWS0tLS0tCk1JSUpRUUlCQURBTkJna3Foa2lHOXcwQkFRRUZBQVNDQ1Nzd2dna25BZ0VBQW9JQ0FRREpOejFKdzQ4VU4rMmQKU2daVm9HS1Nta1UrREhDc0UvZGFzeE5TZGZ3NTBJT3NKUnh1OG9XYWJQcEJkbCtydGEyQmdxUjVQQU5IWkFZcApKM25QMHhycms2ajdRWTQrd1BGL3VtSTV5YlQxenB2b0NycTJjOWNCbEJzSUxlZGhmYXhha3RodUY0RzZ2TG54Ck45cENrcDQzWTFLcmF5Z3cxRy96bjB1clBRQm9QODRMb3NsbkNLNk1qQkorWlIyN2U3YUlKUXZPcFdJL3hyZ2EKdTN4b3N3bkh5WGdaeUhlOGdxVmwyNVFrVmNCVU0va0R2b1hkMTFWQlIxMXIxUll1ZExkSGhZcFQ4a2ZYVFZmdwptQ21OMUdwU2FnRnB6V3I2cFZjSHlZMEpWbVhuYVRnMk5hczcxM1U1azdNamdnMFdPbFdEK2twejdnZmtPMUxnCi9lcU5hTzlOV1JMbjErYW9rNGRzRVJUY0FEQW9qcGRIWEtTVDJKWHdCUFdneG0wR21SeHE1NjNYem5ackpGTTcKZWNwOWJIVlgySVFGL3paU1BtazNvWXRNVFp2YXZ6eVl0aEwxbW1MVUNrS0lWWTVlVUkzbmd5b0FpM3hsSEh4KwpnbTRsMXBUZXdmMDROR0cyMUlmN1U5OXp5YnJxUmgwV2NxU3VIbUxSQlV3RldScjllWkhUd0V1ZldSTEl5Mzk5CkxMNmhBMUlsS2ZlYzBUemxYdVNlSUwyVllteGhrMXBCdWNVUFNnSWpxK0lkMTNkWWlVV0wyYjludmxkRTQyUnMKWWR3eUUxOWpTQXJjNHUzL2d4aGs3Qms4WS9iZTVTZFlWQkVLemtMMFpubjhlKzZFWnZFNWQ3T2VoanUxQkJmbQpxOHhnZTRENEFwYzFxR054eXY4Qk5kdmFrOEVBc1FJREFRQUJBb0lDQUM3NDBUcmVCeEdlT0w4R0g5SnZYWE9CCnhmWkZLeXhRZ09iUWRRdEQ5Yng5VzYrYy9qVThNa29wZElaQm52WFR0SXhPTmgwREVIVGkrYmt6dVBEMkJBdkkKYmFNeDI0WDB2cXVsay9hUGlzbnpFRndyM1Fob3BHRis5SXlGUXNWMGMvNjAxd2RrUDJtYlV3RG5tL0Q4anlpNAo2L3lPU0ZTVjB3aVpRQzNhVUxVRmFCRTVVRERZU3Y4T1YrYmZyZ0F0VWlBWUhkaUFyUjNjUXZBaW9sbENxbDdVClBsQXVoeDBmbG9MZnJvYUk2aHYrQWZ5SXhuRlNLbG5SMGhJeExUdHREYmdKaVpBdE4rNGY1UTZLUnZFUUREM3UKOXlOK3k2RjFaWXF3MTFhOXI0NUU5dFZZMC85bFBiMVNLSGg2akptS3hwbkhuUWZSSWxCNk5NL0hGckVxTk9wOApVek9ib08vRFM2b01KYkRCREVmV0ZwSWlpRTB3NXNJTnRDVlp4RGVhbjE1ejZ4VVo4a24weWROMUlyakgxWXcxCjJHeE51Y1VSVUpJdjN0dVZiRTM4b1p2eHdEWmF5aXpLdnlyVXhPL2g1Yk5TOEd2QmlsUW1XMDlLMEgzYVpKMXAKdks2a1V6THJGN3d3OHVFTnBOMDBxN3dlakVpS05CME94Q2pJSGE1MVJZTHNDUUVsYnJJWVl6bnZ6N1dsWlhiRgpZUU03Y3pFdXdUdDhYc0JMYzdDb2NyazkraXh5TTNFVE84cElQY3JyYWM0WEZ2bTJKMVgzWWI1L1BNR2ZRa2VSCmxPNXJUMXBCOFNNS2hnRHhCbWZ0N1dzWGRCZm9nQWtrclBTeWZEakorMzM0YmRVOUVwYlh0TDhFRGdBTFIwYzUKUEpra050M3dDbThBeTlYQm9xc3RBb0lCQVFEeXdBNTVKTHlKTWFNWEc0ak9IcHRTbFZKSUw2QS9HRXUzc2tFbgpIUDhDcDFNQ1RBV0prMFRZQVdKdDZRWmRJWWRoa3JDei9VL1ZjWnMwUmxkR0ExL2Zoc2wzWTZEMy9XMktwRzhWCnJoVzBpdkdmRFFnUWx0RkltTWU3QTh5TzlCbXNTcTlaSDJHWlNoZ0c1Nmx1ZVAwUXkvNE5YU0lUZk5NeHp1bGkKZWdTK21TSkgvOURmbDRyL0U5V2hpWW1FMXNZTXcrUnlZZW85c0tmczNPZnVwaVFUT2lwZ001MC9FcE82Q3BaMApKNUpMc2Q4MkMrNmVzVm9iVFJrSUFYb1pPODFOTGNCaklCTHlKWGhXbnlPWFhFVEMvdlcrQ3htMzRSR2lBYUJoCmJFUHl0MnFLUjRXMFhPb3dkYVYzNngreWVBdXQ4WHozcDVMUzFUbXhCT01BcGZaUEFvSUJBUURVTXRLeVMvOUgKa2pnbml1ZWZhejQvZHU5U2JkUHY1c2JaRlBBdVkxVmRycS8reW0rK2RxL285a3Y4b2tQMjMvUVZOZSs4OEpOQQpJTEwrRDIvaEtWazFPU05mZXF0eUhMZjlkOUlBUG5KY2YrY3NCUWZYZXM4QVV6SW1ja2txOElvMk5WQjNPeThPCjYraFh0WnhtQ1BnMXI1YzZXaSt2TUtJeWxGTmN0YllJYmZqeDFIckk3dE13dnpiWWVGbnNPNHh5WEVtc1Z1RVAKZFh3T3VlVlNHNHM1T0lLUVlQNzZ3Ym5FSldFaXIrTUFWcGNPTk02Ry9iYTZ4c0gxRE1ESFNlaFh1V0xYV3h2UQp5OENRa3NtRWFWTGxIQWttVDA5NEVqMzhDZ2cyaWNLbXA1eWhCYW1RR0dPY281Q0ZYck1aN3lFSUI0VXNGeDAzCjE5NjVVVWJtQXRqL0FvSUJBQmgzLzMxNmxINkh3RmE5OGNaRkU0YjVnamZBaFRpVzdGcngzdHRnY3R3RG5ieG4KaVU5YXh0KzNGQUxjTUVZRzhTeUdxc1VaelloSVVVcXRwSEpzT0tmQllHRm1hMzFUMEV6ZlVrc3ZKd1R4MUhVaAp6U1JPNzMyUDJPSWkrZVdXK1ZlQ2w3WTJFWkp3QTRmK3BmZDZ2cGVJMkd6a2JHRG1maXRSSGZsTkwxaysrbE9qCmw1eFNIREtsL3l5dlBtdGpjc2NxbjhaZjFFcXZtZDJvVHNDaGdwVmxrWXZzNS9iSm1wWndKc1pDanQ2T2FWOTcKU0d3Nm1FaVVOdWUxcm1jSXZpTC9iNXNPU3BxWjZFMWk4U1Y2cVh1MlUzTDZqM2NYZXIySHlIREpodmZhUVNUNwpIS0VYbEl1WjhEbnNPMSt6OWdsc2hCbzZpL1F3aGdZdjVlblUwTVVDZ2dFQUthZWtacTIyZWhWQkFyb294OG1rCm1ZNitZaDcvS0t2VHd3OXlLcGtEUy9XYnhOZDJZaHdvWWdIZlhzTjN0Z0cyaDJka0hXSFlkaEUxTWR4VDZRNVYKM2JYT3ROSit6MUxGNTNMYS9ZTkVyZFhKeW9GZVRiVms1enB0c05Ca2ZwSmpmMHF0OWduZkxmMnZTWEIwT0M5dwpraEZiRFhCZ3hmSTFGTnE3Rm9yeEpleDRudmhIOWlPenYzRDUwanFsNUZLNE9rYlZpNGd1ZS90akUvejRBRXM1CjVFeXNqSzBOd0tuQXpybEx2U1JyMmtnbE5QdTJ3eGNSQk04NWllSXNBYk1IY2hrSlJ4OHljYVZkc3NPdDNWbFQKbFhnUWI3M0g2dGtoNDlVUVVheHZVb202bkgxaEVORkkxSm5qSjlzMEsxWWUxTld0RmIrZjA3T0RuRHRSUUp1MApmd0tDQVFCaHNXcm9odFkwWW1HaDBNQzkzRUlPTmhtNHRkbDNoQ09UZ1VPeWY5MkJVcno4Z3N5MENOT0xDdUN5CldGbmJUTE1ETHBWRDlHY0xPY2ErQmZqd0lWQ2dadDZxYkdhcjFQcFRYZWRjWllxYkhWejY4SVZSUnlzU3RhaGEKMmI5SnYwclgyQ2dsenRIb01UZk9tMExJeGlzRkxFZHJ1Ty9qNHRudWVDUjdVdGlHZmpwTWhMWFBQY24zNFdZTAp5TG5GWWh3U3VWTG1VSjZ6b2F2WUUzQUU4MVR2M2VpRUtOU0JxbzBPZEQ0Q041c3lrNGpnS2lvTC8ra3JBbncyCi9BN1VYUjZwQ3FNS0dlVm1EU3NsSU1hMmw5am00L1AxbldVRVFWNzlZRXhYZW5icGpLWkY4Ly9VNHhKMklUWHQKQ3pDUThOOTk2V0RPb3JKUzFudExlMStYb1hxcAotLS0tLUVORCBQUklWQVRFIEtFWS0tLS0tCg==
  kind: Secret
  metadata:
	creationTimestamp: "2024-04-02T15:29:35Z"
	name: my-app-secret-tls
	namespace: default
	resourceVersion: "134370"
	uid: 7cb29c78-9d87-409a-bb0b-0fc1d4cc91ae
  type: kubernetes.io/tls
kind: List
metadata:
  resourceVersion: ""
root@baranov:/home/baranovsa/kube_2.3#

```

```
root@baranov:/home/baranovsa/kube_2.3# kubectl apply -f myservice.yaml
service/myservice unchanged
root@baranov:/home/baranovsa/kube_2.3# kubectl get svc
NAME     	TYPE    	CLUSTER-IP   	EXTERNAL-IP     PORT(S)    	AGE
kubernetes      ClusterIP       10.96.0.1    	<none>    	443/TCP    	33d
myservice	NodePort	10.105.150.123  <none>    	80:32000/TCP   124m
root@baranov:/home/baranovsa/kube_2.3#
root@baranov:/home/baranovsa/kube_2.3# kubectl get pods
NAME                              	READY   STATUS	RESTARTS     	AGE
myapp-pod-69db4549f5-dkz95        	1/1 	Running   0            	97m
root@baranov:/home/baranovsa/kube_2.3# kubectl get deployment
NAME    	READY   UP-TO-DATE   AVAILABLE  AGE
myapp-pod       1/1 	1        	1       3h2m
root@baranov:/home/baranovsa/kube_2.3#
root@baranov:/home/baranovsa/kube_2.3# kubectl get ingress
NAME         CLASS   HOSTS        ADDRESS    	 PORTS 	   AGE
my-ingress   nginx   my-app.com   192.168.49.2   80, 443   67m
root@baranov:/home/baranovsa/kube_2.3# kubectl get node
NAME   	   STATUS   ROLES       	AGE   VERSION
minikube   Ready    control-plane       33d   v1.28.3
root@baranov:/home/baranovsa/kube_2.3#
root@baranov:/home/baranovsa/kube_2.3# kubectl get cm
NAME               	DATA   AGE
index-html-configmap   1  	3h4m
kube-root-ca.crt   	1  	33d
root@baranov:/home/baranovsa/kube_2.3#
```

Прописываем днс my-app.com в файл hosts и проверяем доступность измененной страницы.

![monitoring](https://github.com/12sergey12/12.8-Kubernetes_Application-Configuration/blob/main/images/12.8_etc_hosts.png)


```
root@baranov:/home/baranovsa/kube_2.3# curl -k https://my-app.com
<html>
<h1>Welcome</h1>
</br>
<h1>Hi! This is a configmap Index file </h1>
</html
root@baranov:/home/baranovsa/kube_2.3#

```


5. Предоставить манифесты, а также скриншоты или вывод необходимых команд.

Прописываем днс my-app.com в файл hosts и проверяем доступность измененной страницы.

[Deployment](https://github.com/12sergey12/12.8-Kubernetes_Application-Configuration/blob/main/ConfigMapDep2.yaml)

[configmap](https://github.com/12sergey12/12.8-Kubernetes_Application-Configuration/blob/main/index-html-configmap.yaml)


![monitoring](https://github.com/12sergey12/12.8-Kubernetes_Application-Configuration/blob/main/images/12.8_etc_hosts.png)

![monitoring](https://github.com/12sergey12/12.8-Kubernetes_Application-Configuration/blob/main/images/12.8_kube_2.3_2.1.png)

![monitoring](https://github.com/12sergey12/12.8-Kubernetes_Application-Configuration/blob/main/images/12.8_kube_2.3_2.2.png)

![monitoring](https://github.com/12sergey12/12.8-Kubernetes_Application-Configuration/blob/main/images/12.8_kube_2.3_2.3.png)

```
root@baranov:/home/baranovsa/kube_2.3# curl -k https://my-app.com
<html>
<h1>Welcome</h1>
</br>
<h1>Hi! This is a configmap Index file </h1>
</html
root@baranov:/home/baranovsa/kube_2.3#
```

------

### Правила приёма работы

1. Домашняя работа оформляется в своём GitHub-репозитории в файле README.md. Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.
2. Файл README.md должен содержать скриншоты вывода необходимых команд `kubectl`, а также скриншоты результатов.
3. Репозиторий должен содержать тексты манифестов или ссылки на них в файле README.md.

------
