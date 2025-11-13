# Chapter 3 Review Questions

## SECTIONS 3.1–3.3

**R1.** 

Suppose the network layer provides the following service. The network layer in the source host accepts a segment of maximum size 1,200 bytes and a destination host address from the transport layer. The network layer then guarantees to deliver the segment to the transport layer at the destination host. Suppose many network application processes can be running at the destination host.

a. Design the simplest possible transport-layer protocol that will get application data to the desired process at the destination host. Assume the operating system in the destination host has assigned a 4-byte port number to each running application process.  
b. Modify this protocol so that it provides a “return address” to the destination process.  
c. In your protocols, does the transport layer “have to do anything” in the core of the computer network?

---

### Given:
- Network layer guarantees reliable delivery of a segment (max 1,200 bytes) between source and destination hosts (not processes).  
- Multiple processes (applications) may be running on each host.  
- Each process has a unique 4-byte port number.

> So the network layer knows how to reach hosts, but not which process within a host the data is for — that’s the transport layer’s job.

---

## (a) Simplest possible transport-layer protocol

**Goal:** Deliver data to the correct process on the destination host.

**Solution idea:** We need only multiplexing/demultiplexing functionality.

The simplest protocol would be a header containing only:

- Destination port number (4 bytes)

**Segment structure:**

| Field           | Size     | Description              |
|-----------------|----------|--------------------------|
| Destination Port | 4 bytes | Identifies receiving process |
| Data             | up to 1,196 bytes | Application data |

The transport layer at the destination:  
- Looks at the port number.  
- Passes the data to the correct process (via OS).

✅ No sequence numbers, no acknowledgments, no checksums — purely addressing.

---

## (b) Adding a “return address”

Now the receiver process should know where to send a reply.

We add a source port number to identify the sender process.

**Segment structure:**

| Field           | Size     | Description              |
|-----------------|----------|--------------------------|
| Source Port      | 4 bytes | Sending process ID       |
| Destination Port | 4 bytes | Receiving process ID     |
| Data             | up to 1,192 bytes | Application data |

Now, when the destination receives the segment, it can read both:  
- The destination port → to know which process gets the data.  
- The source port → to know where to send a response.

> This is essentially the structure of a UDP header (though we’re skipping checksum and length fields).

---

## (c) Does the transport layer “have to do anything” in the core of the network?

No.

The transport layer operates only at the end hosts, not within routers or network-layer devices.

- The core network (routers) sees only network-layer packets (e.g., IP headers).  
- It does not inspect or process transport-layer headers.  
- Thus, the transport layer does not have to do anything inside the network core.

> Only the end systems perform transport-layer processing (multiplexing, demultiplexing, port management).

---

### ✅ Summary check:
- (a) Only destination port — one-way delivery.  
- (b) Add source port — enables replies.  
- (c) Transport layer acts only at end hosts; core routers remain unaware.


**R2**

Consider a planet where everyone belongs to a family of six, every family lives in its own house, each house has a unique address, and each person in a given house has a unique name. Suppose this planet has a mail service that delivers letters from source house to destination house. The mail service requires that:

1. The letter be in an envelope.  
2. The address of the destination house (and nothing more) be clearly written on the envelope.  

Suppose each family has a delegate family member who collects and distributes letters for the other family members. The letters do not necessarily provide any indication of the recipients of the letters.

a. Using the solution to Problem R1 above as inspiration, describe a protocol that the delegates can use to deliver letters from a sending family member to a receiving family member.  
b. In your protocol, does the mail service ever have to open the envelope and examine the letter in order to provide its service?

---

## 🧩 Analogy mapping

| Planet mail world | Networking concept |
|------------------|------------------|
| House address     | Host address (IP address) |
| Family member     | Process/application running on host |
| Delegate          | Transport layer (e.g., TCP/UDP at the host) |
| Mail service      | Network layer |
| Letter            | Application data |
| Envelope          | Network-layer packet |
| Name written on letter | Port number or process identifier |

---

## (a) Protocol that delegates can use

**Goal:** Make sure a letter (data) from one specific person in one family reaches the correct person in another family, even though the mail service (network layer) only understands house addresses.

### Step-by-step reasoning

1. **Problem:**  
   The mail service can only deliver to houses, not people.  
   So the transport-level (delegate) must handle demultiplexing within the house.

2. **Idea:**  
   Use something in the letter itself that says who in the family it’s for — and who it’s from.

3. **Protocol design (simplest possible):**  
   Each delegate adds a small header to every outgoing letter before putting it in the envelope.  
   This header includes:
   - Destination person name (like destination port number)  
   - Source person name (like source port number)  

   Then, the delegate passes the letter to the mail service, writing only the house address (like IP address) on the envelope.  

   When the envelope arrives at the destination house:  
   - The delegate (transport layer) opens it, reads the letter’s header.  
   - Uses the destination person name to hand it to the right family member.  
   - The source person name tells the recipient who sent it (the “return address”).

---

## 🧾 Protocol summary

| Field             | Analogy             | Function                                   |
|------------------|-------------------|-------------------------------------------|
| Destination name  | Destination port number | Identify which person (process) should receive it |
| Source name       | Source port number      | Identify who sent it, for reply         |
| Letter contents   | Application data        | Payload                                 |
| Envelope address  | IP address             | Destination host (house)                |

✅ This matches the transport layer sitting above the network layer, adding process-level addressing.

