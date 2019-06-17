# Emtrust Wai DID Method Specification
V0.1, Sunil Prakash, Halialabs
## Introduction
EmTrust WAI distributed identity system  provides a permissioned , safe , secure, and transparent implementation of the self soverient ID and Verifiable Claims specifications published by the W3C and the Decentralised Identity Foundation. It utilizes Hyperledger Fabric as the blockchain and decentralized object store for document sharing.
### Hyperledger Fabric
Hyperledger Fabric is a platform for distributed ledger solutions underpinned by a modular architecture delivering high degrees of confidentiality, resiliency, flexibility, and scalability. It is designed to support pluggable implementations of different components and accommodate the complexity and intricacies that exist across the economic ecosystem.

Fabric is the most popular and smartly designed DLT implementation which is closely suited to most of the Enterprise where privacy and security is at high priority. It provides permissioned based blockchain architecture with high scalability for transactions.

### Decentralized Object store
Halialabs Decentralized object store implementation utilizes IPFS as it core, and adding encryption and security layers for confining the sharing of data to only controlled nodes.

## Overview
The Emtrust DID method utilizes Hyperledger fabric as the DLT implementation, having an identity channel which is shared among the identity nodes with participating organizations. The DID document along with metadata of third party endorsements resides on ledger and the private information of users are kept on the mobile or persona devices which never leaves the device.
The Interaction of DID and blockchain ledger happens via the API servers hosted by any participating organizations. 
## Specification
### Method DID Format
Emtrust DIDs are represented by their did\:emtrust: method string and conform to the [Generic DID Scheme](https://w3c-ccg.github.io/did-spec/#the-generic-did-scheme).


A valid Emtrust DID would look like: 

```
did:emtrust:0xdadac9f39033a7205b28849cc1dae698d5ceac18

```

### CRUD Operation Definitions

#### Create (Registration)

The DID document is created with following steps

1. System Application or User Wallet App generates 32bit random phrases as entropy
2. Using entropy the key pair is generated using one of the method listed [here](https://w3c-ccg.github.io/ld-cryptosuite-registry/) 
3. The public key is doubled hash and shortened to generate 40 charater unique id
4. Few more user metata is captured at time of creation and hashed and signed with private key for enrolment purpose.
5. A Service provider (DID participating organization) is contacted to enroll the ID
6. Service Provider registeres the ID on blockchain with registration metadata and organizations singature.

The hash from step 3 is the DID.

The minimum DID document for an Emtrust ID without any service and no transactions would look like this:

```
{
  "@context": "https://w3id.org/did/v1",
  "id": "did:emtrust:0xdadac9f39033a7205b28849cc1dae698d5ceac18",
  "publicKey": [{
      "id": "did:emtrust:0xdadac9f39033a7205b28849cc1dae698d5ceac18#unq1"
      "type": "ED25519SignatureVerification",
      "publicKeyBase58": "0441baa17a778380a5a5ee9f1b3ff8cda5be12915eace85fd91c9daba7a7023f0955548e28c6a4ff5614ebe64aa95740be23599c16b7dfed484313f4a5b43649b2",
      "authorizations": []
    }],
  "authentication": [{
      "type": "ED25519SigningAuthentication",
      "publicKey": "did:emtrust:0xdadac9f39033a7205b28849cc1dae698d5ceac18#unq1"
    }]
}

```
The document once generated need not to be connected to any network. After generation, for registration steps , a service provider API (`https://<service-provider>/enroll`)need to be accessed ( for eg: https://identity.emtrust.io/enroll) to submit and record the DID on the ledger.

#### Read (Resolve)

Entrust DID Document are reolved by querying the smart contract running on blockchain for existance and the respective DID document is fetched and served to the verifier. The verifier usage one of the service provider hosted API server to query the blockchain.

The DID will always have primary public address with the fragment `#unq1` appended.
```
  {
      "id": "did:emtrust:0xdadac9f39033a7205b28849cc1dae698d5ceac18#unq1"
      "type": "ED25519SignatureVerification",
      "publicKeyBase58": "0441baa17a778380a5a5ee9f1b3ff8cda5be12915eace85fd91c9daba7a7023f0955548e28c6a4ff5614ebe64aa95740be23599c16b7dfed484313f4a5b43649b2",
      ...
  }
```
The other public keys with fragements numbered with `unq` would represent alternate identification keys.

The DID document can also have id with `#sub` prefix appended as fragments, which would represents the secondary or delegated identity representation.

```
  {
      "id": "did:emtrust:0xdadac9f39033a7205b28849cc1dae698d5ceac18#sub-1"
       ...
  }
```


#### Update (Modification)
As the documents are saved on permissioned blockchain, the roles for each DID document is assigned by the moderator for the organization ( or auto role assigned based on configured rules). With Write permissions , users are able to submit the changes in did document.

#### Delete (Revoke)

As there is no centralised control of the registry the DID documents could not be deleted. However, Service providers can add assertions on ledger for DID to grant or revoke respective services.

A user deactivation request by the owner of the id could be submitted by adding "intent" attribute to the transaction and calling api for service provider with url `https://<service-provider>/revoke`.

For example, 
```
{
  "@context": "https://w3id.org/did/v1",
  "id": "did:emtrust:0xdadac9f39033a7205b28849cc1dae698d5ceac18",
  "publicKey": [{
      "id": "did:emtrust:0xdadac9f39033a7205b28849cc1dae698d5ceac18#unq1"
      "type": "ED25519SignatureVerification",
      ...
    }],
  "authentication": [{
      "type": "ED25519SigningAuthentication",
      "publicKey": "did:emtrust:0xdadac9f39033a7205b28849cc1dae698d5ceac18#unq1"
    }],
  "intent:revoke":{
    "id":"did:emtrust:0xdadac9f39033a7205b28849cc1dae698d5ceac18",
    "remarks":"No longer needed"
  }
}
```

This would assert the DID doument on ledger and will no loger be eligible for adding any new services.


## Key Management

### Key Storage
The private keys are always stored on users personal device (mobile/ipad) and never leaves the device for any events.

### Key Recovery
As the possibility of loosing the private keys are significant, the Seed phrase generated at time of id creation is used to backup and store in safe storage by user. The Seed phrase is used to regenerate the key pairs and re-enroll with the service provider. The keys generated are identical to previous, but the re-enrolment phase records the key recovery event on the blockchain for transparency.

### Key Revocation
With no presence of centralized control of revocation, the DID document always recides on blockchain, but an assertion is made by service provider in case of revocation request which could be used for controlling the permissions for 3rd party services access.


## Secutity Considerations

- The generated seed pharse which is used to generate private key **MUST** be stored on the client-side (user's private device).
- Utilizing EmTrust Object store for document storage insteads of plain IPFS as to control the readability and share access.
- API access to the Service provider **MUST** be in https and client app should be able to switch between service providers.

## Reference

- [DID Method Registry](https://w3c-ccg.github.io/did-method-registry/)
