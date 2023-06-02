# Raft Distributed Consensus

## Problem

Given 1 client and 3 server nodes, how do the nodes come to a consensus of receiving the message from the client?

<img src="https://github.com/emeraldhieu/raft-consensus/blob/master/images/problem.png" width="55%">

## Solution

[Raft algorithm](https://raft.github.io/) solves it by using Leader Election and Log Replication.

### 1) Leader Election

A node has 3 states: Follower, Candidate, and Leader. 

<img src="https://github.com/emeraldhieu/raft-consensus/blob/master/images/states.png" width="55%">

If followers don't hear from a leader, they can become a candidate.

<img src="https://github.com/emeraldhieu/raft-consensus/blob/master/images/candidate.png" width="55%">

Then the candidate requests votes from other nodes.

<img src="https://github.com/emeraldhieu/raft-consensus/blob/master/images/requestsVotes.png" width="55%">

The nodes replies with their votes

<img src="https://github.com/emeraldhieu/raft-consensus/blob/master/images/repliesWithVote.png" width="55%">

The candidate has become the leader. From now, all changes go through the leader. 

<img src="https://github.com/emeraldhieu/raft-consensus/blob/master/images/candidateBecomesALeader.png" width="70%">

Every change from the client to the server is added as an entry in the node's log. This log entry is uncommitted so it won't update the node's value.

<img src="https://github.com/emeraldhieu/raft-consensus/blob/master/images/uncommitted.png" width="70%">

Before committing the entry, the node first replicates the log entry to the follower nodes.

<img src="https://github.com/emeraldhieu/raft-consensus/blob/master/images/replicatesLogEntry.png" width="75%">

The leader waits until a majority of nodes have written the entry and send back acknowledgment messages.

<img src="https://github.com/emeraldhieu/raft-consensus/blob/master/images/sendsBackAcknowledgement.png" width="75%">

The leader then notifies the followers that the entry has been committed.

<img src="https://github.com/emeraldhieu/raft-consensus/blob/master/images/committed.png" width="75%">

The cluster has come to a consensus about the system state.

<img src="https://github.com/emeraldhieu/raft-consensus/blob/master/images/consensus.png" width="75%">

#### 2) Timeout

There are two timeout settings that control the election.

##### 2.1) Election timeout

The election timeout is the amount of time a follower waits until becoming a candidate.

The election timeout is randomized to be between 150ms and 300ms.

After the election timeout, the follower becomes a candidate. It starts a new election term and votes for itself.

<img src="https://github.com/emeraldhieu/raft-consensus/blob/master/images/votesForItself.png" width="50%">

then it sends vote requests to other nodes

<img src="https://github.com/emeraldhieu/raft-consensus/blob/master/images/sendsVoteRequest.png" width="55%">

If the receiving node hasn't voted yet in this term then it votes for the candidate. Then it resets its election timeout.

<img src="https://github.com/emeraldhieu/raft-consensus/blob/master/images/votesForTheCandidate.png" width="50%">

##### 2.2) Heartbeat timeout

Once a candidate has a majority of votes, it becomes leader.

<img src="https://github.com/emeraldhieu/raft-consensus/blob/master/images/becomesALeader.png" width="50%">

The leader starts sending out AppendEntries messages specified by the heartbeat timeout to its followers.

<img src="https://github.com/emeraldhieu/raft-consensus/blob/master/images/sendsOutAppendEntries.png" width="55%">

Followers then respond to each Append Entries message.

<img src="https://github.com/emeraldhieu/raft-consensus/blob/master/images/respondsToAppendEntries.png" width="55%">

This election term will continue until a follower stops receiving heartbeats and becomes a candidate.

### 2) Log Replication

Once we have a leader elected, we need to replicate all changes to our system to all nodes. This is done by using the same AppendEntries message that was used for heartbeats.


First a client sends a change to the leader. The change is appended to the leader's log.

<img src="https://github.com/emeraldhieu/raft-consensus/blob/master/images/changeIsAppended.png" width="70%">

Then the change is sent to the followers on the next heartbeat.

<img src="https://github.com/emeraldhieu/raft-consensus/blob/master/images/sendsTheChangeOnTheNextHeartbeat.png" width="70%">

An entry is committed once a majority of followers acknowledge it.

<img src="https://github.com/emeraldhieu/raft-consensus/blob/master/images/committedOnceAcknowledged.png" width="70%">

A response is sent to the client.

<img src="https://github.com/emeraldhieu/raft-consensus/blob/master/images/sendsResponseToClient.png" width="70%">

## References

+ Visualisations: https://raft.github.io
+ Specification: https://raft.github.io/raft.pdf