---

## (b) Does the mail service ever have to open the envelope?

No.  
- The mail service (network layer) only needs to know house addresses.  
- It delivers envelopes from one address to another — it never looks inside.

> If the mail service opened the envelope, that would violate layer separation: it would be mixing transport and network functionality.

So:  
The mail service never examines the letter’s contents. Only the delegate (transport layer) at the destination house opens it to find the correct person.



**R3**

Consider a TCP connection between Host A and Host B. Suppose that the TCP segments traveling from Host A to Host B have source port number x and destination port number y. What are the source and destination port numbers for the segments traveling from Host B to Host A?

---

## 🧩 Given

TCP connection between Host A and Host B:  
- Segments from A → B have:  
  - Source port = x  
  - Destination port = y

---

## 🔁 What happens on the return path (B → A)?

In TCP, a connection is defined by a 4-tuple:


That 4-tuple uniquely identifies the connection on both ends.

For the reverse direction (B → A), the connection is the same, but the roles of source and destination are swapped.

So the reverse-direction segments must have:  
- Source port = y  
- Destination port = x

---

## ✅ Final Answer

| Direction | Source Port | Destination Port |
|-----------|-------------|-----------------|
| A → B     | x           | y               |
| B → A     | y           | x               |

---

## 🧠 Concept check

- TCP connections are bidirectional, but each direction uses the same pair of ports — just reversed.  
- This is how TCP knows which application on each host the returning data belongs to.  
- Routers and the network core don’t care about ports — only end hosts use them for demultiplexing.

**R4**

Describe why an application developer might choose to run an application over UDP rather than TCP.

---

## ⚙️ First: what TCP gives you

TCP provides:  
- Reliable data transfer (retransmissions, ACKs, sequence numbers)  
- In-order delivery  
- Congestion control  
- Flow control  
- Connection-oriented service (3-way handshake before data transfer)  

> That’s great for correctness — but it adds delay, state, and overhead.

---

## 🧩 What UDP gives instead

UDP is:  
- Connectionless (no handshake)  
- Unreliable (no ACKs, no retransmissions)  
- No ordering guarantees  
- No congestion or flow control  
- Lightweight header (8 bytes)  

> This means lower latency and finer control for the application.

---

## 🚀 Why choose UDP

An application developer might choose UDP when speed, timing, or control matter more than reliability.  

**Typical reasons:**

1. **Low latency / real-time delivery**  
   - Voice, video, or online games care more about fresh data than perfect reliability.  
   - If a packet is late, it’s useless — better to skip than to wait for retransmission.  
   - Example: VoIP (Skype), live streaming, multiplayer games.

2. **Application-level control**  
   - Developer can implement custom reliability, ordering, or error control tuned for the app.  
   - Example: QUIC (built over UDP but adds reliability at user space).

3. **No need for connection setup**  
   - UDP avoids the 3-way handshake — faster start (useful for DNS queries, small request/response).  
   - Example: DNS, DHCP.

4. **Stateless communication**  
   - Servers can handle many clients easily without storing connection state (lightweight).  
   - Example: Simple request/response protocols.

---

## ✅ Summary

| TCP                           | UDP                           |
|-------------------------------|-------------------------------|
| Reliable, ordered, connection-oriented | Unreliable, unordered, connectionless |
| Higher overhead, slower start | Lightweight, low latency      |
| Good for file transfer, web, email | Good for streaming, gaming, DNS |


# R5
Why is it that voice and video traffic is often sent over TCP rather than UDP in today’s Internet?  
*(Hint: The answer we are looking for has nothing to do with TCP’s congestion-control mechanism.)*

> This is tricky because it’s tempting to say “UDP is always better for voice/video because of low latency,” but the question explicitly rules out congestion control.  

---

## Step 1: Recall the main differences

- **UDP:** connectionless, unreliable, no ordering, no delivery guarantees.  
- **TCP:** reliable, in-order delivery, stream abstraction.  

> The hint tells us: the answer is not about congestion control — so focus on reliability / ordering.

---

## Step 2: Problem with UDP for real-world Internet voice/video

1. **Packet loss and network variability**  
   - UDP delivers packets “as is.”  
   - On real networks, even a few packet drops can seriously degrade video or voice quality.  
   - Applications would need to implement their own error handling (e.g., retransmissions, FEC, or smoothing buffers).

2. **Middleboxes and firewalls**  
   - Many NATs, firewalls, and proxies block or restrict UDP traffic.  
   - TCP is almost always allowed, so using TCP increases reachability.

3. **Compatibility with streaming protocols**  
   - HTTP-based streaming (HLS, DASH) relies on TCP.  
   - Even though it introduces some buffering, using TCP ensures delivery of all chunks — which is often preferable to missing frames.

---

## Step 3: Conclusion

Voice/video can still use UDP for low-latency real-time apps (like Zoom or online games), but on the “modern Internet,” many apps choose TCP because:

- Reliable delivery of packets is sometimes more important than strict low latency, especially for video streaming where frames can be buffered.  
- UDP may be blocked by network devices, TCP has better reachability.

> So the reason is reliability and traversing firewalls/NATs, not congestion control.

---

## ✅ Key takeaway

- **UDP** = low-latency, potentially lossy  
- **TCP** = reliable delivery, widely supported  
- Modern streaming apps often favor TCP (or TCP-like protocols over UDP, e.g., QUIC) for practical deployment, not because of congestion control.


