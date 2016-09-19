# Ansible Role: NFS

Installs NFS (server and/or client) on RedHat/CentOS or Debian/Ubuntu.

By default this role is opinionated in favor of NFSv4 and squashes a
bunch of daemons such as `idmapd`, `statd`, &c. which are only
necessary for previous NFS versions.  This obviously decreases the
breadth of applicability of this role but results in a system with
fewer moving parts, running processes, and ports to poke firewall
holes for.

## Requirements

None.

## Role Variables

Default usage will install NFS client utilities on a host.  To
configure a host as an NFS server, set the following variables:

* `nfsd_server`: true
* `nfs_exports`: a list of exports (see example below)

Additional variables are defined in `defaults/main.yml`.


## Dependencies

None.

## Example Playbook

```yaml

# This NFS server defines a single export.
- hosts: server0
  roles:
    - role: nfs
	  nfs_server: true
      nfsd_exports:
        - path:    /data
		  acl:     "*"
          options: "rw,sync,no_subtree_check,no_root_squash"

# This NFS server defines two exports with the same ACL.
- hosts: server1
  roles:
    - role: nfs
	  nfs_server: true
	  nfsd_export_acl: "123.123.0.0/16"
      nfsd_exports:
        - path:    /data1
          options: "rw,sync,no_subtree_check,no_root_squash"
        - path:    /data2
          options: "rw,sync,no_subtree_check,no_root_squash"
		  allow_localhost: true  # lets server1 mount this share in addition to the ACL

# This is an NFS client.  Note that you have to explicitly mount the
# NFS shares -- this role does not do it for you.
- hosts: server3
  roles:
    - role: nfs
  post_tasks:
    - name: Make directory for mount
	  file: state=directory path=/data
	  
    - name: Mount server0:/data
	  mount: state=mounted name=/data src=server0:/data fstype=nfs4 opts="sec=sys,noatime,nodiratime"
```

## License

MIT / BSD

## Author Information

This role was created in 2014 by
[Jeff Geerling](http://www.jeffgeerling.com/), author of
[Ansible for DevOps](https://www.ansiblefordevops.com/).  It was
further modified by Dhruv Bansal.
