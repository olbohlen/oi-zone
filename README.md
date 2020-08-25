Role Name
=========

This role is supposed to deploy a new OpenIndiana Zone, with different settings. It will also allow to provision (multiple) ZFS filesystems,
vnics and other related resources.

Requirements
------------

This role obiously requires an OpenIndiana host system and following ansible modules:
- solaris_zone
- zfs
- dladm_vnic

Role Variables
--------------

You need to create a data structure called "oizone". The mandatory minimum is shown here:
```
oizone:
  name: samplezone                             # name of the zone
  zoneroot: /export/zones/                     # basedir for zoneroot
  brand: ipkg                                  # which brand does the zone shall have
  #uninstall: true                             # if set to true, we will remove all we would create
  updateinventory: true                        # will put the new zone on the first line of your local inventory
  autoboot: "true"                             # shall the zone boot upon system boot
  iptype: exclusive                            # exclusive-ip or shared-ip zone
  filesystems:                                 # list of filesystems, zoneroot is mandatory
    - path: rpool/export/zones/samplezone      # dataset for zoneroot
      type: zoneroot                           # type can be: zoneroot, dataset (add zfs dataset), lofs (mount loopback), volume (map a device)
      zfscreate: true                          # shall the role create this for you?
      extra_zfs_properties:                    # extra zfs parameters, see ansible-doc zfs
        mountpoint: /export/zones/samplezone   # ensure providing the correct mountpoint
  nics:                                        # list of NICs for the zone
    - physical: oce0                           # physical interface of the host
      logical: samplezone0                     # name of the VNIC
      address: 172.18.1.199/22                 # IPaddr/CIDR
      addrsuffix: v4                           # address suffix for ipadm
  sysding:                                     # needed parameters for the sysding(1M) file
    timezone: CET                              # which timezone shall the zone be in
    locale: de_DE.UTF-8                        # any special locale?
    ip:                                        # IP configuration
      routes:                                  # list of routes
        - target: default                      # can be a CIDR or a host ip or "default"
          router: 172.18.0.200                 # IP of the router
      dns:                                     # DNS settings
        nameservers:                           # list of nameservers
          - 1.1.1.1
	  - 8.8.8.8
        search:                                # DNS search list
          - example.com
          - openindiana.org
        domain: notebook.lan                   # local DNS domain
    users:                                     # list of users to be created
      - name: root                             # root is mandatory, only specify
        hashedpassword: "$5$..."               # hashedpassword here, everything else is pre-defined
      - name: localadm                         # new user "localadm"
        uid: 100                               
        gid: 10
        shell: /usr/bin/bash
        gecos: "Local Admin Account"
        home: /export/home/localadm
        sudoers: "localadm ALL=(ALL) NOPASSWD: ALL"  # line to be added to sudoers
        hashedpassword: "$5$..."               # hashed password
        authorized_key: "ssh-rsa ..."          # ssh public key to be added to authorized_keys

```

A set of all possible attributes is here:

```
oizone:
  name: oizone
  updateinventory: true
  #uninstall: true
  zoneroot: /export/zones/
  autoboot: "true"
  bootargs: # -v
  iptype: exclusive
  cpus: dedicated # dedicated or capped-cpu
  ncpus: 1
  mem: capped-memory # or nil
  ram: 1G
  swap: 1G
  locked: 1G
  brand: ipkg
  filesystems:
    - path: rpool/export/zones/oizone
      type: zoneroot
      zfscreate: true
      extra_zfs_properties:
        refquota: 10G
	mountpoint: /export/zones/oizone
    - path: apppool/oizone/datavol1
      type: volume
      zfscreate: true
      extra_zfs_properties:
        volsize: 5G
    - path: apppool/oizone/dataset1
      type: dataset
      zfscreate: true
      extra_zfs_properties:
        quota: 2G
    - path: /disk1
      type: lofs
      mountpoint: /hostdisks/disk1
      zfscreate: false
      options:
        - ro
        - nodevices
  nics:
    - physical: ixgbe0
      logical: oizoneint0
      vlan: 100
      address: dhcp  # can be "dhcp" or a regular IP address
      addrsuffix: v4 # can be a string, interface0/suffix will be the ipadm create-addr
  kvm:
    vnc: "on"
    bootorder: cd
  sysding:
    timezone: UTC
    locale: C
    ip:
      routes:
        - target: default # can be a CIDR or a host ip or "default"
          router: 172.18.0.200 # IP of the router
      dns:
        nameservers:
          - 1.1.1.1
          - 8.8.8.8
        search:
          - example.com
          - openindiana.org
        domain: example.com
    users:
      - name: root
        hashedpassword: "$5$foobar...."
      - name: localadm
        uid: 100
        gid: 10
        shell: /usr/bin/bash
        gecos: "Local Admin Account"
        home: /export/home/localadm
        hashedpassword: "$5$barfoo...."
```

Dependencies
------------

No dependencies to others roles so far.

Example Playbook
----------------

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

    - hosts: servers
      vars:
        - oizone:
	    [... see above ..]
      roles:
         - role: oi-zone

License
-------

BSD 3-Clause

Copyright 2020 Olaf Bohlen

Redistribution and use in source and binary forms, with or without modification, are permitted provided that the following conditions are met:

1. Redistributions of source code must retain the above copyright notice, this list of conditions and the following disclaimer.

2. Redistributions in binary form must reproduce the above copyright notice, this list of conditions and the following disclaimer in the documentation and/or other materials provided with the distribution.

3. Neither the name of the copyright holder nor the names of its contributors may be used to endorse or promote products derived from this software without specific prior written permission.

THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS" AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.

Author Information
------------------

Olaf Bohlen <olbohlen@eenfach.de>
