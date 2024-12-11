# HypervReplication

### Step-by-Step Guide to Replicate **Win Server 2016** from **SERVERDC19** to **SERVERDC22**

This guide explains how to replicate the virtual machine (**Win Server 2016**) from **Windows Server 2019 (SERVERDC19)** to **Windows Server 2022 (SERVERDC22)** using Hyper-V Replica.

---

### **Pre-requisites**
1. **Admin Privileges**: Ensure you have administrator rights on both servers.
2. **Network Connectivity**: Both servers must be able to communicate over the network.
3. **Firewall Rules**:
   - Open port **80** (HTTP) or **443** (HTTPS) for replication traffic.
4. **Certificate Setup** (if using HTTPS):
   - Use a valid SSL certificate or create self-signed certificates as needed.

---

### **Steps**

#### **1. Verify DNS Resolution**
Ensure the FQDN of **SERVERDC22** is resolvable from **SERVERDC19**:
- Run:
  ```cmd
  nslookup SERVERDC22.nextechiq.local
  ```
![image](https://github.com/user-attachments/assets/48a86855-7e93-4060-af0e-74e21c3c96a1)

- If DNS is not resolving, add the server to the hosts file on **SERVERDC19**:
  1. Edit the file:
     ```
     C:\Windows\System32\drivers\etc\hosts
     ```
  2. Add the entry:
     ```
     <IP_ADDRESS_OF_SERVERDC22> SERVERDC22.nextechiq.local 
     ```
     
     Replace `SERVERDC22.nextechiq.local` with your server FQDN or hotsname
     
![image](https://github.com/user-attachments/assets/56c514e7-f978-4c4a-8c3d-5efb7567fabf)

---

#### **2. Configure Firewall Rules**
Ensure the appropriate port is open for replication traffic:
- For **HTTP**: Port **80**.
- For **HTTPS**: Port **443**.

Run the following commands on both servers:
```cmd
netsh advfirewall firewall add rule name="Hyper-V Replica HTTP" dir=in action=allow protocol=TCP localport=80
netsh advfirewall firewall add rule name="Hyper-V Replica HTTPS" dir=in action=allow protocol=TCP localport=443
```

Alternatively:
- Open **Windows Defender Firewall** > **Advanced Settings**.
- Create inbound rules for ports **80** and/or **443**.

---

#### **3. Enable Hyper-V Replica on SERVERDC22 (Replica Server)**
1. Open **Hyper-V Manager** on **SERVERDC22**.
2. Right-click **SERVERDC22** and select **Hyper-V Settings**.
3. Navigate to **Replication Configuration**:
   - Check **Enable this computer as a Replica server**.
   - Select the authentication method:
     - **Kerberos (HTTP)**: For non-secured replication.
     - **Certificate-based authentication (HTTPS)**: For secured replication (ensure the certificate is configured correctly).
   - Configure **Allowed Storage Locations** (e.g., `D:\Replicas`).
   - Under **Authorization and Storage**, specify who can replicate:
     - **Option 1**: Allow replication from any authenticated server.
     - **Option 2**: Allow replication from specified servers (recommended).
       - Add **SERVERDC19** and specify the storage path.
4. Click **Apply** and **OK**.

---

#### **4. Configure the Virtual Machine for Replication on SERVERDC19**
1. Open **Hyper-V Manager** on **SERVERDC19**.
2. Right-click on the virtual machine (**Win Server 2016**) and select **Enable Replication**.

---

#### **5. Configure the Replication Wizard**
1. **Specify Replica Server**:
   - Enter the FQDN or hostname of **SERVERDC22**.
   - Click **Next**.
2. **Choose Authentication Method**:
   - Select **HTTP** or **HTTPS** based on your configuration.
3. **Select VHDs**:
   - Choose the virtual hard drives (VHDs) to replicate.
4. **Configure Replication Frequency**:
   - Options: Every **30 seconds**, **5 minutes**, or **15 minutes**.
5. **Initial Replication Method**:
   - Options:
     - **Send initial copy over the network**.
     - **Use an existing VM backup** if the VM already exists on the replica server.
6. **Schedule Initial Replication**:
   - Choose to start the replication immediately or schedule it for later.
7. Review the settings and click **Finish** to start replication.

---

#### **6. Monitor Replication**
1. In **Hyper-V Manager**, select the **Win Server 2016** VM.
2. In the lower pane, switch to the **Replication** tab.
3. Verify that the initial replication completes successfully.

---

#### **7. Test the Failover**
1. On **SERVERDC22**, open **Hyper-V Manager**.
2. Right-click on the replicated VM and select **Test Failover**.
3. Choose a recovery point and click **OK**.
4. Verify that the VM starts successfully on **SERVERDC22**.

---

#### **8. Perform Planned Failover (Optional)**
To migrate the VM permanently to **SERVERDC22**:
1. Shut down the VM on **SERVERDC19**.
2. On **SERVERDC19**, right-click the VM and select **Planned Failover**.
3. Confirm the failover to **SERVERDC22**.

---

This step-by-step process ensures the smooth replication and failover of the **Win Server 2016** virtual machine from **SERVERDC19** to **SERVERDC22**.
