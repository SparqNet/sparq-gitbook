---
description: The functions used on the Utils class on SparqNet
---

# How Utils works

This serves as a glossary of the functions in the Utils class and how to use them to make development on SparqNet easier.

* Debugging through log prints
* Type conversions
* StringContainer and its aliases
* Address manipulation

### Debugging through log prints

Due to the Subnet's behavior when it is initialized (`subnet->start()`), printing details in the terminal via `std::cout` doesn't work as it should. To properly debug the Subnet, you have to use the functions `Utils::logToFile()` and `Utils::LogPrint()`.

The function `Utils::logToFile()` logs to the Node's `log.txt` file and requires only the string that will be logged. The function `Utils::LogPrint()` logs to the Node's `debug.txt` file and requires the string that will be logged, the function name (you can use the `__func__ macro` here) and the module prefix from where the function is being called, that matches an item from the Log namespace (e.g. "`Log::Subnet = 'Subnet::'`").\


Example:

```cpp
Utils::logToFile("Logging");
// log.txt will say "Logging"

void testFunc() {
  Utils::LogPrint(Log::db, __func__, "Debugging");
  // debug.txt will say "DBService::testFunc: Debugging"
}
```

### Type conversions

At several points in SparqNet's code, raw bytes are used instead of hex strings, but both are treated as `std::string`, which can be confusing for whoever prints a "string" and gets incoherent values instead of a "`Hello World`".

If this happens, place the variable that has said data in the function `Utils::bytesToHex()` in case of a raw bytes string, or `Utils::hexToBytes()` in case of a hex string.&#x20;

Example:

```cpp
Hex data("0x1234567890abcdef");
std::string bytes = data.bytes();
Hex hex = Hex::fromBytes(bytes);
std::cout << data << std::endl;
std::cout << bytes << std::endl;
std::cout << hex << std::endl;
```

To convert integers to bytes and vice-versa, you should use the functions `Utils::uintXToBytes()` and `Utils::bytesToUintX()`, respectively. "X" is the integer size in bits - it can be 8, 16, 32, 64, 160 or 256.

Example:

```cpp
uint64_t timestampOri = 1234567890;
std::string timestampBytes = Utils::uint64ToBytes(timestampOri);
Hex timestampHex = Hex::fromBytes(timestampBytes);
uint64_t timestampNew = Utils::bytesToUint64(timestampHex);
std::cout << timestampOri << std::endl;
std::cout << timestampBytes << std::endl;
std::cout << timestampHex << std::endl;
std::cout << timestampNew << std::endl;
```

Alternatively, there are other helper functions:

* `Utils::uintToHex()` works with any `uint`, without having to know its size, e.g. `Utils::uintToHex(32)`.
* `Utils::hexToUint()` works the same way, but it only returns 256-bit integers.
* You can use the `HexTo` struct along with `boost::lexical_cast` to convert a hex string to other types:

`std::string hex = "0x37285422";`

`uint256_t bigInt = boost::lexical_cast<HexTo<uint256_t>>(hex);`

### StringContainer and its aliases

The `StringContainer` class abstracts a fixed-size string (e.g. `StringContainer<10>` is a string with exactly 10 characters). Several frequently repeated types in the code are actually aliases to `StringContainer`, or classes that inherit a `StringContainer` of a given size.

For a simpler outlook, refer to this simple guide:

* `PrivKey` is an alias to `StringContainer<32>`
* `UncompressedPubkey` is an alias to `StringContainer<65>`
* `CompressedPubkey` is an alias to `StringContainer<33>`
* `Hash` is a class that inherits `StringContainer<32>`
* `Signature` is a class that inherits `StringContainer<65>`

To get the content of any `StringContainer`, use the `data()` function:

`StringContainer<10> str = "HelloWorld";`

`std::cout << str.data() << std::endl;`

### Address manipulation

Some Utils functions exist to make address manipulation easier:

* `toLowercaseAddress()` converts the address to an all-lowercase format (e.g. "0xacbdef...").
* t`oUppercaseAddress()` converts the address to an all-uppercase format (e.g. "0XABCDEF...").
* `toChecksumAddress()` converts the address to a mixed-case format, also known as "checksum", according to the [EIP-55](https://eips.ethereum.org/EIPS/eip-55) specification (e.g. "0xaBcdEF...").
* `isAddress()` checks if a string is an address (size, format, and if the checksum is valid when in mixed-case).
* `checkAddressChecksum()` checks if the address checksum is correct.

Example:

```cpp
Address addOri = Address("0x1a2b3c4d5e6f7e8d9c0b1a2b3c4d5e6f7e8d9c0b", true);
Hex addUpp = addOri.toUppercaseAddress(addOri);
Hex addLow = addOri.toLowercaseAddress(addUpp);
Hex addChk = addOri.toChecksumAddress(addOri);
std::string << addOri << std::endl;
std::string << addUpp << std::endl;
std::string << addLow << std::endl;
std::string << addChk << std::endl;
std::string << Utils::isAddress(addChk.get()) << std::endl;
std::string << Utils::isChksum(addChk) << std::endl;
```

There's no "default" behavior here, but it's ideal to always have addresses in lower case and convert them to other types when required.

\
