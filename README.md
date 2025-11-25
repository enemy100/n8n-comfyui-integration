# n8n ComfyUI Integration Guide

Complete guide to integrate n8n with ComfyUI running on vast.ai for automated video generation workflows.

## 🎯 Overview

This guide covers the complete setup to connect your n8n instance with ComfyUI running on vast.ai, enabling automated video generation workflows.

> **Related Projects**:
> - [n8n ComfyUI Video Generation Workflow](https://github.com/enemy100/n8n-comfyui-workflow) - The automation workflow
> - [ComfyUI Flux & WAN 2.2 Setup on vast.ai](https://github.com/enemy100/comfyui-flux-wan-vast-ai) - Infrastructure setup

## 📋 Prerequisites

1. **ComfyUI instance** running on vast.ai (see [ComfyUI Setup Guide](https://github.com/enemy100/comfyui-flux-wan-vast-ai))
2. **n8n instance** (self-hosted or cloud)
   - For self-hosted setup on Raspberry Pi, see [n8n Setup on Raspberry Pi with Cloudflare Tunnel](https://github.com/enemy100/n8n-setup-on-raspberry-with-cloudflare-tunnel)
3. **Network access** between n8n and ComfyUI

## 🔧 Step 1: Expose ComfyUI

ComfyUI needs to be accessible from your n8n instance. Choose one of the following methods based on your setup:

### Option A: vast.ai Built-in Tunnel (Recommended for vast.ai Instances)

**Use this when**: ComfyUI is running on a vast.ai instance

**vast.ai provides automatic tunneling:**

1. **When you create a vast.ai instance**, they automatically provide:
   - A public access URL with token
   - Automatic port forwarding to your container

2. **Access your ComfyUI:**
   - Use the URL provided by vast.ai in the instance dashboard
   - The URL automatically routes to port 8188 (or the port you configure)
   - No additional tunnel setup needed!

3. **Find your URL:**
   - Go to vast.ai dashboard → Your instance → "Connect" or "Access"
   - Copy the provided URL (includes token automatically)

**Example:** `https://xxxxx.vast.ai:8188` or `https://xxxxx.vast.ai?token=xxxxx`

### Option B: Cloudflare Tunnel (Alternative for Other Cloud Services)

**Use this when**: ComfyUI is on a cloud service that doesn't provide built-in tunneling

1. **SSH into your instance**
   ```bash
   ssh root@<instance-ip>
   ```

2. **Install cloudflared**
   ```bash
   wget https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64
   chmod +x cloudflared-linux-amd64
   mv cloudflared-linux-amd64 /usr/local/bin/cloudflared
   ```

3. **Start tunnel**
   ```bash
   cloudflared tunnel --url http://localhost:8188
   ```

4. **Save the URL** (e.g., `https://xxxxx.trycloudflare.com`)
   - This is your ComfyUI URL for n8n

### Option C: Local Network Access (For Local Installations)

**Use this when**: Both n8n and ComfyUI are on the same local network

1. **Configure firewall** on the machine running ComfyUI:
   ```bash
   # Allow port 8188
   ufw allow 8188/tcp
   ufw enable
   ```

2. **Configure port forwarding** on your router (if accessing from outside):
   - Forward external port to ComfyUI machine's IP:8188
   - Or use your router's dynamic DNS service

3. **Use local IP address**:
   - Local access: `http://192.168.1.100:8188` (replace with ComfyUI machine's IP)
   - External access: `http://your-domain.com:8188` or `http://your-dynamic-dns:8188`

### Option D: SSH Tunnel

**Use this when**: n8n is on your local machine and ComfyUI is remote

```bash
# On your local machine
ssh -L 8188:localhost:8188 root@<vast-ai-instance-ip>
```

Then use `http://localhost:8188` as your ComfyUI URL.

## 🔑 Step 2: Get ComfyUI Access URL and Token

> **Note**: For vast.ai instances, the access URL and token are automatically generated when you create the instance. For local installations, you can skip authentication.

### For Cloud/Remote Access (vast.ai)

**vast.ai automatically provides access:**

1. **When you create an instance on vast.ai**, they generate:
   - A public access URL (e.g., `https://xxxxx.vast.ai`)
   - An access token (included in the URL or provided separately)

2. **Access your instance services:**
   - The vast.ai URL automatically tunnels to your container's services
   - ComfyUI running on port 8188 will be accessible via the vast.ai URL
   - The token is embedded in the URL or provided in the instance details

3. **Find your access information:**
   - Check the vast.ai dashboard for your instance
   - Look for "Connect" or "Access" section
   - Copy the URL and token provided

4. **Start ComfyUI** (no special authentication needed - vast.ai handles it):
   ```bash
   cd ComfyUI
   python3 main.py --listen 0.0.0.0 --port 8188
   ```
   
   The vast.ai tunnel will automatically route traffic to port 8188.

**Example vast.ai URL format:**
```
https://xxxxx.vast.ai:port
# or
https://xxxxx.vast.ai?token=xxxxx
```

### For Local Installation

If both n8n and ComfyUI are on the same local network:
- **API token is optional** - you can skip authentication
- Configure firewall to allow port 8188
- Set up port forwarding on your router if accessing from outside
- Use local IP address (e.g., `http://192.168.1.100:8188`) instead of tunnel URL

## 🔌 Step 3: Configure n8n

### 3.1. Create HTTP Bearer Auth Credential (If Required)

> **Note**: 
> - **For vast.ai**: The token is usually included in the URL automatically. You may not need separate Bearer Auth.
> - **For Cloudflare Tunnel**: Usually no authentication needed.
> - **For local installations**: Skip this step entirely.

**If your ComfyUI requires authentication:**

1. In n8n, go to **Credentials** → **Add Credential**
2. Select **HTTP Bearer Auth**
3. Enter your access token:
   - **vast.ai**: Extract token from the URL provided by vast.ai (if separate from URL)
   - **Other services**: Use the token provided by your service
4. Save as "ComfyUI Bearer Auth"

**For vast.ai**: Check if the token is in the URL format. If the URL works without separate auth, you can skip Bearer Auth.

### 3.2. Configure Workflow

1. **Import the workflow** (see [n8n ComfyUI Workflow](https://github.com/enemy100/n8n-comfyui-workflow))

2. **Open "Set ComfyUI URL" node**
   - Set `comfyBaseUrl`: Your ComfyUI URL
     - **vast.ai**: Use the URL provided in the instance dashboard (e.g., `https://xxxxx.vast.ai:8188`)
     - **Cloudflare Tunnel**: Use the tunnel URL (e.g., `https://xxxxx.trycloudflare.com`)
     - **Local**: Use local IP (e.g., `http://192.168.1.100:8188`)
   - Set `comfyApiToken`: 
     - **vast.ai**: Usually included in URL, or extract from vast.ai dashboard
     - **Other services**: Use provided token
     - **Local**: Can be empty or a simple token if you set one

3. **Update HTTP Request nodes**
   - **For cloud/remote access**: All nodes that connect to ComfyUI should use the "ComfyUI Bearer Auth" credential
   - **For local access**: You can leave authentication as "None" or remove Bearer Auth
   - Verify URLs use `={{ $('Set ComfyUI URL').first().json.comfyBaseUrl }}`

## ✅ Step 4: Test Connection

### Test 1: Basic Connectivity

1. Create a test workflow in n8n:
   - Add an **HTTP Request** node
   - Method: `GET`
   - URL: `{{ YOUR_COMFYUI_URL }}/system_stats`
   - Authentication: 
     - **Cloud/Remote**: Use your Bearer Auth credential
     - **Local**: Set to "None" (no authentication)
   - Execute the node

2. **Expected result**: JSON response with system statistics

### Test 2: API Endpoint

1. In the same test workflow:
   - Change URL to: `{{ YOUR_COMFYUI_URL }}/prompt`
   - Method: `POST`
   - Body: Simple test prompt JSON

2. **Expected result**: `prompt_id` in response

### Test 3: Full Workflow

1. Activate the main workflow
2. Send a test message via chat
3. Monitor execution logs
4. Check if images are generated
5. Check if video generation starts

## 🔒 Step 5: Security Considerations

### Firewall Configuration

On your vast.ai instance:

```bash
# Allow only necessary ports
ufw allow 8188/tcp
ufw enable
```

### API Token Security

- **Never commit tokens to git**
- Use n8n's credential management
- Rotate tokens periodically
- Use different tokens for different environments

### Network Security

- Prefer Cloudflare Tunnel over direct IP exposure
- Use HTTPS when possible
- Monitor access logs

## 🐛 Troubleshooting

### Connection Refused

**Problem**: n8n can't connect to ComfyUI

**Solutions**:
- Verify ComfyUI is running: `ps aux | grep main.py`
- Check if ComfyUI is listening on `0.0.0.0`: `netstat -tulpn | grep 8188`
- Verify firewall rules
- Check tunnel status (if using Cloudflare)

### Authentication Failed

**Problem**: 401 Unauthorized errors

**Solutions**:
- **For vast.ai**: 
  - Verify you're using the correct URL with token from vast.ai dashboard
  - Check if token needs to be extracted from URL or used separately
  - Try accessing the URL directly in browser first to verify it works
- **For other cloud services**: Verify API token is correct
- **For local installations**: Remove Bearer Auth from n8n credentials - authentication is usually not needed
- Verify Bearer Auth credential in n8n matches the token (if required)
- Check ComfyUI logs for authentication errors
- If using local network, ensure you're not requiring authentication unnecessarily

### Timeout Errors

**Problem**: Requests timeout

**Solutions**:
- Increase timeout in HTTP Request nodes
- Check network latency
- Verify instance is not overloaded
- Check ComfyUI is processing requests

### Video Not Found

**Problem**: Workflow can't find generated video

**Solutions**:
- Verify `output/video/` folder exists
- Check ComfyUI execution logs
- Verify video was actually generated
- Check file permissions

## 📊 Monitoring

### Monitor ComfyUI

```bash
# Check if ComfyUI is running
ps aux | grep main.py

# Check GPU usage
watch -n 1 nvidia-smi

# Check ComfyUI logs
tail -f /path/to/comfyui/logs
```

### Monitor n8n

- Check n8n execution logs
- Monitor workflow execution times
- Check for error messages
- Review execution history

## 🔄 Workflow Execution

### Manual Execution

1. Activate workflow in n8n
2. Send message via chat interface
3. Monitor execution in n8n dashboard
4. Check ComfyUI for generated content

### Automated Execution

- Set up webhook triggers
- Schedule workflows
- Integrate with other services

## 📝 Best Practices

1. **Use Cloudflare Tunnel** for secure access
2. **Monitor costs** on [vast.ai dashboard](https://cloud.vast.ai/?ref_id=350820)
3. **Stop instances** when not in use
4. **Backup configurations** regularly
5. **Test workflows** before production use
6. **Monitor logs** for errors
7. **Set up alerts** for failures

💡 **Support this project**: Use our [referral link](https://cloud.vast.ai/?ref_id=350820) when using vast.ai!

## 🔗 Useful Links

- [n8n Documentation](https://docs.n8n.io/)
- [ComfyUI API Documentation](https://github.com/comfyanonymous/ComfyUI/wiki/API)
- [Cloudflare Tunnel Docs](https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/)
- [vast.ai Dashboard (Referral Link)](https://cloud.vast.ai/?ref_id=350820)

## 📄 License

This guide is provided as-is for educational purposes.

---

**Need Help?** Open an issue in the [n8n ComfyUI Workflow](https://github.com/enemy100/n8n-comfyui-workflow) repository.

