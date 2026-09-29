# 🚀 DoH Proxy Pro - Cloudflare

A personal DoH (DNS over HTTPS) proxy that forwards only to Cloudflare's own resolver, with automatic failover, caching, and privacy hardening - completely free!

> **About this fork:** unlike upstream, which races queries across 190+ third-party resolvers, this fork forwards each query to a single Cloudflare endpoint (`PARALLEL_RACING_COUNT = 1`) and keeps only Cloudflare's own unfiltered resolvers. Cloudflare already hosts the proxy and sees the queries anyway, so this adds no extra party that can see your DNS traffic, and an untrusted resolver can never "win the race".

[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Cloudflare Workers](https://img.shields.io/badge/Cloudflare-Workers-orange.svg)](https://workers.cloudflare.com/)
[![Cloudflare Pages](https://img.shields.io/badge/Cloudflare-Pages-green.svg)](https://pages.cloudflare.com/)

## 📖 About the Project

DoH Proxy Pro is an advanced DNS over HTTPS service built with Cloudflare Workers and Pages that lets you fully encrypt your DNS queries with the highest level of security and speed.

### ✨ Advanced Features

#### ☁️ Cloudflare-Only, Single-Upstream Forwarding
- Each query is forwarded to **one** Cloudflare resolver endpoint
- The next endpoint is tried only if that one fails
- No third-party resolvers ever see your queries
- Consistent answers (no mixing of filtered and unfiltered resolvers)

#### 🔌 Circuit Breaker Pattern
- Automatically detects unhealthy servers
- Temporarily cuts off servers with more than 5 consecutive failures
- Automatic recovery after 60 seconds
- Three states: Closed, Open, Half-Open

#### 🧠 Endpoint Scoring
- Tracks each endpoint's health, speed, and reliability
- The best-scoring endpoint becomes the primary
- Scoring: **35% health + 30% speed + 20% reliability + 15% region** (all endpoints are global, so region has no effect)

#### 🌐 Upstream Endpoints (all Cloudflare, unfiltered)
- `cloudflare-dns.com` (primary)
- `1.1.1.1` and `1.0.0.1`
- `mozilla.cloudflare-dns.com`
- `brave.cloudflare-dns.com`
- Upstream's other ~190 resolvers remain in the source, commented out

#### 🔒 Advanced Privacy and Security
- **DNS Padding (RFC 8467)**: A real, standard-compliant implementation with a complete OPT Record to prevent traffic analysis
- **Advanced ECS Stripping**: Genuinely parses and removes EDNS Client Subnet from the OPT Record to prevent IP leakage
- **Enhanced Header Randomization**: Randomizes User-Agent and Accept, and randomly adds one extra header (X-Request-ID, X-Client-Version, Accept-Language, Sec-CH-UA) to each request
- **Random X-Forwarded-For**: Occasionally adds a random IP to the headers to reduce traceability

#### 🛡️ Anti-Censorship and Error Handling
- Automatic **Health Check** every 90 seconds
- **Circuit Breaker** for failure management
- **Intelligent Fallback** to the next Cloudflare endpoint when the primary fails
- **Request Coalescing**: Intelligently merges duplicate concurrent requests to reduce load and latency

#### ⚡ High Performance and Efficiency
- **Smart LRU Cache** with automatic TTL and intelligent management (8000 entries)
- **Negative Caching** for NXDOMAIN responses (300s TTL, 2000 entries)
- **Adaptive Timeouts** based on each server's average response time
- **Concurrent Request Management** with a limit of 150 simultaneous requests
- **Advanced Rate Limiting** (200 requests per minute per IP)
- **FNV-1a Cache Key**: A stronger hash algorithm for the cache key that ignores the Transaction ID
- Uses Cloudflare's global CDN network

#### 🌐 Advanced Compatibility
- **CORS Support**: Full support for cross-origin requests for direct use from the browser
- **JSON DoH API**: Supports the `application/dns-json` format and the `?name=domain&type=A` parameter for compatibility with a wider range of clients

#### 📊 Monitoring and Statistics
- **Real-time Stats Page**
- Displays the Top 15 active servers
- Live stats: number of servers, healthy servers, average health, total requests
- Displays success rate, response time, and health for each server
- Responsive design for mobile
- Scrollable table with a sticky header

## 🎭 Two Deployment Methods

### 1️⃣ Cloudflare Workers

**Suitable for:** Fast, standalone deployment

**Advantages:**
- Faster setup
- Easier management
- No GitHub required

**Required file:** [`worker.js`](https://github.com/4n0nymou3/cloudflare-doh-proxy/blob/main/manual-worker/worker.js)

### 2️⃣ Cloudflare Pages

**Suitable for:** Connecting to GitHub with automatic updates

**Advantages:**
- Direct connection to GitHub
- Automatic updates on every push
- CI/CD capability

**Required file:** [`[[path]].js`](https://github.com/4n0nymou3/cloudflare-doh-proxy/blob/main/functions/%5B%5Bpath%5D%5D.js) in the `functions/` folder

## 🚀 Installation Guide

### Method 1: Cloudflare [Workers](https://github.com/4n0nymou3/cloudflare-doh-proxy/blob/main/manual-worker/worker.js) (recommended for beginners)

#### Step 1: Create a Worker

1. Go to [dash.cloudflare.com](https://dash.cloudflare.com) and sign in
2. Select **Workers & Pages** from the left menu
3. Click **Create Application**
4. Select **Create Worker**
5. Choose a name for the Worker (e.g. `my-doh-proxy`)
6. Click **Deploy**

#### Step 2: Copy the Code

1. Click **Edit Code**
2. Delete all the default code
3. Copy the contents of [`worker.js`](https://github.com/4n0nymou3/cloudflare-doh-proxy/blob/main/manual-worker/worker.js) and paste it in
4. Click **Save and Deploy**

#### Step 3: Get the URL

After deploying, your service URL is:

```
https://your-worker-name.your-subdomain.workers.dev/dns-query
```

### Method 2: Cloudflare [Pages](https://github.com/4n0nymou3/cloudflare-doh-proxy/blob/main/functions/%5B%5Bpath%5D%5D.js) (recommended for developers)

#### Step 1: Prepare the Repository

1. Fork this repository or create a new one
2. Folder structure:
```
your-repository/
├── functions/
│   └── [[path]].js
└── README.md
```
3. Place the [`[[path]].js`](https://github.com/4n0nymou3/cloudflare-doh-proxy/blob/main/functions/%5B%5Bpath%5D%5D.js) file in the `functions/` folder

#### Step 2: Connect to Cloudflare Pages

1. Go to [dash.cloudflare.com](https://dash.cloudflare.com) and sign in
2. Select **Workers & Pages** from the left menu
3. Click **Create Application**
4. Select **Pages**
5. Click **Connect to Git**
6. Select your repository
7. In Build settings:
   - Framework preset: **None**
   - Build command: leave empty
   - Build output directory: leave empty
8. Click **Save and Deploy**

#### Step 3: Get the URL

After deploying, your service URL is:

```
https://your-page-name.pages.dev/dns-query
```

## 📱 Usage Guide

### 🌐 Browsers

#### Firefox

```
Settings → Privacy & Security → DNS over HTTPS
→ Choose provider: Custom
→ Enter the URL
```

**Enable ECH for extra security:**
1. In the address bar: `about:config`
2. Search for: `network.dns.echconfig.enabled`
3. Set the value to `true`

#### Chrome / Edge / Brave

```
Settings → Privacy and security → Security
→ Use secure DNS → Custom
→ Enter the URL
```

### 📱 Mobile

#### Android (Intra app)

1. Install [Intra](https://play.google.com/store/apps/details?id=app.intra) from Google Play
2. Open the app
3. Tap **Configure custom server URL**
4. Enter your URL: `https://your-domain/dns-query`
5. Turn on the **ON** switch

#### iOS, iPadOS, and macOS

**Automatic profile download:**

1. Go to your service address (without `/dns-query`)
2. Click the **🍎 Download iOS/macOS Profile** button
3. The `.mobileconfig` file is downloaded

**Installing on iOS/iPadOS:**
```
Safari → download the file
Settings → General → VPN, DNS & Device Management
→ Downloaded Profile → Install
```

**Installing on macOS:**
```
Download the file
System Settings → Privacy & Security → Profiles
→ Install the profile
```

### 🔧 Xray Clients

#### Simple config (DoH only)

To use DoH in v2rayNG and similar clients:

1. Go to the service's main page (without `/dns-query`)
2. Copy the simple config
3. Import it into your client

**Capability:** DNS encryption

#### Fragment config (recommended)

To bypass more advanced filtering:

1. Go to the service's main page
2. Copy the Fragment config
3. Import it into your client

**Capabilities:**
- DNS encryption
- Fragment for bypassing DPI
- Splitting the TLS Hello
- Mixed port (SOCKS5 and HTTP on a single shared port: 10808)

### 💻 Desktop

#### Windows 10/11

```
Settings → Network & Internet → Properties
→ DNS server assignment → Edit
→ Preferred DNS encryption: Encrypted only (DNS over HTTPS)
→ Enter the URL
```

#### Linux (systemd-resolved)

1. Edit the config file:
```bash
sudo nano /etc/systemd/resolved.conf
```

2. Add these lines:
```ini
[Resolve]
DNS=https://your-domain/dns-query
DNSOverTLS=yes
```

3. Restart the service:
```bash
sudo systemctl restart systemd-resolved
```

#### macOS

Use the iOS method (download the profile)

### 🔧 Router

If your router supports DoH:

```
DNS settings → DoH/DNS over HTTPS
→ Enter your service URL
```

**Advantage:** Every device connected to the network uses secure DNS

## 📊 Viewing Live Statistics

To view real-time server statistics:

```
https://your-domain/stats
```

**Information available:**
- Total number of endpoints (5)
- Number of healthy servers
- Average health of the whole system
- Total number of requests
- Endpoint table with:
  - Rank and server name
  - Geographic region
  - Success rate
  - Average response time
  - Health (with a graphical bar)

## 🧪 Testing the Service

### Method 1: Browser

Go to the service's main address (without `/dns-query`):

```
https://your-domain
```

If the page displays with a green "Pro" badge and the status "Active and ready", the service is working correctly.

### Method 2: Stats Page

```
https://your-domain/stats
```

Displays server status, cache, and request-count information.

### Method 3: cURL

```bash
curl -H 'accept: application/dns-json' \
  'https://your-domain/dns-query?name=google.com&type=A'
```

## ⚙️ Advanced Settings

### Changing the number of concurrent upstream requests

This fork uses `1` (single-upstream forwarding). Raising it re-enables upstream's parallel racing.

```javascript
const PARALLEL_RACING_COUNT = 1;
```

### Changing timeouts

```javascript
const RACE_TIMEOUT = 4000;
const FALLBACK_TIMEOUT = 3000;
```

### Changing the Circuit Breaker

```javascript
const CIRCUIT_BREAKER_THRESHOLD = 5;
const CIRCUIT_BREAKER_TIMEOUT = 60000;
```

### Changing the Rate Limit

```javascript
const RATE_LIMIT_REQUESTS = 200;
const RATE_LIMIT_WINDOW = 60000;
```

### Changing the Cache TTL

```javascript
const DNS_CACHE_TTL_MIN = 60;
const DNS_CACHE_TTL_MAX = 3600;
const DNS_CACHE_TTL_DEFAULT = 300;
```

### Changing the Negative Cache TTL

```javascript
const NEGATIVE_CACHE_TTL = 300;
```

### Changing the random X-Forwarded-For probability

```javascript
if (Math.random() < 0.25) {
```

### Enabling/disabling privacy features

```javascript
const DNS_PADDING_ENABLED = true;
const ECS_STRIPPING_ENABLED = true;
```

### Changing the cache size

```javascript
// Main cache size
if (dnsCache.size > 8000) {
    // Evict 2000 oldest entries
}

// Negative cache size
if (negativeDnsCache.size > 2000) {
    // Evict 500 oldest entries
}
```

## 📊 Limits

### Cloudflare Workers Free Plan:

- ✅ **100,000** requests per day
- ✅ **10ms** CPU time per request
- ✅ No bandwidth limit

### Cloudflare Pages Free Plan:

- ✅ **Unlimited** requests
- ✅ **500** builds per month
- ✅ No bandwidth limit

**These values are more than enough for personal use and even for a small organization!**

## 💡 Understanding Filtering Types

### 1. DNS Filtering ✅
- The site is blocked at the DNS level
- **This DoH Proxy bypasses this type of filtering**

### 2. SNI Filtering ⚠️
- The site is blocked based on Server Name Indication
- Requires ECH or additional tooling

### 3. IP Blocking ❌
- The server's IP address is blocked
- Requires a VPN

### 4. Deep Packet Inspection (DPI) ⚠️
- Deep inspection of packet contents
- The Fragment config can help
- Fully bypassing it requires a VPN

## ❓ Frequently Asked Questions

### Is this service a VPN?
No. This service only encrypts DNS queries and is not a replacement for a VPN.

### Which sites become accessible?
Sites that are filtered only by DNS. For other cases you need a VPN.

### Why forward only to Cloudflare?
Cloudflare already terminates TLS for this proxy, so it can see your queries regardless. Forwarding only to Cloudflare's own resolver means no additional party sees your DNS traffic. Racing many third-party resolvers would expose every query to all of them, and the fastest — not the most trustworthy — answer would win.

### How is this different from using 1.1.1.1 directly?
It uses the same resolver, but through your own domain — useful where `cloudflare-dns.com` itself is blocked — and adds caching, padding, ECS stripping, and automatic failover between Cloudflare endpoints.

### What is the Circuit Breaker?
A mechanism for automatically detecting unhealthy servers. If a server fails 5 times in a row, it is taken out of rotation for 60 seconds and then tested again.

### How is the primary endpoint chosen?
The system records each endpoint's performance (speed, success, reliability) and uses the best-scoring one as the primary. Scoring: 35% health + 30% speed + 20% reliability + 15% geographic region.

### What is DNS Padding?
A technique compliant with RFC 8467 that adds a standard OPT Record with a Padding Option (code 12) to the query in order to prevent traffic analysis and usage-pattern detection. A real, complete implementation of this standard guarantees that all upstream servers accept it.

### What is ECS Stripping?
Genuinely parsing and removing EDNS Client Subnet from the OPT Record in queries, which prevents your IP information from leaking to DNS servers. This implementation parses the binary structure of the DNS message and precisely identifies and removes option code 8.

### What is Negative Caching?
Caching NXDOMAIN responses (domain does not exist) for 300 seconds to prevent repeated queries for invalid domains.

### What is an Adaptive Timeout?
Based on each server's average response time, the system dynamically adjusts the timeout: `min(baseTimeout, max(1000ms, avgResponseTime * 3))`

### What is Fragment?
A technique for splitting TLS Hello packets that prevents detection by DPI.

### Does internet speed decrease?
No — it may actually increase. The proxy runs at the Cloudflare edge right next to Cloudflare's resolver, and Smart Caching and Adaptive Timeouts keep it fast.

### What is Request Coalescing?
When several users or apps query the same domain at the same moment, instead of sending several separate requests upstream, the system sends one request and shares the response among all of them. This reduces server load and latency.

### What is Enhanced Header Randomization?
Advanced randomization of HTTP headers (User-Agent, Accept, X-Request-ID, X-Client-Version, Accept-Language, Sec-CH-UA) to prevent fingerprinting and identification.

### What is CORS Support and how does it help?
Full support for Cross-Origin Resource Sharing, which lets browsers and web applications communicate directly with this DoH Proxy without the same-origin restriction. This provides compatibility with a wider range of clients and tools.

### What is the JSON DoH API?
Support for the `application/dns-json` format, which enables queries in the form `?name=domain&type=A`. This format is very useful for clients that cannot construct binary DNS messages, and it increases the service's compatibility with more tools.

### Is this service free?
Yes, completely free and with no traffic limit.

### How can I see the server statistics?
Go to `/stats`. A real-time page with complete server information is displayed.

## 🛡️ Security Recommendations

### Scenario 1: DNS filtering only
✅ Using this DoH Proxy is enough

### Scenario 2: More advanced filtering
✅ Use the DoH Proxy  
✅ Enable ECH in the browser  
✅ Use the Fragment config  
✅ VPN for the other layers

### General Tips
- Use up-to-date browsers
- Keep HTTPS enabled at all times
- Use reputable security software
- Use strong passwords

## 🔬 Technical Architecture

### Endpoint Scoring Algorithm

```
Score = (Health × 0.35) + (Speed × 0.30) + (Reliability × 0.20) + (Region × 0.15) - Freshness_Penalty

Health Score: 0-100 (with a 12-point penalty for each consecutive failure)
Speed Score: 100 - (avgResponseTime / 40)
Reliability Score: (successCount / totalRequests) × 100
Region Score: 100 (matching region) | 75 (Global) | 50 (other regions)
Freshness Penalty: min(15, timeSinceLastCheck / 12000)
```

### Cache Management

**Main DNS Cache:**
- Size: 8000 entries
- TTL: 60-3600 seconds (extracted from the DNS response)
- Eviction: LRU, removing the 2000 oldest entries when full
- Cache Key: FNV-1a hash algorithm, ignoring the Transaction ID

**Negative Cache:**
- Size: 2000 entries
- TTL: 300 seconds (fixed)
- Eviction: removes the 500 oldest entries when full

### Circuit Breaker States

```
CLOSED → (5 failures) → OPEN → (60s timeout) → HALF-OPEN → (success) → CLOSED
                                              ↓ (failure)
                                            OPEN
```

### Health Check Cycle

```
Interval: 90 seconds
Concurrent Checks: 12 servers
Test Query: example.com A Record
Timeout: 2500ms
```

## 🔗 Useful Links

- [Cloudflare Workers documentation](https://developers.cloudflare.com/workers/)
- [Cloudflare Pages documentation](https://developers.cloudflare.com/pages/)
- [RFC 8484 - DNS over HTTPS](https://datatracker.ietf.org/doc/html/rfc8484)
- [RFC 8467 - DNS Padding](https://datatracker.ietf.org/doc/html/rfc8467)
- [Cloudflare DNS](https://1.1.1.1/)
- [Intra app](https://getintra.org/)

## 🧰 Helper Tool for Forkers: Automatic Fragment Config Updates

If you fork this repository, there is also an optional helper tool under `.github/` that is unrelated to the core DoH Proxy: whenever you find a new Fragment config (for example from another source), just replace the contents of `configs/fragment-draft.json` with it and commit. A GitHub Action then — fully automatically and without any external API or key — strips any personal or promotional information from it, validates it technically, and saves and commits the final version directly to `configs/doh-proxy-fragment.template.json` (the same file the web panel always reads from).

## 📝 License

This project is released under the MIT license. For more information see the [LICENSE](LICENSE) file.

## 👨‍💻 Author

Designed and developed by: [Anonymous](https://t.me/An0nymou3Bot)

---

⭐ If this project was useful to you, give it a star!

🔒 For a free and secure internet
