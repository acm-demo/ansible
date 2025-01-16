# Service Now, Connection with Ansible.

## Important link on setting up Services Now to AAP.
  * [ServiceNow to Ansible RestMethod](https://cloudautomation.pharriso.co.uk/post/snow-call-tower/)
  * [Ansible, ServiceNow Outbound API](https://www.redhat.com/en/blog/ansible-and-servicenow-part-3-making-outbound-restful-api-calls-red-hat-ansible-tower?intcmp=7015Y000003t7aWQAQ)
  * [Automate ServiceNow with AAP](https://www.redhat.com/en/blog/servicenow-ansible-automation)



## Documentation for the ServiceNow Playbook

The playbook are use to create or close and Non Compliant Incident on ServiceNow.



### How to use:

The playbook have been develop to use with the Ansible Applicaiton Platform. They can also be use locally.

* Use in Ansible Automation PlateForm
    1. Define a new credential type for servicenow.
![credential type](/documentation/img/credentialtype.png)
    >>>
        Name: servicenow.itsm

    >>>
        Input configuration YAML
        fields:
        - id: SN_USERNAME
            type: string
            label: Username
        - id: SN_PASSWORD
            type: string
            label: Password
            secret: true
        - id: SN_HOST
            type: string
            label: Snow Host
        required:
        - SN_USERNAME
        - SN_PASSWORD
        - SN_HOST


    >>>
        Injector configuration YAML

        env:
        SN_HOST: '{{ SN_HOST }}'
        SN_PASSWORD: '{{ SN_PASSWORD }}'
        SN_USERNAME: '{{ SN_USERNAME }}'


    2. Create a new credential of type servicenow to connect to the servicenow instance.
    ![credentials](documentation/img/credential.png)

    3. Link a project to the source control containing the playbook.

    4. Create a template for that source control using the Credential define at steps 2.


* Run the playbook locally
    If running locally the instance must be define for service now. Some variables also needs to pass as extra variable.


    #### Config Service Now Instance
    ```

    vars:
        # Replace by your instance of service-now
        servicenow_instance: "https://devXXXXX.service-now.com"
        servicenow_username: ""
        servicenow_password: ""
        incident_number: "" 

    tasks:
        - name: ...
        servicenow.itsm.incident:
        instance:
            host: "{{ servicenow_instance }}"
            username: "{{ servicenow_username }}"
            password: "{{ servicenow_password }}"
    ```

    #### Extra variable

    * extra variabe needed for the create incident playbook.
        >>>
            tower_user_name: 
            policy_name:
            target_clusters:
            hub_cluster:

    * extra variabe needed for the close incident playbook.
        >>>
            incident_number

