# Chapter 3 Review Questions

## SECTIONS 3.1–3.3

**R1.** Suppose the network layer provides the following service. The network layer in the source host accepts a segment of maximum size 1,200 bytes and a destination host address from the transport layer. The network layer then guarantees to deliver the segment to the transport layer at the destination host. Suppose many network application processes can be running at the destination host.

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


**R1.** Consider a planet where everyone belongs to a family of six, every family lives in its own house, each house has a unique address, and each person in a given house has a unique name. Suppose this planet has a mail service that delivers letters from source house to destination house. The mail service requires that:

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



# R3

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

# R4

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


















