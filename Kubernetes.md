# Guestbook example kompose
`kompose` is a tool to help users, familiar with `docker-compose`, move to [Kubernetes](http://kubernetes.io). It takes a Docker Compose file and translates it into Kubernetes resources.

`kompose` is a convenience tool to go from local Docker development to managing your application with Kubernetes. We don't assume that the transformation from docker compose format to Kubernetes API objects will be perfect, but it helps tremendously to start _Kubernetizing_ your application.

In this example we will create Kubernetes artifacts for guestbook app from the docker-compose file using `kompose`.

## Creating Kubenetes artifacts.
This step will create the service.json, deployment.json files for Kubernetes.

`$ kompose --provider=kubernetes convert`
```bash
INFO[0000] file "frontend-service.json" created         
INFO[0000] file "redis-master-service.json" created     
INFO[0000] file "redis-slave-service.json" created      
INFO[0000] file "frontend-deployment.json" created      
INFO[0000] file "redis-master-deployment.json" created  
INFO[0000] file "redis-slave-deployment.json" created   
```

Once generated, you can inspect the files and edit these files if you want to make some additional changes as per your requirement.

```json
{
  "kind": "Deployment",
  "apiVersion": "extensions/v1beta1",
  "metadata": {
    "name": "redis-master",
    "creationTimestamp": null
  },
  "spec": {
    "replicas": 1,
    "template": {
      "metadata": {
        "creationTimestamp": null,
        "labels": {
          "service": "redis-master"
        }
      },
      "spec": {
        "containers": [
          {
            "name": "redis-master",
            "image": "gcr.io/google_containers/redis:e2e",
            "ports": [
              {
                "containerPort": 6379,
                "protocol": "TCP"
              }
            ],
            "resources": {}
          }
        ],
        "restartPolicy": "Always"
      }
    },
    "strategy": {}
  },
  "status": {}
}
```

If you prefer `YAML` files instead of `JSON`, you can do that using the `-y` option like so :

`$ kompose --provider=kubernetes convert -y`
```bash
INFO[0000] file "frontend-service.yaml" created         
INFO[0000] file "redis-master-service.yaml" created     
INFO[0000] file "redis-slave-service.yaml" created      
INFO[0000] file "frontend-deployment.yaml" created      
INFO[0000] file "redis-master-deployment.yaml" created  
INFO[0000] file "redis-slave-deployment.yaml" created
```

You can inspect the file and make some additional changes, if required.
```yaml
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  labels:
    service: redis-master
  name: redis-master
spec:
  ports:
  - name: "6379"
    port: 6379
    protocol: TCP
    targetPort: 6379
  selector:
    service: redis-master
status:
  loadBalancer: {}
```

## Deploy the application on Kubernetes.
### There are two ways in which you can deploy the application on Kubernetes.
#### 1) Using Kubernetes cli.

`$ kubectl create -f <path/to/artifacts>`

Parse a configuration file and create one or more Kubernetes objects based on the file contents.

```bash
deployment "frontend" created
service "frontend" created
deployment "redis-master" created
service "redis-master" created
deployment "redis-slave" created
service "redis-slave" created
```

#### View the Services and Deployment on Kubernetes.

```bash
$ kubectl get svc

NAME           CLUSTER-IP   EXTERNAL-IP   PORT(S)    AGE
frontend       10.0.0.28    <none>        80/TCP     1m
kubernetes     10.0.0.1     <none>        443/TCP    3m
redis-master   10.0.0.246   <none>        6379/TCP   1m
redis-slave    10.0.0.26    <none>        6379/TCP   1m
```


```bash
$ kubectl get deployment 

NAME           DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
frontend       1         1         1            0           2m
redis-master   1         1         1            0           2m
redis-slave    1         1         1            0           2m
```

#### Next, take a look at the pods created by the deployments.

```bash
$ kubectl get pods

NAME                   READY     STATUS    RESTARTS   AGE
frontend-1-ojsux       1/1       Running   0          4m
redis-master-1-1ep72   1/1       Running   0          4m
redis-slave-2504961    1/1       Running   0          4m
```

#### Verify the application.

```bash
$ curl <frontend-ip>:80

<html ng-app="redis">
  <head>
    <title>Guestbook</title>
    <link rel="stylesheet" href="//netdna.bootstrapcdn.com/bootstrap/3.1.1/css/bootstrap.min.css">
    <script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.2.12/angular.min.js"></script>
    <script src="controllers.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/angular-ui-bootstrap/0.13.0/ui-bootstrap-tpls.js"></script>
  </head>
  <body ng-controller="RedisCtrl">
    <div style="width: 50%; margin-left: 20px">
      <h2>Guestbook</h2>
    <form>
    <fieldset>
    <input ng-model="msg" placeholder="Messages" class="form-control" type="text" name="input"><br>
    <button type="button" class="btn btn-primary" ng-click="controller.onRedis()">Submit</button>
    </fieldset>
    </form>
    <div>
      <div ng-repeat="msg in messages track by $index">
        {{msg}}
      </div>
    </div>
    </div>
  </body>
</html>

```

That's it! Your application has been deployed on Kubernetes in just 2 simple steps.

Asciinema for the above steps.
[![asciicast](https://asciinema.org/a/93cw0sd1i3rxcia9qjjccr5lp.png)](https://asciinema.org/a/93cw0sd1i3rxcia9qjjccr5lp)

### 2) Using kompose cli
There is one more way to deploy your application directly on Kubernetes if you do not want to inspect the artifacts created by kompose. Hence you can completely skip step 1 mentioned in the previous method mentioned above

`$ kompose --provider kubernetes up`
This will crate the Kubernetes artifacts and deploy them at the same time.

```bash
We are going to create Kubernetes Deployments, Services and PersistentVolumeClaims for your Dockerized application. 
If you need different kind of resources, use the 'kompose convert' and 'kubectl create -f' commands instead. 

INFO[0000] Successfully created service: redis-master   
INFO[0000] Successfully created service: redis-slave    
INFO[0000] Successfully created service: frontend       
INFO[0000] Successfully created deployment: redis-master 
INFO[0000] Successfully created deployment: redis-slave 
INFO[0000] Successfully created deployment: frontend

Your application has been deployed to Kubernetes. You can run 'kubectl get deployment,svc,pod,pvc' for details.
```

#### View the Services and Deployment on Kubernetes.

```bash
$ kubectl get svc
NAME           CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
frontend       10.254.60.179   <none>        80/TCP     3m
kubernetes     10.254.0.1      <none>        443/TCP    1h
redis-master   10.254.113.10   <none>        6379/TCP   3m
redis-slave    10.254.77.181   <none>        6379/TCP   3m
```

```bash
$ kubectl get deployment 
NAME           DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
frontend       1         1         1            1           4m
redis-master   1         1         1            1           4m
redis-slave    1         1         1            1           4m
```

#### Next, take a look at the pods created by the deployments.

```bash
$ kubectl get pods
NAME                            READY     STATUS    RESTARTS   AGE
frontend-2768218532-1llhz       1/1       Running   0          5m
redis-master-1432129712-nb6ie   1/1       Running   0          5m
redis-slave-2504961300-8fjuy    1/1       Running   0          5m
```

#### Verify the application.

```bash
curl <frontend-ip>:80
<html ng-app="redis">
  <head>
    <title>Guestbook</title>
    <link rel="stylesheet" href="//netdna.bootstrapcdn.com/bootstrap/3.1.1/css/bootstrap.min.css">
    <script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.2.12/angular.min.js"></script>
    <script src="controllers.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/angular-ui-bootstrap/0.13.0/ui-bootstrap-tpls.js"></script>
  </head>
  <body ng-controller="RedisCtrl">
    <div style="width: 50%; margin-left: 20px">
      <h2>Guestbook</h2>
    <form>
    <fieldset>
    <input ng-model="msg" placeholder="Messages" class="form-control" type="text" name="input"><br>
    <button type="button" class="btn btn-primary" ng-click="controller.onRedis()">Submit</button>
    </fieldset>
    </form>
    <div>
      <div ng-repeat="msg in messages track by $index">
        {{msg}}
      </div>
    </div>
    </div>
  </body>
</html>
```

Your application has been deployed on Kubernetes using one single command.

Asciinema of the above steps.
[![asciicast](https://asciinema.org/a/8ympbd3fr3s9zyxb70ticz54w.png)](https://asciinema.org/a/8ympbd3fr3s9zyxb70ticz54w)

## More
`kompose` can do more. Feel free to explore it [here](https://github.com/kubernetes-incubator/kompose).

`kompose --help` to find out more options.
Use `kompose convert --help` to find more about the convert sub command.