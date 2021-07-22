+ Feature name: `selective-permanode`
+ Start date: 2021-07-19
+ RFC PR: [iotaledger/chronicle-rfcs#0001](https://github.com/iotaledger/chronicle-rfcs/pull/1)
+ Chronicle issue:  [iotaledger/chronicle.rs#102](https://github.com/iotaledger/chronicle.rs/issues/102)
+ Author list
    + Louay Kamel (louay.kamel@iota.org)
    + Alexander Coats (alexander.coats@iota.org)
    + Bing-Yang Lin (bingyang.lin@iota.org)

# Summary

An abstraction for the scalable selective permanode feature for Chrysalis PH2, which enables users to select which messages should be stored in the permanode.

# Pre-limitation

In a tangle, the solidification mechanism is needed to make sure all the messages in a sub tangle is collected or not. In Chrysalis PH2, the milestone, which is issued by IOTA coordinator, is used as a global-trusted message to solidify a sub tangle. In a coordicide tangle, it is still needed to have a mechanism to solidify a sub tangle, or we cannot guarantee all of the messages are collected. This solidification mechanism will impact the design of the selective permanode, because in the permanode we need to use this mechanism to make sure all of the selected messages in a given period of time are all collected. In the future coordicide tangle, if we can define another kind of global trusted messages (which should exist for solidification), then the proposed design still holds.

# Motivation

The number of messages in tangle is huge and keeps increasing. For different user applications, not all data in tangle is necessary to keep. To reduce the maintenance cost, power consumption, as well as storage capacity, it is essential to enable users to **select** which messages should be persisted in Chronicle and which should not. This is called **selective-permanode**.

Note that the sub tangle constructed by selective permanode does not have to preserve path(s) between user interested data for confirmation tracing purpose. To trace the confirmation flow, the only path(s) needed to be kept in the selective-permanode is from an interested message to its corresponding **closest referencing milestone message**, because the paths between milestones must exist in IOTA Chrysalis PH2 protocol. For confirmation path tracing, it is not necessary to consume extra computing power or waste storage capacity to store the paths between milestones if there are no interested message between them.

In summary, a **selective permanode** provides a more cost-effective way than a **full permanode** (which preserves all the messages in the tangle). It enables users to store only desired messages but also confirmation path tracing without any additional effort.

Use cases:

1. Only persist specific data of the tangle in Chronicle and truncate/remove the unwanted ones.
2. Provide a more cost-effective storage solution than a `full permanode`.

# Current storage overhead of full permanode
- Around 2.6TB/year
    - Based on the information from https://chrysalis-dbfiles.iota.org/?prefix=chronicle/
    - Note the milestone period is 10 seconds.

# Selective Permanode Features
- User-defined messages to persist based on filtering
    - Each filter can be combined via AND/OR with another filter
    - Types of filters (each can be used with a pre-defined set of data or function to include/exclude messages)
        - **address filter**
        - **milestone filter**
        - **message_id filter**
        - **indexation_key filter**
  
- API calls to prune the chronicle database
    - Each API call can be triggered manually by user when the permanode is running
    - API list (each can be used with a pre-defined set of data or function to remove messages)
        - **prune_address**
        - **prune_milestone**
        - **prune_message_id**
        - **prune_indexation_key**

- Selective tables to create
    - Note that the user cannot select which column (field) in the table to be stored
        - Otherwise the data model needs to be customized
    - Some API calls will return `None` if the corresponding table is not created

- Traceable selective message paths
    - This feature is ease of tracing the selected message from a globally trusted message which is used to solidify the tangle
      - 
    - The messages which are in the linked solidification paths between selected messages should be kept
        - Those messages should be stored in the selective permanode in three different `Proof of Insertion` levels
            - **light proof**: Navigator points to the selected message
                - We only store the path from the milestone index and the linked parent position of the middle messages in the path
                - Need full permanode to verify the path
            - **hash proof**: Hashes which point to the selected message
                - We store the message ids in the path from the milestone to the selected message
                - Need full permanode to verify the path
            - **full proof**: Full messages which point to the selected message
                - We store the full message in the path from the milestone to the selected message
                - The selective permanode is self-verifiable
    - Block definition
        - Selected message: the message to be selected to persist
        - Middle message: the message which is not selected but exists in the path between the milestone and the selected message, which should be also persisted to ease of message tracing
        - Milestone: the milestone message
    - The solidification process is depth first, the middle messages which exist in the path between the milestone message and the selected messages will be stored in selective permanode

# Example
In the following example, messages A, C, G will be stored with full information

![](../figure/0001/blocks.svg)
![](../figure/0001/example.svg)

Messages D, E, K and J will be persisted in the [messages table](#messages)

### **Light proof**
`proof` column in the [message table](#messages) contains:
- Message A
    - Milestone index of M1 (`u32`)
    - parent position of G (`u8`)
    - parent position of D (`u8`)
- Message C
    - Milestone index of M1
    - parent position of J
    - parent position of K
    - parent position of E
- Message G
    - Milestone index of M1

### **Hash proof**
`proof` column in the [message table](#messages) contains:
- Message A
    - hash(M1)
    - hash(G)
    - hash(D)
- Message C
    - hash(M1)
    - hash(J)
    - hash(K)
    - hash(E)
- Message G
    - hash(M1)

### **Full proof**
Milestone M1 will be stored with full information

Full messages D, E, K, J will be persisted

#### Option 1
- Store the parent information for them in the [parents table](#parents)
- Store the message information for them in the [messages table](#messages)

`proof` column in the [message table](#messages) contains:
- Message A
    - hash(M1)
    - hash(G)
    - hash(D)
- For message C
    - hash(M1)
    - hash(J)
    - hash(K)
    - hash(E)
- For message G
    - hash(M1)

### Option 2
`proof` column in the [message table](#messages) contains:
- Message A
    - Full message of M1
    - Full message of G
    - Full message of D
- Message C
    - Full message of M1 
    - Full message of J
    - Full message of K
    - Full message of E
- Message G
    - Full message of M1


## Solidifiable/Verifiable selective messages
The solidification process in a selective permanode is exactly the same as a full permanode

Note that the solidification process is performed in caches

In selective permanode, we will select the interested messages and then store them to the database

# Related table list
- The following tables follow the same data model design of current [full permanode](#https://github.com/iotaledger/chronicle.rs/blob/a5bbd5f04ef31b518b3567bc818fff9bc994ba73/chronicle/src/main.rs#L171-L253) with extra `proof` column in the [message table](#messages)

## messages
- Primary key: message_id

| Column     | Data type |
| ---------- | --------- |
| message_id | text      |
| message    | blob      |
| metadata   | blob      |
| proof      | blob      |

## parents
- Primary key: (parent_id, partition_id)

| Column          | Data type |
| --------------- | --------- |
| parent_id       | text      |
| partition_id    | smallint  |
| milestone_index | int       |
| message_id      | text      |
| inclusion_state | blob      |


# Drawbacks

- If the there is no milestone or similar concepts in future IOTA protocol, then this selective permanode design needs to be changed.

# Limitations

- To solidify messages of a milestone, the queried IOTA nodes or permanodes need to contain the smallest full set of messages which are attached directly or indirectly by the milestone.

# Unsolved questions
- For **full proof** level, which option do we need to implement?
    - Option 1
        - Pros
            - Save the storage cost
        - Cons
            - Need to traverse the selective permanode to proof the selected messages
    - Option 2
        - Pros
            - Ease of tracing and verifying the selected messages
        - Cons
            - May consume lots of storage space if the number of middle messages (which might be repetitive) are many
- Do we force the user store the full path in the proof column, or we just store the full information of selected messages and the middle message in the [messages table](#messages) and [parents table](#parents)?
    - Pros
        - Save the storage cost
    - Cons
        - To trace a selected message will be time consuming