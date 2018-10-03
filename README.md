# genie-surveys
## Description
An Ansible Role to deploy and ensure job template surveys are in a desired state in Ansible Tower.
## Variables
|Variable Name|Default Value|Required|Description|Type|
|---|:---:|:---:|---|:---:|
|`tower_url`|""|yes|URL to the Ansible Tower Server.|string|
|`tower_verify_ssl`|False|no|Whether or not to validate the Ansible Tower Server's SSL certificate.|boolean|
|`tower_secrets`|False|yes|Whether or not to include variables stored in vars/tower-secrets.yml.  Set this value to `False` if you will be providing your sensitive values from elsewhere.|boolean|
|`tower_user`|""|yes|Admin User on the Ansible Tower Server.|string|
|`tower_pass`|""|yes|Tower Admin User's password on the Ansible Tower Server.  This should be stored in an Ansible Vault at vars/tower-secrets.yml or elsewhere and called from a parent playbook.|string|
|`tower_surveys`|[]|yes|Ansible Tower survey definitions|List of dictionaries|

### Variable `tower_surveys` Dictionary Specificaiton
The `tower_surveys` skeleton structure is as follows:  
*Collapsed YAML:*
```yaml
tower_surveys: [{job_template_name: "", job_template_survey_enabled: True, survey: [{}]}]
```
*Expanded YAML:*
```yaml
tower_surveys:
  - job_template_name: ""
    job_template_survey_enabled: True
    survey: [{}]
```

|Key Name|Required|Description|Type|
|---|:---:|---|:---:|
|`job_template_name`|yes|Name of the Ansible Tower job template to operate on.|string|
|`job_template_survey_enabled`|yes|Whether or not to enable the survey on the job template.|boolean|
|`survey`|yes|The survey definition for the job template. Each dictionary in the list defines a field in the survey.|List of dictionaries.|

#### `survey` Dictionary
Based on the type of survey field, different dictionary keys are required, which are described below.

##### Common Keys for `survey` Dictionaries
|Key Name|Required|Description|Type|
|---|:---:|---|
|`variable`|yes|The variable to map the end-user's survey response to when executing the job template.|string|
|`prompt`|yes|The survey field label that the end-user will see when executing the job template.|string|
|`type`|yes|The type of survey field to generate in the job template's survey.  The value can be one of the following: float, integer, multiselect, multiplechoice, password, textarea, text|string|
|`description`|no|The survey field's description the end-user will see when executing the job template.|string|
|`default`|no|The survey field's default/pre-filled value that the end-user will see when executing the job template.|string (on type multiselect this is a list, and on type password this is not available because Ansible Tower stores the value encrypted and it becomes "$encrypted$" in the stored specification and unable to ensure the values match the desired state otherwise)|
|`required`|Whether or not the end-user is required to enter data into the survey field when executing the job template|boolean|

###### Type float
A numeric value with a floating decimal point.

|Key Name|Required|Default Value|Description|Type|
|---|:---:|:---:|---|---|
|min|no|0|Minimum value accepted|float|
|max|no|100|Maximum value accepted|float|

###### Type integer
An integer value.

|Key Name|Required|Default Value|Description|Type|
|---|:---:|:---:|---|---|
|min|no|0|Minimum value accepted.|integer|
|max|no|100|Maximum value accepted.|integer|

###### Type multiselect
A multiple-choice multiple-select (Ansible Tower converts this to a list).

|Key Name|Required|Default Value|Description|Type|
|---|:---:|:---:|---|---|
|choices|yes|n/a|List of choices to present the end-user with when executing the job template|list|

###### Type multiplechoice
A multiple-choice single-select (Ansible Tower converts this to a string).

|Key Name|Required|Default Value|Description|Type|
|---|:---:|:---:|---|---|
|choices|yes|n/a|List of choices to present the end-user with when executing the job template.|list|

###### Type password
A password that Ansible Tower encrypts.  It is handled differently than a text type.

|Key Name|Required|Default Value|Description|Type|
|---|:---:|:---:|---|---|
|min|no|0|Minimum number of characters accepted in the survey field.|integer|
|max|no|32|Maximum number of characters accepted in the survey field.|integer|

