# AI-Powered 3D Printing

⚠️ **Note:** This document only covers the **installation and setup steps** (OS, OctoPrint, Cura, and environment prep).  
It is **not a full code repository** for the AI webapp. The application code for STL generation, error handling, and Streamlit UI is still under development.

Turn a spoken/typed prompt into a printed object using **Orange Pi (Debian Bookworm) + OctoPrint + Cura CLI + Streamlit + ChatGPT**.

---

## Quick Links

- **Orange Pi OS:** http://www.orangepi.org/html/serviceAndSupport/index.html
- **BalenaEtcher (flash OS to SD card):** https://etcher.balena.io  
- **Ultimaker Cura (Desktop / AppImage):** https://ultimaker.com/software/ultimaker-cura  
- **OpenAI (ChatGPT / API):** https://platform.openai.com  
- **OctoPrint Docs:** https://docs.octoprint.org  

---

## 0) Flash Debian Bookworm to SD

1. Download a Debian/Orange-Pi Debian image.
2. Use **BalenaEtcher** to write the image to the micro-SD card.
3. Insert SD into **Orange Pi Zero LTS** and power on.

> **About default username/password**  
> Debian images often **create a user on first boot** or document a default in their release notes.  
> For vendor images (e.g., Orange Pi’s Debian), the default may differ. **Check the image’s page** and **change the password immediately**:
```bash
passwd
```

Enable SSH (if disabled) and note the board’s IP from your router/DHCP page.

---

## 1) First Boot & System Prep

SSH in (replace with your IP):
```bash
ssh <user>@<BOARD_IP>
```

Update & upgrade:
```bash
sudo apt update && sudo apt -y upgrade
sudo apt -y install python3 python3-venv python3-pip git curl build-essential   avahi-daemon libffi-dev libjpeg-dev zlib1g-dev libusb-1.0-0
```

(Avahi is optional; it lets you reach the board as `hostname.local`.)

---

## 2) Create a Python Virtual Environment (recommended)

```bash
mkdir -p ~/octo && cd ~/octo
python3 -m venv venv
source venv/bin/activate
python -m pip install --upgrade pip wheel
```

---

## 3) Install OctoPrint (in venv)

```bash
pip install octoprint
```

### First run (to generate configs)
```bash
octoprint serve
```
- Default web UI: `http://<BOARD_IP>:5000`
- Follow the on-screen wizard.
- **Add your user to serial groups** (so OctoPrint can talk to the USB device):
```bash
sudo usermod -a -G tty,dialout $USER
# log out/in or reboot after this
```

### Optional: run OctoPrint as a systemd service
```bash
sudo tee /etc/systemd/system/octoprint.service >/dev/null <<'EOF'
[Unit]
Description=OctoPrint
After=network.target

[Service]
Type=simple
User=%i
ExecStart=/home/%i/octo/venv/bin/octoprint serve
Restart=on-failure

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl enable octoprint@${USER}
sudo systemctl start octoprint@${USER}
```

---

## 4) Connect the Printer

- Plug the **3D printer via USB** to the Orange Pi.
- In OctoPrint UI → **Connect**.  
  If port/baud aren’t auto-detected, try `/dev/ttyUSB0` or `/dev/ttyACM0`.

---

## 5) Install Cura

- **Ultimaker Cura (Desktop / AppImage):** https://ultimaker.com/software/ultimaker-cura  

---

## 6) Programmatic Upload to OctoPrint

Get your **OctoPrint API Key** (User menu → *Settings → API*). Example upload & print via `curl`:

```bash
APIKEY=YOUR_API_KEY
HOST=<BOARD_IP>

# Upload G-code
curl -k -H "X-Api-Key: $APIKEY" -F "select=true" -F "print=true"   -F "file=@/path/to/output.gcode"   http://$HOST:5000/api/files/local
```

---

## 7) Streamlit Web App (Voice → Code → STL → G-code → Print)

Run:
```bash
streamlit run app.py --server.port 8501
```

---

## 8) Voice & TTS

- **Input**: `sounddevice`, `whisper` (or any STT of choice).  
- **Output**: `TTS` (e.g., Coqui TTS) or **system audio** greeting.

---

## 9) Web UI Ports

- **OctoPrint:** `http://<BOARD_IP>:5000`  
- **Your Streamlit app:** `http://<BOARD_IP>:8501` *(or your chosen port)*

---

## 10) License / Credits

- OctoPrint © their authors (AGPLv3).  
- Cura & CuraEngine © Ultimaker.  
- Streamlit © Streamlit Inc.  
- ChatGPT/OpenAI © OpenAI.  
- This project glues them together for an experimental **voice→print** pipeline.
