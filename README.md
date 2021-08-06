---
eip: <to be assigned>
title: Smart Non Fungible Token (SmartNFT)
author: Javier Arcenegui <arcenegui@imse-cnm.csic.es>, Rosario Arjona <arjona@imse-cnm.csic.es>, Roberto Román <roman@imse-cnm.csic.es> and Iluminada Baturone <lumi@imse-cnm.csic.es>
discussions-to: https://github.com/Hardblock-IMSE/Smart-Non-Fungible-Token
status: Draft
type: Standards Track
category (*only required for Standards Track): ERC
created: 2021-04-22
requires (*optional): 721
---
## Simple Summary
A Standar interface to Smart Non Fungible Tokens binded a IoT devices thats generate their own blockchain account.

## Abstract
Current Non-fungible tokens (NFTs) allow representing assets by a unique identifier, as a possession of an owner. The novelty introduced in this EIP is the proposal of smart NFTs to represent IoT devices, which are physical smart assets. Hence, they are also identified as the utility of a user, they have a blockchain account (BCA) address to participate actively in the blockchain transactions, they can establish secure communication channels with owners and users, and they operate dynamically with several modes associated with their token states. A smart NFT is physically bound to its IoT device thanks to the use of a physical unclonable function (PUF) that allows recovering its private key and, then, its BCA address.
 
## Motivation
Current NFTs are designed like pasive assets, and this assets only have an owner, or in the best of case a manager for an owner. In the Smart Contract literature apear a lot of projects about renting, or managment system of tokens. In this proposal we introduce the role of user and the role of device. The IoT device can interact actively with other blockchain participants, that is, it can have a BCA. Hence, we propose the addition of the NFT attribute device, which is the BCA address associated with the IoT device. Besides, an IoT device is not only a possession of an owner but also an active agent that obeys to a user to carry out certain tasks in an application. Hence, we propose the addition of the NFT attribute user, which is the BCA address of the user of the IoT device. In order to represent the dynamic activity of the IoT device, six more attributes are proposed: timestamp, timeout, state, hashK_OD, hashK_UD, and dataEngage.  The first of them, timestamp, registers in the blockchain whenever the device checks it is bound with its token. This is very important to register if the bound is alive or not. timeout define how many time is the maximum time to update timestamp from device to be considered still on live.

## Specification
 
This kind of non-fungible token is designend to associated with a secure device with the capacity of generateits own BCA and interact with the BC. And this SmartNFT is designed to assign the token to a user if its necesary. But this SmartNFT can be used like an improved ERC721 token. This mean that an normal token can be improve with managment of user system of this kind of token. Or only to interaction of token binded to a device without users.
 
In the original desing is important that the device can save securely its BCA and not share it to avoid a spoofed device. This BCA can be save using a SRAM PUF o a truth memory zone. Only manufacturer can generate a new token, that ensures the identity of the device. And only manufacturer can compile the an provide the firmware to regenerate the BCA of the device. This firmware serves like a bootloader to upload the owner or user firmware. This firmware cant modify the data require to generate the BCA, and can not recover the secrect key of the BCA. Owner have de capacity of transfer and update the firmware of the tokens, but owner can not overwrite the bootloader layer described to the manufacturer. To do this, the firmware must be sign by manufacturer in a secure bootloader process. Owner assign a user to use the token, if a owner need to use the token need to assign itself like user. Depending of how is described the smart contract an owner can decline the use of the token of an user. Only a user can use the token.
 
To complete a token transaction the new owner must authenticate offchain and onchain whith the token using their blockchain account. The smart contract define how owner start a new authentication and how the token can complete this autentication. This autentication is necesary to the device know who is its owner and stay secure that is not an impostor. In the same way, the smart contract define how the user and device must be authenticate in the same way. This process is descrive below.
 
![Figure 1 : State diagram of the token](/Images/Figure1.png)

