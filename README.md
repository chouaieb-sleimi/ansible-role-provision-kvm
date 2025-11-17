# Ansible Role: Provision KVM

Provisions KVM virtual machines natively on RHEL/CentOS hosts using `libvirt` and `virt-customize`.

**What This Role Does:**

1. **Ensures requirements** are installed (`guestfs-tools`, `python3-libvirt`)
2. **Copies base image** from `libvirt_image_dir` to VM pool directory
3. **Customizes the image** with hostname, root password, and SSH keys using `virt-customize`
4. **Defines the VM** in libvirt using an XML template
5. **Connects to the VM** for post-provisioning tasks

**Dependencies:**

- **Host:**
  - **Infrastructure:** Virtualization must be enabled in the host with at least 1 Virtual Network
  - **Packages:** Only `ansible` is needed to run this role
- **Guest:**
  - **Packages:** `guestfs-tools`, `python3-libvirt` (installed by the role)

---

## Role Variables

For the whole list of variables, see `defaults/main.yml`:

- **`home_dir`:** Home directory of the user whom the public ssh key will be injected in the VM
- **`ssh_key_name`:** Name of the ssh public key to be injected in the VM guests
- **`ssh_key`:** Path to the ssh public key
- **`virt_user`**: User that runs VMs and own their storage disks. When using a custom user, it should belong to the `libvirt` group
- **`virt_group`**: Group of user that runs VMs and own their storage disks. When using a custom user, it should belong to the `libvirt` group
- **`libvirt_pool_dir`:** directory containing `qcow2` base image files
- **`libvirt_image_dir`:** storage directory for vm disk
- **`vm_vcpus`:** VCPUs to assign to VM
- **`vm_ram_mb`:** RAM in mb to assign to VM
- **`vm_net`:** virtual network to assign VMs to
- **`root_pass`:** `root` user password
- **`domain`:** VM's domain name
- **`vm_name`:** VM name (visible in the `virsh list` command)
- **`hostname`:** VM hostname

Custom variables example:

```yaml
# connection info
home_dir: "{{ lookup('ansible.builtin.env', 'HOME') }}"
ssh_key_name: id_ed25519.pub
ssh_key: "{{ home_dir }}/.ssh/{{ ssh_key_name }}"

# vm spec variables
virt_user: "{{ lookup('ansible.builtin.env', 'USER') }}"
virt_group: "{{ lookup('ansible.builtin.env', 'USER') }}"
libvirt_pool_dir: "/var/lib/libvirt/images"
libvirt_image_dir: "/var/lib/libvirt/images"
base_image_name: "rhel-8.7-x86_64-kvm.qcow2"
vm_vcpus: 1
vm_ram_mb: 2048
vm_net: "network_labs"

# guest variables
root_pass: "root"
domain: "ansible.labs"
vm_name: "control_rhel8.7"
hostname: "control"
```

---

## Quick Start

1. **Install Ansible:**

   ```bash
   sudo dnf install -y ansible
   ```

2. **Configure `requirements.yaml`:**

   ```bash
    - name: ansible-role-provision-kvm-native
      scm: git
      src: https://github.com/chouaieb-sleimi/ansible-role-provision-kvm-native.git
      version: main
   ```
    > **Note:** you might need to configure `roles_path` directive in your `ansible.cfg` to include the `./roles` directory

3. **Install role**

   ```bash
   mkdir roles/
   ansible-galaxy install -r requirements.yaml -p ./roles
   ```

4. **Call role in playbook**

   see [usage examples](#usage-examples) below

---

## Usage Examples

For complete examples including multi-VM provisioning and variable customization, see `samples/playbook.yaml`.

### Single VM

```yaml
---
- name: Provision single VM
  hosts: localhost
  become: true
  vars:
    vm_name: "webserver"
    hostname: "web01"
    base_image_name: "rhel-8.7-x86_64-kvm.qcow2"
  roles:
    - ansible-role-provision-kvm
```

### Multiple VMs (Looping)

Use `vars` to define a list and loop through the role:

```yaml
---
- name: Provision multiple VMs
  hosts: localhost
  become: true
  vars:
    vm_list:
      - {
          vm_name: "control",
          hostname: "control",
          base_image_name: "rhel-8.7-x86_64-kvm.qcow2",
        }
      - {
          vm_name: "node1",
          hostname: "node1",
          base_image_name: "AlmaLinux-8-GenericCloud-8.7.x86_64.qcow2",
        }
      - {
          vm_name: "node2",
          hostname: "node2",
          base_image_name: "AlmaLinux-8-GenericCloud-8.7.x86_64.qcow2",
        }
  tasks:
    - ansible.builtin.include_role:
        name: ansible-role-provision-kvm
      vars:
        config_vm_name: "{{ item.vm_name }}"
        config_hostname: "{{ item.hostname }}"
        base_image_name: "{{ item.base_image_name }}"
      loop: "{{ vm_list }}"
```

---

## License

BSD
