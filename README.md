# my-deploymentstrategy--blue-and-green-deployment

Alias k=kubectl

delete all the deployment, services, ingress
--------------------------------------------
k delete -f .

Blue Green Deployment
---------------------
vi deployment-blue.yml
----------------------
apiVersion: apps/v1
kind: Deployment
metadata:
  name: springboot-deployment-blue
  labels:
    app: springboot
spec:
  replicas: 2
  selector:
    matchLabels:
      app: springboot
      version: v1
  template:
    metadata:
      labels:
        app: springboot
        version: v1
    spec:
      containers:
      - name: springboot-blue
        image: sarath750/springboot-hello:v1
        ports:
        - containerPort: 8080

k apply -f deployment-blue.yml

vi service-blue.yml
-------------------
---
apiVersion: v1
kind: Service
metadata:
  name: springboot-service-blue
spec:
  type: LoadBalancer
  selector:
    app: springboot
    version: v1
  ports:
    - protocol: TCP
      port: 8080
      targetPort: 8080

k apply -f service-blue.yml

k get pods

k get service

<DNS name:8080>

Greetings from Springboot..!!!


create ingress as well
----------------------
vi ingress.yml
--------------
---
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: springboot-ingress
  annotations:
    kubernetes.io/ingress.class: alb
    alb.ingress.kubernetes.io/scheme: internet-facing
spec:
  rules:
  - http:
      paths:
      - path: /
        backend:
          serviceName: springboot-service-blue
          servicePort:  8080

k apply -f ingress.yml

Now this ingress will create alb. we can check in Loadbalancer section.

k get ingress.
our ingress will be running with 80 PN.

<DNS>

Greetings from Springboot..!!!




vi deployment-green.yml
-----------------------
apiVersion: apps/v1
kind: Deployment
metadata:
  name: springboot-deployment-green
  labels:
    app: springboot
spec:
  replicas: 2
  selector:
    matchLabels:
      app: springboot
      version: v2
  template:
    metadata:
      labels:
        app: springboot
        version: v2
    spec:
      containers:
      - name: springboot-blue
        image: sarath750/springboot-hello:v2
        ports:
        - containerPort: 8080

k apply -f deployment-green.yml
 

vi service-green.yml
--------------------
---
apiVersion: v1
kind: Service
metadata:
  name: springboot-service-green
spec:
  type: LoadBalancer
  selector:
    app: springboot
    version: v2
  ports:
    - protocol: TCP
      port: 8080
      targetPort: 8080

k apply -f service-green.yml

k get service/k get svc

<DNS:8080>

Greetings from Springboot123..!!!

k get pods

once this green deployment is working fine, we can attach this green deployment to ingress and remove the blue deployment from ingress.


create ingress as well
----------------------
vi ingress.yml
--------------
---
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: springboot-ingress
  annotations:
    kubernetes.io/ingress.class: alb
    alb.ingress.kubernetes.io/scheme: internet-facing
spec:
  rules:
  - http:
      paths:
      - path: /
        backend:
          serviceName: springboot-service-green
          servicePort:  8080

k apply -f ingress.yml

k get ingress

<DNS>


Finally our deployment has been changed to green from blue. if there is any issues with the green deployment then again we need to rollback to the blue deployment. 

for rolling back to blue, just change the servicename in the ingress.yml file. here we will be moving 100% traffic to green at once, but if we want to move traffic slowly to green like 10%, 20% ..then we need to use canary deployment.