**R6**

Is it possible for an application to enjoy reliable data transfer even when the application runs over UDP? If so, how?

---

## Step 1: UDP itself

- UDP is unreliable: no ACKs, no retransmissions, no ordering, no guarantees.  
- By itself, if a packet is lost, the application will never see it.

> So, UDP does not provide reliability natively.

---

## Step 2: Can reliability be added?

Yes — the application can implement reliability itself, on top of UDP.  

This is sometimes called **reliable UDP** or **application-level transport**.

**Mechanisms the application can use:**

1. **Acknowledgments (ACKs)**  
   - Sender waits for confirmation that a packet was received.  
   - If no ACK is received in time, sender retransmits.

2. **Sequence numbers**  
   - Each packet has a number so the receiver can detect loss or out-of-order delivery.  
   - The receiver can reorder packets or request retransmission.

3. **Checksums / error detection**  
   - Detect corruption in the packet content.

4. **Timers and retransmission logic**  
   - If a packet or ACK is delayed or lost, retransmit after a timeout.

---

## Step 3: Real-world examples

- **QUIC protocol:** built on UDP, adds reliability, ordering, congestion control.  
- **RTP + application-level retransmission:** some VoIP/video apps use UDP but implement selective retransmission.

> So, reliable data transfer doesn’t require TCP — it just requires the application to implement the necessary mechanisms.

---

## Step 4: Conceptual diagram 

- TCP implements all of this in the transport layer.  
- Using UDP + application logic is more flexible, e.g., selective reliability, partial retransmissions, or low-latency trade-offs.

---

## ✅ Key takeaway

- Yes, reliable data transfer is possible over UDP.  
- It requires the application to handle ACKs, sequencing, retransmissions.  
- This is a good illustration of the **end-to-end principle** in networking: reliability is an end-host function, not a network function.

**7**

Suppose a process in Host C has a UDP socket with port number 6789.  
Suppose both Host A and Host B each send a UDP segment to Host C with destination port number 6789. Will both of these segments be directed to the same socket at Host C? If so, how will the process at Host C know that these two segments originated from two different hosts?

> This is about UDP sockets and demultiplexing.

---

## Step 1: What identifies a UDP socket?

On a host, a UDP socket is identified by a local port number (and optionally a local IP if the host has multiple interfaces).

- Host C has a UDP socket on port 6789.  
- Any UDP segment arriving with destination port = 6789 will be delivered to that socket.

✅ So both segments from Host A and Host B will go to the same socket.

---

## Step 2: How does the process know who sent each segment?

- UDP segments include a **source IP address** and **source port number** in the UDP header.  
- When the process at Host C calls `recvfrom()` (or equivalent), it not only receives the data but also the source address and port.

| Segment      | Destination socket | Source info               |
|-------------|-----------------|---------------------------|
| From Host A | port 6789       | IP of A + source port of A |
| From Host B | port 6789       | IP of B + source port of B |

> The process can inspect the source IP and port to distinguish between senders.

---

## Step 3: Key points

1. Single socket can handle multiple senders — UDP is connectionless.  
2. Source address + source port is how the application distinguishes senders.  
3. If the process wants to reply, it uses the source IP/port from each incoming segment as the “return address.”

---

## ✅ Summary

- Yes, both segments go to the same socket.  
- Process knows the origin by examining the source IP address and source port number in each received UDP segment.


**R8**

Suppose that a Web server runs in Host C on port 80. Suppose this Web server uses persistent connections, and is currently receiving requests from two different Hosts, A and B. Are all of the requests being sent through the same socket at Host C? If they are being passed through different sockets, do both of the sockets have port 80? Discuss and explain.

> This is about TCP, sockets, and connection multiplexing.

---

## Step 1: Key facts

- Web server runs on Host C, port 80 (standard HTTP port).  
- Persistent connections: multiple requests from the same client can use a single TCP connection.  
- Two clients: Host A and Host B.

---

## Step 2: TCP connection identification

A TCP connection is uniquely identified by a 4-tuple:


- **Destination IP/port** → server address and port (here, Host C:80)  
- **Source IP/port** → client address and ephemeral port (assigned by client OS)  

> Each connection between Host C and a client is distinct, even if the server port is the same.

---

## Step 3: Sockets on Host C

- The server listens on port 80 using a **listening socket**.  
- When a connection is accepted (`accept()` call), the OS creates a **new socket** for that connection:  
  - This socket is used for all communication with that specific client.  
  - It has the same local port 80, but is distinguished by the client IP and client port.

| Client  | Server socket on C | Local port | Distinguishing info           |
|---------|------------------|-----------|-------------------------------|
| Host A  | socket #1        | 80        | client IP A + ephemeral port A |
| Host B  | socket #2        | 80        | client IP B + ephemeral port B |

---

## Step 4: Persistent connections

- Within each connection, the client can send multiple requests over the **same socket**.  
- Host C keeps **different sockets** for each client.

✅ So requests from A and B are **not sent through the same socket**, even though the server port is still 80 on both.

---

## Step 5: Key points / summary

1. TCP **listening socket**: port 80, waits for new connections.  
2. **Accepted socket**: one per client connection, also uses port 80 locally.  
3. Different clients → different sockets, but same server port.  
4. Persistent connections allow multiple requests per socket from a single client.

---

## ✅ Analogy (helpful for exams)

