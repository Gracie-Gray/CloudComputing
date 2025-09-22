# DESIGN.md

## Overview
This project provisions an AWS EC2 instance and deploys a minimal REST API that converts pounds (lbs) to kilograms (kg). The service is exposed over HTTP and responds with JSON according to the provided specification, including error handling for invalid inputs.

The goal was to demonstrate cloud provisioning, basic DevOps practices, and reliable service management on AWS.

---------------------------------------------------------------------------------------

## Architecture
- **Cloud Provider:** AWS
- **Instance Type:** t2.micro (free-tier eligible, cost efficient for small workloads)
- **AMI:** Ubuntu LTS (could have also used Amazon Linux 2)
- **Runtime:** Node.js with Express
- **Service Management:** systemd unit file
- **Access:** 
  - HTTP traffic exposed via Security Group (port 80)
  - SSH restricted to my IP
- **Client Workflow:**  
  Client → EC2 Public IP/DNS → Express web service (Node.js)  

---------------------------------------------------------------------------------------

## Design Decisions

### Language & Framework
- **Node.js/Express** was chosen because:
  - Lightweight and simple for building small REST services
  - Quick setup
  - Express provides built-in routing and middleware
- Alternatives like Python/Flask were allowed, but Node.js was used as it was included with the provided starter code.

### Instance & Networking
- **t2.micro** chosen to minimize cost while meeting performance needs.
- Security Group rules:
  - **22 (SSH):** allowed only from my personal IP
  - **80 (HTTP):** open for testing the service
  - No inbound access on other ports.

### Deployment & Service Management
- Connected via **MobaXterm** using the generated key pair.
- Installed Node.js and npm from system package manager.
- Implemented service in `server.js` as provided.
- Registered a **systemd service** (`p1.service`) so the server:
  - Starts automatically on boot
  - Restarts if the process crashes
- Logs were available through `journalctl` or `systemctl status`.

### Reliability
- Running as a systemd service avoids manual restarts.
- Optionally, NGINX can proxy port 80 to 8080 for production-style deployments.

---------------------------------------------------------------------------------------

## Error Handling & API Design
The service follows the required spec:
- `200 OK`: valid conversion with formula string
- `400 Bad Request`: missing or non-numeric query parameter
- `422 Unprocessable Entity`: negative or non-finite (NaN/Infinity) input

Examples:
- `/convert?lbs=150 → {"lbs":150,"kg":68.039,"formula":"kg = lbs * 0.45359237"}`
- `/convert?lbs=-5 → 422 {"error":"lbs must be a non-negative, finite number"}`

---------------------------------------------------------------------------------------
## Security Considerations
- SSH limited to a single trusted IP via Security Group.
- Service runs under the default user.
- No unnecessary ports exposed.
- System updates applied before installing runtime 

---------------------------------------------------------------------------------------

## Logging
- Logs available via:
  - Express console output (captured by systemd)
- This provides both access logs and error traces.

---------------------------------------------------------------------------------------

## Cleanup & Cost Hygiene
- Instance stopped/terminated after project submission.
- Associated Security Group, key pair, and EBS volumes cleaned up.
- Used free-tier resources to avoid unnecessary billing.

---------------------------------------------------------------------------------------
## Notes
- Used MobaXterm for SSH connectivity.
- Chose to follow the provided Node.js/Express code with minimal modifications.
- Future improvements could include:
  - HTTPS with Let’s Encrypt (via NGINX + Certbot)
  - Log rotation (e.g., logrotate) for long-term uptime
  - Deployment automation with cloud-init or scripts
