# StringContainer class

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
