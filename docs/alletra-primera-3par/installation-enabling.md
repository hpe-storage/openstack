# Installation & Enabling the Driver

The ``HPE3PARFCDriver`` and ``HPE3PARISCSIDriver`` are installed with the
OpenStack software.

1. Install the ``python-3parclient`` Python package on the OpenStack Block
Storage system.

```
$ pip install 'python-3parclient>=4.0,<5.0'
```

2. Verify that the HPE 3PAR Web Services API server is enabled and running on
the HPE 3PAR / Primera / Alletra 9k / Alletra MP storage system.

a. Log onto the HPE 3PAR / Primera / Alletra 9k / Alletra MP storage system
with administrator access.

```
$ ssh 3paradm@<HPE storage system IP Address>
```

b. View the current state of the Web Services API Server.

```
$ showwsapi
-Service- -State- -HTTP_State- HTTP_Port -HTTPS_State- HTTPS_Port -Version-
Enabled   Active Enabled       8008        Enabled       8080       1.1
```

c. If the Web Services API Server is disabled, start it.

```
$ startwsapi
```

3. If the HTTP or HTTPS state is disabled, enable one of them.

```
$ setwsapi -http enable
```

or

```
$ setwsapi -https enable
```

Note

To stop the Web Services API Server, use the :command:`stopwsapi` command. For
other options run the :command:`setwsapi -h` command.

4. If you are not using an existing CPG, create a CPG on the HPE 3PAR / Primera
   / Alletra 9k / Alletra MP storage system to be used as the default location
   for creating volumes.

5. Make the following changes in the ``/etc/cinder/cinder.conf`` file.

```
# WSAPI Server URL.
# This setting applies to all: 3PAR, Primera, Alletra 9k and Alletra MP.
# Example 1: for 3PAR, URL is:
https://<3par ip>:8080/api/v1

# Example 2: for Primera/Alletra 9k/Alletra MP, URL is:
https://<primera/alletra_9k/alletra_mp ip>:443/api/v1

# 3PAR / Primera / Alletra 9k / Alletra MP username with the 'edit' role
hpe3par_username=edit3par

# 3PAR / Primera / Alletra 9k / Alletra MP password for the user specified in hpe3par_username
hpe3par_password=3parpass

# 3PAR / Primera / Alletra 9k / Alletra MP CPG to use for volume creation
hpe3par_cpg=OpenStackCPG_RAID5_NL

# IP address of SAN controller for SSH access to the array
san_ip=10.10.22.241

# Username for SAN controller for SSH access to the array
san_login=3paradm

# Password for SAN controller for SSH access to the array
san_password=3parpass

# FIBRE CHANNEL DRIVER
# (uncomment the next line to enable the FC driver)
#volume_driver=cinder.volume.drivers.hpe.hpe_3par_fc.HPE3PARFCDriver

# iSCSI DRIVER
# If you enable the iSCSI driver, you must also set values
# for hpe3par_iscsi_ips or iscsi_ip_address in this file.
# Note: The iSCSI driver is supported with 3PAR (all versions)
# and Primera (version 4.2 or higher). If you configure iSCSI
# with Primera 4.0 or 4.1, the driver will fail to start.
# (uncomment the next line to enable the iSCSI driver)
#volume_driver=cinder.volume.drivers.hpe.hpe_3par_iscsi.HPE3PARISCSIDriver

# iSCSI multiple port configuration
# hpe3par_iscsi_ips=10.10.220.253:3261,10.10.222.234

# Still available for single port iSCSI configuration
#iscsi_ip_address=10.10.220.253

# Enable HTTP debugging to 3PAR / Primera / Alletra 9k / Alletra MP
hpe3par_debug=False

# Enable CHAP authentication for iSCSI connections.
hpe3par_iscsi_chap_enabled=false

# The CPG to use for Snapshots for volumes. If empty hpe3par_cpg will be
# used.
hpe3par_cpg_snap=OpenStackSNAP_CPG

# Time in hours to retain a snapshot. You can't delete it before this
# expires.
hpe3par_snapshot_retention=48

# Time in hours when a snapshot expires and is deleted. This must be
# larger than retention.
hpe3par_snapshot_expiration=72

# The ratio of oversubscription when thin provisioned volumes are
# involved. Default ratio is 20.0, this means that a provisioned
# capacity can be 20 times of the total physical capacity.
max_over_subscription_ratio=20.0

# This flag represents the percentage of reserved back-end capacity.
reserved_percentage=15
```

Note

You can enable only one driver on each cinder instance unless you enable
multiple back-end support. See the Cinder multiple back-end support
instructions to enable this feature.

Note

You can configure one or more iSCSI addresses by using the
``hpe3par_iscsi_ips`` option. Separate multiple IP addresses with a
comma (``,``). When you configure multiple addresses, the driver selects
the iSCSI port with the fewest active volumes at attach time. The 3PAR
array does not allow the default port 3260 to be changed, so IP ports
need not be specified.

6. Save the changes to the ``cinder.conf`` file and restart the cinder-volume
service.

The HPE 3PAR Fibre Channel and iSCSI drivers are now enabled on your
OpenStack system. If you experience problems, review the Block Storage
service log files for errors.

Note

Following an array reboot, a restart of the Cinder services is required,
as the previous session is no longer valid and a new session must be created.
This is expected behavior.

The following table contains all the configuration options supported by
the HPE 3PAR Fibre Channel and iSCSI drivers.

| Configuration option = Default value| Description |
| ----------------------------------- | ----------- |
| hpe3par_api_url = <> | (String) WSAPI Server URL. This setting applies to: 3PAR, Primera, Alletra 9k and Alletra MP Example 1: for 3PAR, URL is: https://<3par ip>:8080/api/v1 Example 2: for Primera/Alletra 9k/Alletra MP, URL is: https://<primera ip>:443/api/v1 |
| hpe3par_cpg = [OpenStack] | (List of String) List of the 3PAR/Primera/Alletra 9k/Alletra MP CPG(s) to use for volume creation |
| hpe3par_cpg_snap = <> | (String) The 3PAR/Primera/Alletra 9k/Alletra MP CPG to use for snapshots of volumes. If empty the userCPG will be used |
| hpe3par_debug = False | (Boolean) Enable HTTP debugging to 3PAR/Primera/Alletra 9k/Alletra MP |
| hpe3par_iscsi_chap_enabled = False | (Boolean) Enable CHAP authentication for iSCSI connections. |
| hpe3par_iscsi_ips = [] | (List of String) List of target iSCSI addresses to use. |
| hpe3par_password = <> | (String) 3PAR/Primera/Alletra 9k/Alletra MP password for the user specified in hpe3par_username |
| hpe3par_snapshot_expiration = <> | (String) The time in hours when a snapshot expires and is deleted. This must be larger than expiration |
| hpe3par_snapshot_retention = <> | (String) The time in hours to retain a snapshot. You can’t delete it before this expires. |
| hpe3par_target_nsp = <> | (String) The nsp of 3PAR/Primera/Alletra 9k/Alletra MP backend to be used when: (1) multipath is not enabled in cinder.conf. (2) Fiber Channel Zone Manager is not used. (3) the backend is prezoned with this specific nsp only. For example if nsp is 2 1 2, the format of the option’s value is 2:1:2 |
| hpe3par_username = <> | (String) 3PAR/Primera/Alletra 9k/Alletra MP username with the ‘edit’ role |