###### Type textarea
A field to enter multi-lined text.

|Key Name|Required|Default Value|Description|Type|
|---|:---:|:---:|---|---|
|min|no|0|Minimum number of characters accepted in the survey field.|integer|
|max|no|4096|Maximum number of characters accepted in the survey field.|integer|

###### Type text
A field to enter text (string).

|Key Name|Required|Default Value|Description|Type|
|---|:---:|:---:|---|---|
|min|no|0|Minimum number of characters accepted in the survey field.|integer|
|max|no|1024|Maximum number of characters accepted in the survey field.|integer|

## Playbook Examples
### Normal Role Definition in Play
```yaml
---
- name: "Ensure my surveys are created"
  hosts: "localhost"
  connection: "local"
  vars_files:
    - "vars/myvault.yml"
  roles:
    - role: "genie-survey"
      tower_url: "https://mytower.mydomain.com"
      tower_verify_ssl: False
      tower_secrets: False
      tower_user: ""
      tower_pass: "{{ vaulted_tower_pass }}"
      tower_surveys:
        - job_template_name: "My Job Template 1"
          job_template_survey_enabled: True
          survey:
            - variable: "myfloatvar"
              type: "float"
              prompt: "Enter a float value between 4 and 10"
              description: "A value to do some math with"
              default: "6.5"
              min: "4"
              max: "10"
              required: True
            - variable: "mypetlist"
              type: "multiselect"
              prompt: "Select the kinds of pets you have"
              choices:
                - "dog"
                - "cat"
                - "fish"
                - "bird"
                - "hamster"
                - "chinchilla"
              default:
                - "dog"
                - "fish"
              required: False
            - variable: "petname"
              type: "multiplechoice"
              prompt: "What's your favorite pet name?"
              description: "A list of pet names to choose from"
              choices:
                - "Fritz"
                - "Ava"
                - "Bob"
              default: "Fritz"
              required: True
        - job_template_name: "My Job Template 2"
          job_template_survey_enabled: False
          survey:
            - variable: "text1"
              type: "text"
              prompt: "Enter some text"
              description: "A string to use in my playbook"
            - variable: "my_pass"
              type: "password"
              prompt: "Enter the password to my application"
              min: "10"
              max: "25"
```

### Import Role in Task
```yaml
---
- name: "Ensure my surveys are created"
  hosts: "localhost"
  connection: "local"
  vars_files:
    - "vars/myvault.yml"
  tasks:
    - name: "Build my job template surveys"
      import_role:
        name: "genie-survey"
      vars:
        tower_url: "https://mytower.mydomain.com"
        tower_verify_ssl: False
        tower_secrets: False
        tower_user: ""
        tower_pass: "{{ vaulted_tower_pass }}"
        tower_surveys:
          - job_template_name: "My Job Template 1"
            job_template_survey_enabled: True
            survey:
              - variable: "myfloatvar"
                type: "float"
                prompt: "Enter a float value between 4 and 10"
                description: "A value to do some math with"
                default: "6.5"
                min: "4"
                max: "10"
                required: True
              - variable: "mypetlist"
                type: "multiselect"
                prompt: "Select the kinds of pets you have"
                choices:
                  - "dog"
                  - "cat"
                  - "fish"
                  - "bird"
                  - "hamster"
                  - "chinchilla"
                default:
                  - "dog"
                  - "fish"
                required: False
              - variable: "petname"
                type: "multiplechoice"
                prompt: "What's your favorite pet name?"
                description: "A list of pet names to choose from"
                choices:
                  - "Fritz"
                  - "Ava"
                  - "Bob"
                default: "Fritz"
                required: True
          - job_template_name: "My Job Template 2"
            job_template_survey_enabled: False
            survey:
              - variable: "text1"
                type: "text"
                prompt: "Enter some text"
                description: "A string to use in my playbook"
              - variable: "my_pass"
                type: "password"
                prompt: "Enter the password to my application"
                min: "10"
                max: "25"
```


## Author
[Andrew J. Huffman](mailto:ahuffman@redhat.com)
