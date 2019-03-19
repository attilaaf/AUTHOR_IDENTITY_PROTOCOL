# AUTHOR IDENTITY PROTOCOL
> A simple and flexible method to sign arbitrary OP_RETURN data with Bitcoin ECDSA signatures.

Authors: Attila Aros, Satchmo

Special thanks to Monkeylord and Unwriter for feedback and ideas.

Note: Use the [bitcoinfiles-sdk](https://github.com/BitcoinFiles/bitcoinfiles-sdk#sign-and-create-file) to build, sign, and verify document signatures.

# Intro

The design goals:

1. A simple protocol to sign arbitrary OP_RETURN data in a single transaction
2. Decouple the signing with an address from the funding source address (ie: does not require any on-chain transactions from the signing identity address)
3. Allow multiple signatures to be layered on top to provide multi-party contracts.

# Use Cases

- Prove ownership and authoring of any file
- Add multiple signatures to form agreements and contracts
- Decouple identity from funding addresses

The last point of being able to decouple identity from the funding source addresses means that we can now upload files and content and not have to expose our identity with an on-chain payment transaction.

An example is being able to upload a blog post and using Money Button to pay for the mining fees, yet never exposing your Identity key with an on-chain payment.


# Protocol

- The prefix for AUTHOR IDENTITY Protocol is `15PciHG22SNLQJXMoSUaWVi7WSqc7hCfva`

Here's an example of what **POST transactions** look like:

```
OP_RETURN
  19HxigV4QyBv3tHpQVcUEQyq1pzZVdoAut
  [Data]
  [Media Type]
  [Encoding]
  [Filename]
  |
  15PciHG22SNLQJXMoSUaWVi7WSqc7hCfva
  [Signing Algorithm]
  [Signing Address]
  [Signature]
  [Field Offset]
  [Field Count]
  [Field Index 0]
  [Field Index 1]
  ...
  [Field Index (Field Count - 1)]
```

An example with signing [B:// Bitcoin Data](https://github.com/unwriter/B]) is shown, however any arbitrary OP_RETURN content can be signed provided that the fields being signed are before the AUTHOR IDENTITY `15PciHG22SNLQJXMoSUaWVi7WSqc7hCfva` prefix.

We use the Bitcom convention to use the pipe '|' to indicate the protocol boundary.

Fields:

1. **Signing Algorithm:** ECDSA - This is the default Bitcoin signing algorithm built into bsv.js. UTF-8 encoding.
2. **Signing Address:** Bitcoin Address that is used to sign the content. UTF-8 encoding.
3. **Signature:** The signature of the signed content with the Signing Address. Base64 encoding.
4. **Field Offset:** An offset used indicate which field position to start looking for fields in the OP_RETURN. This offset is _negative_ relative to the AUTHOR IDENTITY prefix. Positive integer hex encoding.
5. **Field Count:** The total number of fields being signed. The specific field indexes being signed (relative to Field Offset) are listed next. Positive integer hex encoding.
6. **Field Index <index>:** The specific index (relative to Field Offset) that is covered by the Signature.  Non-negative integer hex encoding.

Example:

```
OP_RETURN
  19HxigV4QyBv3tHpQVcUEQyq1pzZVdoAut  // B Prefix
  { "message": "Hello world!" }       // Content
  applciation/json                    // Content Type
  UTF-8                               // Encoding
  0x00                                // File name (empty in this case with 0x00 to indicate null)
  |                                   // Pipe to seperate protocols
  15PciHG22SNLQJXMoSUaWVi7WSqc7hCfva, // AUTHOR IDENTITY prefix
  BITCOIN_ECDSA                       // Signing Algorithm
  1EXhSbGFiEAZCE5eeBvUxT6cBVHhrpPWXz, // Signing Address
  0x1b3ffcb62a3bce00c9b4d2d66196d123803e31fa88d0a276c125f3d2524858f4d16bf05479fb1f988b852fe407f39e680a1d6d954afa0051cc34b9d444ee6cb0af, // Signature
  6 // Negative offset from the left of the AUTHOR IDENTITY prefix
  6 // Field Count that follows (in this example we are signing everything before the AUTHOR IDENTITY prefix
  0 // OP_RETURN index (-6 + 0) = -6 (relative to AUTHOR IDENTITY prefix)
  1 // OP_RETURN index (-6 + 1) = -5 (relative to AUTHOR IDENTITY prefix)
  2 // OP_RETURN index (-6 + 2) = -4 (relative to AUTHOR IDENTITY prefix)
  3 // OP_RETURN index (-6 + 3) = -3 (relative to AUTHOR IDENTITY prefix)
  4 // OP_RETURN index (-6 + 4) = -2 (relative to AUTHOR IDENTITY prefix)
  5 // OP_RETURN index (-6 + 5) = -1 (relative to AUTHOR IDENTITY prefix)
];

```

In Hex form:
```

OP_RETURN
[
  '0x31394878696756345179427633744870515663554551797131707a5a56646f417574',
  '0x7b20226d657373616765223a202248656c6c6f20776f726c6421227d',
  '0x6170706c69636174696f6e2f6a736f6e',
  '0x7574662d38',
  '0x00',
  '0x7c',
  '0x313550636948473232534e4c514a584d6f5355615756693757537163376843667661',
  '0x424954434f494e5f4543445341',
  '0x31455868536247466945415a4345356565427655785436634256486872705057587a',
  '0x1b3ffcb62a3bce00c9b4d2d66196d123803e31fa88d0a276c125f3d2524858f4d16bf05479fb1f988b852fe407f39e680a1d6d954afa0051cc34b9d444ee6cb0af',
  '0x06',
  '0x06',
  '0x00',
  '0x01',
  '0x02',
  '0x03',
  '0x04',
  '0x05'
]

```

# Transaction Examples

*1 signature*:

File:

https://www.bitcoinfiles.org/bd5b707d4ab4caef96ff45296738c648e9d9db82ba0df2377eb95a8a6bf7e6a9?


Transaction:

https://whatsonchain.com/tx/bd5b707d4ab4caef96ff45296738c648e9d9db82ba0df2377eb95a8a6bf7e6a9?


*2 signatures*:

File:

https://www.bitcoinfiles.org/aed4adf1e77d2a913af14243beca11ff1c988c9a158db208a83de15f51467c81?


Transaction:

https://whatsonchain.com/tx/aed4adf1e77d2a913af14243beca11ff1c988c9a158db208a83de15f51467c81


# Usage and Library Examples

*Create and Sign a File*

https://github.com/BitcoinFiles/bitcoinfiles-sdk/blob/master/test/create.js#L128

*Build and Sign a File*

https://github.com/BitcoinFiles/bitcoinfiles-sdk/blob/master/test/build.js#L16

*Build and Sign a File with 2 Signatures (Contract)*

https://github.com/BitcoinFiles/bitcoinfiles-sdk/blob/master/test/build.js#L258

*Verify Signature for OP_RETURN fields*:

https://github.com/BitcoinFiles/bitcoinfiles-sdk/blob/master/test/build.js#L418

*Broadcast Signed File with Datapay*:

https://github.com/BitcoinFiles/bitcoinfiles-sdk/blob/master/test/build.js#L437

# Libraries

BitcoinFiles SDK has support for directly being able to sign B data files.
[bitcoinfiles-sdk](https://github.com/BitcoinFiles/bitcoinfiles-sdk#sign-and-create-file)
