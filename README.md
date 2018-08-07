# Ansible Playbook for CA Single Sign-On

## Purpose

To enable deployment and configuration management of CA SSO 12.8/Directory 14.0 in a highly automated and repeatable fashion that is independent of how or where the infrastructure is hosted. 

The playbook is provided AS IS with no support or warranty.  It may be redistributed or modified and distributed under the terms of an MIT License. See [License](#license) for details.

#### Ansible Configuration

Ansible 2.5 or higher is required for this playbook.

The following configurations are recommended in /etc/ansible/ansible.cfg (Yum install) or ~/.ansible.cfg (Git install):

- Avoid "Timeout waiting for privilege escalation prompt" error:
  - Increase "timeout" value to 600
- Disable host checking:
  - host_key_checking = False
- Increase ControlPersist timeout to improve performance:
  - ssh_args = -o ControlMaster=auto -o ControlPersist=30m
- Add allow_world_readable_tmpfiles to allow becoming non-privileged user (i.e.: installing product as non-root user):
  - allow_world_readable_tmpfiles = True
- Add ask_vault_pass to prompt for vault password by default:
  - ask_vault_pass = True

## Download and Configure the Playbook

Perform the following on the Ansible control machine.

- If you plan to use database as the policy store, it will need to be prepared and made available outside of the playbook. Otherwise, CA Directory can be deployed as the policy/session/sample user store as part of the SSO playbook.
- Download the playbook to your control machine.
- Edit/create an inventory file in the inventory directory.
  - For example on single-leg environment, use single.sso.
  - For example on HA environment, use ha.sso.
  - For example on test environments running in Docker, use docker.sso.
    - The inventory hostname (first value) corresponds to the local FQDN inside private network.
    - Set ansible_host to the corresponding Docker container name.
    - For services having external FQDN requirements, set external_fqdn to the public DNS name (e.g.: for Admin UI and Access Gateway.)
  - For HA environment, enter as many hosts as required and the software/configurations will be pushed to them accordingly.
  - For SSO HA environment, specify sps_lb with your load balancer FQDN so Access Gateway will be configured to respond to requests to the LB correctly.
  - Optionally specify internal_lb in the pstore inventory group with the FQDN of the internal load balancer for the backend components (e.g.: Directory, SSO Policy Server, etc.) If internal_lb is not defined, the playbook will use product built-in HA/LB mechanisms as appropriate.
  - Use the pstore group to define policy store information.
- Edit the variables files in the vars directory.
  - Utilize existing examples and make changes as necessary. Ansible facts/variables should be used where appropriate to make it as generic/repeatable as possible. Most of the variables defined in the playbook are used by product silent installers and have names that resemble silent install parameters, so please refer to the product documentation for usage information.
- CA product binaries
  - If available, specify a download URL in the vars file so the binaries can be downloaded to endpoints during deployment.
  - Otherwise, download the install binaries into the playbook files directory and modify the vars file to use the correct file names. Comment out the appropriate download_base_url variables to utilize local binaries.
- Edit the playbook vault to have the desired passwords. See below for more details.

## Ansible vault

The playbook utilizes Ansible Vault to store sensitive information so no plaintext passwords are used. A sample vault has been provided as vars/vault.yml. To edit the vault:
- Ensure that you have $EDITOR environment set pointing to your favorite editor.
- Execute `ansible-vault edit vars/vault.yml` to edit the vault. The sample vault uses password CAdemo123.
- Edit the password/key values in the vault as required.
- Please use `ansible-vault rekey` to change the default vault password.

## Execute the Playbook

**NOTE:** Ensure that all endpoints in the inventory have OS package manager configured (e.g.: RHEL Subscription Manager) or the pre-requisite install tasks will fail.

It is recommended that the endpoints are set up with SSH public key authentication to eliminate the need for providing login passwords. The control machine should have SSH agent running with the required key added.

- Test if all machines defined in the inventory are reachable.
```
ansible -i inventory/<inventory_file> all -m ping <--ask-pass>
(e.g.: ansible -i inventory/ha.sso all -m ping)
```
- Run the playbook. Use --ask-pass if you do not have public key authentication configured. Use --ask-become-pass if sudo is used and requires password.
```
ansible-playbook -i inventory/<inventory_file> <--ask-pass> <--ask-become-pass> <playbook>.yml
(e.g.: ansible-playbook -i inventory/ha.sso --ask-pass sso.yml)
```

## License

Copyright (c) 2018 CA Technologies

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
