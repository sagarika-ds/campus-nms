Smart Campus Network Monitoring System

A small open-source Network Management System (NMS) lab built with Docker. It monitors a simulated network device over SNMP, shows live performance graphs in Grafana (via Prometheus), and manages faults and alarms in OpenNMS — including the full alarm lifecycle (New → Acknowledged → Escalated → Cleared).

Built as a placement-prep project for NMS/network-monitoring roles.

Architecture

                 Performance monitoring (graphs)
snmp-agent  --SNMP-->  snmp-exporter  --metrics-->  Prometheus  --queries-->  Grafana

     |
     | SNMP polling (fault checks)
     v
OpenNMS + Postgres   -->   Alarms: Node down, SNMP failed, Cleared
                 Fault / alarm management

                 
Tool                   Role
snmp-agent	           Simulated network device (net-snmp running in a container)
snmp_exporter	         Converts SNMP data into a format Prometheus understands
Prometheus	           Polls the device every 15s and stores metrics
Grafana	               Live dashboard: traffic, reachability, interface status
OpenNMS + PostgreSQL	 Node discovery, fault detection, alarm lifecycle
Docker                 Compose	Runs and networks all of the above


Features
SNMP polling of interface-level metrics (ifHCInOctets, ifHCOutOctets, ifOperStatus, etc.)
Live Grafana dashboard with 4 panels: outbound traffic, inbound traffic, device reachability, interface status
Node auto-discovery and SNMP monitoring in OpenNMS
Fault simulation: stopping the device triggers real alarms
Full alarm lifecycle demo: New → Acknowledged → Escalated → Cleared
How to Run It

1. Start the core monitoring stack

bash
git clone https://github.com/YOUR-USERNAME/campus-nms.git
cd campus-nms
docker compose up -d
Prometheus: http://localhost:9090
Grafana: http://localhost:3000 (default admin / admin, you'll be asked to set a new password)

2. Start OpenNMS (fault/alarm management)

bash
docker compose -f docker-compose.opennms.yml up -d
OpenNMS: http://localhost:8980/opennms (default admin / admin)
First start takes 3–5 minutes to initialize.

3. Add the device to OpenNMS

Get the agent's IP:

bash
docker network inspect campus-nms_nms-net -f '{{range .Containers}}{{.Name}} {{.IPv4Address}}{{"\n"}}{{end}}'

In OpenNMS: ⚙ → Quick-Add Node, enter the IP, label campus-switch-01, SNMP community public, then Provision.

4. Simulate a failure

bash
docker stop snmp-agent     # break it
docker start snmp-agent    # fix it

Watch the alarm appear in OpenNMS (Monitoring → Alarms) and the Grafana "Device Reachable" panel drop to 0, then recover.

Grafana Panels & Queries
Panel	Query
eth0 Outbound Traffic	rate(ifHCOutOctets{ifDescr="eth0"}[5m])
eth0 Inbound Traffic	rate(ifHCInOctets{ifDescr="eth0"}[5m])
Device Reachable	up{job="snmp"}
eth0 Interface Status	ifOperStatus{ifDescr="eth0"}
Screenshots


What This Demonstrates
SNMP-based monitoring fundamentals (OIDs, MIBs, polling)
Performance monitoring vs. fault management as two distinct NMS concerns
Alarm lifecycle management (raise, acknowledge, escalate, clear)
Running and troubleshooting a multi-container Docker stack

Tech Stack

Docker · net-snmp · snmp_exporter · Prometheus · Grafana · OpenNMS Horizon · PostgreSQL
