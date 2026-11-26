# KMOU-WLAN Connected but No Internet | Korea Maritime & Ocean University Wi-Fi

[한국어](README.md)

**The laptop signed in to KMOU-WLAN but could not get an IPv4 address from the campus network. Internet access returned once the network assigned an address.** On November 3, the laptop made several attempts before it received an address offer from the campus network. The capture also shows the network approving the lease. The laptop’s records do not show why no reply reached it on the earlier attempts.

## Check your Windows laptop

While connected to KMOU-WLAN, check two lines in Windows.

1. Open Start, search for `cmd`, open **Command Prompt**, and enter `ipconfig`.
2. Under **Wireless LAN adapter Wi-Fi** (or the name you gave your wireless adapter), find `IPv4 Address` (sometimes `Autoconfiguration IPv4 Address`) and `Default Gateway`.

| What you see | What to do next |
| --- | --- |
| IPv4 starts with `169.254.` and the default gateway is blank | In this case, Windows had assigned itself a temporary address. Try opening a website on the same laptop using a phone hotspot. Reconnect to campus Wi-Fi once, run `ipconfig` again, and try the website there. |
| IPv4 is outside `169.254.x.x` and the default gateway has an IPv4 address | The laptop in this case showed a different result during the failure. Try several websites to see whether the problem affects all of them or just one. |
| You cannot sign in to campus Wi-Fi | Open the university IT site and select **스마트캠퍼스 안내 → 학내 무선네트워크** for campus Wi-Fi setup instructions. [KMOU IT](https://www.kmou.ac.kr/cisc/main.do) |

If the issue persists after reconnecting, note the time and location and select **Help-Desk → 장애접수** on the university IT site. A report can be as simple as the example below, even without a packet capture. [KMOU IT Help Desk menu](https://www.kmou.ac.kr/cisc/main.do)

> At [time] today in [building and floor], I connected to KMOU-WLAN. The IPv4 address starts with 169.254 and the default gateway is blank. Websites load on the same laptop using my phone hotspot. After reconnecting to campus Wi-Fi, I still see the same result.

## What the records show

**November 3, 2026.** The laptop sent several messages looking for an address (`Discover`). The campus network offered one (`Offer`), the laptop asked to use it (`Request`), and the network approved it (`ACK`). This is how the laptop obtained an IP address through DHCP.

![Packet fields in Command Prompt style. The left panel shows repeated Discovers on campus Wi-Fi on November 26. The right shows the Offer, Request, and ACK on November 3.](assets/dhcp-packet-evidence.png)

*Figure 1. The right panel measures **21.84 seconds** from the first Discover the laptop captured (frame 1, 0.000 s) to the first Offer (frame 6, 21.840 s). Those frames have different transaction IDs. Starting from the first Discover with the same transaction ID as the Offer (frame 3, 6.544 s) gives **15.30 seconds**. Measuring server processing time would require server-side records.*

**November 26, 2026 (Korea Standard Time).** Between 14:22 and 14:29, the laptop authenticated at three campus access points. During the intervals on campus Wi-Fi, it sent 15 Discovers in total and received zero Offers. The same laptop obtained an address and reached the web on a phone hotspot.

The laptop later reconnected to the original access point. Windows recorded **successful authentication at 14:42:53.909 and the start of an IP lease at 14:42:58**. A web request over campus Wi-Fi also succeeded (HTTPS 204). Packet capture had ended before this recovery.

![Windows lease details in Command Prompt style showing a campus IP lease starting at 14:42:58 on November 26.](assets/windows-recovery-evidence.png)

*Figure 2. The `ipconfig /all` excerpt shows a ten-hour lease. The IP address, default gateway, and DHCP server address are redacted. Devices normally try to renew before a lease expires. The lease length alone cannot establish whether the campus server was overloaded or running out of addresses.* [DHCP lease renewal specification](https://datatracker.ietf.org/doc/html/rfc2131#section-4.4.5)

<details>
<summary>References for the write-up</summary>

[Google SRE example postmortem](https://sre.google/sre-book/example-postmortem/) · [Cloudflare incident report](https://blog.cloudflare.com/cloudflare-outage-on-july-17-2020/) · [Julia Evans on diagnosing a network problem](https://jvns.ca/blog/2021/11/04/how-do-you-tell-if-a-problem-is-caused-by-dns/)

</details>