- **Listening socket** = front door of a house  
- **Accepted socket** = hallway to a specific guest  
- Multiple guests = multiple hallways, all enter through the same front door (port 80)

# SECTION 3.4

**R9**

In our rdt protocols, why did we need to introduce sequence numbers?

> This is core transport-layer reasoning about reliable data transfer (RDT).

---

## Step 1: What rdt protocols are

- rdt = reliable data transfer protocols in the book (Kurose & Ross)  
- Goal: ensure no data is lost, duplicated, or corrupted  
- Operate over unreliable channels (e.g., messages may be lost or corrupted)

---

## Step 2: What problem sequence numbers solve

1. **Duplicate detection**  
   - Suppose the sender retransmits a packet because it thinks the original was lost.  
   - The receiver may already have received the original, but without a way to distinguish, it would deliver the same data twice to the application.  
   - **Solution:** add a sequence number to each packet. Receiver checks the sequence number to see if it’s new or a duplicate.

2. **Ordering**  
   - In protocols that allow multiple outstanding packets (like sliding window protocols), packets may arrive out of order.  
   - Sequence numbers allow the receiver to reorder packets correctly before delivering to the application.

---

## Step 3: Minimal case

- Even in the simplest stop-and-wait protocol:  
  - Only two sequence numbers (0 and 1) are enough.  
  - Why? Because only one packet is in flight at a time, and the number helps detect duplicate retransmissions.

---

## ✅ Step 4: Summary

- Sequence numbers are needed to:  
  1. Detect duplicate packets caused by retransmissions  
  2. Maintain correct order when multiple packets are in transit

> Without sequence numbers, the receiver cannot distinguish old (duplicate) packets from new ones → the application could receive incorrect or repeated data.

**R10**

In our rdt protocols, why did we need to introduce timers?

> This is about stop-and-wait and reliable data transfer over unreliable channels. Let’s reason carefully.

---

## Step 1: What problem we’re solving

- rdt protocols run over channels where packets may be lost.  
- If a packet is lost, the receiver never receives it, so it cannot acknowledge it.  
- If the sender just waits forever for an ACK, the data will never reach the application.

---

## Step 2: Role of the timer

- Timers allow the sender to detect lost packets.  

**Mechanism:**

1. Sender sends a packet  
2. Starts a timer  
3. If ACK is received before timer expires → all good  
4. If timer expires before ACK → assume packet (or ACK) is lost → retransmit the packet  

> Without timers:  
> - Lost packets would never be retransmitted  
> - Reliability would fail

---

## Step 3: Minimal rdt scenario

- Stop-and-wait rdt uses one timer per packet.  
- Sliding window protocols use timers per packet or cumulative timers, but the principle is the same:  
  - **Timer = safeguard against loss and triggers retransmission**

---

## ✅ Step 4: Summary

- Timers are needed to detect lost packets or lost ACKs  
- They allow the sender to retransmit and ensure reliable data delivery over an unreliable channel  
- Without timers, rdt protocols cannot guarantee that all data eventually reaches the receiver

**12**

Suppose that the roundtrip delay between sender and receiver is constant and known to the sender. Would a timer still be necessary in protocol rdt 3.0, assuming that packets can be lost? Explain.

> This is subtle because it tests understanding of why timers are needed even if RTT is known.

---

## Step 1: rdt 3.0 context

- rdt 3.0 = stop-and-wait protocol over an unreliable channel (packets/ACKs may be lost)  
- Mechanisms: sequence numbers, ACKs, timer for retransmission  
- Sender waits for ACK, retransmits on timeout

---

## Step 2: Suppose RTT is known and constant

- Let’s assume the sender knows exactly how long it takes for a packet + ACK to travel  
- Could we avoid a timer?

---

## Step 3: Consider packet loss

Even if RTT is known and constant, the problem is packet loss:

1. If a data packet is lost, no ACK will ever arrive  
2. If the sender does not have a timer, it will wait forever for the ACK  
3. Without a retransmission mechanism triggered by a timer, the receiver never gets the packet, and reliability fails

- The exact RTT only tells the sender how long it should normally wait  
- It cannot replace the need to detect a lost packet

---

## Step 4: Conclusion

- Yes, a timer is still necessary  
- Reason: even with known RTT, lost packets or lost ACKs can occur  
- The timer ensures that the sender can retransmit lost packets to guarantee reliable delivery

---

## ✅ Step 5: Key point

- Known RTT helps set the timer value precisely, reducing unnecessary retransmissions  
- But it cannot eliminate the need for a timer — a timer is the only way to detect loss in rdt 3.0

**12**

Visit the Go-Back-N interactive animation at the companion Web site.  

a. Have the source send five packets, and then pause the animation before any of the five packets reach the destination. Then kill the first packet and resume the animation. Describe what happens.  
b. Repeat the experiment, but now let the first packet reach the destination and kill the first acknowledgment. Describe again what happens.  
c. Finally, try sending six packets. What happens?

> This is all about Go-Back-N (GBN) behavior: sequence numbers, cumulative ACKs, and retransmissions.

---

## (a) Send 5 packets, pause, then kill the first packet

### Scenario
- Sender sends 5 packets: Pkt 0, 1, 2, 3, 4  
- First packet (Pkt 0) is lost before reaching the receiver  
- Resume animation

