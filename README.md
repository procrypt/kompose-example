# Guestbook example kompose
`kompose` is a tool to help users familiar with `docker-compose` move to [Kubernetes](http://kubernetes.io). It takes a Docker Compose file and translates it into Kubernetes resources.

￼`kompose` is a convenience tool to go from local Docker development to managing your application with Kubernetes. We don't assume that the transformation from docker compose format to Kubernetes API objects will be perfect, but it helps tremendously to start _Kubernetizing_ your application.

In example we will create OpenShift artifacts for guestbook app from the docker-compose file using `kompose`.

## Creating OpenShift artifacts.
`$ kompose --provider=openshift convert`

This step will create the service.json, deploymentconfig.json and imagestream.json files for OpenShift.

```
INFO[0000] file "frontend-service.json" created         
INFO[0000] file "redis-master-service.json" created     
INFO[0000] file "redis-slave-service.json" created      
INFO[0000] file "frontend-deploymentconfig.json" created 
INFO[0000] file "frontend-imagestream.json" created     
INFO[0000] file "redis-master-deploymentconfig.json" created 
INFO[0000] file "redis-master-imagestream.json" created 
INFO[0000] file "redis-slave-deploymentconfig.json" created 
INFO[0000] file "redis-slave-imagestream.json" created  
```
Once generated, you can inspect the files and edit these files if you want to make some adddional changes as per your requirement.


```
{
  "kind": "DeploymentConfig",
  "apiVersion": "v1",
  "metadata": {
    "name": "redis-master",
    "creationTimestamp": null,
    "labels": {
      "service": "redis-master"
    }
  },
  "spec": {
    "strategy": {
      "resources": {}
    },
    "triggers": [
      {
        "type": "ConfigChange"
      },
      {
        "type": "ImageChange",
        "imageChangeParams": {
          "automatic": true,
          "containerNames": [
            "redis-master"
          ],
          "from": {
            "kind": "ImageStreamTag",
            "name": "redis-master:e2e"
          }
        }
      }
    ],
    "replicas": 1,
    "test": false,
    "selector": {
      "service": "redis-master"
    },
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
            "image": " ",
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
    }
  },
  "status": {}
}
```

If you prefer YAML files instead of JSON, you can do that using the `-y` option like so :

`$ kompose --provider=openshift convert -y`


## Deploy the application on OpenShift.

### a) Using OpenShift cli.
`$ oc create -f <path/to/artifacts>`

```
deploymentconfig "frontend" created
imagestream "frontend" created
service "frontend" created
deploymentconfig "redis-master" created
imagestream "redis-master" created
service "redis-master" created
deploymentconfig "redis-slave" created
imagestream "redis-slave" created
service "redis-slave" created
```
#### View the Services and Deploymentconfig on OpenShift.
`$ oc get svc`

```
NAME           CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
frontend       172.30.99.116   <none>        80/TCP     2m
redis-master   172.30.94.229   <none>        6379/TCP   2m
redis-slave    172.30.43.20    <none>        6379/TCP   2m
```
`$ oc get dc`

```
NAME           REVISION   REPLICAS   TRIGGERED BY
frontend       1          1          config,image(frontend:v4)
redis-master   1          1          config,image(redis-master:e2e)
redis-slave    0          1          config,image(redis-slave:v1)
``` 
#### Next, take a look at the pods created by the deployments.
`$ oc get pods`

```
NAME                   READY     STATUS    RESTARTS   AGE
frontend-1-ojsux       1/1       Running   0          4m
redis-master-1-1ep72   1/1       Running   0          4m

```

#### Verify the application.

`$ curl <frontend-ip>:80`

```
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

That's it! Your application has been deployed on OpenShift in just 2 simple steps.

Asciinema for the above steps.
[![asciicast](https://asciinema.org/a/5snir7l4ccvcstgtugll7z41e.png)](https://asciinema.org/a/5snir7l4ccvcstgtugll7z41e)

### b) Using kompose cli
There is one more way to deploy your aplication directly on OpenShift if you do not want to inspect the artifacts created by kompose. Hence you can completely skip step 1.

`$ kompose --provider openshift up`

```
We are going to create OpenShift DeploymentConfigs and Services for your Dockerized application. 
If you need different kind of resources, use the 'kompose convert' and 'oc create -f' commands instead. 

INFO[0000] Successfully created service: frontend       
INFO[0000] Successfully created service: redis-master   
INFO[0000] Successfully created service: redis-slave    
INFO[0000] Successfully created deployment: frontend    
INFO[0000] Successfully created ImageStream: frontend   
INFO[0000] Successfully created deployment: redis-master 
INFO[0000] Successfully created ImageStream: redis-master 
INFO[0000] Successfully created deployment: redis-slave 
INFO[0000] Successfully created ImageStream: redis-slave 

Your application has been deployed to OpenShift. You can run 'oc get dc,svc,is' for details.
```
#### View the Services, Deploymentconfig, ImageStream and Pods created on OpenShift.

`$ oc get svc`

```
NAME           CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
frontend       172.30.152.108   <none>        80/TCP     3m
redis-master   172.30.31.110    <none>        6379/TCP   3m
redis-slave    172.30.220.107   <none>        6379/TCP   3m
```

`$ oc get dc`

```
NAME           REVISION   REPLICAS   TRIGGERED BY
frontend       1          1          config,image(frontend:v4)
redis-master   1          1          config,image(redis-master:e2e)
redis-slave    0          1          config,image(redis-slave:v1)
```

`$ oc get is`

```
NAME           DOCKER REPO                                 TAGS      UPDATED
frontend       172.30.78.228:5000/guestbook/frontend       v4        5 minutes ago
redis-master   172.30.78.228:5000/guestbook/redis-master   e2e       5 minutes ago
redis-slave    172.30.78.228:5000/guestbook/redis-slave    v1        
```

`$ oc get pods`

```
NAME                   READY     STATUS    RESTARTS   AGE
frontend-1-3dx8l       1/1       Running   0          3m
redis-master-1-skbbu   1/1       Running   0          3m
```
#### Verify the application.
`$ curl <frontend-ip>:80`

```
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

Your application has been deployed on OpenShift using one single command.

Asciinema of the above steps.
[![asciicast](https://asciinema.org/a/7z2ispckpi8gxi4r1xv8xs08i.png)](https://asciinema.org/a/7z2ispckpi8gxi4r1xv8xs08i)


## More
`kompose` can do more. Feel free to explore it.

`kompose --help` - To find out more general options
`kompose convert --help`- To find out more about `convert` sub command.
