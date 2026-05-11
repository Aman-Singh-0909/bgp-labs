# Lab — DNSSEC on BIND 🔐

## What is DNS?

DNS (Domain Name System) is like the internet's phone book.
When you type `google.com`, DNS translates it to an IP address like `142.250.x.x`.
Without DNS, you'd have to memorize every website's IP address!

## What is DNSSEC?

Normal DNS has NO security — anyone can fake a DNS answer!
This is called **DNS Spoofing** or **DNS Cache Poisoning**.

DNSSEC (DNS Security Extensions) fixes this by adding digital signatures to every DNS record.
If someone tries to fake a DNS answer, the signature won't match → request REJECTED! ✅

## How DNSSEC Works

## Key Concepts

| Term | Meaning |
|------|---------|
| ZSK | Zone Signing Key — signs all DNS records |
| KSK | Key Signing Key — signs the ZSK itself |
| RRSIG | Resource Record Signature — the actual signature |
| DNSKEY | Public key published in DNS |
| Chain of Trust | Root → TLD → Domain → all verified! |

## Lab Setup

### Requirements
- Kali Linux
- BIND9 (`sudo apt install bind9 bind9utils dnsutils`)

### Step 1 — Install BIND:
```bash
sudo apt install -y bind9 bind9utils bind9-doc dnsutils
sudo systemctl start named
```

### Step 2 — Create zone file:
```bash
sudo nano /etc/bind/db.lab.local
```

```dns
$TTL    604800
@       IN      SOA     ns1.lab.local. admin.lab.local. (
                        2024010101      ; Serial
                        604800          ; Refresh
                        86400           ; Retry
                        2419200         ; Expire
                        604800 )        ; Negative Cache TTL
;
@       IN      NS      ns1.lab.local.
ns1     IN      A       127.0.0.1
www     IN      A       10.0.0.1
mail    IN      A       10.0.0.2
```

### Step 3 — Add zone to BIND:
```bash
sudo nano /etc/bind/named.conf.local
```

### Step 4 — Test basic DNS:
```bash
sudo systemctl restart named
dig @127.0.0.1 www.lab.local
```

### Step 5 — Generate DNSSEC keys:
```bash
sudo mkdir -p /etc/bind/keys
cd /etc/bind/keys

# Generate Zone Signing Key (ZSK)
sudo dnssec-keygen -a ECDSAP256SHA256 -n ZONE lab.local

# Generate Key Signing Key (KSK)
sudo dnssec-keygen -a ECDSAP256SHA256 -n ZONE -f KSK lab.local
```

### Step 6 — Sign the zone:
```bash
# Add public keys to zone file
sudo bash -c 'cat /etc/bind/keys/*.key >> /etc/bind/db.lab.local'

# Sign the zone
cd /etc/bind
sudo dnssec-signzone -A -3 $(head -c 1000 /dev/random | sha1sum | cut -b 1-16) \
  -N INCREMENT -o lab.local -t /etc/bind/db.lab.local
```

### Step 7 — Use signed zone:
```bash
sudo nano /etc/bind/named.conf.local
```

```bash
sudo systemctl restart named
```

### Step 8 — Verify DNSSEC:
```bash
dig @127.0.0.1 www.lab.local +dnssec
```

## Expected Output

> `RRSIG` record = digital signature present = DNSSEC working! ✅

## Key Learning

> DNSSEC protects against DNS spoofing and cache poisoning!
> Every DNS record has a cryptographic signature!
> If someone fakes a DNS answer → signature won't match → REJECTED!
> This is why major domains like .gov and .edu use DNSSEC!
