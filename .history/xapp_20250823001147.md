# Near-RT RIC Deployment & xApp Installation Documentation

## 1. Platform Setup and Environment Preparation

### 1.1 Clone the Repository

First, the repository for the Near-RT RIC platform is cloned using the following command:

```bash
git clone "https://gerrit.o-ran-sc.org/r/ric-plt/ric-dep"
```

This fetches the code for deployment under ~/Music/ric-dep.

### 1.2 Switch to the Appropriate Branch

Switch to the j-release branch:

```bash
cd ric-dep/bin
git checkout j-release
```

### 1.3 Installing Dependencies

Before setting up the platform, we need to install Python 3.9 and set up a virtual environment.

```bash
# Add Python PPA repository
sudo add-apt-repository ppa:deadsnakes/ppa
sudo apt update

# Install Python 3.9 and venv
sudo apt install python3.9-dev python3.9-venv
```

Create and activate the virtual environment:

```bash
# Create virtual environment
python3.9 -m venv .venv

# Activate the virtual environment
. .venv/bin/activate
```

Install the required dependencies:

```bash
pip install -r requirements.txt
pip install .
```

### 1.4 xApp Onboarding

Navigate to the xApp Onboarding directory:

```bash
cd ric-plt-appmgr/xapp_orchestrater/dev/xapp_onboarder
```

### 1.5 Install Kubernetes, Helm, and Docker

To deploy the RIC platform, Kubernetes and Helm need to be installed. Follow the installation guide from the documentation. Once the installation is complete, run:

```bash
cd ric-dep/bin
./install_k8s_and_helm.sh
```

This script will set up Kubernetes, Helm, and other dependencies required for deployment.

### 1.6 Deploy Chartmuseum

To deploy the Chartmuseum Helm chart repository, run:

```bash
helm repo add chartmuseum https://chartmuseum.github.io/charts
helm repo update
helm install chartmuseum chartmuseum/chartmuseum -n ricplt --create-namespace --set env.open.DISABLE_API=false --set persistence.enabled=false
```

Confirm the installation using kubectl:

```bash
kubectl get svc -n ricplt | grep chartmuseum
```

## 2. xApp Deployment and Troubleshooting

### 2.1 Clone xApp Example

Next, clone the xApp example repository:

```bash
git clone https://github.com/wineslab/xDevSM-xapps-examples.git
```

Change the directory to the xApp folder:

```bash
cd xDevSM-xapps-examples
```

### 2.2 Building Docker Image

**Docker Command to Build and Push Image:**

```bash
docker build --tag kpm-basic-xapp:0.1.0 --file docker/Dockerfile.kpm_basic_xapp .
```

* **Tag the image** with your DockerHub username:

```bash
docker tag kpm-basic-xapp:0.1.0 anirudh819/kpm-basic-xapp:0.1.0
```

* **Push the image** to DockerHub:

```bash
docker push anirudh819/kpm-basic-xapp:0.1.0
```

**Docker Run Command:**

```bash
docker run --rm -u 0 -it -d -p 8090:8080 \
  -e DEBUG=1 -e STORAGE=local -e STORAGE_LOCAL_ROOTDIR=/charts \
  -v $(pwd)/charts:/charts chartmuseum/chartmuseum:latest
```

This command sets up a container for **ChartMuseum** with the following options:
* Runs in debug mode (`DEBUG=1`)
* Uses local storage (`STORAGE=local`)
* Binds the local charts directory to the container's `/charts` directory

### 2.3 Port-Forwarding the Service

Once the chart has been installed and pods are running, port-forward the service to access the xApp locally:

```bash
kubectl port-forward svc/chartmuseum 8080:8080 -n ricplt
```

If the port is already in use, use:

```bash
kubectl port-forward svc/chartmuseum 8081:8080 -n ricplt
```

### 2.4 Verify xApp Installation

List the installed Helm charts:

```bash
helm list -n ricxapp
```

