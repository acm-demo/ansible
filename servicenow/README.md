# ServiceNow Playbook

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

* __extra variabe needed for the create incident playbook.__
    ```
        tower_user_name: from Ansible Automation Plateform."
        policy_name:
        target_clusters:
        hub_cluster
    ```

* __extra variabe needed for the close incident playbook.__
    ```
        incident_number
    ```
