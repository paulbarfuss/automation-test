# Identity Management

This playbook generates an inventory based off of a list of `engagement_users` found in the `engagement.json` file. It parses out the users into the expected structure found [here](https://github.com/redhat-cop/infra-ansible).

*Note:* This currently does not handle multiple targets

# Variables

| name      | description                                           | required | default |
|-----------|-------------------------------------------------------|----------|---------|
| directory | The directory path to create the appropriate files in | y        | NA      |
|           |                                                       |          |         |

# Example File

---
customer_project_pretty: "{{ customer_name }} {{ project_name }}"

ansible_tower:
  url: TODO
  admin_password: TODO
  projects:
    - name: "infra-ansible"
      description: "Infra Ansible Test"
      scm_type: "git"
      scm_url: "https://github.com/paulbarfuss/infra-ansible.git"
      scm_branch: "add-tower-schedule-module"
      organization: na
    - name: "{{ customer_name }} {{ project_name } Project"
      description: "{{ description }}"
      organization: na
      scm_type: "git"
      scm_url: "ssh://git@gitlab.consulting.redhat.com:2222/rht-labs/residencies/sandbox/hooli/nucleus/iac" #TODO
      scm_branch: "master"
      scm_credential_name: "gitlab-scm-dev"
      scm_update_on_launch: true
  job_templates:
    - name: "Email and notify list of users - {{ customer_name }} {{ project_name }"
      description: "Job Template to email a list of users"
      inventory: "{{ customer_name }} {{ project_name } Inventory"
      project: "infra-ansible"
      playbook: "playbooks/notifications/email-notify-list-of-users.yml"
      credential: "{{ smtp_credential }}"
      ask_variables_on_launch: true
  inventories:
    - name: "{{ customer_name }} {{ project_name } Inventory"
      variables: ""
      organization: na
      sources:
        - name: "{{ customer_name }} {{ project_name } Inventory Source"
          description: "GitLab inventory source for the {{ customer_name }} {{ project_name }} engagement"
          credential: "gitlab-scm-dev"
          source: "scm"
          source_project: "{{ customer_name }} {{ project_name } Project"
          source_path: "iac/inventories/mail-host/inventory/hosts"
          update_on_launch: true
          source_vars: |-
            ---
  schedules:
    - name: "{{ customer_name }} {{ project_name } - Welcome Internal"
      description: "Welcome Internal for internal Red Hat engagement members"
      rrule: "{{ start_date | parse_datetime | subtract_time(days=1) | to_rrule(freq=3, interval=1, count=1)}}"
      unified_job_template: "Email and notify list of users - {{ customer_name }} {{ project_name }"
    - name: "{{ customer_name }} {{ project_name } Shutoff"
      description: "Shutoff e-mail for the {{ customer_name }} {{ project_name }} engagement"
      rrule: "{{ archive_date | parse_datetime | to_rrule(freq=3, interval=1, count=1)}}"
      unified_job_template: "Email and notify list of users - {{ customer_name }} {{ project_name }"
    - name: "{{ customer_name }} {{ project_name } Welcome All"
      description: "Welcome notification for Red Hat and customer"
      rrule: "{{ start_date | parse_datetime | to_rrule(freq=3, interval=1, count=1) }}"
      unified_job_template: "Email and notify list of users - {{ customer_name }} {{ project_name }"