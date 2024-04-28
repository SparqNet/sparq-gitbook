---
description: How Applayer's BDK handles chains of contract execution.
---

# Contract call handling

Each time a transaction or contract execution alters any state variables—such as creating a new contract, updating a variable, or initiating transfers—those changes are *not* set directly at the state, but rather kept track of in a separate location. This is crucially important, as contracts can call other contracts, and nested calls can prolong themselves to a point you have dozens if not hundreds of variables with changed values, or several transactions that were made that must be reverted because the call failed.

This is done within `ContractHost` by means of `ContractStack`. The class maintains a record of the original states of these variables, and registers changes made to them during a contract call. This record-keeping is essential for enabling a complete restoration of the original state in the event of a transaction rollback, ensuring that any adverse changes can be undone, safeguarding the blockchain's consistency, reliability and integrity.

For example, a call with a variable that starts with the value "10", then five calls down the line it throws for some reason, but the original value was altered mid-way to "50". The role of `ContractStack` in this situation is to revert that variable and *all* the other variables that were changed along the way back to their original values *before* the call was made (so our variable would have to go back to "10"), ensuring the state is not left with any stale or wrong data because of the failed call. Likewise, if the call is successful, `ContractStack` then applies all changes in order at once, in an atomic fashion, ensuring the state now has up-to-date data from after the contract's execution.

## ContractStack class overview

Here's an overview of the `ContractStack` class definition and functionalities:

```c++
class ContractStack {
  private:
    std::unordered_map<Address, Bytes, SafeHash> code_;
    std::unordered_map<Address, uint256_t, SafeHash> balance_;
    std::unordered_map<Address, uint64_t, SafeHash> nonce_;
    std::unordered_map<StorageKey, Hash, SafeHash> storage_;
    std::vector<Event> events_;
    std::vector<Address> contracts_; // Contracts that have been created during the execution of the call, we need to revert them if the call reverts.
    std::vector<std::reference_wrapper<SafeBase>> usedVars_;

  public:
    ContractStack() = default;
    ~ContractStack() = default;

    inline void registerCode(const Address& addr, const Bytes& code)  {
      if (!this->code_.contains(addr)) {
        this->code_[addr] = code;
      }
    }

    inline void registerBalance(const Address& addr, const uint256_t& balance) {
      if (!this->balance_.contains(addr)) {
        this->balance_[addr] = balance;
      }
    }

    inline void registerNonce(const Address& addr, const uint64_t& nonce) {
      if (!this->nonce_.contains(addr)) {
        this->nonce_[addr] = nonce;
      }
    }

    inline void registerStorageChange(const StorageKey& key, const Hash& value) {
      if (!this->storage_.contains(key)) {
        this->storage_[key] = value;
      }
    }

    inline void registerEvent(Event&& event) {
      this->events_.emplace_back(std::move(event));
    }

    inline void registerContract(const Address& addr) {
      this->contracts_.push_back(addr);
    }

    inline void registerVariableUse(SafeBase& var) {
      this->usedVars_.emplace_back(var);
    }

    /// Getters
    inline const std::unordered_map<Address, Bytes, SafeHash>& getCode() const { return this->code_; }
    inline const std::unordered_map<Address, uint256_t, SafeHash>& getBalance() const { return this->balance_; }
    inline const std::unordered_map<Address, uint64_t, SafeHash>& getNonce() const { return this->nonce_; }
    inline const std::unordered_map<StorageKey, Hash, SafeHash>& getStorage() const { return this->storage_; }
    inline std::vector<Event>& getEvents() { return this->events_; }
    inline const std::vector<Address>& getContracts() const { return this->contracts_; }
    inline const std::vector<std::reference_wrapper<SafeBase>>& getUsedVars() const { return this->usedVars_; }
};
```

The existence of only *one* instance of `ContractStack` per `ContractHost`, and its integration within the RAII framework of `ContractHost`, guarantees that state values are meticulously commited or reverted upon the completion or rollback of transactions. This robust design prevents state spill-over between different contract executions, fortifying transaction isolation and integrity across the blockchain network - even in the dynamic and mutable landscape of blockchain transactions, the integrity and consistency of state changes are meticulously maintained, safeguarding against unintended consequences and errors during contract execution.
