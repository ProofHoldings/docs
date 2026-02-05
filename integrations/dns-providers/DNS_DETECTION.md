# DNS Provider Detection

> Automatic DNS provider detection via nameserver (NS) record patterns

proof.holdings automatically detects which DNS provider manages a domain by analyzing its nameserver records. This enables intelligent routing to the appropriate API adapter for automated TXT record management.

---

## Overview

| Metric | Count |
|--------|-------|
| **Total Providers Detected** | 156 |
| **With API Adapters** | 55 |
| **With OAuth Support** | 8 |
| **Detection-Only** | 101 |

---

## How Detection Works

1. Query domain's NS records via DNS
2. Match nameserver hostnames against known patterns
3. Return detected provider with API capabilities

```typescript
// Example: Detecting Cloudflare
// NS records: ns1.cloudflare.com, ns2.cloudflare.com
// Pattern: /\.cloudflare\.com$/i → Matched!
// Result: { provider: 'cloudflare', hasApi: true, hasOAuth: true }
```

---

## Provider Categories

### Tier 1: Global Leaders
High market share, priority API implementation

| Provider | Detection Pattern | API | OAuth |
|----------|------------------|-----|-------|
| Cloudflare | `*.cloudflare.com`, `*.cloudflare.net` | ✅ | ✅ |
| GoDaddy | `*.domaincontrol.com`, `*.secureserver.net` | ✅ | ❌ |
| AWS Route 53 | `*.awsdns-*.com/net/org` | ✅ | ❌ |
| Google Cloud DNS | `*.googledomains.com` | ✅ | ✅ |
| Microsoft Azure DNS | `*.azure-dns.com/net/org` | ✅ | ✅ |
| DigitalOcean | `ns*.digitalocean.com` | ✅ | ✅ |

### Tier 2: Major Registrars

| Provider | Detection Pattern | API | OAuth |
|----------|------------------|-----|-------|
| Namecheap | `*.registrar-servers.com` | ✅ | ❌ |
| Namesilo | `ns*.namesilo.com` | ✅ | ❌ |
| Dynadot | `ns*.dynadot.com` | ✅ | ❌ |
| Porkbun | `*.porkbun.com` | ✅ | ❌ |
| Name.com | `ns*.name.com` | ✅ | ❌ |
| Hover | `ns*.hover.com` | ❌ | ❌ |
| Epik | `ns*.epik.com` | ✅ | ❌ |
| 101domain | `ns*.101domain.com` | ✅ | ❌ |
| Internet.bs | `ns*.internet.bs`, `*.topdns.com` | ✅ | ❌ |

### Tier 3: EU Regional Leaders

| Provider | Detection Pattern | API | OAuth |
|----------|------------------|-----|-------|
| IONOS (1&1) | `*.ui-dns.com/de`, `*.ionos.*` | ✅ | ❌ |
| OVH | `*.ovh.net/com` | ✅ | ❌ |
| Hetzner | `*.hetzner.com/de` | ✅ | ❌ |
| Gandi | `*.gandi.net` | ✅ | ❌ |
| Infomaniak | `ns*.infomaniak.com/ch` | ✅ | ❌ |
| TransIP | `ns*.transip.nl/net` | ✅ | ❌ |
| INWX | `ns*.inwx.de/net` | ✅ | ❌ |
| Netcup | `*.netcup.net` | ✅ | ❌ |
| Hosttech | `ns*.hosttech.eu/ch` | ✅ | ❌ |
| Domeneshop | `ns*.domeneshop.no`, `ns*.hyp.net` | ✅ | ❌ |
| Simply.com | `ns*.simply.com` | ✅ | ❌ |
| Loopia | `ns*.loopia.se/com` | ✅ | ❌ |
| Websupport | `ns*.websupport.sk` | ✅ | ❌ |
| WEDOS | `ns*.wedos.cz/net` | ✅ | ❌ |
| Active24 | `ns*.active24.com` | ✅ | ❌ |
| Zone.ee | `ns*.zone.ee` | ✅ | ❌ |
| Versio | `ns*.versio.nl/uk` | ✅ | ❌ |
| Aruba.it | `dns*.aruba.it`, `*.technorail.com` | ✅ | ❌ |

