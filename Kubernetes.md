# Guestbook example kompose

README ^^ should be removed. Copy-pasta'd from the Kompose README.

In this example we will create Kubernetes artifacts for guestbook app from the docker-compose file using `kompose`.

What is the guestbook app? Can you explain this a bit more?

## Creating Kubenetes artifacts.
This step will create the `service.json` and `deployment.json` files for Kubernetes.

Errr... Shouldn't all of this be `yaml` NOT json? It's more common now-a-days to use yaml for the examples instead

`$ kompose --provider=kubernetes convert`

I don't think you need to provide `--provider=kubernetes`...

```bash
INFO[0000] file "frontend-service.json" created         
INFO[0000] file "redis-master-service.json" created     
INFO[0000] file "redis-slave-service.json" created      
INFO[0000] file "frontend-deployment.json" created      
INFO[0000] file "redis-master-deployment.json" created  
INFO[0000] file "redis-slave-deployment.json" created   
```

Once generated, you can inspect and edit these files if you prefer to make additional changes.

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

Nvm! I see why you included JSON up above! Maybe add this `-y` notion instead at the beginning of the example?

`$ kompose --provider=kubernetes convert -y`
```bash
INFO[0000] file "frontend-service.yaml" created         
INFO[0000] file "redis-master-service.yaml" created     
INFO[0000] file "redis-slave-service.yaml" created      
INFO[0000] file "frontend-deployment.yaml" created      
INFO[0000] file "redis-master-deployment.yaml" created  
INFO[0000] file "redis-slave-deployment.yaml" created
```

No need to have "inspect the file and make some additional changes sentence here as it's implied by the previous example.

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
### There are two ways to deploy your application
#### 1) Using `kubectl`

`$ kubectl create -f <path/to/artifacts>`

This command won't actually work, -R needs to be passed (recursive). Use `kubectl create -R -f <path/to/artifacts>` unless I'm mistaken.

```bash
deployment "frontend" created
service "frontend" created
deployment "redis-master" created
service "redis-master" created
deployment "redis-slave" created
service "redis-slave" created
```

#### View the deployment and services

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

#### Verify the pods are running

```bash
$ kubectl get pods
NAME                   READY     STATUS    RESTARTS   AGE
frontend-1-ojsux       1/1       Running   0          4m
redis-master-1-1ep72   1/1       Running   0          4m
redis-slave-2504961    1/1       Running   0          4m
```

#### Ping the application

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

Specify the steps please :) So... the 1st one would be `kompose convert` and the 2nd would be `kubectl -f foobar`, etc.


#### Asciicast example

[![asciicast](https://asciinema.org/a/93cw0sd1i3rxcia9qjjccr5lp.png)](https://asciinema.org/a/93cw0sd1i3rxcia9qjjccr5lp)

### 2) Using kompose cli
There is one more way to deploy your application directly on Kubernetes if you do not want to inspect the artifacts created by kompose. Hence you can completely skip step 1 mentioned in the previous method mentioned above

This feels out of place ^^ maybe add this to the "step 2" example up above?

`$ kompose --provider kubernetes up`

It will crate the Kubernetes artifacts and deploy them at the same time.

```bash
We are going to create Kubernetes Deployments, Services and PersistentVolumeClaims for your Dockerized application. 

What if the user doesn't have a persistent volume? Isn't there a recent option for this that has been merged into Kompose?

If you need different kind of resources, use the 'kompose convert' and 'kubectl create -f' commands instead. 

INFO[0000] Successfully created service: redis-master   
INFO[0000] Successfully created service: redis-slave    
INFO[0000] Successfully created service: frontend       
INFO[0000] Successfully created deployment: redis-master 
INFO[0000] Successfully created deployment: redis-slave 
INFO[0000] Successfully created deployment: frontend

Your application has been deployed to Kubernetes. You can run 'kubectl get deployment,svc,pod,pvc' to verify your application is running.
```

No need for all this here. Feels like a copy/paste. It'd be preferably to have something straight-to-the-point for the example as all this bash / cat output may be a bit too much for a first-time user (since this is a "helloworld" example we're using).


## More
If you'd like to find out more about `kompose`, feel free to explore the Github repo at [kubernetes-incubator/kompose](https://github.com/kubernetes-incubator/kompose)

`kompose --help` to find out more options.

Use `kompose convert --help` to find more about the convert sub command.
