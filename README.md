# Home Lab

My home network and self-hosted services, built to learn networking
and security hands-on.

## Network Diagram

```mermaid
graph TD
    Internet --> Gateway["Xfinity gateway"]
    Gateway --> Eero["eero router"]
    Eero -- "DNS queries" --> PiHole["Pi-hole (Pi Zero 2 W)"]
    Eero --> Desktop["Desktop: Jellyfin + Tailscale"]
    Eero --> Others["Other household devices"]
    Eero --> Phone["My phone"]
    Phone -. "Tailscale" .-> Desktop
```

## Services

### Pi-hole
Pi-hole is a DNS server that blocks ads and trackers for every device on
my network. It runs on a Raspberry Pi Zero 2 W, and my eero router is
set to use it for all DNS lookups.

**How it works:** when a device visits a site, it first asks a DNS server
to turn the name into an address. Pi-hole checks each name against
blocklists (about 78,000 domains). Blocked names get a dead-end answer
(`0.0.0.0`), and everything else is forwarded to a normal DNS server.

**How I tested it:** I looked up a known ad domain twice, once through
Pi-hole and once directly through Cloudflare (1.1.1.1):

    nslookup doubleclick.net
    nslookup doubleclick.net 1.1.1.1

Pi-hole returned `0.0.0.0` (blocked). Cloudflare returned a real address.

<img src="images/pihole-nslookup.png" alt="nslookup through Pi-hole vs Cloudflare" width="600">

**What I learned:** DNS is the first step of almost every connection, so
a DNS server can see and filter traffic for the whole network. Some
devices and browsers bypass it with their own DNS settings.
### Jellyfin
Jellyfin is a self-hosted media server. I run it on my desktop to stream
my own movies and shows to my other devices.

- **Runs on:** desktop PC
- **Access:** locally over home Wi-Fi, and remotely through Tailscale

### Tailscale
Tailscale creates a private network (a VPN) between my devices, so my
phone can reach my desktop from anywhere.

**Why Tailscale instead of port forwarding:** port forwarding exposes a
service to the whole internet, where anyone can scan and attack it.
With Tailscale, only devices signed in to my account can reach Jellyfin,
and I don't open any ports on my router.

**How I tested it:**
1. Installed Tailscale on my desktop and phone and signed in to the same account
2. Confirmed both devices appeared in the Tailscale admin console
3. Turned off Wi-Fi on my phone (cellular only)
4. Opened Jellyfin using the desktop's Tailscale address, and it loaded

<img src="images/jellyfin-over-tailscale.jpg" alt="Jellyfin loaded on my phone over Tailscale" width="300">

**What I learned:** Each device gets its own private 100.x address, and
traffic between my devices is encrypted. That let me reach Jellyfin from
outside my home without opening any ports on my router.

### Wireshark: DNS capture
I captured a DNS lookup on my desktop to see what Pi-hole's answers
look like on the wire. I filtered the capture to a single domain,
`doubleclick.net`.

- DNS queries go over **UDP port 53** from my desktop to the Pi-hole
- My desktop picks a random source port for each lookup
- Each response repeats the query's **transaction ID**, which is how
  the desktop matches answers to questions
- The `A` (IPv4) lookup was answered with `0.0.0.0` and the `AAAA`
  (IPv6) lookup with `::`, so the domain is blocked for both

<img src="images/wireshark-dns.png" alt="Wireshark DNS capture of a blocked domain" width="700">

## Wireshark: JChat traffic capture
I captured traffic from JChat, a chat app I wrote in Java, to see what
it looks like on the network. I ran the server and two clients on my
desktop and captured on the loopback adapter, filtering to the chat
server's port (3000).

Using Wireshark's **Follow TCP Stream**, the whole conversation shows up
as readable text: usernames, room choice, messages, and server replies.

**What this shows:**
- JChat sends everything in **plaintext**. Anyone who can see the
  traffic can read it, which is why real chat apps use TLS
- There's no authentication. The server accepts whatever username a
  client types
- Each client connects over **TCP**, so the capture also shows the
  connection setup (SYN, SYN-ACK, ACK) before any messages are sent

All messages in the screenshot are test messages.

<img src="images/jchat-plaintext.png" alt="Wireshark Follow TCP Stream of a JChat conversation in plaintext" width="700">

**Possible fix:** wrap the sockets in TLS (Java's `SSLSocket` /
`SSLServerSocket`), which would encrypt the stream.


## What I'm learning
-