### Tier 4: Cloud & Hosting Providers

| Provider | Detection Pattern | API | OAuth |
|----------|------------------|-----|-------|
| Vercel | `*.vercel-dns.com` | ✅ | ✅ |
| Netlify | `*.nsone.net`, `*.netlify.com` | ✅ | ✅ |
| Linode (Akamai) | `ns*.linode.com` | ✅ | ❌ |
| Vultr | `ns*.vultr.com` | ✅ | ❌ |
| Scaleway | `ns*.scaleway.com` | ✅ | ❌ |
| Exoscale | `ns*.exoscale.com/ch` | ✅ | ❌ |
| Civo | `ns*.civo.com` | ✅ | ❌ |
| Hostinger | `ns*.hostinger.com`, `*.hstgr.io` | ❌ | ❌ |
| Bluehost | `ns*.bluehost.com` | ❌ | ❌ |
| SiteGround | `ns*.siteground.biz/eu` | ❌ | ❌ |
| DreamHost | `ns*.dreamhost.com` | ✅ | ❌ |
| Liquid Web | `ns*.liquidweb.com` | ✅ | ❌ |
| Amazon Lightsail | `ns-*.awsdns-*.*` | ✅ | ❌ |
| Oracle Cloud DNS | `ns*.oraclecloud.net` | ✅ | ❌ |
| IBM Cloud DNS | `ns*.softlayer.com` | ✅ | ❌ |
| Tencent Cloud DNS | `*.dnspod.net`, `ns*.dnsv*.com` | ✅ | ❌ |
| Alibaba Cloud DNS | `dns*.hichina.com`, `ns*.alidns.com` | ✅ | ❌ |
| Baidu Cloud DNS | `ns*.bdydns.com` | ✅ | ❌ |

### Tier 5: Managed DNS Providers

| Provider | Detection Pattern | API | OAuth |
|----------|------------------|-----|-------|
| DNSimple | `ns*.dnsimple.com`, `*.dnsimple-dns.com` | ✅ | ✅ |
| DNS Made Easy | `ns*.dnsmadeeasy.com` | ✅ | ❌ |
| NS1 (IBM) | `dns*.p*.nsone.net` | ✅ | ❌ |
| UltraDNS | `pdns*.ultradns.com/net/org` | ✅ | ❌ |
| Constellix | `ns*.constellix.com` | ✅ | ❌ |
| ClouDNS | `ns*.cloudns.net`, `pns*.cloudns.net` | ✅ | ❌ |
| LuaDNS | `ns*.luadns.net` | ✅ | ❌ |
| easyDNS | `dns*.easydns.com/net/org` | ✅ | ❌ |
| Dyn (Oracle) | `ns*.p*.dynect.net` | ✅ | ❌ |
| Akamai Edge DNS | `a*-*.akam.net` | ✅ | ❌ |
| G-Core Labs | `ns*.gcorelabs.net` | ✅ | ❌ |
| Bunny DNS | `ns*.bunny.net` | ✅ | ❌ |
| StackPath | `ns*.stackpathdns.com` | ✅ | ❌ |
| ArvanCloud | `ns*.arvancdn.ir` | ✅ | ❌ |

### Tier 6: Additional Providers

