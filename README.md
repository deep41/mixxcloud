# MIXXCLOUD

MIXXCLOUD is a cloud environment project that integrates load balancing and auto-scaling using both IaaS and LBaaS models. The project is designed to provide high availability, fault tolerance, and security for modern cloud applications.

## Overview

The main goals of MIXXCLOUD are to:
- **Distribute traffic efficiently:** Implement load balancers that evenly distribute incoming requests to multiple server instances.
- **Ensure high availability:** Use techniques such as the Centralized Agents for Redundancy in Datacenter (CARD) to provide failover capabilities.
- **Support auto-scaling:** Integrate an interface to scale instances dynamically based on demand.
- **Provide multi-tenancy:** Allow tenants to create and manage Virtual Private Clouds (VPCs), subnets, and instances.

## Project Objectives

- **Infrastructure as a Service (IaaS) and LBaaS:** Offer two load balancing models within the same environment.
- **VPC & Subnet Management:** Enable tenants to create isolated networks with custom subnets and instances.
- **High Availability and Fault Tolerance:** Leverage redundant load balancers and health-check mechanisms to maintain service continuity.
- **Security:** Implement firewall rules, DDoS protection, and IP blacklisting to secure tenant applications.
- **Auditability:** Log API calls and system events for transparency and debugging.

## Architecture

MIXXCLOUD employs a layered architecture that includes:

- **VPCs and Subnets:** Isolated layer-3 networks with dedicated Linux bridges and VXLAN overlays for inter-host communication.
- **Load Balancers:** Two models are supported:
  - **IaaS Load Balancers:** Deployed on the VPC router and distribute traffic using iptables DNAT rules.
  - **LBaaS Load Balancers:** Run at the instance level within a subnet.
- **Auto-scaling Integration:** APIs allow auto-scalers to add or remove target IPs from the load balancer's pool.
- **CARD Mechanism:** A centralized agent monitors load balancer health and ensures that endpoint IPs are always assigned to active nodes.

### Sample iptables Rules

The following example shows how a load balancer distributes incoming traffic (from IP 10.10.10.2 on port 8080) to three backend instances using weighted DNAT rules:

```bash
sudo iptables -A PREROUTING -t nat -p tcp -d 10.10.10.2 --dport 8080 \
  -m statistic --mode random --probability 0.33 -j DNAT --to-destination 192.168.1.106:80

sudo iptables -A PREROUTING -t nat -p tcp -d 10.10.10.2 --dport 8080 \
  -m statistic --mode random --probability 0.5 -j DNAT --to-destination 192.168.1.103:80

sudo iptables -A PREROUTING -t nat -p tcp -d 10.10.10.2 --dport 8080 \
  -j DNAT --to-destination 192.168.1.102:80
```

*Explanation:*  
These rules show how traffic is probabilistically routed to different backend servers. The `--probability` flag helps balance load by directing roughly 33% of the traffic to one server, 50% to another, and the remaining traffic to a third server.

## API and CLI

MIXXCLOUD includes a CLI tool (built with Python and Click) that allows tenants to manage:

- **Tenants and VPCs:** Create, list, and delete VPCs.
- **Subnets and Instances:** Define subnets and launch containerized instances.
- **Load Balancers:** Configure load balancing rules, add or remove IPs from target groups, and view load balancer status.

### Infrastructure-in-One-File (I2OF)

The CLI also supports a YAML-based topology file to automate infrastructure creation. For example:

```yaml
T1: 
  V1: 
    lb:  
      name: app 
      type: IAAS 
    subnets:  
      - name: S1 
        cidr: 192.168.45.0/24 
        servers: 
          - name: A1 
            image: zecaro/php-info 
            vcpu: 1 
            mem: 1024m 
            region: east 
            weight: 1 
          - name: A2 
            image: zecaro/php-info 
            vcpu: 1 
            mem: 1024m 
            region: east 
            weight: 1 
```

*Explanation:*  
This YAML file defines a tenant (T1) with a single VPC (V1). Inside V1, a load balancer named "app" of type IaaS is created. A subnet (S1) with the specified CIDR block hosts two server instances. Each instance includes details like image, CPU, memory, region, and a weight value used for load balancing.

## Database and Logging

- **Database:** MIXXCLOUD uses MongoDB to store the state of all cloud entities (instances, subnets, VPCs, load balancers).
- **Audit Logs:** Every API call is logged to ensure accountability and traceability. For instance, API calls for tenant operations are recorded in log files such as `JK_api_log.log`.

## How to Run

1. **Clone the repository:**

   ```bash
   git clone https://github.com/yourusername/mixxcloud.git
   cd mixxcloud
   ```

2. **Install dependencies:**

   ```bash
   pip install -r requirements.txt
   ```

3. **Launch the CLI:**

   ```bash
   python mixxcloud_cli.py
   ```

4. **(Optional) Deploy using a YAML topology file:**

   ```bash
   python infrastructure.py topology.yaml
   ```

## Contributors

- Yo-Che Lee
- Sumalatha Mashetty
- Jay Karan Telukunta
- Deepak Rajendran

## License
MIT License

---

This README provides a concise explanation of the sample code and the overall design of the MIXXCLOUD project. Adjust any sections as needed to better fit your project details.
