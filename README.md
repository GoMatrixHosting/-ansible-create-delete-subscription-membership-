# Ansible Create Delete Subscription Membership

For creating a subscription there are 4 modes, where either the script is run after a memberpress subscription is created which sends a webhook to AWX. Or a subscription is created manually by completing a survey inside AWX. Created subscriptions can either be a 'DigitalOcean Plan' where the user is assigned a droplet, or a 'On-Premises Plan' where the user is prompted to connect their own hardware during the provision stage.

This repository also contains the playbooks needed to delete a subscription or membership. With the MemberPress subscription deletion being prompted by the another webhook to AWX.


# Prerequisites

You need to setup the AWX system to use this playbook, see the [create-awx-system repository](https://gitlab.com/GoMatrixHosting/create-awx-system).


# 00 - Create Account (pre_create.yml)

Creates the AWX 'organisation' and 'team' upon MP subscription creation, so they exist before the client logs in to AWX. Makes the initial login process more seamless. 

Trigger: The "member-added" webhook.


# 00 - Bind User Account (bind_user_account.yml)

After creating a MP subscription and first logging in, this connects a clients AWX account to the already created team/organisation. Makes the initial login process more seamless. 

Trigger: First user login to the AWX system causes swatchdog to launch the job template.


# 00 - Create MP Subscription (create.yml)

After recieving the initial payment, this creates a new MemberPress DigitalOcean or On-Premises subscription for that user. 

Trigger: The "recurring-transaction-completed" webhook.

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

PROCESSING: It creates the credentials, projects, inventory resources in AWX for that subscription, as well as the initial configuration files. Also creates initial '{{ subscription_id }} Provision Server' playbook in users account.

OUTPUT: Working AWX account at provision stage.


# 00 - Create Manual Subscription (create.yml)

After being executed by the AWX admin, this first creates the user/team/organisation, then it creates a new DigitalOcean or On-Premises subscription for that user. 

Trigger: Manually running that job template in AWX.

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

PROCESSING: Creates AWX user/team/organisation initially. Then it creates the credentials, projects, inventory resources in AWX for that subscription, as well as the initial configuration files. Also creates initial '{{ subscription_id }} Provision Server' playbook in users account.

OUTPUT: Working AWX account at provision stage.


# 00 - Delete Subscription (schedule_delete_subscription.yml)

Schedules deletion of a subscription after 0 (immediate) to N hours and stops the matrix services. Data export and SFTP access is still possible before deletion. The deletion can be cancelled by the administrator, removing the schedule and starting the Matrix services again. If the schedule triggers it removes job templates, digitalocean resources, and files/folders associated with that subscription, then executes the '00 - Cleanup Deletion Template'.

Trigger: The "subscription-expired" webhook.


# 00 - Delete Membership (delete_member.yml)

Playbook to remove a members AWX account and organisation. It also deletes local files on the AWX server associated with that subscription.

Trigger: Manually running that job template in AWX.


# 00 - Backup All Servers (backup_all.yml)

Performs a sequential backup of every server connected to AWX.

Trigger: The daily schedule, or manually running that job template in AWX.


# 00 - Create Wireguard Server (setup_wireguard_server.yml)

Configures a wireguard server AWX can use to SSH into an on-premises server.

Trigger: Manually running that job template in AWX.


# 00 - Deploy/Update All Servers (deploy_all.yml)

Checks for updates hourly, if ones available it applies it to every Matrix server connected to AWX.

Trigger: The hourly schedule, or manually running that job template in AWX.


# 00 - Reprovision All Servers (reprovision_all.yml)

Re-provisions every Matrix server connected to AWX.

Trigger: Manually running that job template in AWX.


# 00 - Rotate SSH Keys (rotate_ssh_all.yml)

Rotates the client SSH key for every server and re-creates all the SSH credentials in AWX.

Trigger: Manually running that job template in AWX.


## License

    Copyright (C) 2021 GoMatrixHosting.com

    This program is free software: you can redistribute it and/or modify
    it under the terms of the GNU Affero General Public License as published by
    the Free Software Foundation, either version 3 of the License, or
    (at your option) any later version.

    This program is distributed in the hope that it will be useful,
    but WITHOUT ANY WARRANTY; without even the implied warranty of
    MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
    GNU Affero General Public License for more details.

    You should have received a copy of the GNU Affero General Public License
    along with this program.  If not, see <https://www.gnu.org/licenses/>.
