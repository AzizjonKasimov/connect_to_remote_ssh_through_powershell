# Connect to Remote Server via SSH using PowerShell

This guide provides step-by-step instructions on how to connect to any remote server via SSH using Windows PowerShell. While some examples reference AWS Lightsail instances, these instructions apply to connecting to any SSH-enabled server from a Windows environment.

## Prerequisites

- Windows 10/11
- Access to any remote server with SSH enabled
- Authentication credentials (either a private key file or username/password)

## Step 1: Install OpenSSH Client

Recent Windows 10/11 builds include the OpenSSH client by default. To check if it's installed:

```powershell
Get-WindowsCapability -Online | Where-Object Name -like 'OpenSSH.Client*'
```

If it shows **Installed**, you're all set. If not, install it:

```powershell
# Run as Administrator
Add-WindowsCapability -Online -Name OpenSSH.Client~~~~0.0.1.0
```

## Step 2: Secure Your Private Key File

PowerShell (and OpenSSH) will warn you if your key has overly permissive settings. To lock it down:

```powershell
# Assuming your key is at C:\Users\You\lightsail-key.pem
icacls "C:\Users\You\lightsail-key.pem" /inheritance:r
icacls "C:\Users\You\lightsail-key.pem" /grant:r "$($env:USERNAME):R"
```

This removes inherited permissions and then grants **read-only** access to your user.

## Step 3: SSH into the Server

### Using Key-Based Authentication

Use the `ssh` command specifying your key, username, and the server's IP address or hostname:

```powershell
ssh -i "C:\Path\To\Your\private-key.pem" username@server-ip-or-hostname
```

* **-i** points to your private key file
* Replace `username` with the appropriate username for your server:
  * Ubuntu servers often use: `ubuntu`
  * Debian may use: `admin` or `debian`
  * CentOS/RHEL/Amazon Linux may use: `ec2-user`, `centos`, or `root`
  * Windows servers with OpenSSH: typically your Windows username
* Replace `server-ip-or-hostname` with your server's IP address or domain name

### Using Password Authentication

If your server allows password authentication:

```powershell
ssh username@server-ip-or-hostname
```

You'll be prompted to enter your password.

The first time you connect, you'll be prompted to accept the host fingerprint—type **yes** and press **Enter**.

## Step 4: (Optional) Create a Named SSH Profile

If you connect to servers frequently, create profiles in `~/.ssh/config` to simplify connections:

1. Open (or create) `C:\Users\YourUsername\.ssh\config` in a text editor
2. Add configurations for each server:

```
# Server with key authentication
Host myserver1
    HostName 192.168.1.100
    User admin
    IdentityFile C:\Path\To\Your\private-key.pem
    
# Server with password authentication
Host myserver2
    HostName example.com
    User username
    # Password will be prompted when connecting
    
# Server with non-standard port
Host myserver3
    HostName 10.0.0.5
    User root
    Port 2222
```

3. Then connect simply with:

```powershell
ssh myserver1
```

## Troubleshooting

* **Permission denied (publickey)**: Make sure your private key matches the public key installed on the server and check that file permissions are locked down properly.
* **Connection timed out**: Verify that the server is online and that firewalls (both on the server and any network equipment) allow inbound connections on TCP port 22.
* **Host key verification failed**: If you've previously connected to a different server at the same IP address, you may need to remove the old entry from your `known_hosts` file with `ssh-keygen -R server-ip-or-hostname`.
* **Bad permissions**: If your key has incorrect permissions, fix them using the commands in Step 2.
* **Connection refused**: Make sure the SSH service is running on the remote server and listening on the expected port.

## Additional Resources

- [OpenSSH Documentation](https://www.openssh.com/manual.html)
- [Microsoft's OpenSSH for Windows Documentation](https://docs.microsoft.com/en-us/windows-server/administration/openssh/openssh_overview)
- [SSH Config File Examples](https://linuxize.com/post/using-the-ssh-config-file/)

## Advanced SSH Options

### Port Forwarding

Forward a local port to a port on the remote server:

```powershell
# Local port 8080 -> remote port 80
ssh -L 8080:localhost:80 username@server-ip-or-hostname
```

### Jump Host (ProxyJump)

Connect through an intermediate server:

```powershell
# Connect to target-server via jump-host
ssh -J username@jump-host username@target-server
```

### SSH Agent

Use SSH agent to manage your keys:

```powershell
# Start the SSH agent
Start-Service ssh-agent

# Add your key to the agent
ssh-add C:\Path\To\Your\private-key.pem
```
