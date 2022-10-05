## Seaport

Seaport is a marketplace contract for safely and efficiently creating and fulfilling orders for ERC721 and ERC1155 items. Each order contains an arbitrary number of items that the offerer is willing to give (the "offer") along with an arbitrary number of items that must be received along with their respective receivers (the "consideration").

One could say seaport is a generalizied ETH/ERC20/ERC721/ERC1155 marketplace.

### Conduit Controller
ConduitController enables deploying and managing new conduits, or contracts that allow registered callers (or open "channels") to transfer approved ERC20/721/1155 tokens on their behalf.

``` solidity
1. pragma solidity ^0.8.7;
```
specifies the compiler version that should be used 

``` solidity
4. import {
5.   ConduitControllerInterface
6. } from "../interfaces/ConduitControllerInterface.sol";
```
ConduitControllerInterface.sol contains all external functions, interfaces, struct, events  and errors for the ConduitController.sol.
The ConduitController.sol uses a struct form the ConduitControllerInterface.sol and this struct tracks the current key, current owner, new potential owner and open channels for each deployed conduit.
``` solidity
8. import { ConduitInterface } from "../interfaces/ConduitInterface.sol";

```
ConduitInterface.sol contains all external function interfaces, events and error for conduit contracts.
The NewConduit event which takes an address and a bytes32(conduit,conduitkey) was emitted in the ConduitController.sol

``` solidity
10. import { Conduit } from "./Conduit.sol";
```
The Conduit.sol serves as an originator for "proxied" transfers. Each conduit is deployed and controlled by the ConduitController.sol that can add and remove "channels" or contracts that can instruct the Conduit.sol to transfer approve ERC20/721/1155 tokens. 

``` solidity
21. mapping(address => ConduitProperties) internal _conduits;
```
The above is a mapping of addresses to a struct registering keys, owners, new potential owners and channels by conduit
``` solidity
24. bytes32 internal immutable _CONDUIT_CREATION_CODE_HASH;
```
The above code sets the hash of the conduit creation code as an immutable argument.
``` solidity
25. bytes32 internal immutable _CONDUIT_RUNTIME_CODE_HASH;
```
The above code sets the hash of the runtime code as an immutable argument.
``` solidity
31. constructor() {
```
This constructor initialize the contract by deploying a conduit and setting the creation code and runtime code hashes as immutable arguments.
``` solidity
33. _CONDUIT_CREATION_CODE_HASH = keccak256(type(Conduit).creationCode);
```
This derive the conduit creation code hash and sets it as an immutable.
``` solidity
36. Conduit zzeroConduit = new Conduit{ salt: bytes32(0) }();
```
This deploys a conduit with the zero hash as the salt.
``` solidity
39. _CONDUIT_RUNTIME_CODE_HASH = address(zeroConduit). codehash;
40. }
```
This retrieve the conduit runtime code hash and set it as an immutable.
``` solidity
41. function createConduit(bytes32 conduitKey, address initialOwner)
        external
        override
        returns(address conduit)
    {
```
This function deploys a new conduit using a supplied conduit key and assigning an initial owner for the deployed conduit.
The argument conduitkey it used to deploy the conduit.
The argument initialOwner is the initial owner to set for the new conduit.
This function returns the address of the newly deployed conduit.
``` solidity
62. if (initialOwner == address(0)) {
63.            revert InvalidInitialOwner();
64.        }
```
An if statement to ensure that an initail owner has been supplied.
``` solidity
67. if (address(uint160(bytes20(conduitKey))) != msg.sender) {
69.                revert InvalidCreator();
70.        }
```
This if statement ensures that if the first 20 bytes of the conduit key do not match the  caller it reverts with an error indicating that the creator is invalid.
``` solidity
 73. conduit = address(
 74.           uint160(
 75.               uint256(
 76.                   keccak256(
 77.                       abi.encodePacked(
 78.                           bytes1(0xff),
 79.                           address(this),
 80.                           conduitKey,
 81.                           _CONDUIT_CREATION_CODE_HASH
 82.                       )
 83.                   )
 84.               )
 85.           )
 86.       );
```
This derives address from deployer, conduit key and creation code hash.
``` solidity
89. if (conduit.codehash == _CONDUIT_RUNTIME_CODE_HASH) {
91. revert ConduitAlreadyExists(conduit);
92.        }
```
Here if derived conduit exists, aas evidenced by comparing runtime code this should revert with an error indicating that the conduit already exists.
``` solidity
 new Conduit{ salt: conduitKey }();
```
The code above deploys the conduit via CREAT2 using the conduit key as the salt.
``` solidity






  










   







