# Distributed Storage Protocol Version 1(DSP-1) Draft
> This document is a proposal of DSP which initialed by DSP Labs and it is still draft.We hope it would represent the consensus of the DSP community and would updated continously by the proposals of public. 

## Introduction

The DSP(Distributed Storage Protocol)protocol is a new-generation Internet protocol paradigm based on data file encryption, distribution, storage, sharing and other dimensions. The aim of the DSP protocol is to become the core infrastructure of NGI(Next Generation Internet).

# Table of Contents

- [DSP Protocol Overview](#dsp-protocol-overview)
  - [Document Organization](#document-organization)
  - [Terminology](#terminology)
- [Account](#account)
  - [Create](#create)
  - [Digital Signature](#digital-signature)
- [Storage Service](#storage-service)
  - [Block Management](#block-management)
  - [FS API](#fs-api)
  - [FS Message Format](#fs-message-format)
  - [Storage Process](#storage-process)
  - [Prove of Data Possession](#prove-of-data-possession)
  - [Replicator and Recovery](#replicator-and-recovery)
  - [FS Block Sharing](#fs-block-sharing)
  - [Garbage Collection](#garbage-collection)
- [Sharing Service](#sharing-service)
  - [Data Discovery](#data-discovery)
    - [DNS Registry](#dns-registry)
    - [SCAN Discovery](#scan-discovery)
  - [Block Sharing](#block-sharing)
  - [Sharing Message Format](#sharing-message-format)
- [Proxying](#proxying)
  - [Proxying Message Format](#proxying-message-format)
  - [Proxying Connection](#proxying-connection)
  - [Data Exchange](#data-exchange)
- [P2P Layer](#p2p-layer)
  - [Component](#component)
  - [P2P Interface](#p2p-interface)
  - [Message](#message)
    - [Message Format](#message-format)
    - [Serialization and Deserialization](#serialization-and-deserialization)
    - [Payload Format](#payload-format)
    - [Message Compression](#message-compression)
  - [Connection](#connection)
    - [Connection Establishment](#connection-establishment)
    - [Disconnection](#disconnection)
    - [Stream](#stream)
- [Status Code Definitions](#status-code-definitions)
  - [Task Error Code](#task-error-code)
  - [Transmission Status Code](#transmission-status-code)
  - [RESTful API Error Code](#restful-api-error-code)
- [Contributing](#contributing)

## DSP Protocol Overview
This specification defines the DSP protocol stack,the interfaces, and how it all fits together. DSP aims to be more efficient,safe,decentralized and trustless than current Internet.  

The basic storage unit in DSP is a block. All services running in DSP based on the management of blocks. For example, trustd data block storage, exchange under privary control etc.

The core protocol in DSP is account and crypto architecture, which use crypto algorithm in data file encryption, distribution, storage, sharing and other dimensions, protect the transation trusted and data value.

Storage Service ensure that it is possible to efficiently keep data safely and exchangeable. PDP wih proved random draw algorithm support the prove of space and time in services.

Sharing Service ensure the block transmitted between peers and the source of data could get its incentive.

Proxying Service ensure that who don`t have public address can be accessed by others, and support block exchanging between peers. 

In order to support peer-to-peer communication efficiently, DSP contain a networking layer design which would help building a more robust P2P network.

### Document Organization
The protocol document is split into eight parts:

1. Account and Crypto Architecture: crypto account management, pledge / unlock, import and export.
2. Storage Service:including file upload storage, copy between nodes, shared data, delete and other operations.
3. Sharing Service:Definition of how sharing service start, and block distributing.
4. Proxying: Assign a proxy address to the request host. Forward data to the request host, and the data from requst host will be forwarded to relative node.
5. P2P Layers:the underlying communication module for the above services.
6. Status Code Definitions

The lastest updates of protocol implementation in detail wound be found in [DSP-Labs](https://github.com/DSP-Labs/specs/) 

### Terminology
All numeric values are in network byte order. Values are unsigned
unless otherwise indicated.Literal values are provided in decimal
or hexadecimal as appropriate.Hexadecimal literals are prefixed
with "0x" to distinguish them from decimal literals.

The following terms are used:

**client**:  The endpoint that retrieval in ledger, send transaction, upload or download data file.

**ledger** Distributed Ledger which record the transactions and crypto assets.

**connection**:  A transport-layer connection between two endpoints.

**endpoint**:  Either the client or server of the connection.

**block**:  The basic unit in storage, transmition, encrption and other service.

**peer**:  An endpoint.When discussing a particular endpoint, "peer"
      refers to the endpoint that is remote to the primary subject of
      discussion.

**server**:  The endpoint that accepts connection define in DSP and support service defined in DSP.

**FS**:  An server that support storage service.

**SCAN**:  An server that support data retrieval service

**DNS**: Distributed Name Service be recorded in ledger.

**Proxy**: An server that help exchange of block data between two endpoint.

**ledger**: A ledger help store relative information of service or file information, ensure account balance or validation of storage. 

## Account
All exchanged message in DSP should be signed with endpoint`s private key. That is the core arthitecture of DSP privace and trustless mechanism. The crypto account is the only valid representative of a endpoint.

### Create
After choose a crypto schema, generates a pair of private and public keys.Then endpoint can create a account address as the representative of account.

```
address_code = byteof publickey + 0xAC
address_hash =  sha256 of address_code
address_ripemd = ripemd160 of address_hash
address_string = byte{0x17} + address_ripemd
suffix_hash  =sha256(sha256(address_string)
address_string = address_string + suffix_hash[0:4]
adderss = base58(address_string)
```
The string of account created in 7 steps:
1. byte of public key add byte 0xAC 
2. the calculate the sha256 hash of address_code
3. calculate ripemd160 byte of address_hash
4. add byte 0x17 at the beginning of address_ripemd
5. do twice sha256 hash of address_string
6. first 4 bytes of suffix_hash add to the end of address_string
7. base58 create the string of address


### Digital Signature
Endpoint in DSP should use crypto private key to create account. At lease one signature mode listed below should be supported in protocol.

- SHA224withECDSA
- SHA256withECDSA
- SHA384withECDSA
- SHA512withECDSA
- SHA3-224withECDSA
- SHA3-256withECDSA
- RIPEMD160withECDSA
- SHA512withEdDSA

**The support scheme will be increased by the proposals of public.**
## Storage Service
Storage Service is the core service in DSP protocol, it is the fundamental of decentralized storage framework.

In Storage service, the endpoint which named FS is the underlying module to provide reliable file storage and efficient file management. 

In the DSP ecosystem FS support a variety of scenarios like file uploading, file downloading, and also file sharing.

The latest storage service document could be found in [FS](https://github.com/DSP-Labs/specs/blob/master/FS.md/)


### Block Management
- Block

The basic unit in FS is the Block.

A hash-based Block Identifier could be calculated from the Block data, we call it Block Hash.

A Block can either be a data block whose main purpose is to store data or a non-data block who has no data stored but contains Block Hashes as links to other Blocks.

With a given Block and its Block Hash, it can be easily verified if the Block data has been corrupted. In this way Blocks can be distributed within the DSP with guarantee it's unchanged.

- File

A File should be chunked into Blocks with defined block size, and it can be saved in local storage with support of different underlying storage types ranging from flat files to KV databases like LevelDB.

The result of file chunking is a tree structure with a Root Block. The leaves of the tree contains the data of the file, and Root Block or other intermediate Blocks contains links to other Blocks.

The chunking method is definitive so the Root Block Hash can be used to identify a specific file, we call it File Hash.

The tree structrue of File should stored on the ledger for the validation of PDP.

### FS API
Storage Server should support basic FS APIs, which is listed below:

- AddFile

Add a file to FS after chunking into Blocks.

- DeleteFile

Delete a file from FS.

- GetBlockHashesOfFile

Returns all Blocks hashes of a file with a given File Hash.

- PutBlock

Store a Block in the FS.

- GetBlock

Get Block with given Hash from FS.

- GarbageCollect

Manually trigger the garbage collector to remove all Blocks those have no reference/link to them.

### FS Message Format
- FS Information
```
type NodeInfo struct {
	HostAddr   string
	Account string
}
```
- File Information

```
type FileInfo struct {
	FileHash       []byte
	FileOwner      common.Address
	FileDesc       []byte
	Privilege      uint64
	FileBlockNum   uint64
	FileBlockSize  uint64
	ProveInterval  uint64
	ProveTimes     uint64
	ExpiredHeight  uint64
	CopyNum        uint64
	Deposit        uint64
	FileProveParam []byte
	ProveBlockNum  uint64
	BlockHeight    uint64 // store file info block height
	ValidFlag      bool
	StorageType    uint64
	RealFileSize   uint64
	PrimaryNodes   NodeList // Nodes store file
	CandidateNodes NodeList // Nodes backup file
	BlocksRoot     []byte
}

```
- Upload Option record on ledger

```
type UploadOption struct {
	FileDesc        []byte
	FileSize        uint64
	ProveInterval   uint64
	ExpiredHeight   uint64
	Privilege       uint64
	CopyNum         uint64
	Encrypt         bool
	EncryptPassword []byte
	RegisterDNS     bool
	BindDNS         bool
	DnsURL          []byte
	WhiteList       WhiteList
	Share           bool
	StorageType     uint64
}
```
more formats can be found in [specs](https://github.com/DSP-Labs/specs)

### Storage Process

1. FS should record it information on ledger ensure remote endpoint find it.
2. Remote endpoint chunked it file into Blocks and submit the file hash tree on ledger, then connect to FS.
3. After exchange the information of to be saved file, remote endpoint transmit blocks to FS.
4. FS should submit first PDP on ledger to declare it service has start when all blocks received.
5. FS should submit PDP according to the requirements of storage task setup on ledger.

### Prove of Data Possession

The FS must submit PDP validation to public ledger to prove the safe and integrity of its data.

The PDP should be submited when remote peers initiate a PDP challege or periodically.

The PDP should contain the information from data, not the raw data, it will keep the privacy of who save or use the data. A suggested PDP logic as following

1. A public challege should be declare on ledger or can be proved calculated from public information.
2. According to challege and setup PDP algorithm, FS should start calculate the prove of specific blocks.
3. FS submit prove on ledger on time and the prove is validated, incentive should record for FS.
4. If FS can not submit the prove on time or prove is invalid, FS should be punished, Recovery should started on other FS server.

### Replicator and Recovery
FS should support replicator, that means data would store in more than one FS servers

BLOCKS of to be saved file duplicated to several copies and transmit to different FS address.

When download file, client of FS would get blocks from all servers who have the blocks of file.

When FS can not submit the PDP correctly, other FS should start Recovery, Recovery would duplicated the file from other replica, after new FS submit the PDP of file, the ledger should record it as the storage supporter and shift the punishment of broken FS to new one.

### FS Block Sharing

As mentioned above, a File in the FS is organized as a tree with links to Blocks. It is possible that different files could have some Blocks in common. In the FS, there is only one copy of the Block is saved in storage, and the Block is shared by these files.

### Garbage Collection

An Mark-and-Sweep garbage collection mechanism should be used to delete a Block that has no reference/link to it. When a file is deleted, its Root Block Hash will be dereferenced and all the Blocks that no longer referenced will be deleted in next garbage collection.

Garbage collection is costly since it needs to check all the reference/link for all Blocks. It should be performed with caution.

## Sharing Service
All roles in DSP protocol could share blocks to others.

Peers could find the block-address pair from SCAN server. After connect to specific address, peers can download blocks they need.

The latest Sharing service information could be found in [SDK](https://github.com/DSP-Labs/specs/blob/master/SDK.md)

### Data Discovery
Blocks in different endpoint will record the hash-address pair information on SCAN server,and endpoint could search the block they needed on SCAN discovery service.

Ledger should record a hash-url list to help data sharing via p2p network.

More DNS information can be found in [DNS](https://github.com/DSP-Labs/specs/blob/master/DNS.md)

#### DNS Registry
DNS(Distributed Name Service) is a distributed and extensible domain name mapping service. Gradually change the data storage paths and extension protocols used by the system to words and abbreviations.

Map specific service entities such as addresses, paths, id, content, etc.  

- DNS format

```
dsp://urlformultiservices
```
- *dsp*:header

- *urlformultiservices*:sharing url

- Naming

```
Header：     protocol
URL：        link
Name ：      source path
Owner：      owner
Description：name description
BlockHeight：create height
TTL:         Expiration date
```
##### Basic Function

 1. Registration service
 2. Management Service
 3. Inquiry service

###### Registration service
The registration service is responsible for generating connections according to different registration methods, while storing the name owner, TTL and other content.
Interfaces:

**url registration**
1. Default mode: Use the default protocol and generate url randomly
2. Custom protocol mode: Use custom protocol and generate url randomly
3. Custom url mode: Use the default protocol and attach a custom url
4. Naming mode: both protocol and url are customized

**header registration**
 - Register a new header
 - Each registered header has an expiration date
 - Can't register system reservation header
 - Register the header mainly in the community governance mode

###### Management Service
-  Provide services for the transfer, update and deletion of url / header.
-  Management of url / header, which requires name creator or service management account permissions.

**Service mode**
 1. Transfer: transfer the ownership of the url / header, the account with ownership can update and delete the url / header
 2. Update: update url mapping object path
 3. Delete: delete the url / header. If a header is deleted or expires, all links named by the header cannot be resolved.

###### Inquiry service
Query the mapping information, owner and TTL stored during header / url registration

#### SCAN Discovery
SCAN is provides data discovery service in DSP protocol.

**Node Mechanism Principles**

1. Joining SCAN network requires mutual authentication, relying on cryptographic identity rather than IP or other identities as the authentication condition.
2. For the database update and message processing system of hundreds of nodes in the P2P environment in the authentication system, SCAN uses the same algorithm (if not, see principle 3) to execute and get the same result. Able to handle a certain flow peak.
3. Scan choose trusted peers for connection and synchronization within the authentication system. If there are spoofing and data errors, the challenger finds that the nodes will receive economic reduction penalties. 

###### Content addressing
1. Endpoint upload the block hash information according to DNS record and its address.
2. SCAN servers should synchronize the information with each other.
3. Endpoint send request to SCAN to search specific url or hash.
4. SCAN return the request answers to endpoint.
5. Endpoint connect to peers to download blocks of file.

######  Message Format
```
message Torrent {
    bytes infoHash = 1;
    uint64 left = 2;
    bytes peerinfo = 3;
    int32 type = 4;
}

message Endpoint {
    string walletAddr = 1;
    string hostPort = 2;
    int32 type = 3;
}
```

### Block Sharing
Blocks in FS or other endpoints can be shared according to the permissions recorded on ledger.

A list setup on ledger will declare which account could download the specific file blocks or which one coundn`t.

The blocks will builded as a merkle tree, then this hash tree information would be exchanged before sharing start. Endpoint used this hash tree to check the validation of blocks.


### Sharing Message Format
- File

handshake messga for sharing file

```
message File {
   string sessionId = 1;
   string hash = 2;  
   repeated string blockHashes = 3;
   int32 operation = 4;
   bytes prefix = 5;
   int32 chunkSize = 6;
   Tx tx = 8;
   Breakpoint breakpoint = 9;
   uint64 totalBlockCount = 10;
   Chain chainInfo = 11;
   string blocksRoot = 12;
}
```
- BlockFlights

batch transmit blocks between peers

```
message BlockFlights{
   int64 TimeStamp = 1;
   repeated Block blocks = 3;
}
```

- Progress

progress for sharing data

```
message Progress {
   string id = 1;
   string hash = 2;  
   int32 operation = 3;
   string sender = 4;
   repeated ProgressInfo infos = 5;
}
```


## Proxying
Proxying service help setup a mapping address for endpoint who don`t have public address, then remote peers can connect to specific address mapped by proxy ,and exchange data with a intranet endpoint.

The general proxying steps are as follows:

Firstly, send a proxy request message to the proxy service port; 

After internal processing, proxy will assign a proxy address to the request host;

When other hosts connect to the proxy address, the data will be forwarded to the request host and the data from requst host will be forwarded to relative peer.

The latest Proxying document could be found in [Proxying](https://github.com/DSP-Labs/specs/blob/master/PROXY.md)

### Proxying Message Format
- Proxy message

When endpoint want to apply for a PROXY IP, it must to send ProxyRequest to proxy server. If server receive the proxy request, proxy server allocate the IP:Port to node and send ProxyResponse Message back.

```
message ProxyRequest {
    string origin_address = 1;
    string proxy_address = 2;
}

message ProxyResponse {
    string origin_address = 1;
    string proxy_address = 2;
}
```

- Keepalive
In order to keep alive between peers and proxy server, keepalive message should be sent to server each 15s by default.
```
message Keepalive {
}

message KeepaliveResponse {
}
```

### Proxying Connection
Start proxying service, then listen on a special IP:PORT, which is known to all peers.

Endpoint send **ProxyRequest** message to the server. If Endpoint has been request before, the server will send back the allocated proxy ip:port, or porxy server will send back a new address pair.

Server sends back **ProxyResponse**  message, which contains the proxy IP:port allocated by server.

The endpoint will send **Keepalive** message to proxy server each 15s by default, and the server will send back **keepalive response** at the same time.

The allocated proxy IP:Port will be cached by proxy server for 180 min by default. That means, if a endpoint disconnect with server and reconnect between 180 min, it will get the same proxy ip:port as well.

### Data Exchange
endpoint1 establish connection with proxy server, named **Main-Connection-1**;

endpoint2 establish connection with proxy server, named **Main-Connection-2**;

Proxy server start a proxy server listening on allocated ip:port for endpoint1, named **Proxy-Server-1**;

Proxy server start a proxy server listening on allocated ip:port for endpoint2, named **Proxy-Server-2**;

When endpoint1 want to exchange message with endpoint2, endpoint1 need to establish a connection with **Proxy-Server-2** named **connection-3** and endpoint2 need to establish a connection with **Proxy-Server-1** named **connection-4**;

endpoint1 send message by **connection-3** and proxy will transfer the data received from **connection-3** to **Main-Connection-2**;

When endpoint2 send back message, it must use **connection-4** and **Main-Connection-1**;

## P2P Layer
P2P layer ensure the underlying communication more efficiently, robustly and expandable.

The latest P2P document could be found in [P2P Layer](https://github.com/DSP-Labs/specs/blob/master/P2P.md)

### Component
P2P layer in DSP should following component:
1. cryptographic primitives (Ed25519, AES-256,etc.),
2. message serialization/deserialization schemes (byte-order little endian, protobuf, msgpack etc.),
3. network timeout/error management (on dial, on receive message, on send buffer full),
4. network-level atomic operations (receive-then-lock),
5. and NAT traversal support(NAT-PMP, UPnP).

### P2P Interface
Application layer can define interface by themself,following is the default design:
```
type P2PInterface interface {
	// Callback for when the network starts listening for peers.
	Startup(net *Network)
	// Callback for when an incoming message is received. Return true
	// if the Component will intercept messages to be processed.
	Receive(ctx *ComponentContext) error
	// Callback for when the network stops listening for peers.
	Cleanup(net *Network)
	// Callback for when a peer connects to the network.
	PeerConnect(client *PeerClient)
	// Callback for when a peer disconnects from the network.
	PeerDisconnect(client *PeerClient)
}
```
The P2P protocol suppose there are five kinds of activities in network layer, application can use the default design or setup their own interface.

### Message
There are serveral definations in DSP.

**ID** used to communication with each other.

**Message** core unit used in underlying communication.

**Keepalive** to find other peers whether online or not.

**Proxy** used in communication with proxy server.

**MAX_PACKAGE_SIZE** max package size in p2p layer is 1024 * 64.

**STORE_PACKAGE_REAL_SIZE** 2 

#### Message Format
- Message

```
message Message {
    bytes message = 1;

    // Sender's address and net key.
    ID sender = 2;

    // Sender's signature of message.
    bytes signature = 3;

    // request_nonce is the request/response ID. Null if ID associated to a message is not a request/response.
    uint64 request_nonce = 4;

    // message_nonce is the sequence ID.
    uint64 message_nonce = 5;

    // reply_flag indicates this is a reply to a request
    bool reply_flag = 6;

    // opcode specifies the message type
    uint32 opcode = 7;

    uint32 netID = 8;
    string MessageID = 9;
    bool need_ack = 10;
}
```
- ID

```
message ID {
    // net_key of the peer (we no longer use the public key as the peer ID, but use it to verify messages)
    bytes net_key = 1;
    // address is the network address of the peer
    string address = 2;
    // id is the computed hash of the net key
    bytes id = 3;
    //used in communication with proxy server
    bytes connection_id=4;
}
```

#### Serialization and Deserialization

The message is serializd to a raw bytes packet before send it out, and receiver of the packet will deserializd it to a protocol readable struct.The default serialization and deserialization schema in P2P layer is protobuf.

The schema could setup freely, but if endpoint want to connect to others, the schema in peers should be the same.

#### Payload Format

|Network ID(4) | CompressINFO(2) |Message Length(4)| Message(...)|
|---|---|---|---|---
|ID | Compress Information | raw data length | raw data|

#### Message Compression
P2P layer could compress the raw package when it exceed the config size which setup before the P2P started. The flag is in CompressINFO.

- Compress Information 

 16bits :  CompressEnable << 8 |Compress Algorithm

- Compress Algorithm List

|algo | code|
|---|---|
|NONE  | 0 |
|BZIP2 | 1 |
|GZIP  | 2 |
|ZLIB  | 3 |
|LZW   | 4 |
|FLAT  | 5 |

**All code encoded as 16bits length**
   
### Opcode
For each message,must registe an opcode for special message to build a opcode-message pairing.Default opcode for special message as follow:

```
type Opcode uint32
const (
	UnregisteredCode       Opcode = 0x00000 // 0
	BytesCode              Opcode = 0x00001 // 1
	KeepaliveCode          Opcode = 0x00002 // 2
	KeepaliveResponseCode  Opcode = 0x00003 // 3
	ProxyResponseCode      Opcode = 0x00009 // 9
	PingCode               Opcode = 0x0000a // 10
	PongCode               Opcode = 0x0000b // 11
	LookupNodeRequestCode  Opcode = 0x0000c // 12
	LookupNodeResponseCode Opcode = 0x0000d // 13
	DisconnectCode         Opcode = 0x0000e // 14
	ProxyRequestCode       Opcode = 0x0000f // 15
	MetricRequestCode      Opcode = 0x00010 // 16
	MetricResponseCode     Opcode = 0x00011 // 17
	AckResponseCode        Opcode = 0x00012 // 18

	ApplicationOpCodeStart Opcode = 1000
)
```

### Connection
Connection in DSP should be setup by serveral transport protocol, and the session could be setup many streams to handle different transmissions

#### Connection Establishment
The common processes of establishing a connection with others as follow:
1. Setup transport protocol, for example: TCP/UDP/QUIC and so on
2. Endpoint start listening with a speciall IP/Port, which will be connectd from outbound
3. Outbound connect to speciall address(IP:Port)
4. Connection will be managed in P2P layer
5. Only **ONE** connection between peers

#### Disconnection
Connection would be destoried under follow situations:
- EOF msg received
- receiving an ERROR message
- Peerdisconnect action be called
- Relative resource will be recycled in PeerClose flow
- rebuild a new connection immediatelly after disconnectiong
- When service quit, all connection will disconnect automatically

#### Stream
P2P layer should use FIFO to process message queue which ensure the stream layer write all data correctly.

There are both stream level write operation and connection level write operation in the upper layer, so the connection will only be used by one write operation at onetime.

When stream close abnomally, packets related to the stream will be discarded, and the packets that have entered the retransmission process will be removed from the retransmission queue.

If the same data in the link layer is not processed, and the same package comes from the upper layer again, the bottom layer will ignore it; if it has been processed, it will be processed as a new package.

If connection close or broken, all streams are closed; all resources related to stream are released; all packets based on the connection are discarded; and the upper layer get a failure notify from P2P layer.

## Status Code Definitions
This document establishes definitions for DSP status codes.Include task API error code, transmission process status code, RESTful API error code.

The latest definition could be found in [Code Definitions](https://github.com/DSP-Labs/specs/blob/master/RESTFUL_API_ERROR.md/).

### Task Error Code

| Error Code | Content | Meaning |
| ------ | ---- |--------------------- |
| 40000  | INVALID_PARAMS                      |The parameter is wrong|
| 40001  | INTERNAL_ERROR                      |internal error                           |
| 50000  | NEW_TASK_FAILED                     |Failed to create task|
| 50001  | PREPARE_UPLOAD_ERROR                |Abnormal data processing before uploading|
| 50002  | TASK_INTERNAL_ERROR                 |Abnormal task|
| 50003  | UPLOAD_TASK_EXIST                   |Upload task already exists|
| 50004  | SHARDING_FAIELD                     |Sharding failed|
| 50005  | FILE_HAS_UPLOADED                   |The file has been uploaded|
| 50006  | GET_STORAGE_NODES_FAILED            |Failed to obtain storage node|
| 50007  | ONLINE_NODES_NOT_ENOUGH             |The number of storage nodes in the entire network is insufficient|
| 50008  | PAY_FOR_STORE_FILE_FAILED           |File upload failed|
| 50009  | SET_FILEINFO_DB_ERROR               |File database save failed|
| 50010  | ADD_WHITELIST_ERROR                 |Failed to add whitelist|
| 50011  | GET_PDP_PARAMS_ERROR                |Failed to obtain PDP proof parameters|
| 50012  | GET_ALL_BLOCK_ERROR                 |Abnormal access to file block data|
| 50013  | SEARCH_RECEIVERS_FAILED             |Failed to find storage node|
| 50014  | RECEIVERS_NOT_ENOUGH                |Insufficient number of online storage nodes|
| 50015  | RECEIVERS_REJECTED                  |Storage node refuses to respond|
| 50016  | TASK_WAIT_TIMEOUT                   |Task wait timeout|
| 50017  | FILE_UPLOADED_CHECK_PDP_FAILED      |File has been uploaded and the storage node failed to submit the PDP prove|
| 50018  | GET_SESSION_ID_FAILED               |Failed to obtain SessionID|
| 50019  | FILE_UNIT_PRICE_ERROR               |File download price is abnormal|
| 50020  | TOO_MANY_TASKS                      |The task has reached the limit|
| 50021  | FILE_NOT_FOUND_FROM_CHAIN           |File information chain does not exist|
| 50022  | DOWNLOAD_REFUSED                    |Download rejected|
| 50023  | CHAIN_ERROR                         |Abnormal main chain request|
| 50024  | TASK_PAUSE_ERROR                    |Task pause failed|
| 50025  | GET_FILEINFO_FROM_DB_ERROR          |Failed to get file information from database|
| 50026  | DOWNLOAD_FILEHASH_NOT_FOUND         |File Hash not found|
| 50027  | NO_CONNECTED_DNS                    |No DNS connected|
| 50028  | NO_DOWNLOAD_SEED                    |No downloadable data source|
| 50029  | GET_DOWNLOAD_INFO_FAILED_FROM_PEERS |Failed to obtain information from the download node|
| 50030  | PREPARE_CHANNEL_ERROR               |Failed to prepare Channel before downloading|
| 50031  | FILEINFO_NOT_EXIST                  |File information does not exist|
| 50032  | PAY_UNPAID_BLOCK_FAILED             |Failed to pay for unpaid file blocks|
| 50033  | CREATE_DOWNLOAD_FILE_FAILED         |Failed to create download file|
| 50034  | GET_UNDOWNLOAD_BLOCK_FAILED         |Failed to get undownloaded file block|
| 50035  | DOWNLOAD_BLOCK_FAILED               |Failed to download file block|
| 50036  | GET_FILE_STATE_ERROR                |Failed to obtain file information|
| 50037  | WRITE_FILE_DATA_FAILED              |Failed to write data to file|
| 50038  | FS_PUT_BLOCK_FAILED                 |Failed to save file block to FS|
| 50039  | ADD_GET_BLOCK_REQUEST_FAILED        |Failed to add request file block task|
| 50040  | DECRYPT_FILE_FAILED                 |Decryption failed|
| 50041  | RENAME_FILED_FAILED                 |File rename failed|
| 50042  | DOWNLOAD_FILE_TIMEOUT               |File download timeout|
| 50043  | DOWNLOAD_FILE_RESUSED               |Download file refused|
| 50044  | UNITPRICE_ERROR                     |Failed to get file download unit price|
| 50045  | DOWNLOAD_TASK_EXIST                 |Download task already exists|
| 50046  | DECRYPT_WRONG_PASSWORD              |Wrong decryption password|
| 50047  | DELETE_FILE_HASHES_EMPTY            |Deleted file Hash is empty|
| 50048  | NO_FILE_NEED_DELETED                |No files need to be deleted|
| 50049  | DELETE_FILE_ACCESS_DENIED           |No permission to delete files|

### Transmission Status Code

| Status code | Content | Meaning |
| ------ | --------------------- | ---------------------- |
| 0      | None                              |None|
| 1      | TaskPause                         |Task pause|
| 2      | TaskDoing                         |Mission continues|
| 3      | TaskUploadFileMakeSlice           |Start sharding|
| 4      | TaskUploadFileMakeSliceDone       |Sharding completed|
| 7      | TaskUploadFileCommitWhitelist     |Submit whitelist information|
| 8      | TaskUploadFileCommitWhitelistDone |Complete whitelist information submission|
| 9      | TaskUploadFileFindReceivers       |Find storage nodes|
| 10     | TaskUploadFileFindReceiversDone   |Finding the storage node is complete|
| 11     | TaskUploadFileGeneratePDPData     |Generate PDP certification data|
| 12     | TaskUploadFileTransferBlocks      |Start transferring file block data|
| 13     | TaskUploadFileTransferBlocksDone  |Transfer file block data is complete|
| 14     | TaskUploadFileWaitForPDPProve     |Waiting for storage node to submit PDP certificate|
| 15     | TaskUploadFileWaitForPDPProveDone |The storage node submits the PDP certificate to complete|
| 16     | TaskUploadFileRegisterDNS         |Register information to DNS node|
| 17     | TaskUploadFileRegisterDNSDone     |Registration information is completed to the DNS node|
| 18     | TaskDownloadFileStart             |Start downloading files|
| 19     | TaskDownloadSearchPeers           |Find nodes for download|
| 20     | TaskDownloadFileDownloading       |File downloading|
| 21     | TaskDownloadRequestBlocks         |Download data block|
| 22     | TaskDownloadReceiveBlocks         |Data block received|
| 23     | TaskDownloadPayForBlocks          |Pay the download fee for the data block|
| 24     | TaskDownloadPayForBlocksDone      |Complete the download fee for the data block|
| 25     | TaskDownloadFileMakeSeed          |File download is complete, submit sharing information to DNS node|

### RESTful API Error Code
| Error Code | Content | Meaning |
| ------ | ------------- | ---------- |
| 0      |   SUCCESS                            | Success                        |
| 40001  | INTERNAL_ERROR                       | Internal Server Error          |
| 40002  | INVALID_PARAMS                       | Incorrect parameter (missing)  |
| 40003  | NO_DSP                               | Service not instantiated       |
| 40004  | NO_DB                                | Database initialization failed |
| 40005  | CONTRACT_ERROR                       | Contract call failed           |
| 40006  | INSUFFICIENT_BALANCE                 | Insufficient balance           |
| 40007  | NO_ACCOUNT                           | No account                     |
| 40008  | ACCOUNT_EXIST                        | Account already exists         |
| 40009  | NO_DNS                               | No DNS node connected          |
| 40010  | INVALID_WALLET_ADDRESS               | Wallet address is illegal      |
| 50000  | CHAIN_INTERNAL_ERROR                 | Internal error                 |
| 50001  | CHAIN_GET_HEIGHT_FAILED              | Failed to get current height   |
| 50002  | CHAIN_GET_BLK_BY_HEIGHT_FAILED       | Failed to get block            |
| 50003  | CHAIN_WAIT_TX_COMFIRMED_TIMEOUT      | Timeout waiting for transaction confirmation|
| 50004  | CHAIN_UNKNOWN_BLOCK                  | Unknown block |
| 50005  | CHAIN_UNKNOWN_TX                     | Unknown transaction |
| 50006  | CHAIN_UNKNOWN_SMARTCONTRACT          | Unknown contract |
| 50007  | CHAIN_UNKNOWN_SMARTCONTRACT_EVENT    | Unknown contract event |
| 50008  | CHAIN_UNKNOWN_ASSET                  | Unknown asset |
| 50009  | CHAIN_TRANSFER_ERROR                 | Transfer failed |
| 50013  | WALLET_FILE_NOT_EXIST                | Wallet file does not exist |
| 50014  | ACCOUNTDATA_NOT_EXIST                | Wallet account data does not exist |
| 50015  | ACCOUNT_PASSWORD_WRONG               | wrong password                       |
| 50016  | CREATE_ACCOUNT_FAILED                | Account creation failed |
| 50017  | ACCOUNT_EXPORT_FAILED                | Account export failed |
| 54001  | FS_GET_SETTING_FAILED                | Failed to obtain FS contract configuration |
| 54002  | FS_GET_USER_SPACE_FAILED             | Failed to obtain user space |
| 54003  | FS_GET_FILE_LIST_FAILED              | Failed to get file list |
| 54004  | FS_UPDATE_USERSPACE_FAILED           | Failed to update user space |
| 54005  | FS_CANT_REVOKE_OF_EXISTS_FILE        | Can't revoke space for storing files |
| 54006  | FS_NO_USER_SPACE_TO_REVOKE           | There is no room to revoke |
| 54007  | FS_USER_SPACE_SECOND_TOO_SMALL       | Increased seconds are too small |
| 54008  | FS_USER_SPACE_PERMISSION_DENIED      | No permission to update |
| 54009  | FS_UPLOAD_FILEPATH_ERROR             | Error in file upload path |
| 54010  | FS_UPLOAD_INTERVAL_TOO_SMALL         | The verification period for uploading calculated fee is too short |
| 54011  | FS_UPLOAD_GET_FILESIZE_FAILED        | Failed to calculate file fee to get file size |
| 54012  | FS_UPLOAD_CALC_FEE_FAILED            | Failed to calculate file fee |
| 55000  | DSP_INIT_FAILED                      | DSP initialization failed |
| 55001  | DSP_START_FAILED                     | DSP failed to start |
| 55002  | DSP_STOP_FAILED                      | DSP stop failed |
| 55010  | DSP_UPLOAD_FILE_FAILED               | File upload failed |
| 55011  | DSP_USER_SPACE_EXPIRED               | User space expired |
| 55012  | DSP_USER_SPACE_NOT_ENOUGH            | Insufficient user space |
| 55013  | DSP_UPLOAD_URL_EXIST                 | The file URL already exists |
| 55014  | DSP_DELETE_FILE_FAILED               | Failed to delete file |
| 55015  | DSP_CALC_UPLOAD_FEE_FAILED           | Failed to calculate upload cost |
| 55016  | DSP_GET_FILE_LINK_FAILED             | Failed to get file link |
| 55017  | DSP_ENCRYPTED_FILE_FAILED            | File encryption failed |
| 55018  | DSP_DECRYPTED_FILE_FAILED            | File decryption failed|
| 55019  | DSP_WHITELIST_OP_FAILED              | Whitelist operation failed|
| 55020  | DSP_GET_WHITELIST_FAILED             | Failed to get whitelist|
| 55021  | DSP_UPDATE_CONFIG_FAILED             | Failed to update system configuration|
| 55022  | DSP_UPLOAD_FILE_EXIST                | File uploaded|
| 55023  | DSP_PAUSE_UPLOAD_FAIELD              | Upload pause failed|
| 55024  | DSP_RESUME_UPLOAD_FAIELD             | Continue upload failure|
| 55025  | DSP_RETRY_UPLOAD_FAIELD              | Retry upload failed|
| 55026  | DSP_PAUSE_DOWNLOAD_FAIELD            | Failed to pause download|
| 55027  | DSP_RESUME_DOWNLOAD_FAIELD           | Continue download failed|
| 55028  | DSP_RETRY_DOWNLOAD_FAIELD            | Retry download failed|
| 55029  | DSP_CANCEL_TASK_FAILED               | Failed to cancel the task|
| 55030  | DSP_NODE_REGISTER_FAILED             | Node registration failed|
| 55031  | DSP_NODE_UNREGISTER_FAILED           | Node logout failed|
| 55032  | DSP_NODE_UPDATE_FAILED               | Node update failed|
| 55033  | DSP_NODE_WITHDRAW_FAILED             | Node withdrawal failed|
| 55034  | DSP_NODE_QUERY_FAILED                | Node query failed|
| 55040  | DSP_URL_REGISTER_FAILED              | URL registration failed|
| 55041  | DSP_URL_BIND_FAILED                  | URL binding failed|
| 55050  | DSP_DNS_REGISTER_FAILED              | DNS node registration failed|
| 55051  | DSP_DNS_UNREGISTER_FAILED            | DNS node logout failed|
| 55052  | DSP_DNS_UPDATE_FAILED                | DNS node update failed|
| 55053  | DSP_DNS_WITHDRAW_FAILED              | DNS node withdrawal failed|
| 55054  | DSP_DNS_QUIT_FAILED                  | DNS node exit failed|
| 55055  | DSP_DNS_ADDPOS_FAILED                | DNS node failed to add mortgage|
| 55056  | DSP_DNS_REDUCEPOS_FAILED             | DNS node failure to reduce mortgage|
| 55057  | DSP_DNS_GET_NODE_BY_ADDR             | Query DNS node failed|
| 55058  | DSP_DNS_QUERY_INFOS_FAILED           | Query all DNS node information failed      |
| 55059  | DSP_DNS_QUERY_INFO_FAILED            | Failed to query information about a single DNS node      |
| 55060  | DSP_DNS_QUERY_ALLINFOS_FAILED        | Query all DNS node information failed      |
| 55061  | DSP_DNS_GET_EXTERNALIP_FAILED        | Query DNS node to obtain IP information failed      |
| 55062  | DSP_USER_SPACE_PERIOD_NOT_ENOUGH     | Insufficient time in user space      |
| 55063  | DSP_CUSTOM_EXPIRED_NOT_ENOUGH        | Not enough storage time for upload settings      |
| 55100  | DSP_FILE_INFO_NOT_FOUND              | No file information found on the chain      |
| 55101  | DSP_FILE_NOT_EXISTS                  | file does not exist                           |
| 55102  | DSP_FILE_DECRYPTED_WRONG_PWD         | Wrong decryption password      |
| 58000  | DSP_TASK_NOT_EXIST                   | Operational task does not exist      |
| 59000  | DB_FIND_SHARE_RECORDS_FAILED         | Database query sharing revenue failed      |
| 59001  | DB_SUM_SHARE_PROFIT_FAILED           | Database statistics sharing revenue failure      |
| 59002  | DB_FIND_USER_SPACE_RECORD_FAILED     | Database query adjustment space record lost败     |
| 59003  | DB_ADD_USER_SPACE_RECORD_FAILED      | Database addition adjustment space record is missing败     |
| 59004  | DB_GET_FILEINFO_FAILED               | Failed to obtain file information      |
| 59100  | NET_RECONNECT_PEER_FAILED            | Node reconnection failed      |
## Contributing
Thank you for considering contributing to DSP. We welcome any individuals and organizations on the Internet to participate in this open source project.
