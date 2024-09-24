# kubernetes-advanced

->Create Helm charts for a sample application   

``` helm create my-web-app```

This will generate a directory structure like this:  
my-web-app/  
    Chart.yaml  
    values.yaml  
    charts/  
    templates/  

In **Chart.yml** Update metadata about your application.  

In **values.yml** Define configurable values for your application.
``` replicaCount: 2

image:
  repository: nginx
  tag: latest
  pullPolicy: IfNotPresent

service:
  type: ClusterIP
  port: 80

resources: {}
```

In **templates/deployment.yaml** update the deployment to use the specified values  

```apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}-web
  labels:
    app: {{ .Release.Name }}-web
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app: {{ .Release.Name }}-web
  template:
    metadata:
      labels:
        app: {{ .Release.Name }}-web
    spec:
      containers:
        - name: web
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          ports:
            - containerPort: 80
```

In **templates/service.yaml** update the service to expose the application  
```
apiVersion: v1
kind: Service
metadata:
  name: {{ .Release.Name }}-web
spec:
  type: {{ .Values.service.type }}
  ports:
    - port: {{ .Values.service.port }}
      targetPort: 80
  selector:
    app: {{ .Release.Name }}-web
```

Now, the Helm chart is successfully created with configurable values.  

-> Use the Helm charts to deploy the application to a Kubernetes cluster.  
Here we have created a new chart to deploy a html application.  

```helm create my-app```

**Charts.yaml**  
```
apiVersion: v2
name: my-app
description: A Helm chart for Kubernetes
```

**Deployment.yaml**  
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app: {{ .Release.Name }}
  template:
    metadata:
      labels:
        app: {{ .Release.Name }}
    spec:
      containers:
      - name: hello-world-html
        image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
        imagePullPolicy: Never
        ports:
        - containerPort: 80

```

**Service.yaml**
```
apiVersion: v1
kind: Service
metadata:
  name: {{ .Release.Name }}
spec:
  type: {{ .Values.service.type }}
  ports:
  - port: {{ .Values.service.port }}
    targetPort: 80
  selector:
    app: {{ .Release.Name }}

```

**Values.yaml**
```
replicaCount: 3

image:
  repository: hello-world-html
  pullPolicy: IfNotPresent
  tag: latest
service:
  type: ClusterIP
  port: 80

```

To deploy the application use:  
```
helm install hello-world-html ./my-app
```

To upgrade it we can change the values in the Values.yml file and the run the below command  
```
helm upgrade hello-world-html ./my-app
```

-> •	Implement a namespace strategy for your Kubernetes cluster and deploying the application in 3 different namespaces i.e dev,stage,prod  
```
kubectl create namespace dev
kubectl create namespace stage
kubectl create namespace prod
```

To deploy your application to a specific namespace use (eg. dev)  
```
helm install hello-world-html ./my-app  --namespace dev
```

Now to verify the deployments use  
```
kubectl get pods -n dev
 ```

-> Now to upgrade the application lets change the replicas count to 2 in values.yml and upgrade it in the dev namespace  
```
helm upgrade hello-world-html ./my-app  --namespace dev
 ```

Now if you check the no of pods in the dev namespace it will be 2  

-> To rollback the application to a previous version  
```
helm rollback hello-world-html --namespace dev
```

Now if you again check the no of pods in the dev namespace it will be 3  


************* **END of document**   ***********

