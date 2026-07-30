# Extra Specs & QoS

Volume type support for both HPE 3PAR drivers includes the ability to set the following capabilities in the OpenStack Block Storage API __cinder.api.contrib.types_extra_specs__ volume type extra specs extension module:

- hpe3par:snap_cpg
- hpe3par:provisioning
- hpe3par:persona
- hpe3par:vvs
- hpe3par:flash_cache
- hpe3par:compression

To work with the default filter scheduler, the key values are case sensitive and scoped with __hpe3par:__. For information about how to set the key-value pairs and associate them with a volume type, run the following command:

```
openstack help volume type
```

Note

Volumes that are cloned only support the extra specs keys cpg, snap_cpg, provisioning and vvs. The others are ignored. In addition the comments section of the cloned volume in the HPE 3PAR / Primera / Alletra 9k / Alletra MP array is not populated.

If volume types are not used or a particular key is not set for a volume type, the following defaults are used:

- hpe3par:cpg - Defaults to the hpe3par_cpg setting in the cinder.conf file.
- hpe3par:snap_cpg - Defaults to the hpe3par_snap setting in the cinder.conf file. If hpe3par_snap is not set, it defaults to the hpe3par_cpg setting.
- hpe3par:provisioning - Defaults to thin provisioning, the valid values are thin, full, and dedup.
- hpe3par:persona - Defaults to the 2 - Generic-ALUA persona. The valid values are:

  - 1 - Generic
  - 2 - Generic-ALUA
  - 3 - Generic-legacy
  - 4 - HPUX-legacy
  - 5 - AIX-legacy
  - 6 - EGENERA
  - 7 - ONTAP-legacy
  - 8 - VMware
  - 9 - OpenVMS
  - 10 - HPUX
  - 11 - WindowsServer

- hpe3par:flash_cache - Defaults to false, the valid values are true and false.

QoS support for both HPE 3PAR drivers includes the ability to set the following capabilities in the OpenStack Block Storage API __cinder.api.contrib.qos_specs_manage qos specs__ extension module:

- minBWS
- maxBWS
- minIOPS
- maxIOPS
- latency
- priority

Note: The above QoS values are available till 10.4.x.
On 10.5.x, only two values are allowed (maxBWS and maxIOPS). Other QoS values are not available.

The qos keys above no longer require to be scoped but must be created and associated to a volume type. For information about how to set the key-value pairs and associate them with a volume type, run the following commands:

```
openstack help volume qos
```

The following keys require that the HPE 3PAR/Primera/Alletra 9k/ Alletra MP array has a Priority Optimization enabled.

- hpe3par:vvs

The virtual volume set name that has been predefined by the Administrator with quality of service (QoS) rules associated to it. If you specify extra_specs hpe3par:vvs, the qos_specs minIOPS, maxIOPS, minBWS, and maxBWS settings are ignored.

- minBWS

The QoS I/O issue bandwidth minimum goal in MBs. If not set, the I/O issue bandwidth rate has no minimum goal.

- maxBWS

The QoS I/O issue bandwidth rate limit in MBs. If not set, the I/O issue bandwidth rate has no limit.

- minIOPS

The QoS I/O issue count minimum goal. If not set, the I/O issue count has no minimum goal.

- maxIOPS

The QoS I/O issue count rate limit. If not set, the I/O issue count rate has no limit.

- latency

The latency goal in milliseconds.

- priority

The priority of the QoS rule over other rules. If not set, the priority is normal, valid values are low, normal and high.

Note

Since the Icehouse release, minIOPS and maxIOPS must be used together to set I/O limits. Similarly, minBWS and maxBWS must be used together. If only one is set the other will be set to the same value.

The following key requires that the HPE 3PAR/Primera/Alletra 9k/Alletra MP array has an Adaptive Flash Cache enabled.

- hpe3par:flash_cache - The flash-cache policy, which can be turned on and off by setting the value to __true__ or __false__.

- hpe3par:compression - The volume compression, which can be turned on and off by setting the value to __true__ or __false__.

Other restrictions and considerations for __hpe3par:compression__:

* For a compressed volume, minimum volume size needed is 16 GB; otherwise resulting volume will be created successfully but will not be a compressed volume.

* A full provisioned volume cannot be compressed, if a compression is enabled and provisioning type requested is full, the resulting volume defaults to thinly provisioned compressed volume.

* While creating volume on HPE Primera/Alletra 9k/Alletra MP storage system, only below two combinations are supported. If any other combination is used, then volume is not created.
  * thin volume: provisioning = __thin__ and compression = __false__
  * deco volume: provisioning = __dedup__ and compression = __true__


