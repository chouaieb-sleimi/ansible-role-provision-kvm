# Ansible Role: Provision KVM

Provisions KVM vms natively on RHEL/CentOS hosts

---

## Dependencies

**Infrastructure:** Virtualization must be enabled in the host with at least 1 Virtual Network
**Packages:** Only `ansible` is needed to run this role

---

## Role Variables

For the whole list of variables, see `defaults/main.yml`:

- **`home_dir`:** Home directory of the user whom the public ssh key will be injected in the VM
- **`ssh_key_name`:** Name of the ssh public key to be injected in the VM guests
- **`ssh_key`:** Path to the ssh public key
- **``virt_user``**: User that runs VMs and own their storage disks. When using a custom user, it should belong to the `libvirt` group
- **``virt_group``**: Group of user that runs VMs and own their storage disks. When using a custom user, it should belong to the `libvirt` group
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

## Example Playbook

You can use this playbook to easily provision a list of VMs with a great deal of control over the specs of each VM. You only need to customize the `vm_list` list variable before looping the role over it. example:

```yaml
---
- name: Provision KVM hosts
  hosts: localhost
  become: true
  vars:
    vm_list:
      - {
          vm_name: "control_rhel8.7",
          hostname: "control",
          base_image_name: "rhel-8.7-x86_64-kvm.qcow2",
        }
      - {
          vm_name: "node-1_alma8.7",
          hostname: "node1",
          base_image_name: "AlmaLinux-8-GenericCloud-8.7.x86_64.qcow2",
        }
      - {
          vm_name: "node-2_alma8.7",
          hostname: "node2",
          base_image_name: "AlmaLinux-8-GenericCloud-8.7.x86_64.qcow2",
        }
      - {
          vm_name: "node-3_alma8.7",
          hostname: "node3",
          base_image_name: "AlmaLinux-8-GenericCloud-8.7.x86_64.qcow2",
        }
      - {
          vm_name: "node-4_alma8.7",
          hostname: "node4",
          base_image_name: "AlmaLinux-8-GenericCloud-8.7.x86_64.qcow2",
        }

  tasks:
    - name: Include role kvm_provision
      ansible.builtin.include_role:
        name: kvm_provision
      vars:
        vm_name: "{{ item.vm_name }}"
        hostname: "{{ item.hostname }}"
        base_image_name: "{{ item.base_image_name }}"
      loop: "{{ vm_list }}"
```

## License

BSD
