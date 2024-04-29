<<<<<<< Updated upstream:how-contracts-work/safevariables-and-commit-revert-logic.md
---
description: How we simulate Solidity commit/revert logic.
---

# SafeVariables and commit/revert logic

In C++, when you call a function that changes a variable and then throw an exception later, the changed variable is _not_ reverted automatically. Consider the following example:

```cpp
MyClass::updateValueAndThrow(const uint64_t key, const uint64_t value) {
  // Suppose the original value of myMap[key] is 5
  this->myMap[key] = value; // e.g. 10
  throw std::runtime_error("Error");
  // myMap[key] is now 10, but it shouldn't be because it threw
}
```

Even if you use a try/catch block, the value of `myMap[key]` will be permanently changed and won't roll back on its own, because that's how C++ works - it doesn't go out of its way to do things you didn't specified it to do. You would have to manually roll this logic by storing the old value in a temporary variable before changing it, and implementing specific logic to assign it back again in case of an error. The previous example could theoretically be coded like this:

```cpp
MyClass::updateValueAndThrow(const uint64_t key, const uint64_t value) {
  const uint64_t oldValue = this->myMap[key]; // Store the old value of myMap[key] first
  try {
    this->myMap[key] = value; // myMap[key] is now 10
    throw std::runtime_error("Error"); // Something went wrong
  } catch (std::exception& e) {
    this->myMap[key] = oldValue; // Assign the original value back, myMap[key] is now 5 again
  }
}
```

For Protocol Contracts, that's pretty much the gist of it. Dynamic Contracts, however, automatically provide said functionality with the use of special types called **SafeVariables**, declared inside the `src/contract/variables` folder. Each Dynamic Contract includes a vector of references to SafeVariables, which is used to register used variables within a specific function call.

All SafeVariables inherit from the `SafeBase` class, which adheres to the following rules:

* Has two internal variables: one for the original value and another that is a pointer to a temporary value
* Must override the `check()`, `commit()`, and `revert()` functions
  * `check()` should verify if the temporary value is `nullptr`; if it is, it should set the temporary value to the original value
  * `commit()` should copy the temporary value to the original value
  * `revert()` should set the temporary value to `nullptr`
* Must be declared as _private_ within contracts and initialized with `this` as the parameter - this enables the SafeVariable to mark itself as used in the `ContractManager`, allowing proper reversion of changes made to the contract
* Must call `check()` and return the temporary value (or reference to the temporary value)
* If non-`const`, must call `markAsUsed()` when accessed, allowing the contract to properly register the variable as used within the function call, and properly revert or commit changes made to the contract accordingly
* When initialized with values in a contract's constructor, must call `commit()` followed by `enableRegister()` so the commit/revert functionality can actually work

#### Types of SafeVariables

We provide the following SafeVariable types, fully functional and ready for use in contracts:

* `SafeAddress` - abstracts a 20-byte address
* `SafeArray` - abstracts an array
* `SafeBool` - abstracts a boolean
* `SafeInt` and `SafeUint` - abstract a range of signed/unsigned integers (see Solidity ABI)
* `SafeString` - abstracts a string (either literal or raw bytes)
* `SafeTuple` - abstracts a tuple/struct of any type
* `SafeUnorderedMap` - abstracts a mapping
* `SafeVector` - abstracts a vector

#### Caveats with safe containers

Containers have some exceptions to these rules. Copying the entire container would be prohibitively expensive, so only accessed values are copied to the temporary container. This means they do not behave like regular containers, requiring developers to exercise caution when using iterators or looping through them.

Our `SafeUnorderedMap` variable allows limited looping through the container. This limitation is due to the inability to access both the original and temporary containers simultaneously; you can only access one at a time. It is recommended to loop through a container within a `const` function, as this will not modify the temporary container.

`SafeUnorderedMap` includes the following functions for looping through both containers:

| Function   | Description                                                          | Return type     |
| ---------- | -------------------------------------------------------------------- | --------------- |
| cbegin()   | Returns a const iterator to the beginning of the original container  | const\_iterator |
| cend()     | Returns a const iterator to the end of the original container        | const\_iterator |
| begin()    | Returns a const iterator to the beginning of the temporary container | iterator        |
| end()      | Returns a const iterator to the end of the temporary container       | iterator        |
| empty()    | Returns true both the original and temporary container is empty      | bool            |
| size()     | Returns the size of the original container                           | size\_type      |
| tempSize() | Returns the size of the temporary container                          | size\_type      |

Keep in mind that the temporary and original containers are not the same, so duplicates within `size()` and `tempSize()` are possible.
=======
---
description: How we simulate Solidity commit/revert logic in precompiled contracts.
---

