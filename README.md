Module implementing the [Ethereum ABI](https://github.com/ethereum/wiki/wiki/Ethereum-Contract-ABI) in Javascript. Can be used with RPC libraries for communication or with ethereumjs-vm to implement a fully fledged simulator.

## Usage

#### Manual encoding and decoding

There are three methods of interest:
- ```methodID``` to create a function signature
- ```rawEncode``` to encode fields and
- ```rawDecode``` to decode fields

Example code:
```js
var abi = require('ethereumjs-abi')

// returns the encoded binary (as a Buffer) data to be sent
var encoded = abi.rawEncode([ "address" ], [ "0x0000000000000000000000000000000000000000" ])

// returns the decoded array of arguments
var decoded = abi.rawDecode([ "address" ], data)
```

#### Encoding and decoding aided by the JSON ABI definition

Supporting the JSON ABI definition:

```js
var  abi = [{"constant":false,"inputs":[{"name":"val","type":"int256"}],"name":"setVal","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":false,"inputs":[],"name":"getVal","outputs":[{"name":"","type":"int256"}],"payable":false,"stateMutability":"nonpayable","type":"function"}]

var encoded = ABI.encode(abi,"setVal",[1234]);

console.log(encoded.toString("hex"));

Output :
5362f8a200000000000000000000000000000000000000000000000000000000000004d2


var decoded = abi.decode(abi, 'setVal', '5362f8a200000000000000000000000000000000000000000000000000000000000004d2')

Output:
decoded[0].type ====> 'int256'
decoded[0].value ====> 1234

```

#### Simple encoding and decoding

```js
var abi = require('ethereumjs-abi')

// returns the encoded binary (as a Buffer) data to be sent
var encoded = abi.simpleEncode("balanceOf(address):(uint256)", "0x0000000000000000000000000000000000000000")

// returns the decoded array of arguments
var decoded = abi.simpleDecode("balanceOf(address):(uint256)", data)
```

#### Solidity *tightly packed* formats

This library also supports creating Solidity's tightly packed data constructs, which are used together with ```sha3```, ```sha256``` and ```ripemd160``` to create hashes.

Solidity code:
```js
contract HashTest {
  function testSha3() returns (bytes32) {
   address addr1 = 0x43989fb883ba8111221e89123897538475893837;
   address addr2 = 0;
   uint val = 10000;
   uint timestamp = 1448075779;

   return sha3(addr1, addr2, val, timestamp); // will return 0xc3ab5ca31a013757f26a88561f0ff5057a97dfcc33f43d6b479abc3ac2d1d595
 }
}
```

Creating the same hash using this library:
```js
var abi = require('ethereumjs-abi')
var BN = require('bn.js')

abi.soliditySHA3(
    [ "address", "address", "uint", "uint" ],
    [ new BN("43989fb883ba8111221e89123897538475893837", 16), 0, 10000, 1448075779 ]
).toString('hex')
```

For the same data structure:
* sha3 will return ```0xc3ab5ca31a013757f26a88561f0ff5057a97dfcc33f43d6b479abc3ac2d1d595```
* sha256 will return ```0x344d8cb0711672efbdfe991f35943847c1058e1ecf515ff63ad936b91fd16231```
* ripemd160 will return ```0x000000000000000000000000a398cc72490f72048efa52c4e92067e8499672e7``` (NOTE: it is 160bits, left padded to 256bits)

Note that ```ripemd160()``` in Solidity returns bytes20 and if you cast it to bytes32, it will be right padded with zeroes.

#### Using Serpent types

Serpent uses a different notation for the types, even though it will serialize to the same ABI.

We provide two helpers to convert between these notations:
* ```fromSerpent```: convert a Serpent notation to the ABI notation
* ```toSerpent```: the other way around

Example usage:
```js
abi.fromSerpent('s')    // [ 'bytes' ]
abi.fromSerpent('i')    // [ 'int256' ]
abi.fromSerpent('a')    // [ 'int256[]' ]
abi.fromSerpent('b8')   // [ 'bytes8' ]
abi.fromSerpent('b8i')  // [ 'bytes8', 'int256' ]

abi.toSerpent([ 'bytes' ])             // 's'
abi.toSerpent([ 'int256' ])            // 'i'
abi.toSerpent([ 'int256[]' ])          // 'a'
abi.toSerpent([ 'bytes8' ])            // 'b8'
abi.toSerpent([ 'bytes8', 'int256' ])  // 'b8i'
```

It is to be used in conjunction with ```rawEncode``` and ```rawDecode```:

```js
var encoded = abi.rawEncode(abi.fromSerpent("i"), [ "0x0000000000000000000000000000000000000000" ])

var decoded = abi.rawDecode([...abi.fromSerpent("i"), ...abi.fromSerpent("i")], data)
```

Note: Serpent uses arbitary binary fields. If you want to store strings it is preferable to ensure it is stored as UTF8. `new Buffer(<string>, 'utf8')` can be used to ensure it is properly encoded.


