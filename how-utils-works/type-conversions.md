# Type conversions

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