### GBN behavior
1. **Receiver only accepts in-order packets**  
   - Since Pkt 0 is lost, it cannot deliver Pkt 1–4 yet  
   - Receiver discards out-of-order packets (1–4)  
   - Receiver continues to ACK the last correctly received in-order packet (none, so ACK 0 is sent)

2. **Sender waits for ACK 0**  
   - Timer for Pkt 0 eventually expires  
   - Sender retransmits Pkt 0 and all subsequent packets (Pkt 1–4)

3. **Once Pkt 0 is received**, the receiver accepts it, then cumulatively accepts 1–4 and sends ACKs for the last in-order packet

✅ **Key point:** Losing the first packet triggers retransmission of the entire window.

---

## (b) First packet reaches destination, kill first ACK

### Scenario
- Pkt 0 successfully reaches receiver  
- Receiver sends ACK 0, but it is lost

### GBN behavior
1. Sender does not receive ACK 0 before timer expires  
2. Timer retransmits Pkt 0 and all packets in the window (even if receiver already got them)  
3. Receiver sees duplicate Pkt 0, discards it, but still re-ACKs 0  
4. When retransmissions arrive, receiver accepts in-order packets and updates cumulative ACKs

✅ **Key point:** Lost ACK triggers retransmission of the entire window, but receiver uses sequence numbers to detect duplicates.

---

## (c) Send six packets

### Scenario
- Suppose sender window size = 5 (typical in GBN)  
- Sender tries to send 6 packets

### GBN behavior
1. Sender can only send up to window size packets without waiting for ACK  
2. The sixth packet cannot be sent until ACKs free up space in the window  
3. This enforces flow control and sliding window behavior

✅ **Key point:** GBN restricts sending beyond the window size, ensuring in-order delivery and manageable buffering.

---

## ✅ Summary Table

| Scenario | GBN Behavior |
|-----------|---------------|
| **First packet lost** | Receiver discards all subsequent packets; sender retransmits entire window on timeout |
| **First ACK lost** | Sender retransmits entire window; receiver discards duplicates but re-ACKs cumulative sequence number |
| **Sending more packets than window** | Sixth packet waits until window slides (ACKs free space) |


**13**

Repeat R12, but now with the Selective Repeat interactive animation. How are Selective Repeat and Go-Back-N different?

> This is a classic comparison between GBN and Selective Repeat (SR).

---

## Step 1: Scenario (Selective Repeat)

Same setup as R12, but now using **Selective Repeat**.

---

### (a) First packet lost

**SR behavior:**
1. Sender sends 5 packets: Pkt 0–4  
2. Pkt 0 is lost  
3. Receiver accepts out-of-order packets (Pkt 1–4) and buffers them  
4. Receiver ACKs packets it has received correctly (individual ACKs, not cumulative)  
5. Sender retransmits only the lost packet (Pkt 0) when its timer expires  

✅ **Difference from GBN:**  
- GBN retransmits the entire window  
- SR retransmits only the missing packet  
- SR uses per-packet timers and buffering for out-of-order packets  

---

### (b) First packet reaches, first ACK lost

**SR behavior:**
1. Pkt 0 reaches receiver; ACK 0 is lost  
2. Sender retransmits only Pkt 0 after timer expires  
3. Receiver sees duplicate Pkt 0, ignores it, but resends ACK  
4. Other packets are not affected, since they are already buffered and individually ACKed  

✅ **Key point:**  
SR avoids unnecessary retransmissions of the whole window.

---

### (c) Sending more packets than window

- Same as in GBN: sender cannot send beyond its window size  
- Controlled by the sliding window mechanism  

---

## Step 2: Key Differences Between GBN and SR

| **Feature** | **Go-Back-N (GBN)** | **Selective Repeat (SR)** |
|--------------|----------------------|-----------------------------|
| Out-of-order packets at receiver | Discarded | Buffered |
| ACK type | Cumulative | Individual per packet |
| Retransmission | Entire window from lost packet | Only lost packets |
| Timer | Single timer for oldest unACKed packet | Timer per packet |
| Efficiency | Less efficient on lossy channels | More efficient on lossy channels |

---

## ✅ Summary / Intuition

- **GBN** → simpler, less state, but wasteful when packets are lost  
- **SR** → more complex, requires buffering and per-packet timers, but more efficient by retransmitting only missing packets

# R14

**True or False?**  
a. Host A is sending Host B a large file over a TCP connection. Assume Host B has no data to send Host A. Host B will not send acknowledgments to Host A because Host B cannot piggyback the acknowledgments on data.

---

## Step 1: What is Piggybacking?

- In TCP (and other bidirectional protocols), **piggybacking** means:
  > When a host has data to send in the reverse direction, it includes the acknowledgment (ACK) for received data inside that same data segment — instead of sending a separate ACK.

- **Benefit:** reduces the number of separate packets and lowers protocol overhead.

### Example
- Host A → Host B: sends data segment #1  
- Host B → Host A: sends its own data  
- Host B **piggybacks** the ACK for segment #1 within that outgoing data segment.

---

## Step 2: Analyzing the Statement

> “Host B will not send acknowledgments to Host A because Host B cannot piggyback the acknowledgments on data.”

**Answer:** ❌ False

### Reason:
1. TCP **requires** acknowledgments to ensure reliable delivery.  
2. Even if Host B has no data to send, it will still send **ACK-only** segments.  
3. Piggybacking is merely an **optimization**, not a requirement.

---

## ✅ Step 3: Summary

