Etherpad example kompose
========================
`kompose` is a tool to help users familiar with `docker-compose` move to [Kubernetes](http://kubernetes.io). It takes a Docker Compose file and translates it into Kubernetes resources.

￼`kompose` is a convenience tool to go from local Docker development to managing your application with Kubernetes. We don't assume that the transformation from docker compose format to Kubernetes API objects will be perfect, but it helps tremendously to start _Kubernetizing_ your application.

In example we will crate Openshift and Kubernetes artifacts for etherpad app from the docker-compose file using `kompose`.

Creating openshift artifacts
----------------------------
`kompose --provider=openshift convert`

It will create the service.json, deploymentcongif.json and imagestream.json files for openshift.

```
INFO[0000] file "etherpad-service.json" created       
INFO[0000] file "mariadb-service.json" created          
INFO[0000] file "etherpad-deploymentconfig.json" created
INFO[0000] file "etherpad-imagestream.json" created
INFO[0000] file "mariadb-deploymentconfig.json" created
INFO[0000] file "mariadb-imagestream.json" created
```
The deploymentcongif file created by kompose using only 25 lines of docker-compose file.
Here you can the edit the files if you want to make some adddional changes.This is how This is how 

```
{
  "kind": "DeploymentConfig",
  "apiVersion": "v1",
  "metadata": {
    "name": "etherpad",
    "creationTimestamp": null,
    "labels": {
      "service": "etherpad"
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
            "etherpad"
          ],
          "from": {
            "kind": "ImageStreamTag",
            "name": "etherpad:latest"
          }
        }
      }
    ],
    "replicas": 1,
    "test": false,
    "selector": {
      "service": "etherpad"
    },
    "template": {
      "metadata": {
        "creationTimestamp": null,
        "labels": {
          "service": "etherpad"
        }
      },
      "spec": {
        "containers": [
          {
            "name": "etherpad",
            "image": " ",
            "ports": [
              {
                "containerPort": 9001,
                "protocol": "TCP"
              }
            ],
            "env": [
              {
                "name": "DB_DBID",
                "value": "etherpad"
              },
              {
                "name": "DB_HOST",
                "value": "etherpad"
              },
              {
                "name": "DB_PASS",
                "value": "etherpad"
              },
              {
                "name": "DB_PORT",
                "value": "3306"
              },
              {
                "name": "DB_USER",
                "value": "etherpad"
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

```

Deploy the application on Openshift.
------------------------------------
`kubectl create -f <path/to/the/artifacts>.`

That it! Your application has been deployed on Openshift in just 2 simple steps.

There is one more way to deploy your aplication directly on Openshift if you do not want to inspect the artifacts created by `kompose`

`kompose up --provider=openshift`

Your application has been deployed on Openshift using one single command.
