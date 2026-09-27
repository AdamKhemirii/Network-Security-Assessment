# Project 03 - Network Security Assessment

## 1. Introduction

This project was completed in a controlled virtual lab using Kali Linux as the assessment machine and an Ubuntu Server virtual machine as the target.

The purpose of the assessment was to identify the target's network exposure, identify the services that were reachable, check the services from the target itself, make controlled changes, and verify the result with another scan.

The assessment was performed on the local lab network 192.168.1.0/24. The target used in the assessment was 192.168.1.10 and the Kali machine used 192.168.1.8.

## 2. Scope

The assessment was limited to the Ubuntu virtual machine at 192.168.1.10 and the services reachable from the lab network.

The main checks were:

- Network connectivity
- Host discovery
- TCP port discovery
- Service and version identification
- Basic operating system fingerprinting
- HTTP header inspection
- SSH exposure
- Listening services on the target
- Processes associated with network services
- UFW firewall configuration
- Verification after remediation

No password cracking or exploitation was performed.

## 3. Network Identification

The Ubuntu target reported the address 192.168.1.10/24 on interface ens33. Kali reported 192.168.1.8/24 on eth0. Both systems were on the same 192.168.1.0/24 network.

The connectivity test from Kali to the target was successful. Four ICMP packets were sent and all four were received, with 0% packet loss.

Evidence: Screenshots 01, 02 and 03.

## 4. Host Discovery

Nmap host discovery was used against the lab network. Four hosts responded during the scan. The target at 192.168.1.10 was identified as one of the active hosts.

This step was used to identify active systems before focusing on the target.

Evidence: Screenshot 04.

## 5. Port and Service Enumeration

The initial Nmap scan found two open TCP ports on the target:

- 22/tcp - SSH
- 80/tcp - HTTP

A service version scan identified the SSH service as OpenSSH 10.2p1 on Ubuntu and the web service as nginx 1.28.3 on Ubuntu.

A full TCP scan of ports 1 through 65535 also found only ports 22 and 80 open at that stage.

Evidence: Screenshots 05, 06 and 08.

## 6. Operating System Detection

Nmap was also used for operating system detection. The result suggested a Linux system, but Nmap reported that the OS result could be unreliable because it could not find the required combination of open and closed ports for a stronger fingerprint.

For this reason, the result was treated as an indication rather than a confirmed OS identification.

Evidence: Screenshot 07.

## 7. HTTP Check

The HTTP service was checked with curl. The target returned HTTP 200 OK and identified the web server as nginx 1.28.3 (Ubuntu).

This confirmed that the web service was reachable from the assessment machine and was serving a response.

Evidence: Screenshot 09.

## 8. SSH Check

Port 22 was checked with Nmap and was confirmed as an OpenSSH service. A verbose SSH connection attempt was also made from Kali to confirm that the service accepted network connections.

The test did not attempt password guessing or brute force.

Evidence: Screenshot 10.

## 9. Local Service Review

The target was checked locally with ss and lsof to compare the external scan with the services actually listening on the machine.

The local results showed SSH listening on port 22 and nginx listening on port 80. Other local services were also present, including system DNS resolution, Avahi, chrony and CUPS. Some of these were bound to loopback addresses rather than being exposed directly to the network.

The lsof output also showed an established SSH session from 192.168.1.8 to 192.168.1.10 while the assessment was taking place.

Evidence: Screenshots 11 and 12.

## 10. Firewall Review

UFW was active on the target. Its default incoming policy was deny and its outgoing policy was allow. The firewall contained allow rules for TCP ports 22 and 80, including IPv6 rules.

The firewall was therefore active, but SSH and HTTP were still explicitly allowed at the time of the baseline check.

Evidence: Screenshot 13.

## 11. Remediation

The nginx service was stopped and disabled so that it would no longer run as a network service. The target was then checked again with ss to confirm that nginx was no longer listening on port 80.

SSH was also stopped and its socket was disabled during the lab. This was a deliberate lab change to remove the externally reachable SSH service as well.

Evidence: Screenshot 14 and the additional service-disable screenshots.

## 12. Post-Remediation Verification

After the service changes, Nmap was run again from Kali.

The result showed:

- 22/tcp - closed
- 80/tcp - closed

A full TCP scan also showed the same result, with no other open TCP ports identified on the target.

This confirms that the services that were previously reachable on ports 22 and 80 were no longer accepting TCP connections after the remediation.

Evidence: Screenshots 16 and 17.

## 13. Findings

### Finding 1: SSH was network reachable

Port 22 was open during the initial assessment and OpenSSH was identified. SSH provides remote administration, so exposing it increases the reachable attack surface.

In this lab, SSH was later stopped and disabled. In a real server environment, SSH should not normally be disabled if remote administration is required. Instead, access should be restricted to trusted networks or management systems and the service should be hardened appropriately.

### Finding 2: HTTP was network reachable

Port 80 was open and nginx was identified. The HTTP service returned a normal 200 OK response.

The nginx service was not required for the final state of this lab, so it was stopped and disabled.

### Finding 3: Firewall rules still allowed ports 22 and 80

UFW was active and had a deny-by-default incoming policy, but it explicitly allowed ports 22 and 80. The firewall itself was therefore not the main reason those services became unavailable. The final change was mainly caused by stopping and disabling the services.

## 14. Before and After

| Area | Before remediation | After remediation |
| --- | --- | --- |
| Target IP | 192.168.1.10 | 192.168.1.10 |
| SSH | Open on TCP 22 | Closed |
| HTTP | Open on TCP 80 | Closed |
| nginx | Running | Stopped and disabled |
| SSH service | Running | Stopped/disabled in lab |
| UFW | Active | Active |
| Incoming default | Deny | Deny |
| Full TCP scan | 22 and 80 open | 22 and 80 closed |

## 15. Conclusion

The assessment identified two externally reachable services on the Ubuntu target: SSH on port 22 and HTTP on port 80. Service enumeration confirmed OpenSSH and nginx, and local checks confirmed that both services were listening on the target.

The nginx and SSH services were then stopped and disabled in the lab. A second Nmap assessment confirmed that ports 22 and 80 were closed and that no other open TCP ports were found in the full scan.

The main lesson from the assessment was that network security should be verified from both sides. External scanning showed what was reachable, while local commands showed which processes were responsible. The final scan provided evidence that the changes had an observable effect.

## 16. Evidence List

01 - Target network configuration
02 - Kali network configuration
03 - Connectivity test
04 - Network host discovery
05 - Initial open port scan
06 - Service version enumeration
07 - OS detection
08 - Full TCP port scan
09 - HTTP header enumeration
10 - SSH exposure test
11 - Target listening services
12 - Network process mapping
13 - Firewall baseline
14 - Service remediation
15 - Firewall verification
16 - Post-remediation scan
17 - Final network assessment

## 17. Limitations

This was a small virtual lab assessment. The results only describe the services and configuration observed during the assessment window.

The OS detection result was not considered conclusive because Nmap reported unreliable test conditions. The assessment also did not include authenticated vulnerability scanning, exploitation, password attacks, web application testing, or traffic analysis.

The firewall rules for ports 22 and 80 remained present in the screenshots even after the services were disabled. Because the services were stopped, the ports were closed during the final scans. A production hardening exercise would also review and remove firewall rules that are no longer required.
