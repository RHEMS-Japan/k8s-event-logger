# Kubernetes event logger for rhems logsystem

<img src="https://raw.githubusercontent.com/max-rocket-internet/k8s-event-logger/master/img/k8s-logo.png" width="100">

This tool simply watches Kubernetes Events and logs them to stdout in JSON to be collected and stored by your logging solution, e.g. [fluentd](https://github.com/fluent/fluentd-kubernetes-daemonset) or [fluent-bit](https://fluentbit.io/). Other tools exist for persisting Kubernetes Events, such as Sysdig, Datadog or Google's [event-exporter](https://github.com/GoogleCloudPlatform/k8s-stackdriver/tree/master/event-exporter) but this tool is open and will work with any logging solution.

thanks to max-rocket-internet and RHEMS Japan is customizing for rhems logsystem

### Why?

Events in Kubernetes log very important information. If are trying to understand what happened in the past then these events show clearly what your Kubernetes cluster was thinking and doing. Some examples:

- Pod events like failed probes, crashes, scheduling related information like `TriggeredScaleUp` or `FailedScheduling`
- HorizontalPodAutoscaler events like scaling up and down
- Deployment events like scaling in and out of ReplicaSets
- Ingress events like create and update

The problem is that these events are simply API objects in Kubernetes and are only stored for about 1 hour. Without some way of storing these events, debugging a problem in the past very tricky.

Example of events:

```
39m   Normal  UpdatedLoadBalancer      Service     Updated load balancer with new hosts
40m   Normal  SuccessfulDelete         DaemonSet   Deleted pod: ingress02-nginx-ingress-controller-vqqjp
41m   Normal  ScaleDown                Node        node removed by cluster autoscaler
54m   Normal  Started                  Pod         Started container
55m   Normal  Starting                 Node        Starting kubelet.
55m   Normal  Starting                 Node        Starting kube-proxy.
55m   Normal  NodeAllocatableEnforced  Node        Updated Node Allocatable limit across pods
55m   Normal  NodeReady                Node        Node ip-10-0-23-14.compute.internal status is now: NodeReady
58m   Normal  SuccessfulCreate         DaemonSet   Created pod: ingress02-nginx-ingress-controller-bz7xj
58m   Normal  CREATE                   ConfigMap   ConfigMap default/ingress02-nginx-ingress-controller
```

### Installation

Use the [Helm](https://helm.sh/) chart:

```
helm install chart/
```

### Testing

Run it:

```
go run main.go
```

#### sample

```json
{
  "metadata": {
    "name": "bluedeploy-checker-1631119500.16a2e68e24396680",
    "namespace": "deploy-checker",
    "selfLink": "/api/v1/namespaces/deploy-checker/events/bluedeploy-checker-1631119500.16a2e68e24396680",
    "uid": "c6ca1b93-bb85-454e-8ac8-047a5947db6d",
    "resourceVersion": "113459993",
    "creationTimestamp": "2021-09-08T16:45:13Z",
    "managedFields": [
      {
        "manager": "kube-controller-manager",
        "operation": "Update",
        "apiVersion": "v1",
        "time": "2021-09-08T16:45:13Z",
        "fieldsType": "FieldsV1",
        "fieldsV1": {
          "f:count": {},
          "f:firstTimestamp": {},
          "f:involvedObject": {
            "f:apiVersion": {},
            "f:kind": {},
            "f:name": {},
            "f:namespace": {},
            "f:resourceVersion": {},
            "f:uid": {}
          },
          "f:lastTimestamp": {},
          "f:message": {},
          "f:reason": {},
          "f:source": {
            "f:component": {}
          },
          "f:type": {}
        }
      }
    ]
  },
  "involvedObject": {
    "kind": "Job",
    "namespace": "deploy-checker",
    "name": "bluedeploy-checker-1631119500",
    "uid": "e92cac1a-925e-45fa-a2c7-c8a466df728e",
    "apiVersion": "batch/v1",
    "resourceVersion": "113459936"
  },
  "reason": "Completed",
  "message": "Job completed",
  "source": {
    "component": "job-controller"
  },
  "firstTimestamp": "2021-09-08T16:45:13Z",
  "lastTimestamp": "2021-09-08T16:45:13Z",
  "count": 1,
  "type": "Normal",
  "eventTime": null,
  "reportingComponent": "",
  "reportingInstance": ""
}
```

log sample: <creationTimestamp [  firstTimestamp - lastTimestamp ] NAMESPACE  TYPE REASON OBJECT MESSAGE>

```json
{
"filename":"k8s-event.log",
"log":"<message>",
"namespace":"<namespace>",
"nodename":"-",
"podip":"-",
"tagname":"k8s-event-<type>",
"time":"<time>"
}
```

```json
{
"filename":"k8s-event-raw.log",
"log":"<message>",
"namespace":"<namespace>",
"nodename":"-",
"podip":"-",
"tagname":"k8s-event-raw",
"time":"<time>"
}
```

```terminal
NAMESPACE        LAST SEEN   TYPE      REASON                   OBJECT                                                MESSAGE
deploy-checker   59m         Normal    Scheduled                pod/bluedeploy-checker-1631116620-rlrh5               Successfully assigned deploy-checker/bluedeploy-checker-1631116620-rlrh5 to ip-10-224-108-224.ap-northeast-1.compute.internal
deploy-checker   59m         Normal    Pulling                  pod/bluedeploy-checker-1631116620-rlrh5               Pulling image "rhemsjapan/deploy-check"
deploy-checker   59m         Normal    Pulled                   pod/bluedeploy-checker-1631116620-rlrh5               Successfully pulled image "rhemsjapan/deploy-check" in 1.818543796s
deploy-checker   59m         Normal    Created                  pod/bluedeploy-checker-1631116620-rlrh5               Created container rotator
deploy-checker   59m         Normal    Started                  pod/bluedeploy-checker-1631116620-rlrh5               Started container rotator
deploy-checker   59m         Normal    SuccessfulCreate         job/bluedeploy-checker-1631116620                     Created pod: bluedeploy-checker-1631116620-rlrh5
deploy-checker   59m         Normal    Completed                job/bluedeploy-checker-1631116620                     Job completed
```
