# Installation & Enabling the Driver

The ``HPEAlletraMPB10000ShareDriver`` is installed with the OpenStack Shared
File Systems (Manila) software.

## Enabling the Driver in Manila

Make the following changes in the ``/etc/manila/manila.conf`` file.

```
[DEFAULT]
# Comma-separated list of backend names to enable.
enabled_share_backends = alletra1
# Enable the NFS share protocol.
enabled_share_protocols = NFS

[alletra1]
# Name of the backend reported to the scheduler.
share_backend_name = hpealletra1

# The HPE Alletra MP B10000 share driver.
share_driver = manila.share.drivers.hpe.alletra_mp_b10000.hpe_alletra_driver.HPEAlletraMPB10000ShareDriver

# This driver does not support share servers (share networks).
driver_handles_share_servers = False

# Alletra WSAPI V3 Server URL, for example https://<alletra ip>:8080/api/v3
hpealletra_wsapi_url = https://<alletra ip>:8080/api/v3

# Alletra username with the 'edit' role.
hpealletra_username = <username>

# Alletra password for the user specified in hpealletra_username.
hpealletra_password = <password>

# Enable HTTP debugging to Alletra.
hpealletra_debug = False
```

Save the changes to the ``manila.conf`` file and restart the manila-share
service.

The HPE Alletra MP B10000 share driver is now enabled on your OpenStack
system. If you experience problems, review the Shared File Systems service log
files for errors.

The following table contains all the configuration options supported by the
HPE Alletra MP B10000 share driver.

| Configuration option = Default value | Description |
| ------------------------------------ | ----------- |
| hpealletra_wsapi_url = <> | (String) Alletra WSAPI V3 Server URL like https://<alletra ip>:8080/api/v3 |
| hpealletra_username = <> | (String) Alletra username with the 'edit' role |
| hpealletra_password = <> | (String) Alletra password for the user specified in hpealletra_username |
| hpealletra_debug = False | (Boolean) Enable HTTP debugging to Alletra |
| share_driver = <> | (String) Full class path of the HPE Alletra MP B10000 share driver |
| share_backend_name = <> | (String) Name of the backend reported to the Manila scheduler |
| driver_handles_share_servers = False | (Boolean) Must be set to False; share networks are not supported |
| enabled_share_protocols = NFS | (List) Share protocols to enable; only NFS is supported by this driver |



## Pre-Configuration on the HPE Alletra MP B10000

Before configuring the Manila driver, verify that the file service is running
on the array:

```
$ showfileservice
```

If the file service is not enabled, run:

```
$ setfileservice enable
```

If ``setfileservice enable`` fails, the file ports may not be configured
correctly. Check the port persona of slot 4, ports 1 & 2 using ``showport``.
The ports must be configured as file persona ports before the file service
can be enabled.

If the ports are iSCSI, delete and reconfigure them as file ports:

```
$ controliscsiport delete 0:4:1
$ controliscsiport delete 0:4:2
$ controliscsiport delete 1:4:1
$ controliscsiport delete 1:4:2
```

If the ports are NVMe, run the following commands instead:

```
$ controlport nvme delete 0:4:1
$ controlport nvme delete 0:4:2
$ controlport nvme delete 1:4:1
$ controlport nvme delete 1:4:2
```

After converting the ports, reboot the cluster:

```
$ shutdownsys reboot
```

After the cluster comes up, assign IP addresses to all four ports
(0:4:1, 0:4:2, 1:4:1, 1:4:2), then enable the file service and verify:

```
$ setfileservice enable
$ showfileservice
```

Also verify the WSAPI V3 service is enabled and running:

```
$ showwsapi
```
