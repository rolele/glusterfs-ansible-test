
```
cd ansible
vagrant up
ansible-playbook --connection=ssh glusterfs.yml
```

The first time I run ``` ansible-playbook --connection=ssh glusterfs.yml``` I get the error bellow that will stop ansible:
````
TASK [glusterfs : Configure Gluster volume.] ***********************************
fatal: [manager0]: FAILED! => {"changed": false, "failed": true, "msg": "error running gluster (/sbin/gluster volume create gluster replica 2 transport tcp manager0:/srv/gluster/brick manager1:/srv/gluster/brick manager2:/srv/gluster/brick worker0:/srv/gluster/brick force) command (rc=1): volume create: gluster: failed: Host worker0 is not in 'Peer in Cluster' state\n"}
````

If I connect to manager0 and check the status I this everything is ok:
```
vagrant ssh manager0
sudo su
[root@manager0 vagrant]# gluster peer status
Number of Peers: 3

Hostname: manager1
Uuid: 7805e0d8-4c29-405b-8603-12bda680cc01
State: Peer in Cluster (Connected)

Hostname: manager2
Uuid: 67d01bf8-5955-44b5-9236-ccc320bc442e
State: Peer in Cluster (Connected)

Hostname: worker0
Uuid: c386cfdb-9430-49b9-8879-44a6879d8c0a
State: Peer in Cluster (Connected)
[root@manager0 vagrant]# gluster pool list
UUID                                    Hostname        State
7805e0d8-4c29-405b-8603-12bda680cc01    manager1        Connected
67d01bf8-5955-44b5-9236-ccc320bc442e    manager2        Connected
c386cfdb-9430-49b9-8879-44a6879d8c0a    worker0         Connected
9702469c-2a35-45fb-92e4-28a6db51eade    localhost       Connected
```


If I re-run the command the ansible error disapear.
I check the status, nothing has changed
```
[root@manager0 vagrant]# gluster peer status
Number of Peers: 3

Hostname: manager1
Uuid: 7805e0d8-4c29-405b-8603-12bda680cc01
State: Peer in Cluster (Connected)

Hostname: manager2
Uuid: 67d01bf8-5955-44b5-9236-ccc320bc442e
State: Peer in Cluster (Connected)

Hostname: worker0
Uuid: c386cfdb-9430-49b9-8879-44a6879d8c0a
State: Peer in Cluster (Connected)
```


I would like to be able to run the ansible role and it works right away.
I tried adding a pause delay before the failing test but this does not solve the problem.

If you want to run into the error again:
```
cd ansible
yes | vagrant destroy
vagrant up
ansible-playbook --connection=ssh glusterfs.yml
```

full output
```
PLAY [gluster] *****************************************************************

TASK [setup] *******************************************************************
ok: [manager0]
ok: [manager2]
ok: [worker0]
ok: [manager1]

TASK [geerlingguy.glusterfs : Include OS-specific variables.] ******************
ok: [manager1]
ok: [manager2]
ok: [manager0]
ok: [worker0]

TASK [geerlingguy.glusterfs : Install Prerequisites] ***************************
changed: [manager1] => (item=[u'centos-release-gluster'])
changed: [worker0] => (item=[u'centos-release-gluster'])
changed: [manager2] => (item=[u'centos-release-gluster'])
changed: [manager0] => (item=[u'centos-release-gluster'])

TASK [geerlingguy.glusterfs : Install Packages] ********************************
changed: [worker0] => (item=[u'glusterfs-server', u'glusterfs-client'])
changed: [manager0] => (item=[u'glusterfs-server', u'glusterfs-client'])
changed: [manager2] => (item=[u'glusterfs-server', u'glusterfs-client'])
changed: [manager1] => (item=[u'glusterfs-server', u'glusterfs-client'])

TASK [geerlingguy.glusterfs : Add PPA for GlusterFS.] **************************
skipping: [manager0]
skipping: [manager1]
skipping: [manager2]
skipping: [worker0]

TASK [geerlingguy.glusterfs : Ensure GlusterFS will reinstall if the PPA was just added.] ***
skipping: [manager0] => (item=[])
skipping: [manager1] => (item=[])
skipping: [manager2] => (item=[])
skipping: [worker0] => (item=[])

TASK [geerlingguy.glusterfs : Ensure GlusterFS is installed.] ******************
skipping: [manager0] => (item=[])
skipping: [manager1] => (item=[])
skipping: [manager2] => (item=[])
skipping: [worker0] => (item=[])

TASK [geerlingguy.glusterfs : Ensure GlusterFS is started and enabled at boot.]
changed: [manager1]
changed: [worker0]
changed: [manager0]
changed: [manager2]

TASK [glusterfs : Ensure Gluster brick and mount directories exist.] ***********
changed: [manager1] => (item=/srv/gluster/brick)
changed: [worker0] => (item=/srv/gluster/brick)
changed: [manager0] => (item=/srv/gluster/brick)
changed: [manager2] => (item=/srv/gluster/brick)
changed: [manager1] => (item=/mnt/gluster)
changed: [worker0] => (item=/mnt/gluster)
changed: [manager0] => (item=/mnt/gluster)
changed: [manager2] => (item=/mnt/gluster)

TASK [glusterfs : pause] *******************************************************
Pausing for 10 seconds
(ctrl+C then 'C' = continue early, ctrl+C then 'A' = abort)
ok: [manager0]

TASK [glusterfs : Configure Gluster volume.] ***********************************
fatal: [manager0]: FAILED! => {"changed": false, "failed": true, "msg": "error running gluster (/sbin/gluster volume create gluster replica 2 transport tcp manager0:/srv/gluster/brick manager1:/srv/gluster/brick manager2:/srv/gluster/brick worker0:/srv/gluster/brick force) command (rc=1): volume create: gluster: failed: Host worker0 is not in 'Peer in Cluster' state\n"}

NO MORE HOSTS LEFT *************************************************************
        to retry, use: --limit @/Users/others/glusterfs-ansible-test/ansible/glusterfs.retry

PLAY RECAP *********************************************************************
manager0                   : ok=7    changed=4    unreachable=0    failed=1
manager1                   : ok=6    changed=4    unreachable=0    failed=0
manager2                   : ok=6    changed=4    unreachable=0    failed=0
worker0                    : ok=6    changed=4    unreachable=0    failed=0
```