Confirm the chart is deployed:

```bash
kubectl get pods -n ricxapp
```

Check the status:

```bash
kubectl get svc -n ricxapp
```

## 3. API Calls and Troubleshooting

### 3.1 Issue with API Access

Attempting to query the API endpoints results in a 404 error due to incorrect path or chart naming. Here's the output from the curl command:

```bash
curl -X GET http://localhost:8080/api/charts | jq .
{
  "code": 404,
  "message": "path /api/charts was not found"
}
```

After troubleshooting and ensuring ChartMuseum is up and running, the error persists due to missing configuration.

### 3.2 Solution to API Query Issues

The issue was resolved by checking the repository configuration and ensuring the correct paths were specified for both the chart and schema files.

```bash
dms_cli onboard --config-file-path ~/Music/ric-dep/bin/ric-plt-appmgr/xapp_orchestrater/dev/xapp_onboarder/xDevSM-xapps-examples/kpm_basic_xapp/config/config-file.json --schema-file-path ~/Music/ric-dep/bin/ric-plt-appmgr/xapp_orchestrater/dev/xapp_onboarder/xDevSM-xapps-examples/kmp_basic_xapp/config/schema.json
```

## 4. Final Notes

### 4.1 xApp Deployment Status

The deployment process for kpm-basic-xapp was completed successfully:

```bash
helm install kpm-basic-xapp /tmp/kpm_basic_chart -n ricxapp
```

### 4.2 Known Issues

Port forwarding fails when ports are already in use. Ensure ports are freed before retrying.

API query fails if the Helm charts are not properly installed. Recheck Helm repository and chart paths.

## 5. Conclusion

The Near-RT RIC platform and xApp deployment were successfully executed, though troubleshooting issues related to Helm chart installation, API access, and port conflicts were encountered. These issues were resolved by confirming chart installation, verifying API paths, and resolving port conflicts.

---

## Deployment Status Summary

Here is the detailed summary of the Kubernetes pods, services, and helm chart data you requested, formatted for clarity:

**Pods (in namespace `ricplt`):**
* **chartmuseum-6c5bcf5cb5-mn22l**: 1/1, Running, 0 restarts, 22m
* **deployment-ricplt-a1mediator-78f79cbb6b-6gxbz**: 1/1, Running, 0 restarts, 23h
* **deployment-ricplt-alarmmanager-5b49476676-xw2lh**: 1/1, Running, 0 restarts, 23h
* **deployment-ricplt-appmgr-5564b65869-4vxxt**: 1/1, Running, 0 restarts, 23h
* **deployment-ricplt-e2mgr-9b6b8f99f-njh62**: 1/1, Running, 0 restarts, 23h
* **deployment-ricplt-e2term-alpha-f8f7d7855-w4xgz**: 1/1, Running, 0 restarts, 23h
* **deployment-ricplt-o1mediator-bf4fb5758-z9kkk**: 1/1, Running, 0 restarts, 23h
* **deployment-ricplt-rtmgr-657457c4bb-kl6vx**: 1/1, Running, 5 restarts, 23h
* **deployment-ricplt-submgr-858956fbdc-fhdcr**: 1/1, Running, 0 restarts, 23h
* **deployment-ricplt-vespamgr-848f7bb874-c5k68**: 1/1, Running, 0 restarts, 23h
* **r4-infrastructure-kong-78657d8f48-nwqc6**: 2/2, Running, 0 restarts, 23h
* **r4-infrastructure-prometheus-alertmanager-b9cc56766-9qqbm**: 2/2, Running, 0 restarts, 23h
* **r4-infrastructure-prometheus-server-6476958975-pm9bf**: 1/1, Running, 0 restarts, 23h
* **statefulset-ricplt-dbaas-server-0**: 1/1, Running, 0 restarts, 23h