An IoT device is a dynamic asset that can change of operating modes, which can be represented by states. In particular, the states defining if the device has been engaged or not with an owner or with a user deserve attention because the software to be executed by the device, which should be verified prior to be executed, is different. Hence, we propose the addition of the NFT attribute state and propose that the operating mode of the device should be in correspondence with its token state. We consider four main states of the to-ken. The state Waiting for owner defines the situation whenever the token is created or transferred to a new owner but device and owner have not verified mutually yet. Once they verify each other, the device recognizes its owner and the state of its token changes to Engaged with owner. In this state, the owner can transfer the token to a user. If the token is transferred to a user that coincides with the owner, the token state changes to Engaged with user and the device is ready to operate in its application, obeying the commands from its user. Otherwise, the token changes its state to Waiting for user. Once the user and the IoT device are verified mutually, the token changes its state to Engaged with user. From this state, the token can be transferred to another user, thus returning to Waiting for user state. To cope with a user that could not reply, the token can be transferred to another user if its state is Waiting for user. From all the states, the token can be transferred to another owner, thus returning to Waiting for owner state. Figure 1 illustrates the state diagram of the token. The state changes are controlled by token functions as explained below.
 


| Type | Name of variable | Defined by ERC-721 | Optional |
|--|--|--|--|
| uint256 | tokenId | Yes | No |
| address | owner | Yes | No |
| address | approved | Yes | Yes |
| address | approvedForAll | Yes | Yes |
| address | device | No | Yes* |
| address | user | No | No |
| enum | state | No | No |
| uint256 | hashK_OD | No | Yes |
| uint256 | HashK_UD | No | Yes |
| uint256 | dataEngagement | No | Yes |
| uint256 | timeStamp | No | No |
| uint256 | timeout | No | No |
 
The atributes of don't need to be in the same struct, but is important tha existe coherence to know the data of the token easily. This organization is depending only by the designer of the smart contract and the characteristic of project. Like in the ERC721, the role of operator (aprovedForAll) is an attribute from the owner, because is the operator of an specefic owner, not is an atribute of the token. 
 
