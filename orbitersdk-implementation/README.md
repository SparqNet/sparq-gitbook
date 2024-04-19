---
description: How the functional elements of OrbiterSDK interact with each other.
---

# OrbiterSDK implementation

This subchapter aims to explain in technical detail how OrbiterSDK is implemented, as well as its submodules and how everything comes together to deliver a blazing fast native blockchain.

The first few subchapters paint a more holistic view of the OrbiterSDK project, as most components are pretty straight-forward to understand, and developers are expected to use the [Doxygen](https://doxygen.nl) documentation as a reference to further understand how the project works. The later subchapters show some components that are particularly denser and/or complex enough that they warrant their own separated explanations.

Looking at a higher level of abstraction, the original C++ implementation of OrbiterSDK is structured like this:

* The `src/bins` folder contains the source files for the project's main executables - the blockchain executable itself, contract ABI generator and other testing-related executables are all coded here in their respective subfolders
* The `src/contract` folder contains everything related to the logic of smart contracts - from ABI parsing to custom variable types and template contracts
* The `src/core` folder contains the heart of OrbiterSDK - the main components of the blockchain and what makes it tick
* The `src/libs` folder contains third-party libraries not inherently tied to the project but used throughout development
* The `src/net` folder contains everything related to networking, communication between nodes and support for protocols such as gRPC, HTTP, P2P, JSON-RPC, etc.
* The `src/utils` folder contains several commonly-used functions, structures, classes and overall logic to support the functioning of OrbiterSDK as a whole

### Source tree

For the more visually inclined, here is a source tree (headers only) containing all of the files inside the `src` folder (except `src/bins` as it only contains source files), their respective subfolders and which components are declared in them. Each component is further explained through the following subchapters of this documentation. For more technical details (e.g. API references for developers), please refer to the [Doxygen](https://www.doxygen.nl) documentation on the project's own repository.

```
src
├── contract (Contracts)
│   ├── abi.h (ABI - encoders, decoders, helper structs, etc.)
│   ├── contractcalllogger.h (ContractCallLogger)
│   ├── contractfactory.h (ContractFactory)
│   ├── contract.h (ContractGlobals, ContractLocals, BaseContract)
│   ├── contractmanager.h (ContractManager, ContractManagerInterface)
│   ├── customcontracts.h (for declaring custom contracts)
│   ├── dynamiccontract.h (DynamicContract)
│   ├── event.h (Event, EventManager)
│   ├── templates (folder for contract templates)
│   │   ├── dexv2 (subfolder for the DEXV2 contract components)
│   │   │   ├── dexv2factory.h (DEXV2Factory)
│   │   │   ├── dexv2library.h (DEXV2Library)
│   │   │   ├── dexv2pair.h (DEXV2Pair)
│   │   │   ├── dexv2router02.h (DEXV2Router02)
│   │   │   └── uq112x112.h (UQ112x112 - used in DEX contracts for fixed-point fractions)
│   │   ├── erc20.h (ERC20)
│   │   ├── erc20wrapper.h (ERC20Wrapper)
│   │   ├── erc721.h (ERC721)
│   │   ├── erc721test.h (ERC721Test, used solely for testing purposes)
│   │   ├── nativewrapper.h (NativeWrapper)
│   │   ├── simplecontract.h (SimpleContract)
│   │   ├── testThrowVars.h (TestThrowVars, used solely for testing purposes)
│   │   └── throwtestA.h, throwtestB.h, throwtestC.h (for testing CM nested calls)
│   └── variables (Safe Variables for use within Dynamic Contracts)
│       ├── reentrancyguard.h (ReentrancyGuard)
│       ├── safeaddress.h (SafeAddress)
│       ├── safearray.h (SafeArray)
│       ├── safebase.h (SafeBase - used as base for all other types)
│       ├── safebool.h (SafeBool)
│       ├── safeint.h (SafeInt)
│       ├── safestring.h (SafeString)
│       ├── safetuple.h (SafeTuple)
│       ├── safeuint.h (SafeUint)
│       ├── safeunorderedmap.h (SafeUnorderedMap)
│       └── safevector.h (SafeVector)
├── core (Core components)
│   ├── blockchain.h (Blockchain, Syncer)
│   ├── rdpos.h (Validator, rdPoS, rdPoSWorker)
│   ├── snowmanVM.h (InitializeRequest, SnowmanVM)
│   ├── state.h (State)
│   └── storage.h (Storage)
├── libs (Third-party libs)
│   ├── BS_thread_pool_light.hpp (https://github.com/bshoshany/thread-pool)
│   ├── catch2/catch_amalgamated.hpp (https://github.com/catchorg/Catch2)
│   └── json.hpp (https://github.com/nlohmann/json)
├── net (Networking)
│   ├── grpcclient.h (gRPCClient)
│   ├── grpcserver.h (gRPCServer)
│   ├── http (HTTP part of networking)
│   │   ├── httplistener.h (HTTPListener)
│   │   ├── httpparser.h (parser functions for HTTP requests)
│   │   ├── httpserver.h (HTTPServer)
│   │   ├── httpsession.h (HTTPQueue, HTTPSession)
│   │   └── jsonrpc (Namespace for handling JSONRPC data)
│   │       ├── decoding.h (functions for decoding JSONRPC data)
│   │       ├── encoding.h (functions for encoding JSONRPC data)
│   │       └── methods.h (declarations for JSONRPC methods)
│   └── p2p (P2P part of networking)
│       ├── client.h (ClientFactory)
│       ├── discovery.h (DiscoveryWorker - worker thread for ManagerDiscovery)
│       └── encoding.h (collection of enums, structs, classes, encoders and decoders used in P2P communications)
│       ├── managerbase.h (ManagerBase - used as base for ManagerDiscovery and ManagerNormal)
│       ├── managerdiscovery.h (ManagerDiscovery)
│       ├── managernormal.h (ManagerNormal)
│       ├── server.h (ServerListener, Server)
│       └── session.h (Session)
└── utils (Base components)
    ├── contractreflectioninterface.h (ContractReflectionInterface - interface for registering contracts)
    ├── db.h (DBPrefix, DBServer, DBEntry, DBBatch, DB)
    ├── dynamicexception.h (DynamicException - custom exception class)
    ├── ecdsa.h (PrivKey, Pubkey, UPubkey, Secp256k1)
    ├── finalizedblock.h (FinalizedBlock)
    ├── hex.h (Hex)
    ├── jsonabi.h (JsonAbi - namespace for writing contract ABIs to JSON format)
    ├── logger.h (LogType, Log, LogInfo, Logger)
    ├── merkle.h (Merkle)
    ├── mutableblock.h (MutableBlock)
    ├── options.h (Options singleton - generated by CMake through a .in file)
    ├── randomgen.h (RandomGen)
    ├── safehash.h (SafeHash, FNVHash)
    ├── strings.h (FixedBytes, Hash, Functor, Signature, Address)
    ├── tx.h (TxBlock, TxValidator)
    └── utils.h (definitions for Bytes/Account structs, uint/ethCallInfo types, Networks, and the Utils namespace)
```
