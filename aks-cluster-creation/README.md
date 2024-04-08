# Execution Environement

## Prerequisites

* Podman
* Login into registry.redhat.io registry `podman login registry.redhat.io`
* Login into your registry, where you will push your image `podman login quay.io` for instance
* Create an ansible.cfg file in `/execution-env` folder to give you Automation Hub Token (see next step)

## Build the execution environement 
I am using a Red Hat Enterprise Linux instance as my developer environment. Here is the step taken to make it viable to build the execution environment needed to execute the different playbooks to create AKS clusters and register them into Red Hat Advanced Cluster Management.

### 1. Create the ansible.cfg file

For ansible-build to be able to fetch collections from Red Hat Automation Hub, you will need to provide your Automation Hub token. This is done by editing the ansible.cfg file. Make sure this file won't be push into a public git repo since it will contain your personal AH token.

``` ini
[galaxy]
server_list = automation_hub

[galaxy_server.automation_hub]
url=https://console.redhat.com/api/automation-hub/content/published/
auth_url=https://sso.redhat.com/auth/realms/redhat-external/protocol/openid-connect/token

token=<my_ah_token>
```

Save this file in /exectution-env/ansible.cfg

### 2. Build the image

``` bash
BV=0.0 # <-- Set the version you want to tag
cd execution-env && \
ansible-builder build -v 3 -t ansible-ee-azure:${BV} && \
podman tag ansible-ee-azure:${BV} ansible-ee-azure:latest && \
podman tag ansible-ee-azure:${BV} quay.io/dovid/ansible-ee-azure:${BV} && \
podman tag ansible-ee-azure:${BV} quay.io/dovid/ansible-ee-azure:latest && \
podman push quay.io/dovid/ansible-ee-azure:${BV} && \
podman push quay.io/dovid/ansible-ee-azure:latest
```

### 3. Test the image

Since our test is not done in Ansible Automation Platform, we will need to manually provide the Azure credentials.

``` bash
cd ansible && \
ansible-navigator run create-aks.yaml \
--eei ansible-ee-azure:latest \
--mode stdout \
--pull-policy missing \
--enable-prompts \
--senv AZURE_SUBSCRIPTION_ID=*********************** \
--senv AZURE_TENANT==*********************** \
--senv AZURE_CLIENT_ID==*********************** \
--senv AZURE_SECRET==***********************
```

# Ansible Playbooks

## Create-AKS-Cluster

## Delete-AKS-Cluster

# Ansible Automation Platform

## Execution Environement
We will need to add our newly created Execution Environement to AAP by going in: Administration->ExecutionEnvironements
Add the new EE by filling out the form with your registry URI and credential.

## Create Credentials

### Microsoft Azure Resource Manager

### OpenShift or Kubernetes API Bearer Token

## Create the project

## Create the job template

### Activate the survey

# OpenShift Advanced Cluster Management

## Create the shared image pull secret if not already done in ACM
When creating the multiclusterhub, you can provide an imagepullsecret which will be used when importing a non-openshift cluster. If you didn't add an imagepullsecret at that time, proceed with the following to copy the clusterhub internal imagepullsecret to the multiclusterengin.

1. Copy the imagepullsecret from your clusterhub cluster:

``` bash
DOCKER_CONFIG_JSON=`oc extract secret/pull-secret -n openshift-config --to=-`
```

2. Then create a imagepullsecret secret into the open-cluster-management namespace 
``` yaml
oc create secret generic multiclusterhub-operator-pull-secret \
    -n open-cluster-management \
    --from-literal=.dockerconfigjson="$DOCKER_CONFIG_JSON" \
    --type=kubernetes.io/dockerconfigjson
```

3. Patch the MultiClusterHub to use the secret
``` bash
oc patch multiclusterhubs.operator.open-cluster-management.io/multiclusterhub -n open-cluster-management --type='merge' -p '{"spec":{"imagePulSecret":"multiclusterhub-operator-pull-secret"}}'

```
apiVersion: operator.open-cluster-management.io/v1
kind: MultiClusterHub
metadata:
  name: multiclusterhub
  namespace: <namespace>
spec:
  imagePullSecret: <secret>

4. Once there, you can also create the same imagepullsecret for the multiclusterhub-observability addon

``` yaml
oc create secret generic multiclusterhub-operator-pull-secret \
    -n open-cluster-management-observability \
    --from-literal=.dockerconfigjson="$DOCKER_CONFIG_JSON" \
    --type=kubernetes.io/dockerconfigjson
```

## Observability

### Create you image pull secret https://access.redhat.com/documentation/en-us/red_hat_advanced_cluster_management_for_kubernetes/2.7/html-single/observability/index#:~:text=stable%20object%20stores.-,1.2.2.%20Enabling%20observability,-Enable%20the%20observability

## Links

https://access.redhat.com/documentation/en-us/red_hat_advanced_cluster_management_for_kubernetes/2.9/html/install/installing#advanced-config-hub:~:text=console%20is%20disabled.-,1.4.2.%C2%A0Custom%20Image%20Pull%20Secret,-If%20you%20plan
https://access.redhat.com/documentation/en-us/red_hat_advanced_cluster_management_for_kubernetes/2.7/html-single/observability/index