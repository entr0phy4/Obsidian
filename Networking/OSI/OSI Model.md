The Open Systems Interconnection (OSI) model is a conceptual model created by the International Organization for standardization which enables diverse communication systems to communicate using standard [[Protocols]]. In plain English, the OSI provides a standard for different computers systems to be able to communicate witch each other.
The OSI model can be seen as a universal language for computer networking. It is based on the concept of splitting up a communication system into seven abstract layers, each one stacked upon the last.

- [[7. Application layer]]: Human-computer interaction layer, where application can access the network services.
- [[6. Presentation layer]]: Ensures that data is in a usable format and is where data encryption occurs.
- [[5. Session layer]]: Maintains connections and is responsible for controlling ports and sessions.
- [[4. Transport layer]]: Transmits data using transmission protocols, including TCP and UDP.
- [[3. Network layer]]: Decides which physical path the data will take.
- [[2. Data link layer]]: Defines the format of data on the network.
- [[1. Physical layer]]: Transmits raw bit stream over the physical medium.

### Flows through the OSI model
Mr. Cooper wants to sends Ms. Palmer an email. Mr. cooper composes his message in an email application on his laptop and the hits 'send'. His email application will pass his email message over to the application layer, which will pick a protocol (SMTP) and pass the data along to the presentation layer. The presentation layer will then compress the data and then it will hit the session layer, which will initialize the communication session.

The data will then hit the sender's transportation layer where it will be segmented, then those segments will be broken up into packets at the network layer, which will be broken down even further into frames at the data link layer. The data link layer will then deliver those frames to the physical layer, which will convert the data into a bitstream of 1's and 0's and send it through a physical medium, such as a cable.

Once Ms. Palmer's computer receives the bit stream through a physical medium (such as her WiFi), the data will flow through the same series of layers on her device, but in the opposite order. First the physical layer will convert the bitstream from 1's and 0's into frames that get passed to the data link layer. The data link layer will the reassemble the frames into packets for the network layer. The network layer will the make segments out of the packets for the transport layer, which will reassemble the segments into one piece of data.

The data will then flow into the receiver's session layer, which will pass the data along to the presentation layer and the end the communication session. The presentation layer will the remove the compression and pass the raw data up to the application layer. The application layer will then feed the human-readable data along to Ms. Palmer's email software, which will allow her to read Mr. Cooper's emails on her laptop screen.
