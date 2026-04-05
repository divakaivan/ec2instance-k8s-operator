# EC2Instance Operator

A Kubernetes operator that manages the lifecycle of AWS EC2 instances as native Kubernetes resources. Define, create, and delete EC2 instances using standard `kubectl` commands — no AWS console or CLI required.

## Description

The operator introduces a custom resource called **EC2Instance**. When you apply an `EC2Instance` manifest to your cluster, the operator:

1. Calls the AWS EC2 API to launch an instance with the requested configuration (AMI, instance type, key pair, subnet, etc.)
2. Waits for the instance to reach a running state
3. Writes the instance details (ID, public/private IP, DNS names, state) back to the resource's `status`
4. Adds a finalizer to ensure the EC2 instance is terminated before the Kubernetes resource is deleted

When you delete the `EC2Instance` resource, the operator terminates the corresponding AWS EC2 instance and removes the finalizer, keeping your AWS environment clean.

### Example

```yaml
apiVersion: compute.cloud.com/v1
kind: EC2Instance
metadata:
  name: my-instance
  namespace: default
spec:
  instanceType: t3.micro
  amiId: ami-0c02fb55956c7d316
  region: us-east-1
  keyPair: my-key-pair
  subnet: subnet-0123456789abcdef0
```

Apply it, then watch the status populate with the instance ID, IP addresses, and state:

```sh
kubectl apply -f my-instance.yaml
kubectl get ec2instances -w
```

## Getting Started

### Prerequisites
- go version v1.24.6+
- docker version 17.03+
- kubectl version v1.11.3+
- Access to a Kubernetes v1.11.3+ cluster
- AWS credentials (`AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`) with EC2 permissions

### To Deploy on the cluster
**Build and push your image to the location specified by `IMG`:**

```sh
make docker-build docker-push IMG=<some-registry>/k8s-operator:tag
```

**NOTE:** This image ought to be published in the personal registry you specified.
And it is required to have access to pull the image from the working environment.
Make sure you have the proper permission to the registry if the above commands don’t work.

**Install the CRDs into the cluster:**

```sh
make install
```

**Deploy the Manager to the cluster with the image specified by `IMG`:**

```sh
make deploy IMG=<some-registry>/k8s-operator:tag
```

> **NOTE**: If you encounter RBAC errors, you may need to grant yourself cluster-admin
privileges or be logged in as admin.

**Create instances of your solution**
You can apply the samples (examples) from the config/sample:

```sh
kubectl apply -k config/samples/
```

>**NOTE**: Ensure that the samples has default values to test it out.

### To Uninstall
**Delete the instances (CRs) from the cluster:**

```sh
kubectl delete -k config/samples/
```

**Delete the APIs(CRDs) from the cluster:**

```sh
make uninstall
```

**UnDeploy the controller from the cluster:**

```sh
make undeploy
```

## Project Distribution

Following the options to release and provide this solution to the users.

### By providing a bundle with all YAML files

1. Build the installer for the image built and published in the registry:

```sh
make build-installer IMG=<some-registry>/k8s-operator:tag
```

**NOTE:** The makefile target mentioned above generates an 'install.yaml'
file in the dist directory. This file contains all the resources built
with Kustomize, which are necessary to install this project without its
dependencies.

2. Using the installer

Users can just run 'kubectl apply -f <URL for YAML BUNDLE>' to install
the project, i.e.:

```sh
kubectl apply -f https://raw.githubusercontent.com/<org>/k8s-operator/<tag or branch>/dist/install.yaml
```

### By providing a Helm Chart

The Helm chart is pre-generated under `dist/chart` and uses the image `divakaivan/k8s-operator:latest` from DockerHub.

Install it with:

```sh
helm install ec2-operator ./dist/chart \
  --namespace ec2-operator-system \
  --create-namespace \
  --set manager.env[0].name=AWS_ACCESS_KEY_ID \
  --set manager.env[0].value=<your-access-key-id> \
  --set manager.env[1].name=AWS_SECRET_ACCESS_KEY \
  --set manager.env[1].value=<your-secret-access-key>
```

**NOTE:** If you change the project, regenerate the chart by re-running:

```sh
kubebuilder edit --plugins=helm/v2-alpha
```

If you have webhooks or custom `values.yaml` changes, use `--force` and re-apply your customisations afterwards.

## CI/CD

On every push to `main`, the [Publish workflow](.github/workflows/publish.yml) builds the operator image and pushes it to DockerHub as `divakaivan/k8s-operator:latest`. The Helm chart's `values.yaml` is pre-configured to pull this image.

To enable the workflow, add the following secrets to your GitHub repository:

| Secret | Description |
|--------|-------------|
| `DOCKERHUB_USERNAME` | Your DockerHub username |
| `DOCKERHUB_TOKEN` | A DockerHub access token |

## Contributing
// TODO(user): Add detailed information on how you would like others to contribute to this project

**NOTE:** Run `make help` for more information on all potential `make` targets

More information can be found via the [Kubebuilder Documentation](https://book.kubebuilder.io/introduction.html)

## License

Copyright 2026.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.

