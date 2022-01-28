Ansible Role - Fedora DNF Local Plugin
=========

Installs and configures the DNF Local Plugin on the target host (python3-dnf-plugin-local-4.0.18-1.fc33.noarch at the time of this writing). This plugin is useful when there are multiple machines on the same local area network that use Fedora and execute dnf to install or update rpms. These machines can be VMs, Raspberry PIs, desktops, and so on. 

Once one machine downloads and installs an rpm via dnf, other machines of the same architecture sharing the same local repository via the plugin retrieve that rpm from the local repository and not across the internet. This will reduce internet bandwidth consumption and reduce the traffic load on the volunteer Fedora mirror community. Conducting a DNF update across the fleet of machines should also be collectively faster.

Requirements
------------

The plugin will create the repository path if needed and will also initiate the repository if needed. If all machines are VMs on the same physical hardware, then the repository path can be local to that machine. If machines are on a local network then the repository needs to be on a shared file system. In the spirit of focusing a role on a single purpose, this role will not attach a shared file system but there is an example below of how this can easily be accomplished with an NFS share and a separate role that attaches the NFS share.

Role Variables
--------------

End user configurable variables are listed below, along with default values (see `defaults/main.yaml`):

| Variable   | Default Value | Notes |
| ---------- | ------------- | ----- |
| dlp_enabled | true | Enables (true) or disables (false) use of local repository |
| dlp_repodir | /var/lib/dnf/plugins/local | filesystem directory for the local repository |
| dlp_createrepo_enabled | true    | Enable plugin to create local repo if repodir does not contain repository artifacts |
| dlp_nfs_utils | true    | Ensure nfs_utils package is installed when true. |
| dlp_nfs_utilt_fast | false | Skips most tasks if true and dnf local plugin is already installed. |

The variable below is defined in `./vars/main.yaml` and not intended for modification at runtime.
| Variable   | Default Value | Notes |
| ---------- | ------------- | ----- |
| dlp_packages | python3-dnf-plugin-local<br>createrepo_c | The packages installed by the role |

Dependencies
------------

None at this time

Technical Notes
---------------

The plugin creates and uses a local repository as a directory on a filesystem shared between all client machines. As such, performance may not be acceptable if the shared filesystem is accessed on a wide area network. A shared filesystem on a local area network should be acceptable in most cases. 

If the local repository is located on a shared filesystem such as nfs be aware that there could be problems if more than one (1) machine is executing DNF and using the local repository oncurrently. It may be prudent to run dnf serially rather than in parallel across the fleet but I have not explored this in any detail.

TBD - simple example on using dnf repomanage to manage repo contents

Example Playbooks
----------------

The partial playbook below executes the role with default options. After completion, the next time dnf is executed, the plugin will create the directory for the local repository if needed, download and install the rpms identified by dnf, copy the rpms to the new local repository, and run createrepo_c, also if needed. On a future run of dnf, the local repository is checked first before sending the query to a fedora mirror. 
```
- name: Default plugin options
  hosts: all
  become: true
  tasks:
    - name: "DNF Local Plugin role"
      include_role:
        name: "fedora_dnf_local_plugin"
```

The partial playbook below, attaches an nfs shared drive at the `/mnt/repodir` directory on each client host and then installs the dnf local plugin. The plugin, on each client, is configured to use the nfs filesystem share at `/mnt/repodir` for the local repository. The NFS role can be found at https://github.com/buckaroogeek/ansible-role-fedora-nfs-mount. One feature of the nfs client role used below is the use of systemd automount (https://www.freedesktop.org/software/systemd/man/systemd.automount.html). The nfs share is not attached until needed and then dropped when no longer in use. This can reduce network and cpu load.

```
hosts: all
become: true
vars:
  repodir = /mnt/repodir

- name: Configure NFS client
  tasks:
    - name: "Include fedora_nfs_mount"
      include_role:
        name: "fedora_nfs_client"
      vars:
        fnm_server: nfsserver.lan
        fnm_export: /srv/nfs-share/dnfrepo
        fnm_mnt_path: "{{ repoddir }}"

- name: Configure dnf local plugin to use nfs share
  tasks:
    - name: "DNF Local Plugin role"
      include_role:
        name: "fedora_dnf_local_plugin"
      vars:
        dlp_repodir: "{{ repodir }}"
```

License
-------

BSD

Author Information
------------------

Author: Bradley G Smith

email: bradley.g.smith@gmail.com
