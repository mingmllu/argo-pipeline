# argo-pipeline

Argo project github: https://github.com/argoproj/argo

Argo project website: https://argoproj.github.io/

## Installation on Ubuntu 16.04

### Requirements

* Installed Kubernetes 1.9 or later
* Installed kubectl
* Have a kubeconfig file (default location is ~/.kube/config).

### Download Argo

```
$ sudo curl -sSL -o /usr/local/bin/argo https://github.com/argoproj/argo/releases/download/v2.2.1/argo-linux-amd64
$ sudo chmod +x /usr/local/bin/argo
```
### Install the Controller and UI

```
$ kubectl create ns argo
$ kubectl apply -n argo -f https://raw.githubusercontent.com/argoproj/argo/v2.2.1/manifests/install.yaml
```
### Configure the service account to run workflows

For demo purposes, run the following command to grant admin privileges to the 'default' service account in the namespace 'default':
```
$ kubectl create rolebinding default-admin --clusterrole=admin --serviceaccount=default:default
```

## Run Simple Example Workflows

### Example 1

```
$ argo submit --watch https://raw.githubusercontent.com/argoproj/argo/master/examples/hello-world.yaml
```
The yaml file is as below:
```
apiVersion: argoproj.io/v1alpha1
kind: Workflow
metadata:
  generateName: hello-world-
spec:
  entrypoint: whalesay
  templates:
  - name: whalesay
    container:
      image: docker/whalesay:latest
      command: [cowsay]
      args: ["hello world"]
```
List workflows
```
$ argo list
NAME                STATUS      AGE   DURATION
hello-world-2t7jc   Succeeded   12m   2m
```

Display details about the above workflow:
```
$ argo get hello-world-2t7jc
Name:                hello-world-2t7jc
Namespace:           default
ServiceAccount:      default
Status:              Succeeded
Created:             Mon Dec 24 22:33:15 -0500 (13 minutes ago)
Started:             Mon Dec 24 22:33:15 -0500 (13 minutes ago)
Finished:            Mon Dec 24 22:35:34 -0500 (10 minutes ago)
Duration:            2 minutes 19 seconds

STEP                  PODNAME            DURATION  MESSAGE
 ✔ hello-world-2t7jc  hello-world-2t7jc  2m
```

View the logs of the workflow:
```
$ argo logs hello-world-2t7jc
 _____________
< hello world >
 -------------
    \
     \
      \
                    ##        .
              ## ## ##       ==
           ## ## ## ##      ===
       /""""""""""""""""___/ ===
  ~~~ {~~ ~~~~ ~~~ ~~~~ ~~ ~ /  ===- ~~~
       \______ o          __/
        \    \        __/
          \____\______/
```

