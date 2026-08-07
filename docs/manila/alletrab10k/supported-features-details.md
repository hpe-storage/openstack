# Details on Supported Features

## Managing Share Access

A share must have access rules configured before it can be accessed by
clients. IP-based access rules are required for NFS shares.

Note

When no Manila access rules are configured, the driver blocks all IP addresses
by setting a default access rule of 0.0.0.0 with read-only and root_squash
permissions on the backend Alletra B10000 array. You must explicitly create
access rules to allow client access.


## Extending Shares

The driver supports extending shares to increase their size.

Note

The share size shown in Manila includes filesystem metadata and other
overhead. Client-usable space will be less than the displayed share size.

## Managing Existing Shares

The driver supports bringing existing shares on the HPE Alletra array into
Manila management using the manage operation.

### Prerequisites for the Manage Operation

Before managing an existing share, ensure the following requirements are met:

__Share type compatibility__: The backend share ``reduce`` value must match
   the ``hpe_alletra_b10000:reduce`` value from the share type. If they don't
   match, the manage operation will fail. Associate the correct share type.
   The default reduce value (if the share type doesn't have this key) is
   ``true``. Similarly, if the share type uses ``compression`` and ``dedupe``
   parameters instead of ``reduce``, those values must also match the backend
   share's compression and deduplication settings.

__No existing access rules__: The backend share must have either an empty
   access rules list or only the default 0.0.0.0 access rule with read-only
   and root_squash. If other access rules exist, clear them from the backend
   share before managing:

   ```
   $ setsharesetting -remove <ipaddr_list> <sharesetting_name>
   ```

__Filesystem size alignment__: The filesystem size of the backend fileshare
   must be a multiple of 1024 MiB (1 GiB). If not, the manage operation will
   fail. Manila logs will indicate how much MiB to expand the backend
   filesystem.

   To expand size by a specific amount on the alletra array cli (e.g., 500 MiB):

   ```
   $ setfilesystem -size 500 <filesystem_name>
   ```

## Ensure Shares Operation

The driver supports the ``ensure_shares`` operation, which validates that
shares exist on the backend and updates their status in Manila. Shares found
on the backend are updated with the latest export locations. Shares not found
on the backend are marked with the ``error`` state.

The ensure_shares operation for the driver is executed only in case of service
restarts after configuration changes in ``/etc/manila/manila.conf``.

If the backend fileshare export path changes due to a file port IP change or
other reasons, the administrator must manually trigger the ensure shares
command in OpenStack to update the latest export paths.
