# Ansible Create Delete Subscription Membership

Has 4 modes, where either the script is run after a memberpress subscription is created, prompted by a webhook. Or a subscription is created manually by completing a survey inside AWX. Created subscriptions can either be a 'DigitalOcean Plan' where the user is assigned a droplet, or a 'On-Premises Plan' where the user is prompted to connect their own hardware suring the provision stage.

# Prerequisites

Relies on the following file on the AWX host: /var/lib/awx/hosting/hosting_vars.yml
```
do_api_token: value
do_spaces_access_key: value
do_spaces_secret_key: value
do_image_master: debian-10-x64
tower_host: value
tower_username: value
tower_password: value
```

# Ansible Create MP (MemberPress) Subscription (create.yml)

INPUT (extra_variables):
- client_first_name
- client_last_name
- client_email - used as the AWX login for that client.
- member_id - a memberpress member number (eg: '25')
- subscription_id - memberpress subscription number (eg: 'I-VCRY4H9EKT9A')
- plan_title - name of the plan, possible values are:
	- 'Small DigitalOcean Server'
	- 'Medium DigitalOcean Server'
	- 'Large DigitalOcean Server'
	- 'Small On-Premises Server'
	- 'Medium On-Premises Server'
	- 'Large On-Premises Server'

PROCESSING: Creates AWX Account for user, creates initial organisation.yml and server_vars.yml file. Also creates initial '{{ subscription_id }} Provision Server' playbook in users account, which will allow the client to select their base url, element client url, the DigitalOcean droplet location or to connect their own On-Premises server.

OUTPUT: Working AWX account at provision stage.

# pre_create.yml

Creates the AWX 'organisation' and 'team' upon MP subscription creation, so they exist before the client logs in to AWX. Makes the initial login process more seamless.

# bind_user_account.yml

After creating a MP subscription and first logging in, this connects a clients AWX account to the appropriate 'team'. Makes the initial login process more seamless. 

# Ansible Create Manual Subscription (create.yml)

INPUT (extra_variables):
- client_first_name
- client_last_name
- client_email - used as the AWX login for that client. 
- client_password
- member_id - a unique value that represents this client.
- plan_title - name of the plan, possible values are:
	- 'Small DigitalOcean Server'
	- 'Medium DigitalOcean Server'
	- 'Large DigitalOcean Server'
	- 'Small On-Premises Server'
	- 'Medium On-Premises Server'
	- 'Large On-Premises Server'

PROCESSING: Creates AWX Account for user, generates them a subscription_id, creates initial organisation.yml and server_vars.yml file. Also creates a '{{ subscription_id }} Provision Server' playbook in users account, which will allow the client to select their base url, element client url, the DigitalOcean droplet location or to connect their own On-Premises server.

OUTPUT: Working AWX account at provision stage.

# Ansible Delete Subscription (delete.yml)

TAGS:
delete-subscription

INPUT (extra_variables): 
- subscription_id
- member_id

PROCESSING: Removes job templates, digitalocean resources, and files/folders associated with a subscription.

# Ansible Delete Membership (delete.yml)

TAGS:
delete-membership

INPUT (extra_variables): 
- member_id

PROCESSING: Playbook to remove clients AWX Organisation and local files on the AWX server.