### Example 2
The yaml file used in this example is as below:
```
# The coinflip example combines the use of a script result,
# along with conditionals, to take a dynamic path in the
# workflow. In this example, depending on the result of the
# first step, 'flip-coin', the template will either run the
# 'heads' step or the 'tails' step.
apiVersion: argoproj.io/v1alpha1
kind: Workflow
metadata:
  generateName: coinflip-
spec:
  entrypoint: coinflip
  templates:
  - name: coinflip
    steps:
    - - name: flip-coin
        template: flip-coin
    - - name: heads
        template: heads
        when: "{{steps.flip-coin.outputs.result}} == heads"
      - name: tails
        template: tails
        when: "{{steps.flip-coin.outputs.result}} == tails"

  - name: flip-coin
    script:
      image: python:alpine3.6
      command: [python]
      source: |
        import random
        result = "heads" if random.randint(0,1) == 0 else "tails"
        print(result)

  - name: heads
    container:
      image: alpine:3.6
      command: [sh, -c]
      args: ["echo \"it was heads\""]

  - name: tails
    container:
      image: alpine:3.6
      command: [sh, -c]
      args: ["echo \"it was tails\""]
```
You can download the yaml file from the URL https://raw.githubusercontent.com/argoproj/argo/master/examples/coinflip.yaml.
```
$ argo submit --watch https://raw.githubusercontent.com/argoproj/argo/master/examples/coinflip.yaml
Name:                coinflip-6kvxp
Namespace:           default
ServiceAccount:      default
Status:              Pending
Created:             Mon Dec 24 23:00:49 -0500 (now)
Name:                coinflip-6kvxp
Namespace:           default
ServiceAccount:      default
Status:              Pending
Created:             Mon Dec 24 23:00:49 -0500 (now)
Name:                coinflip-6kvxp
Namespace:           default
ServiceAccount:      default
Status:              Running
Created:             Mon Dec 24 23:00:49 -0500 (now)
Started:             Mon Dec 24 23:00:49 -0500 (now)
Duration:            0 seconds

STEP               PODNAME                    DURATION  MESSAGE
 ● coinflip-6kvxp
 └---◷ flip-coin   coinflip-6kvxp-2242750624  0s
Name:                coinflip-6kvxp
Namespace:           default
ServiceAccount:      default
Status:              Running
Created:             Mon Dec 24 23:00:49 -0500 (1 second ago)
Started:             Mon Dec 24 23:00:49 -0500 (1 second ago)
Duration:            1 second

STEP               PODNAME                    DURATION  MESSAGE
 ● coinflip-6kvxp
 └---◷ flip-coin   coinflip-6kvxp-2242750624  1s
Name:                coinflip-6kvxp
Namespace:           default
ServiceAccount:      default
Status:              Running
Created:             Mon Dec 24 23:00:49 -0500 (1 second ago)
Started:             Mon Dec 24 23:00:49 -0500 (1 second ago)
Duration:            1 second

STEP               PODNAME                    DURATION  MESSAGE
 ● coinflip-6kvxp
 └---◷ flip-coin   coinflip-6kvxp-2242750624  1s        PodInitializing
Name:                coinflip-6kvxp
Namespace:           default
ServiceAccount:      default
Status:              Running
Created:             Mon Dec 24 23:00:49 -0500 (2 seconds ago)
Started:             Mon Dec 24 23:00:49 -0500 (2 seconds ago)
Duration:            2 seconds

STEP               PODNAME                    DURATION  MESSAGE
 ● coinflip-6kvxp
 └---◷ flip-coin   coinflip-6kvxp-2242750624  2s        PodInitializing
Name:                coinflip-6kvxp
Namespace:           default
ServiceAccount:      default
Status:              Running
Created:             Mon Dec 24 23:00:49 -0500 (3 seconds ago)
Started:             Mon Dec 24 23:00:49 -0500 (3 seconds ago)
Duration:            3 seconds

STEP               PODNAME                    DURATION  MESSAGE
 ● coinflip-6kvxp
 └---◷ flip-coin   coinflip-6kvxp-2242750624  3s        PodInitializing
Name:                coinflip-6kvxp
Namespace:           default
ServiceAccount:      default
Status:              Running
Created:             Mon Dec 24 23:00:49 -0500 (4 seconds ago)
Started:             Mon Dec 24 23:00:49 -0500 (4 seconds ago)
Duration:            4 seconds

STEP               PODNAME                    DURATION  MESSAGE
 ● coinflip-6kvxp
 └---◷ flip-coin   coinflip-6kvxp-2242750624  4s        PodInitializing
Name:                coinflip-6kvxp
Namespace:           default
ServiceAccount:      default
Status:              Running
Created:             Mon Dec 24 23:00:49 -0500 (5 seconds ago)
Started:             Mon Dec 24 23:00:49 -0500 (5 seconds ago)
Duration:            5 seconds

STEP               PODNAME                    DURATION  MESSAGE
 ● coinflip-6kvxp
 └---◷ flip-coin   coinflip-6kvxp-2242750624  5s        PodInitializing
Name:                coinflip-6kvxp
Namespace:           default
ServiceAccount:      default
Status:              Running
Created:             Mon Dec 24 23:00:49 -0500 (6 seconds ago)
Started:             Mon Dec 24 23:00:49 -0500 (6 seconds ago)
Duration:            6 seconds

STEP               PODNAME                    DURATION  MESSAGE
 ● coinflip-6kvxp
 └---◷ flip-coin   coinflip-6kvxp-2242750624  6s        PodInitializing
Name:                coinflip-6kvxp
Namespace:           default
ServiceAccount:      default
Status:              Running
Created:             Mon Dec 24 23:00:49 -0500 (7 seconds ago)
Started:             Mon Dec 24 23:00:49 -0500 (7 seconds ago)
Duration:            7 seconds

STEP               PODNAME                    DURATION  MESSAGE
 ● coinflip-6kvxp
 └---◷ flip-coin   coinflip-6kvxp-2242750624  7s        PodInitializing
Name:                coinflip-6kvxp
Namespace:           default
ServiceAccount:      default
Status:              Running
Created:             Mon Dec 24 23:00:49 -0500 (8 seconds ago)
Started:             Mon Dec 24 23:00:49 -0500 (8 seconds ago)
Duration:            8 seconds

STEP               PODNAME                    DURATION  MESSAGE
 ● coinflip-6kvxp
 └---◷ flip-coin   coinflip-6kvxp-2242750624  8s        PodInitializing
Name:                coinflip-6kvxp
Namespace:           default
ServiceAccount:      default
Status:              Running
Created:             Mon Dec 24 23:00:49 -0500 (9 seconds ago)
Started:             Mon Dec 24 23:00:49 -0500 (9 seconds ago)
Duration:            9 seconds

STEP               PODNAME                    DURATION  MESSAGE
 ● coinflip-6kvxp
 └---◷ flip-coin   coinflip-6kvxp-2242750624  9s        PodInitializing
Name:                coinflip-6kvxp
Namespace:           default
ServiceAccount:      default
Status:              Running
Created:             Mon Dec 24 23:00:49 -0500 (10 seconds ago)
Started:             Mon Dec 24 23:00:49 -0500 (10 seconds ago)
Duration:            10 seconds

STEP               PODNAME                    DURATION  MESSAGE
 ● coinflip-6kvxp
 └---◷ flip-coin   coinflip-6kvxp-2242750624  10s       PodInitializing
Name:                coinflip-6kvxp
Namespace:           default
ServiceAccount:      default
Status:              Running
Created:             Mon Dec 24 23:00:49 -0500 (11 seconds ago)
Started:             Mon Dec 24 23:00:49 -0500 (11 seconds ago)
Duration:            11 seconds

STEP               PODNAME                    DURATION  MESSAGE
 ● coinflip-6kvxp
 └---◷ flip-coin   coinflip-6kvxp-2242750624  11s       PodInitializing
Name:                coinflip-6kvxp
Namespace:           default
ServiceAccount:      default
Status:              Running
Created:             Mon Dec 24 23:00:49 -0500 (12 seconds ago)
Started:             Mon Dec 24 23:00:49 -0500 (12 seconds ago)
Duration:            12 seconds

STEP               PODNAME                    DURATION  MESSAGE
 ● coinflip-6kvxp
 └---◷ flip-coin   coinflip-6kvxp-2242750624  12s       PodInitializing
Name:                coinflip-6kvxp
Namespace:           default
ServiceAccount:      default
Status:              Running
Created:             Mon Dec 24 23:00:49 -0500 (13 seconds ago)
Started:             Mon Dec 24 23:00:49 -0500 (13 seconds ago)
Duration:            13 seconds

STEP               PODNAME                    DURATION  MESSAGE
 ● coinflip-6kvxp
 └---◷ flip-coin   coinflip-6kvxp-2242750624  13s       PodInitializing
Name:                coinflip-6kvxp
Namespace:           default
ServiceAccount:      default
Status:              Running
Created:             Mon Dec 24 23:00:49 -0500 (14 seconds ago)
Started:             Mon Dec 24 23:00:49 -0500 (14 seconds ago)
Duration:            14 seconds

STEP               PODNAME                    DURATION  MESSAGE
 ● coinflip-6kvxp
 └---◷ flip-coin   coinflip-6kvxp-2242750624  14s       PodInitializing
Name:                coinflip-6kvxp
Namespace:           default
ServiceAccount:      default
Status:              Running
Created:             Mon Dec 24 23:00:49 -0500 (14 seconds ago)
Started:             Mon Dec 24 23:00:49 -0500 (14 seconds ago)
Duration:            14 seconds

STEP               PODNAME                    DURATION  MESSAGE
 ● coinflip-6kvxp
 └---● flip-coin   coinflip-6kvxp-2242750624  14s
Name:                coinflip-6kvxp
Namespace:           default
ServiceAccount:      default
Status:              Running
Created:             Mon Dec 24 23:00:49 -0500 (15 seconds ago)
Started:             Mon Dec 24 23:00:49 -0500 (15 seconds ago)
Duration:            15 seconds

STEP               PODNAME                    DURATION  MESSAGE
 ● coinflip-6kvxp
 └---● flip-coin   coinflip-6kvxp-2242750624  15s
Name:                coinflip-6kvxp
Namespace:           default
ServiceAccount:      default
Status:              Running
Created:             Mon Dec 24 23:00:49 -0500 (15 seconds ago)
Started:             Mon Dec 24 23:00:49 -0500 (15 seconds ago)
Duration:            15 seconds

STEP               PODNAME                    DURATION  MESSAGE
 ● coinflip-6kvxp
 └---● flip-coin   coinflip-6kvxp-2242750624  15s
Name:                coinflip-6kvxp
Namespace:           default
ServiceAccount:      default
Status:              Running
Created:             Mon Dec 24 23:00:49 -0500 (16 seconds ago)
Started:             Mon Dec 24 23:00:49 -0500 (16 seconds ago)
Duration:            16 seconds

STEP               PODNAME                    DURATION  MESSAGE
 ● coinflip-6kvxp
 └---● flip-coin   coinflip-6kvxp-2242750624  16s
Name:                coinflip-6kvxp
Namespace:           default
ServiceAccount:      default
Status:              Running
Created:             Mon Dec 24 23:00:49 -0500 (16 seconds ago)
Started:             Mon Dec 24 23:00:49 -0500 (16 seconds ago)
Duration:            16 seconds

STEP               PODNAME                    DURATION  MESSAGE
 ● coinflip-6kvxp
 ├---✔ flip-coin   coinflip-6kvxp-2242750624  15s
 └-·-◷ heads       coinflip-6kvxp-1176800745  0s
   └-○ tails                                            when 'heads == tails' evaluated false
Name:                coinflip-6kvxp
Namespace:           default
ServiceAccount:      default
Status:              Running
Created:             Mon Dec 24 23:00:49 -0500 (17 seconds ago)
Started:             Mon Dec 24 23:00:49 -0500 (17 seconds ago)
Duration:            17 seconds

STEP               PODNAME                    DURATION  MESSAGE
 ● coinflip-6kvxp
 ├---✔ flip-coin   coinflip-6kvxp-2242750624  15s
 └-·-◷ heads       coinflip-6kvxp-1176800745  1s
   └-○ tails                                            when 'heads == tails' evaluated false
Name:                coinflip-6kvxp
Namespace:           default
ServiceAccount:      default
Status:              Running
Created:             Mon Dec 24 23:00:49 -0500 (17 seconds ago)
Started:             Mon Dec 24 23:00:49 -0500 (17 seconds ago)
Duration:            17 seconds

STEP               PODNAME                    DURATION  MESSAGE
 ● coinflip-6kvxp
 ├---✔ flip-coin   coinflip-6kvxp-2242750624  15s
 └-·-◷ heads       coinflip-6kvxp-1176800745  1s        ContainerCreating
   └-○ tails                                            when 'heads == tails' evaluated false
Name:                coinflip-6kvxp
Namespace:           default
ServiceAccount:      default
Status:              Running
Created:             Mon Dec 24 23:00:49 -0500 (18 seconds ago)
Started:             Mon Dec 24 23:00:49 -0500 (18 seconds ago)
Duration:            18 seconds

STEP               PODNAME                    DURATION  MESSAGE
 ● coinflip-6kvxp
 ├---✔ flip-coin   coinflip-6kvxp-2242750624  15s
 └-·-◷ heads       coinflip-6kvxp-1176800745  2s        ContainerCreating
   └-○ tails                                            when 'heads == tails' evaluated false
Name:                coinflip-6kvxp
Namespace:           default
ServiceAccount:      default
Status:              Running
Created:             Mon Dec 24 23:00:49 -0500 (19 seconds ago)
Started:             Mon Dec 24 23:00:49 -0500 (19 seconds ago)
Duration:            19 seconds

STEP               PODNAME                    DURATION  MESSAGE
 ● coinflip-6kvxp
 ├---✔ flip-coin   coinflip-6kvxp-2242750624  15s
 └-·-◷ heads       coinflip-6kvxp-1176800745  3s        ContainerCreating
   └-○ tails                                            when 'heads == tails' evaluated false
Name:                coinflip-6kvxp
Namespace:           default
ServiceAccount:      default
Status:              Running
Created:             Mon Dec 24 23:00:49 -0500 (19 seconds ago)
Started:             Mon Dec 24 23:00:49 -0500 (19 seconds ago)
Duration:            19 seconds

STEP               PODNAME                    DURATION  MESSAGE
 ● coinflip-6kvxp
 ├---✔ flip-coin   coinflip-6kvxp-2242750624  15s
 └-·-● heads       coinflip-6kvxp-1176800745  3s
   └-○ tails                                            when 'heads == tails' evaluated false
Name:                coinflip-6kvxp
Namespace:           default
ServiceAccount:      default
Status:              Running
Created:             Mon Dec 24 23:00:49 -0500 (20 seconds ago)
Started:             Mon Dec 24 23:00:49 -0500 (20 seconds ago)
Duration:            20 seconds

STEP               PODNAME                    DURATION  MESSAGE
 ● coinflip-6kvxp
 ├---✔ flip-coin   coinflip-6kvxp-2242750624  15s
 └-·-● heads       coinflip-6kvxp-1176800745  4s
   └-○ tails                                            when 'heads == tails' evaluated false
Name:                coinflip-6kvxp
Namespace:           default
ServiceAccount:      default
Status:              Succeeded
Created:             Mon Dec 24 23:00:49 -0500 (20 seconds ago)
Started:             Mon Dec 24 23:00:49 -0500 (20 seconds ago)
Finished:            Mon Dec 24 23:01:09 -0500 (now)
Duration:            20 seconds

STEP               PODNAME                    DURATION  MESSAGE
 ✔ coinflip-6kvxp
 ├---✔ flip-coin   coinflip-6kvxp-2242750624  15s
 └-·-✔ heads       coinflip-6kvxp-1176800745  4s
   └-○ tails                                            when 'heads == tails' evaluated false
```

