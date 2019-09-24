---
title: Configuring Encryption at Rest
description: Configure Encryption at Rest
---

## Encryption at Rest

Aerospike’s encryption at rest feature encrypts database record data on storage devices using symmetric AES-128 encryption. 
By encrypting and decrypting each record in the database, even if storage is stolen, the data will be unreadable.

The encryption key is derived from a user-provided encryption key file. That key file contains a key, which is the
secret for encrypting data to and from the drive. If that key is lost, the data on the device is inaccessible.

Encryption at Rest is an Enterprise Edition feature requiring a [`feature-key-file`](/docs/reference/configuration#feature-key-file).
* For Aerospike Server version 4.1 & 4.2, a [`feature-key-file`](/docs/reference/configuration#feature-key-file) with  *encryption-at-rest* set to *true* is required.
* For Aerospike Server version 4.3 and newer, a [`feature-key-file`](/docs/reference/configuration#feature-key-file) with  *asdb-encryption-at-rest* set to *true* is required.

The default location of the feature-key-file is `/etc/aerospike/features.conf`.  
Contact Aerospike sales/support if you do not have a feature-key.

## Configuration

Encryption is configured per namespace, using two configuration directives, [`encryption-key-file`](/docs/reference/configuration/#encryption-key-file) and [`encryption`](/docs/reference/configuration/#encryption).

  * `encryption-key-file` provides an encryption key file for the namespace’s storage devices. This directive also controls, whether encryption is enabled or not. If it is specified, encryption is enabled. If it is not specified, then encryption is disabled.

  * `encryption` specifies the encryption algorithm to be used. Valid parameters are `aes-128` and `aes-256`. If this directive is not specified, it defaults to `aes-128`.

The configuration directives belong to a namespace's `storage-engine` section:

```
namespace test {
    ...
    storage-engine device {
        device /dev/sda1
        ...
        encryption-key-file key.dat
        encryption aes-128
    }
    ...
}
```

The content of the encryption key file needs to be unpredictable. It is suggested to generate the content of the file 
from a reliable source of randomness, for example:

```
head --bytes 256 /dev/urandom >key.dat
```

The keys may be different for different namespaces, or for different servers in the cluster. In all cases,
the file must be kept private. If the key file is stolen, then the device's data can be accessed. For this reason,
when running in this mode, the key file should only be kept on encrypted and secure operating system environments.

Enabling or disabling encryption for a namespace or changing the encryption algorithm requires that the namespace’s
storage devices or disk files are empty.

If you wish to enable this feature on a cluster which has been running without encryption, you may. It can
also be added to a cluster while the cluster is in production, with no outage or downtime, since the conversion
process can be done on a node by node basis.

This leads to the following conversion process, which enables or disables encryption across an Aerospike cluster 
for a single node in the cluster:

- Stop the Aerospike server process on the cluster node to be converted.

- If you need to maintain your configured replication count of data, wait for Aerospike replication to complete. If
your deployment durability allows a short period with fewer copies of the data, you may skip this step.

- Remove the unencrypted data on the device, using a program such as `dd` for raw devices, or simply `rm` for
file based storage.

- Either enable or disable encryption by adding or removing the appropriate files and directive(s) to `aerospike.conf`.

- Restart the Aerospike server process on the cluster node.

- Wait until data migration is complete.

Repeat this process for all nodes in the cluster.

Changing the encryption algorithm works analogously.

## Key rotation

In order to change the encryption key, you will follow a similar process. Just like enabling or disabling encryption, 
you will move through the cluster taking servers down, updating their key file, removing the data from the device(s),
and restarting servers.

The file specified by the encryption-key-file configuration directive is read exactly once at server startup, 
just after the configuration file, `aerospike.conf`, has been parsed. For increased security, the encryption key 
file may thus reside in a RAM-backed file system instead of a file system backed by physical storage media.

Once the Aerospike server process has been started, the encryption key file - or the complete RAM-backed file 
system - may be discarded. In this way, the encryption key file would only temporarily be available as a 
file on each cluster node.

Note that the same encryption key file must be provided again on the next start of the Aerospike server process. 
This means that the encryption key file still needs to be stored permanently somewhere, just not necessarily on the 
cluster nodes. The idea is to only make it temporarily available on the cluster nodes during Aerospike startup.

Keep your encryption key file safe! If you lose it, you lose access to your data.

## Technical Details

Encryption at rest involves two encryption keys:

The on-device data is encrypted using the data encryption key. This data encryption key is generated by the 
Aerospike server using a cryptographically strong random number generator: OpenSSL’s RAND_bytes() function.

The data encryption key is stored on the encrypted device. It is protected by a second encryption key, 
the user-provided encryption key. This second key is derived from the encryption key file provided by the user 
via the encryption-key-file configuration directive.

On Aerospike startup, the following happens:

Aerospike reads the encryption key file provided by the user and hashes it with SHA-256 (for AES-128) or
SHA-512 (for AES-256). This yields the user-provided encryption key.

Aerospike reads the data encryption key from each device and decrypts it using the user-provided 
encryption key. This yields the data encryption key for each device.

The data encryption key consists of two 128-bit (AES-128) or 256-bit (AES-256) encryption keys, which are used
to key AES in XTS mode as per the IEEE P1619 standard for disk encryption. This standard is also endorsed by NIST
in Special Publication (SP) 800-38E.

Disk encryption typically works on a per-sector basis. However, for better performance, Aerospike encrypts 
on a per-record basis. Records on disk are basically treated as variable-size disk sectors up to a size of 1 MiB. Beyond 1 MiB, records are split into 1-MiB chunks.

## Performance

The performance impact of encryption depends on the deployed hardware as well as the average size of the 
encrypted records. A definitive answer regarding the performance impact can thus only be obtained by 
benchmarking on the deployed hardware with real-life data.

However, use of Linux's OpenSSL library on Intel Xeon hardware uses on-CPU hardware acceleration. In one typical hardware configuration, we observed that:

Records smaller than 512 bytes resulted in 20% performance loss in terms of TPS.

Records with sizes of 1 KiB, 5 KiB, and 10 KiB didn’t lead to any measurable performance loss.



