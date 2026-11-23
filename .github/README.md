# RDP-POWER: Remote Desktop Workflow Suite

## Updated workflow A: single test session

Workflow A now reads `TAILSCALE_AUTH_KEY` and `RDP_PASSWORD` from repository **Settings → Secrets and variables → Actions**. Create both secrets before starting it. Passwords and auth keys are not entered into the Run workflow form. The username remains `Bullettemporary`.

1. Open Actions and select the workflow ending in **(A)**.
2. Start with `rdp_count=1`, `quick_test=false`, and `runtime_minutes=30`. Leave `crd_code` empty for Tailscale + RDP.
3. Connect your client to the same Tailscale network. Find the IP in **Install + Up Tailscale** or **Enable RDP user + firewall** logs, and connect over RDP using your secret password.
4. Cancel the workflow when testing is complete. The job has a 360-minute total limit, including setup; the requested runtime is the keep-alive duration only. Save needed test results before ending the job.

A no longer uploads desktop screenshots to Gofile, prints the RDP password, purges Tailscale devices, or dispatches another workflow. It needs no Tailscale API key. RustDesk still installs and launches. No MuMu installation or emulator compatibility has been verified.

**Scope:** B, C, and Stop are legacy workflows and have not been updated. The original documentation below describes those legacy options; its hardcoded credentials, form-based auth keys, purge and cycle instructions do not apply to updated A. Use A for these instructions. Use hosted runners only for testing/development associated with the repository.

---


A collection of GitHub Actions workflows to set up temporary remote desktop environments with Tailscale, RustDesk, and optional Chrome Remote Desktop (CRD) access.

## Features

- **Multiple Remote Access Options**: Supports RDP, Tailscale, RustDesk, and Chrome Remote Desktop
- **Temporary Instances**: Automatically managed runtime (max 360 minutes)
- **Scalable**: Run 1-10 RDP instances simultaneously
- **Automatic Cleanup**: Built-in device purging and stop workflow
- **Cycle Support**: Chain multiple workflow runs for extended sessions

## Workflows Included

1. `rdp-tailscale-rustdesk-A.yml` - Primary workflow with cycle support
2. `rdp-tailscale-rustdesk-B.yml` - Secondary workflow for cycle handoff
3. `rdp-tailscale-stop.yml` - Cleanup workflow to remove devices

## Prerequisites

- GitHub account with repository access
- Tailscale account with:
  - Tailnet (e.g., your@email.com)
  - API key (with device admin permissions)
  - Auth key (reusable or ephemeral)
- (Optional) Chrome Remote Desktop setup for CRD access

## Usage Instructions

### 1. Basic RDP + Tailscale + RustDesk Setup

1. Go to the Actions tab in your GitHub repository
2. Select "RDP + Tailscale + RustDesk + (Optional) Chrome Remote Desktop (A)"
3. Click "Run workflow" and fill in the required inputs:
   - `ts_tailnet`: Your Tailscale tailnet
   - `ts_api_key`: Your Tailscale API key
   - `ts_authkey`: Your Tailscale Auth key
4. Adjust optional parameters as needed:
   - `quick_test`: Run a 5-minute test (default: false)
   - `runtime_minutes`: Session duration (max 360, default: 355)
   - `rdp_count`: Number of instances (1-10, default: 1)
5. Click "Run workflow" to start

### 2. Using Chrome Remote Desktop (CRD)

1. In the workflow inputs, provide a `crd_code`:
   - Can be a raw CRD code (starts with 4/)
   - Full PowerShell command for CRD
   - Text/URL containing `--code="4/..."`
2. (Optional) Set a custom `crd_pin` (6-12 digits, default: 123456)
3. Note: CRD takes precedence and forces single instance mode

### 3. Running Multiple Cycles

To extend your session beyond the maximum runtime, use the `cycles` parameter:
- `cycles=0`: Stop after current run (default)
- `cycles=N`: Run N additional times after the initial run

The workflow will automatically hand off between A and B workflows for cycle management.

### 4. Cleaning Up Devices

To manually remove devices:
1. Run the "Tailscale Stop/Cleanup" workflow
2. Provide your Tailscale credentials
3. Specify the `base_prefix` (default: "bullet")
4. Set `dry_run=false` to perform actual deletion

## Input Parameters Reference

| Parameter         | Required | Description                                                                 |
|-------------------|----------|-----------------------------------------------------------------------------|
| `ts_tailnet`      | Yes      | Tailscale tailnet (e.g., you@gmail.com)                                    |
| `ts_api_key`      | Yes      | Tailscale API key (device admin permissions)                                |
| `ts_authkey`      | Yes      | Tailscale Auth key (reusable or ephemeral)                                  |
| `quick_test`      | No       | Run 5-minute test (boolean, default: false)                                 |
| `runtime_minutes` | No       | Session duration (max 360, default: 355)                                    |
| `do_purge`        | No       | Purge bullet* devices at start (single instance only, default: true)        |
| `cycles`          | No       | Number of handoffs (0=stop after run, default: 0)                           |
| `rdp_count`       | No       | Number of RDP instances (1-10, ignored if CRD provided, default: 1)         |
| `crd_code`        | No       | CRD code/command (enables CRD, forces single instance)                      |
| `crd_pin`         | No       | CRD PIN (6-12 digits, default: 123456)                                      |

## Access Information

After workflow startup, you can access the instance via:

- **RDP**: Use the provided Tailscale IP with credentials:
  - Username: `Bullettemporary`
  - Password: `Bullet@12345`
- **Tailscale**: Connect to the device hostname (bullet or bullet[ID])
- **RustDesk**: Check workflow logs for connection details
- **Chrome Remote Desktop**: Via your Chrome Remote Desktop dashboard

## Notes

- Maximum runtime per session is 360 minutes (6 hours)
- All instances are temporary and will automatically terminate after the specified runtime
- Multiple instances share the same credentials
- CRD takes priority over RDP count (forces single instance)
- Device purging only happens for single-instance runs

## Troubleshooting

- **Tailscale Connection Issues**: Check API and Auth keys, ensure they have proper permissions
- **CRD Not Working**: Verify CRD code is valid and PIN is 6-12 digits
- **Instance Limits**: GitHub Actions may have runner limits affecting multiple instances
- **Logs**: Check workflow logs for detailed error information and connection details

## Security Considerations

- These workflows create temporary admin accounts with known credentials
- Always use in secure environments
- Purge devices when no longer needed
- Rotate Tailscale keys regularly
- Avoid using in production environments
