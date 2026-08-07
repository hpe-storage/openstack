# Restrictions & Limitations

The HPE Alletra MP B10000 share driver has the following restrictions:

- Only the NFS protocol is supported; CIFS/SMB is not supported.
- Share networks are not supported (``driver_handles_share_servers`` must be
  ``False``).
- Share shrink is not currently supported.
- Snapshots are not supported.
- Creating a share from a snapshot is not supported.
- Share migration is not supported.
- Share replication is not supported.
- Share groups and consistency groups are not supported.
- Security services (LDAP, Active Directory, Kerberos) are not supported.
- Only IP-based access rules are supported for NFS shares.
