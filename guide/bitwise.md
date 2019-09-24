---
title: Bitwise Operations
description: Overview of Aerospike's bitwise operations.
---

Aerospike supports a rich set of bitwise operations which can be used on the
[Bytes](/docs/guide/data-types.html#bytes) data type. These operations allow an
application to manipulate a large Bytes bin on the server without needing to
pull the entire blob to the client which can save client to server
bandwidth.

## Write Operations
### General Write Flags
All write operations support a subset of the following flags.
  - **create_only** - disallow updating an existing Bytes bin.
  - **update_only** - disallow creation of a new Bytes bin.
  - **no_fail** - if the operation should fail, continue as if it had succeeded.
  - **partial** - if the number of bytes from the offset to the end of the
    existing Bytes bin is less than the specified number of bytes, then only
    apply the operations from the offset to the end.
  
### Resize
Specify the size of the Bytes bin to be **n_bytes**. This operation may (by
default) create a new Bytes bin or extend or trim an existing Byte bin to the
specified size of **n_bytes**. By default the **resize** operation will extend
or trim from the end of the Bytes bin.

Arguments:
  - **n_bytes** - size of the resulting bin.
    
Flags (optional): **create_only**, **update_only**, **no_fail**.
    
Operation Flags (optional):
  - **from_front** - extend or trim the Bytes bin from the beginning instead of
    the end.
  - **shrink_only** - disallow extending existing objects.
  - **grow_only** - disallow trimming existing objects.
    
### Insert
Inserts bytes at the specified **byte_offset** with the contents of **buffer**.

Arguments:
  - **byte_offset** - offset to location of insertion.
  - **buffer** - bytes to be inserted.
    
Flags (optional): **create_only**, **update_only**, **no_fail**.

### Remove
Remove **n_bytes** bytes beginning at the specified **byte_offset**.

Arguments:
  - **byte_offset** - offset to location of removal.
  - **n_bytes** - number of bytes to remove.
    
Flags (optional): **no_fail**, **partial**.

### Set
Overwrites **n_bits** bits at the specified **offset** (in bits) with the first
**n_bits** of **buffer**.

Arguments:
  - **offset** - offset (in bits) to the first bit to overwrite.
  - **n_bits** - number of bits to overwrite.
  - **buffer** - buffer containing at least **n_bits** bits to be written. Bits
    are taken in order from the beginning of the buffer.
    
Flags (optional): **no_fail**, **partial**.

### Or
Bitwise OR **n_bits** of the **buffer** with the leading **n_bits** of the Bytes
bin starting from the specified **offset** (in bits).

Arguments:
  - **offset** - offset (in bits) to the first bit to apply operation.
  - **n_bits** - number of bits to apply operation to.
  - **buffer** - buffer containing at least **n_bits** bits.
    
Flags (optional): **no_fail**, **partial**.
    
### Xor
Bitwise XOR **n_bits** of the **buffer** with the leading **n_bits** of the
Bytes bin starting from the specified **offset** (in bits).

Arguments:
  - **offset** - offset (in bits) to the first bit to apply operation.
  - **n_bits** - number of bits to apply operation to.
  - **buffer** - buffer containing at least **n_bits** bits.
    
Flags (optional): **no_fail**, **partial**.

### And
Bitwise AND **n_bits** of the **buffer** with the leading **n_bits** of the
Bytes bin starting from the specified **offset** (in bits).

Arguments:
  - **offset** - offset (in bits) to the first bit to apply operation.
  - **n_bits** - number of bits to apply operation to.
  - **buffer** - buffer containing at least **n_bits** bits.
    
Flags (optional): **no_fail**, **partial**.
    
### Not
Bitwise NOT **n_bits** of the Bytes bin starting from the specified **offset**
(in bits).

Arguments:
  - **offset** - offset (in bits) to the first bit to apply operation.
  - **n_bits** - number of bits to apply operation to.
    
Flags (optional): **no_fail**, **partial**.
    
### LShift
Bitwise shift **n_bits** bits of the Bytes bin **n_bits** to the left starting
at the specified **offset** (in bits).

Arguments:
  - **offset** - offset (in bits) to the first bit to apply operation.
  - **n_bits** - number of bits to apply operation to.
  - **n_shift** - number of bits to shift.
    
Flags (optional): **no_fail**, **partial**.
    
### RShift
Bitwise shift **n_bits** bits of the Bytes bin **n_bits** to the right starting
at the specified **offset** (in bits).

Arguments:
  - **offset** - offset (in bits) to the first bit to apply operation.
  - **n_bits** - number of bits to apply operation to.
  - **n_shift** - number of bits to shift.
    
Flags (optional): **no_fail**, **partial**.
    
### Add
Treat the **n_bits** bits beginning at **offset** in the Bytes bin as an
**n_bits** bit integer and add the integer **uint64** to it - the integer
**uint64** will be converted to an **n_bits** bit integer. By default, fail if
the result overflows or underflows. Integers in the Bytes bin are stored and
interpreted as big-endian integers.

Arguments:
  - **offset** - offset (in bits) to the first bit to apply operation.
  - **n_bits** - size of the integer in bits (maximum of 64 bits).
  - **uint64** - value (unsigned integer) to be added.
    
Flags (optional): **no_fail**.

Operation Flags (optional):
  - **saturate** - instead of failing, if the result overflows, set to maximum
    **n_bit** value, if the result underflows set to minimum **n_bit** value.
    Maximum and minimum values are different for signed integers.
  - **wrap** - instead of failing, if the result overflows or overflows, wrap.
  - **signed** - treat the value at offset as a signed **n_bit** bit integer.
    
### Subtract
Treat the **n_bits** bits beginning at **offset** in the Bytes bin as an
**n_bits** bit integer and subtract the integer **uint64** from it - the integer
**uint64** will be converted to an **n_bits** bit integer. By default, fail if
the result overflows or underflows. Integers in the Bytes bin are stored and
interpreted as big-endian integers.

Arguments:
  - **offset** - offset (in bits) to the first bit to apply operation.
  - **n_bits** - size of the integer in bits (maximum of 64 bits).
  - **uint64** - value (unsigned integer) to be subtracted.
    
Flags (optional): **no_fail**.

Operation Flags (optional):
  - **saturate** - instead of failing, if the result overflows, set to maximum
    **n_bit** value, if the result underflows set to minimum **n_bit** value.
    Maximum and minimum values are different for signed integers.
  - **wrap** - instead of failing, if the result overflows or overflows, wrap.
  - **signed** - treat the value at offset as a signed **n_bit** bit integer.

### Set-Integer
Overwrite **n_bits** bits at offset **offset** with **uint64** converted to an
**n_bits** bit big_endian integer.

Arguments:
  - **offset** - offset (in bits) to the first bit to apply operation.
  - **n_bits** - size of the integer in bits (maximum of 64 bits).
  - **uint64** - value to store as an **n_bit** bit integer.
    
Flags (optional):  **no_fail**.

## Read Operations
### Get
Retrieve **n_bits** bits beginning at offset **offset**. If **n_bits** is not a
multiple of 8 then there will be **n_bits** `modulo 8` zeroed bits padding the
end. 

Arguments:
  - **offset** - offset (in bits) to the first bit to be retrieved.
  - **n_bits** - number of bits to retrieve.
    
### Count
Count the number of bits set to 1 in the **n_bits** beginning at **offset**.

Arguments:
  - **offset** - offset (in bits) to the first bit to be checked.
  - **n_bits** - number of bits to check.
    
### LScan
Return the position relative to the **offset** of the first bit set to **value**
searching from **offset** plus **n_bits** to **offset**. If the **value** isn't
found, return -1.

Arguments:
  - **offset** - offset (in bits) to the first (leftmost) bit to be scanned.
  - **n_bits** - number of bits from the **offset** to scan.
  - **value** - may be either 1 or 0, value to scan for.
    
### RScan
Return the position relative to the **offset** of the first bit set to **value**
searching from **offset** to **offset** plus **n_bits**. If the **value** isn't
found, return -1.

Arguments:
  - **offset** - offset (in bits) to the first (leftmost) bit to be scanned.
  - **n_bits** - number of bits from the **offset** to scan.
  - **value** - may be either 1 or 0, value to scan for.
    
### Get-Integer
Retrieve the **n_bits** bit big-endian integer beginning at offset **offset**
as a 64 bit integer.

Arguments:
  - **offset** - offset (in bits) to the first bit to retrieve.
  - **n_bits** - number of bits to retrieve.

Operation Flags (optional):
  - **signed** - treat the value at offset as a signed **n_bit** bit integer.
