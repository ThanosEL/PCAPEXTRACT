# PCAPEXTRACT

A Bash + `tshark` tool that automates analysis of multiple `.pcap` files — no need to open each one manually in Wireshark.

## Purpose

Scans the current directory for `.pcap` files and, for each one, creates a results folder containing:

- Source/destination IPs and ports
- Domain names resolved via DNS
- HTTP GET requests (flags `curl`-based requests separately — a sign of scripted/automated activity)
- Files transferred over **plain HTTP** (images, pages, etc.), each with a SHA-256 hash
- Hostnames from Kerberos traffic (Active Directory environments only)
- TCP flags per connection (SYN, ACK, FIN, RST) — useful for spotting scanning or unusual behavior

It's a quick **triage** step across many captures, not a Wireshark replacement.

## Dependencies

```bash
sudo apt update
sudo apt install -y tshark wireshark figlet lolcat gawk
```

`grep`, `sort`, and `sha256sum` are standard on any Linux system.

## Files

- **`pcapextract`** — the main script (run this)
- **`pcapextrfunc`** — helper functions (`banner`, `dnsfilter`, `cname`, `hash`), auto-loaded by `pcapextract`. Must stay in the same folder.

## Usage

```bash
chmod +x pcapextract pcapextrfunc
./pcapextract
```

Output lands in `<filename>.pcap.<date>/` next to each `.pcap`.

## Output files

| File | Contents |
|---|---|
| `*.GET.txt` | HTTP GET requests (curl-only, then all) |
| `*.HTTP.txt` | All HTTP traffic |
| `*.IP_PORTS.txt` | Unique source/destination IP + port pairs |
| `*.IPSORTED.txt` | Sorted list of unique IPs |
| `*.TCP_FLAGS.txt` | Per-packet TCP flags, chronological |
| `*.Domain_Names_LIST.txt` | Domains seen in DNS queries |
| `*.HOSTNAMES.txt` | Kerberos CNameString hostnames (AD only — empty otherwise) |
| `*.EXPORT/` | Files transferred over unencrypted HTTP |

## Important: HTTP vs HTTPS

`--export-objects` and the HTTP filters only see **plaintext HTTP**. Most modern web traffic is HTTPS, so you'll typically get nothing but an initial redirect. Test with an HTTP-only site (`neverssl.com`) or your own local server (`python3 -m http.server`) to see real extracted content.
