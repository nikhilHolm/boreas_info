# Boreas Openvas host detection mechanism

## Boreas Technology

Boreas is a command-line tool used to scan for alive hosts. This command-line client uses the libgvm_boreas library, which is part of gvm-libs. The openvas-scanner 21.04 also uses this library, and it is enabled by default in OAS 21.04. Otherwise, it uses the classic method via a nasl script.

Boreas supports IPv4 and IPv6 address ranges and allows the exclusion of certain addresses from a range. The alive ping tests support ICMP, TCP-ACK, TCP-SYN, and ARP, and any combination. For TCP ping, an individual port list can be applied.

Boreas replaces the old alive test method, a nasl script [host_alive_detection.nasl], which uses nmap. We will not see a major difference when we scan a single host, but a significant difference for large networks. 

The main difference is that with the old classic method via host_alive_detection.nasl, the host scan must be started, the script is launched, and once it detects that the host is alive, the scan continues. Otherwise, the host scan finishes. With boreas, the scanner first detects all alive hosts (which is faster than parsing and running a script) and only starts a host scan for those hosts that were found to be alive. Additionally, boreas runs in parallel with the scanner. While the scanner already scans the first alive hosts found by boreas, boreas continues testing for other alive hosts in the target list.

## Advantages & Disadvantages

- Scanning is becoming faster, which is especially helpful in large networks. GOS 20.08 introduced the Boreas Alive Scanner, a host alive scanner that identifies active hosts in a target network. With GOS 21.04, the Boreas Alive Scanner becomes the standard, eliminating the need for manual activation.

- The Boreas Alive Scanner is not limited in terms of the maximum number of simultaneous alive scans it can perform, making it faster than its predecessor Nmap. This significantly reduces scanning time for large networks. Initial scan results are available faster, regardless of the percentage of reachable hosts in the network.

- I've read a lot of cases where users are either getting errors while scanning, scans are getting stuck at 0%, wrong host count, and many more because of Boreas. Disabling it will solve the issue. References: https://github.com/greenbone/boreas/issues/40, https://forum.greenbone.net/t/boreas-scans-result-unreliable/13587, https://forum.greenbone.net/t/gvm-21-4-4-on-kali-2020-1-problem/12092/4, https://forum.greenbone.net/t/test-immediatly-interrupted-no-valid-alive-test-specified/11750/4

## Real World Scenerio

- Openvas 9 Scan - Able to detect 13 hosts [12 hosts are acutally up], Scan Duration: 38 Minutes

- Openvas 21.4.4 Scan With test_alive_hosts_only Option Yes - Able to detect 12 hosts, Scan Duration: 26 Minutes
  
  Logs:

  ![Alt text](Screenshot%20from%202023-03-03%2012-33-00.png)

- Openvas 21.4.4 Scan with test_alive_hosts_only Option No - Able to detect 13 hosts, Scan Duration: 33 Minutes

Scan Reports for Reference: [click here](https://drive.google.com/drive/folders/1zcPNWvnUUuwQ0W6vVqqlJEwyTlCohFph?usp=share_link)


## Installation and Usage

As far as I understand, <b>`test_alive_hosts_only`</b> option activates a new alive scan system called Boreas. It's by default enable in GOS 21.04. 

- Running <b>`openvas -s`</b> on a server with GOS 21.04 running will result in the current configuration being shown.

  ![Alt text](Screenshot%20from%202023-03-02%2018-00-05.png)

- Save `test_alive_hosts_only=no` in the openvas.conf file (located by default at /opt/gvm/etc/openvas/openvas.conf) to disable Boreas alive host detection. This will overwrite any existing default values.


So, the conclusion is `test_alive_hosts_only` is a setting for the openvas scanner application to use our new alive host detection. It is a global setting and can only be changed via the config file. This new alive host detection is run before all Nasl scripts. Only as alive detected hosts will be tested for vulnerabilities.


Note: Most of the Ping Host parameters are no longer supported in GOS 21.04, because they are incompatible with the new Boreas alive scanner. 

References:
https://forum.greenbone.net/t/use-boreas-as-alive-scanner/9035, https://www.greenbone.net/en/blog/gos-21-04-even-faster-more-reliable-and-clearer/, https://docs.greenbone.net/GSM-Manual/gos-21.04/en/scanning.html#:~:text=VT%20Ping%20Host-,Note,-Most%20of%20the