# SafeVariables and commit/revert logic

In C++, when you call a function that changes a variable and then throw an exception later, the changed variable is *not* reverted automatically. Consider the following example:

```cpp
MyClass::updateValueAndThrow(const uint64_t key, const uint64_t value) {
  // Suppose the original value of myMap[key] is 5
  this->myMap[key] = value; // e.g. 10
  throw std::runtime_error("Error");
  // myMap[key] is now 10, but it shouldn't be because it threw
}
```

Even if you use a try/catch block, the value of `myMap[key]` will be permanently changed and won't roll back on its own, because that's how C++ works - it doesn't go out of its way to do things you didn't explicitly tell it to do. You would have to manually roll this logic by storing the old value in a temporary variable before changing it, and implementing specific logic to assign it back again in case of an error. The previous example could theoretically be coded like this:

```cpp
MyClass::updateValueAndThrow(const uint64_t key, const uint64_t value) {
  const uint64_t oldValue = this->myMap[key]; // Store the old value of myMap[key] first
  try {
    this->myMap[key] = value; // myMap[key] is now 10
    throw std::runtime_error("Error"); // Something went wrong
  } catch (std::exception& e) {
    this->myMap[key] = oldValue; // Assign the original value back, myMap[key] is now 5 again
  }
}
```

For Protocol Contracts, that's most of the required context. Dynamic Contracts, however, automatically provide their functionality with the use of special types called **SafeVariables**, declared inside the `src/contract/variables` folder. Each Dynamic Contract includes a vector of references to SafeVariables, which is used to register used variables within a specific function call.

All SafeVariables inherit from the `SafeBase` class, which adhere to the following rules:

* Have two internal variables: one for the original value and another that is a pointer to a temporary value
* Must override the `check()`, `commit()`, and `revert()` functions
  * `check()` should verify if the temporary value is `nullptr`; if it is, it should set the temporary value to the original value
  * `commit()` should copy the temporary value to the original value
  * `revert()` should set the temporary value to `nullptr`
* Must be declared as *private* within contracts and initialized with `this` as the parameter - this enables the SafeVariable to mark itself as used in the `ContractManager`, allowing proper reversion of changes made to the contract
* Must call `check()` and return the temporary value (or reference to the temporary value)
* If non-`const`, must call `markAsUsed()` when accessed, allowing the contract to properly register the variable as used within the function call, and properly revert or commit changes made to the contract accordingly
* When initialized with values in a contract's constructor, must call `commit()` followed by `enableRegister()` so the commit/revert functionality can actually work

## Types of SafeVariables

We provide the following SafeVariable types, fully functional and ready for use in contracts:

* `SafeAddress` - abstracts a 20-byte address
* `SafeArray` - abstracts an array
* `SafeBool` - abstracts a boolean
* `SafeInt` and `SafeUint` - abstract a range of signed/unsigned integers (see Solidity ABI)
* `SafeString` - abstracts a string (either literal or raw bytes)
* `SafeTuple` - abstracts a tuple/struct of any type
* `SafeUnorderedMap` - abstracts a mapping
* `SafeVector` - abstracts a vector

## Caveats with safe containers

Containers have some exceptions to these rules. Copying the entire container would be prohibitively expensive, so only accessed values are copied to the temporary container. This means they do not behave like regular containers, requiring developers to exercise caution when using iterators or looping through them.

Our `SafeUnorderedMap` variable allows limited looping through the container. This limitation is due to the inability to access both the original and temporary containers simultaneously; you can only access one at a time. It is recommended to loop through a container within a `const` function, as this will not modify the temporary container.

`SafeUnorderedMap` includes the following functions for looping through both containers:

| Function   | Description                                                          | Return type     |
| ---------- | -------------------------------------------------------------------- | --------------- |
| cbegin()   | Returns a const iterator to the beginning of the original container  | const\_iterator |
| cend()     | Returns a const iterator to the end of the original container        | const\_iterator |
| begin()    | Returns a const iterator to the beginning of the temporary container | iterator        |
| end()      | Returns a const iterator to the end of the temporary container       | iterator        |
| empty()    | Returns true both the original and temporary container is empty      | bool            |
| size()     | Returns the size of the original container                           | size\_type      |
| tempSize() | Returns the size of the temporary container                          | size\_type      |

Keep in mind that the temporary and original containers are not the same, so duplicates within `size()` and `tempSize()` are possible.
>>>>>>> Stashed changes:precompiled-contracts/safevariables-and-commit-revert-logic.md
