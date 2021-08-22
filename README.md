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
A standar interface of Smart Non Fungible Tokens representing secure IoT devices that generate their own blockchain accounts.

## Abstract
Current Non-Fungible Tokens (NFTs) allow representing assets by a unique identifier, as a possession of an owner. The novelty introduced in this EIP is the proposal of smart NFTs to represent IoT devices, which are physical smart assets. Hence, they have a blockchain account (BCA) address to participate actively in the blockchain transactions, they are also identified as the utility of a user, they can establish secure communication channels with owners and users, and they operate dynamically with several modes associated with their token states. A smart NFT is physically bound to its IoT device because the device is the only one able to generate its BCA address from its private key. The device is the only one in possesion of its private key. This can be ensured, for example, if the device does not store the private key but uses a physical unclonable function (PUF) that allows recovering its private key.
 
## Motivation
Current NFTs are designed to represent passive assets, which only have an owner that owns the token, can transfer ownership, and can approve others to act in its name. However, IoT devices are smart assets that can interact actively with other blockchain participants, that is, they can have a BCA. Hence, we propose the addition of the NFT attribute "device", which is the BCA address associated with the IoT device. Besides, an IoT device is not only a possession of an owner but also an active agent that obeys to a user to carry out certain tasks in an application. Hence, we propose the addition of the NFT attribute "user", which is the BCA address of the user of the IoT device. In order to represent the secure dynamic activity of the IoT device, six more attributes are proposed: "timestamp", "timeout", "state", "hashK_OD", "hashK_UD", and "dataEngage".  The attribute "timestamp" registers in the blockchain whenever the device checks it is bound to its token. The attribute "timeout" is the maximum delay time established for the device to prove again the bounding. Token "state" is included to trace the operating mode of the IoT device while "hashK_OD", "hashK_UD", and "dataEngage" are public data added to trace if secure communication channels are established between the device and its owner and user.

## Specification
 
This kind of non-fungible token is designed to represent a secure device with the capacity of generating its own BCA and, hence, interacting with the blockchain. Besides, this SmartNFT can assign the token to a user and establish secure communication channels with the owner and user. SmartNFTs can be considered as improved ERC721 tokens.
 
In order to avoid device counterfeiting, it is important that the private key of the device from which its BCA is derived could not be known by anyone except for the device itself. This can be achieved by using a PUF or a trusted module like a TPM. Only the device manufacturer can generate a new SmartNFT linked to the identity of the device. The manufacturer is in charge of compiling an providing the trusted firmmware to generate the device BCA. After executing a secure boot, the device can upload the owner and user firmwares. Owner and user firmwares cannot extract imformation from the device private key because they cannot overwrite the bootloader layer described by the manufacturer. To ensure this, the manufacturer firmware can be signed and the signature can be checked in a secure boot process. The owner is in charge of assigning a user to the token (the owner itself can be assigned as user). Only a user can use the token.
 
In order to complete a token transaction, a new owner must carry out a mutual authentication process offchain with the device and onchain with the token, by using their blockchain accounts. Similarly, a new user must carry out a mutual authentication process. SmartNFTs define how the authentication processes start and finish. These autentication processes allow deriving fresh session cryptographic keys for secure communication between devices and owners, and between devices and users. Therefore, the trustworthiness of the devices can be traced even if new owners and users manage them. These processes are described below.
 
![Figure 1 : State diagram of the token](/Images/Figure1.png)

An IoT device is a dynamic asset with several operating modes, which are represented by states in SmartNFTs. In particular, the states defining if the device has been engaged or not with an owner or with a user deserve attention because the software to be executed by the device, which should be verified prior to be executed, is different. Hence, we propose the addition of the SmartNFT attribute "state" and propose that the operating mode of the device should be in correspondence with its token state. We consider four main states of the token. The state "Waiting for owner" defines the situation whenever the token is created or transferred to a new owner but device and owner have not verified mutually yet. Once they verify each other, the device recognizes its owner and the state of its token changes to "Engaged with owner". In this state, the owner can transfer the token to a user. If the token is transferred to a user that coincides with the owner, the token state changes to "Engaged with user" and the device is ready to operate in its application, obeying the commands from its user. Otherwise, the token changes its state to "Waiting for user". Once the user and the IoT device are verified mutually, the token changes its state to "Engaged with user". From this state, the token can be transferred to another user, thus returning to "Waiting for user" state. To cope with a user that could not reply, the token can be transferred to another user if its state is "Waiting for user". From all the states, the token can be transferred to another owner, thus returning to "Waiting for owner" state. Figure 1 illustrates the state diagram of the token. The state changes are controlled by token functions as explained below.

