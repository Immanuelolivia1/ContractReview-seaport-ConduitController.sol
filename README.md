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
173. address finalChannel = 
(
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

``` solidity
conduitProperties.channels.pop();
```
 Remove the last channel from the channels array for the conduit.
 
 ``` solidity
 delete conduitProperties.channelIndexesPlusOne[channel];
        }
    }
```
this removes the colsed channel from the asociated mapping of indexes.
`` solidity
   
    function transferOwnership(address conduit, address newPotentialOwner)
        external
        override
    {
    ```
     /**
The function above is responsible for initiating conduit ownership transfer by assigning a new potential
owner for the given conduit. Once set, the new potential owner
may call `acceptOwnership` to claim ownership of the conduit.
Only the owner of the conduit in question may call this function.
The parameter conduit stands for the conduit for which to initiate ownership transfer.
The parameter newPotentialOwner stands for the new potential owner of the conduit.

``` solidity  
 _assertCallerIsConduitOwner(conduit);
```
This line of code ensures the caller is the current owner of the conduit in question.

``` solidity
if (newPotentialOwner == address(0)) {
            revert NewPotentialOwnerIsZeroAddress(conduit);
        }
```
The above code ensures the new potential owner is not an invalid address.
``` solidity
 if (newPotentialOwner == _conduits[conduit].potentialOwner) {
            revert NewPotentialOwnerAlreadySet(conduit, newPotentialOwner);
        }
```
This ensures the new potential owner is not already set.

``` solidity
 emit PotentialOwnerUpdated(newPotentialOwner);
```
Here an event is emited indicating that the potential owner has been updated.
``` solidity
   _conduits[conduit].potentialOwner = newPotentialOwner;
    }
```
This sets the new potential owner as the potential owner of the conduit.

``` solidity
 function cancelOwnershipTransfer(address conduit) external override {
 ```
 This function clears the currently set potential owner, if any, from a conduit.
 This can only be done by the owner of the conduit in questestion
 The parameter conduit The conduit for which to cancel ownership transfer.

``` solidity
        _assertCallerIsConduitOwner(conduit);
```
Here its ensured that the caller of this function is the current owner of the conduit in question.
``` solidity
        if (_conduits[conduit].potentialOwner == address(0)) {
            revert NoPotentialOwnerCurrentlySet(conduit);
        }
```
The above line of code ensures that ownership transfer is currently possible.

``` solidity
 emit PotentialOwnerUpdated(address(0));
```
The above line emits an event indicating that the potential owner has been cleared.
``` solidity
  _conduits[conduit].potentialOwner = address(0);
    }
```
This Clears the current new potential owner from the conduit.

``` solidity
    function acceptOwnership(address conduit) external override {
```
This function accepts ownership of a supplied conduit. Only accounts that the
current owner has set as the new potential owner may call this function.
The  prameter conduit signifies the conduit for which to accept ownership.
``` solidity
_assertConduitExists(conduit);
```
This ensures that the conduit in question exists.
``` solidity
        if (msg.sender != _conduits[conduit].potentialOwner) {
           revert CallerIsNotNewPotentialOwner(conduit);
        }
```
This checks if caller does not match current potential owner of the conduit,
it reverts indicating that caller is not current potential owner.

        emit PotentialOwnerUpdated(address(0));
This emits an event indicating that the potential owner has been cleared.
``` solidity
        _conduits[conduit].potentialOwner = address(0);
```
This clears the current new potential owner from the conduit.
``` solidity
emit OwnershipTransferred(
            conduit,
            _conduits[conduit].owner,
            msg.sender
        );
```
This emits an event indicating conduit ownership has been transferred.

``` solidity
        _conduits[conduit].owner = msg.sender;
    }
```
This sets the caller as the owner of the conduit.
``` solidity
function ownerOf(address conduit)
        external
        view
        override
        returns (address owner)
    {
```
This function retrieves the current owner of a deployed conduit 
return owner the owner of the supplied conduit.

The paramerter conduit The conduit for which to retrieve the associated owner.
``` solidity
        _assertConduitExists(conduit);
```
This ensures that the conduit in question exists.

``` solidity
owner = _conduits[conduit].owner;
    }
```
This retrieves the current owner of the conduit in question.

``` solidity
 function getKey(address conduit)
        external
        view
        override
        returns (bytes32 conduitKey)
    {
```
This function retrieves the conduit key for a deployed conduit via reverse
lookup, and it returns conduitKey used to deploy the supplied conduit.
This parameter conduit The conduit for which to retrieve the associated conduit
key.
``` solidity
        conduitKey = _conduits[conduit].key;
```
The above line of code attempts to retrieve a conduit key for the conduit in question.
``` solidity
if (conduitKey == bytes32(0)) {
            revert NoConduit();
        }
    }
``` 
The above if statement reverts if no conduit key was located.

``` solidity
 function getConduit(bytes32 conduitKey)
        external
        view
        override
        returns (address conduit, bool exists)
    {
```        
The function above derives the conduit associated with a given conduit key and
determine whether that conduit exists (i.e. whether it has been deployed) this function returns 
conduit which signifies the address derived from the coduit it also returns exist which is a
boolean indicating whether the derived conduit has been deployed or not.
The parameter conduitKey is the conduit key used to derive the conduit.
``` solidity
 conduit = address(
            uint160(
                uint256(
                    keccak256(
                        abi.encodePacked(
                            bytes1(0xff),
                            address(this),
                            conduitKey,
                            _CONDUIT_CREATION_CODE_HASH
                        )
                    )
                )
            )
        );
```
The above line of code derive address from deployer, conduit key and creation code hash.
``` solidity
exists = (conduit.codehash == _CONDUIT_RUNTIME_CODE_HASH);
    }
```    
Determine whether conduit exists by retrieving its runtime code.
``` solidity
 function getPotentialOwner(address conduit)
        external
        view
        override
        returns (address potentialOwner)
    {
```
This function retrieves the potential owner, if any, for a given conduit. The
current owner may set a new potential owner via`transferOwnership` and that 
owner may then accept ownership of the conduit in question via `acceptOwnership`
and it returns the potential owner, if any, for the conduit.
The conduit parameter signifies the conduit for which to retrieve the potential owner.
``` solidity
        _assertConduitExists(conduit);
```        
Ensures that the conduit in question exists.
``` solidity
 potentialOwner = _conduits[conduit].potentialOwner;
    }
```    
Retrieves the current potential owner of the conduit in question.
``` solidity
   function getChannelStatus(address conduit, address channel)
        external
        view
        override
        returns (bool isOpen)
    {       
```
This function retrieves the status (either open or closed) of a given channel on
a conduit and it returns the status of thr channel on the given conduit.
The parameter conduit signifies the conduit for which to retrieve the channel status.
The parameter channel signifies the channel for which to retrieve the status.
``` solidity
 _assertConduitExists(conduit);
```
Ensures that the conduit in question exists.
``` solidity
  isOpen = _conduits[conduit].channelIndexesPlusOne[channel] != 0;
    }
```    
Retrieves the current channel status for the conduit in question.
``` solidity
 function getTotalChannels(address conduit)
        external
        view
        override
        returns (uint256 totalChannels)
    {
```
This function retrieves the total number of open channels for a given conduit,
it returns the total number of open channels for the conduit.
Here the conduit parameter signifies the conduit for which to retrieve the total channel count.
``` solidity
        _assertConduitExists(conduit);
```
Ensures that the conduit in question exists.
``` solidity
totalChannels = _conduits[conduit].channels.length;
    }
```   
Retrieves the total open channel count for the conduit in question.
``` solidity
 function getChannel(address conduit, uint256 channelIndex)
        external
        view
        override
        returns (address channel)
    {
```    

The functoin above retrieves an open channel at a specific index for a given conduit and 
returns the open channel, if any, at the specified channel index.
 ``` solidity
        _assertConduitExists(conduit);
 ```
Ensures that the conduit in question exists.
``` solidity
        uint256 totalChannels = _conduits[conduit].channels.length;
```
Retrieves the total open channel count for the conduit in question.
``` solidity
if (channelIndex >= totalChannels) {
            revert ChannelOutOfRange(conduit);
        }
```        
Ensures that the supplied index is within range.
``` solidity
channel = _conduits[conduit].channels[channelIndex];
    }
```    
Retrieves the channel at the given index.
``` solidity
function getChannels(address conduit)
        external
        view
        override
        returns (address[] memory channels)
    {
```
The function above retrieves all open channels for a given conduit. Note that calling
this function for a conduit with many channels will revert with an out-of-gas error 
it returns an array of open channels on the given conduit.
The parameter conduit signifies the conduit for which to retrieve open channels.
 
``` solidity
        _assertConduitExists(conduit);
```
Ensures that the conduit in question exists.

        // Retrieve all of the open channels on the conduit in question.
        channels = _conduits[conduit].channels;
    }

    /**
     * @dev Retrieve the conduit creation code and runtime code hashes.
     */
    function getConduitCodeHashes()
        external
        view
        override
        returns (bytes32 creationCodeHash, bytes32 runtimeCodeHash)
    {
        // Retrieve the conduit creation code hash from runtime.
        creationCodeHash = _CONDUIT_CREATION_CODE_HASH;

        // Retrieve the conduit runtime code hash from runtime.
        runtimeCodeHash = _CONDUIT_RUNTIME_CODE_HASH;
    }

    /**
     * @dev Private view function to revert if the caller is not the owner of a
     *      given conduit.
     *
     * @param conduit The conduit for which to assert ownership.
     */
    function _assertCallerIsConduitOwner(address conduit) private view {
        // Ensure that the conduit in question exists.
        _assertConduitExists(conduit);

        // If the caller does not match the current owner of the conduit...
        if (msg.sender != _conduits[conduit].owner) {
   // Revert, indicating that the caller is not the owner.
            revert CallerIsNotOwner(conduit);
        }
    }

    /**
     * @dev Private view function to revert if a given conduit does not exist.
     *
     * @param conduit The conduit for which to assert existence.
     */
    function _assertConduitExists(address conduit) private view {
        // Attempt to retrieve a conduit key for the conduit in question.
        if (_conduits[conduit].key == bytes32(0)) {
            // Revert if no conduit key was located.
            revert NoConduit();
        }
    }
    
}    