List argo workflows:
```
$ argo list
NAME                STATUS      AGE   DURATION
coinflip-6kvxp      Succeeded   41m   20s
hello-world-2t7jc   Succeeded   1h    2m
```

Then view the logs of the workflow ```coinflip-6kvxp```:
```
$ argo logs coinflip-6kvxp
FATA[0000] pods "coinflip-6kvxp" not found
```

Ooops! Let's look at the pods:
```
$ kubectl get pod
NAME                        READY   STATUS      RESTARTS   AGE
coinflip-6kvxp-1176800745   0/2     Completed   0          44m
coinflip-6kvxp-2242750624   0/2     Completed   0          44m
hello-world-2t7jc           0/2     Completed   0          1h
```

Okay, use the pod's name to view the logs:
```
$ argo logs coinflip-6kvxp-1176800745
it was heads
$ argo logs coinflip-6kvxp-2242750624
heads
```

### Example 3

The example 3 uses the the following yaml file
```
apiVersion: argoproj.io/v1alpha1
kind: Workflow
metadata:
  generateName: loops-maps-
spec:
  entrypoint: loop-map-example
  templates:
  - name: loop-map-example
    steps:
    - - name: test-linux
        template: cat-os-release
        arguments:
          parameters:
          - name: image
            value: "{{item.image}}"
          - name: tag
            value: "{{item.tag}}"
        withItems:
        - { image: 'debian', tag: '9.1' }
        - { image: 'debian', tag: '8.9' }
        - { image: 'alpine', tag: '3.6' }
        - { image: 'ubuntu', tag: '17.10' }

  - name: cat-os-release
    inputs:
      parameters:
      - name: image
      - name: tag
    container:
      image: "{{inputs.parameters.image}}:{{inputs.parameters.tag}}"
      command: [cat]
      args: [/etc/os-release]
```

