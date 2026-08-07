# Share Types & Extra Specs

Share type support for the HPE Alletra MP B10000 driver includes the ability to set the following capabilities in the OpenStack Shared File Systems API share type extra specs:

- hpe_alletra_b10000:reduce
- hpe_alletra_b10000:squash_option
- compression
- dedupe
- thin_provisioning

To work with the default filter scheduler, the key values are case sensitive and scoped with __hpe_alletra_b10000:__

Manila requires that the share type includes the ``driver_handles_share_servers`` extra spec set to ``False``. To create a share type for the HPE Alletra MP B10000 backend:

```
$ openstack share type create alletra_nfs False
$ openstack share type set alletra_nfs --extra-specs share_backend_name=hpealletra1
```

If share types are not used or a particular key is not set for a share type, the following defaults are used:

- hpe_alletra_b10000:reduce - Defaults to ``true``.
- hpe_alletra_b10000:squash_option - Defaults to ``root_squash``.
- thin_provisioning - Defaults to ``true``.
- compression/dedupe - Default to ``true``. (Same as reduce)

Note

Modifying share type extra specs after shares have been created is not
recommended, as it will cause inconsistency between the share type definition
and the actual backend share properties. Backend share characteristics like
reduce, compression, and dedupe cannot be changed after creation.

- hpe_alletra_b10000:reduce

  Controls the ``compression`` and ``dedupe`` capabilities of the share. The valid values are ``true`` and ``false``.

  - If reduce = true: compression = true, dedupe = true
  - If reduce = false: compression = false, dedupe = false

  The reduce setting is applied at share creation time and cannot be changed for existing shares. Alternatively, you can use the ``compression`` and ``dedupe`` keys directly instead of ``reduce``, but you cannot specify both in the same share type.

  ```
  $ openstack share type set alletra_nfs --extra-specs hpe_alletra_b10000:reduce=true
  ```

- hpe_alletra_b10000:squash_option

  The NFS squash option applied to all access rules on the backend. The valid values are ``root_squash``, ``all_squash``, and ``no_root_squash``. If the share type is modified to change the squash option, the next share access rule update will use the new value.

  ```
  $ openstack share type set alletra_nfs --extra-specs hpe_alletra_b10000:squash_option=root_squash
  ```

- compression

  Controls whether compression is enabled on the share. The valid values are ``true`` and ``false``. When specifying ``compression``, you must also specify ``dedupe`` with the same value. You cannot use ``compression`` together with ``hpe_alletra_b10000:reduce``.

  ```
  $ openstack share type set alletra_nfs --extra-specs compression=true dedupe=true
  ```

- dedupe

  Controls whether data deduplication is enabled on the share. The valid values are ``true`` and ``false``. When specifying ``dedupe``, you must also specify ``compression`` with the same value. You cannot use ``dedupe`` together with ``hpe_alletra_b10000:reduce``.

  ```
  $ openstack share type set alletra_nfs --extra-specs dedupe=true compression=true
  ```

- thin_provisioning

  Controls whether thin provisioning is enabled on the share. This extra spec must be set to ``true`` or not specified at all. Setting it to ``false`` is not supported by this driver.

  ```
  $ openstack share type set alletra_nfs --extra-specs thin_provisioning=true
  ```
