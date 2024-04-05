# Address manipulation

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