| Type | Name of variable | Defined by ERC-721 | Optional |
|--|--|--|--|
| uint256 | tokenId | Yes | No |
| address | owner | Yes | No |
| address | approved | Yes | Yes |
| address | approvedForAll | Yes | Yes |
| address | device | No | Yes* ¿No sería No?¿Qué significa el *?|
| address | user | No | No |
| enum | state | No | No |
| uint256 | hashK_OD | No | Yes |
| uint256 | HashK_UD | No | Yes |
| uint256 | dataEngagement | No | Yes |
| uint256 | timeStamp | No | No |
| uint256 | timeout | No | No |
 
The atributes do not need to be given in an specific order. The designer of the smart contract defines the order. Like in the ERC721, the role of operator (aprovedForAll) is an attribute more related to the owner than to the token. 
 
```solidity
 pragma solidity ^0.8.0;

 /// @title SmartNFT: Extension of ERC-721 Non-Fungible Token Standard. 
 interface SmartNFT is ERC721{
    /// @dev This emits when the NFT is assigned as utility of a new user.
    ///  This event emits when the user of the token changes.
    ///  (`_addressUser` == 0) when no user is assigned.
    event UserAssigned(uint256 indexed tokenID, address indexed _addressUser);
    
    /// @dev This emits when user and device finish mutual authentication process successfully.
    ///  This event emits when both the user and the device prove they share a secret communication channel.
    event UserEngaged(uint256 indexed tokenID);
    
    /// @dev This emits when owner and device finish mutual authentication process successfully.
    ///  This event emits when both the owner and the device prove they share a secret communication channel.
    event OwnerEngaged(uint256 indexed tokenID);
    
    /// @dev This emits when the timeout expires.
    ///  This event emits when the timestamp of the SmartNFT is not updated in timeout.
    event TimeoutAlarm(uint256 indexed tokenID);

    /// @notice This function defines how the NFT is assigned as utility of a new user.
    /// @dev Only the owner of the SmartNFT can assign a user provided that the state of the token is 
    /// "engagedWithOwner","waitingForUser" or "engagedWithUser". The function changes the state of the token defined by "_tokenID" to
    /// "waitingForUser" and sets the parameter "addressUser" to "_addressUser".
    /// @param _tokenId is the tokenID of the SmartNFT bound to the device.
    /// @param _addressUser is the address of the new user.
    function setUser(uint256 _tokenId, address _addressUser) external; 

    /// @notice This function defines the initialization of the mutual authentication process between the owner and the device.
    /// @dev Only the owner of the token can start this authentication process provided that the state of the token is
    /// "waitingForOwner". The function does not change the state of the token and saves "_dataEngage" and "_hashK_O"
    /// in the parameters of the token.
    /// @param _tokenId is the tokenID of the SmartNFT bound to the device.
    /// @param _dataEngage defines the public data needed for the agreement of the shared key.
    /// @param _hashK_O is the hash of shared between owner and device generated by owner
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
This interface is an extension of the ERC721, is compatible with the standard, and needs the ERC721 interface. ERC721 interface is available in https://eips.ethereum.org/EIPS/eip-721. Then, metadata and enumeration extensions are compatible and included in this draft.
 
## Rationale
This SmartNFT was developed because the ERC721 standard does not take into account the users of an asset, only the ownership. In addition we decided implement a secure way to share a secret key confirmed with the consensus of the blockchain. In this way, device and owner, or device and user can be sure thar they are exchange information only with its interested.

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
Regarding the security improvements implemented with respect to the ERC-721 standard, without counting the previous mentions, a user management system is established, so the use of an asset or device does not affect the token's ownership.


## Copyright
Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
