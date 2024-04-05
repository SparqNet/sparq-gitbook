# Solidity ABI

SparqNet is a _native_ blockchain and has no EVM, but the vast majority of the smart contract ecossystem operates and depends on [Solidity](https://docs.soliditylang.org/en/latest) - not only the contracts themselves but also the data they share across each other.

<figure><img src="../.gitbook/assets/SolidityABI.png" alt=""><figcaption></figcaption></figure>

The **ABI** namespace, declared in `contract/abi.h`, contains a few classes with several Solidity ABI-related operations for managing and manipulating data in Solidity format.

This is only an overview, check the Doxygen docs for more details on how those classes work.

### Solidity types

The **Types** enum contains the supported Solidity data types. Each value has an intrinsic equivalency with both the Solidity data type and the native C++ data type that it represents:

|    Enum    |  Solidity  |                                                  C++                                                 |
| :--------: | :--------: | :--------------------------------------------------------------------------------------------------: |
|   uint256  |   uint256  |                                              uint256\_t                                              |
| uint256Arr | uint256\[] |                                       std::vector\<uint256\_t>                                       |
|   address  |   address  |        [Address](https://github.com/SparqNet/sparq-docs/blob/main/Sparq\_en-US/ch3/ch2/2-5.md)       |
| addressArr | address\[] | std::vector<[Address](https://github.com/SparqNet/sparq-docs/blob/main/Sparq\_en-US/ch3/ch2/2-5.md)> |
|    bool    |    bool    |                                                 bool                                                 |
|   boolArr  |   bool\[]  |                                          std::vector\<bool>                                          |
|    bytes   |    bytes   |                                              std::string                                             |
|  bytesArr  |  bytes\[]  |                                       std::vector\<std::string>                                      |
|   string   |   string   |                                              std::string                                             |
|  stringArr |  string\[] |                                       std::vector\<std::string>                                      |

Note that `bytes` and `string` on Solidity use the same type `std::string` on C++. This is because both Solidity `bytes` and `string` types are parsed the exact same way, and `std::string` is used extensively for both raw byte and literal strings.

### The Encoder class

The **Encoder** class is responsible for encoding and packing native C++ data types into Solidity ABI-formatted strings.

The class automatically does the encoding on the constructor, so all you have to do is invoke it with a list of the supported types above (in the C++ column), and optionally a string with the functor (the full function header with no spaces between arguments, e.g. `func(type1,type2)`).

Here's an example:

```
ABI::Encoder e(
  {1000000000000000000, Address(std::string("0x1a2b3c4d5e6f7e8d9c0b1a2b3c4d5e6f7e8d9c0b"), false)},
  "transfer(uint256,address)"
);
std::cout << e.getData() << std::endl;
$ b7760c8f0000000000000000000000000000000000000000000000000de0b6b3a76400000000000000000000000000001a2b3c4d5e6f7e8d9c0b1a2b3c4d5e6f7e8d9c0b
```

### The Decoder class

The **Decoder** class is responsible for decoding Solidity ABI-formatted strings back to native C++ data types.

The class automatically does the decoding on the constructor, so all you have to do is invoke it with the Solidity string and a list with the respective enum types, in order. Note that the Solidity string has to be in **raw bytes** format.

Here's an example:

```
ABI::Decoder d(
  {ABI::Types::uint256, ABI::Types::address},
  Hex(std::string("0000000000000000000000000000000000000000000000000de0b6b3a76400000000000000000000000000001a2b3c4d5e6f7e8d9c0b1a2b3c4d5e6f7e8d9c0b")).bytes()
);
uint256_t value = d.getData<uint256_t>(0);
Address to = d.getData<Address>(1);
std::cout << value << std::endl;
std::cout << to.hex().get() << std::endl;
$ 1000000000000000000
$ 1a2b3c4d5e6f7e8d9c0b1a2b3c4d5e6f7e8d9c0b
```
