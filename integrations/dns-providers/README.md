# DNS Provider Integrations

Auto-DNS verification allows instant domain verification by automatically creating TXT records via provider APIs. No manual DNS configuration required.

**48 providers supported.**

---

## How It Works

```
1. User adds domain → NS records analyzed → provider detected
2. User saves credentials for their provider (once per account)
3. System creates TXT record automatically via API
4. Verification completes instantly (no propagation wait)
```

---

## Supported Providers

### Global Leaders

| Provider | Auth Type | Rate Limit | Notes |
|----------|-----------|------------|-------|
| [Cloudflare](cloudflare.md) | API Token | 1200/5min | Most common |
| [GoDaddy](godaddy.md) | API Key + Secret | 60/min | Also verifies registrar ownership |
| [AWS Route53](route53.md) | Access Key + Secret | 5/sec | IAM permissions required |
| [Google Cloud DNS](google-cloud.md) | Service Account JSON | 1500/100sec | Project-level access |
| [Azure DNS](azure.md) | Service Principal | 500/5min | Subscription required |
| [DigitalOcean](digitalocean.md) | Personal Access Token | 250/min | Simple setup |

### Major Registrars

| Provider | Auth Type | Rate Limit | Notes |
|----------|-----------|------------|-------|
| [Namecheap](namecheap.md) | API Key + Username | 50/min | IP whitelist required |
| [NameSilo](namesilo.md) | API Key | 1/sec | Very strict limits |
| [Porkbun](porkbun.md) | API Key + Secret | 100/min | Good API |
| [Name.com](name-com.md) | API Token | 100/min | |
| [IONOS](ionos.md) | API Key | 100/min | 1&1 United Internet |
| [OVH](ovh.md) | App Key + Secret + Consumer | 60/min | EU focused |
| [Gandi](gandi.md) | Personal Access Token | 30/min | LiveDNS API |

### EU Regional

| Provider | Auth Type | Rate Limit | Notes |
|----------|-----------|------------|-------|
| [Hetzner](hetzner.md) | API Token | 3600/hour | German |
| [TransIP](transip.md) | Private Key (RSA) | 100/min | Netherlands |
| [INWX](inwx.md) | Username + Password | 60/min | German, XML-RPC |
| [Netcup](netcup.md) | API Key + Password | 100/min | German |
| [Infomaniak](infomaniak.md) | API Token | 60/min | Swiss |
| [Domeneshop](domeneshop.md) | API Token + Secret | 100/min | Norwegian |
| [Scaleway](scaleway.md) | Secret Key | 100/min | French |

### Cloud/Hosting

| Provider | Auth Type | Rate Limit | Notes |
|----------|-----------|------------|-------|
| [Vercel](vercel.md) | API Token | 100/min | |
| [Netlify](netlify.md) | Personal Access Token | 500/min | |
| [Linode](linode.md) | Personal Access Token | 400/min | Akamai |
| [Vultr](vultr.md) | API Key | 30/sec | |
| [Hostinger](hostinger.md) | API Token | 100/min | |
| [Oracle Cloud](oracle.md) | API Key + Fingerprint | 100/min | Complex auth |
| [IBM Cloud](ibm.md) | IAM Token | 100/min | |
| [Tencent Cloud](tencent.md) | Secret ID + Key | 3000/day | China |
| [Alibaba Cloud](alibaba.md) | Access Key + Secret | 500/day | China |

### Managed DNS

| Provider | Auth Type | Rate Limit | Notes |
|----------|-----------|------------|-------|
| [DNSimple](dnsimple.md) | OAuth or API Token | 30/sec | |
| [DNSMadeEasy](dnsmadeeasy.md) | API Key + Secret | 150/5min | HMAC signing |
| [NS1](ns1.md) | API Key | 10/sec | |
| [UltraDNS](ultradns.md) | Username + Password | 100/min | Neustar |
| [Constellix](constellix.md) | API Key + Secret | 100/min | HMAC signing |
| [ClouDNS](cloudns.md) | Auth ID + Password | 100/min | |
| [LuaDNS](luadns.md) | Email + API Key | 30/min | |
| [Akamai Edge](akamai.md) | Client Token + Secret | 100/min | Complex auth |

### Additional

| Provider | Auth Type | Rate Limit | Notes |
|----------|-----------|------------|-------|
| [Hurricane Electric](he.md) | Session + DDNS Key | 10/min | Session-based |
| [PowerDNS](powerdns.md) | API Key | Varies | Self-hosted |
| [Dynu](dynu.md) | API Key | 100/min | Free DNS |
| [Mythic Beasts](mythic-beasts.md) | API Key | 100/min | UK |
| [Selectel](selectel.md) | API Key | 60/min | Russia |
| [Rackspace](rackspace.md) | Username + API Key | 100/min | |
| [Yandex Cloud](yandexcloud.md) | IAM/OAuth Token | 100/min | Russia |
| [Exoscale](exoscale.md) | API Key + Secret | 100/min | Swiss |
| [easyDNS](easydns.md) | API Token + Key | 100/min | |
| [Dyn](dyn.md) | Session-based | 100/min | Oracle |
| [DNSPod](dnspod.md) | Login Token | 100/min | Tencent China |
| [Rage4](rage4.md) | Email + API Key | 100/min | |
| [Zilore](zilore.md) | API Key | 100/min | |
| [eNom](enom.md) | UID + Password | 100/min | Registrar |
| [CentralNic](centralnic.md) | Username + Password | 100/min | Registrar |
| [Hexonet](hexonet.md) | Username + Password | 100/min | Registrar |
| [Dynadot](dynadot.md) | API Key | 100/min | Registrar |
| [Aruba](aruba.md) | API Token | 100/min | Italian |

---

## Credential Security

- All credentials encrypted at rest (AES-256-GCM)
- Credentials scoped per user, not shared
- Minimal permissions requested (DNS edit only)
- Token refresh handled automatically for OAuth providers

---

## Fallback Methods

If API access unavailable:

1. **Manual DNS** - User adds TXT record manually
2. **HTTP File** - User uploads verification file to domain
3. **Email** - Verification sent to admin@domain.com

---

## Adding New Providers

Contact hello@proof.holdings to request additional provider support.