- **Piggybacking:** combining an ACK with outgoing data in the reverse direction.  
- **If no data exists:** TCP still sends standalone ACKs.  
- **Conclusion:** The statement is **false** — TCP does not depend on piggybacking to send acknowledgments.

## R14 (continued)

**b.** The size of the TCP `rwnd` never changes throughout the duration of the connection.

---

## Step 1: What is `rwnd`?

- `rwnd` = receiver window  
- It is advertised by the receiver to tell the sender how much free buffer space is available for incoming data.  
- Purpose: flow control — prevents the sender from overwhelming the receiver.

---

## Step 2: Does it stay constant?

- **No.**

### Reasons:
1. The receiver buffers incoming data.  
   - As the application reads data from the buffer, space becomes available, and `rwnd` increases.  
   - As new TCP segments arrive and occupy buffer space, `rwnd` decreases.  
2. `rwnd` is dynamic: it depends on buffer usage at the receiver.

#### Example:

| Time | Receiver buffer free space | `rwnd` advertised |
|------|----------------------------|-------------------|
| t1   | 10 KB free                 | 10 KB             |
| t2   | 6 KB free (new data arrives) | 6 KB            |
| t3   | 8 KB free (app reads 2 KB)  | 8 KB             |

---

## ✅ Step 3: Conclusion

- Statement is **false**.  
- The size of TCP `rwnd` changes over time depending on buffer usage.

## R14 (continued)

**c.** Suppose Host A is sending Host B a large file over a TCP connection.  
The number of unacknowledged bytes that A sends cannot exceed the size of the receive buffer.

---

## Step 1: Key Concepts

- TCP flow control prevents the sender from overwhelming the receiver.  
- The receiver advertises **`rwnd` (receiver window)** — the amount of free buffer space available.  
- The sender must ensure that the number of unacknowledged bytes in flight ≤ `rwnd`.

---

## Step 2: What Happens in Practice

1. Host A sends data to Host B.  
2. Host B advertises `rwnd`, reflecting its current free buffer space.  
3. Host A tracks bytes sent but not yet acknowledged.

- If bytes in flight = `rwnd`, Host A **must stop sending** until ACKs arrive.  
- ACKs free up buffer space → Host B increases `rwnd` → Host A resumes sending.

✅ This guarantees that the receiver’s buffer never overflows.

---

## ✅ Step 3: Summary

- **Statement:** “The number of unacknowledged bytes that A sends cannot exceed the size of the receive buffer.”  
- **Answer:** **True.**  
- **Reason:** Enforced by TCP flow control through the advertised `rwnd`.

## R14 (continued)

**d.** Suppose Host A is sending a large file to Host B over a TCP connection.  
If the sequence number for a segment of this connection is *m*, then the sequence number for the subsequent segment will necessarily be *m + 1*.

---

## Step 1: Key Facts About TCP Sequence Numbers

- TCP is **byte-oriented**, not segment-oriented.  
- The **sequence number** in a TCP segment identifies the **first byte** of data carried in that segment.  
- Each segment may contain a variable number of bytes (depending on MSS, congestion window, etc.).

---

## Step 2: Analyzing the Statement

> “If the sequence number for a segment is *m*, then the sequence number for the subsequent segment will necessarily be *m + 1*.”

- **False.**  
- Example: If the first segment carries 100 bytes starting at byte *m*,  
  then the next segment’s first byte will have sequence number *m + 100*.  
- Only if each segment contained exactly **1 byte** would the next sequence number be *m + 1*.

---

## ✅ Step 3: Summary

- TCP sequence numbers count **bytes**, not packets or segments.  
- Next segment’s sequence number = *m* + (number of bytes sent in current segment).  
- **Statement:** False.

## R14 (continued)

**e.** The TCP segment has a field in its header for *rwnd*.

---

## Step 1: What Is `rwnd`?

- `rwnd` = **receiver window**  
- Purpose: **flow control** — it tells the sender how much buffer space is available at the receiver.  
- It ensures the sender never sends more data than the receiver can handle.

---

## Step 2: Where Is `rwnd` in TCP?

- The **TCP header** contains a **Window** field (16 bits).  
- This field is used by the receiver to advertise its current `rwnd` value to the sender.  
- Every acknowledgment segment updates this value, helping the sender adapt its sending rate dynamically.

---

## ✅ Step 3: Conclusion

- **True.**  
- TCP includes a header field for the receiver window (`rwnd`).  
- It enables dynamic, reliable **flow control** between sender and receiver.

## R14 (continued)

**g.** Suppose Host A sends one segment with sequence number 38 and 4 bytes of data over a TCP connection to Host B.  
In this same segment, the acknowledgment number is necessarily 42.

---

## Step 1: Given

- Host A sends:
  - **Sequence number:** 38  
  - **Data length:** 4 bytes  
- Question: Is the acknowledgment number necessarily 42?

---

## Step 2: TCP Acknowledgment Semantics

- TCP **sequence numbers** count **bytes**, not segments.  
- The **acknowledgment number** indicates the **next byte expected** from the *other side*.  
- The ACK field in a segment from **A → B** acknowledges data **received from B**, not the data A is sending.

---

## Step 3: Analysis

- The statement assumes:
  > “Because A is sending bytes 38–41, the ACK must be 42.”

- ❌ **Incorrect.**  
  - That would only make sense in a segment **from B → A**, after B receives bytes 38–41.  
  - The ACK in A’s outgoing segment reflects **what A has received from B**, not what it’s sending now.

