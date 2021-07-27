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
Current NFTs are designed like pasive assets, and this assets only have an owner, or in the best of case a manager for an owner. In the Smart Contract literature apear a lot of projects about renting, or managment system of tokens. In this proposal we introduce the role of user and the role of device. The device must create its own identity to know its state, its user and its owner. Owner can not lose the ownership when let use the device to an user, and users can use the device without obtain the ownership of a device, only a user permition. In addition the device known in each moment who can use it. This proposal make easier a lots of projects using Smart Non Fungible token binded to a Smart Device.

## Specification
This kind of non-fungible token must be associated to a secure device that need to have the capacity to generate its own blockchain account, and safe it on a secure memory or using any secure algorithm to save it like SRAM PUF or similar. And only the manufacturer can create the token. This is for secure purpose. And the manufacturer is the responsible of the smart contract. Its mean that each manufacturer need to be its own smart contract to interact with its own tokens. The manufactuter define de bootloader that define the account saved on the device. 
Owner have de capacity of transfer and update the firmware of the tokens, but owner can not overwrite the bootloader layer described to the manufacturer. To do this, the firmware must be sign by manufacturer in a secure bootloader process. Owner assign a user to use the token, if a owner need to use the token need to assign itself like user. Depending of how is described the smart contract an owner can decline the use of the token of an user.
Only a user can use the token, the user can define some parameters of token but never upload any firmware. This firmware is defined by the manufacturer or the owner.
To complete a token transaction the new owner must authenticate offchain and onchain whith the token using their blockchain account. The smart contract define how owner start a new authentication and how the token can complete this autentication. This autentication is necesary to the device know who is its owner and stay secure that is not an impostor. In the same way, the smart contract define how the user and device must be authenticate in the same way.
 
the IoT device can interact actively with other blockchain participants, that is, it can have a BCA. Hence, we propose the addition of the NFT attribute device, which is the BCA ad-dress associated with the IoT device. Besides, an IoT device is not only a possession of an owner but also an active agent that obeys to a user to carry out certain tasks in an applica-tion. Hence, we propose the addition of the NFT attribute user, which is the BCA address of the user of the IoT device. In order to represent the dynamic activity of the IoT device, six more attributes are proposed: timestamp, timeout, state, hashK_OD, hashK_UD, and da-taEngage.  The first of them, timestamp, registers in the blockchain whenever the device checks it is bound with its token. This is very important to register if the bound is alive or not. timeout define how many time is the maximum time to update timestamp from device to be considered still on live.
 
An IoT device is a dynamic asset that can change of operating modes, which can be represented by states. In particular, the states defining if the device has been engaged or not with an owner or with a user deserve attention because the software to be executed by the device, which should be verified prior to be executed, is different. Hence, we propose the addition of the NFT attribute state and propose that the operating mode of the device should be in correspondence with its token state. We consider four main states of the to-ken. The state Waiting for owner defines the situation whenever the token is created or transferred to a new owner but device and owner have not verified mutually yet. Once they verify each other, the device recognizes its owner and the state of its token changes to Engaged with owner. In this state, the owner can transfer the token to a user. If the token is transferred to a user that coincides with the owner, the token state changes to Engaged with user and the device is ready to operate in its application, obeying the commands from its user. Otherwise, the token changes its state to Waiting for user. Once the user and the IoT device are verified mutually, the token changes its state to Engaged with user. From this state, the token can be transferred to another user, thus returning to Waiting for user state. To cope with a user that could not reply, the token can be transferred to another user if its state is Waiting for user. From all the states, the token can be transferred to another owner, thus returning to Waiting for owner state. Figure 1 illustrates the state diagram of the token. The state changes are controlled by token functions as explained in the following subsection.
 
![State diagram of the token](/Images/Figure1.png)
 
The engagements of the device with an owner and with a user are carried out after mutual authentication protocols based on elliptic curve Diffie-Hellman key exchange protocols. These protocols allow a key agreement between the device and its owner, in the one side, and the device and its user, in the other side. Since the establishment of a shared secret is very important for a secure communication between them, we propose the inclu-sion of the attributes hashK_OD, hashK_UD, and dataEngage. The first two attributes define, respectively, the hash of the secret shared between the device and its owner and between the device and its user. Devices, owners, and users should check they are using the correct shared secrets. The attribute dataEngage defines the public data needed for the agreement. If the mutual authentication fails, dataEngage allows detecting which parts failed.
This table shows the attributes of Smart NFTs. The standard attributes approved and operator (which help the owner to transfer ERC-721 NFTs to other owners) are omitted in the table because they are the same that in the ERC-721. Of course, they can be considered also in the proposed SmartNFT.

| Type | Name of variable | Defined by ERC-721 |
|--|--|--|
| uint256 | tokenId | Yes |
| address | owner | Yes |
| address | device | No |
| address | user | No |
| enum | state | No |
| uint256 | hashK_OD | No |
| uint256 | HashK_UD | No |
| uint256 | owner | No |
| uint256 | owner | No |
| uint256 | owner | No |
 
 
 
## Rationale
The rationale fleshes out the specification by describing what motivated the design and why particular design decisions were made. It should describe alternate designs that were considered and related work, e.g. how the feature is supported in other languages.

## Backwards Compatibility
All EIPs that introduce backwards incompatibilities must include a section describing these incompatibilities and their severity. The EIP must explain how the author proposes to deal with these incompatibilities. EIP submissions without a sufficient backwards compatibility treatise may be rejected outright.

## Test Cases
Test cases for an implementation are mandatory for EIPs that are affecting consensus changes. Other EIPs can choose to include links to test cases if applicable.

## Reference Implementation
An optional section that contains a reference/example implementation that people can use to assist in understanding or implementing this specification.  If the implementation is too large to reasonably be included inline, then consider adding it as one or more files in `../assets/eip-####/`.

## Security Considerations
All EIPs must contain a section that discusses the security implications/considerations relevant to the proposed change. Include information that might be important for security discussions, surfaces risks and can be used throughout the life cycle of the proposal. E.g. include security-relevant design decisions, concerns, important discussions, implementation-specific guidance and pitfalls, an outline of threats and risks and how they are being addressed. EIP submissions missing the "Security Considerations" section will be rejected. An EIP cannot proceed to status "Final" without a Security Considerations discussion deemed sufficient by the reviewers.

## Copyright
Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