```solidity
 pragma solidity ^0.8.0;

 /// @title smartNFT: Extension of ERC-721 Non-Fungible Token Standard. 
 interface smartNFT is ERC721{
    /// @dev This emits when ownership of any NFT changes an user.
    ///  This event emits when the user of a token is transfered.
    ///  (`_addressUser` == 0) when a user is unassigned.
    event UserAssigned(uint256 indexed tokenID, address indexed _addressUser);
    
    /// @dev This emits when user of any NFT verify a device.
    ///  This event emits when the user a device finishing the assignment.
    event UserEngaged(uint256 indexed tokenID);
    
    /// @dev This emits when ownership  any NFT verify a device.
    ///  This event emits when the owner a device finishing the transference.
    event OwnerEngaged(uint256 indexed tokenID);
    
    /// @dev This emits when the SC verify the Timeout is extinted.
    ///  This event emits when the timestamp of token is not updated in timeout.
    event TimeoutAlarm(uint256 indexed tokenID);

    /// @notice This function define the transference of use of a Smart Device
    /// @dev Only the owner of token account can transfer a token and the state of token need to 
    /// "engagedWithOwner","waitingForUser" or "engagedWithUser". The state of token must change to
    /// "waitingForUser" and change the parameter addressUser of token defined by _tokenID with _addressUser
    /// @param _tokenId the tokenID of Smart Device.
    /// @param _addressUser The address of the new user.
    function setUser(uint256 _tokenId, address _addressUser) external; 

    /// @notice This function define the initiation of authentication between owner and device
    /// @dev Only the owner of token can start this authentication if the state of token is
    /// "waitingForOwner". The state of token must not change, but save _dataEngage and _hashK_O
    /// in the parameters of tokens.
    /// @param _tokenId the tokenID of Smart Device
    /// @param _dataEngage defines the public data needed for the agreement
    /// @param _hashK_O The hash of shared between owner and device generated by owner
    function startOwnerEngagement(uint256 _tokenId, uint256 _dataEngage, uint256 _hashK_O) external;
 
        /// @notice This function complete the authentication between owner and device
    /// @dev Only the device binded of token can start finish authentication if the state of token is
    /// "waitingForOwner" and dataEngage if different of 0. This function must compare hashK_O saved in
    /// the token. If are equal then the state of token must change to "engagedWithOwner" and set to 0
    /// dataEngage. Finally emit the event "OwnerEngaged".
    /// @param _hashK_D The hash of shared between user and device generated by device
    function ownerEngagement(uint256 _hashK_D) external; 
 
    /// @notice This function define the initiation of authentication between user and device
    /// @dev Only the user of token can start this authentication if the state of token is
    /// "waitingForUser". The state of token must not change, but save _dataEngage and _hashK_D
    /// in the parameters of the token.
    /// @param _tokenId the tokenID of Smart Device
    /// @param _dataEngage defines the public data needed for the agreement
    /// @param _hashK_D The hash of shared between user and device generated by user
    function startUserEngagement(uint256 _tokenId, uint256 _dataEngage, uint256 _hashK_U) external;
    
    /// @notice This function complete the authentication between user and device
    /// @dev Only the device binded of token can start finish authentication if the state of token is
    /// "waitingForUser" and dataEngage if different of 0. This function must compare hashK_D saved in
    /// the token. If are equal then the state of token must change to "engagedWithUser" and set to 0
    /// dataEngage. Finally emit the event "UserEngaged".
    /// @param _hashK_D The hash of shared between user and device generated by device
    function userEngagement(uint256 _hashK_D) external; 
 
    /// @notice This function check if the timeout is extinted
    /// @dev Everybody can call this function, check if the timeout is extinted. In that case emit 
    /// the event "TimeoutAlarm"
    /// @param _tokenId the tokenID of Smart Device
    /// @return true if timeout is extinted and false in other case.
    function checkTimeout(uint256 _tokenId) external returns (bool);
    
    /// @notice This function change the limit of time known as timeout
    /// @dev Only the owner of token can change this limit only if the state is "engagedWithOwner",
    /// "waitingForUser" or "engagedWithUser".
    /// @param _tokenId the tokenID of Smart Device
    /// @param _timeout the new limit of time
    function setTimeout(uint256 _tokenId, uint256 _timeout) external; 
    
    /// @notice This function update the timestamp to avoid the timeout alarm.
    /// @dev Only the device binded of token can update its own timestamp.
    function updateTimestamp() external; 
    
    /// @notice This function let obtain the tokenID from an address of Smart Device 
    /// @dev Everybody can call this function, can be execute any code on blockchain only read.
    /// @param _addressSD the addres to obtain it's token ID
    /// @return The tokenID of the token generated binded to _addressSD.
    function tokenFromBCA(address _addressSD) external view returns (uint256);
    
    /// @notice This function let know  who is the owner of token by the address of Smart Device
    /// @dev Everybody can call this function, can be execute any code on blockchain only read.
    /// @param _addressSD the addres to obtain it's owner.
    /// @return The owner of the token generated binded to _addressSD.
    function ownerOfFromBCA(address _addressSD) external view returns (address);
    
    /// @notice This function let know  who is the user of the token with its tokenID
    /// @dev Everybody can call this function, can be execute any code on blockchain only read.
    /// @param _tokenId of the token to obtain it's user.
    /// @return The user of the token with _tokenId.
    function userOf(uint256 _tokenId) external view returns (address);
    
    /// @notice This function let know  who is the user of token by the address of Smart Device
    /// @dev Everybody can call this function, can be execute any code on blockchain only read.
    /// @param _addressSD the addres to obtain it's user.
    /// @return The user of the token generated binded to _addressSD.
    function userOfFromBCA(address _addressSD) external view returns (address);
    
    /// @notice This function let know many tokens have an user.
    /// @dev Everybody can call this function, can be execute any code on blockchain only read.
    /// @param _addressUser the address to know the number of device can use.
    /// @return the number of device can use.
    function userBalanceOf(address _addressUser) external view returns (uint256);
    
    /// @notice This function let know many tokens of an owner have an user.
    /// @dev Everybody can call this function, can be execute any code on blockchain only read.
    /// @param _addressUser the address to know the number of device of an owner can use.
    /// @param _addressOwner the address the owner of this tokens.
    /// @return the number of device of an owner can use.
    function userBalanceOfAnOwner(address _addressUser, address _addressOwner) external view returns (uint256);
}
```
This interface is an extesion of the ERC721, and not is only compatible with the standard, this standard needs of ERC721 interface. This interface is available in https://eips.ethereum.org/EIPS/eip-721. Then metadata extension and enumaration extension are compatible and include in this draft.
 