---

## ✅ Step 4: Conclusion

- **False.**  
- In TCP, the acknowledgment number always corresponds to **bytes received from the other host**,  
  not the bytes being sent in the same segment.

## R15

Suppose Host A sends two TCP segments back to back to Host B over a TCP connection.  
The first segment has sequence number **90**; the second has sequence number **110**.

---

### a. How much data is in the first segment?

TCP sequence numbers are **byte-oriented**.

---

### Step 1: Given
- First segment sequence number = **90**  
- Second segment sequence number = **110**

TCP sequence numbers count **bytes**, so:

\[
\text{Seq(next)} = \text{Seq(current)} + \text{bytes in current segment}
\]

---

### Step 2: Compute data in first segment

\[
110 - 90 = 20 \text{ bytes}
\]

---

✅ **Answer:**  
The first segment carries **20 bytes of data**.

### R15 (continued)

**b.** Suppose the first segment is lost but the second segment arrives at B.  

- First segment: seq = 90, length = 20 bytes → bytes 90–109  
- Second segment: seq = 110 → bytes 110–??  
- First segment lost, second arrives

---

### Step 1: TCP Acknowledgment Rule

- TCP uses **cumulative ACKs**:  
  > ACK number = next **in-order byte** expected by the receiver.  
- Out-of-order data cannot be acknowledged until all prior bytes are received.

---

### Step 2: Apply to Scenario

- Receiver gets second segment (110–??) but missing first (90–109)  
- First missing byte = 90  
- **ACK sent by B = 90**  

✅ Even though bytes 110+ arrived, TCP cumulatively ACKs **90**.

---

### Step 3: If Only One Segment Is Sent

- Host A sends segment seq = 90, length = 20 bytes  
- Host B receives it in order  
- Next byte expected = 90 + 20 = 110  
- **ACK number sent = 110**

---

### Step 4: How TCP ACK Numbers Work Overall

1. **Cumulative ACK:** acknowledges all bytes up to the first missing byte  
2. **Next byte expected:** ACK number = sequence number of next in-order byte  
3. **Out-of-order data:** not acknowledged until missing bytes arrive

#### Example Timeline

| Received Segment | Bytes      | ACK Sent |
|-----------------|-----------|----------|
| 90–109          | 90–109    | 110      |
| 110–129 (first lost) | N/A  | 90       |
| 110–129 (arrives later) | 110–129 | 110 after 90–109 arrives |

---

## ✅ Summary

- First segment lost → ACK = 90 (waiting for first missing byte)  
- Single in-order segment → ACK = sequence number + length  
- ACK number always = **next byte expected in order**

## R16

Consider the Telnet example. A few seconds after the user types the letter **'C'**, the user types the letter **'R.'**  

- How many segments are sent?  
- What are the sequence and acknowledgment numbers?

---

### Step 1: Telnet Characteristics

- Interactive application, usually **one character per segment**  
- TCP is **full-duplex**, so each side can send data independently  
- Each typed character can trigger a TCP segment (subject to Nagle’s algorithm / OS buffering)

---

### Step 2: Scenario

- User types **'C'** → TCP sends segment carrying **'C'**  
- After a few seconds, user types **'R'** → TCP sends segment carrying **'R'**  

Assumptions:  
1. No data has been sent back from the server yet  
2. Each character triggers its own TCP segment

---

### Step 3: Segments Sent

| Segment | Data | Sequence Number | ACK Number |
|---------|------|----------------|------------|
| 1       | 'C'  | seq0            | ack0       |
| 2       | 'R'  | seq0 + 1        | ack0       |

- Each character → **one TCP segment** (unless Nagle’s algorithm coalesces packets)  
- Sequence numbers increment by **1 per byte**  
- ACK numbers are **cumulative**, acknowledging the last byte received from the server

---

### ✅ Step 4: Notes

1. If the server sends data during this time, ACK numbers will update accordingly  
2. Sequence numbers always **count bytes**; each character = 1 byte  
3. Nagle’s algorithm may delay small segments and combine multiple characters if previous data is unacknowledged

## R17

Suppose two TCP connections share a bottleneck link of rate **R bps**, both sending huge files in the same direction, starting at the same time.  

- What transmission rate would TCP like to give to each connection?

---

### Step 1: Given

- Bottleneck link rate = **R bps**  
- Two TCP connections, long-lived, same direction  
- Both start transmitting simultaneously

---

### Step 2: TCP’s Goal

- TCP uses **congestion control** (slow start, congestion avoidance)  
- In steady state, TCP aims for **fair sharing** of the bottleneck  
- Fair → each connection gets roughly equal bandwidth

---

### Step 3: Target Transmission Rate

\[
\text{Rate per connection} \approx \frac{R}{2} \text{ bps}
\]

- Assumes no other flows and long-lived transmissions  
- Each connection eventually stabilizes at about **R/2 bps**

---

### Step 4: Notes

1. Initially, slow start → rates grow exponentially until first loss  
2. Congestion avoidance → rates converge to fair share (~R/2)  
3. Demonstrates **TCP fairness**: multiple flows sharing a bottleneck converge to equal bandwidth

---

### ✅ Step 5: Summary Table

| Scenario                     | TCP Target Rate per Flow |
|-------------------------------|------------------------|
| 2 long-lived flows, same bottleneck | R/2 bps each          |

