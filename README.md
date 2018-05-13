# Running Selenium Grid on Kubernetes cluster
In this tutorial, I’ll show you how to get Selenium Grid up and running on Kubernetes cluster using Helm package manager.

## **Contents**

01. **[Prerequisites](#prerequisites)**
    - **[Verify the Installation](#verify-the-installation)**
02. **[Selenium Grid](#selenium-grid)**
02. **[Kubernetes](#kubernetes)**
    - **[Helm](#helm)**
03. **[Selenium-Grid Installation](#selenium-grid-installation)**
04. **[Run Selenium tests](#run-selenium-test)**
05. **[Debugging with VNC Viewer](#debugging-with-vnc-viewer)**
06. **[Deletes a release](#deletes-a-release-from-kubernetes)**

&nbsp;

![](https://github.com/sayems/selenium.grid/blob/master/images/selenium.png)


### **Selenium Grid**
Selenium Grid makes automation execution jobs much easier. Using Selenium Grid, one can run multiple tests on multiple machines in parallel, which reduces execution times from days to hours. However, setting up our own Selenium Grid means we have to configure browsers across multiple machines, virtual or physical, and making sure each machine is running the Selenium Server correctly. This also means maintaining the Grid and making updates would be a time-consuming. This is where Kubernetes comes in and saves the day!

### **Kubernetes**
Kubernetes is an open-source system for automating deployment, scaling, and management of containerized applications.

### **Helm** 
Helm is a package manager for Kubernetes applications, responsible for installing and managing Kubernetes applications. Helm packages all of the different Kubernetes resources (such as deployments, services, and ingress) into a chart, which may be hosted in a repository. Users can pull down charts and install them on any number of Kubernetes clusters. This is similar to Homebrew and apt-get package manager. We will use Helm to deploy selenium-grid on Kubernetes Cluster.


###### **What We’ll Accomplish**
1. Set up a Kubernetes cluster using Docker for Mac 
2. Deploy Selenium-Grid to the cluster
3. Running selenium test and viewing your Selenium tests running on Kubernetes cluster

## Prerequisites
- [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/) - macOS users can use ``` brew install kubectl``` to install 
- [Docker for Mac](https://download.docker.com/mac/edge/Docker.dmg) - Docker with Kubernetes is currently only available on Docker for Mac in the latest Edge version
- [Virtualbox](https://www.virtualbox.org/wiki/Downloads) - macOS users can use ```brew cask install virtualbox``` to install
- [Helm](https://github.com/kubernetes/helm/blob/master/docs/install.md) - macOS users can use ```brew install kubernetes-helm``` to install 


### Verify the Installation

    docker --version                # Docker version 17.09.0-ce, build afdb6d4
    docker-compose --version        # docker-compose version 1.16.1, build 6d1ac21
    docker-machine --version        # docker-machine version 0.12.2, build 9371605
    kubectl version --client        # Client Version: version.Info{Major:"1", Minor:"8", GitVersion:"v1.8.1", GitCommit:"f38e43b221d08850172a9a4ea785a86a3ffa3b3a", GitTreeState:"clean", BuildDate:"2017-10-12T00:45:05Z", GoVersion:"go1.9.1", Compiler:"gc", Platform:"darwin/amd64"} 
    helm version --client           # Client: &version.Version{SemVer:"v2.9.0", GitCommit:"f6025bb9ee7daf9fee0026541c90a6f557a3e0bc", GitTreeState:"clean"}
   
  
   
#### Enabling the local Kubernetes cluster
Click the Docker icon in the status bar, go to “****Preferences****”, and on the “****Kubernetes****”-tab, check “****Enable Kubernetes****”. This will start a single node Kubernetes cluster for you and install the kubectl command line utility. This might take a while, but the dialog will let you know once the Kubernetes cluster is ready.

![](https://github.com/sayems/selenium.grid/blob/master/images/enable-kubernetes.png)
    
##### Next step, you’ll need to increase Docker’s available memory to 4GB or more.
    
Click the Docker icon in the status bar, go to “****Preferences****”, and on the “****Advanced****”-tab, change memoery to 4GB or more.

![](https://github.com/sayems/selenium.grid/blob/master/images/increase-memeory.png)

&nbsp;


## **Selenium Grid installation**

To install the chart with the release name `selenium-grid`:

```
helm install --name selenium-grid stable/selenium \
--set chromeDebug.enabled=true \
--set firefoxDebug.enabled=true
```

The command deploys Selenium Grid on the Kubernetes cluster in the default configuration with chromeDebug and firefoxDebug enabled.

**Note**: You should see the following output after executing ```helm install ...```

```
NAME:   selenium-grid
LAST DEPLOYED: Fri May 11 23:09:29 2018
NAMESPACE: default
STATUS: DEPLOYED

RESOURCES:
==> v1/Service
NAME                        TYPE          CLUSTER-IP      EXTERNAL-IP  PORT(S)         AGE
selenium-grid-selenium-hub  LoadBalancer  10.110.225.161  localhost    4444:30665/TCP  0s

==> v1beta1/Deployment
NAME                                  DESIRED  CURRENT  UP-TO-DATE  AVAILABLE  AGE
selenium-grid-selenium-chrome-debug   1        1        1           0          0s
selenium-grid-selenium-firefox-debug  1        1        1           0          0s
selenium-grid-selenium-hub            1        1        1           0          0s

==> v1/Pod(related)
NAME                                                   READY  STATUS             RESTARTS  AGE
selenium-grid-selenium-chrome-debug-6d8898d7c4-7psdh   0/1    ContainerCreating  0         0s
selenium-grid-selenium-firefox-debug-84f6f484d8-wh66d  0/1    ContainerCreating  0         0s
selenium-grid-selenium-hub-f7f874cd-p5lrd              0/1    ContainerCreating  0         0s
```

Selenium hub will automatically start-up using port 4444 by default. You can view the status of the hub by opening a browser window and navigating to: http://localhost:4444/grid/console

![](https://github.com/sayems/selenium.grid/blob/master/images/gird-console.png)

### Run Selenium Test
Let's run a quick Selenium test to validate our setup.

##### Clone this github repo:
```
git clone https://github.com/sayems/selenium.grid.git
```

##### Go into the project
```
$ cd selenium.grid
```

##### Run tests
Before you run ```docker-compose up```, verify Selenium grid is up & running on Kubernetes cluster, you can verify by executing the following command: ```$ kubectl get pods```

```
$ docker-compose up
```

It typically takes several minutes for docker setup for the first time, so wait for few minutes …

### Debugging with VNC Viewer

It is useful to be able to view your Selenium tests running on Kubernetes cluster. This can help us to debug issues better and have more confidence in our tests. To view running tests on Kubernetes cluster.

1. [Download VNC® Viewer for Google Chrome](https://chrome.google.com/webstore/detail/vnc%C2%AE-viewer-for-google-ch/iabmpiboiopbgfabjmgeedhcmjenhbla?hl=en)
2. Execute the following command to bind ports between Kubernetes cluster to the local machine:

```
  kubectl port-forward --namespace default \
  $(kubectl get pods --namespace default \
    -l app=selenium-grid-selenium-chrome-debug \
    -o jsonpath='{ .items[0].metadata.name }') 5900
```
3. Open VNC® Viewer for Google Chrome
    - **Address**: 127.0.0.1:5900
    - **Password**: secret

&nbsp;

![](https://github.com/sayems/selenium.grid/blob/master/images/vnc.png)


### Deletes a release from Kubernetes

The helm list function will show you a list of all deployed releases.
```
$ helm list
```

List of all releases
```
NAME         	 REVISION	 UPDATED   STATUS  	 CHART     NAMESPACE
selenium-grid	 1       	 Fri May 	 DEPLOYED	 selenium	 default
```

If you want to delete selenium-grid, you can execute the following command
```
$ helm delete selenium-grid
release "selenium-grid" deleted
```

If you want to delete all Helm releases, you can pipe the output of helm ls --short to xargs, and run helm delete for each release name.
```
$ helm ls --short | xargs -L1 helm delete
```

## Configuration

The following table lists the configurable parameters of the Selenium chart and their default values.

| Parameter | Description | Default |
| --------- | ----------- | ------- |
| `global.nodeselector` | Node label to be useed globally for scheduling of all images | `nil` |
| `hub.image` | The selenium hub image | `selenium/hub` |
| `hub.tag` | The selenium hub image tag | `3.11.0` |
| `hub.pullPolicy` | The pull policy for the hub image | `IfNotPresent` |
| `hub.port` | The port the hub listens on | `4444` |
| `hub.javaOpts` | The java options for the selenium hub JVM, default sets the maximum heap size to 1,000 mb | `-Xmx1000m` |
| `hub.resources` | The resources for the hub container, defaults to minimum half a cpu and maximum 1,000 mb RAM | `{"limits":{"cpu":".5", "memory":"1000Mi"}}` |
| `hub.serviceType` | The Service type | `NodePort` |
| `hub.serviceSessionAffinity` | The session affinity for the hub service| `None` |
| `hub.gridNewSessionWaitTimeout` | | `nil` |
| `hub.gridJettyMaxThreads` | | `nil` |
| `hub.gridNodePolling` | | `nil` |
| `hub.gridCleanUpCycle` | | `nil` |
| `hub.gridTimeout` | | `nil` |
| `hub.gridBrowserTimeout` | | `nil` |
| `hub.gridMaxSession` | | `nil` |
| `hub.gridUnregisterIfStillDownAfer` | | `nil` |
| `hub.seOpts` | Command line arguments to pass to hub | `nil` |
| `hub.timeZone` | The time zone for the container | `nil` |
| `hub.nodeselector` | Node label to use for scheduling of the hub if set this takes precedence over the global value | `nil` |
| `chrome.enabled` | Schedule a chrome node pod | `false` |
| `chrome.image` | The selenium node chrome image | `selenium/node-chrome` |
| `chrome.tag` | The selenium node chrome tag | `3.11.0` |
| `chrome.pullPolicy` | The pull policy for the node chrome image | `IfNotPresent` |
| `chrome.replicas` | The number of selenium node chrome pods | `1` |
| `chrome.javaOpts` | The java options for the selenium node chrome JVM, default sets the maximum heap size to 900 mb | `-Xmx900m` |
| `chrome.volumeMounts` | Additional volumes to mount, the default provides a larger shared memory | `[{"mountPath":"/dev/shm", "name":"dshm"}]` |
| `chrome.volumes` | Additional volumes import, the default provides a larger shared memory | `[{"name":"dshm", "emptyDir":{"medium":"Memory"}}]` |
| `chrome.resources` | The resources for the node chrome container, defaults to minimum half a cpu and maximum 1,000 mb | `{"limits":{"cpu":".5", "memory":"1000Mi"}}` |
| `chrome.screenWidth` | | `nil` |
| `chrome.screenHeight` | | `nil` |
| `chrome.screenDepth` | | `nil` |
| `chrome.display` | The vnc display | `nil` |
| `chrome.chromeVersion` | The version of chrome to use | `nil` |
| `chrome.nodeMaxInstances` | The maximum number of browser instances | `nil` |
| `chrome.nodeMaxSession` | The maximum number of sessions | `nil` |
| `chrome.nodeRegistryCycle` | The number of milliseconds to wait, registering a node| `nil` |
| `chrome.nodePort` | The port to listen on | `nil` |
| `chrome.seOpts` | Command line arguments to pass to node | `nil` |
| `chrome.timeZone` | The time zone for the container | `nil` |
| `chrome.nodeselector` | Node label to use for scheduling of chrome images if set this takes precedence over the global value | `nil` |
| `chromeDebug.enabled` | Schedule a selenium node chrome debug pod | `false` |
| `chromeDebug.image` | The selenium node chrome debug image | `selenium/node-chrome-debug` |
| `chromeDebug.tag` | The selenium node chrome debug tag | `3.11.0` |
| `chromeDebug.pullPolicy` | The selenium node chrome debug pull policy | `IfNotPresent` |
| `chromeDebug.replicas` | The number of selenium node chrome debug pods | `1` |
| `chromeDebug.javaOpts` | The java options for a selenium node chrome debug JVM, default sets the max heap size to 900 mb | `-Xmx900m` |
| `chromeDebug.volumeMounts` | Additional volumes to mount, the default provides a larger shared | `[{"mountPath":"/dev/shm", "name":"dshm"}]` |
| `chromeDebug.volumes` | Additional volumes import, the default provides a larger shared | `[{"name":"dshm", "emptyDir":{"medium":"Memory"}}]` |
| `chromeDebug.resources` | The resources for the hub container, defaults to minimum half a cpu and maximum 1,000 mb | `{"limits":{"cpu":".5", "memory":"1000Mi"}}` |
| `chromeDebug.screenWidth` | | `nil` |
| `chromeDebug.screenHeight` | | `nil` |
| `chromeDebug.screenDepth` | | `nil` |
| `chromeDebug.display` | The vnc display | `nil` |
| `chromeDebug.chromeVersion` | The version of chrome to use | `nil` |
| `chromeDebug.nodeMaxInstances` | The maximum number of browser instances | `nil` |
| `chromeDebug.nodeMaxSession` | The maximum number of sessions | `nil` |
| `chromeDebug.nodeRegistryCycle` | The number of milliseconds to wait, registering a node| `nil` |
| `chromeDebug.nodePort` | The port to listen on | `nil` |
| `chromeDebug.seOpts` | Command line arguments to pass to node | `nil` |
| `chromeDebug.timeZone` | The time zone for the container | `nil` |
| `chromeDebug.nodeselector` | Node label to use for scheduling of chromeDebug images if set this takes precedence over the global value | `nil` |
| `firefox.enabled` | Schedule a selenium node firefox pod | `false` |
| `firefox.image` | The selenium node firefox image | `selenium/node-firefox` |
| `firefox.tag` | The selenium node firefox tag | `3.11.0` |
| `firefox.pullPolicy` | The selenium node firefox pull policy | `IfNotPresent` |
| `firefox.replicas` | The number of selenium node firefox pods | `1` |
| `firefox.javaOpts` | The java options for a selenium node firefox JVM, default sets the max heap size to 900 mb | `-Xmx900m` |
| `firefox.resources` | The resources for the hub container, defaults to minimum half a cpu and maximum 1,000 mb | `{"limits":{"cpu":".5", "memory":"1000Mi"}}` |
| `firefox.screenWidth` | | `nil` |
| `firefox.screenHeight` | | `nil` |
| `firefox.screenDepth` | | `nil` |
| `firefox.display` | The vnc display | `nil` |
| `firefox.firefoxVersion` | The version of firefox to use | `nil` |
| `firefox.nodeMaxInstances` | The maximum number of browser instances | `nil` |
| `firefox.nodeMaxSession` | The maximum number of sessions | `nil` |
| `firefox.nodeRegistryCycle` | The number of milliseconds to wait, registering a node| `nil` |
| `firefox.nodePort` | The port to listen on | `nil` |
| `firefox.seOpts` | Command line arguments to pass to node | `nil` |
| `firefox.timeZone` | The time zone for the container | `nil` |
| `firefox.nodeselector` | Node label to use for scheduling of firefox images if set this takes precedence over the global value | `nil` |
| `firefoxDebug.enabled` | Schedule a selenium node firefox debug pod | `false` |
| `firefoxDebug.image` | The selenium node firefox debug image | `selenium/node-firefox-debug` |
| `firefoxDebug.tag` | The selenium node firefox debug tag | `3.11.0` |
| `firefoxDebug.pullPolicy` | The selenium node firefox debug pull policy | `IfNotPresent` |
| `firefoxDebug.replicas` | The numer of selenium node firefox debug pods | `1` |
| `firefoxDebug.javaOpts` | The java options for a selenium node firefox debug JVM, default sets the max heap size to 900 mb | `-Xmx900m` |
| `firefoxDebug.resources` | The resources for the selenium node firefox debug container, defaults to minimum half a cpu and maximum 1,000 mb | `{"limits":{"cpu":".5", "memory":"1000Mi"}}` |
| `firefoxDebug.screenWidth` | | `nil` |
| `firefoxDebug.screenHeight` | | `nil` |
| `firefoxDebug.screenDepth` | | `nil` |
| `firefoxDebug.display` | The vnc display | `nil` |
| `firefoxDebug.firefoxVersion` | The version of firefox to use | `nil` |
| `firefoxDebug.nodeMaxInstances` | The maximum number of browser instances | `nil` |
| `firefoxDebug.nodeMaxSession` | The maximum number of sessions | `nil` |
| `firefoxDebug.nodeRegistryCycle` | The number of milliseconds to wait, registering a node| `nil` |
| `firefoxDebug.nodePort` | The port to listen on | `nil` |
| `firefoxDebug.seOpts` | Command line arguments to pass to node | `nil` |
| `firefoxDebug.timeZone` | The time zone for the container | `nil` |
| `firefoxDebug.nodeselector` | Node label to use for scheduling of firefoxDebug images if set this takes precedence over the global value | `nil` |

Specify each parameter using the `--set key=value[,key=value]` argument to `helm install`. For example,

```console
$ helm install --name my-release \
  --set chrome.enabled=true \
    stable/selenium
```
