# KMOU-WLAN Connected but No Internet | Korea Maritime & Ocean University Wi-Fi

[한국어](README.md) · [Troubleshooting write-up template](TEMPLATE.md)

My Windows laptop could connect to Korea Maritime & Ocean University (KMOU) Wi-Fi, but the internet still would not load. The network was KMOU-WLAN, and Windows showed a 169.254.x.x address after DHCP failed. A friend had the same symptom on another laptop. The first useful clue was in the DHCP capture: **three access points, 15 Discover packets, and no Offer back to my laptop**. Later that afternoon, the original access point worked again. Here's what I could learn from both moments.

## Where the connection stopped

Windows showed the network as connected, and its logs confirmed that 802.1X authentication had succeeded. But `ipconfig` showed an address in `169.254.0.0/16` and no IPv4 default gateway. Windows had picked a local address after it failed to get one through DHCP. There was no IPv4 route out to the internet.

I tried my phone hotspot with the same laptop. It got a usable address and passed a web check. Then I tried two other access points broadcasting the campus network. The IP request went unanswered on all three. That made a problem limited to one access point less likely, though it didn't tell me which part of the campus path to inspect first.

## What the logs showed

All times are Korea Standard Time (UTC+9). These are my test windows. I didn't measure how many people were affected or how long the campus network had been having trouble.

| When | What I checked | What happened |
| --- | --- | --- |
| Nov 3, 2026 | Campus Wi-Fi connection and IP retry | The first captured Discover preceded the first Offer by 21.84 seconds. A later transaction in that capture completed as Discover → Offer → Request → ACK. |
| Nov 26, 14:22–14:29 | Same campus network across three access points | Authentication succeeded on all three. The capture showed 7, 4, and 4 Discovers, with zero Offers during the school Wi-Fi test windows. |
| After the failure tests | Phone hotspot on the same laptop | DHCP replied, Windows got a usable address, and the web check passed. |
| Nov 26, 14:42 | Reconnect to the original campus access point | Authentication succeeded at 14:42:53.909. Windows recorded a lease obtained at 14:42:58. An HTTPS check later returned 204 over the campus connection. |

![Packet excerpts: repeated Discovers without an Offer on November 26, and a successful DHCP exchange on November 3](assets/dhcp-packet-evidence.png)

*Figure 1. Actual packet fields exported with TShark and rendered in a Command Prompt style. Each panel starts its elapsed time at the first packet. The failed panel shows the first access point. The successful panel is from November 3.*

![Redacted ipconfig output showing a ten-hour IP lease obtained at 14:42:58](assets/windows-recovery-evidence.png)

*Figure 2. The November 26 recovery happened after the packet capture ended. This excerpt shows the lease recorded by ipconfig. The authentication timestamp in the table came from Windows WLAN AutoConfig.*

## Could short leases explain it?

I wondered whether leases were expiring often enough for the campus equipment to have trouble keeping up. On November 3 the lease I saw was 24 hours. After the November 26 recovery it was 10 hours. Shorter leases can increase how often many connected devices renew. A device can usually renew **before** the lease expires and keep the same IP. The 10-hour value alone tells me very little about how busy the server was or how many addresses were available. [RFC 2131 §4.4.5](https://datatracker.ietf.org/doc/html/rfc2131#section-4.4.5)

I'd check **free addresses in the pool** and **Discover-to-Offer latency** during a failure. I'd also compare the request counts and queue or drop metrics. Even a healthy DHCP server can't answer a request that never gets to it, so I'd match a few exchanges at the laptop, the AP uplink or relay, and the server. That's the evidence I'd need to tell address exhaustion, slow processing, and a forwarding problem apart. [Microsoft's DHCP troubleshooting guidance](https://learn.microsoft.com/en-us/troubleshoot/windows-server/networking/troubleshoot-dhcp-guidance)

## What I'd do next time

1. Start with the connection state, IPv4 address, and gateway. Write down the time and time zone.
2. Start a short capture on the affected laptop before reconnecting. In Wireshark, filter for `dhcp` and check which step of the exchange is visible. Try the hotspot on the same device as a control.
3. When the connection comes back, verify it with a real web request. Keep the recovery time separate from the cause. On November 3 a retry coincided with a successful exchange. The same retry still failed on November 26, so I can't credit it as a permanent fix.
4. Give the network team a short failure window and a short success window. Those timestamps are better starting points for joining logs than “the Wi-Fi was slow this afternoon.”

The Windows checks I kept coming back to:

~~~cmd
netsh wlan show interfaces
ipconfig /all
~~~

To read a saved capture without opening Wireshark's GUI:

~~~powershell
tshark -r capture.pcapng -Y dhcp -T fields -e frame.time_relative -e dhcp.option.dhcp
~~~

DHCP types 1, 2, 3, and 5 are Discover, Offer, Request, and ACK. The public figures contain selected fields from the logs.

## What I borrowed for this write-up

- [Google SRE's example postmortem](https://sre.google/sre-book/example-postmortem/) for separating impact, timeline, and follow-up.
- [Cloudflare's July 17, 2020 outage report](https://blog.cloudflare.com/cloudflare-outage-on-july-17-2020/) for timestamped evidence and concrete follow-up.
- [Julia Evans's DNS troubleshooting write-up](https://jvns.ca/blog/2021/11/04/how-do-you-tell-if-a-problem-is-caused-by-dns/) for the habit of checking one step at a time with small experiments.