**Services (in namespace `ricplt`):**
* **aux-entry**: ClusterIP, 10.96.94.80, Ports: 80/TCP, 443/TCP, 23h
* **chartmuseum**: ClusterIP, 10.96.83.168, Port: 8080/TCP, 22m
* **r4-infrastructure-kong-manager**: NodePort, 10.96.247.166, Ports: 8002:30186/TCP, 8445:30294/TCP, 23h
* **r4-infrastructure-kong-proxy**: LoadBalancer, 10.96.138.166, Ports: 80:32080/TCP, 443:32443/TCP, 23h
* **r4-infrastructure-kong-validation-webhook**: ClusterIP, 10.96.204.208, Port: 443/TCP, 23h
* **r4-infrastructure-prometheus-alertmanager**: ClusterIP, 10.96.64.236, Port: 80/TCP, 23h
* **r4-infrastructure-prometheus-server**: ClusterIP, 10.96.200.63, Port: 80/TCP, 23h
* **service-ricplt-a1mediator-http**: ClusterIP, 10.96.93.152, Port: 10000/TCP, 23h
* **service-ricplt-a1mediator-rmr**: ClusterIP, 10.96.204.118, Ports: 4561/TCP, 4562/TCP, 23h
* **service-ricplt-alarmmanager-http**: ClusterIP, 10.96.45.105, Port: 8080/TCP, 23h
* **service-ricplt-alarmmanager-rmr**: ClusterIP, 10.96.252.41, Ports: 4560/TCP, 4561/TCP, 23h
* **service-ricplt-appmgr-http**: ClusterIP, 10.96.190.196, Port: 8080/TCP, 23h
* **service-ricplt-appmgr-rmr**: ClusterIP, 10.96.113.73, Ports: 4561/TCP, 4560/TCP, 23h
* **service-ricplt-dbaas-tcp**: ClusterIP, None, Port: 6379/TCP, 23h
* **service-ricplt-e2mgr-http**: ClusterIP, 10.96.127.5, Port: 3800/TCP, 23h
* **service-ricplt-e2mgr-rmr**: ClusterIP, 10.96.27.33, Ports: 4561/TCP, 3801/TCP, 23h
* **service-ricplt-e2term-prometheus-alpha**: ClusterIP, 10.96.245.191, Port: 8088/TCP, 23h
* **service-ricplt-e2term-rmr-alpha**: ClusterIP, 10.96.21.183, Ports: 4561/TCP, 38000/TCP, 23h
* **service-ricplt-e2term-sctp-alpha**: NodePort, 10.96.171.43, Port: 36422:32222/SCTP, 23h
* **service-ricplt-o1mediator-http**: ClusterIP, 10.96.207.225, Ports: 9001/TCP, 8080/TCP, 3000/TCP, 23h
* **service-ricplt-o1mediator-tcp-netconf**: NodePort, 10.96.4.144, Port: 830:30830/TCP, 23h
* **service-ricplt-rtmgr-http**: ClusterIP, 10.96.214.158, Port: 3800/TCP, 23h
* **service-ricplt-rtmgr-rmr**: ClusterIP, 10.96.103.110, Ports: 4561/TCP, 4560/TCP, 23h
* **service-ricplt-submgr-http**: ClusterIP, None, Port: 3800/TCP, 23h
* **service-ricplt-submgr-rmr**: ClusterIP, None, Ports: 4560/TCP, 4561/TCP, 23h
* **service-ricplt-vespamgr-http**: ClusterIP, 10.96.118.168, Ports: 8080/TCP, 9095/TCP, 23h

**Xapps (in namespace `ricxapp`):**
* **kpm-basic-xapp-758fdbfc7b-kqlrm**: 1/1, Running, 0 restarts, 11m
* **kmp-basic-xapp**: ClusterIP, 10.96.173.58, Port: 80/TCP, 11m

**Helm Chart:**
* **kpm-basic-xapp**: Deployed, Revision 1, Chart version: 0.1.0, App version: 1.16.0