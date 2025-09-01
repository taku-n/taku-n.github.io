---
title: "Type Dictionary"
date: 2021-11-22
tags: ["Rust"]
---

# Type Dictionary


<a id="bytes-bytes"></a>
## bytes::Bytes

[struct bytes::Bytes](https://docs.rs/bytes/latest/bytes/struct.Bytes.html)  
[struct bytes::BytesMut](https://docs.rs/bytes/latest/bytes/struct.BytesMut.html)  

```
use bytes::BytesMut;

// Create an instance
let mut bytes_mut = BytesMut::new();
println!("{:?}", bytes_mut.len()); //=> 0

// Extend bytes_mut length from 0 to 3 and fill it with 0x00, 0x01, 0x02
bytes_mut.extend_from_slice(&[0x00, 0x01, 0x02]);
println!("{bytes_mut:?}"); //=> b"\0\x01\x02"
println!("{:?}", bytes_mut.len()); //=> 3

// copy_from_slice() to update the data
// I can use copy_from_slice() because the length of bytes_mut is the same as the length of the slice given
bytes_mut.copy_from_slice(&[0x03, 0x04, 0x05]);
println!("{bytes_mut:?}");         //=> b"\x03\x04\x05"
println!("{:?}", bytes_mut.len()); //=> 3

// Extend bytes_mut length again from 3 to 6 and fill it with 0x06, 0x07, 0x08
bytes_mut.extend_from_slice(&[0x06, 0x07, 0x08]);
println!("{bytes_mut:?}");         //=> b"\x03\x04\x05\x06\x07\x08"
println!("{:?}", bytes_mut.len()); //=> 6

// resize() shrinks the length of bytes_mut but doesnt change the data because the second argument is only used when extending
bytes_mut.resize(3, 0x00);
println!("{bytes_mut:?}");         //=> b"\x03\x04\x05"
println!("{:?}", bytes_mut.len()); //=> 3

// copy_from_slice() with a slice of another BytesMut instance
// Be careful that the length is the same
let mut another_bytes_mut = BytesMut::new();
another_bytes_mut.extend_from_slice(b"hello");
bytes_mut.copy_from_slice(&another_bytes_mut[1..4]);
println!("{bytes_mut:?}");         //=> b"ell"
println!("{:?}", bytes_mut.len()); //=> 3

// Convert from BytesMut to Bytes
let bytes = bytes_mut.freeze();
```


<a id="bytes-bytesmut"></a>
## bytes::BytesMut

[See bytes::Bytes](#bytes-bytes)
