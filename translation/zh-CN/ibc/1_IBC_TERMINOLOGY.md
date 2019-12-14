# 1：IBC术语

**这是IBC规范中使用的术语的概述。**

**For an architectural overview, see [here](./2_IBC_ARCHITECTURE.md).**

**有关广泛的协议设计原则，请参见[此处](./3_IBC_DESIGN_PRINCIPLES.md) 。**

**有关一组示例用例，请参见[此处](./4_IBC_USECASES.md) 。**

**有关设计模式的讨论，请参见[此处](./5_IBC_DESIGN_PATTERNS.md) 。**

本文档以简明的英语提供了整个IBC规范集中使用的关键术语的定义。

## 抽象定义

### Actor

An *actor*, or a *user* (used interchangeably), is an entity interacting with the IBC protocol. An actor can be a human end-user, a module or smart contract running on a blockchain, or an off-chain relayer process capable of signing transactions.

### 机器/链/账本

A *machine*, *chain*, *blockchain*, or *ledger* (used interchangeably), is a state machine (which may be a distributed ledger, or "blockchain", although a strict chain of blocks may not be required) implementing part or all of the IBC specification.

### Relayer process

A *relayer process* is an off-chain process responsible for relaying IBC packet data & metadata between two or more machines by scanning their states & submitting transactions.

### 状态机

The *state machine* of a particular chain defines the structure of the state as well as the set of rules which determines valid transactions that trigger state-transitions based on the current state agreed upon by the consensus algorithm of the chain.

### 共识

A *consensus* algorithm is the protocol used by the set of processes operating a distributed ledger to come to agreement on the same state, generally under the presence of a bounded number of Byzantine faults.

### 共识状态

The *consensus state* is the set of information about the state of a consensus algorithm required to verify proofs about the output of that consensus algorithm (e.g. commitment roots in signed headers).

### Commitment

A cryptographic *commitment* is a way to cheaply verify membership or non-membership of a key/value pair in a mapping, where the mapping can be committed to with a short witness string.

### Header

A *header* is an update to the consensus state of a particular blockchain, including a commitment to the current state, that can be verified in a well-defined fashion by a "light client" algorithm.

### CommitmentProof

A *commitment proof* is the proof structure which proves whether a particular key maps to a particular value in a committed-to set or not.

### Handler Module

The IBC *handler module* is the module within the state machine which implements [ICS 25](../spec/ics-025-handler-module), managing clients, connections, & channels, verifying proofs, and storing appropriate commitments for packets.

### 路由模块

The IBC *routing module* is the module within the state machine which implements [ICS 26](../spec/ics-026-routing-module), routing packets between the handler module and other modules on the host state machine which utilise the routing module's external interface.

### 数据报

A *datagram* is an opaque bytestring transmitted over some physical network, and handled by the IBC routing module implemented in the ledger's state machine. In some implementations, the datagram may be a field in a ledger-specific transaction or message data structure which also contains other information (e.g. a fee for spam prevention, nonce for replay prevention, type identifier to route to the IBC handler, etc.). All IBC sub-protocols (such as opening a connection, creating a channel, sending a packet) are defined in terms of sets of datagrams and protocols for handling them through the routing module.

### 连接

A *connection* is a set of persistent data structures on two chains that contain information about the consensus state of the other ledger in the connection. Updates to the consensus state of one chain changes the state of the connection object on the other chain.

### Channel

A *channel* is a set of persistent data structures on two chains that contain metadata to facilitate packet ordering, exactly-once delivery, and replay prevention. Packets sent through a channel change its internal state. Channels are associated with connections in a many-to-one relationship — a single connection can have any number of associated channels, and all channels must have a single associated connection, which must have been created prior to the creation of the channel.

### Packet

A *packet* is a particular data structure with sequence-related metadata (defined by the IBC specification) and an opaque value field referred to as the packet *data* (with semantics defined by the application layer, e.g. token amount and denomination). Packets are sent through a particular channel (and by extension, through a particular connection).

### Module

A *module* is a sub-component of the state machine of a particular blockchain which may interact with the IBC handler and alter state according to the *data* field of particular IBC packets sent or received (minting or burning tokens, for example).

### 握手

A *handshake* is a particular class of sub-protocol involving multiple datagrams, generally used to initialise some common state on the two involved chains such as trusted states for each others' consensus algorithms.

### 子协议

Sub-protocols are defined as a set of datagram kinds and functions which must be implemented by the IBC handler module of the implementing blockchain.

Datagrams must be relayed between chains by an external relayer process. This relayer process is assumed to behave in an arbitrary manner — no safety properties are dependent on its behaviour, although progress is generally dependent on the existence of at least one correct relayer process.

IBC sub-protocols are reasoned about as interactions between two chains `A` and `B` — there is no prior distinction between these two chains and they are assumed to be executing the same, correct IBC protocol. `A` is simply by convention the chain which goes first in the sub-protocol and `B` the chain which goes second. Protocol definitions should generally avoid including `A` and `B` in variable names to avoid confusion (as the chains themselves do not know whether they are `A` or `B` in the protocol).

### Authentication

*Authentication* is the property of ensuring that datagrams were in fact sent by a particular chain in the manner defined by the IBC handler.

## 属性定义

### Finality

*Finality* is the quantifiable assurance provided by a consensus algorithm that a particular block will not be reverted, subject to certain assumptions about the behaviour of the validator set. The IBC protocol requires finality, although it need not be absolute (for example, a threshold finality gadget for a Nakamoto consensus algorithm will provide finality subject to economic assumptions about how miners behave).

### Misbehaviour

*Misbehaviour* is a class of consensus fault defined by a consensus algorithm & detectable (possibly also attributable) by the light client of that consensus algorithm.

### Equivocation

*Equivocation* is a particular class of consensus fault committed by a validator or validators which sign votes on multiple different successors to a single block in an invalid manner.
All equivocations are misbehaviours.

### Data availability

*Data availability* is the ability of off-chain relayer processes to retrieve data in the state of a machine within some time bound.

### Data confidentiality

*Data confidentiality* is the ability of the host state machine to refuse to make particular data available to particular parties without impairing the functionality of the IBC protocol.

### 不可否认性

*Non-repudiability* is the inability of a machine to successfully dispute having sent a particular packet or committed a particular state. IBC is a non-repudiable protocol, modulo data confidentiality choices made by state machines.

### Consensus liveness

*Consensus liveness* is the continuance of block production by the consensus algorithm of a particular machine.

### Transactional liveness

*Transactional liveness* is the continued confirmation of incoming transactions (which transactions should be clear by context) by the consensus algorithm of a particular machine. Transactional liveness requires consensus liveness, but consensus liveness does not necessarily provide transactional liveness. Transactional liveness implies censorship resistance.

### Bounded consensus liveness

*Bounded consensus liveness* is consensus liveness within a particular bound.

### Bounded transactional liveness

*Bounded transactional liveness* is transactional liveness within a particular bound.

### Exactly-once safety

*Exactly-once safety* is the property that a packet is confirmed no more than once (and generally exactly-once assuming eventual transactional liveness).

### Deliver-or-timeout safety

*Deliver-or-timeout safety* is the property that a packet will either be delivered & executed or will timeout in a way that can be proved back to the sender.

### Constant (w.r.t. complexity)

*Constant*, when referring to space or time complexity, means `O(1)`.

### 简洁的

*Succinct*, when referring to space or time complexity, means `O(poly(log n))` or better.
