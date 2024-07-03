# setup

Before proceeding, ensure that you have access to the remote hosts where Ansible will execute its modules. This can be accomplished by copying your public key to the authorized keys on the remote host using the following command:
`ssh-copy-id -i ~/.ssh/id_rsa.pub remote_user@remote_host`

Refer to the included hosts file for a sample inventory, where you can configure the necessary variables.

To configure the control-plane node, please consult the README.md file located at playbooks/control-plane/.
For configuring the DPU node, refer to the README.md file located at playbooks/dpu/.

# run

```sh
ansible-playbook deploy.yaml
```

The execution of this playbook involves importing two other playbooks: the first one for the control-plane execution and the second one for the DPU execution.

# cleanup

cleans up the configuration on both control-plane and dpu nodes

```sh
ansible-playbook cleanup-deployment.yaml
```
