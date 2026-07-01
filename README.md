# Reviewing DNS Forward Lookup Zone Records in Windows Server DNS Manager
**Subtitle:** Technical Workflow Manual

| Document Info | Details |
| :--- | :--- |
| **Prepared By** | AI SOP Generator |
| **Date** | 1/7/2026 |
| **Version** | 1.0 |
| **Department** | Operations & Infrastructure |
| **Target Audience** | Technical Engineers / Support Teams |

## 1. Executive Summary & Context

### 1.1 Introduction
This Standard Operating Procedure (SOP) outlines the step-by-step sequence required to achieve: Demonstrate general technical software configuration steps. Follow these guidelines to secure system integrity and maintain error-free results.

### 1.2 Operational Scope
This manual applies directly to system administrators and engineers tasked with maintaining this configuration.

## 2. Execution Prerequisites & Guidelines

### 2.1 Prerequisites
- Access privileges with root/administrator scope.
- Verified workspace internet connection.
- Pre-configured environment variables.

### 2.2 Operational Best Practices
- Execute in a clean or isolated testing/sandbox terminal first.
- Verify status check reports after each key section.
- Document all warning messages for the compliance audit log.

## 3. Step-by-Step Procedure

### Step 1: Expand Domain Controller Node
- **Action Category:** Click
- **Technical Purpose:** To access the subcategories and DNS structures hosted on the selected domain controller.
- **Expected Outcome:** The domain controller node expands, showing subfolders like Forward Lookup Zones, Reverse Lookup Zones, and Conditional Forwarders.
- **Targeted Inputs:** `DC01 node expand arrow`

**Description:**  
In the left-hand navigation pane of the DNS Manager console, locate and click the expand arrow next to the primary domain controller, such as DC01.

*(Screenshot 1 reference embedded)*

---

### Step 2: Expand Forward Lookup Zones Folder
- **Action Category:** Click
- **Technical Purpose:** To view the specific domain search zones currently configured on the DNS server.
- **Expected Outcome:** The available lookup zones, including '_msdcs.kelvglobal.com' and 'kelvglobal.com', are listed underneath.
- **Targeted Inputs:** `Forward Lookup Zones`

**Description:**  
Click the expansion arrow next to the 'Forward Lookup Zones' folder under the DC01 server node.

*(Screenshot 2 reference embedded)*

---

### Step 3: Select Target Domain Zone
- **Action Category:** Click
- **Technical Purpose:** To load and display the specific DNS records associated with the kelvglobal.com domain.
- **Expected Outcome:** The right details pane populates with the zone's subdirectories and core DNS records, including SOA, NS, and A-records.
- **Targeted Inputs:** `kelvglobal.com`

**Description:**  
Click directly on the target zone name, 'kelvglobal.com', within the expanded Forward Lookup Zones list.

*(Screenshot 3 reference embedded)*

---

### Step 4: Verify Host (A) and Infrastructure Records
- **Action Category:** Observe Status
- **Technical Purpose:** To confirm that the domain name services properly map active server hosts (like DHCP01, FS01, and domain controllers) to their correct IP configurations.
- **Expected Outcome:** A comprehensive list of host records is verified with accurate IP addresses and static/timestamp configurations.
- **Targeted Inputs:** `Right-hand details list scroll pane`

**Description:**  
Scroll down and inspect the details pane to review Host (A) IP addresses, directory structures (_tcp, _udp), and name servers to ensure all values are configured correctly.

*(Screenshot 4 reference embedded)*

---

## 4. Troubleshooting & Escalation

- Verify system port availability before initiating service runs.
- Confirm credentials in the environment vault on connectivity failure.
