---
title: scalable selective permanode for Chrysalis PH2
tags: RFC
---

# scalable-selective-permanode
+ Feature name: `scalable-selective-permanode` for Chrysalis PH2
+ Start date: 2021-07-19
+ RFC PR: [iotaledger/chronicle-rfcs#xx](https://github.com/iotaledger/chronicle.rs-rfcs/pull/xx)
+ Chronicle issue:  [iotaledger/chronicle.rs#102](https://github.com/iotaledger/chronicle.rs/issues/102)
+ Author list
    + Louay Kamel (louay.kamel@iota.org)
    + Alexander Coats (alexander.coats@iota.org)
    + Bing-Yang Lin (bingyang.lin@iota.org)

# Summary

An abstraction for the scalable selective permanode feature for Chrysalis PH2, which enables users to select which messages should be stored in the permanode.

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

- Selective tables to create
    - Note that the user cannot select which column (field) in the table to be stored
        - Otherwise the data model needs to be customized
    - Some API calls will return `None` if the corresponding table is not created

- Traceable selective message paths
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

<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="285pt" height="44pt" viewBox="0.00 0.00 284.71 44.00">
<g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 40)">
<title>strategy</title>
<polygon fill="#ffffff" stroke="transparent" points="-4,4 -4,-40 280.7062,-40 280.7062,4 -4,4"/>
<!-- Selected -->
<g id="node1" class="node">
<title>Selected</title>
<polygon fill="#1be4e3" stroke="#000000" points="63.6347,-36 -.2125,-36 -.2125,0 63.6347,0 63.6347,-36"/>
<text text-anchor="middle" x="31.7111" y="-13.8" font-family="Times,serif" font-size="14.00" fill="#000000">Selected</text>
</g>
<!-- MiddleMessage -->
<g id="node2" class="node">
<title>MiddleMessage</title>
<polygon fill="#9fff00" stroke="#000000" points="186.6299,-36 80.7923,-36 80.7923,0 186.6299,0 186.6299,-36"/>
<text text-anchor="middle" x="133.7111" y="-13.8" font-family="Times,serif" font-size="14.00" fill="#000000">MiddleMessage</text>
</g>
<!-- Milestone -->
<g id="node3" class="node">
<title>Milestone</title>
<polygon fill="#fff100" stroke="#000000" points="276.7013,-36 204.7209,-36 204.7209,0 276.7013,0 276.7013,-36"/>
<text text-anchor="middle" x="240.7111" y="-13.8" font-family="Times,serif" font-size="14.00" fill="#000000">Milestone</text>
</g>
</g>
</svg>

<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="520pt" height="241pt" viewBox="0.00 0.00 519.98 241.00">
<g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 237)">
<title>strategy</title>
<polygon fill="#ffffff" stroke="transparent" points="-4,4 -4,-237 515.9804,-237 515.9804,4 -4,4"/>
<!-- A -->
<g id="node1" class="node">
<title>A</title>
<polygon fill="#1be4e3" stroke="#000000" points="54,-151 0,-151 0,-115 54,-115 54,-151"/>
<text text-anchor="middle" x="27" y="-128.8" font-family="Times,serif" font-size="14.00" fill="#000000">A</text>
</g>
<!-- B -->
<g id="node2" class="node">
<title>B</title>
<polygon fill="#ffffff" stroke="#000000" points="54,-209 0,-209 0,-173 54,-173 54,-209"/>
<text text-anchor="middle" x="27" y="-186.8" font-family="Times,serif" font-size="14.00" fill="#000000">B</text>
</g>
<!-- C -->
<g id="node3" class="node">
<title>C</title>
<polygon fill="#1be4e3" stroke="#000000" points="54,-89 0,-89 0,-53 54,-53 54,-89"/>
<text text-anchor="middle" x="27" y="-66.8" font-family="Times,serif" font-size="14.00" fill="#000000">C</text>
</g>
<!-- D -->
<g id="node4" class="node">
<title>D</title>
<polygon fill="#9fff00" stroke="#000000" points="145,-214 91,-214 91,-178 145,-178 145,-214"/>
<text text-anchor="middle" x="118" y="-191.8" font-family="Times,serif" font-size="14.00" fill="#000000">D</text>
</g>
<!-- D&#45;&gt;A -->
<g id="edge1" class="edge">
<title>D-&gt;A</title>
<path fill="none" stroke="#0000ff" stroke-dasharray="5,2" d="M91.679,-177.7778C82.3412,-171.3132 71.6843,-163.9353 61.7522,-157.0592"/>
<polygon fill="#0000ff" stroke="#0000ff" points="63.476,-153.9957 53.2618,-151.1813 59.4915,-159.7511 63.476,-153.9957"/>
</g>
<!-- D&#45;&gt;B -->
<g id="edge2" class="edge">
<title>D-&gt;B</title>
<path fill="none" stroke="#000000" d="M90.697,-194.4998C82.4187,-194.045 73.1783,-193.5373 64.3595,-193.0527"/>
<polygon fill="#000000" stroke="#000000" points="64.4607,-189.5531 54.2837,-192.4991 64.0766,-196.5426 64.4607,-189.5531"/>
</g>
<!-- E -->
<g id="node5" class="node">
<title>E</title>
<polygon fill="#9fff00" stroke="#000000" points="145,-156 91,-156 91,-120 145,-120 145,-156"/>
<text text-anchor="middle" x="118" y="-133.8" font-family="Times,serif" font-size="14.00" fill="#000000">E</text>
</g>
<!-- E&#45;&gt;B -->
<g id="edge3" class="edge">
<title>E-&gt;B</title>
<path fill="none" stroke="#000000" d="M90.697,-153.9017C81.9688,-158.9852 72.171,-164.6916 62.9256,-170.0763"/>
<polygon fill="#000000" stroke="#000000" points="61.1635,-167.0522 54.2837,-175.1095 64.6865,-173.1011 61.1635,-167.0522"/>
</g>
<!-- E&#45;&gt;C -->
<g id="edge4" class="edge">
<title>E-&gt;C</title>
<path fill="none" stroke="#0000ff" stroke-dasharray="5,2" d="M93.1339,-119.692C82.8179,-112.0967 70.7067,-103.1797 59.6871,-95.0663"/>
<polygon fill="#0000ff" stroke="#0000ff" points="61.649,-92.1645 51.521,-89.0539 57.4987,-97.8014 61.649,-92.1645"/>
</g>
<!-- F -->
<g id="node6" class="node">
<title>F</title>
<polygon fill="#ffffff" stroke="#000000" points="145,-94 91,-94 91,-58 145,-58 145,-94"/>
<text text-anchor="middle" x="118" y="-71.8" font-family="Times,serif" font-size="14.00" fill="#000000">F</text>
</g>
<!-- F&#45;&gt;A -->
<g id="edge5" class="edge">
<title>F-&gt;A</title>
<path fill="none" stroke="#000000" d="M90.697,-93.1019C81.9688,-98.569 72.171,-104.7061 62.9256,-110.4971"/>
<polygon fill="#000000" stroke="#000000" points="60.9006,-107.6356 54.2837,-115.9102 64.6164,-113.568 60.9006,-107.6356"/>
</g>
<!-- F&#45;&gt;C -->
<g id="edge6" class="edge">
<title>F-&gt;C</title>
<path fill="none" stroke="#0000ff" stroke-dasharray="5,2" d="M90.697,-74.4998C82.4187,-74.045 73.1783,-73.5373 64.3595,-73.0527"/>
<polygon fill="#0000ff" stroke="#0000ff" points="64.4607,-69.5531 54.2837,-72.4991 64.0766,-76.5426 64.4607,-69.5531"/>
</g>
<!-- G -->
<g id="node7" class="node">
<title>G</title>
<polygon fill="#1be4e3" stroke="#000000" points="308,-233 254,-233 254,-197 308,-197 308,-233"/>
<text text-anchor="middle" x="281" y="-210.8" font-family="Times,serif" font-size="14.00" fill="#000000">G</text>
</g>
<!-- G&#45;&gt;D -->
<g id="edge7" class="edge">
<title>G-&gt;D</title>
<path fill="none" stroke="#000000" d="M253.7175,-211.8198C226.721,-208.673 185.199,-203.833 155.1911,-200.3352"/>
<polygon fill="#000000" stroke="#000000" points="155.4378,-196.8403 145.0998,-199.1589 154.6273,-203.7932 155.4378,-196.8403"/>
<text text-anchor="middle" x="209" y="-213.2" font-family="Times,serif" font-size="14.00" fill="#000000">parent1</text>
</g>
<!-- H -->
<g id="node8" class="node">
<title>H</title>
<polygon fill="#ffffff" stroke="#000000" points="236,-128 182,-128 182,-92 236,-92 236,-128"/>
<text text-anchor="middle" x="209" y="-105.8" font-family="Times,serif" font-size="14.00" fill="#000000">H</text>
</g>
<!-- H&#45;&gt;E -->
<g id="edge8" class="edge">
<title>H-&gt;E</title>
<path fill="none" stroke="#000000" d="M181.697,-118.4009C173.3287,-120.9758 163.9773,-123.8531 155.0721,-126.5932"/>
<polygon fill="#000000" stroke="#000000" points="153.8122,-123.3189 145.2837,-129.605 155.8709,-130.0093 153.8122,-123.3189"/>
</g>
<!-- H&#45;&gt;F -->
<g id="edge9" class="edge">
<title>H-&gt;F</title>
<path fill="none" stroke="#0000ff" stroke-dasharray="5,2" d="M181.697,-99.7989C173.2388,-96.6387 163.776,-93.1031 154.785,-89.7438"/>
<polygon fill="#0000ff" stroke="#0000ff" points="155.8763,-86.4153 145.2837,-86.1939 153.4262,-92.9726 155.8763,-86.4153"/>
</g>
<!-- I -->
<g id="node9" class="node">
<title>I</title>
<polygon fill="#ffffff" stroke="#000000" points="308,-82 254,-82 254,-46 308,-46 308,-82"/>
<text text-anchor="middle" x="281" y="-59.8" font-family="Times,serif" font-size="14.00" fill="#000000">I</text>
</g>
<!-- I&#45;&gt;F -->
<g id="edge10" class="edge">
<title>I-&gt;F</title>
<path fill="none" stroke="#000000" d="M253.7175,-66.0085C226.721,-67.996 185.199,-71.0528 155.1911,-73.262"/>
<polygon fill="#000000" stroke="#000000" points="154.8158,-69.7801 145.0998,-74.0049 155.3298,-76.7612 154.8158,-69.7801"/>
</g>
<!-- J -->
<g id="node10" class="node">
<title>J</title>
<polygon fill="#9fff00" stroke="#000000" points="380,-181 326,-181 326,-145 380,-145 380,-181"/>
<text text-anchor="middle" x="353" y="-158.8" font-family="Times,serif" font-size="14.00" fill="#000000">J</text>
</g>
<!-- J&#45;&gt;H -->
<g id="edge15" class="edge">
<title>J-&gt;H</title>
<path fill="none" stroke="#0000ff" stroke-dasharray="5,2" d="M325.9147,-153.0311C303.3306,-144.7189 270.9302,-132.7938 245.9629,-123.6044"/>
<polygon fill="#0000ff" stroke="#0000ff" points="247.0469,-120.2739 236.4534,-120.1044 244.6291,-126.8431 247.0469,-120.2739"/>
</g>
<!-- K -->
<g id="node11" class="node">
<title>K</title>
<polygon fill="#9fff00" stroke="#000000" points="236,-182 182,-182 182,-146 236,-146 236,-182"/>
<text text-anchor="middle" x="209" y="-159.8" font-family="Times,serif" font-size="14.00" fill="#000000">K</text>
</g>
<!-- J&#45;&gt;K -->
<g id="edge16" class="edge">
<title>J-&gt;K</title>
<path fill="none" stroke="#000000" d="M325.9147,-163.1881C303.5305,-163.3435 271.5029,-163.566 246.6275,-163.7387"/>
<polygon fill="#000000" stroke="#000000" points="246.4289,-160.2399 236.4534,-163.8094 246.4775,-167.2397 246.4289,-160.2399"/>
</g>
<!-- K&#45;&gt;E -->
<g id="edge14" class="edge">
<title>K-&gt;E</title>
<path fill="none" stroke="#000000" d="M181.697,-156.1991C173.3287,-153.8082 163.9773,-151.1364 155.0721,-148.592"/>
<polygon fill="#000000" stroke="#000000" points="155.8605,-145.1773 145.2837,-145.7954 153.9374,-151.908 155.8605,-145.1773"/>
</g>
<!-- K&#45;&gt;F -->
<g id="edge13" class="edge">
<title>K-&gt;F</title>
<path fill="none" stroke="#0000ff" stroke-dasharray="5,2" d="M190.1447,-145.7663C176.8452,-132.9052 158.8562,-115.5093 144.1685,-101.3058"/>
<polygon fill="#0000ff" stroke="#0000ff" points="146.4528,-98.6459 136.8311,-94.2103 141.5866,-103.6779 146.4528,-98.6459"/>
</g>
<!-- L -->
<g id="node12" class="node">
<title>L</title>
<polygon fill="#ffffff" stroke="#000000" points="236,-36 182,-36 182,0 236,0 236,-36"/>
<text text-anchor="middle" x="209" y="-13.8" font-family="Times,serif" font-size="14.00" fill="#000000">L</text>
</g>
<!-- L&#45;&gt;C -->
<g id="edge12" class="edge">
<title>L-&gt;C</title>
<path fill="none" stroke="#000000" d="M181.6559,-24.5724C157.6361,-30.4909 121.8141,-39.6702 91,-49 82.1761,-51.6717 72.7485,-54.7725 63.9164,-57.7883"/>
<polygon fill="#000000" stroke="#000000" points="62.5669,-54.5516 54.2616,-61.1299 64.8564,-61.1667 62.5669,-54.5516"/>
</g>
<!-- L&#45;&gt;F -->
<g id="edge11" class="edge">
<title>L-&gt;F</title>
<path fill="none" stroke="#0000ff" stroke-dasharray="5,2" d="M181.697,-35.4019C172.9688,-40.9649 163.171,-47.2097 153.9256,-53.1023"/>
<polygon fill="#0000ff" stroke="#0000ff" points="151.8354,-50.2841 145.2837,-58.6104 155.5977,-56.1871 151.8354,-50.2841"/>
</g>
<!-- M1 -->
<g id="node13" class="node">
<title>M1</title>
<polygon fill="#fff100" stroke="#000000" points="511.9804,-128 457.9804,-128 457.9804,-92 511.9804,-92 511.9804,-128"/>
<text text-anchor="middle" x="484.9804" y="-105.8" font-family="Times,serif" font-size="14.00" fill="#000000">M1</text>
</g>
<!-- M1&#45;&gt;G -->
<g id="edge18" class="edge">
<title>M1-&gt;G</title>
<path fill="none" stroke="#000000" d="M473.6379,-128.0612C465.4721,-139.8644 453.5377,-154.9062 439.9804,-165 403.8053,-191.9332 352.7424,-204.605 318.3333,-210.3973"/>
<polygon fill="#000000" stroke="#000000" points="317.5614,-206.9755 308.2242,-211.9822 318.6457,-213.891 317.5614,-206.9755"/>
<text text-anchor="middle" x="418.9902" y="-191.2" font-family="Times,serif" font-size="14.00" fill="#000000">parent1</text>
</g>
<!-- M1&#45;&gt;H -->
<g id="edge19" class="edge">
<title>M1-&gt;H</title>
<path fill="none" stroke="#ff0000" stroke-dasharray="5,2" d="M457.763,-110C408.0667,-110 302.5787,-110 246.2614,-110"/>
<polygon fill="#ff0000" stroke="#ff0000" points="246.0135,-106.5001 236.0134,-110 246.0134,-113.5001 246.0135,-106.5001"/>
<text text-anchor="middle" x="353" y="-114.2" font-family="Times,serif" font-size="14.00" fill="#000000">parent3</text>
</g>
<!-- M1&#45;&gt;I -->
<g id="edge17" class="edge">
<title>M1-&gt;I</title>
<path fill="none" stroke="#a020f0" stroke-dasharray="5,2" d="M463.3201,-91.929C456.2836,-86.9809 448.1688,-82.1683 439.9804,-79.2 400.1119,-64.7478 351.1347,-62.1389 318.1638,-62.3805"/>
<polygon fill="#a020f0" stroke="#a020f0" points="317.994,-58.8827 308.0532,-62.5477 318.1098,-65.8818 317.994,-58.8827"/>
<text text-anchor="middle" x="418.9902" y="-83.2" font-family="Times,serif" font-size="14.00" fill="#000000">parent4</text>
</g>
<!-- M1&#45;&gt;J -->
<g id="edge20" class="edge">
<title>M1-&gt;J</title>
<path fill="none" stroke="#0000ff" stroke-dasharray="5,2" d="M457.9537,-120.8532C438.334,-128.732 411.5552,-139.4857 389.9581,-148.1586"/>
<polygon fill="#0000ff" stroke="#0000ff" points="388.4195,-145.0047 380.4441,-151.9791 391.0281,-151.5005 388.4195,-145.0047"/>
<text text-anchor="middle" x="418.9902" y="-148.2" font-family="Times,serif" font-size="14.00" fill="#000000">parent2</text>
</g>
<!-- M1&#45;&gt;L -->
<g id="edge21" class="edge">
<title>M1-&gt;L</title>
<path fill="none" stroke="#a52a2a" stroke-dasharray="5,2" d="M470.6696,-91.9759C462.4895,-82.6195 451.5695,-71.5898 439.9804,-64 380.3887,-24.9726 294.9321,-17.7603 246.4131,-17.1353"/>
<polygon fill="#a52a2a" stroke="#a52a2a" points="246.3685,-13.6351 236.3542,-17.0941 246.3398,-20.6351 246.3685,-13.6351"/>
<text text-anchor="middle" x="353" y="-40.2" font-family="Times,serif" font-size="14.00" fill="#000000">parent5</text>
</g>
</g>
</svg>

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