- With **n flows**, each would get roughly **R/n**


## R17: Two TCP Connections over a Bottleneck Link

**Question:** Two TCP connections start simultaneously, sending huge files in the same direction over a link with capacity **R**. What transmission rate would TCP like to give each connection?

---

### Step 1: Implicit Assumptions

1. Bottleneck link = only source of congestion; no other losses or cross-traffic.  
2. Huge files → long-lived flows, congestion window stabilizes (steady-state).  
3. Standard TCP (Reno-style) behavior: slow start + congestion avoidance.  
4. “Transmission rate TCP would like to give” refers to **steady-state**, not instantaneous rate.

---

### Step 2: Apply TCP Fairness Principle

- TCP strives for fair sharing among long-lived flows.  
- With **2 flows**, each flow’s average throughput:

\[
\text{Rate per flow} \approx \frac{R}{2}
\]

Assumptions:

- No packet drops from other causes  
- Flows are always backlogged  
- Similar RTTs; otherwise, shorter RTT flows may get slightly more throughput (TCP RTT bias)

---

### Step 3: Caveats / Edge Cases

1. Different RTTs → shorter RTT flow may grab more bandwidth  
2. Slow start → temporary overshoot above R/2 in early RTTs  
3. Small bottleneck buffers → early packet loss → slightly lower than R/2  
4. TCP variant (Reno vs Cubic) affects exact convergence but fairness principle still holds

---

### Step 4: Conclusion

- **Steady-state target rate per connection ≈ R/2**  
- Exact rate may fluctuate due to RTT differences and TCP flavor, but **TCP fairness** ensures roughly equal sharing

## R18: TCP Congestion Control – Timeout and ssthresh

**Question:** True or false? When the timer expires at the sender, the value of **ssthresh** is set to one half of its previous value.

---

### Step 1: TCP Congestion Control Behavior

- TCP uses **slow start (SS)** and **congestion avoidance (CA)**.  
- **ssthresh** = slow start threshold; separates slow start from congestion avoidance.  
- On **timeout** (segment/ACK lost):

1. TCP treats it as a congestion event.  
2. **ssthresh is set to half of the congestion window (cwnd) at the time of loss**, not half of old ssthresh.  
3. **cwnd** is reset to 1 MSS, and TCP enters slow start.

> Subtlety: Many confuse “previous ssthresh” with “cwnd at loss.”

---

### Step 2: Analyze the Statement

> “When the timer expires at the sender, the value of ssthresh is set to one half of its previous value.”

- **False**  
- Reason: TCP sets **ssthresh = cwnd / 2**, not old ssthresh / 2. Previous ssthresh could differ from cwnd.

---

### Step 3: Key Points / Summary

- Timeout → congestion detected  
- **ssthresh = cwnd / 2** (at loss)  
- **cwnd reset to 1 MSS** → slow start  
- Statement about halving previous ssthresh → **false**

## R19: TCP Splitting and Response Time

**Question:** Why is the response time with TCP splitting approximately  
`4×RTTFE + RTTBE + processing time`? What is TCP splitting?

---

### Step 1: What is TCP Splitting?

- **TCP splitting** (or connection splitting) is an optimization for **long-delay networks**, e.g., satellite links.  
- Idea: split one long end-to-end TCP connection into **two shorter TCP connections** through a proxy:

Host A --- TCP --- Proxy/Intermediate --- TCP --- Host B


- Each segment has its **own TCP connection** and congestion control.  
- **Benefit:** reduces slow-start penalty and RTT impact on congestion window growth.

---

### Step 2: Why response time ≈ 4×RTTFE + RTTBE + processing

- **RTTFE** = RTT between **front-end (client) and proxy**  
- **RTTBE** = RTT between **proxy and back-end (server)**  
- **Processing time** = time proxy spends handling the split connection

#### Step 2a: Data flow analysis

1. **Client → Proxy (Front-end TCP)**  
   - Request sent to proxy: RTTFE / 2  
   - Proxy acknowledges: RTTFE / 2  
   - **Handshake + request** = 1 RTTFE

2. **Proxy → Server (Back-end TCP)**  
   - Proxy sends request to server: RTTBE / 2  
   - Server ACKs: RTTBE / 2  
   - **Handshake + request** = 1 RTTBE

3. **Data transfer and ACKs**  
   - TCP front-end: 2 RTTFE for request and first ACK from proxy  
   - TCP back-end: 1 RTTBE for server response  
   - Proxy forwards data back to client: 2 RTTFE

**Total dominant delay terms:**  

Response time ≈ 4 × RTTFE + RTTBE + proxy processing time


> Small details like Nagle’s algorithm, segment sizes, and minor delays are ignored; this captures the major contributions.

---

### Step 3: Why TCP splitting helps

- Without splitting: end-to-end RTT = RTTFE + RTTBE → slow start is limited by large RTT.  
- Splitting: **shorter RTTs per segment** → faster congestion window growth, better throughput for long-delay paths.

---

### Step 4: Summary

- **TCP splitting:** divide one TCP connection into two through a proxy.  
- **Response time formula:** `4×RTTFE + RTTBE + processing time`  
  - 4×RTTFE: front-end handshake, request, and ACKs  
  - RTTBE: back-end handshake and response  
  - Processing: proxy handling time  
- **Assumption:** dominant delays are RTTs; minor factors ignored.  
- **Benefit:** reduces slow-start delay for long RTT paths.