| Provider | Detection Pattern | API | OAuth |
|----------|------------------|-----|-------|
| all-inkl | `ns*.all-inkl.com` | ✅ | ❌ |
| Alwaysdata | `ns*.alwaysdata.com` | ✅ | ❌ |
| Anexia | `ns*.anexia.net` | ✅ | ❌ |
| AutoDNS | `ns*.autodns*.com` | ✅ | ❌ |
| Azion | `ns*.aziondns.net` | ✅ | ❌ |
| Beget | `ns*.beget.com/ru` | ✅ | ❌ |
| BlueCat | `ns*.bluecat.io` | ✅ | ❌ |
| BookMyName | `ns*.bookmyname.com` | ✅ | ❌ |
| Brandit | `ns*.brandit.com` | ✅ | ❌ |
| Checkdomain | `ns*.checkdomain.de` | ✅ | ❌ |
| ConoHa | `ns*.conoha.io` | ✅ | ❌ |
| Core-Networks | `ns*.core-networks.de` | ✅ | ❌ |
| cPanel/WHM | *(varies by host)* | ✅ | ❌ |
| DDNSS | `ns*.ddnss.de` | ✅ | ❌ |
| Derak Cloud | `ns*.derak.cloud` | ✅ | ❌ |
| deSEC | `ns*.desec.io/org` | ✅ | ❌ |
| DirectAdmin | *(varies by host)* | ✅ | ❌ |
| DNSExit | `ns*.dnsexit.com` | ✅ | ❌ |
| dnsHome.de | `ns*.dnshome.de` | ✅ | ❌ |
| Domain Offensive | `ns*.do.de` | ✅ | ❌ |
| Duck DNS | `ns*.duckdns.org` | ✅ | ❌ |
| Dynu | `ns*.dynu.com` | ✅ | ❌ |
| EdgeCenter | `ns*.edgecenter.ru` | ✅ | ❌ |
| Efficient IP | `ns*.efficientip.com` | ✅ | ❌ |
| freemyip.com | `ns*.freemyip.com` | ✅ | ❌ |
| GleSYS | `ns*.glesys.se` | ✅ | ❌ |
| Hosting.de | `ns*.hosting.de` | ✅ | ❌ |
| Hosting.nl | `ns*.hosting.nl` | ✅ | ❌ |
| http.net | `ns*.http.net` | ✅ | ❌ |
| Huawei Cloud | `ns*.hwclouds-dns.com` | ✅ | ❌ |
| Hurricane Electric | `ns*.he.net` | ✅ | ❌ |
| HyperOne | `ns*.hyperone.com` | ✅ | ❌ |
| IIJ DNS | `ns*.iij.jp` | ✅ | ❌ |
| Infoblox | `ns*.infoblox.com` | ✅ | ❌ |
| IPv64 | `ns*.ipv64.net` | ✅ | ❌ |
| ISPConfig | *(varies by host)* | ✅ | ❌ |
| iwantmyname | `ns*.iwantmyname.net` | ❌ | ❌ |
| JD Cloud | `ns*.jdcloud-dns.com` | ✅ | ❌ |
| Joker.com | `ns*.joker.com` | ✅ | ❌ |
| KeyHelp | *(varies by host)* | ✅ | ❌ |
| Liara | `ns*.liara.ir` | ✅ | ❌ |
| Lima-City | `ns*.lima-city.de` | ✅ | ❌ |
| Mail-in-a-Box | *(self-hosted)* | ✅ | ❌ |
| Metaname | `ns*.metaname.net` | ✅ | ❌ |
| mijn.host | `ns*.mijn.host` | ✅ | ❌ |
| Mittwald | `ns*.mittwald.de` | ✅ | ❌ |
| MyDNS.jp | `ns*.mydns.jp` | ✅ | ❌ |
| Mythic Beasts | `ns*.mythic-beasts.com` | ✅ | ❌ |
| NearlyFreeSpeech.NET | `ns.phx*.nearlyfreespeech.net` | ✅ | ❌ |
| Neodigit | `ns*.neodigit.net` | ✅ | ❌ |
| Nicmanager | `ns*.nicmanager.com` | ✅ | ❌ |
| NIFCLOUD | `ns*.nifty.com` | ✅ | ❌ |
| Njalla | `ns*.njal.la` | ✅ | ❌ |
| Nodion | `ns*.nodion.com` | ✅ | ❌ |
| Plesk | *(varies by host)* | ✅ | ❌ |
| PowerDNS | *(self-hosted)* | ✅ | ❌ |
| Rackspace | `dns*.rackspace.com` | ✅ | ❌ |
| RcodeZero | `ns*.rcodezero.at` | ✅ | ❌ |
| Reg.ru | `ns*.reg.ru` | ✅ | ❌ |
| Regfish | `ns*.regfish.de` | ✅ | ❌ |
| RimuHosting | `ns*.rimuhosting.com` | ✅ | ❌ |
| RU CENTER | `ns*.nic.ru` | ✅ | ❌ |
| Sakura Cloud | `ns*.gslb*.sakura.ne.jp` | ✅ | ❌ |
| Selectel | `ns*.selectel.org` | ✅ | ❌ |
| SelfHost.de | `ns*.selfhost.de` | ✅ | ❌ |
| Servercow | `ns*.servercow.de` | ✅ | ❌ |
| Shellrent | `ns*.shellrent.com` | ✅ | ❌ |
| Sonic | `ns*.sonic.net` | ✅ | ❌ |
| Spaceship | `ns*.spaceship.com` | ✅ | ❌ |
| Technitium | *(self-hosted)* | ✅ | ❌ |
| Timeweb Cloud | `ns*.timeweb.ru/com` | ✅ | ❌ |
| TodayNIC | `ns*.todaynic.com` | ✅ | ❌ |
| UKFast SafeDNS | `ns*.ukfast.net` | ✅ | ❌ |
| Variomedia | `ns*.variomedia.de` | ✅ | ❌ |
| VegaDNS | *(self-hosted)* | ✅ | ❌ |
| VinylDNS | *(self-hosted)* | ✅ | ❌ |
| VK Cloud | `ns*.mcs.mail.ru` | ✅ | ❌ |
| Volcano Engine | `ns*.volcengine.com` | ✅ | ❌ |
| Vscale | `ns*.vscale.io` | ✅ | ❌ |
| Webnames | `ns*.webnames.ca/ru` | ✅ | ❌ |
| West.cn | `ns*.west.cn` | ✅ | ❌ |
| Yandex 360 | `dns*.yandex.net` | ✅ | ❌ |
| Yandex Cloud | `ns*.yandexcloud.net` | ✅ | ❌ |
| ZoneEdit | `ns*.zoneedit.com` | ✅ | ❌ |
| Zonomi | `ns*.zonomi.com` | ✅ | ❌ |
| eNom | `*.name-services.com`, `dns*.enom.com` | ✅ | ❌ |
| CentralNic | `*.centralnic.com/net` | ✅ | ❌ |
| Hexonet | `*.hexonet.net` | ✅ | ❌ |
| DNSPod | `*.dnspod.net`, `*.dnsv*.com` | ✅ | ❌ |
| Rage4 | `*.rage4.com` | ✅ | ❌ |
| Zilore | `*.zilore.com` | ✅ | ❌ |

