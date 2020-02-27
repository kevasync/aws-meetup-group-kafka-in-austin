# AWS Meetup Group
## Kafka Stream Processing Overview and Demo

### Install dependencies:
* [kubectl - CLI to interact with k8s](https://kubernetes.io/docs/tasks/tools/install-kubectl/)
* [helm - packaging resource that desecribes k8s resources](https://helm.sh/docs/intro/install/)
* [aws-cli - CLI to interact with AWS](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-install.html)
* [pulumi-cli - CLI interface to create infra with Pulumi](https://www.pulumi.com/docs/get-started/install/)
* [aws iam-authenticator - Used by pulumi to access AWS using your credentials](https://docs.aws.amazon.com/eks/latest/userguide/install-aws-iam-authenticator.html)
* [node.js - javascript all the things](https://nodejs.org/en/download/)
* [watch - continuously run a commands](https://osxdaily.com/2010/08/22/install-watch-command-on-os-x/)

### Create CLI account and set up credentials
* Create user in IAM
* run `aws configure` - I am using US West 2 region, it supports EKS
    * So does Northern VA and Ohio (US East 1 and 2)

### Log into Pulumi
* `pulumi login`

### Create EKS Cluster Pulumi Stack 
* `mkdir aws-eks-infra && cd aws-eks-infra`
* `pulumi new https://github.com/wwt/pulumi-templates/aws-eks-infra`
* Use the default options
* `pulumi up`
* What are we creating?  Let's take a look at the [pulumi file](https://github.com/wwt/pulumi-templates/blob/master/aws-eks-infra/index.ts)...
* This will take a bout 5 minutes
* export the k8s config so we can do things with kubectl `pulumi stack output kubeconfig > ~/.kube/config`
* Take a ppek at the cluster: `watch kubectl get pods --namespace kafka`
    * Don't like typing the namespace? try `kubens`


### Create Kafka Cluster using Confluent Operator
* `mkdir confluent-kafka-operator && cd confluent-kafka-operator`
* `pulumi new https://github.com/wwt/pulumi-templates/confluent-kafka-operator`
    * Trial license key: `eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJhdWQiOiJ3d3QtYnJpYW4iLCJleHAiOjE1ODUwMDgwMDAsImlhdCI6MTU4MjUwMjQwMCwiaXNzIjoiQ29uZmx1ZW50IiwibW9uaXRvcmluZyI6dHJ1ZSwibmI0IjoxNTgyNTcxODk4LCJzdWIiOiJjb250cm9sLWNlbnRlciJ9.adOegJ39U_HZGLEEosvgXkPaf7leHXRRkPFdnpoF_7Dd3OYEQLuHoVV0jqsug9QnRBZSqsXl_9yb0edG-Lg8IZTdyBCsdLahPEpV4y9-8fSMRCb_HhoTzDoQYBa3hnMy2SsGVmfQsqrjOIBM59aijWFMDmstzg4xSU8tc-x4l6Azl-VGzRyR7EkeEpyevWSSungIZF9Up4kGJ9jN6bw5pQc3EKbNEM4A3XkmtH_YQw6VEIkK4Fvx4iJfXBOe5A6-2ei7-CdoSKSTDeNrUIJKYdwPo6qPGIkpjYCudilqDUo5i9tnlNs-Q8zLVOfRVQv0S8rObRFCjT5GS-ldnYAeMw`
    * Only enable the Confluent Operator the first time
* `pulumi up --skip-preview`
* Keep an eye on the pods: `watch kubectl get pods --namespace kafka`
* Edit `Pulumi.dev.yaml` and enable Zookeeper: `confluent-kafka-operator:enableZookeeper: "true"`
    * You can also do this via CLI: `pulumi config set enableZookeeper true`
* `pulumi up --skip-preview`
* Edit `Pulumi.dev.yaml` and enable Kafka Brokers: `confluent-kafka-operator:enableKafka: "true"`
    * You can also do this via CLI: `pulumi config set enableZookeeper true`
* `pulumi up --skip-preview`
* Enable all of the other components in the config, except replicator (May need more nodes than 8 in the k8s clsuter if we want to enable this)
* `pulumi up --skip-preview`
* Keep an eye on the k8s pods to see the operator spinning components up
* Forward Control Center port and take a look at the cluster
    * `kubectl port-forward svc/controlcenter 9021:9021`
    * [Check out the cluster](http://localhost:9021/)


### Destroy Resouces
* Turn all features off besides Confluent Operator in `Pulumi.dev.yaml` in `confluent-kafka-operator` directory:
```
  confluent-kafka-operator:enableConnect: "false"
  confluent-kafka-operator:enableControlCenter: "false"
  confluent-kafka-operator:enableKafka: "false"
  confluent-kafka-operator:enableKsql: "false"
  confluent-kafka-operator:enableOperator: "true"
  confluent-kafka-operator:enableReplicator: "false"
  confluent-kafka-operator:enableSchemaRegistry: "false"
  confluent-kafka-operator:enableZookeeper: "false"
```
* `pulumi up --skip-preview`
* In `pulumi destroy --skip-preview` in the following directories:
    * `aws-eks-infra`
    * `confluent-kafka-operator`