## Rationale
This smartNFT was developed becouse the ERC721 standard do not take into account the users of an asset, only the ownership. In addition we decided implement a secure way to share a secret key confirmed with the consensus of the blockchain. In this way, device and owner, or device and user can be sure thar they are exchange information only with its interested.

**Secure Device binded to a SmartNFT**
 
 An Non-Fungible token usually is associated to an passive asset like virtual o physical things. But it is not exist any standardized mechanism to bind a Non-Fungible token to an assets. This is interesting because the device know in any moment who is its owner o who can use it. Then its not only necesary a transference, but also need any kind of authentication to verify de transference. The device have more control of itself for example to revoke orders form a non authorized user, or dont work until the authentication with the user or the owner.
 
**Users Management Mechanism**
 
We implement a new and usefull user management mechanism, in the last years a lot of project about vehicles sharings or assets sharing in general have been created and developed. In this way, we incorporate the blockchain account of the user of the token as another atribbute in order to distinguish between users, who employ the asset for an expecific application, and owners, who assign the token to users.
 
**Secure Key Exchange Mechanism**
 
The engagements of the device with an owner and with a user are carried out after mutual authentication protocols based on elliptic curve Diffie-Hellman key exchange protocols. These protocols allow a key agreement between the device and its owner, in the one side, and the device and its user, in the other side. 
 
When the Smart NFT is created or when the owmnership is transfered, the operating mode of the device defined in the state is Waiting for owner. then the device saves in its memory the owner BCA address. Owner generate a pair of keys using secp256k1. A secret key (SK<sub>OD</sub>) and a Public Key (PK<sub>OD</sub>). To generate the key between owner and device (K<sub>OD</sub>) is need the public key of device (PK<sub>dev</sub>) and the point P such that PK<sub>OD</sub> = SK<sub>OD</sub>*P.
 
K<sub>OD</sub>=PK<sub>dev</sub>*SK<sub>OD</sub>

Using startOwnerEngage, owner save PK<sub>OD</sub> and the hash of K<sub>OD</sub> in the smart contract. Owner send PK<sub>OD</sub> signed to device. The device check the sign to verify the identity of the owner and calculate K<sub>OD</sub> using P such that PK<sub>dev</sub> = SK<sub>dev</sub>*P.

K<sub>OD</sub>=PK<sub>dev</sub>*SK<sub>OD</sub>=(SK<sub>dev</sub>*P)*SK<sub>OD</sub>= SK<sub>dev</sub>*(SK<sub>OD</sub>*P)=SK<sub>dev</sub>*PK<sub>OD</sub>

Using ownerEngage function, the device send K<sub>OD</sub> obtain and if it is the same Key then changes the state of the token to Engaged with owner and sends the event OwnerEngaged. Once the device receives the event, it changes its operation mode to Engaged with owner. This process is shown in Figure 2. From this moment, the device can be managed by the owner. 
 
 ![Figure 2 : Steps in successful owner and device muthual authentication](/Images/Figure2.PNG)
 
