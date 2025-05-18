# Install KrakenSDR DOA from Kraken ARM64 install scripts for Raspberry Pi OS 64 bit

Follow this link to the install script section of the KrakenSDR repo and use the ARM 64 bit instructions. This last worked for Raspberry Pi OS on 5-17-25
https://github.com/krakenrf/krakensdr_docs/wiki/09.-VirtualBox,-Docker-Images-and-Install-Scripts#install-scripts

# KrakenSDR DOA Auto-Start on Boot

This guide explains how to configure your Raspberry Pi to automatically launch the KrakenSDR Direction of Arrival (DOA) software on boot using a `systemd` service. This is intended for users performing a manual install rather than using the prebuilt image.

---

## Manual Setup for systemd Service

The following files are created to support automatic startup.

---

### 1. `start.sh`

**Path:** `/home/kg7oow/krakensdr_doa/start.sh`

```bash
#!/bin/bash
source /home/kg7oow/miniforge3/etc/profile.d/conda.sh
conda activate kraken
exec /home/kg7oow/krakensdr_doa/kraken_doa_start.sh
```

Make it executable:
```bash
chmod +x /home/kg7oow/krakensdr_doa/start.sh
```

---

### 2. `kraken_doa_start.sh`

**Path:** `/home/kg7oow/krakensdr_doa/kraken_doa_start.sh`

```bash
#!/bin/bash

source /home/kg7oow/miniforge3/etc/profile.d/conda.sh
conda activate kraken

while getopts c flag
do
    case "${flag}" in
        c) sudo py3clean . ;;
    esac
done

./kraken_doa_stop.sh

cd heimdall_daq_fw/Firmware
sudo env "PATH=$PATH" ./daq_start_sm.sh
sleep 1

cd ../../krakensdr_doa
sudo env "PATH=$PATH" ./gui_run.sh &

if [ -d "../Kraken-to-TAK-Python" ]; then
    echo "TAK Server Installed"
    cd ../Kraken-to-TAK-Python

    if pgrep -f "python.*KrakenToTAK.py" > /dev/null; then
        echo "KrakenToTAK.py is already running."
        exit 0
    else
        echo "Starting KrakenToTAK.py"
        exec python KrakenToTAK.py
    fi
else
    echo "TAK Server NOT Installed"
    exit 1
fi
```

Make it executable:
```bash
chmod +x /home/kg7oow/krakensdr_doa/kraken_doa_start.sh
```

---

### 3. `krakensdr_doa.service`

**Path:** `/etc/systemd/system/krakensdr_doa.service`

```ini
[Unit]
Description=Start KrakenSDR DOA
After=network.target

[Service]
Type=simple
User=kg7oow
WorkingDirectory=/home/kg7oow/krakensdr_doa
ExecStart=/usr/bin/bash /home/kg7oow/krakensdr_doa/start.sh
Restart=always
RestartSec=5
Environment=HOME=/home/kg7oow
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
```

---

## Enabling and Starting the Service

After placing all files correctly and ensuring permissions:

```bash
sudo systemctl daemon-reload
sudo systemctl enable krakensdr_doa
sudo systemctl start krakensdr_doa
```

---

## Verifying the Service

Check that it is running:

```bash
systemctl status krakensdr_doa
```

View live logs:

```bash
journalctl -u krakensdr_doa -f
```

---

This configuration ensures that the KrakenSDR DOA software (including the GUI and TAK server) starts automatically on system boot and stays running under supervision by `systemd`.

