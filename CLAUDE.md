# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This repository contains Ansible playbooks and automation for the Helios home lab, managed by AWX (Ansible Automation Platform).

## AWX Integration

**IMPORTANT:** Always use the Ansible Tower MCP server for all AWX/Ansible Tower API interactions.

### MCP Server Configuration

The Ansible Tower MCP server is configured in `~/.claude/settings.json`:

```json
{
  "mcpServers": {
    "ansible-tower": {
      "command": "/home/sstormes/Lab/ansible-tower-mcp/.venv/bin/python",
      "args": ["/home/sstormes/Lab/ansible-tower-mcp/src/ansible-tower-mcp/main.py"],
      "env": {
        "ANSIBLE_BASE_URL": "https://awx.home.brainstormes.org/",
        "ANSIBLE_TOKEN": "<token>"
      }
    }
  }
}
```

### AWX Instance Details

- **URL:** https://awx.home.brainstormes.org/
- **API:** https://awx.home.brainstormes.org/api/v2/

### If MCP Server is Unavailable

If the Ansible Tower MCP tools are not available in the current session, use the AWX REST API directly with curl:

```bash
# Set token (retrieve from ~/.claude/settings.json if needed)
TOKEN="<ansible_token>"

# Example API calls
curl -sk -H "Authorization: Bearer $TOKEN" "https://awx.home.brainstormes.org/api/v2/inventories/"
curl -sk -H "Authorization: Bearer $TOKEN" "https://awx.home.brainstormes.org/api/v2/job_templates/"
curl -sk -X POST -H "Authorization: Bearer $TOKEN" -H "Content-Type: application/json" \
  "https://awx.home.brainstormes.org/api/v2/job_templates/{id}/launch/" -d '{}'
```

## Inventory Structure

The "Lab Hosts" inventory (id: 2) contains:

| Group | Hosts | Description |
|-------|-------|-------------|
| `fedora` | helios, argus, hermes | Fedora Linux systems |
| `ubuntu` | nebula, vega | Ubuntu Linux systems |
| `qnap` | storage | QNAP NAS (limited shell, no sudo) |

### Host Details

- **helios** (192.168.1.20) - Fedora Workstation, control plane
- **argus** (192.168.1.26) - Fedora Server, K3s node running AWX
- **hermes** - Laptop (often offline, disabled)
- **nebula** (192.168.1.25) - Ubuntu Server, main workload host
- **vega** (192.168.1.30) - Ubuntu on Raspberry Pi 5
- **storage** (192.168.6.200) - QNAP TS-870 NAS

## AWX Resources

### Project

- **Name:** Helios Project (id: 8)
- **Repository:** https://github.com/sstormesStellarus/helios-awx
- **Branch:** main
- **Auto-sync on launch:** Yes

### Credentials

- **Lab SSH Key** (id: 3) - SSH key for lab hosts (user: sstormes)

### Job Templates

- **Collect System Information** (id: 9) - Gathers system info from all hosts by OS group

## Playbook Development

### QNAP Considerations

QNAP NAS devices have limited shell capabilities:
- Use `ansible.builtin.raw` module instead of standard modules
- Set `gather_facts: false`
- Set `become: false` (no sudo available)
- Shell type is `sh`, not bash

### Testing Playbooks

After pushing changes to the repository:

1. Sync the AWX project: `POST /api/v2/projects/8/update/`
2. Launch job template: `POST /api/v2/job_templates/{id}/launch/`
3. Check job status: `GET /api/v2/jobs/{id}/`
4. View job output: `GET /api/v2/jobs/{id}/stdout/?format=txt`

## Git Configuration

```bash
git config user.email "sonny@stormesfamily.com"
git config user.name "sstormesStellarus"
```

Commits require `--signoff` flag.
