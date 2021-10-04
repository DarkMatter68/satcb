# satcb:  SATellite server 6 - Content Baseline

This playbook will publish new versions of CV and CCV's. And promote CCV's to designated lifecycle.

 

This playbook can also control the number of versions of CV / CCV to keep after promoting to a lifecycle.

 

The first (alpha) version of this playbook originally used the Ansible Role: satellite6-content_views from Ansible Galaxy:-

 

   https://galaxy.ansible.com/ogerbron/satellite6-content_views

 

However, I have since modified the playbook to incorporate the role as it became clear it did not fullfil all my requirements.

 

 
 

**NOTE**:

> In the future it is my intention to replace this playbook with a Ansible module.

 

 

## Disclaimer

This is my first 'major' attempt at doing something _useful_ in Ansible. If you see something that does not follow good practice or could just be done better then please (politely :-) ) let me know.

I am aware there is some code cleanup is required ...  :-)
 

 

 

## RPM Pre-requisites

+ ansible-engine

+ python-dns (used for performing a pre-check on the satellite server hostname)

 

## Installation

 

< still to be written :-) >

 

```bash

< still to be written :-) >

```

 

 

 

## Define *ANSIBLE_LOG_PATH*

It is recommended to define a value for the **environment** variable: *ANSIBLE_LOG_PATH* if it is not already set. This will make is easier to check the playbook run if there are any issues.

 

```bash

export ANSIBLE_LOG_PATH=~/code/ansible/log/ansible_$(date +%Y%m%d-%H%M%S.%N).log

 

```

Notice the use of "%N" in the date command. This will create a date string using *nano* second precission. This will prevent the same logfile from being used if the playbook is executed at the same time more than once.

 

The log path will be referenced in the Email report summary.

 

 

## Playbook Configuration Parameters

In the *var* directory of the playbook there are various configuration files:-

 

 

### satcb_config-default.yaml

 

 

Here is a list of all the default variables for this playbook.

(If a vaiable in the playbook does not begin with '_satcb_' then it is

a internal variable and should not be touched.)

 

 

---

Here are the main variables needed (excluding API credentials)

| Variable name                 | Required  | Default                                 | Choices       | TYPE                  | Comments                                                                                          |
|-------------------------------|:---------:|:---------------------------------------:|:-------------:|:---------------------:|:-------------------------------------------------------------------------------------------------:|
| `satcb_hostname`              | yes       | <empty string>                          |               | STRING                | FQDN of the Satellite 6 Server to connect to                                                      |
| `satcb_organization`          | yes       | <empty string>                          |               | STRING                | Name of the ORGRANIZATION to use in the Satellite Server.                                         |
| `satcb_promote_ccv_only`      | no        | true                                    | true / false  | BOOLEAN               | Determine if the playbook is to create new versions of CV/CCV's or simply promote to a lifecycle. |
| `satcb_promote_ccv_to_env`    | no        | "Library"                               |               | STRING                | Name of a VALID lifecycle environment to promote CCV's to.                                        |
| `satcb_build_frequency`       | yes       | <empty string>                          | See comment   | STRING               | Used to create the baseline description. Valid choices: daily, weekly, monthly, quarterly, yearly |
| `satcb_email_to`              | yes\*     | <EMPTY LIST>                            |               | LIST                  | Required if *satcb_email_send* is 'true'. One or more Email addresses in a list.                  |
 | `satcb_email_send`            | no\*      | true                                    | true / false  | BOOLEAN               | If set to 'false' no Email report summary will be sent - unless there is an error.                |
| `satcb_email_from`            | yes\*     | <EMPTY STRING>                          |               | STRING                | Required if *satcb_email_send* is 'true'.                                                         |
| `satcb_remove_old_cv_ccv`     | no        | true                                    | true / false  | BOOLEAN               | If set to FALSE, then OLD versions will **NOT** be removed                                        |
| `satcb_email_port`            | yes\*     | 25                                      |               | INTEGER               | Port to use for SMTP traffic                                                                      |
| `satcb_email_host`            | yes\*     | localhost                               |               | STRING                | Name of the Email server (typically 'localhost')                                                  |
| `satcb_email_subject`         | no\*      | <automatically generated>               |               | STRING                | Subject is automatically generated if not specified.                                              |

\* required only if `satcb_email_send` is TRUE.

 

#### Define API related parameters ####

| Variable name                 | Required  | Default                                 | Choices       | TYPE                  | Comments                                                                                          |
|-------------------------------|:---------:|:---------------------------------------:|:-------------:|:---------------------:|:-------------------------------------------------------------------------------------------------:|
| `satcb_http_protocol`         | no        | https                                   | https or http | STRING                | Define the HTTP protocol to use.                                                                  |
| `satcb_url_timeout`           | yes       | 180                                     |               | INTEGER               | Number of seconds before a API call will time-out                                                 |
| `satcb_wait_task_retries`     | yes       | 90                                      |               | INTEGER               | Number of times to check if a API call has finished                                               |
| `satcb_wait_task_delay`       | yes       | 10                                      |               | INTEGER               | Number of SECONDS to wait before checking again if a API call has finished                        |

  

