# Ansible Playbook to create or delete a GKE Cluster


## Create an Execution Environement for Ansible Automation Plateform.

## Prerequisites

* Podman
* Login into registry.redhat.io registry `podman login registry.redhat.io`
* A registry to push your execution environment image like for example quay.io.  `podman login quay.io`
* Create an ansible.cfg file in `/execution-env` folder to give you Automation Hub Token (see next step)
---

## Build the execution environement 

1. Create the ansible.cfg file

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

2. Build the execution image

    1. Navigate to the execution-environment folder
    _~/github/org/acm-demo/ansible/gke-cluster-creation/execution-env_
    1. Define the build version we want to start with in a variable.
        ```
        buildVersion=0.1
        ```

    1. Now lets build the image
        ```
        ansible-builder build -v 3 -t ansible-ee-gcp:${buildVersion}
        ```
    1. Let's tag the images, one tag with the version one with latest. ( in this command I use my quay registry, change for your location.)

        ```
        podman tag ansible-ee-gcp:${buildVersion} quay.io/froberge/ansible-ee-gcp:${buildVersion}
        ```
        ```
        podman tag ansible-ee-gcp:${buildVersion} quay.io/froberge/ansible-ee-gcp:latest
        ```

    1. Now push the 2 images to the registry
        ```
        podman push quay.io/froberge/ansible-ee-gcp:latest
        ```

        ```
        podman push quay.io/froberge/ansible-ee-gcp:${buildVersion}
        ```
---

### Deploy in Ansible Automation Platform aka AAP

1. Create a new credential for the Google Cloud Plateform
1. Follow the [instruction](../servicenow/README.md) to create the credential for serviceNow
1. Add the new Execution Environment in AAP
1. Create the job project
    * create a survey to fill in the information that will come from ServiceNow.
1. Add the right access permission.
1. Create a project to point to the the right git repository
1. Create the job template
    * Use the newly added credential
    * Use the newly added execution environment
    * Use the newly created project