```
$ argo submit --watch https://raw.githubusercontent.com/argoproj/argo/master/examples/loops-maps.yaml
Name:                loops-maps-64szb
Namespace:           default
ServiceAccount:      default
Status:              Pending
Created:             Mon Dec 24 23:49:25 -0500 (now)
Name:                loops-maps-64szb
Namespace:           default
ServiceAccount:      default
Status:              Pending
Created:             Mon Dec 24 23:49:25 -0500 (now)
Name:                loops-maps-64szb
Namespace:           default
ServiceAccount:      default
Status:              Running
Created:             Mon Dec 24 23:49:25 -0500 (now)
Started:             Mon Dec 24 23:49:25 -0500 (now)
Duration:            0 seconds

STEP                                         PODNAME                      DURATION  MESSAGE
 ● loops-maps-64szb
 └-·-◷ test-linux(0:image:debian,tag:9.1)    loops-maps-64szb-1451238192  0s
   ├-◷ test-linux(1:image:debian,tag:8.9)    loops-maps-64szb-364729162   0s
   ├-◷ test-linux(2:image:alpine,tag:3.6)    loops-maps-64szb-892921377   0s
   └-◷ test-linux(3:image:ubuntu,tag:17.10)  loops-maps-64szb-244881556   0s
Name:                loops-maps-64szb
Namespace:           default
ServiceAccount:      default
Status:              Running
Created:             Mon Dec 24 23:49:25 -0500 (1 second ago)
Started:             Mon Dec 24 23:49:25 -0500 (1 second ago)
Duration:            1 second

STEP                                         PODNAME                      DURATION  MESSAGE
 ● loops-maps-64szb
 └-·-◷ test-linux(0:image:debian,tag:9.1)    loops-maps-64szb-1451238192  1s
   ├-◷ test-linux(1:image:debian,tag:8.9)    loops-maps-64szb-364729162   1s
   ├-◷ test-linux(2:image:alpine,tag:3.6)    loops-maps-64szb-892921377   1s
   └-◷ test-linux(3:image:ubuntu,tag:17.10)  loops-maps-64szb-244881556   1s
Name:                loops-maps-64szb
Namespace:           default
ServiceAccount:      default
Status:              Running
Created:             Mon Dec 24 23:49:25 -0500 (1 second ago)
Started:             Mon Dec 24 23:49:25 -0500 (1 second ago)
Duration:            1 second

STEP                                         PODNAME                      DURATION  MESSAGE
 ● loops-maps-64szb
 └-·-◷ test-linux(0:image:debian,tag:9.1)    loops-maps-64szb-1451238192  1s        ContainerCreating
   ├-◷ test-linux(1:image:debian,tag:8.9)    loops-maps-64szb-364729162   1s        ContainerCreating
   ├-◷ test-linux(2:image:alpine,tag:3.6)    loops-maps-64szb-892921377   1s        ContainerCreating
   └-◷ test-linux(3:image:ubuntu,tag:17.10)  loops-maps-64szb-244881556   1s        ContainerCreating
Name:                loops-maps-64szb
Namespace:           default
ServiceAccount:      default
Status:              Running
Created:             Mon Dec 24 23:49:25 -0500 (2 seconds ago)
Started:             Mon Dec 24 23:49:25 -0500 (2 seconds ago)
Duration:            2 seconds

STEP                                         PODNAME                      DURATION  MESSAGE
 ● loops-maps-64szb
 └-·-◷ test-linux(0:image:debian,tag:9.1)    loops-maps-64szb-1451238192  2s        ContainerCreating
   ├-◷ test-linux(1:image:debian,tag:8.9)    loops-maps-64szb-364729162   2s        ContainerCreating
   ├-◷ test-linux(2:image:alpine,tag:3.6)    loops-maps-64szb-892921377   2s        ContainerCreating
   └-◷ test-linux(3:image:ubuntu,tag:17.10)  loops-maps-64szb-244881556   2s        ContainerCreating
Name:                loops-maps-64szb
Namespace:           default
ServiceAccount:      default
Status:              Running
Created:             Mon Dec 24 23:49:25 -0500 (2 seconds ago)
Started:             Mon Dec 24 23:49:25 -0500 (2 seconds ago)
Duration:            2 seconds

STEP                                         PODNAME                      DURATION  MESSAGE
 ● loops-maps-64szb
 └-·-◷ test-linux(0:image:debian,tag:9.1)    loops-maps-64szb-1451238192  2s        ContainerCreating
   ├-◷ test-linux(1:image:debian,tag:8.9)    loops-maps-64szb-364729162   2s        ContainerCreating
   ├-● test-linux(2:image:alpine,tag:3.6)    loops-maps-64szb-892921377   2s
   └-◷ test-linux(3:image:ubuntu,tag:17.10)  loops-maps-64szb-244881556   2s        ContainerCreating
Name:                loops-maps-64szb
Namespace:           default
ServiceAccount:      default
Status:              Running
Created:             Mon Dec 24 23:49:25 -0500 (3 seconds ago)
Started:             Mon Dec 24 23:49:25 -0500 (3 seconds ago)
Duration:            3 seconds

STEP                                         PODNAME                      DURATION  MESSAGE
 ● loops-maps-64szb
 └-·-◷ test-linux(0:image:debian,tag:9.1)    loops-maps-64szb-1451238192  3s        ContainerCreating
   ├-◷ test-linux(1:image:debian,tag:8.9)    loops-maps-64szb-364729162   3s        ContainerCreating
   ├-● test-linux(2:image:alpine,tag:3.6)    loops-maps-64szb-892921377   3s
   └-◷ test-linux(3:image:ubuntu,tag:17.10)  loops-maps-64szb-244881556   3s        ContainerCreating
Name:                loops-maps-64szb
Namespace:           default
ServiceAccount:      default
Status:              Running
Created:             Mon Dec 24 23:49:25 -0500 (3 seconds ago)
Started:             Mon Dec 24 23:49:25 -0500 (3 seconds ago)
Duration:            3 seconds

STEP                                         PODNAME                      DURATION  MESSAGE
 ● loops-maps-64szb
 └-·-◷ test-linux(0:image:debian,tag:9.1)    loops-maps-64szb-1451238192  3s        ContainerCreating
   ├-◷ test-linux(1:image:debian,tag:8.9)    loops-maps-64szb-364729162   3s        ContainerCreating
   ├-✔ test-linux(2:image:alpine,tag:3.6)    loops-maps-64szb-892921377   2s
   └-◷ test-linux(3:image:ubuntu,tag:17.10)  loops-maps-64szb-244881556   3s        ContainerCreating
Name:                loops-maps-64szb
Namespace:           default
ServiceAccount:      default
Status:              Running
Created:             Mon Dec 24 23:49:25 -0500 (4 seconds ago)
Started:             Mon Dec 24 23:49:25 -0500 (4 seconds ago)
Duration:            4 seconds

STEP                                         PODNAME                      DURATION  MESSAGE
 ● loops-maps-64szb
 └-·-◷ test-linux(0:image:debian,tag:9.1)    loops-maps-64szb-1451238192  4s        ContainerCreating
   ├-◷ test-linux(1:image:debian,tag:8.9)    loops-maps-64szb-364729162   4s        ContainerCreating
   ├-✔ test-linux(2:image:alpine,tag:3.6)    loops-maps-64szb-892921377   2s
   └-◷ test-linux(3:image:ubuntu,tag:17.10)  loops-maps-64szb-244881556   4s        ContainerCreating
Name:                loops-maps-64szb
Namespace:           default
ServiceAccount:      default
Status:              Running
Created:             Mon Dec 24 23:49:25 -0500 (5 seconds ago)
Started:             Mon Dec 24 23:49:25 -0500 (5 seconds ago)
Duration:            5 seconds

STEP                                         PODNAME                      DURATION  MESSAGE
 ● loops-maps-64szb
 └-·-◷ test-linux(0:image:debian,tag:9.1)    loops-maps-64szb-1451238192  5s        ContainerCreating
   ├-◷ test-linux(1:image:debian,tag:8.9)    loops-maps-64szb-364729162   5s        ContainerCreating
   ├-✔ test-linux(2:image:alpine,tag:3.6)    loops-maps-64szb-892921377   2s
   └-◷ test-linux(3:image:ubuntu,tag:17.10)  loops-maps-64szb-244881556   5s        ContainerCreating
Name:                loops-maps-64szb
Namespace:           default
ServiceAccount:      default
Status:              Running
Created:             Mon Dec 24 23:49:25 -0500 (6 seconds ago)
Started:             Mon Dec 24 23:49:25 -0500 (6 seconds ago)
Duration:            6 seconds

STEP                                         PODNAME                      DURATION  MESSAGE
 ● loops-maps-64szb
 └-·-◷ test-linux(0:image:debian,tag:9.1)    loops-maps-64szb-1451238192  6s        ContainerCreating
   ├-◷ test-linux(1:image:debian,tag:8.9)    loops-maps-64szb-364729162   6s        ContainerCreating
   ├-✔ test-linux(2:image:alpine,tag:3.6)    loops-maps-64szb-892921377   2s
   └-◷ test-linux(3:image:ubuntu,tag:17.10)  loops-maps-64szb-244881556   6s        ContainerCreating
Name:                loops-maps-64szb
Namespace:           default
ServiceAccount:      default
Status:              Running
Created:             Mon Dec 24 23:49:25 -0500 (7 seconds ago)
Started:             Mon Dec 24 23:49:25 -0500 (7 seconds ago)
Duration:            7 seconds

STEP                                         PODNAME                      DURATION  MESSAGE
 ● loops-maps-64szb
 └-·-◷ test-linux(0:image:debian,tag:9.1)    loops-maps-64szb-1451238192  7s        ContainerCreating
   ├-◷ test-linux(1:image:debian,tag:8.9)    loops-maps-64szb-364729162   7s        ContainerCreating
   ├-✔ test-linux(2:image:alpine,tag:3.6)    loops-maps-64szb-892921377   2s
   └-◷ test-linux(3:image:ubuntu,tag:17.10)  loops-maps-64szb-244881556   7s        ContainerCreating
Name:                loops-maps-64szb
Namespace:           default
ServiceAccount:      default
Status:              Running
Created:             Mon Dec 24 23:49:25 -0500 (8 seconds ago)
Started:             Mon Dec 24 23:49:25 -0500 (8 seconds ago)
Duration:            8 seconds

STEP                                         PODNAME                      DURATION  MESSAGE
 ● loops-maps-64szb
 └-·-◷ test-linux(0:image:debian,tag:9.1)    loops-maps-64szb-1451238192  8s        ContainerCreating
   ├-◷ test-linux(1:image:debian,tag:8.9)    loops-maps-64szb-364729162   8s        ContainerCreating
   ├-✔ test-linux(2:image:alpine,tag:3.6)    loops-maps-64szb-892921377   2s
   └-◷ test-linux(3:image:ubuntu,tag:17.10)  loops-maps-64szb-244881556   8s        ContainerCreating
Name:                loops-maps-64szb
Namespace:           default
ServiceAccount:      default
Status:              Running
Created:             Mon Dec 24 23:49:25 -0500 (9 seconds ago)
Started:             Mon Dec 24 23:49:25 -0500 (9 seconds ago)
Duration:            9 seconds

STEP                                         PODNAME                      DURATION  MESSAGE
 ● loops-maps-64szb
 └-·-◷ test-linux(0:image:debian,tag:9.1)    loops-maps-64szb-1451238192  9s        ContainerCreating
   ├-◷ test-linux(1:image:debian,tag:8.9)    loops-maps-64szb-364729162   9s        ContainerCreating
   ├-✔ test-linux(2:image:alpine,tag:3.6)    loops-maps-64szb-892921377   2s
   └-◷ test-linux(3:image:ubuntu,tag:17.10)  loops-maps-64szb-244881556   9s        ContainerCreating
Name:                loops-maps-64szb
Namespace:           default
ServiceAccount:      default
Status:              Running
Created:             Mon Dec 24 23:49:25 -0500 (10 seconds ago)
Started:             Mon Dec 24 23:49:25 -0500 (10 seconds ago)
Duration:            10 seconds

STEP                                         PODNAME                      DURATION  MESSAGE
 ● loops-maps-64szb
 └-·-◷ test-linux(0:image:debian,tag:9.1)    loops-maps-64szb-1451238192  10s       ContainerCreating
   ├-◷ test-linux(1:image:debian,tag:8.9)    loops-maps-64szb-364729162   10s       ContainerCreating
   ├-✔ test-linux(2:image:alpine,tag:3.6)    loops-maps-64szb-892921377   2s
   └-◷ test-linux(3:image:ubuntu,tag:17.10)  loops-maps-64szb-244881556   10s       ContainerCreating
Name:                loops-maps-64szb
Namespace:           default
ServiceAccount:      default
Status:              Running
Created:             Mon Dec 24 23:49:25 -0500 (11 seconds ago)
Started:             Mon Dec 24 23:49:25 -0500 (11 seconds ago)
Duration:            11 seconds

STEP                                         PODNAME                      DURATION  MESSAGE
 ● loops-maps-64szb
 └-·-◷ test-linux(0:image:debian,tag:9.1)    loops-maps-64szb-1451238192  11s       ContainerCreating
   ├-◷ test-linux(1:image:debian,tag:8.9)    loops-maps-64szb-364729162   11s       ContainerCreating
   ├-✔ test-linux(2:image:alpine,tag:3.6)    loops-maps-64szb-892921377   2s
   └-◷ test-linux(3:image:ubuntu,tag:17.10)  loops-maps-64szb-244881556   11s       ContainerCreating
Name:                loops-maps-64szb
Namespace:           default
ServiceAccount:      default
Status:              Running
Created:             Mon Dec 24 23:49:25 -0500 (12 seconds ago)
Started:             Mon Dec 24 23:49:25 -0500 (12 seconds ago)
Duration:            12 seconds

STEP                                         PODNAME                      DURATION  MESSAGE
 ● loops-maps-64szb
 └-·-◷ test-linux(0:image:debian,tag:9.1)    loops-maps-64szb-1451238192  12s       ContainerCreating
   ├-◷ test-linux(1:image:debian,tag:8.9)    loops-maps-64szb-364729162   12s       ContainerCreating
   ├-✔ test-linux(2:image:alpine,tag:3.6)    loops-maps-64szb-892921377   2s
   └-◷ test-linux(3:image:ubuntu,tag:17.10)  loops-maps-64szb-244881556   12s       ContainerCreating
Name:                loops-maps-64szb
Namespace:           default
ServiceAccount:      default
Status:              Running
Created:             Mon Dec 24 23:49:25 -0500 (13 seconds ago)
Started:             Mon Dec 24 23:49:25 -0500 (13 seconds ago)
Duration:            13 seconds

STEP                                         PODNAME                      DURATION  MESSAGE
 ● loops-maps-64szb
 └-·-◷ test-linux(0:image:debian,tag:9.1)    loops-maps-64szb-1451238192  13s       ContainerCreating
   ├-◷ test-linux(1:image:debian,tag:8.9)    loops-maps-64szb-364729162   13s       ContainerCreating
   ├-✔ test-linux(2:image:alpine,tag:3.6)    loops-maps-64szb-892921377   2s
   └-◷ test-linux(3:image:ubuntu,tag:17.10)  loops-maps-64szb-244881556   13s       ContainerCreating
Name:                loops-maps-64szb
Namespace:           default
ServiceAccount:      default
Status:              Running
Created:             Mon Dec 24 23:49:25 -0500 (14 seconds ago)
Started:             Mon Dec 24 23:49:25 -0500 (14 seconds ago)
Duration:            14 seconds

STEP                                         PODNAME                      DURATION  MESSAGE
 ● loops-maps-64szb
 └-·-◷ test-linux(0:image:debian,tag:9.1)    loops-maps-64szb-1451238192  14s       ContainerCreating
   ├-◷ test-linux(1:image:debian,tag:8.9)    loops-maps-64szb-364729162   14s       ContainerCreating
   ├-✔ test-linux(2:image:alpine,tag:3.6)    loops-maps-64szb-892921377   2s
   └-◷ test-linux(3:image:ubuntu,tag:17.10)  loops-maps-64szb-244881556   14s       ContainerCreating
Name:                loops-maps-64szb
Namespace:           default
ServiceAccount:      default
Status:              Running
Created:             Mon Dec 24 23:49:25 -0500 (15 seconds ago)
Started:             Mon Dec 24 23:49:25 -0500 (15 seconds ago)
Duration:            15 seconds

STEP                                         PODNAME                      DURATION  MESSAGE
 ● loops-maps-64szb
 └-·-◷ test-linux(0:image:debian,tag:9.1)    loops-maps-64szb-1451238192  15s       ContainerCreating
   ├-◷ test-linux(1:image:debian,tag:8.9)    loops-maps-64szb-364729162   15s       ContainerCreating
   ├-✔ test-linux(2:image:alpine,tag:3.6)    loops-maps-64szb-892921377   2s
   └-◷ test-linux(3:image:ubuntu,tag:17.10)  loops-maps-64szb-244881556   15s       ContainerCreating
Name:                loops-maps-64szb
Namespace:           default
ServiceAccount:      default
Status:              Running
Created:             Mon Dec 24 23:49:25 -0500 (16 seconds ago)
Started:             Mon Dec 24 23:49:25 -0500 (16 seconds ago)
Duration:            16 seconds

STEP                                         PODNAME                      DURATION  MESSAGE
 ● loops-maps-64szb
 └-·-◷ test-linux(0:image:debian,tag:9.1)    loops-maps-64szb-1451238192  16s       ContainerCreating
   ├-◷ test-linux(1:image:debian,tag:8.9)    loops-maps-64szb-364729162   16s       ContainerCreating
   ├-✔ test-linux(2:image:alpine,tag:3.6)    loops-maps-64szb-892921377   2s
   └-◷ test-linux(3:image:ubuntu,tag:17.10)  loops-maps-64szb-244881556   16s       ContainerCreating
Name:                loops-maps-64szb
Namespace:           default
ServiceAccount:      default
Status:              Running
Created:             Mon Dec 24 23:49:25 -0500 (16 seconds ago)
Started:             Mon Dec 24 23:49:25 -0500 (16 seconds ago)
Duration:            16 seconds

STEP                                         PODNAME                      DURATION  MESSAGE
 ● loops-maps-64szb
 └-·-● test-linux(0:image:debian,tag:9.1)    loops-maps-64szb-1451238192  16s
   ├-◷ test-linux(1:image:debian,tag:8.9)    loops-maps-64szb-364729162   16s       ContainerCreating
   ├-✔ test-linux(2:image:alpine,tag:3.6)    loops-maps-64szb-892921377   2s
   └-◷ test-linux(3:image:ubuntu,tag:17.10)  loops-maps-64szb-244881556   16s       ContainerCreating
Name:                loops-maps-64szb
Namespace:           default
ServiceAccount:      default
Status:              Running
Created:             Mon Dec 24 23:49:25 -0500 (17 seconds ago)
Started:             Mon Dec 24 23:49:25 -0500 (17 seconds ago)
Duration:            17 seconds

STEP                                         PODNAME                      DURATION  MESSAGE
 ● loops-maps-64szb
 └-·-● test-linux(0:image:debian,tag:9.1)    loops-maps-64szb-1451238192  17s
   ├-◷ test-linux(1:image:debian,tag:8.9)    loops-maps-64szb-364729162   17s       ContainerCreating
   ├-✔ test-linux(2:image:alpine,tag:3.6)    loops-maps-64szb-892921377   2s
   └-◷ test-linux(3:image:ubuntu,tag:17.10)  loops-maps-64szb-244881556   17s       ContainerCreating
Name:                loops-maps-64szb
Namespace:           default
ServiceAccount:      default
Status:              Running
Created:             Mon Dec 24 23:49:25 -0500 (17 seconds ago)
Started:             Mon Dec 24 23:49:25 -0500 (17 seconds ago)
Duration:            17 seconds

STEP                                         PODNAME                      DURATION  MESSAGE
 ● loops-maps-64szb
 └-·-✔ test-linux(0:image:debian,tag:9.1)    loops-maps-64szb-1451238192  16s
   ├-◷ test-linux(1:image:debian,tag:8.9)    loops-maps-64szb-364729162   17s       ContainerCreating
   ├-✔ test-linux(2:image:alpine,tag:3.6)    loops-maps-64szb-892921377   2s
   └-◷ test-linux(3:image:ubuntu,tag:17.10)  loops-maps-64szb-244881556   17s       ContainerCreating
Name:                loops-maps-64szb
Namespace:           default
ServiceAccount:      default
Status:              Running
Created:             Mon Dec 24 23:49:25 -0500 (18 seconds ago)
Started:             Mon Dec 24 23:49:25 -0500 (18 seconds ago)
Duration:            18 seconds

STEP                                         PODNAME                      DURATION  MESSAGE
 ● loops-maps-64szb
 └-·-✔ test-linux(0:image:debian,tag:9.1)    loops-maps-64szb-1451238192  16s
   ├-◷ test-linux(1:image:debian,tag:8.9)    loops-maps-64szb-364729162   18s       ContainerCreating
   ├-✔ test-linux(2:image:alpine,tag:3.6)    loops-maps-64szb-892921377   2s
   └-◷ test-linux(3:image:ubuntu,tag:17.10)  loops-maps-64szb-244881556   18s       ContainerCreating
Name:                loops-maps-64szb
Namespace:           default
ServiceAccount:      default
Status:              Running
Created:             Mon Dec 24 23:49:25 -0500 (19 seconds ago)
Started:             Mon Dec 24 23:49:25 -0500 (19 seconds ago)
Duration:            19 seconds

STEP                                         PODNAME                      DURATION  MESSAGE
 ● loops-maps-64szb
 └-·-✔ test-linux(0:image:debian,tag:9.1)    loops-maps-64szb-1451238192  16s
   ├-◷ test-linux(1:image:debian,tag:8.9)    loops-maps-64szb-364729162   19s       ContainerCreating
   ├-✔ test-linux(2:image:alpine,tag:3.6)    loops-maps-64szb-892921377   2s
   └-◷ test-linux(3:image:ubuntu,tag:17.10)  loops-maps-64szb-244881556   19s       ContainerCreating
Name:                loops-maps-64szb
Namespace:           default
ServiceAccount:      default
Status:              Running
Created:             Mon Dec 24 23:49:25 -0500 (20 seconds ago)
Started:             Mon Dec 24 23:49:25 -0500 (20 seconds ago)
Duration:            20 seconds

STEP                                         PODNAME                      DURATION  MESSAGE
 ● loops-maps-64szb
 └-·-✔ test-linux(0:image:debian,tag:9.1)    loops-maps-64szb-1451238192  16s
   ├-◷ test-linux(1:image:debian,tag:8.9)    loops-maps-64szb-364729162   20s       ContainerCreating
   ├-✔ test-linux(2:image:alpine,tag:3.6)    loops-maps-64szb-892921377   2s
   └-◷ test-linux(3:image:ubuntu,tag:17.10)  loops-maps-64szb-244881556   20s       ContainerCreating
Name:                loops-maps-64szb
Namespace:           default
ServiceAccount:      default
Status:              Running
Created:             Mon Dec 24 23:49:25 -0500 (21 seconds ago)
Started:             Mon Dec 24 23:49:25 -0500 (21 seconds ago)
Duration:            21 seconds

STEP                                         PODNAME                      DURATION  MESSAGE
 ● loops-maps-64szb
 └-·-✔ test-linux(0:image:debian,tag:9.1)    loops-maps-64szb-1451238192  16s
   ├-◷ test-linux(1:image:debian,tag:8.9)    loops-maps-64szb-364729162   21s       ContainerCreating
   ├-✔ test-linux(2:image:alpine,tag:3.6)    loops-maps-64szb-892921377   2s
   └-◷ test-linux(3:image:ubuntu,tag:17.10)  loops-maps-64szb-244881556   21s       ContainerCreating
Name:                loops-maps-64szb
Namespace:           default
ServiceAccount:      default
Status:              Running
Created:             Mon Dec 24 23:49:25 -0500 (22 seconds ago)
Started:             Mon Dec 24 23:49:25 -0500 (22 seconds ago)
Duration:            22 seconds

STEP                                         PODNAME                      DURATION  MESSAGE
 ● loops-maps-64szb
 └-·-✔ test-linux(0:image:debian,tag:9.1)    loops-maps-64szb-1451238192  16s
   ├-◷ test-linux(1:image:debian,tag:8.9)    loops-maps-64szb-364729162   22s       ContainerCreating
   ├-✔ test-linux(2:image:alpine,tag:3.6)    loops-maps-64szb-892921377   2s
   └-◷ test-linux(3:image:ubuntu,tag:17.10)  loops-maps-64szb-244881556   22s       ContainerCreating
Name:                loops-maps-64szb
Namespace:           default
ServiceAccount:      default
Status:              Running
Created:             Mon Dec 24 23:49:25 -0500 (23 seconds ago)
Started:             Mon Dec 24 23:49:25 -0500 (23 seconds ago)
Duration:            23 seconds

STEP                                         PODNAME                      DURATION  MESSAGE
 ● loops-maps-64szb
 └-·-✔ test-linux(0:image:debian,tag:9.1)    loops-maps-64szb-1451238192  16s
   ├-◷ test-linux(1:image:debian,tag:8.9)    loops-maps-64szb-364729162   23s       ContainerCreating
   ├-✔ test-linux(2:image:alpine,tag:3.6)    loops-maps-64szb-892921377   2s
   └-◷ test-linux(3:image:ubuntu,tag:17.10)  loops-maps-64szb-244881556   23s       ContainerCreating
Name:                loops-maps-64szb
Namespace:           default
ServiceAccount:      default
Status:              Running
Created:             Mon Dec 24 23:49:25 -0500 (24 seconds ago)
Started:             Mon Dec 24 23:49:25 -0500 (24 seconds ago)
Duration:            24 seconds

STEP                                         PODNAME                      DURATION  MESSAGE
 ● loops-maps-64szb
 └-·-✔ test-linux(0:image:debian,tag:9.1)    loops-maps-64szb-1451238192  16s
   ├-◷ test-linux(1:image:debian,tag:8.9)    loops-maps-64szb-364729162   24s       ContainerCreating
   ├-✔ test-linux(2:image:alpine,tag:3.6)    loops-maps-64szb-892921377   2s
   └-◷ test-linux(3:image:ubuntu,tag:17.10)  loops-maps-64szb-244881556   24s       ContainerCreating
Name:                loops-maps-64szb
Namespace:           default
ServiceAccount:      default
Status:              Running
Created:             Mon Dec 24 23:49:25 -0500 (25 seconds ago)
Started:             Mon Dec 24 23:49:25 -0500 (25 seconds ago)
Duration:            25 seconds

STEP                                         PODNAME                      DURATION  MESSAGE
 ● loops-maps-64szb
 └-·-✔ test-linux(0:image:debian,tag:9.1)    loops-maps-64szb-1451238192  16s
   ├-◷ test-linux(1:image:debian,tag:8.9)    loops-maps-64szb-364729162   25s       ContainerCreating
   ├-✔ test-linux(2:image:alpine,tag:3.6)    loops-maps-64szb-892921377   2s
   └-◷ test-linux(3:image:ubuntu,tag:17.10)  loops-maps-64szb-244881556   25s       ContainerCreating
Name:                loops-maps-64szb
Namespace:           default
ServiceAccount:      default
Status:              Running
Created:             Mon Dec 24 23:49:25 -0500 (26 seconds ago)
Started:             Mon Dec 24 23:49:25 -0500 (26 seconds ago)
Duration:            26 seconds

STEP                                         PODNAME                      DURATION  MESSAGE
 ● loops-maps-64szb
 └-·-✔ test-linux(0:image:debian,tag:9.1)    loops-maps-64szb-1451238192  16s
   ├-◷ test-linux(1:image:debian,tag:8.9)    loops-maps-64szb-364729162   26s       ContainerCreating
   ├-✔ test-linux(2:image:alpine,tag:3.6)    loops-maps-64szb-892921377   2s
   └-◷ test-linux(3:image:ubuntu,tag:17.10)  loops-maps-64szb-244881556   26s       ContainerCreating
Name:                loops-maps-64szb
Namespace:           default
ServiceAccount:      default
Status:              Running
Created:             Mon Dec 24 23:49:25 -0500 (27 seconds ago)
Started:             Mon Dec 24 23:49:25 -0500 (27 seconds ago)
Duration:            27 seconds

STEP                                         PODNAME                      DURATION  MESSAGE
 ● loops-maps-64szb
 └-·-✔ test-linux(0:image:debian,tag:9.1)    loops-maps-64szb-1451238192  16s
   ├-◷ test-linux(1:image:debian,tag:8.9)    loops-maps-64szb-364729162   27s       ContainerCreating
   ├-✔ test-linux(2:image:alpine,tag:3.6)    loops-maps-64szb-892921377   2s
   └-◷ test-linux(3:image:ubuntu,tag:17.10)  loops-maps-64szb-244881556   27s       ContainerCreating
Name:                loops-maps-64szb
Namespace:           default
ServiceAccount:      default
Status:              Running
Created:             Mon Dec 24 23:49:25 -0500 (28 seconds ago)
Started:             Mon Dec 24 23:49:25 -0500 (28 seconds ago)
Duration:            28 seconds

STEP                                         PODNAME                      DURATION  MESSAGE
 ● loops-maps-64szb
 └-·-✔ test-linux(0:image:debian,tag:9.1)    loops-maps-64szb-1451238192  16s
   ├-◷ test-linux(1:image:debian,tag:8.9)    loops-maps-64szb-364729162   28s       ContainerCreating
   ├-✔ test-linux(2:image:alpine,tag:3.6)    loops-maps-64szb-892921377   2s
   └-◷ test-linux(3:image:ubuntu,tag:17.10)  loops-maps-64szb-244881556   28s       ContainerCreating
Name:                loops-maps-64szb
Namespace:           default
ServiceAccount:      default
Status:              Running
Created:             Mon Dec 24 23:49:25 -0500 (29 seconds ago)
Started:             Mon Dec 24 23:49:25 -0500 (29 seconds ago)
Duration:            29 seconds

STEP                                         PODNAME                      DURATION  MESSAGE
 ● loops-maps-64szb
 └-·-✔ test-linux(0:image:debian,tag:9.1)    loops-maps-64szb-1451238192  16s
   ├-◷ test-linux(1:image:debian,tag:8.9)    loops-maps-64szb-364729162   29s       ContainerCreating
   ├-✔ test-linux(2:image:alpine,tag:3.6)    loops-maps-64szb-892921377   2s
   └-◷ test-linux(3:image:ubuntu,tag:17.10)  loops-maps-64szb-244881556   29s       ContainerCreating
Name:                loops-maps-64szb
Namespace:           default
ServiceAccount:      default
Status:              Running
Created:             Mon Dec 24 23:49:25 -0500 (30 seconds ago)
Started:             Mon Dec 24 23:49:25 -0500 (30 seconds ago)
Duration:            30 seconds

STEP                                         PODNAME                      DURATION  MESSAGE
 ● loops-maps-64szb
 └-·-✔ test-linux(0:image:debian,tag:9.1)    loops-maps-64szb-1451238192  16s
   ├-◷ test-linux(1:image:debian,tag:8.9)    loops-maps-64szb-364729162   30s       ContainerCreating
   ├-✔ test-linux(2:image:alpine,tag:3.6)    loops-maps-64szb-892921377   2s
   └-◷ test-linux(3:image:ubuntu,tag:17.10)  loops-maps-64szb-244881556   30s       ContainerCreating
Name:                loops-maps-64szb
Namespace:           default
ServiceAccount:      default
Status:              Running
Created:             Mon Dec 24 23:49:25 -0500 (31 seconds ago)
Started:             Mon Dec 24 23:49:25 -0500 (31 seconds ago)
Duration:            31 seconds

STEP                                         PODNAME                      DURATION  MESSAGE
 ● loops-maps-64szb
 └-·-✔ test-linux(0:image:debian,tag:9.1)    loops-maps-64szb-1451238192  16s
   ├-◷ test-linux(1:image:debian,tag:8.9)    loops-maps-64szb-364729162   31s       ContainerCreating
   ├-✔ test-linux(2:image:alpine,tag:3.6)    loops-maps-64szb-892921377   2s
   └-◷ test-linux(3:image:ubuntu,tag:17.10)  loops-maps-64szb-244881556   31s       ContainerCreating
Name:                loops-maps-64szb
Namespace:           default
ServiceAccount:      default
Status:              Running
Created:             Mon Dec 24 23:49:25 -0500 (31 seconds ago)
Started:             Mon Dec 24 23:49:25 -0500 (31 seconds ago)
Duration:            31 seconds

STEP                                         PODNAME                      DURATION  MESSAGE
 ● loops-maps-64szb
 └-·-✔ test-linux(0:image:debian,tag:9.1)    loops-maps-64szb-1451238192  16s
   ├-◷ test-linux(1:image:debian,tag:8.9)    loops-maps-64szb-364729162   31s       ContainerCreating
   ├-✔ test-linux(2:image:alpine,tag:3.6)    loops-maps-64szb-892921377   2s
   └-● test-linux(3:image:ubuntu,tag:17.10)  loops-maps-64szb-244881556   31s
Name:                loops-maps-64szb
Namespace:           default
ServiceAccount:      default
Status:              Running
Created:             Mon Dec 24 23:49:25 -0500 (32 seconds ago)
Started:             Mon Dec 24 23:49:25 -0500 (32 seconds ago)
Duration:            32 seconds

STEP                                         PODNAME                      DURATION  MESSAGE
 ● loops-maps-64szb
 └-·-✔ test-linux(0:image:debian,tag:9.1)    loops-maps-64szb-1451238192  16s
   ├-◷ test-linux(1:image:debian,tag:8.9)    loops-maps-64szb-364729162   32s       ContainerCreating
   ├-✔ test-linux(2:image:alpine,tag:3.6)    loops-maps-64szb-892921377   2s
   └-● test-linux(3:image:ubuntu,tag:17.10)  loops-maps-64szb-244881556   32s
Name:                loops-maps-64szb
Namespace:           default
ServiceAccount:      default
Status:              Running
Created:             Mon Dec 24 23:49:25 -0500 (33 seconds ago)
Started:             Mon Dec 24 23:49:25 -0500 (33 seconds ago)
Duration:            33 seconds

STEP                                         PODNAME                      DURATION  MESSAGE
 ● loops-maps-64szb
 └-·-✔ test-linux(0:image:debian,tag:9.1)    loops-maps-64szb-1451238192  16s
   ├-◷ test-linux(1:image:debian,tag:8.9)    loops-maps-64szb-364729162   33s       ContainerCreating
   ├-✔ test-linux(2:image:alpine,tag:3.6)    loops-maps-64szb-892921377   2s
   └-✔ test-linux(3:image:ubuntu,tag:17.10)  loops-maps-64szb-244881556   32s
Name:                loops-maps-64szb
Namespace:           default
ServiceAccount:      default
Status:              Running
Created:             Mon Dec 24 23:49:25 -0500 (33 seconds ago)
Started:             Mon Dec 24 23:49:25 -0500 (33 seconds ago)
Duration:            33 seconds

STEP                                         PODNAME                      DURATION  MESSAGE
 ● loops-maps-64szb
 └-·-✔ test-linux(0:image:debian,tag:9.1)    loops-maps-64szb-1451238192  16s
   ├-◷ test-linux(1:image:debian,tag:8.9)    loops-maps-64szb-364729162   33s       ContainerCreating
   ├-✔ test-linux(2:image:alpine,tag:3.6)    loops-maps-64szb-892921377   2s
   └-✔ test-linux(3:image:ubuntu,tag:17.10)  loops-maps-64szb-244881556   32s
Name:                loops-maps-64szb
Namespace:           default
ServiceAccount:      default
Status:              Running
Created:             Mon Dec 24 23:49:25 -0500 (34 seconds ago)
Started:             Mon Dec 24 23:49:25 -0500 (34 seconds ago)
Duration:            34 seconds

STEP                                         PODNAME                      DURATION  MESSAGE
 ● loops-maps-64szb
 └-·-✔ test-linux(0:image:debian,tag:9.1)    loops-maps-64szb-1451238192  16s
   ├-◷ test-linux(1:image:debian,tag:8.9)    loops-maps-64szb-364729162   34s       ContainerCreating
   ├-✔ test-linux(2:image:alpine,tag:3.6)    loops-maps-64szb-892921377   2s
   └-✔ test-linux(3:image:ubuntu,tag:17.10)  loops-maps-64szb-244881556   32s
Name:                loops-maps-64szb
Namespace:           default
ServiceAccount:      default
Status:              Running
Created:             Mon Dec 24 23:49:25 -0500 (35 seconds ago)
Started:             Mon Dec 24 23:49:25 -0500 (35 seconds ago)
Duration:            35 seconds

STEP                                         PODNAME                      DURATION  MESSAGE
 ● loops-maps-64szb
 └-·-✔ test-linux(0:image:debian,tag:9.1)    loops-maps-64szb-1451238192  16s
   ├-◷ test-linux(1:image:debian,tag:8.9)    loops-maps-64szb-364729162   35s       ContainerCreating
   ├-✔ test-linux(2:image:alpine,tag:3.6)    loops-maps-64szb-892921377   2s
   └-✔ test-linux(3:image:ubuntu,tag:17.10)  loops-maps-64szb-244881556   32s
Name:                loops-maps-64szb
Namespace:           default
ServiceAccount:      default
Status:              Running
Created:             Mon Dec 24 23:49:25 -0500 (36 seconds ago)
Started:             Mon Dec 24 23:49:25 -0500 (36 seconds ago)
Duration:            36 seconds

STEP                                         PODNAME                      DURATION  MESSAGE
 ● loops-maps-64szb
 └-·-✔ test-linux(0:image:debian,tag:9.1)    loops-maps-64szb-1451238192  16s
   ├-◷ test-linux(1:image:debian,tag:8.9)    loops-maps-64szb-364729162   36s       ContainerCreating
   ├-✔ test-linux(2:image:alpine,tag:3.6)    loops-maps-64szb-892921377   2s
   └-✔ test-linux(3:image:ubuntu,tag:17.10)  loops-maps-64szb-244881556   32s
Name:                loops-maps-64szb
Namespace:           default
ServiceAccount:      default
Status:              Running
Created:             Mon Dec 24 23:49:25 -0500 (37 seconds ago)
Started:             Mon Dec 24 23:49:25 -0500 (37 seconds ago)
Duration:            37 seconds

STEP                                         PODNAME                      DURATION  MESSAGE
 ● loops-maps-64szb
 └-·-✔ test-linux(0:image:debian,tag:9.1)    loops-maps-64szb-1451238192  16s
   ├-◷ test-linux(1:image:debian,tag:8.9)    loops-maps-64szb-364729162   37s       ContainerCreating
   ├-✔ test-linux(2:image:alpine,tag:3.6)    loops-maps-64szb-892921377   2s
   └-✔ test-linux(3:image:ubuntu,tag:17.10)  loops-maps-64szb-244881556   32s
Name:                loops-maps-64szb
Namespace:           default
ServiceAccount:      default
Status:              Running
Created:             Mon Dec 24 23:49:25 -0500 (38 seconds ago)
Started:             Mon Dec 24 23:49:25 -0500 (38 seconds ago)
Duration:            38 seconds

STEP                                         PODNAME                      DURATION  MESSAGE
 ● loops-maps-64szb
 └-·-✔ test-linux(0:image:debian,tag:9.1)    loops-maps-64szb-1451238192  16s
   ├-◷ test-linux(1:image:debian,tag:8.9)    loops-maps-64szb-364729162   38s       ContainerCreating
   ├-✔ test-linux(2:image:alpine,tag:3.6)    loops-maps-64szb-892921377   2s
   └-✔ test-linux(3:image:ubuntu,tag:17.10)  loops-maps-64szb-244881556   32s
Name:                loops-maps-64szb
Namespace:           default
ServiceAccount:      default
Status:              Running
Created:             Mon Dec 24 23:49:25 -0500 (39 seconds ago)
Started:             Mon Dec 24 23:49:25 -0500 (39 seconds ago)
Duration:            39 seconds

STEP                                         PODNAME                      DURATION  MESSAGE
 ● loops-maps-64szb
 └-·-✔ test-linux(0:image:debian,tag:9.1)    loops-maps-64szb-1451238192  16s
   ├-◷ test-linux(1:image:debian,tag:8.9)    loops-maps-64szb-364729162   39s       ContainerCreating
   ├-✔ test-linux(2:image:alpine,tag:3.6)    loops-maps-64szb-892921377   2s
   └-✔ test-linux(3:image:ubuntu,tag:17.10)  loops-maps-64szb-244881556   32s
Name:                loops-maps-64szb
Namespace:           default
ServiceAccount:      default
Status:              Running
Created:             Mon Dec 24 23:49:25 -0500 (40 seconds ago)
Started:             Mon Dec 24 23:49:25 -0500 (40 seconds ago)
Duration:            40 seconds

STEP                                         PODNAME                      DURATION  MESSAGE
 ● loops-maps-64szb
 └-·-✔ test-linux(0:image:debian,tag:9.1)    loops-maps-64szb-1451238192  16s
   ├-◷ test-linux(1:image:debian,tag:8.9)    loops-maps-64szb-364729162   40s       ContainerCreating
   ├-✔ test-linux(2:image:alpine,tag:3.6)    loops-maps-64szb-892921377   2s
   └-✔ test-linux(3:image:ubuntu,tag:17.10)  loops-maps-64szb-244881556   32s
Name:                loops-maps-64szb
Namespace:           default
ServiceAccount:      default
Status:              Running
Created:             Mon Dec 24 23:49:25 -0500 (41 seconds ago)
Started:             Mon Dec 24 23:49:25 -0500 (41 seconds ago)
Duration:            41 seconds

STEP                                         PODNAME                      DURATION  MESSAGE
 ● loops-maps-64szb
 └-·-✔ test-linux(0:image:debian,tag:9.1)    loops-maps-64szb-1451238192  16s
   ├-◷ test-linux(1:image:debian,tag:8.9)    loops-maps-64szb-364729162   41s       ContainerCreating
   ├-✔ test-linux(2:image:alpine,tag:3.6)    loops-maps-64szb-892921377   2s
   └-✔ test-linux(3:image:ubuntu,tag:17.10)  loops-maps-64szb-244881556   32s
Name:                loops-maps-64szb
Namespace:           default
ServiceAccount:      default
Status:              Running
Created:             Mon Dec 24 23:49:25 -0500 (42 seconds ago)
Started:             Mon Dec 24 23:49:25 -0500 (42 seconds ago)
Duration:            42 seconds

STEP                                         PODNAME                      DURATION  MESSAGE
 ● loops-maps-64szb
 └-·-✔ test-linux(0:image:debian,tag:9.1)    loops-maps-64szb-1451238192  16s
   ├-◷ test-linux(1:image:debian,tag:8.9)    loops-maps-64szb-364729162   42s       ContainerCreating
   ├-✔ test-linux(2:image:alpine,tag:3.6)    loops-maps-64szb-892921377   2s
   └-✔ test-linux(3:image:ubuntu,tag:17.10)  loops-maps-64szb-244881556   32s
Name:                loops-maps-64szb
Namespace:           default
ServiceAccount:      default
Status:              Running
Created:             Mon Dec 24 23:49:25 -0500 (43 seconds ago)
Started:             Mon Dec 24 23:49:25 -0500 (43 seconds ago)
Duration:            43 seconds

STEP                                         PODNAME                      DURATION  MESSAGE
 ● loops-maps-64szb
 └-·-✔ test-linux(0:image:debian,tag:9.1)    loops-maps-64szb-1451238192  16s
   ├-◷ test-linux(1:image:debian,tag:8.9)    loops-maps-64szb-364729162   43s       ContainerCreating
   ├-✔ test-linux(2:image:alpine,tag:3.6)    loops-maps-64szb-892921377   2s
   └-✔ test-linux(3:image:ubuntu,tag:17.10)  loops-maps-64szb-244881556   32s
Name:                loops-maps-64szb
Namespace:           default
ServiceAccount:      default
Status:              Running
Created:             Mon Dec 24 23:49:25 -0500 (44 seconds ago)
Started:             Mon Dec 24 23:49:25 -0500 (44 seconds ago)
Duration:            44 seconds

STEP                                         PODNAME                      DURATION  MESSAGE
 ● loops-maps-64szb
 └-·-✔ test-linux(0:image:debian,tag:9.1)    loops-maps-64szb-1451238192  16s
   ├-◷ test-linux(1:image:debian,tag:8.9)    loops-maps-64szb-364729162   44s       ContainerCreating
   ├-✔ test-linux(2:image:alpine,tag:3.6)    loops-maps-64szb-892921377   2s
   └-✔ test-linux(3:image:ubuntu,tag:17.10)  loops-maps-64szb-244881556   32s
Name:                loops-maps-64szb
Namespace:           default
ServiceAccount:      default
Status:              Running
Created:             Mon Dec 24 23:49:25 -0500 (45 seconds ago)
Started:             Mon Dec 24 23:49:25 -0500 (45 seconds ago)
Duration:            45 seconds

STEP                                         PODNAME                      DURATION  MESSAGE
 ● loops-maps-64szb
 └-·-✔ test-linux(0:image:debian,tag:9.1)    loops-maps-64szb-1451238192  16s
   ├-◷ test-linux(1:image:debian,tag:8.9)    loops-maps-64szb-364729162   45s       ContainerCreating
   ├-✔ test-linux(2:image:alpine,tag:3.6)    loops-maps-64szb-892921377   2s
   └-✔ test-linux(3:image:ubuntu,tag:17.10)  loops-maps-64szb-244881556   32s
Name:                loops-maps-64szb
Namespace:           default
ServiceAccount:      default
Status:              Running
Created:             Mon Dec 24 23:49:25 -0500 (46 seconds ago)
Started:             Mon Dec 24 23:49:25 -0500 (46 seconds ago)
Duration:            46 seconds

STEP                                         PODNAME                      DURATION  MESSAGE
 ● loops-maps-64szb
 └-·-✔ test-linux(0:image:debian,tag:9.1)    loops-maps-64szb-1451238192  16s
   ├-◷ test-linux(1:image:debian,tag:8.9)    loops-maps-64szb-364729162   46s       ContainerCreating
   ├-✔ test-linux(2:image:alpine,tag:3.6)    loops-maps-64szb-892921377   2s
   └-✔ test-linux(3:image:ubuntu,tag:17.10)  loops-maps-64szb-244881556   32s
Name:                loops-maps-64szb
Namespace:           default
ServiceAccount:      default
Status:              Running
Created:             Mon Dec 24 23:49:25 -0500 (47 seconds ago)
Started:             Mon Dec 24 23:49:25 -0500 (47 seconds ago)
Duration:            47 seconds

STEP                                         PODNAME                      DURATION  MESSAGE
 ● loops-maps-64szb
 └-·-✔ test-linux(0:image:debian,tag:9.1)    loops-maps-64szb-1451238192  16s
   ├-◷ test-linux(1:image:debian,tag:8.9)    loops-maps-64szb-364729162   47s       ContainerCreating
   ├-✔ test-linux(2:image:alpine,tag:3.6)    loops-maps-64szb-892921377   2s
   └-✔ test-linux(3:image:ubuntu,tag:17.10)  loops-maps-64szb-244881556   32s
Name:                loops-maps-64szb
Namespace:           default
ServiceAccount:      default
Status:              Running
Created:             Mon Dec 24 23:49:25 -0500 (48 seconds ago)
Started:             Mon Dec 24 23:49:25 -0500 (48 seconds ago)
Duration:            48 seconds

STEP                                         PODNAME                      DURATION  MESSAGE
 ● loops-maps-64szb
 └-·-✔ test-linux(0:image:debian,tag:9.1)    loops-maps-64szb-1451238192  16s
   ├-◷ test-linux(1:image:debian,tag:8.9)    loops-maps-64szb-364729162   48s       ContainerCreating
   ├-✔ test-linux(2:image:alpine,tag:3.6)    loops-maps-64szb-892921377   2s
   └-✔ test-linux(3:image:ubuntu,tag:17.10)  loops-maps-64szb-244881556   32s
Name:                loops-maps-64szb
Namespace:           default
ServiceAccount:      default
Status:              Running
Created:             Mon Dec 24 23:49:25 -0500 (49 seconds ago)
Started:             Mon Dec 24 23:49:25 -0500 (49 seconds ago)
Duration:            49 seconds

STEP                                         PODNAME                      DURATION  MESSAGE
 ● loops-maps-64szb
 └-·-✔ test-linux(0:image:debian,tag:9.1)    loops-maps-64szb-1451238192  16s
   ├-◷ test-linux(1:image:debian,tag:8.9)    loops-maps-64szb-364729162   49s       ContainerCreating
   ├-✔ test-linux(2:image:alpine,tag:3.6)    loops-maps-64szb-892921377   2s
   └-✔ test-linux(3:image:ubuntu,tag:17.10)  loops-maps-64szb-244881556   32s
Name:                loops-maps-64szb
Namespace:           default
ServiceAccount:      default
Status:              Running
Created:             Mon Dec 24 23:49:25 -0500 (49 seconds ago)
Started:             Mon Dec 24 23:49:25 -0500 (49 seconds ago)
Duration:            49 seconds

STEP                                         PODNAME                      DURATION  MESSAGE
 ● loops-maps-64szb
 └-·-✔ test-linux(0:image:debian,tag:9.1)    loops-maps-64szb-1451238192  16s
   ├-● test-linux(1:image:debian,tag:8.9)    loops-maps-64szb-364729162   49s
   ├-✔ test-linux(2:image:alpine,tag:3.6)    loops-maps-64szb-892921377   2s
   └-✔ test-linux(3:image:ubuntu,tag:17.10)  loops-maps-64szb-244881556   32s
Name:                loops-maps-64szb
Namespace:           default
ServiceAccount:      default
Status:              Running
Created:             Mon Dec 24 23:49:25 -0500 (50 seconds ago)
Started:             Mon Dec 24 23:49:25 -0500 (50 seconds ago)
Duration:            50 seconds

STEP                                         PODNAME                      DURATION  MESSAGE
 ● loops-maps-64szb
 └-·-✔ test-linux(0:image:debian,tag:9.1)    loops-maps-64szb-1451238192  16s
   ├-● test-linux(1:image:debian,tag:8.9)    loops-maps-64szb-364729162   50s
   ├-✔ test-linux(2:image:alpine,tag:3.6)    loops-maps-64szb-892921377   2s
   └-✔ test-linux(3:image:ubuntu,tag:17.10)  loops-maps-64szb-244881556   32s
Name:                loops-maps-64szb
Namespace:           default
ServiceAccount:      default
Status:              Succeeded
Created:             Mon Dec 24 23:49:25 -0500 (50 seconds ago)
Started:             Mon Dec 24 23:49:25 -0500 (50 seconds ago)
Finished:            Mon Dec 24 23:50:15 -0500 (now)
Duration:            50 seconds

STEP                                         PODNAME                      DURATION  MESSAGE
 ✔ loops-maps-64szb
 └-·-✔ test-linux(0:image:debian,tag:9.1)    loops-maps-64szb-1451238192  16s
   ├-✔ test-linux(1:image:debian,tag:8.9)    loops-maps-64szb-364729162   50s
   ├-✔ test-linux(2:image:alpine,tag:3.6)    loops-maps-64szb-892921377   2s
   └-✔ test-linux(3:image:ubuntu,tag:17.10)  loops-maps-64szb-244881556   32s
```
View the logs of the workflow:
```
$ argo logs loops-maps-64szb-1451238192
PRETTY_NAME="Debian GNU/Linux 9 (stretch)"
NAME="Debian GNU/Linux"
VERSION_ID="9"
VERSION="9 (stretch)"
ID=debian
HOME_URL="https://www.debian.org/"
SUPPORT_URL="https://www.debian.org/support"
BUG_REPORT_URL="https://bugs.debian.org/"

$ argo logs loops-maps-64szb-244881556
NAME="Ubuntu"
VERSION="17.10 (Artful Aardvark)"
ID=ubuntu
ID_LIKE=debian
PRETTY_NAME="Ubuntu 17.10"
VERSION_ID="17.10"
HOME_URL="https://www.ubuntu.com/"
SUPPORT_URL="https://help.ubuntu.com/"
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
VERSION_CODENAME=artful
UBUNTU_CODENAME=artful

$ argo logs loops-maps-64szb-364729162
PRETTY_NAME="Debian GNU/Linux 8 (jessie)"
NAME="Debian GNU/Linux"
VERSION_ID="8"
VERSION="8 (jessie)"
ID=debian
HOME_URL="http://www.debian.org/"
SUPPORT_URL="http://www.debian.org/support"
BUG_REPORT_URL="https://bugs.debian.org/"

$ argo logs loops-maps-64szb-892921377
NAME="Alpine Linux"
ID=alpine
VERSION_ID=3.6.3
PRETTY_NAME="Alpine Linux v3.6"
HOME_URL="http://alpinelinux.org"
BUG_REPORT_URL="http://bugs.alpinelinux.org"
```