#### Define Satellite API user/password credentials

| Variable name                 | Required  | Default                                 | Choices       | TYPE                  | Comments                                                                                          |
|-------------------------------|:---------:|:---------------------------------------:|:-------------:|:---------------------:|:-------------------------------------------------------------------------------------------------:|
| `satcb_user`                  | yes       | <Service Account with API RW access>    |               | STRING                | API user name to connect to the Satellite Server in `satcb_hostname`                              |
| `satcb_password`              | yes       | <Encrypted password>                    |               | VAULT ENCRYPED STRING | API user password - stored as a vault encrypted string                                            |

**NOTE:**

>It is _highly_ recomended to encrypt the password in: satcb_password.
> 
> Use the following process:-
> 
> 
>   ansible-vault encrypt_string --vault-password-file <vault password file> '\<password value\>' --name satcb_password
> 
> 
> The above will result in something like the following which you can
> place in the playbook config file.
> 
> 
> satcb_password: !vault |
>           $ANSIBLE_VAULT;1.1;AES256
>           78613037393830313365386631653333356230647432363534383037393339636330343730643161
>           3664663833653439613338303433656362786639393763300a383239326561306339353865653966
>           63396574353865783336383662356238396161663866653332373039383061616666383562636262
>           3863356436393432610a333961336639617232303537326635313065316364726532643732623738
>           38356534733963323564323230303635326137667061363033356637623865796135
> 
> 
> (Note to hackers: the above encrypted text is not a valid password - sorry! :) )
> 
> 
> The vault password used should be stored in a (secure) file and
> referenced when the playbook is run with the following option:-
> 
>   --vault-password-file=\<path to vault password file\>
> 
> e.g.
> 
>   ansible-playbook <playbook name> [<extra vars>]   \
>    --vault-password-file=~/code/ansible/vault/my_vault_password.txt



#### Define the name of custom *configuration*,  *exclude* or *include* file list:-

| Variable name                 | Required  | Default                                 | Choices       | TYPE                  | Comments                                                                                          |
|-------------------------------|:---------:|:---------------------------------------:|:-------------:|:---------------------:|:-------------------------------------------------------------------------------------------------:|
| `satcb_exclude_list_filename` | no        | exclude_list-default.yaml               |               | STRING                | Name of file containing the list of CV/CCV to *EXCLUDE*                                           |
| `satcb_include_list_filename` | no        | include_list-default.yaml               |               | STRING                | Name of file containing the list of CV/CCV to *INCLUDE*                                           |
| `satcb_config_filename`       | no        | satcb_config-default.yaml               |               | STRING                | Custom configuration file
 

**NOTE:**

> If there is a configuration file named:-
> 
>*satcb_config-\<FQDN satellite hostname\>.yaml* then this will take precedence over the 'default' configuration file (*satcb_config-default.yaml*).
 


#### Include / Exclude List

| Variable name                 | Required  | Default                                 | Choices       | TYPE                  | Comments                                                                                          |
|-------------------------------|:---------:|:---------------------------------------:|:-------------:|:---------------------:|:-------------------------------------------------------------------------------------------------:|
| `satcb_exclude_list`          | yes       | <empty list>                            |               | LIST                  | List of CV and/or CCV to *not* publish/promote                                                    |
| `satcb_include_list`          | yes       | <empty list>                            |               | LIST                  | List of CV and/or CCV to publish/promote                                                          |
 

\* required only if `satcb_email_send` is TRUE.


*NOTE*:
>  *This GIT repository provides 'example' files:-*
>
>  *satcb-config_example.yaml*
>  *exclude_list-example.yaml*
>  *include_list-example.yaml*
>
>  *You should copy these to 'default' files. Any future changes required to the config/include/exclude
>  *will be added to the 'example' files.*


## Usage

export ANSIBLE_LOG_PATH=<full pathname to ansible logfile>

ansible-playbook <path to playbook>/tasks/main.yaml                                     \
  --vault-password-file=<path to vault password file to decrypt `satcb_password` var>   \
  --extra-vars='{"satcb_hostname": "<FQDN of satellite server>" [,                      \


