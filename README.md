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
3. **Network access** between n8n and ComfyUI

## 🔧 Step 1: Expose ComfyUI

ComfyUI needs to be accessible from your n8n instance. Choose one of the following methods:

### Option A: Cloudflare Tunnel (Recommended)

1. **SSH into your vast.ai instance**
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

### Option B: SSH Tunnel

If n8n is running on your local machine:

```bash
# On your local machine
ssh -L 8188:localhost:8188 root@<vast-ai-instance-ip>
```

Then use `http://localhost:8188` as your ComfyUI URL.

### Option C: Direct IP Access

If both are on the same network or you've configured firewall rules:

- Use `http://<vast-ai-instance-ip>:8188` as your ComfyUI URL

## 🔑 Step 2: Get ComfyUI API Token

1. **SSH into your vast.ai instance**
   ```bash
   ssh root@<instance-ip>
   ```

2. **Generate API token**
   ```bash
   cd ComfyUI
   python3 -c "import secrets; print(secrets.token_urlsafe(32))"
   ```

3. **Save this token** - you'll need it for n8n configuration

4. **Start ComfyUI with API authentication** (optional but recommended)
   ```bash
   # Edit your startup script or systemd service to include the token
   # Or use environment variables
   export COMFYUI_API_TOKEN=your-token-here
   python3 main.py --listen 0.0.0.0 --port 8188
   ```

## 🔌 Step 3: Configure n8n

### 3.1. Create HTTP Bearer Auth Credential

1. In n8n, go to **Credentials** → **Add Credential**
2. Select **HTTP Bearer Auth**
3. Enter your ComfyUI API token
4. Save as "ComfyUI Bearer Auth"

### 3.2. Configure Workflow

1. **Import the workflow** (see [n8n ComfyUI Workflow](https://github.com/enemy100/n8n-comfyui-workflow))

2. **Open "Set ComfyUI URL" node**
   - Set `comfyBaseUrl`: Your ComfyUI URL (from Step 1)
   - Set `comfyApiToken`: Your API token (from Step 2)

3. **Update HTTP Request nodes**
   - All nodes that connect to ComfyUI should use the "ComfyUI Bearer Auth" credential
   - Verify URLs use `={{ $('Set ComfyUI URL').first().json.comfyBaseUrl }}`

## ✅ Step 4: Test Connection

### Test 1: Basic Connectivity

1. Create a test workflow in n8n:
   - Add an **HTTP Request** node
   - Method: `GET`
   - URL: `{{ YOUR_COMFYUI_URL }}/system_stats`
   - Authentication: Use your Bearer Auth credential
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
- Verify API token is correct
- Check if token is set in ComfyUI startup
- Verify Bearer Auth credential in n8n
- Check ComfyUI logs for authentication errors

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
2. **Monitor costs** on vast.ai dashboard
3. **Stop instances** when not in use
4. **Backup configurations** regularly
5. **Test workflows** before production use
6. **Monitor logs** for errors
7. **Set up alerts** for failures

## 🔗 Useful Links

- [n8n Documentation](https://docs.n8n.io/)
- [ComfyUI API Documentation](https://github.com/comfyanonymous/ComfyUI/wiki/API)
- [Cloudflare Tunnel Docs](https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/)
- [vast.ai Dashboard](https://cloud.vast.ai/?ref_id=350820)

## 📄 License

This guide is provided as-is for educational purposes.

---

**Need Help?** Open an issue in the [n8n ComfyUI Workflow](https://github.com/enemy100/n8n-comfyui-workflow) repository.

