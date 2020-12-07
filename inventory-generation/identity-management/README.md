# Identity Management

This playbook generates an inventory based off of a list of `engagement_users` found in the `engagement.json` file. It parses out the users into the expected structure found [here](https://github.com/redhat-cop/infra-ansible).

*Note:* This currently does not handle multiple targets

# Variables

| name      | description                                           | required | default |
|-----------|-------------------------------------------------------|----------|---------|
| directory | The directory path to create the appropriate files in | y        | NA      |
|           |                                                       |          |         |
