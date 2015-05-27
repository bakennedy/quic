# TRANSMISSION CONTROL PROTOCOL

Extracts taken from RFC793: [https://tools.ietf.org/html/rfc793](https://tools.ietf.org/html/rfc793)

## Table of Content

* [Header Format](#headerformat)
* [Terminology](#terminology)
    * [Send Sequence Variables](#sendsequencevars)
    * [Receive Sequence Variables](#receivesequencevars)
    * [Current Segment Variables](#currentsegmentvars)
* [Sequence number](#sequencenumber)
    * [Initial Sequence Number Selection](#isns)
* [Data Communication](#datacommunication)
    * [Retransmission Timeout](#retransmissiontimeout)
    * [The Communication of Urgent Information](#urgent)
    * [Managing the window](#managingwindow)
* [Interfaces](#interfaces)
    * [OPEN](#open)
    * [SEND](#send)
    * [RECEIVE](#receive)
    * [CLOSE](#close)
    * [STATUS](#status)
    * [ABORT](#abort)
    * [TCP-to-User Messages](#tcptouser)
* [Event Processing](#eventprocessing)

## <A name="headerformat"></A> Header Format

__TCP Header Format__

```
  0                   1                   2                   3
  0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
 |             Source Port       |        Destination Port       |
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
 |                         Sequence Number                       |
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
 |                         Acknowledgment Number                 |
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
 |  Data |           |U|A|P|R|S|F|                               |
 | Offset| Reserved  |R|C|S|S|Y|I|           Window              |
 |       |           |G|K|H|T|N|N|                               |
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
 |           Checksum            |       Urgent Pointer          |
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
 |                   Options                     |    Padding    |
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
 |                               data                            |
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+ 
```

* Source Port: 16 bits
    * The source port number.
* Destination Port: 16 bits
    * The destination port number.
* Sequence Number: 32 bits
    * The sequence number of the first data octet in this segment (except when SYN is present). If SYN is present the sequence number is the initial sequence number (ISN) and the first data octet is ISN+1.
* Acknowledgment Number: 32 bits
    * If the ACK control bit is set this field contains the value of the next sequence number the sender of the segment is expecting to receive. Once a connection is established this is always sent.
* Data Offset: 4 bits
    * The number of 32 bit words in the TCP Header. This indicates where the data begins. The TCP header (even one including options) is an integral number of 32 bits long.
* Reserved: 6 bits
    * Reserved for future use. Must be zero.
* Control Bits: 6 bits (from left to right):
    * URG: Urgent Pointer field significant
    * ACK: Acknowledgment field significant
    * PSH: Push Function
    * RST: Reset the connection
    * SYN: Synchronize sequence numbers
    * FIN: No more data from sender
 * Window: 16 bits
    * The number of data octets beginning with the one indicated in the acknowledgment field which the sender of this segment is willing to accept.
* Checksum: 16 bits
    * The checksum field is the 16 bit one’s complement of the one’s complement sum of all 16 bit words in the header and text. If a segment contains an odd number of header and text octets to be checksummed, the last octet is padded on the right with zeros to form a 16 bit word for checksum purposes. The pad is not transmitted as part of the segment. While computing the checksum, the checksum field itself is replaced with zeros.

The checksum also covers a 96 bit pseudo header conceptually
prefixed to the TCP header. This pseudo header contains the Source
 Address, the Destination Address, the Protocol, and TCP length.
 This gives the TCP protection against misrouted segments. This
 information is carried in the Internet Protocol and is transferred
 across the TCP/Network interface in the arguments or results of
 calls by the TCP on the IP.
```
 +--------+--------+--------+--------+
 |           Source Address          |
 +--------+--------+--------+--------+
 |        Destination Address        |
 +--------+--------+--------+--------+
 | zero   |  PTCL  |    TCP Length   |
 +--------+--------+--------+--------+
```
The TCP Length is the TCP header length plus the data length in
 octets (this is not an explicitly transmitted quantity, but is
 computed), and it does not count the 12 octets of the pseudo
 header.

* Urgent Pointer: 16 bits
    * This field communicates the current value of the urgent pointer as a positive offset from the sequence number in this segment. The urgent pointer points to the sequence number of the octet following the urgent data. This field is only be interpreted in segments with the URG control bit set.
* Options: variable
    * Options may occupy space at the end of the TCP header and are a multiple of 8 bits in length. All options are included in the checksum. An option may begin on any octet boundary. There are two cases for the format of an option:
        * Case 1: A single octet of option-kind.
        * Case 2: An octet of option-kind, an octet of option-length, and the actual option-data octets.
    The option-length counts the two octets of option-kind and option-length as well as the option-data octets. Note that the list of options may be shorter than the data offset field might imply. The content of the header beyond the End-of-Option option must be header padding (i.e., zero).

A TCP must implement all options. Currently defined options include (kind indicated in octal):

```
 Kind Length Meaning
 ---- ------ -------
  0     -    End of option list.
  1     -    No-Operation.
  2     4    Maximum Segment Size.
```

__End of Option List__

```
 +--------+
 |00000000|
 +--------+
  Kind=0
```

 This option code indicates the end of the option list. This
 might not coincide with the end of the TCP header according to
 the Data Offset field. This is used at the end of all options,
 not the end of each option, and need only be used if the end of
 the options would not otherwise coincide with the end of the TCP
 header.

__No-Operation__

```
 +--------+
 |00000001|
 +--------+
  Kind=1
```

This option code may be used between options, for example, to
align the beginning of a subsequent option on a word boundary.
There is no guarantee that senders will use this option, so
receivers must be prepared to process options even if they do
not begin on a word boundary.

__Maximum Segment Size__

```
 +--------+--------+---------+--------+
 |00000010|00000100|    max seg size  |
 +--------+--------+---------+--------+
  Kind=2   Length=4
```

## <A name="terminology"></A> Terminology

### <A name="sendsequencevars"></A> Send Sequence Variables

* SND.UNA - send unacknowledged
* SND.NXT - send next
* SND.WND - send window
* SND.UP - send urgent pointer
* SND.WL1 - segment sequence number used for last window update
* SND.WL2 - segment acknowledgment number used for last window update
* ISS - initial send sequence number

__Send Sequence Space__

```
      1          2          3         4
 ----------|----------|----------|----------
        SND.UNA    SND.NXT    SND.UNA
                             +SND.WND
```

1. old sequence numbers which have been acknowledged
2. sequence numbers of unacknowledged data
3. sequence numbers allowed for new data transmission
4. future sequence numbers which are not yet allowed

### <A name="receivesequencevars"></A> Receive Sequence Variables

* RCV.NXT - receive next
* RCV.WND - receive window
* RCV.UP - receive urgent pointer
* IRS - initial receive sequence number

__Receive Sequence Space__

```
      1          2          3
 ----------|----------|----------
        RCV.NXT    RCV.NXT
                  +RCV.WND
```

1. old sequence numbers which have been acknowledged
2. sequence numbers allowed for new reception
3. future sequence numbers which are not yet allowed

### <A name="currentsegmentvars"></A> Current Segment Variables

* SEG.SEQ - segment sequence number
* SEG.ACK - segment acknowledgment number
* SEG.LEN - segment length
* SEG.WND - segment window
* SEG.UP - segment urgent pointer
* SEG.PRC - segment precedence value

## <A name="sequencenumber"></A> Sequence number

In response to sending data the TCP will receive acknowledgments.
The following comparisons are needed to process the acknowledgments.

* SND.UNA = oldest unacknowledged sequence number
* SND.NXT = next sequence number to be sent
* SEG.ACK = acknowledgment from the receiving TCP (next sequence number expected by the receiving TCP)
* SEG.SEQ = first sequence number of a segment
* SEG.LEN = the number of octets occupied by the data in the segment (counting SYN and FIN)
* SEG.SEQ + SEG.LEN - 1 = last sequence number of a segment

A new acknowledgment (called an "acceptable ack"), is one for which the inequality below holds:

* SND.UNA < SEG.ACK =< SND.NXT

A segment on the retransmission queue is fully acknowledged if the sum of its sequence number and length
is less or equal than the acknowledgment value in the incoming segment.

When data is received the following comparisons are needed:

* RCV.NXT = next sequence number expected on an incoming segments, and is the left or lower edge of the receive window
* RCV.NXT+RCV.WND-1 = last sequence number expected on an incoming segment, and is the right or upper edge of the receive window
* SEG.SEQ = first sequence number occupied by the incoming segment
* SEG.SEQ+SEG.LEN-1 = last sequence number occupied by the incoming segment
 
A segment is judged to occupy a portion of valid receive sequence space if

* RCV.NXT =< SEG.SEQ < RCV.NXT+RCV.WND

 or

* RCV.NXT =< SEG.SEQ+SEG.LEN-1 < RCV.NXT+RCV.WND

The first part of this test checks to see if the beginning of the
segment falls in the window, the second part of the test checks to see
if the end of the segment falls in the window; if the segment passes
either part of the test it contains data in the window.
Actually, it is a little more complicated than this. Due to zero
windows and zero length segments, we have four cases for the
acceptability of an incoming segment:

```
 Segment Receive Test
 Length  Window
 ------- ------- -------------------------------------------
    0       0    SEG.SEQ = RCV.NXT
    0      >0    RCV.NXT =< SEG.SEQ < RCV.NXT+RCV.WND
   >0       0    not acceptable
   >0      >0    RCV.NXT =< SEG.SEQ < RCV.NXT+RCV.WND
                 or RCV.NXT =< SEG.SEQ+SEG.LEN-1 < RCV.NXT+RCV.WND
```

Note that when the receive window is zero no segments should be
acceptable except ACK segments. Thus, it is be possible for a TCP to
maintain a zero receive window while transmitting data and receiving
ACKs. However, even when the receive window is zero, a TCP must
process the RST and URG fields of all incoming segments.

We have taken advantage of the numbering scheme to protect certain
control information as well. This is achieved by implicitly including
some control flags in the sequence space so they can be retransmitted
and acknowledged without confusion (i.e., one and only one copy of the
control will be acted upon). Control information is not physically
carried in the segment data space. Consequently, we must adopt rules
for implicitly assigning sequence numbers to control. The SYN and FIN
are the only controls requiring this protection, and these controls
are used only at connection opening and closing. For sequence number
purposes, the SYN is considered to occur before the first actual data
octet of the segment in which it occurs, while the FIN is considered
to occur after the last actual data octet in a segment in which it
occurs. The segment length (SEG.LEN) includes both data and sequence
space occupying controls. When a SYN is present then SEG.SEQ is the
sequence number of the SYN.

### <A name="isns"></A> Initial Sequence Number Selection

When new connections are created,
an initial sequence number (ISN) generator is employed which selects a
new 32 bit ISN. The generator is bound to a (possibly fictitious) 32
bit clock whose low order bit is incremented roughly every 4
microseconds. Thus, the ISN cycles approximately every 4.55 hours.
Since we assume that segments will stay in the network no more than
the Maximum Segment Lifetime (MSL) and that the MSL is less than 4.55
hours we can reasonably assume that ISN’s will be unique.

For each connection there is a send sequence number and a receive
sequence number. The initial send sequence number (ISS) is chosen by
the data sending TCP, and the initial receive sequence number (IRS) is
learned during the connection establishing procedure.

For a connection to be established or initialized, the two TCPs must
synchronize on each other’s initial sequence numbers. This is done in
an exchange of connection establishing segments carrying a control bit
called "SYN" (for synchronize) and the initial sequence numbers. As a
shorthand, segments carrying the SYN bit are also called "SYNs".
Hence, the solution requires a suitable mechanism for picking an
initial sequence number and a slightly involved handshake to exchange
the ISN’s.
The synchronization requires each side to send it’s own initial
sequence number and to receive a confirmation of it in acknowledgment
from the other side. Each side must also receive the other side’s
initial sequence number and send a confirming acknowledgment.

1. A --> B
    * SYN my sequence number is X
2. A <-- B
    * ACK your sequence number is X
    * SYN my sequence number is Y
4. A --> B
    * ACK your sequence number is Y

This is called the three way (or three message) handshake.

A three way handshake is necessary because sequence numbers are not
tied to a global clock in the network, and TCPs may have different
mechanisms for picking the ISN’s. The receiver of the first SYN has
no way of knowing whether the segment was an old delayed one or not,
unless it remembers the last sequence number used on the connection
(which is not always possible), and so it must ask the sender to
verify this SYN. The three way handshake and the advantages of a
clock-driven scheme are discussed in [3].

## <A name="datacommunication"></A> Data Communication

Once the connection is established data is communicated by the
 exchange of segments. Because segments may be lost due to errors
 (checksum test failure), or network congestion, TCP uses
 retransmission (after a timeout) to ensure delivery of every segment.
 Duplicate segments may arrive due to network or TCP retransmission.
 As discussed in the section on sequence numbers the TCP performs
 certain tests on the sequence and acknowledgment numbers in the
 segments to verify their acceptability.

* The sender of data keeps track of the next sequence number to use in the variable SND.NXT.
* The receiver of data keeps track of the next sequence number to expect in the variable RCV.NXT.
* The sender of data keeps track of the oldest unacknowledged sequence number in the variable SND.UNA.
* If the data flow is momentarily idle and all data sent has been acknowledged then the three variables will be equal.
* When the sender creates a segment and transmits it the sender advances SND.NXT.
* When the receiver accepts a segment it advances RCV.NXT and sends an acknowledgment.
* When the data sender receives an acknowledgment it advances SND.UNA.
* The extent to which the values of these variables differ is a measure of the delay in the communication.
* The amount by which the variables are advanced is the length of the data in the segment.
* Note that once in the ESTABLISHED state all segments must carry current acknowledgment information.
* The CLOSE user call implies a push function, as does the FIN control flag in an incoming segment.

### <A name="retransmissiontimeout"></A> Retransmission Timeout

Because of the variability of the networks that compose an
 internetwork system and the wide range of uses of TCP connections the
 retransmission timeout must be dynamically determined. One procedure
 for determining a retransmission time out is given here as an
 illustration.

 An Example Retransmission Timeout Procedure

* Measure the elapsed time between sending a data octet with a particular sequence number and receiving an acknowledgment that
 covers that sequence number (segments sent do not have to match segments received).
    * This measured elapsed time is the Round Trip Time (RTT).
* Next compute a Smoothed Round Trip Time (SRTT) as:
    * SRTT = ( ALPHA * SRTT ) + ((1-ALPHA) * RTT)
* and based on this, compute the retransmission timeout (RTO) as:
    * RTO = min[UBOUND,max[LBOUND,(BETA*SRTT)]]
        * where UBOUND is an upper bound on the timeout (e.g., 1 minute),
        * LBOUND is a lower bound on the timeout (e.g., 1 second),
        * ALPHA is a smoothing factor (e.g., .8 to .9),
        * and BETA is a delay variance factor (e.g., 1.3 to 2.0).

### <A name="urgent"></A> The Communication of Urgent Information

The objective of the TCP urgent mechanism is to allow the sending user
 to stimulate the receiving user to accept some urgent data and to
 permit the receiving TCP to indicate to the receiving user when all
 the currently known urgent data has been received by the user.

This mechanism permits a point in the data stream to be designated as
the end of urgent information. Whenever this point is in advance of
the receive sequence number (RCV.NXT) at the receiving TCP, that TCP
must tell the user to go into "urgent mode"; when the receive sequence
number catches up to the urgent pointer, the TCP must tell user to go
into "normal mode". If the urgent pointer is updated while the user
is in "urgent mode", the update will be invisible to the user.

The method employs a urgent field which is carried in all segments
transmitted. The URG control flag indicates that the urgent field is
meaningful and must be added to the segment sequence number to yield
the urgent pointer. The absence of this flag indicates that there is
no urgent data outstanding.

To send an urgent indication the user must also send at least one data
octet. If the sending user also indicates a push, timely delivery of
the urgent information to the destination process is enhanced.

### <A name="managingwindow"></A> Managing the Window

The window sent in each segment indicates the range of sequence
 numbers the sender of the window (the data receiver) is currently
 prepared to accept. There is an assumption that this is related to
 the currently available data buffer space available for this
 connection.

Indicating a large window encourages transmissions. If more data
 arrives than can be accepted, it will be discarded. This will result
 in excessive retransmissions, adding unnecessarily to the load on the
 network and the TCPs. Indicating a small window may restrict the
 transmission of data to the point of introducing a round trip delay
 between each new segment transmitted.

The mechanisms provided allow a TCP to advertise a large window and to
 subsequently advertise a much smaller window without having accepted
 that much data. This, so called "shrinking the window," is strongly
 discouraged. The robustness principle dictates that TCPs will not
 shrink the window themselves, but will be prepared for such behavior
 on the part of other TCPs.

The sending TCP must be prepared to accept from the user and send at
 least one octet of new data even if the send window is zero. The
 sending TCP must regularly retransmit to the receiving TCP even when
 the window is zero. Two minutes is recommended for the retransmission
 interval when the window is zero. This retransmission is essential to
 guarantee that when either TCP has a zero window the re-opening of the
 window will be reliably reported to the other.

When the receiving TCP has a zero window and a segment arrives it must
 still send an acknowledgment showing its next expected sequence number
 and current window (zero).

The sending TCP packages the data to be transmitted into segments
which fit the current window, and may repackage segments on the
 retransmission queue. Such repackaging is not required, but may be
 helpful.

In a connection with a one-way data flow, the window information will
 be carried in acknowledgment segments that all have the same sequence
 number so there will be no way to reorder them if they arrive out of
 order. This is not a serious problem, but it will allow the window
 information to be on occasion temporarily based on old reports from
 the data receiver. A refinement to avoid this problem is to act on
 the window information from segments that carry the highest
 acknowledgment number (that is segments with acknowledgment number
 equal or greater than the highest previously received).

The window management procedure has significant influence on the
 communication performance. The following comments are suggestions to
 implementers.

__Window Management Suggestions:__

* Allocating a very small window causes data to be transmitted in
 many small segments when better performance is achieved using
 fewer large segments.
* One suggestion for avoiding small windows is for the receiver to
 defer updating a window until the additional allocation is at
 least X percent of the maximum allocation possible for the
 connection (where X might be 20 to 40).
* Another suggestion is for the sender to avoid sending small
 segments by waiting until the window is large enough before
 sending data. If the the user signals a push function then the
 data must be sent even if it is a small segment.
* Note that the acknowledgments should not be delayed or unnecessary
 retransmissions will result. One strategy would be to send an
 acknowledgment when a small segment arrives (with out updating the
 window information), and then to send another acknowledgment with
 new window information when the window is larger.
* The segment sent to probe a zero window may also begin a break up
 of transmitted data into smaller and smaller segments. If a
 segment containing a single data octet sent to probe a zero window
 is accepted, it consumes one octet of the window now available.
 If the sending TCP simply sends as much as it can whenever the
 window is non zero, the transmitted data will be broken into
 alternating big and small segments. As time goes on, occasional
 pauses in the receiver making window allocation available will
 result in breaking the big segments into a small and not quite so
 big pair. And after a while the data transmission will be in
 mostly small segments.
* The suggestion here is that the TCP implementations need to
 actively attempt to combine small window allocations into larger
 windows, since the mechanisms for managing the window tend to lead
 to many small windows in the simplest minded implementations.

## <A name="interfaces"></A> Interfaces

The timeout, if present, permits the caller to set up a timeout
for all data submitted to TCP. If data is not successfully
delivered to the destination within the timeout period, the TCP
will abort the connection. The present global default is five minutes.

### <A name="open"></A> OPEN

```
OPEN(local port, foreign socket, active/passive [, timeout] [, precedence] [, security/compartment] [, options])
-> local connection name
```

### <A name="send"></A> SEND

```
SEND(local connection name, buffer address, byte count, PUSH flag, URGENT flag [,timeout])
```

### <A name="receive"></A> RECEIVE

```
RECEIVE(local connection name, buffer address, byte count)
-> byte count, urgent flag, push flag
```

### <A name="close"></A> CLOSE
```
CLOSE (local connection name)
```

### <A name="status"></A> STATUS

```
STATUS(local connection name) -> status data
```

This is an implementation dependent user command and could be
 excluded without adverse effect. Information returned would
 typically come from the TCB associated with the connection.
 This command returns a data block containing the following
 information:

* local socket,
* foreign socket,
* local connection name,
* receive window,
* send window,
* connection state,
* number of buffers awaiting acknowledgment,
* number of buffers pending receipt,
* urgent state,
* precedence,
* security/compartment,
* and transmission timeout.

### <A name="abort"></A> ABORT

```
ABORT(local connection name)
```

### <A name="tcptouser"></A> TCP-to-User Messages

It is assumed that the operating system environment provides a
 means for the TCP to asynchronously signal the user program. When
 the TCP does signal a user program, certain information is passed
 to the user. Often in the specification the information will be
 an error message. In other cases there will be information
 relating to the completion of processing a SEND or RECEIVE or
 other user call.

The following information is provided:

* Local Connection Name:
    * Always
* Response String:
    * Always
* Buffer Address:
    * SEND & RECEIVE
* Byte count (counts bytes received):
    * RECEIVE
* Push flag:
    * RECEIVE
* Urgent flag:
    * RECEIVE

## <A name="eventprocessing"></A> Event Processing

Events that occur:

* User Calls
    * OPEN
    * SEND
    * RECEIVE
    * CLOSE
    * ABORT
    * STATUS
* Arriving Segments
    * SEGMENT ARRIVES
* Timeouts
    * USER TIMEOUT
    * RETRANSMISSION TIMEOUT
    * TIME-WAIT TIMEOUT
