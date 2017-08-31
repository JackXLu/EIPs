## Preamble

    EIP: 
    Title: Transferring ethers with privacy protection using precompiled contracts for ring signature verification.
    Author: Jack Lu <jack@wanchain.org>, Demmon Guo <demmon@wanchain.org> and Shi Liu <shi@wanchain.org>
    Type: Standard Track
    Category (*only required for Standard Track): Core 
    Status: Draft
    Created: 2017-08-21
    


## Simple Summary
A mechanism is designed to provide privacy protection for transferring ethers using one-time account and precompiled contracts for linkable ring signature verification.

## Abstract
This EIP suggests implementing a special precompiled contract for ring signature verification and adding one-time account mechanism on Ethereum. This can in turn be combined with *#eip-draft\_Precompiled contracts for ring signature verification on the elliptic curve alt\_bn128.* to provide privacy protection for ether transactions without affecting the in-protocol mechanisms. In each transaction, a one-time account is created for the receiver, of which no one can figure out the very owner without the corresponding scan key. Ring signature is verified by precompiled contracts and helps the receiver transfer ethers to his/her main account from one-time account, which is hidden in a set of one-time accounts to keep untraceable.

## Motivation
Currently, ether transactions on Ethereum are fully transparent, which makes them unsuitable for several use-cases where users’ privacy is of concern and the link between sender and receiver should not be retrieved by anyone else. #196 and #197 suggest adding precompiled contracts for addition and scalar multiplication on a specific pairing-friendly elliptic curve to verify zkSNARKs in Ethereum smart contract, thus increasing the privacy for users. But it has high computation complexity. Monero solves this problem by using Cryptonote under UTXO model. However, this technology cannot be directly used in Ethereum under account model. Considering all of the above, this EIP forgoes zero knowledge proof and proposes to implement a special precompiled contract for ring signature verification and add one-time account mechanism on Ethereum to provide privacy protection for ether transactions. Note that, with this smart contract, users can choose regular transactions or transactions with privacy protection depending on their needs for privacy and anonymity.

## Specification
We implement a smart contract called *EtherSC* for managing tokens where the dispersed value of the token money corresponds in a 1:1 ratio with ether. It is precompiled for ring signature verification and achieves privacy protection for transferring ethers by providing two functions for users: token purchase and token withdraw. The token purchase function allows user to transfer ethers to the contract and receive an equal value of tokens into the one-time account. On the other side, user may withdraw ethers from one-time account to the main account by invoking the token withdraw function using ring signature of the one-time account.  

**One-time account mechanism**  
  
We extend the original account structure to support generating and checking one-time account. It is similar to Monero’s solution:  
Main account: a pair *((A,a),(B,b))* of different ECDSA private keys, where *(A,B)* is the public key and  *(a,b)* is the private key.  
One-time account: a pair *(X,Y)* of different ECDSA public keys. It is generated by the sender.  
Scan key: a pair *(A,b)* of public and private keys. It is used to verify whether a one-time account belongs to main account *((A,a),(B,b))*.  

**Transaction scenario**  

Supposing the transaction scenario is as follows:  
The private key of sender’s main account is set *(a,b)* and the corresponding public key is set *(A,B)*. The private key of receiver’s main account is set *(c,d)*，the corresponding public key is set *(C,D)* and scan key is *(C,d)*. Sender transfers ethers of amount *v* to receiver.   

**Initiate Transaction**  

Sender first generates a one-time account *ota2* for receiver by using the public key *(C,D)*. Then sender invokes the token purchase function of smart contract *EtherSC* using the one-time account as its parameter:   

    Tx=(sender_address,EtherSC,value,payload,sig)
    payload=("purchase",ota2)  

**Confirm Transaction Initiation**  

The Validator verifies the relationship between *sig* and *sender_address*. If *sig* is valid, the smart contract is invoked and *ota2* is created and stored in the account list of the corresponding value in the contract. At the same time, ethers of value *v* are deducted from sender’s main account.  

**Receive Transaction**  

Receiver scans the storage list of  *EtherSC*  using the scan key. Once receiver finds that *ota2* belongs to him and will select n-1 one-time account in the corresponding face value list to generate a ring signature of *ota2*. This ring signature is used as the parameter for the token withdraw function:   

	Tx=(receiver_address,EtherSC,payload,sig)
	payload=("withdraw",OTASet,ringsig)  

**Confirm Transaction Received**  
 
The Validator verifies the relationship between *sig* and *receiver_address*. If *sig* is valid, the smart contract is invoked to verify the validity of the ring signature and makes sure that key image *I* in *ringsig* did not exist before. After the confirmation, ethers with corresponding value are transferred to receiver’s main account.

 

## Rationale
Implementing a precompiled smart contract for ring signature and adding one-time account mechanism on Ethereum to provide privacy protection for ether transactions is because of its accessibility, which is inspired by #86 that provides an idea of “abstracting out” signature verification to allow users to perform any desired signature check. Based on *#eip-draft\_Precompiled contracts for ring signature verification on the elliptic curve alt\_bn128.*, this smart contract is easy to implement and will not change the in-protocol mechanism. Besides, zero knowledge proof is not adopted because it is too complicated and expensive on Ethereum. As a result, this simple smart contract can provide privacy for ether transactions smoothly and efficiently without modifying many of the existing mechanisms of Ethereum.  
Note that, in this smart contract the trace between sender and receiver is cut off completely. Meanwhile, there is a limit that the transferred value must be one of the several amounts determined in the contract. Arbitrary value is not supported.


## Backwards Compatibility
Besides the changes in *#eip-draft\_Precompiled contracts for ring signature verification on the elliptic curve alt\_bn128.*, no extra changes should be made. Deploying this smart contract is all we need to do.

## Test Cases
To be written.

## Implementation
To be written.

## Copyright
License:   
Copyright and related rights waived via CC0.
