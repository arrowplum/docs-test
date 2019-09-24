# Capacity Planning for Specific Data Type

All sizes are in bytes unless otherwise noted.

## Summary

|Type			|In Memory			| In Memory Indexes		| On Disk		| On Disk Metadata |
|---			|---				|---					|---			|--- 	  		   |
|integer/float	|0					|n/a					|8				|n/a			   |
|string	|string-len			|n/a					|string-len		|n/a			   |
|GeoJSON	|string-len + 12 bytes			|n/a					|string-len	+ 12 bytes	|n/a			   |
|list |10 + msgpack-array <sub id="ref1">[1](#ftnote1)</sub> |&lfloor;element-count / 128&rfloor; * 4 |msgpack-array |n/a |
|map			|msgpack-map		|msgpack-ext + 1		|msgpack-map	|4 <sub id="ref2">[2](#ftnote2)</sub> |

## Details

### List

##### Example

For a list of 3 integer elements {0, 1000, 255}:

`1 byte header for 3 elements`</br>
`+1 byte for integer 0`</br>
`+3 byte for integer 1000`</br>
`+2 byte for integer 255`</br>

    1 + 1 + 3 + 2 = 7 bytes. 


### Map

msgpack-ext = header + offset-index + value-index<br/>
index = element-count * size/element<br/>
element-count = number of elements in the map

|Type		| Indexes		|
|---		|---			|
|unordered	| None			|
|key ordered| offset	|
|key and value ordered | offset + value	|

#### Index Size/Element

|var<sub id="ref3">[3](#ftnote3)</sub>	| size/element |
|---			|---			   |
| < 2^8			| 1				   |
| < 2^16		| 2				   |
| < 2^24		| 3				   |
| >= 2^24		| 4				   |


<sup id="ftnote1">1</sup> [Msgpack specification](https://github.com/msgpack/msgpack/blob/master/spec.md) [↩](#ref1)</br>
<sup id="ftnote2">2</sup> No metadata if map is unordered. [↩](#ref2)</br>
<sup id="ftnote3">3</sup> *var* is msgpack-size for offset-index and element-count for value-index. [↩](#ref3)
