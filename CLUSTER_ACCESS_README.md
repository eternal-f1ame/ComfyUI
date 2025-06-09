# ComfyUI SLURM Cluster Access Guide

This guide explains how to run ComfyUI on a SLURM cluster compute node and access the web interface from your local machine using SSH tunneling and tmux.

## Overview

When running ComfyUI on a SLURM cluster, the compute nodes are typically not directly accessible from the internet. This guide shows you how to:
1. Launch ComfyUI on a compute node
2. Create an SSH tunnel to access the web interface
3. Use tmux for better session management

## Prerequisites

- Access to a SLURM cluster
- ComfyUI installed on the cluster
- SSH client on your local machine
- tmux installed on the cluster

## Complete Procedure

### Step 1: SSH into the cluster and get a compute node

**From your local machine:**
```bash
ssh user@remote_host
```

**Once on the login node, request a SLURM allocation:**
```bash
srun --pty bash
```

**Note the compute node name:**
```bash
hostname
```
*Example output: `c2-1`* - Remember this name!

### Step 2: Start tmux and launch ComfyUI

**On the compute node:**
```bash
# Start tmux session
tmux new -s main

# Activate the conda environment
conda activate comfy

# Navigate to ComfyUI directory
cd /home/aeternum/Research2/SE/ComfyUI

# Start ComfyUI with network access enabled
python main.py --front-end-version Comfy-Org/ComfyUI_frontend@latest --listen 0.0.0.0 --port 8188
```

**Wait for this output:**
```
Starting server
To see the GUI go to: http://0.0.0.0:8188
```

⚠️ **Keep this terminal/tmux session running!**

### Step 3: Verify ComfyUI is running (optional)

**Create a new tmux window for verification:**
```bash
# Press Ctrl+B, then C to create a new window
# OR Press Ctrl+B, then " to split horizontally
```

**In the new window/pane, verify ComfyUI is running:**
```bash
netstat -tlnp | grep 8188
curl -I http://localhost:8188
```

**Expected output:**
```bash
# netstat should show:
tcp        0      0 0.0.0.0:8188            0.0.0.0:*               LISTEN      xxxxx/python

# curl should return HTTP headers (not connection refused)
```

### Step 4: Create SSH tunnel from your local machine

**Open a NEW terminal on your LOCAL machine and run:**
```bash
ssh -L 8189:COMPUTE-NODE-NAME:8188 user@remote_host
```

Replace `COMPUTE-NODE-NAME` with the actual hostname from Step 1 (e.g., `c2-1`):
```bash
ssh -L 8189:c2-1:8188 user@remote_host
```

⚠️ **Keep this terminal open!** This maintains the tunnel.

### Step 5: Access ComfyUI

**Open your web browser and go to:**
```
http://localhost:8189
```

You should now see the ComfyUI interface!

## What's happening behind the scenes

1. **ComfyUI** runs on `COMPUTE-NODE:8188` (e.g., `c2-1:8188`)
2. **SSH tunnel** forwards `localhost:8189` → `COMPUTE-NODE:8188`
3. **Your browser** accesses ComfyUI via `localhost:8189`

## Important Notes

- **Keep both sessions running:** The tmux session with ComfyUI and the SSH tunnel
- **Use the correct compute node name:** Replace with whatever `hostname` returns
- **Port numbers:** ComfyUI runs on 8188, tunnel uses 8189 locally to avoid conflicts
- **Run SSH tunnel from local machine:** Not from the cluster

### "Connection refused" errors

1. **Check ComfyUI is running:**
   ```bash
   netstat -tlnp | grep 8188
   ```

2. **Verify local connectivity on compute node:**
   ```bash
   curl -I http://localhost:8188
   ```

3. **Check SSH tunnel is using correct node name:**
   ```bash
   ssh -L 8189:CORRECT-NODE-NAME:8188 user@remote_host
   ```

### "Address already in use" errors

If port 8189 is busy on your local machine, use a different port:
```bash
ssh -L 8190:c2-1:8188 user@remote_host
```
Then access: `http://localhost:8190`

### Lost SSH connection

If your SSH connection drops but you used tmux:
1. SSH back to the cluster
2. Connect to the compute node: `ssh COMPUTE-NODE-NAME`
3. Reattach to tmux: `tmux attach-session -t comfyui`
4. Recreate the SSH tunnel from your local machine

---