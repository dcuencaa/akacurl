# Aka Curl - Akamai Edge Testing Tool
Aka Curl is a cross-platform, graphical user interface (GUI) tool designed specifically for testing and troubleshooting Akamai CDN edge environments. It acts as an advanced wrapper around native OS commands (`curl` and `openssl`) to bypass common Python SSL/SNI limitations when spoofing IP addresses.

## Features
* **Intelligent Edge Routing:** Recursively resolves CNAMEs up to 5 levels deep and routes requests to Staging or Production Edge environments based on your selection.
* **Context-Aware Search:** Press `Ctrl+F` (or `Cmd+F` on macOS) to instantly search through lines of headers, JSON bodies, or certificate outputs with real-time highlighting.
* **Native CURL Execution:** Uses your system's native `curl` to guarantee perfect SNI (Server Name Indication) handshakes and supports `HTTP/1.1`, `HTTP/2`, and `HTTP/3`.
* **Advanced OpenSSL Integration:** Fetches the raw SSL/TLS certificate chain directly from the Akamai Edge Server and automatically decodes the PEM data (showing Expiration, SANs, and Issuer) into a readable format.
* **Akamai Pragma Headers:** One-click toggle to inject standard Akamai debug headers (`akamai-x-cache-on`, `akamai-x-get-cache-key`, etc.).
* **Response Body Handling:** View JSON/HTML/Text responses directly in the app, or seamlessly download binary files (images, videos, PDFs) via a native save dialog.
* **Debug Command Generation:** Automatically generates copy-pasteable `curl --resolve` commands for terminal debugging and sharing with your team.

____________________________________________________________________________________________________________
# Aka Curl - Installation Guide

This guide covers the prerequisites and step-by-step installation instructions for getting **Aka Curl** running on macOS, Linux, and Windows.


## Installation Instructions by OS

### macOS
macOS is highly compatible, but Apple's default system Python can sometimes struggle with Tkinter (the required GUI framework). Using Homebrew is highly recommended to ensure a clean, crash-free setup.

1. **Install Python via Homebrew:**
```bash
brew install python python-tk
```

2. **Ensure curl and openssl are available:** (These are pre-installed on macOS, but Homebrew keeps them up to date)
```bash
brew install curl openssl
```

3. **Install the Python requirement:**
```bash
pip3 install dnspython
```

4. **Run Aka Curl:**
Navigate to the folder containing your script and run:
```bash
python3 akacurl.py
```

---

### Linux (Ubuntu/Debian)
Linux systems usually have `curl` and `openssl` pre-installed, but you will likely need to explicitly install the Python Tkinter system library for the GUI to render.

1. **Install System Dependencies:**
Open your terminal and run:
```bash
sudo apt-get update
sudo apt-get install python3 python3-tk python3-pip curl openssl
```

2. **Install the Python requirement:**
```bash
pip3 install dnspython
```

3. **Run Aka Curl:**
Navigate to the folder containing your script and run:
```bash
python3 akacurl.py
```

---

### Windows
Windows 10 and 11 come with `curl.exe` pre-installed, but they **do not** include `openssl`. 

