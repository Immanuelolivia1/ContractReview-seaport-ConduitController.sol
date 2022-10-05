## Seaport

Seaport is a marketplace contract for safely and efficiently creating and fulfilling orders for ERC721 and ERC1155 items. Each order contains an arbitrary number of items that the offerer is willing to give (the "offer") along with an arbitrary number of items that must be received along with their respective receivers (the "consideration").

One could say seaport is a generalizied ETH/ERC20/ERC721/ERC1155 marketplace.

### Conduit Controller
ConduitController enables deploying and managing new conduits, or contracts that allow registered callers (or open "channels") to transfer approved ERC20/721/1155 tokens on their behalf.

```
1. pragma solidity ^0.8.7;
```
specifies the compiler version that should be used 

```
4. import {
5.   ConduitControllerInterface
6. } from "../interfaces/ConduitControllerInterface.sol";
```
ConduitControllerInterface.sol contains all external functions, interfaces, struct, events  and errors for the ConduitController.sol.
The ConduitController.sol uses a struct form the ConduitControllerInterface.sol and this struct tracks the current key, current owner, new potential owner and open channels for each deployed conduit.
```
8. import { ConduitInterface } from "../interfaces/ConduitInterface.sol";
```
ConduitInterface.sol contains all external function interfaces, events and error for conduit contracts.
The NewConduit event which takes an address and a bytes32(conduit,conduitkey) was emitted in the ConduitController.sol

```
10. import { Conduit } from "./Conduit.sol";
```



   







