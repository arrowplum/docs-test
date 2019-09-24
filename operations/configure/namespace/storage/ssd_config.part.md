```bash
namespace <namespace-name> {
	memory-size <SIZE>G         # Maximum memory allocation for primary
								# and secondary indexes.
	storage-engine device {     # Configure the storage-engine to use persistence
		device /dev/<device>    # raw device. Maximum size is 2 TiB
		# device /dev/<device>  # (optional) another raw device.
		write-block-size 128K   # adjust block size to make it efficient for SSDs.
	}
}
```