1. **Install Python:** Download the official installer from [python.org](https://www.python.org/downloads/). 
*⚠️ Crucial Step:* During the installation process, you **must** check the boxes for **"Add Python to PATH"** and **"tcl/tk and IDLE"** so the GUI framework is included.

2. **Install OpenSSL:** Aka Curl requires OpenSSL to fetch and decode certificate details. You can install it via one of the following methods:
* **Git for Windows:** If you have Git Bash installed, OpenSSL is already included.
* **Chocolatey:** Run `choco install openssl` in an Administrator PowerShell.
* **Direct Download:** Download a pre-compiled Windows binary from [slproweb.com](https://slproweb.com/products/Win32OpenSSL.html).
* *⚠️ Crucial Step:* Ensure the folder containing `openssl.exe` is added to your Windows System `PATH` environment variable.

3. **Install the Python requirement:** Open Command Prompt or PowerShell and run:
```cmd
pip install dnspython
```

4. **Run Aka Curl:**
Navigate to the folder containing your script and run:
```cmd
python akacurl.py
```

---

## Troubleshooting

### Error: `externally-managed-environment` (macOS / Linux)
If you try to run `pip install dnspython` and receive an error stating the environment is externally managed, this is a modern Python safety feature preventing global package installations. 

**Quick Fix:** Append the break-system-packages flag:
```bash
pip3 install dnspython --break-system-packages
```

**Best Practice Fix (Virtual Environment):**
Isolate your tool's environment to avoid conflicts:
```bash
# 1. Create a virtual environment
python3 -m venv venv

# 2. Activate it
source venv/bin/activate

# 3. Install the requirement safely
pip install dnspython

# 4. Run the tool
python akacurl.py
```

____________________________________________________________________________________________________________

# Aka Curl - User Guide

Welcome to the **Aka Curl** User Guide. This tool simplifies the process of testing Akamai edge configurations, spoofing Edge IPs, validating SSL/TLS handshakes.

---

## 1. Request Settings

### Provide URL
Enter the complete URL you wish to test (e.g., `https://www.example.com/api/v1/data`).

### Spoof IP / Hostname
This field dictates exactly where the request is routed at the network level, overriding standard DNS resolution.
* **Leave Blank (Auto-CNAME):** If left blank, Aka Curl will look up the DNS CNAME for your URL (up to 5 levels deep). If it finds an Akamai CNAME (`edgekey.net`, `edgesuite.net`, `akamaiedge.net`, etc.), it will automatically calculate the correct staging or production map based on your Network selection.
* **Provide an IP:** (e.g., `184.28.164.7`) Directly targets a specific Akamai Edge Server while maintaining the original `Host` header.
* **Provide a Edgehostname:** (e.g., `test.edgekey-staging.net`) Overrides the domain's normal map manually.

### Network & Env
Select the Akamai network at which you want to run your test: Staging or Production.
* **Std TLS:** For `edgesuite.net` or `akamai.net` maps (Standard HTTP/HTTPS).
* **Enhanced TLS:** For `edgekey.net` or `akamaiedge.net` maps (PCI/Secure HTTPS).
* **Staging vs. Production:** The tool automatically adjusts the `.net` map to include or remove `-staging` to match your selection.

### Method & HTTP Version
* **Method:** Choose between `GET`, `POST`, `PUT`, and `HEAD`.
* **Version:** Defaults to `HTTP/1.1`. You can test `HTTP/2` or `HTTP/3` (Note: HTTP/3 requires a local `curl` binary compiled with HTTP/3 support).

---

## 2. Toggles & Options

* **Pragma Headers:** Injects standard Akamai debug headers into your request to expose Cache statuses (`TCP_HIT`, `TCP_MISS`), CP Codes, Cache Keys, PMUSER variables, etc.
* **Save/Show Body:** * If the response is readable (JSON, HTML, text), it will display in the "Response Body" tab.
  * If the response is a binary file (PDF, image, video), the app will prompt you with a native "Save As" dialog to download the file to your computer.
* **Ignore Cert Errors (-k):** Bypasses SSL validation. Essential when spoofing directly to an IP address, as the IP's certificate will likely not match your SNI `Host` header.

---

## 3. Custom Headers & Payload

* **Custom Headers:** Add your own headers separated by commas or newlines (e.g., `Authorization: Bearer token123`). The `Accept-Encoding: gzip` header is included by default but can be removed or overwritten.
* **Payload:** Paste JSON, form data, or raw text to be sent with `POST` or `PUT` requests.

---

## 4. Output & Verification Tabs

Once you click **Send Request**, Aka Curl executes native OS commands (`curl` and `openssl`) in the background and populates the output area.

### Quick Status Indicators
* **Routed To:** Displays exactly where your request was sent. It will tell you if it used an *Auto-CNAME Match*, a *Default Map*, or a *User Override*, along with the resolved IP address.
* **Config / CP Code:** Instantly parses the Akamai `X-Cache-Key` header and highlights the CP code assigned to the traffic.

### Tab 1: Response Headers
Displays the raw HTTP response code and all returned headers. Look here to inspect `X-Cache` (Hit/Miss), CORS headers, and standard Akamai Pragma outputs.

### Tab 2: Response Body
Shows the human-readable payload returned by the edge server. If the payload is binary, this tab will confirm where the file was saved on your local machine.

### Tab 3: Raw Cert / SSL
Executes `openssl s_client` behind the scenes, passing your target domain as the SNI. Displays the raw TLS handshake, negotiated ciphers, and the raw PEM certificate chain returned by the Akamai Edge server.

### Tab 4: Decoded Cert
Takes the raw PEM certificate block from Tab 3 and decodes it using `openssl x509`. This provides a readable view of the certificate's **Expiration Date**, **Issuer**, and **Subject Alternative Names (SANs)** without needing to manually copy/paste the block into a decoder.

### Tab 5: CURL & Debug
A highly valuable tab for sharing bugs with your team or Akamai support. It displays:
* The exact, copy-pasteable native `curl` command (using the `--resolve` flag) that accurately replicates the test in a standard terminal.
* Any `STDERR` output if the `curl` or `openssl` background commands failed to execute properly.

---

## 5. Built-in Search (Context-Aware)

Aka Curl features a context-aware search bar to help you dig through thousands of lines of output.

1. **Open Search:** Press `Ctrl + F` (Windows/Linux) or `Cmd + F` (macOS) to slide open the search bar at the bottom of the window.
2. **Search as you type:** The active tab (Headers, Body, Cert, etc.) will instantly highlight all matching terms in yellow.
3. **Navigate:** Press `Enter` on your keyboard or click the **Next** button to cycle through the matches (the active match will highlight in orange).
4. **Close:** Press `Esc` or click the **Close** button to hide the search bar and clear all highlights.
