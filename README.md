# Overview

Konnect is an API lifecycle management platform designed from the ground up for the cloud native era and delivered as a service. This platform lets you build modern applications better, faster, and more securely. The management plane is hosted in the cloud by Kong, while the runtime engine, Kong Gateway — Kong’s lightweight, fast, and flexible API gateway — is managed by you within your preferred network environment.

To learn more about Kong Konnect, see the [Kong Konnect](https://docs.konghq.com/konnect/).

The Kong Konnect platform provides a cloud control plane (CP), which manages all service configurations. It propagates those configurations to all runtime nodes, which use in-memory storage. These nodes can be installed anywhere, on-premise or in the cloud.

This application is pre-configured with an SSL certificate. While you are
installing the application using the steps below, you must replace the
certificate with your own valid SSL certificate.

## About Google Click to Deploy

Popular open stacks on Kubernetes packaged by Google.

## Architecture

![Architecture diagram](resources/konnect-intro.png)

Runtime instances, acting as data planes, listen for traffic on the proxy port 443 by default. The Konnect data plane evaluates incoming client API requests and routes them to the appropriate backend APIs. While routing requests and providing responses, policies can be applied with plugins as necessary.

For example, before routing a request, the client might be required to authenticate. This delivers several benefits, including:

The service doesn’t need its own authentication logic since the data plane is handling authentication.
The service only receives valid requests and therefore cycles are not wasted processing invalid requests.
All requests are logged for central visibility of traffic.

# Installation

## Setup a Konnect account and configure A Runtime
Kong Gateway data planes proxy service traffic. With Konnect Cloud working as the control plane, a runtime doesn’t need a database to store configuration data. Instead, configuration is stored in-memory on each node, and you can easily update multiple runtimes from one Konnect account with a few clicks.

Kong Gateway data planes proxy service traffic.

1. From the left navigation menu, open runtimes icon Runtime Manager.

2. Select the default runtime group.

3. Every account starts with one group named default. If you have an Enterprise subscription, you can create additional custom groups.

4. Click Add runtime instance.

The page opens to a runtime configuration form with the Docker tab selected. Here you will find variables that you would need to use on the next steps.


## Quick install with Google Cloud Marketplace

Get up and running with a few clicks! Install this Konnect app to a Google
Kubernetes Engine cluster using Google Cloud Marketplace. Follow the
[on-screen instructions](https://console.cloud.google.com/marketplace/details/google/konnect).

## Command line instructions

You can use [Google Cloud Shell](https://cloud.google.com/shell/) or a local
workstation to complete the following steps.

### Prerequisites

#### Set up command-line tools

You'll need the following tools in your development environment. If you are
using Cloud Shell, `gcloud`, `kubectl`, Docker, and Git are installed in your
environment by default.

-   [gcloud](https://cloud.google.com/sdk/gcloud/)
-   [kubectl](https://kubernetes.io/docs/reference/kubectl/overview/)
-   [docker](https://docs.docker.com/install/)
-   [git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)
-   [helm](https://helm.sh/)

Configure `gcloud` as a Docker credential helper:

```shell
gcloud auth configure-docker
```

#### Create a Google Kubernetes Engine cluster

Create a new cluster from the command line:

```shell
export CLUSTER=konnect-cluster
export ZONE=us-west1-a

gcloud container clusters create "$CLUSTER" --zone "$ZONE"
```

Configure `kubectl` to connect to the new cluster.

```shell
gcloud container clusters get-credentials "$CLUSTER" --zone "$ZONE"
```

#### Clone this repo

Clone this repo and the associated tools repo.

```shell
git clone --recursive https://github.com/Kong/konnect-gcp-marketplace.git
```

#### Install the Application resource definition

An Application resource is a collection of individual Kubernetes components,
such as Services, Deployments, and so on, that you can manage as a group.

To set up your cluster to understand Application resources, run the following
command:

```shell
kubectl apply -f "https://raw.githubusercontent.com/GoogleCloudPlatform/marketplace-k8s-app-tools/master/crd/app-crd.yaml"
```

You need to run this command once.

The Application resource is defined by the
[Kubernetes SIG-apps](https://github.com/kubernetes/community/tree/master/sig-apps)
community. The source code can be found on
[github.com/kubernetes-sigs/application](https://github.com/kubernetes-sigs/application).

### Install the Application
#### Configure the app with environment variables

Choose the instance name and namespace for the app:

```shell
export APP_INSTANCE_NAME=konnect-gcp
export NAMESPACE=default
```

On the Konnect UI, Create New Runtime Instance, you need to pull these variables.

```shell
export CLUSTER_CP=""
export SERVER_NAME=""
export TELEMETRY_ENDPOINT=""
export TELEMETRY_SERVER_NAME=""
```

#### Encode TLS certificate for Konnect

On the Konnect UI, Create New Runtime Instance you will be able to generate the certificates.

1.  Set `TLS_CERTIFICATE_KEY` and `TLS_CERTIFICATE_CRT` variables:

    ```shell
    export TLS_CERTIFICATE_KEY="$(cat /tmp/tls.key | base64)"
    export TLS_CERTIFICATE_CRT="$(cat /tmp/tls.crt | base64)"
    ```

Set up the image tag:

It is advised to use stable image reference which you can find on
[Marketplace Container Registry](https://marketplace.gcr.io/konghq-public/konnect).
Example:

```shell
export TAG="<BUILD_ID>"
```

Configure the container images:

```shell
export IMAGE_KONG_REPO="marketplace.gcr.io/konghq-public/konnect"
export IMAGE_POSTGRES="marketplace.gcr.io/konghq-public/konnect/postgresql:${TAG}"
```

#### Create a namespace in your Kubernetes cluster

If you use a different namespace than `default`, run the command below to create
a new namespace:

```shell
kubectl create namespace "$NAMESPACE"
```

#### Expand the manifest template

Use `helm template` to expand the template. We recommend that you save the
expanded manifest file for future updates to the application.

```shell
helm template "$APP_INSTANCE_NAME" chart/konnect-dp \
  --namespace "$NAMESPACE" \
  --set kong.image.repository="$IMAGE_KONG_REPO" \
  --set kong.image.tag="$TAG" \
  --set kong.postgresql.image="$IMAGE_POSTGRES" \
  --set kong.env.cluster_control_plane="${CLUSTER_CP}" \
  --set kong.env.cluster_server_name="${SERVER_NAME}" \
  --set kong.env.cluster_telemetry_endpoint="$TELEMETRY_ENDPOINT" \
  --set kong.env.cluster_telemetry_server_name="$TELEMETRY_SERVER_NAME" \
  --set tls.base64EncodedPrivateKey="$TLS_CERTIFICATE_KEY" \
  --set tls.base64EncodedCertificate="$TLS_CERTIFICATE_CRT" \
  > "${APP_INSTANCE_NAME}_manifest.yaml"
```

#### Apply the manifest to your Kubernetes cluster

Use `kubectl` to apply the manifest to your Kubernetes cluster:

```shell
kubectl apply -f "${APP_INSTANCE_NAME}_manifest.yaml" --namespace "${NAMESPACE}"
```

#### View the app in the Google Cloud Console

To get the GCP Console URL for your app, run the following command:

```shell
echo "https://console.cloud.google.com/kubernetes/application/${ZONE}/${CLUSTER}/${NAMESPACE}/${APP_INSTANCE_NAME}"
```

To view your app, open the URL in your browser.

# Using the app

You can get the IP addresses for your Konnect solution either from the command
line, or from the Google Cloud Platform Console.

In the GCP Console, do the following:

1.  Open the
    [Kubernetes Engine Services](https://console.cloud.google.com/kubernetes/discovery)
    page.
1.  Identify the Konnect solution using its name (typically `konnect-*-svc`)
1.  From the Endpoints column, note the IP addresses for ports 80 and 443.

If you are using the command line, run the following command:

```shell
kubectl get svc -l app.kubernetes.io/name=$APP_INSTANCE_NAME --namespace "$NAMESPACE"
```

This command shows the internal and external IP address of your Konnect service.
# Network Resiliency and Availability
Konnect Cloud deployments run in hybrid mode, which means that there is a separate control plane attached to one or more data plane nodes. These planes must communicate with each other to receive and send configuration. If communication is interrupted and either side can’t send or receive config, data plane nodes still continue proxying traffic to clients.

## How do the control plane and data planes communicate?
Data travelling between control planes and data planes is secured through a mutual TLS handshake.

Normally, the data plane maintains a persistent connection with the control plane. The data plane sends a heartbeat to the control plane every 30 seconds to keep the connection alive. If it receives no answer, it tries to reconnect to the control plane node after a 5-10 second random delay.

# Updating the application

These steps assume that you have a new image for the Konnect container available
to your Kubernetes cluster. The new image is used in the following commands as
`[NEW_IMAGE_REFERENCE]`.

```shell
kubectl set image deployment "$APP_INSTANCE_NAME-kong" \
  --namespace "$NAMESPACE" proxy=[NEW_IMAGE_REFERENCE]
```

where `[NEW_IMAGE_REFERENCE]` is the new image.

# Uninstalling the app

You can delete the Konnect application using the Google Cloud Platform Console, or
using the command line.

## Using the Google Cloud Platform Console

1.  In the GCP Console, open
    [Kubernetes Applications](https://console.cloud.google.com/kubernetes/application).

1.  From the list of applications, click **Konnect**.

1.  On the Application Details page, click **Delete**.

## Using the command line

1.  Run the `kubectl delete` command:

    ```shell
    kubectl delete -f ${APP_INSTANCE_NAME}_manifest.yaml --namespace $NAMESPACE
    ```

Optionally, if you don't need the deployed application or the Kubernetes Engine
cluster, delete the cluster using this command:

```shell
gcloud container clusters delete "$CLUSTER" --zone "$ZONE"
```

## To publish a new release of Kong
1. Upload the Kong Docker image to GCP
  - docker pull kong/kong-gateway:3.2.1.0-alpine
  - docker tag kong/kong-gateway:3.2.1.0-alpine gcr.io/konghq-public/konnect:3.2.1
  - docker push gcr.io/konghq-public/konnect:3.2.1

2. Update the Kong version in the [deplment.yaml](gcp-konnect-quickstart/chart/konnect-dp/templates/deployment.yaml)

3. Update the new Deployer image version in the following files
  - [schema.yaml](work/gcp-konnect-quickstart/schema.yaml)
  - [chart.yaml](gcp-konnect-quickstart/chart/konnect-dp/Chart.yaml)
  - [applicaton.yaml](gcp-konnect-quickstart/chart/konnect-dp/templates/application.yaml)

4. Build the Deployer docker image
```export REGISTRY=gcr.io/$(gcloud config get-value project | tr ':' '/')
export APP_NAME=konnect
docker buildx build --tag $REGISTRY/$APP_NAME/deployer:3.1.4 . --platform linux/amd64
docker push $REGISTRY/$APP_NAME/deployer:3.1.4
```   
## Opening a GCP support case for Partner Portal
https://partnersupport.cloud.google.com/partner-contact

## GCP Markeplace K8's App Tools
https://github.com/GoogleCloudPlatform/marketplace-k8s-app-tools/tree/master

## Test using mpdev

## generate the CRD
sh mpdev doctor

## Apply the generated CRD
kubectl apply -f "https://raw.githubusercontent.com/GoogleCloudPlatform/marketplace-k8s-app-tools/master/crd/app-crd.yaml"

## Deploy the deployer images on k8s cluster
sh mpdev install --deployer=gcr.io/konghq-public/konnect/deployer:5.0.1 --parameters='{"name": "test-deployment", "namespace": "test-ns", "cluster_server_name": "default", "cluster_telemetry_server_name": "default", "tls_crt": "LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tDQpNSUlEbkRDQ0FvYWdBd0lCQWdJQkFUQUxCZ2txaGtpRzl3MEJBUTB3T0RFMk1Ba0dBMVVFQmhNQ1ZWTXdLUVlEDQpWUVFESGlJQWF3QnZBRzRBYmdCbEFHTUFkQUF0QUhJQWJ3Qm9BR2tBZEFBdEFHY0FZd0J3TUI0WERUSXpNVEV4DQpOakl5TURBME9Wb1hEVE16TVRFeE5qSXlNREEwT1Zvd09ERTJNQWtHQTFVRUJoTUNWVk13S1FZRFZRUURIaUlBDQphd0J2QUc0QWJnQmxBR01BZEFBdEFISUFid0JvQUdrQWRBQXRBR2NBWXdCd01JSUJJakFOQmdrcWhraUc5dzBCDQpBUUVGQUFPQ0FROEFNSUlCQ2dLQ0FRRUEwZXdjM1NhOE5wZ1BmR1M5UGQwTkNsQy82MFJISXhkdjhtQlhjUHdVDQo3R21FV2t5aC9OVW1MVjlGQWpXNktRWnpCSWM4UytxbFZUZ0o2S0hoWjBFQW8xaUNEVDg5Tk5yM0hhOTA2NDdNDQpyNGFkZWdjTmU3OWtBaTRNL1c0NXp6dzlzeWpXcHJkWStWUFFiR0VZYU55VUlYeWM1TE1XcUd2bDlzMzlXM01aDQpHaGdISW9UUjlDVXVwSGsrZjRFSzJOZ1hKZjNBTkh1dENzeHFUYjRYN3NGNFN6RktzZjVKbVo4bW9Hd3FTYzU3DQpyRlY3emNLNVhHb29tZlhqSWRDdjZWb0NmbHg1Z05JSVExZDdDYXFPTUcxWEZoZUs0OFY2Rzl5bTNYM0tIc1IxDQo0STJRZU92ajBOcVBpd3JlbUVZaGovc1VTcmhORHEwQkpQR09rZktXeFZjUWZRSURBUUFCbzRHME1JR3hNQklHDQpBMVVkRXdFQi93UUlNQVlCQWY4Q0FRTXdDd1lEVlIwUEJBUURBZ0FHTUIwR0ExVWRKUVFXTUJRR0NDc0dBUVVGDQpCd01CQmdnckJnRUZCUWNEQWpBWEJna3JCZ0VFQVlJM0ZBSUVDZ3dJWTJWeWRGUjVjR1V3SXdZSkt3WUJCQUdDDQpOeFVDQkJZRUZBRUJBUUVCQVFFQkFRRUJBUUVCQVFFQkFRRUJNQndHQ1NzR0FRUUJnamNWQndRUE1BMEdCU2tCDQpBUUVCQWdFS0FnRVVNQk1HQ1NzR0FRUUJnamNWQVFRR0FnUUFGQUFLTUFzR0NTcUdTSWIzRFFFQkRRT0NBUUVBDQpSdjNwRGZKNGU4UGxSNlZNMGRlUnc0VFJhd0ljMldXbEZQaEJkaUdxTEpJakZRdkNQZ1UybzNjWUpRWVZjTDhNDQpaUkFoa3ZwcTZZUUc4QVlHb0RoR2lKUnB5VHVuUXFVQlJGRWZWTGpFRzVDaFhFVEVSMlFOOVlabzd4TEtneXNiDQpHcUExWm9HY3NNVFNHa2tRSktyYXQyeXBCbk1MV0VjVmV6bDR6ODF1d28zZGRUMUpydDJzSFdRckV6aGNCcmZoDQpVemtNWEpkcjVxZjZrRzEyMzVwVDBMMFZnNUsvVXB0R3AvQ3hNYUM0SzdYOTFqcWhwWnhHNnVuUWdsZm4wa2dxDQpiZ0dmZTMwQnhKem9wNkxHODhqNDYxb1FYVG9IdGxIakx2b3RZUENLNkpnSElpWFVPV0o4bDlXVGh5a08wQTRyDQpoaC9BYi85RmowTnhCQllCaGROY2VBPT0NCi0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0NCg==", "tls_key": "LS0tLS1CRUdJTiBQUklWQVRFIEtFWS0tLS0tDQpNSUlFdkFJQkFEQU5CZ2txaGtpRzl3MEJBUUVGQUFTQ0JLWXdnZ1NpQWdFQUFvSUJBUURSN0J6ZEpydzJtQTk4DQpaTDA5M1EwS1VML3JSRWNqRjIveVlGZHcvQlRzYVlSYVRLSDgxU1l0WDBVQ05ib3BCbk1FaHp4TDZxVlZPQW5vDQpvZUZuUVFDaldJSU5QejAwMnZjZHIzVHJqc3l2aHAxNkJ3MTd2MlFDTGd6OWJqblBQRDJ6S05hbXQxajVVOUJzDQpZUmhvM0pRaGZKemtzeGFvYStYMnpmMWJjeGthR0FjaWhOSDBKUzZrZVQ1L2dRclkyQmNsL2NBMGU2MEt6R3BODQp2aGZ1d1hoTE1VcXgva21abnlhZ2JDcEp6bnVzVlh2TndybGNhaWlaOWVNaDBLL3BXZ0orWEhtQTBnaERWM3NKDQpxbzR3YlZjV0Y0cmp4WG9iM0tiZGZjb2V4SFhnalpCNDYrUFEybytMQ3Q2WVJpR1AreFJLdUUwT3JRRWs4WTZSDQo4cGJGVnhCOUFnTUJBQUVDZ2dFQUFXTDllM3pWN3ZLNUNDRytqMGRPYy83b1pFODJraFhhOXpTcGpJcVp2OGhnDQpVbHdNWmpnVDVtWlQ5WFJja2JOT3lnZWVWTU8zZFFxU1R0b0NQQlZLN0xQNURmU0RraE1QNFZTOG9XYTNnRUdwDQpJM3A0U0JUbS9maGNaOFhWTmhnb01kbGNvUGJOUWhPMVBpdGVYQ2o0TUlnYkhZdjNvOFFDQ0V2cTRjZXU4WVFXDQppcGpXMHhvdk5HVGdRUE05ZS9JOHpvRHduWnZ5a2Vua25EVHY0YUlDbGIxU0lmcU1tV3c4a0p2Um8yZ1YzRjBCDQpIR3VqYjFpUHB5S2xZamNKTG4vREZDd2s5WVVaOHZZVVp5LzhUTytmTjNjWkZNK3BUZ3JXRjZVZ3BxcUZ6VVdHDQo4L2hpd1A0MVM3bXBQL1FmWmJIYW5GVTZpdG9td1htaFhDNTY5aTVpc1FLQmdRRCtNZEpweHc3blZUZFRKdklJDQo3RlNhalM0Mm8vdWNoK1VlRFZlWllVQ2ZGOXY4eEtEV3Nkb3N6ZlBqc0hWYXo0RlRKNlpWWFhrYkdyMWZhMGdqDQpOUklGOElxYnNxV1FialNoRUR2VU41bGh3R3FDYjdaTkFidUlzLzBFQmFuOVVNYUtKUkR1VDdUUXk4VXMzZkt1DQowZHlYWk55cE9ZRkJYTExJZklHQlZ4TkRMUUtCZ1FEVGFjdHY4b29NYk5OeTZoU2pzVCt0b2tZdGlIclZhUTNxDQo5MzEwZ2dsUmxsYm9WZXlhNHZCTWRGNDFuTkFza21aaDJ5SndVeHhyRHZ4SkRqOHZzaGpVb003SVh1ck1kSlZPDQphSkpEVzBoRFZ2ZmI2UXpLbjFmaWYxQm5teVR2MW42QWxwTWdjcnM3YVRWeWZFTDZuS2lVN3dVb0dNTzFXK2FRDQpsSk0vVWZpVWtRS0JnRlFRMGcrZGYzWk9IbS9uajJBWUdKck1XaDVEK1RCNVdQS3BZdkVjMHF4S3pidzRveUNkDQp6UlBJUVFKcUYwV2pIcGdMb3R6VWZ2clJ5eE5GZmFQM0p6RERybk56ajRIR2tLMDdteTNCL1gzd2pzajRmUWZXDQpyTmkyL2RSWXN6Rk5oM3VrYW9jRjRUeTBSMDloVDZNMVVJalpHSWoydGFLU0w2WlNWdG9abkFzNUFvR0FDc2kzDQp2dU1oVlpicmhrNFlkVzBpTVdvNHFEUHhDQmZPeFBDUTdyTi9aREVHQjkzeUxzaHF0NHVzRHBJTU1HbmJYUngvDQplamxURnNieDZZd1hmd2hYcWVqMkExU01KNWUrMGZ3VmtlZ0RIS1JBQ25DdDNWd1pjSTFML2F6MVNtS25tMG1UDQpBYkc0aVVSSm5LaG9CajZkZnROZWNQZ3FhNExmbFBwdk5HaXJCSEVDZ1lBc0hwcUhTK1U3R3pQL25PaWRZWk5TDQo4TTVlNW5iZ3RPVWxUNG9FTDBoV0YrSk9qN3BMZmFzK2g2dVhGakRHZzZ3aVNrUlcwVUliUGRnenNPcGRnd01EDQpzN1ZNSGxDUTNqME40ei9meE9tSTFZVjNNL3RtVWViQ0d3d2pnVStnVTA3TGFTc0JjcXBnNWhUWWdpWXpnQ3I4DQprMFg3UEZ0bzYyMDBUa1RXNHNxSW13PT0NCi0tLS0tRU5EIFBSSVZBVEUgS0VZLS0tLS0NCg=="}'

## Verify the deployer image
sh mpdev verify --deployer=gcr.io/konghq-public/konnect/deployer:4.0.8 --wait_timeout=500