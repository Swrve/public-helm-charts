# honey-aws-lb

Helm chart to run [honeycomb.io](https://docs.honeycomb.io/getting-data-in/integrations/aws/aws-elastic-load-balancer/) AWS ELB/ALB log ingestion. The chart creates a single-pod Kubernetes deployment with multiple containers, one for each Load Balancer to be monitored, using the Honeycomb docker image from https://hub.docker.com/r/honeycombio/honeyaws

## Requirements

### Load Balancer Logging enabled

AWS Load balancers to be monitored will need to have logging enabled. This can be done in the AWS Load Balancer console.  You will need to specify an S3 bucket and path to write to. The pod will need read access to that bucket.

### AWS IAM credentials

If you are running this on AWS EKS, you can use IAM instance roles to allow the EC2 nodes to read the S3 bucket the ELB/ALB writes logs to as specified above.

If you prefer to use AWS IAM credentials directly, you can use the default AWS environment variable and add them to the helm chart by setting `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY`  in the extra_envs value, using pipe separatad values e.g:

`--set extra_env="AWS_ACCESS_KEY_ID=AKAIxxxxxxxx|AWS_SECRET_ACCESS_KEY=xxx"`

**Note**: Setting IAM access keys in the environment of a Helm chart is insecure. You should use IAM instance roles if possible.


## Adding a Load Balancer to Honeycomb

Update `values.yaml` to add the configuration for the load balanacers you want to monitor. You can also include this in a separate values file and call with `-f` when you install the chart.

The child keys of honeyMonitoredLoadBalancers are the names of your load balancers as viewed on the AWS console. You need to specify whether it is a classic Elastic Load Balancer (type: elb) or an Application Load Balancer (type: alb). Your writekey is from your Honeycomb account. Specify a samplerate `n` suitable for your needs. E.g, 20 for sampling 1 out of 20 log entries.

```
honeyMonitoredLoadBalancers:
  <name-of-load-balancer-in-AWS>:
    type: "<alb|elb>"
    dataset: "<dataset as viewed on Honeycomb>"
    samplerate: n
    writekey: "<writekey from honeycomb"
  my-load-balancer:
    type: "elb"
    dataset: "my-load-balancer"
    samplerate: 20
    writekey: "aaaazzzz11119999"
```

Once added, you can install the helm chart:

```
$ kubectl config use <your kubernetes cluster context>
$ helm upgrade honey-aws-lb honey-aws-lb honey-aws-lb.yaml --namespace mon
itoring
```




