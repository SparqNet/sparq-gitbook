---
description:  How contract calls happen from the EVM side in AppLayer.
---

# EVM to other contract calls

For calls from the EVM to another contract, the `ContractHost::call()` function plays a crucial role. It is tasked with creating and handling calls to other contracts, encapsulating the complexity of contract interaction within a simple interface:

```c++
evmc::Result call(const evmc_message& msg) noexcept final;
```

This function is designed to handle both C++ and EVM contract calls, as shown below:

```c++
evmc::Result ContractHost::call(const evmc_message& msg) noexcept {
  Address recipient(msg.recipient);
  auto &recipientAccount = *accounts_[recipient]; // We need to take a reference to the account, not a reference to the pointer.
  this->leftoverGas_ = msg.gas;
  /// evmc::Result constructor is: _status_code + _gas_left + _output_data + _output_size
  if (recipientAccount.contractType == CPP) {
    // Uh, we are a CPP contract, we need to call the contract evmEthCall function and put the result into a evmc::Result
    try {
      this->deduceGas(1000); // CPP contract call is 1000 gas
      auto& contract = contracts_[recipient];
      if (contract == nullptr) {
        throw DynamicException("ContractHost call: contract not found");
      }
      this->setContractVars(contract.get(), Address(msg.sender), Utils::evmcUint256ToUint256(msg.value));
      Bytes ret = contract->evmEthCall(msg, this);
      return evmc::Result(EVMC_SUCCESS, this->leftoverGas_, 0, ret.data(), ret.size());
    } catch (std::exception& e) {
      this->evmcThrows_.emplace_back(e.what());
      this->evmcThrow_ = true;
      return evmc::Result(EVMC_PRECOMPILE_FAILURE, this->leftoverGas_, 0, nullptr, 0);
    }
  }
  evmc::Result result (evmc_execute(this->vm_, &this->get_interface(), this->to_context(),
           evmc_revision::EVMC_LATEST_STABLE_REVISION, &msg,
           recipientAccount.code.data(), recipientAccount.code.size()));
  this->leftoverGas_ = result.gas_left; // gas_left is not linked with leftoverGas_, we need to link it.
  this->deduceGas(5000); // EVM contract call is 5000 gas
  result.gas_left = this->leftoverGas_; // We need to set the gas left to the leftoverGas_
  return result;
}
```
