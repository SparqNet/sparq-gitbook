---
description: How AppLayer achieves a seamless integration between C++ and EVM contracts.
---

# Seamless C++/EVM integration

Achieving seamless integration between C++ and EVM contracts revolves around the uniformity in the encoding and decoding of arguments. By standardizing these processes, we ensure that calls between different contract types are handled efficiently without the need for separate mechanisms for each.

## The evmc\_message struct

We do this by using the `evmc_message` struct, aligning the call structures between C++ and EVM environments. This uniformity simplifies the interaction framework and reduces the potential for errors and data mismanagement:

```cpp
struct evmc_message {
  enum evmc_call_kind kind; // The kind of the call.
  uint32_t flags;
  int32_t depth;
  int64_t gas;
  evmc_address recipient;
  evmc_address sender;
  const uint8_t* input_data;
  size_t input_size;
  evmc_uint256be value;
  evmc_bytes32 create2_salt;
  evmc_address code_address;
};
```

## Determining contract types and executing calls

`ContractHost` plays a critical role in distinguishing whether a contract is implemented in C++ or EVM and executing it accordingly. Below is an example illustrating how C++ contracts can invoke functions in other contracts, whether they are C++ or EVM:

<pre class="language-cpp"><code class="lang-cpp">template &#x3C;typename R, typename C, typename... Args>
requires (!std::is_same&#x3C;R, void>::value)
R callContractFunction(
  BaseContract* caller, const Address&#x26; targetAddr,
  const uint256_t&#x26; value,
  R(C::*func)(const Args&#x26;...), const Args&#x26;... args
) {
  // 1000 Gas Limit for every C++ contract call!
  auto&#x26; recipientAcc = *this->accounts_[targetAddr];
  if (!recipientAcc.isContract()) {
    throw DynamicException(std::string(__func__) + ": Contract does not exist - Type: "
      + Utils::getRealTypeName&#x3C;C>() + " at address: " + targetAddr.hex().get()
    );
  }
  if (value) {
    this->sendTokens(caller, targetAddr, value);
  }
  NestedCallSafeGuard guard(caller, caller->caller_, caller->value_);
  switch (recipientAcc.contractType) {
    case ContractType::EVM : {
      this->deduceGas(10000);
      evmc_message msg;
      msg.kind = EVMC_CALL;
      msg.flags = 0;
      msg.depth = 1;
      msg.gas = this->leftoverGas_;
      msg.recipient = targetAddr.toEvmcAddress();
      msg.sender = caller->getContractAddress().toEvmcAddress();
      auto functionName = ContractReflectionInterface::getFunctionName(func);
      if (functionName.empty()) {
        throw DynamicException("ContractHost::callContractFunction: EVM contract function name is empty (contract not registered?)");
      }
      auto functor = ABI::FunctorEncoder::encode&#x3C;Args...>(functionName);
<strong>      Bytes fullData;
</strong>      Utils::appendBytes(fullData, Utils::uint32ToBytes(functor.value));
      Utils::appendBytes(fullData, ABI::Encoder::encodeData&#x3C;Args...>(args...));
      msg.input_data = fullData.data();
      msg.input_size = fullData.size();
      msg.value = Utils::uint256ToEvmcUint256(value);
      msg.create2_salt = {};
      msg.code_address = targetAddr.toEvmcAddress();
      evmc::Result result (evmc_execute(this->vm_, &#x26;this->get_interface(), this->to_context(),
      evmc_revision::EVMC_LATEST_STABLE_REVISION, &#x26;msg, recipientAcc.code.data(), recipientAcc.code.size()));
      this->leftoverGas_ = result.gas_left;
      if (result.status_code) {
        auto hexResult = Hex::fromBytes(BytesArrView(result.output_data, result.output_data + result.output_size));
        throw DynamicException("ContractHost::callContractFunction: EVMC call failed - Type: "
          + Utils::getRealTypeName&#x3C;C>() + " at address: " + targetAddr.hex().get() + " - Result: " + hexResult.get()
        );
      }
      return std::get&#x3C;0>(ABI::Decoder::decodeData&#x3C;R>(BytesArrView(result.output_data, result.output_data + result.output_size)));
    } break;
    case ContractType::CPP : {
      this->deduceGas(1000);
      C* contract = this->getContract&#x3C;C>(targetAddr);
      this->setContractVars(contract, caller->getContractAddress(), value);
      try {
        return contract->callContractFunction(this, func, args...);
      } catch (const std::exception&#x26; e) {
        throw DynamicException(e.what() + std::string(" - Type: ")
          + Utils::getRealTypeName&#x3C;C>() + " at address: " + targetAddr.hex().get()
        );
      }
    }
    default : {
      throw DynamicException("PANIC! ContractHost::callContractFunction: Unknown contract type");
    }
  }
}
</code></pre>