If the device consults the blockchain and the state of its Smart NFT is Waiting for user, the device saves in its memory the user BCA address. Then, a mutual authentication process is carried out with the user, as already done with the owner. If the user verifies the device, the user sends the transaction associated to the function startUserEngage to the smart contract. Analogously to starOwnerEngage this function save the public key generated by user PK<sub>UD</sub> and the hash of PK<sub>dev</sub>*SK<sub>UD</sub>.
User send PK<sub>UD</sub> signed to device that check the signature an optionally check PK<sub>UD</sub> on the dataEngage attribute of its token. If verify its user, call userEngage function in the smartcontract with K<sub>UD</sub> such that
 
K<sub>UD</sub>=PK<sub>dev</sub>*SK<sub>UD</sub>=(SK<sub>dev</sub>*P)*SK<sub>UD</sub>= SK<sub>dev</sub>*(SK<sub>UD</sub>*P)=SK<sub>dev</sub>*PK<sub>UD</sub>

 The smart contract compare if K<sub>UD</sub>=PK<sub>dev</sub>*SK<sub>UD</sub> and if it is equal, then changes the state of the token to Engaged with user and sends the event UserEngaged. Once the device receives the event, it changes its operation mode to Engaged with user. This process is shown in Figure 3. 

 ![Figure 3 : Steps in successful owner and device muthual authentication](/Images/Figure3.PNG)
 
Since the establishment of a shared secret is very important for a secure communication between them, we propose the inclusion of the attributes hashK_OD, hashK_UD, and dataEngage. The first two attributes define, respectively, the hash of the secret shared between the device and its owner and between the device and its user. Devices, owners, and users should check they are using the correct shared secrets. The attribute dataEngage defines the public data needed for the agreement. If the mutual authentication fails, dataEngage allows detecting which parts failed.
  
## Backwards Compatibility
This implementation is an extension of ERC-721 standard, then is not only compatible with the standard, but also is a way to improve some of the actual NFT token based on the ERC-721 standard.

## Test Cases
The test case presented on the article show below is addressed in <b>0x7eB5A03E7ED70ABf70fee48965D0411d37F335aC</b> and the code is available in <b>https://github.com/Hardblock-IMSE/Smart-Non-Fungible-Token</b>

## Reference Implementation
A first version was presented on the special issue <b>Security, Trust and Privacy in New Computing Environments)</b> of <b>Sensors</b> magazine of <b>mdpi</b> editorial titled <b><k>Secure Combination of IoT and Blockchain by Physically Binding IoT Devices to Smart Non-Fungible Tokens Using PUFs</k></b> writted by the same authors of this draft. The doi of this article is: <b>https://doi.org/10.3390/s21093119</b>

## Security Considerations
In this draft, a generic system has been proposed for the creation of non-fungible tokens with the possibility of being intelligent. Although it has been wanted to give a generic point of view based on the improvements to the current ERC-721 standard, such as the implementation of the user management mechanism. The main idea is to relate a physical device with a non-fungible token. This physical device should have the ability not only to generate a blockchain account from itself in a totally random way but also the ability to regenerate it, without share this information. In this way, identity theft is avoided and the system is completely genuine. In order to achieve this, it has been established that only the manufacturer of devices has the ability to create tokens, although as mentioned before it is only a recommendation, since it is intended to be backward compatible. In this way, the generated firmware will be unable to obtain this information. It is recommended that systems be used that allow each device to obtain the blockchain account from non-sensitive information such as the data helper using SRAM PUF. 
Similarly, a secure key exchange system has been proposed, however the system is open to coexist with other types of key exchange. 
Regarding the security improvements implemented with respect to the ERC-721 standard, without counting the previous mentions, a user management system is established, so the use of an asset or device does not affect the token's membership.


## Copyright
TODO: No se muy bien que poner aquí