optional parameters:-

 
                    "satcb_email_send": <true|false>,                                   \
                    "satcb_email_to": ["<email address>", "<email address>" ... ],      \
                    "satcb_email_cc": ["<email address>", "<email address>" ... ],      \
                    "satcb_email_bcc": ["<email address>", "<email address>" ... ],     \
                    "satcb_email_port": <port number>                                   \
                    "satcb_email_host": "<hostname>"                                    \
                    "satcb_promote_ccv_only": <true|false>,                             \
                    "satcb_promote_ccv_to_env": "<valid lifecycle environment>",        \
                    "satcb_http_protocol": <"https" | "http>,                           \
                    "satcb_url_timeout": <integer>,                                     \
                    "satcb_wait_task_retries": <integer>,                               \
                    "satcb_wait_task_delay": <integer>,


#### Example #1
PUBLISH new versions of all CV and CCV *not* in the exclude list on the DEVELOPMENT

Satellite Server: *my-satellite.acme.com*.

 
**NOTE**:
> The CV & CCV will only be promoted to **LIBRARY**. Also, the *exclude* / *include* list will be derived from the **default** files.

```bash
export ANSIBLE_LOG_PATH=~/code/ansible/log/ansible_$(date +%Y%m%d-%H%M%S.%N).log
ansible-playbook ./satcb/tasks/main.yaml                              \
    --vault-password-file=~/code/ansible/vault/my_vault_password.txt  \
    --extra-vars='{ "satcb_hostname": "my-satellite.acme.com" }'
```


#### Example #2
PROMOTE **ALL** CCV to Lifecycle Environment: *average*


**NOTE**:
> The *exclude* / *include* list will be derived from the **default** files.


```bash
export ANSIBLE_LOG_PATH=~/code/ansible/log/ansible_$(date +%Y%m%d-%H%M%S.%N).log
ansible-playbook ./satcb/tasks/main.yaml                              \
    --vault-password-file=~/code/ansible/vault/my_vault_password.txt  \
    --extra-vars='{ "satcb_hostname": "my-satellite.acme.com",   \
                    "satcb_promote_ccv_only": true                    \
                    "satcb_promote_ccv_to_env": "average" }'
```
 

#### Example #3
PUBLISH new CV **and** PROMOTE CCV to Lifecycle Environment: *pilot* for Satellite Server: *my-satellite.acme.com*

 
```bash
export ANSIBLE_LOG_PATH=~/code/ansible/log/ansible_$(date +%Y%m%d-%H%M%S.%N).log
ansible-playbook ./satcb/tasks/main.yaml                              \
    --vault-password-file=~/code/ansible/vault/my_vault_password.txt  \
    --extra-vars='{ "satcb_hostname": "my-satellite.acme.com",   \
                    "satcb_promote_ccv_to_env": "pilot" }'
```


#### Example #4
PROMOTE CCV to Lifecycle Environment: *average* **excluding** the CCV listed in the exclude file:-

 
*exclude_list-mytest.yaml*
 

```bash
export ANSIBLE_LOG_PATH=~/code/ansible/log/ansible_$(date +%Y%m%d-%H%M%S.%N).log
ansible-playbook ./satcb/tasks/main.yaml                              \
    --vault-password-file=~/code/ansible/vault/my_vault_password.txt  \
    --extra-vars='{ "satcb_hostname": "my-satellite.acme.com",   \
                    "satcb_promote_ccv_only": true                    \
                    "satcb_promote_ccv_to_env": "average"             \
                    "satcb_exclude_list_filename": "exclude_list-mytest.yaml" }'
```
 

**NOTE:**
> By using different include / exclude files when promoting certain CCV can be **limited** to particular Lifecycle Environments.
> 
> i.e. If a new CCV was needed only for *testing* then it could be left out of dedicated include list files for lifecycles beyond **pilot**. Or, it could be added to exclude list files for each lifecycle environment it was not needed for.
 

#### Example #5
Use a *custom* configuration file:-


```bash
export ANSIBLE_LOG_PATH=~/code/ansible/log/ansible_$(date +%Y%m%d-%H%M%S.%N).log
ansible-playbook ./satcb/tasks/main.yaml                              \
    --vault-password-file=~/code/ansible/vault/my_vault_password.txt  \
    --extra-vars='{ "satcb_hostname": "my-satellite.acme.com",   \
                    "satcb_config_filename": "mytest-conf-file.yaml"  }'
```

 

 

## Future Improvements
- Limit publishing of CV's to only those CV's which are in a CCV.
- Limit promoting CCV's only up-to a specific Lifecycle environment.
 
  i.e. If the Lifecycle Path was:-
  
       dev -> test -> uat -> prod
       
       then CCV: ccv-my-test should only get promoted to LE: 'dev' and
       'test' and not to:  'uat' or 'prod'.


- Replace this playbook by creating an Ansible module .
- Only publish a new CV (or CCV) if there has been a change
  (I expect this to be a non-trivial and time-consuming task).
  

## Contributing
Pull requests are welcome. For major changes, please open an issue first to discuss what you would like to change.

 
Please make sure to update tests as appropriate.

 
## License
[GPLv3](https://www.gnu.org/licenses/gpl-3.0.en.html)
