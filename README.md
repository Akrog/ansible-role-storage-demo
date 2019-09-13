# Ansible Storage Role Demo

This is a simple demo of the
[Storage Role](https://github.com/Akrog/ansible-role-storage) using an LVM
backend that uses a loopback device.

The demo is split in 5 different playbooks for two reasons, idempotency and to
be able to manually check things on the hosts.  The playbooks are:

- `01-setup-nodes.yml`
- `02-volume-create-and-attach.yml`
- `03-extend.yml`
- `04-disconnect.yml`
- `05-delete.yml`

The demo requires the above mentioned [Storage
Role](https://github.com/Akrog/ansible-role-storage).  If we are using Ansible
Tower the project will automatically detect the requirements in the
`roles/requirements.yml` and install it.  Otherwise we'll have to install it
ourselves.

### 01-setup-nodes.yml

Since we are not using a "real" storage backend this playbook creates a *fake*
one using a loopback device to create an LVM PV and VG.

This playbook defines the `storage_controller` group for the controller role
and `storage_consumers` group for the consumer role, thus installing in the
system their common requirements.

For the consumer nodes we have to manually install the iSCSI initiator package
since we don't know beforehand that the nodes are going to connect to an iSCSI
backend and the Storage Role does not currently support just-in-time
installation of storage transport dependencies.

By separating this playbook from the others we can increase the speed at which
the other playbooks can be run by telling them not to do the setup of the
providers on the controller role setting `storage_setup_providers` to `no` as
seen in the `controller.yml` file.

### 02-volume-create-and-attach.yml

This playbook creates the volumes on the LVM backend, maps/exports the volumes
to their respective consumer nodes, attaches them to their consumers, makes a
partition, formats it, and finally mounts the filesystem.

### 03-extend.yml

This playbook extends the volume on the backend, detects it on the host where
it's attached, expands the partition, and finally expands the filesystem.

This operation is separated from the volume creation because, even though each
independent task is idempotent, that doesn't mean that the playbooks are,
moreover when we are doing extend operations.  If we create a 1GB volume and
then expand it to 2GB, when we rerun the playbook and the create volume task is
executed it will not find the 1GB volume, and will create a new one instead of
reusing the one that was created on the first playbook run.

### 04-disconnect.yml

This playbook unmounts the volume, detaches it from the consumer node, and
removes the export and mapping from the backend.

### 05-delete.yml

Finally we just delete the volume now that it's detached.