---

## Detection-Only Providers

These providers can be detected but require manual TXT record setup:

| Provider | Why No API |
|----------|-----------|
| Hover | No public API available |
| Hostinger | API requires enterprise account |
| Bluehost | No DNS API exposed |
| SiteGround | No DNS API exposed |
| iwantmyname | No public API available |

---

## Self-Hosted & Control Panel Providers

These providers have variable NS patterns based on hosting configuration:

| Provider | Notes |
|----------|-------|
| cPanel/WHM | NS depends on hosting provider |
| DirectAdmin | NS depends on hosting provider |
| ISPConfig | NS depends on hosting provider |
| KeyHelp | NS depends on hosting provider |
| Plesk | NS depends on hosting provider |
| Mail-in-a-Box | Self-hosted, user-defined NS |
| PowerDNS | Self-hosted, user-defined NS |
| Technitium | Self-hosted, user-defined NS |
| VegaDNS | Self-hosted, user-defined NS |
| VinylDNS | Self-hosted, user-defined NS |

---

## Unknown Provider Fallback

When no NS pattern matches, the domain is classified as `unknown`. Users can still:

1. **Manual TXT setup** — Add `_proof.{domain}` TXT record manually
2. **Request support** — Contact us to add detection for their provider

---

## Adding New Providers

To request support for a new DNS provider:

1. Open an issue with:
   - Provider name
   - Example NS record hostnames
   - API documentation URL (if available)

2. We'll add:
   - NS detection pattern
   - API adapter (if public API exists)

---

## Related Documentation

- [API Adapters](./README.md) — Full list of supported API integrations
- [Integration Guide](https://proof.holdings/docs/integration) — How to use DNS verification
- [Threat Model](../../THREAT_MODEL.md) — Security considerations

---

*Last updated: 2026-02-04*
