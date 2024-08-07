# Setup

Before proceeding, ensure that you have access to the remote host where Ansible will execute its modules. This can be accomplished by copying your public key to the authorized keys on the remote host using the following command:
`ssh-copy-id -i ~/.ssh/id_rsa.pub remote_user@remote_host`

Refer to the included hosts file for a sample inventory, where you can configure the necessary variables.
Under the `[control_plane]` section, you need to specify the host where the management will be deployed.
In the `[control_plane:vars]` section, you must provide the domain isolation service and OVN database (OVNDB) image details. By default, it will use the latest available images.
Additionally, you need to specify the Docker registry details for the NGC organization. This ensures that the required images can be pulled from the appropriate registry.
Additionally, you need to build and specify the ovn_central_image for the OVN central component. An example of such an image can be found in the [Dockerfile.ovn-central](https://github.com/NVIDIA/ovn-isolation-deployment/blob/master/images/Dockerfile.ovn-central) file.
To ensure data persistence for the OVN and PostgreSQL databases, you can mount host volumes to store their data by configuring `ovn_persistency_data` and `postgress_persistency_data`
To configure the internal IP address pool for routing between domain network segments, specify the `internal_ip_address_pool` variable with a comma-separated list of IP addresses and their subnet masks.

```
[control_plane]
<host-domain-name-or-ip>

[control_plane:vars]
ansible_python_interpreter=/usr/bin/python3
ovn_domain_service_image=nvcr.io/cjlqyxpwuczn/ovn-domain-service:latest
ovn_central_image=<ovn_central_image>
docker_registry=nvcr.io
docker_registry_user=<registry_user>
docker_registry_pass=<registry_password>
ovn_persistency_data=/etc/ovn
postgress_persistency_data=/var/lib/postgresql/data
internal_ip_address_pool="100.64.0.0/16,100.65.0.0/16,100.66.0.0/16,100.67.0.0/16,100.68.0.0/16,100.69.0.0/16,100.70.0.0/16"
```

# Run

```sh
ansible-playbook deploy.yaml
```

The execution of this playbook will install the necessary packages required to run containers and Docker Compose. It will then proceed to build and run the OVN central components, deploy the database, and set up the service.
Upon successful completion, you should see all the containers running. For example:

```sh
docker ps
CONTAINER ID   IMAGE                                                    COMMAND                  CREATED        STATUS                  PORTS     NAMES
1f3cb06cc86a   nvcr.io/nvstaging/doca/ovn-domain-service:latest   "/ovn-domain-service"    20 hours ago   Up 20 hours                       ovn-domain-service_ovn-domain-service_1
f9fdc590b3d4   nvcr.io/nvstaging/doca/ovn-central:v24.03.2                "/ovndb-entrypoint.s…"   20 hours ago   Up 20 hours (healthy)             ovn-domain-service_ovndb_1
bc92a38cef7c   postgres                                                 "docker-entrypoint.s…"   20 hours ago   Up 20 hours (healthy)
```

# Cleanup

Stop all the containers on the management node: ovndb, service and DB containers

```sh
ansible-playbook cleanup-deployment.yaml
```