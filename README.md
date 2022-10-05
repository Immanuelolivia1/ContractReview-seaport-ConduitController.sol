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
The parameter conduitkey it used to deploy the conduit.
The parameter initialOwner is the initial owner to set for the new conduit.
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
95.  new Conduit{ salt: conduitKey }();
```
The code above deploys the conduit via CREAT2 using the conduit key as the salt.
``` solidity
98. ConduitProperties storage conduitProperties = _conduits[conduit];
```
This initialize storage variable referencing conduit properties.
``` solidity
101. conduitProperties.owner = initialOwner;
```
This set the supplied initial owner as the owner of the conduit.
``` solidity
104. conduitProperties.key = conduitKey;
```
This sets conduit key used to deploy the conduit to enable reverse lookup.
``` solidity
107. emit NewConduit(conduit, conduitKey);
```
Emit an event indicating that the conduit has been deployed.
```solidity
 110. emit OwnershipTransferred(conduit, address(0), initialOwner);
    }
```
Emit an event indicating that conduit ownership has been assigned.
``` solidity
 125. function updateChannel(
 126.       address conduit,
 127.       address channel,
 128.       bool isOpen
 129.   ) external override {
 ```
 This function opens or closes a channel on a given conduit, therby allowing the specified account to execute transfers against that conduit.
 Only the owner of the conduit in question may call this function.
 The parameter conduit signifies the conduit for which to open or close the channel.
 This parameter channel signifies the channel to open or close on the conduit.
 this  parameter isOpen takes in a boolean indicating whether to open or close the channel.
 ``` solidity
131. _assertCallerIsConduitOwner(conduit);
This ensures that the caller is the current owner of the conduit in question.
``` solidity
134. ConduitInterface(conduit).updateChannel(channel, isOpen);
```
This calls the conduit, updating the channel.
``` solidity
137. ConduitProperties storage conduitProperties = _conduits[conduit];
```
Retrieve storage region where channels for the conduit are tracked.
``` solidity
140. uint256 channelIndexPlusOne = (
141.            conduitProperties.channelIndexesPlusOne[channel]
142.        );
```
Retrieve the index, if one currently exists, for the updated channel.
``` solidity
145. bool channelPreviouslyOpen = channelIndexPlusOne != 0;
```
This determines whether the updated channel is already tracked as open.
``` solidity
148. if (isOpen && !channelPreviouslyOpen) {
```
An if statement to check that channels has been set to pen and was previously closed
``` solidity
150. conduitProperties.channels.push(channel);
```
This adds the channel to the channels array for the conduit.
``` solidity
153. conduitProperties.channelIndexesPlusOne[channel] = (
154.                conduitProperties.channels.length
155.            );
156.        } else if (!isOpen && channelPreviouslyOpen) {
```
The codes on lines 153-156 adds new open channel length to associated mapping as index + 1.
``` solidity
uint256 removedChannelIndex;
```
This decrements located index to get the index of the closed channel.
``` solidity
163. unchecked {
164.                removedChannelIndex = channelIndexPlusOne - 1;
165.            }
```
This Skips underflow check as channelPreviouslyOpen being true ensures
that channelIndexPlusOne is not zero.
```solidity
168. uint256 finalChannelIndex = conduitProperties.channels.length - 1;
```
This uses the length of channels array to determine index of last channel.
``` solidity
171. if (finalChannelIndex != removedChannelIndex) {
173. address finalChannel = (
174.                    conduitProperties.channels[finalChannelIndex]
175.               );
```
This ensures that if closed channel is not last channel in the channels array the final channel and place the value on the stack is retrieved
``` solidity
conduitProperties.channels[removedChannelIndex] = finalChannel;
```
This overwrites the removed channel using the final channel value.
``` solidity
181. conduitProperties.channelIndexesPlusOne[finalChannel] = (
182.                    channelIndexPlusOne
183.                );
184.           }
```
Updates the final index in associated mapping to removed index.






















 
















  










   







