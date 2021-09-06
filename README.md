# Ansible Create Delete Subscription Membership

For creating a subscription there are 4 modes, where either the script is run after a memberpress subscription is created which generates a webhook to AWX. Or a subscription is created manually by completing a survey inside AWX. Created subscriptions can either be a 'DigitalOcean Plan' where the user is assigned a droplet, or a 'On-Premises Plan' where the user is prompted to connect their own hardware during the provision stage.

This repository also contains the playbooks needed to delete a subscription or membership. With the MemberPress subscription deletion being prompted by the 'subscription-expired' MemberPress webhook.


# Prerequisites

You need to setup the AWX system to use this playbook, see the (create-awx-system repository)[https://gitlab.com/GoMatrixHosting/create-awx-system].


# 00 - Create MP Subscription (create.yml)

INPUT (extra_variables):
- client_first_name
- client_last_name
- client_email - used as the AWX login for that client.
- member_id - a memberpress member number (eg: '25')
- subscription_id - memberpress subscription number (eg: 'I-VCRY4H9EKT9A')
- plan_title - name of the plan, possible values are:
	- 'Micro DigitalOcean Server'
	- 'Small DigitalOcean Server'
	- 'Medium DigitalOcean Server'
	- 'Large DigitalOcean Server'
	- 'Jumbo 500 DigitalOcean Server'
	- 'Jumbo 1000 DigitalOcean Server'
	- 'Jumbo 2000 DigitalOcean Server'
	- 'Jumbo 5000 DigitalOcean Server'
	- 'Micro On-Premises Server'
	- 'Small On-Premises Server'
	- 'Medium On-Premises Server'
	- 'Large On-Premises Server'
	- 'Jumbo 500 On-Premises Server'
	- 'Jumbo 1000 On-Premises Server'
	- 'Jumbo 2000 On-Premises Server'
	- 'Jumbo 5000 On-Premises Server'

PROCESSING: Creates AWX Account for user, creates initial organisation.yml and server_vars.yml file. Also creates initial '{{ subscription_id }} Provision Server' playbook in users account, which will allow the client to select their base url, element client url, the DigitalOcean droplet location or to connect their own On-Premises server.

OUTPUT: Working AWX account at provision stage.


# pre_create.yml

Creates the AWX 'organisation' and 'team' upon MP subscription creation, so they exist before the client logs in to AWX. Makes the initial login process more seamless.


# bind_user_account.yml

After creating a MP subscription and first logging in, this connects a clients AWX account to the appropriate 'team'. Makes the initial login process more seamless. 


# 00 - Create Manual Subscription (create.yml)

INPUT (extra_variables):
- client_first_name
- client_last_name
- client_email - used as the AWX login for that client. 
- client_password
- member_id - a unique value that represents this client.
- plan_title - name of the plan, possible values are:
	- 'Micro DigitalOcean Server'
	- 'Small DigitalOcean Server'
	- 'Medium DigitalOcean Server'
	- 'Large DigitalOcean Server'
	- 'Jumbo 500 DigitalOcean Server'
	- 'Jumbo 1000 DigitalOcean Server'
	- 'Jumbo 2000 DigitalOcean Server'
	- 'Jumbo 5000 DigitalOcean Server'
	- 'Micro On-Premises Server'
	- 'Small On-Premises Server'
	- 'Medium On-Premises Server'
	- 'Large On-Premises Server'
	- 'Jumbo 500 On-Premises Server'
	- 'Jumbo 1000 On-Premises Server'
	- 'Jumbo 2000 On-Premises Server'
	- 'Jumbo 5000 On-Premises Server'

PROCESSING: Creates AWX Account for user, generates them a subscription_id, creates initial organisation.yml and server_vars.yml file. Also creates a '{{ subscription_id }} Provision Server' playbook in users account, which will allow the client to select their base url, element client url, the DigitalOcean droplet location or to connect their own On-Premises server.

OUTPUT: Working AWX account at provision stage.


# 00 - Delete Subscription (schedule_delete_subscription.yml)

TAGS:
schedule-delete-subscription

INPUT (extra_variables): 
- subscription_id
- member_id

PROCESSING: Schedules deletion of a subscription after 0 (immediate) to N hours and stops the matrix services. Data export and SFTP access is still possible before deletion. The deletion can be cancelled by the administrator, removing the schedule and starting the Matrix services again. If the schedule triggers it removes job templates, digitalocean resources, and files/folders associated with that subscription, then executes the '00 - Cleanup Deletion Template'.


# 00 - Delete Membership (delete_member.yml)

TAGS:
delete-membership

INPUT (extra_variables): 
- member_id

PROCESSING: Playbook to remove a members AWX account and organisation. It also deletes local files on the AWX server associated with that subscription..

