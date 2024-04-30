---
description: How the AppLayer ecosystem manages the blockchain state alongside an EVM.
---

# State management and VM instance creation

The VM itself is owned and instantiated by the `State` class, which reflects a crucial design decision: centralizing virtual machine resource management like this ensures that each contract execution context is cleanly managed and isolated. Whenever a new transaction or contract call needs to be executed, regardless of its nature (be it an EVM/C++ contract execution or a simple native transfer), the `State` class is responsible for instantiating a new `ContractHost` object with the relevant parameters required for execution:

```cpp
ContractHost(
  evmc_vm* vm,
  EventManager& eventManager,
  const Storage& storage,
  const evmc_tx_context& currentTxContext,
  std::unordered_map<Address, NonNullUniquePtr<Account>, SafeHash>& accounts,
  std::unordered_map<StorageKey, Hash, SafeHash>& vmStorage,
  const Hash& txHash,
  const uint64_t txIndex,
  const Hash& blockHash,
  int64_t& txGasLimit
);
```

Once an instance of `ContractHost` is created, it offers methods like `execute()` to run the contract, `simulate()` for simulating the transaction (useful for gas estimation), and `ethCallView()` for making calls to other contracts within a non-state-changing context.

`ContractHost` also extends the functionalities of `evmc::Host` by overriding several key functions that interface directly with the Ethereum Virtual Machine, which are obligatory for the VM to interact with the State:

```cpp
bool account_exists(const evmc::address& addr) const noexcept final;
evmc::bytes32 get_storage(const evmc::address& addr, const evmc::bytes32& key) const noexcept final;
evmc_storage_status set_storage(const evmc::address& addr, const evmc::bytes32& key, const evmc::bytes32& value) noexcept final;
evmc::uint256be get_balance(const evmc::address& addr) const noexcept final;
size_t get_code_size(const evmc::address& addr) const noexcept final;
evmc::bytes32 get_code_hash(const evmc::address& addr) const noexcept final;
size_t copy_code(const evmc::address& addr, size_t code_offset, uint8_t* buffer_data, size_t buffer_size) const noexcept final;
bool selfdestruct(const evmc::address& addr, const evmc::address& beneficiary) noexcept final;
evmc::Result call(const evmc_message& msg) noexcept final;
evmc_tx_context get_tx_context() const noexcept final;
evmc::bytes32 get_block_hash(int64_t number) const noexcept final;
void emit_log(const evmc::address& addr, const uint8_t* data, size_t data_size, const evmc::bytes32 topics[], size_t topics_count) noexcept final;
evmc_access_status access_account(const evmc::address& addr) noexcept final;
evmc_access_status access_storage(const evmc::address& addr, const evmc::bytes32& key) noexcept final;
evmc::bytes32 get_transient_storage(const evmc::address &addr, const evmc::bytes32 &key) const noexcept final;
void set_transient_storage(const evmc::address &addr, const evmc::bytes32 &key, const evmc::bytes32 &value) noexcept final;
```

These methods manage everything from account validation to logging, providing access to the state and storage, and handling calls between contracts. The `ContractHost` class encapsulates these functions, ensuring that each contract execution is isolated and secure.
