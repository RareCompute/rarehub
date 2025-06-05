

# Install JupyterHub on Raspberry Pi 5 with AI Hat

## 1. Prepare Python Environment
### Steps...

```markdown
# 🧪 Deploy JupyterHub on Raspberry Pi 5 with AI Hat (26 TOPS)

*A multi-user Jupyter environment for home data science with hardware acceleration*

---

## 🧰 1. Prepare Python Environment

### 🐍 Ensure Python 3 is Default
```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install Python 3 and pip
sudo apt install -y python3 python3-pip

# Make `python` point to Python 3
sudo update-alternatives --install /usr/bin/python python /usr/bin/python3 1

# Verify Python version
python --version  # Should return Python 3.x
```

---

## 🧩 2. Install JupyterHub & JupyterLab

### ⚙️ Install Core Components
```bash
# Install proxy (required for routing)
sudo apt install -y npm
sudo npm install -g configurable-http-proxy

# Install JupyterHub and JupyterLab system-wide
sudo -H pip3 install notebook jupyterhub jupyterlab
```

### 🖥️ Enable JupyterLab Interface
```bash
sudo jupyter server extension enable --py jupyterlab --system
```

---

## ⚙️ 3. Configure JupyterHub

### 📝 Generate Config File
```bash
# Create config directory
sudo mkdir -p /etc/jupyterhub

# Generate config
sudo jupyterhub --generate-config -f /etc/jupyterhub/jupyterhub_config.py
```

### 🛠️ Edit Config (via `sudo nano /etc/jupyterhub/jupyterhub_config.py`)
```python
# Set bind URL (change port if needed)
c.JupyterHub.bind_url = 'http://:8888'

# Use JupyterLab by default
c.Spawner.default_url = '/lab'

# Allow users to run notebooks (important for multi-user setup)
c.Spawner.options_form = None  # Simplifies login for non-technical users
```

---

## ⚡ 4. Set Up as System Service

### 📁 Create Service File
```bash
sudo nano /etc/systemd/system/jupyterhub.service
```

### 📄 Paste This Service Template
```ini
[Unit]
Description=JupyterHub Service
After=network.target

[Service]
User=pi  # Run as non-root user (replace with your actual username)
ExecStart=/usr/local/bin/jupyterhub --config=/etc/jupyterhub/jupyterhub_config.py
Restart=always
Environment="PATH=/usr/local/bin:/usr/bin:/bin:/home/pi/.local/bin"  # Adjust for user-specific paths

[Install]
WantedBy=multi-user.target
```

### 🔁 Enable & Start Service
```bash
sudo systemctl daemon-reload
sudo systemctl start jupyterhub
sudo systemctl enable jupyterhub
sudo systemctl status jupyterhub  # Check for errors
```

---

## 📦 5. Install Python Libraries

### 🧠 System-Wide (Recommended for Shared Use)
```bash
sudo -H pip3 install numpy pandas matplotlib scikit-learn
```

### 🧬 For AI Hat Acceleration (e.g., TensorFlow Lite)
```bash
sudo -H pip3 install tflite-runtime
```

### 🧑‍💻 Per-User (for custom packages)
```bash
# Users can install their own libraries
pip install --user seaborn
```

---

## 👥 6. User Management

### 🛡️ Add Admin Users
```bash
# Create an admin group
sudo addgroup jupyter_admin

# Add a user to the admin group (e.g., pi)
sudo usermod -aG jupyter_admin pi

# Update JupyterHub config to recognize the group
c.PAMAuthenticator.admin_groups = {'jupyter_admin'}
```

### 👤 Create New Users
```bash
sudo adduser data_scientist  # Replace with desired username
```

Users will automatically have access if their account exists on the Pi.

---

## ⚠️ 7. Common Issues & Solutions

### 🚧 Problem: Proxy Fails to Start
**Solution**:  
- Check port `8888` is free:  
  ```bash
  sudo netstat -tulpn | grep :8888
  ```
- Force proxy to use `127.0.0.1` in config:  
  ```python
  c.Spawner.ip = '127.0.0.1'  # Prevents network routing issues
  ```

### 🕒 Problem: User Server Timeout
**Solution**:  
- Ensure users have Jupyter installed:  
  ```bash
  sudo -u <username> pip3 install notebook
  ```
- Increase timeout in config:  
  ```python
  c.Spawner.http_timeout = 120  # Default is 60 seconds
  ```

### 🔒 Problem: SSL Warning in Logs
**Solution**:  
- For production, set up HTTPS:  
  ```python
  c.JupyterHub.ssl_cert = '/path/to/cert.pem'
  c.JupyterHub.ssl_key = '/path/to/privkey.pem'
  ```
- Or suppress warning (not recommended for public networks):  
  ```python
  c.JupyterHub.proxy.check_proxy = False  # Bypass proxy health check
  ```

### 🧨 Problem: `InvalidStateError` or "Exception is not set"
**Solution**:  
- Upgrade JupyterHub and dependencies:  
  ```bash
  sudo -H pip3 install --upgrade jupyterhub notebook
  ```

### 🐢 Problem: Slow AI Hat Performance
**Solution**:  
- Use quantized TensorFlow Lite models (8-bit integers) for NPU acceleration:  
  ```python
  interpreter = tflite_runtime.interpreter.Interpreter(
      model_path="model_edgetpu.tflite",
      experimental_delegates=[tflite_runtime.load_delegate('libedgetpu.so.1')]
  )
  ```

---

## 🌐 8. Access JupyterHub

Open a browser on any device in your network and navigate to:  
`http://<raspberry-pi-ip>:8888`  
Log in with system credentials (e.g., `pi` user).

---

## 🚀 Tips for AI Hat Integration

1. **Optimize Models**: Use TensorFlow Lite or ONNX models compiled for Synaptics B1000 NPU.
2. **Swap Space**: Increase swap to 1GB+ for memory-heavy tasks:
   ```bash
   sudo dphys-swapfile swapoff
   sudo dphys-swapfile setup  # Set `CONF_SWAPSIZE=1024`
   sudo dphys-swapfile swapon
   ```
3. **Monitor NPU**:  
   ```bash
   synaptics-ai-monitor  # Check real-time NPU utilization
   ```

---

## 📝 Final Notes

- **Security**: Add HTTPS with Let's Encrypt for external access.
- **Backups**: Regularly back up `/etc/jupyterhub` and user data.
- **Scaling**: For heavy workloads, use external storage or a Kubernetes cluster with AI Hat nodes.

Let me know if you need help with SSL setup, AI Hat optimizations, or advanced user management! 🧬
``` 

You can paste this directly into a Jupyter Notebook (`.ipynb`) or save it as a Markdown file (`.md`) for documentation. The AI Hat optimizations are included for hardware acceleration! 🚀
