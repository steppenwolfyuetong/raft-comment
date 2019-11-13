# raft-comment

对etd中raft协议进行注释

## Raft library

Raft is a protocol with which a cluster of nodes can maintain a replicated state machine.
The state machine is kept in sync through the use of a replicated log.
For more details on Raft, see "In Search of an Understandable Consensus Algorithm"
(https://ramcloud.stanford.edu/raft.pdf) by Diego Ongaro and John Ousterhout.

Raft是一种协议，节点集群可使用该协议维护复制的状态机。
通过使用复制的日志，状态机保持同步。
有关Raft的更多详细信息，请参见“寻找可理解的共识算法”
（https://ramcloud.stanford.edu/raft.pdf），作者是Diego Ongaro和John Ousterhout。


This Raft library is stable and feature complete. As of 2016, it is **the most widely used** Raft library in production, serving tens of thousands clusters each day. It powers distributed systems such as etcd, Kubernetes, Docker Swarm, Cloud Foundry Diego, CockroachDB, TiDB, Project Calico, Flannel, and more.

该Raft库稳定且功能齐全。截至2016年，它是生产中使用最广泛的Raft库，每天服务数万个集群。它为etcd，Kubernetes，Docker Swarm，Cloud Foundry Diego，CockroachDB，TiDB，Project Calico，Flannel等分布式系统提供支持。

Most Raft implementations have a monolithic design, including storage handling, messaging serialization, and network transport. This library instead follows a minimalistic design philosophy by only implementing the core raft algorithm. This minimalism buys flexibility, determinism, and performance.
大多数Raft实现都具有整体设计，包括存储处理，消息传递序列化和网络传输。该库仅通过实现核心raft算法来遵循简约的设计理念。这种极简主义具有灵活性，确定性和性能。

To keep the codebase small as well as provide flexibility, the library only implements the Raft algorithm; both network and disk IO are left to the user. Library users must implement their own transportation layer for message passing between Raft peers over the wire. Similarly, users must implement their own storage layer to persist the Raft log and state.

为了使代码库较小并提供灵活性，该库仅实现Raft算法。网络和磁盘IO都留给用户。库用户必须实现自己的传输层，才能在网上的Raft对等点之间传递消息。同样，用户必须实现自己的存储层才能保留Raft日志和状态。

In order to easily test the Raft library, its behavior should be deterministic. To achieve this determinism, the library models Raft as a state machine.  The state machine takes a `Message` as input. A message can either be a local timer update or a network message sent from a remote peer. The state machine's output is a 3-tuple `{[]Messages, []LogEntries, NextState}` consisting of an array of `Messages`, `log entries`, and `Raft state changes`. For state machines with the same state, the same state machine input should always generate the same state machine output.

为了轻松测试Raft库，其行为应具有确定性。为了实现这种确定性，库将Raft建模为状态机。状态机将“消息”作为输入。消息可以是本地计时器更新，也可以是从远程对等方发送的网络消息。状态机的输出是一个三元组的{{] Messages，[] LogEntries，NextState}`，它由一系列Messages，log条目和Raft状态更改组成。对于状态相同的状态机，相同的状态机输入应始终生成相同的状态机输出。


A simple example application, _raftexample_, is also available to help illustrate how to use this package in practice: https://github.com/coreos/etcd/tree/master/contrib/raftexample

还提供了一个简单的示例应用程序_raftexample_来帮助说明如何在实践中使用此软件包：https://github.com/coreos/etcd/tree/master/contrib/raftexample


## Features

This raft implementation is a full feature implementation of Raft protocol. Features includes:

- Leader election
- Log replication
- Log compaction 
- Membership changes
- Leadership transfer extension
- Efficient linearizable read-only queries served by both the leader and followers
- leader checks with quorum and bypasses Raft log before processing read-only queries
- followers asks leader to get a safe read index before processing read-only queries
- More efficient lease-based linearizable read-only queries served by both the leader and followers
- leader bypasses Raft log and processing read-only queries locally
- this approach relies on the clock of the all the machines in raft group
 
 该Raft实现是Raft协议的全功能实现。 功能包括：
 
 -领导人选举
 -日志复制
 -日志压缩
 -成员变更
 -领导转移扩展
 -领导者和追随者均能有效地线性化的只读查询
 -领导检查法定人数并在处理只读查询之前绕过Raft日志
 -追随者要求领导者在处理只读查询之前获得安全的读索引
 -领导者和追随者均能更有效地进行基于租约的线性化只读查询
 -领导者绕过Raft日志并在本地处理只读查询
 -这种方法依赖于raft组中所有机器的时钟

This raft implementation also includes a few optional enhancements:

- Optimistic pipelining to reduce log replication latency
- Flow control for log replication
- Batching Raft messages to reduce synchronized network I/O calls
- Batching log entries to reduce disk synchronized I/O
- Writing to leader's disk in parallel
- Internal proposal redirection from followers to leader
- Automatic stepping down when the leader loses quorum 

该raft实现还包括一些可选的增强功能：
-优化流水线以减少日志复制延迟
-日志复制的流控制
-批处理raft消息以减少同步的网络I/O调用
-批处理日志条目以减少磁盘同步的I/O
-并行写入领导者的磁盘
-内部建议从追随者重定向到领导者
-当领导者失去法定人数时自动辞职

### Notable Users

- [cockroachdb](https://github.com/cockroachdb/cockroach) A Scalable, Survivable, Strongly-Consistent SQL Database
可扩展，可生存，高度一致的SQL数据库
- [dgraph](https://github.com/dgraph-io/dgraph) A Scalable, Distributed, Low Latency, High Throughput Graph Database
可扩展，分布式，低延迟，高吞吐量的图形数据库
- [etcd](https://github.com/coreos/etcd) A distributed reliable key-value store
分布式可靠的键值存储
- [tikv](https://github.com/pingcap/tikv) A Distributed transactional key value database powered by Rust and Raft
由Rust和Raft支持的分布式事务键值数据库
- [swarmkit](https://github.com/docker/swarmkit) A toolkit for orchestrating distributed systems at any scale.
用于编排任何规模的分布式系统的工具包。

### Usage

The primary object in raft is a Node. Either start a Node from scratch using raft.StartNode or start a Node from some initial state using raft.RestartNode.

raft中的主要对象是一个node。 使用raft.StartNode从头启动Node或使用raft.RestartNode从某个初始状态启动Node。

To start a three-node cluster
```go
  storage := raft.NewMemoryStorage()
  c := &Config{
    ID:              0x01,
    ElectionTick:    10,
    HeartbeatTick:   1,
    Storage:         storage,
    MaxSizePerMsg:   4096,
    MaxInflightMsgs: 256,
  }
  // Set peer list to the other nodes in the cluster.
  // Note that they need to be started separately as well.
  n := raft.StartNode(c, []raft.Peer{{ID: 0x02}, {ID: 0x03}})
```

Start a single node cluster, like so:
```go
  // Create storage and config as shown above.
  // Set peer list to itself, so this node can become the leader of this single-node cluster.
  peers := []raft.Peer{{ID: 0x01}}
  n := raft.StartNode(c, peers)
```

To allow a new node to join this cluster, do not pass in any peers. First, add the node to the existing cluster by calling `ProposeConfChange` on any existing node inside the cluster. Then, start the node with an empty peer list, like so:
```go
  // Create storage and config as shown above.
  n := raft.StartNode(c, nil)
```

To restart a node from previous state:
```go
  storage := raft.NewMemoryStorage()

  // Recover the in-memory storage from persistent snapshot, state and entries.
  storage.ApplySnapshot(snapshot)
  storage.SetHardState(state)
  storage.Append(entries)

  c := &Config{
    ID:              0x01,
    ElectionTick:    10,
    HeartbeatTick:   1,
    Storage:         storage,
    MaxSizePerMsg:   4096,
    MaxInflightMsgs: 256,
  }

  // Restart raft without peer information.
  // Peer information is already included in the storage.
  n := raft.RestartNode(c)
```

After creating a Node, the user has a few responsibilities:

First, read from the Node.Ready() channel and process the updates it contains. These steps may be performed in parallel, except as noted in step 2.

1. Write HardState, Entries, and Snapshot to persistent storage if they are not empty. Note that when writing an Entry with Index i, any previously-persisted entries with Index >= i must be discarded.

2. Send all Messages to the nodes named in the To field. It is important that no messages be sent until the latest HardState has been persisted to disk, and all Entries written by any previous Ready batch (Messages may be sent while entries from the same batch are being persisted). To reduce the I/O latency, an optimization can be applied to make leader write to disk in parallel with its followers (as explained at section 10.2.1 in Raft thesis). If any Message has type MsgSnap, call Node.ReportSnapshot() after it has been sent (these messages may be large). Note: Marshalling messages is not thread-safe; it is important to make sure that no new entries are persisted while marshalling. The easiest way to achieve this is to serialise the messages directly inside the main raft loop.

3. Apply Snapshot (if any) and CommittedEntries to the state machine. If any committed Entry has Type EntryConfChange, call Node.ApplyConfChange() to apply it to the node. The configuration change may be cancelled at this point by setting the NodeID field to zero before calling ApplyConfChange (but ApplyConfChange must be called one way or the other, and the decision to cancel must be based solely on the state machine and not external information such as the observed health of the node).

4. Call Node.Advance() to signal readiness for the next batch of updates. This may be done at any time after step 1, although all updates must be processed in the order they were returned by Ready.

Second, all persisted log entries must be made available via an implementation of the Storage interface. The provided MemoryStorage type can be used for this (if repopulating its state upon a restart), or a custom disk-backed implementation can be supplied.

Third, after receiving a message from another node, pass it to Node.Step:

创建节点后，用户将承担以下几项责任：
首先，从Node.Ready（）通道读取并处理其包含的更新。这些步骤可以并行执行，除非步骤2中另有说明。
1. 如果HardState，Entries和Snapshot不为空，则将它们写入持久性存储。请注意，在写索引为i的条目时，必须丢弃索引 >= i的所有先前存在的条目。

2. 将所有消息发送到“TO”字段中命名的节点。重要的是，直到将最新的HardState持久化到磁盘上，以及任何先前的Ready批处理写入的所有条目之前，都不要发送任何消息（可以在持久化来自同一批处理的条目时发送消息）。为了减少I
 / O延迟，可以应用优化以使领导者与其跟随者并行地写入磁盘（如Raft论文中的10.2.1节所述）。如果任何消息的类型为MsgSnap，请在发送消息后调用Node.ReportSnapshot（这些消息可能很大）。注意：编组消息不是线程安全的；重要的是要确保在编组时不保留任何新条目。实现此目的最简单的方法是直接在主Raft循环内序列化消息。

3. 将快照（如果有）和CommittedEntries应用于状态机。如果任何已提交的Entry的类型为EntryConfChange，则调用Node.ApplyConfChange将其应用于节点。此时可以通过在调用ApplyConfChange之前将NodeID字段设置为零来取消配置更改（但是ApplyConfChange必须以一种或另一种方式调用，并且取消决定必须仅基于状态机而不是外部信息，例如观察到的节点运行状况）。

4. 调用Node.Advance（）表示已准备好进行下一批更新。尽管必须按照Ready返回的顺序处理所有更新，但是可以在步骤1之后的任何时间完成此操作。

其次，必须通过存储接口的实现使所有持久日志条目可用。提供的MemoryStorage类型可以用于此目的（如果在重新启动时重新填充其状态），或者可以提供定制的磁盘支持的实现。

第三，从另一个节点收到消息后，将其传递给Node.Step：

```go
	func recvRaftRPC(ctx context.Context, m raftpb.Message) {
		n.Step(ctx, m)
	}
```

Finally, call `Node.Tick()` at regular intervals (probably via a `time.Ticker`). Raft has two important timeouts: heartbeat and the election timeout. However, internally to the raft package time is represented by an abstract "tick".

The total state machine handling loop will look something like this:

```go
  for {
    select {
    case <-s.Ticker:
      n.Tick()
    case rd := <-s.Node.Ready():
      saveToStorage(rd.State, rd.Entries, rd.Snapshot)
      send(rd.Messages)
      if !raft.IsEmptySnap(rd.Snapshot) {
        processSnapshot(rd.Snapshot)
      }
      for _, entry := range rd.CommittedEntries {
        process(entry)
        if entry.Type == raftpb.EntryConfChange {
          var cc raftpb.ConfChange
          cc.Unmarshal(entry.Data)
          s.Node.ApplyConfChange(cc)
        }
      }
      s.Node.Advance()
    case <-s.done:
      return
    }
  }
```

To propose changes to the state machine from the node to take application data, serialize it into a byte slice and call:

```go
	n.Propose(ctx, data)
```

If the proposal is committed, data will appear in committed entries with type raftpb.EntryNormal. There is no guarantee that a proposed command will be committed; the command may have to be reproposed after a timeout. 

To add or remove node in a cluster, build ConfChange struct 'cc' and call:

```go
	n.ProposeConfChange(ctx, cc)
```

After config change is committed, some committed entry with type raftpb.EntryConfChange will be returned. This must be applied to node through:

```go
	var cc raftpb.ConfChange
	cc.Unmarshal(data)
	n.ApplyConfChange(cc)
```

Note: An ID represents a unique node in a cluster for all time. A
given ID MUST be used only once even if the old node has been removed.
This means that for example IP addresses make poor node IDs since they
may be reused. Node IDs must be non-zero.

注意：ID始终代表集群中的唯一节点。一种
即使删除了旧节点，给定ID也必须仅使用一次。
这意味着，例如，IP地址使node ID可怜，因为它们
可以重复使用。节点ID必须为非零。

### Implementation notes

This implementation is up to date with the final Raft thesis (https://ramcloud.stanford.edu/~ongaro/thesis.pdf), although this implementation of the membership change protocol differs somewhat from that described in chapter 4. The key invariant that membership changes happen one node at a time is preserved, but in our implementation the membership change takes effect when its entry is applied, not when it is added to the log (so the entry is committed under the old membership instead of the new). This is equivalent in terms of safety, since the old and new configurations are guaranteed to overlap.

To ensure there is no attempt to commit two membership changes at once by matching log positions (which would be unsafe since they should have different quorum requirements), any proposed membership change is simply disallowed while any uncommitted change appears in the leader's log.

This approach introduces a problem when removing a member from a two-member cluster: If one of the members dies before the other one receives the commit of the confchange entry, then the member cannot be removed any more since the cluster cannot make progress. For this reason it is highly recommended to use three or more nodes in every cluster.

该实现与最新的Raft论文（https://ramcloud.stanford.edu/~ongaro/thesis.pdf）是最新的，尽管这种成员变化协议的实现与第4章中描述的有所不同。关键不变式成员变化一次保留一个节点，但是在我们的实现中，成员改变在应用其条目时生效，而不是在将其添加到日志时生效（因此该条目是在旧成员资格下而不是在新成员资格下提交的） 。就安全性而言，这是等效的，因为保证了新旧配置的重叠。

为了确保没有尝试通过匹配日志位置来一次提交两个成员更改（这是不安全的，因为它们应具有不同的法定人数要求，这是不安全的），任何提议的成员资格更改都将被禁止，而任何未提交的更ßßß将显示在领导者的日志中。

当从两个成员的集群中删除一个成员时，这种方法会带来一个问题：如果一个成员在另一个成员收到confchange条目的提交之前就已去世，则该成员将无法再删除，因为该集群无法取得进展。因此，强烈建议在每个群集中使用三个或更多节点。