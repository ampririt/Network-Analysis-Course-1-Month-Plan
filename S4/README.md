## Session 4 — Application Layer Protocols: HTTP, HTTPS & FTP

**Learning Objectives**: Analyze web traffic. Understand plaintext vs. encrypted protocols. Extract data from HTTP sessions.

### 📖 Lecture (30 min)
- **HTTP**: Methods (GET, POST), status codes (200, 301, 404, 500), headers, cookies
- **HTTPS/TLS**: Why encryption matters, TLS handshake overview, certificate inspection
- **FTP**: Control vs. data channel, plaintext credential risk
- Plaintext vs. encrypted traffic — what analysts can and cannot see
- Live demo: Compare HTTP vs. HTTPS capture in Wireshark

### 🛠️ Hands-on Lab (45 min)

**Lab A — Wireshark: HTTP Deep Dive (25 min)**
1. Open the Kurose & Ross HTTP PCAP (or capture from `http://httpbin.org`)
2. Filter: `http.request.method == "GET"` → inspect request headers
3. Right-click → **Follow HTTP Stream** — read the full request/response
4. Find: User-Agent, Content-Type, Set-Cookie headers
5. **Credential Extraction Exercise**: Open provided PCAP with FTP login
   - Filter: `ftp` → find `USER` and `PASS` commands
   - Discuss: *Why is plaintext authentication dangerous?*
6. Attempt the same with HTTPS — observe that payload is encrypted

**Lab B — Cisco Packet Tracer: Web & FTP Server (20 min)**
1. Add a Server to existing topology → enable HTTP and FTP services
2. Customize the web page content on the server
3. From a PC, open the web browser → navigate to the server IP
4. Switch to **Simulation Mode** → observe:
   - TCP 3-way handshake
   - HTTP GET request and response
5. Use the PC's command prompt → `ftp [server IP]` → log in → transfer a file
6. Observe FTP control channel in simulation mode

### 🎯 Workshop Activity: "File Carving Challenge" (15 min)
- Provide a PCAP containing an image transferred over HTTP
- Students must:
  1. Find the HTTP transfer using `http.content_type contains "image"`
  2. **File → Export Objects → HTTP** → save the image
  3. First student to recover and display the image wins
- Bonus: Find a hidden text file transferred via FTP in the same PCAP

### 📚 Homework
- Kurose & Ross Wireshark Lab: **HTTP**
- Capture your own web browsing traffic for 5 minutes → identify 3 different websites visited and the HTTP status codes returned

---