Fedora DNF Local Plugin Ansible Role
=========

Installs and configures the DNF Local Plugin on the target host (python3-dnf-plugin-local-4.0.18-1.fc33.noarch at the time of this writing). This plugin is useful when there are multiple machines on the same network that use Fedora (VMs, Raspberry PIs, desktops, etc). 

Once one machine downloads and installs an rpm via dnf, other machines of the same architecture sharing the same local repository via the plugin retrieve that rpm from the local repo and not across the internet. This will reduce internet bandwidth consumption and reduce the traffic load on the volunteer Fedora mirror community. Conducting a DNF update across the fleet of machines should also be collectively faster.

Requirements
------------

The plugin will create the repository path if needed and will also initiate the repository if needed. If all machines are VMs on the same physical hardware, then the repository path can be local to that machine. If machines are on a local network then the repository needs to be on a shared file system. In the spirit of focusing a role on a single purpose, this role will not attach a shared file system but there is an example below of how this can easily be accomplished with an NFS share and a Fedora NFS Attach role.

Role Variables
--------------

Available variables are listed below, along with default values (see `defaults/main.yaml`):

| Variable   | Default Value | Notes |
| ---------- | ------------- | ----- |
| dlp_enabled | true | Enables (true) or disables (false) use of local repodir |
| dpl_repodir | /var/lib/dnf/plugins/local | filesystem directory for the local repository |
| dpl_createrepo_enabled | true    | Enable plugin to create local repo if repodir does not contain repository artifacts |

The variable below is defined in `./vars/main.yaml` and not intended for modification at runtime.
| Variable   | Default Value | Notes |
| ---------- | ------------- | ----- |
| dlp_packages | python3-dnf-plugin-local<br>createrepo_c | The packages installed by the role |

Dependencies
------------

None at this time

Technical Notes
---------------

If the local repository is located on a shared filesystem such as NSF be aware that there could be problems if more than one (1) machine is executing DNF and using the local repository oncurrently. It is prudent to run dnf serially rather than in parallel across the fleet.

Example Playbook
----------------


License
-------

BSD

Author Information
------------------

Author: Bradley G Smith
email: bradley.g.smith@gmail.com
