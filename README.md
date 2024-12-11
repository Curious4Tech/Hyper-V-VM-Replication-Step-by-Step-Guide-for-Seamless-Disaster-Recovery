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
     
     Replace `SERVERDC22.nextechiq.local` with your server FQDN or hotsname and `IP_ADDRESS_OF_SERVERDC22` with your server IP.
     
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

`SERVERDC19`

![image](https://github.com/user-attachments/assets/82205df4-c8b2-4721-927a-a202c5cc0100)

`SERVERDC22`

![image](https://github.com/user-attachments/assets/dc61c376-0280-4a1c-9656-e47403a2a04c)


Alternatively:
- Open **Windows Defender Firewall** > **Advanced Settings**.
- Create inbound rules for ports **80** and/or **443**.

---

#### **3. Enable Hyper-V Replica on SERVERDC22 (Replica Server)**
1. Open **Hyper-V Manager** on **SERVERDC22**.
2. Right-click **SERVERDC22** and select **Hyper-V Settings**.

![image](https://github.com/user-attachments/assets/50dc61f0-615f-4f8c-b11e-f8bafbb12b31)

4. Navigate to **Replication Configuration**:
   - Check **Enable this computer as a Replica server**.
   - Select the authentication method:
     - **Kerberos (HTTP)**: For non-secured replication.
     - **Certificate-based authentication (HTTPS)**: For secured replication (ensure the certificate is configured correctly).
   - Configure **Allowed Storage Locations** (e.g., `D:\Replicas`).
   - Under **Authorization and Storage**, specify who can replicate:
     - **Option 1**: Allow replication from any authenticated server.
     - **Option 2**: Allow replication from specified servers (recommended).
       - Add **SERVERDC19** and specify the storage path.
5. Click **Apply** and **OK**.

![image](https://github.com/user-attachments/assets/4423bf62-da42-48d1-94bc-4a0adca85798)


---

#### **4. Configure the Virtual Machine for Replication on SERVERDC19**
1. Open **Hyper-V Manager** on **SERVERDC19**.
2. Right-click on the virtual machine (**Win Server 2016**) and select **Enable Replication**.

![image](https://github.com/user-attachments/assets/9f67d863-925c-42cc-9aa0-f66acbec742c)

---

#### **5. Configure the Replication Wizard**
1. **Specify Replica Server**:
   - Enter the FQDN or hostname of **SERVERDC22**.
   - Click **Next**.
  
![image](https://github.com/user-attachments/assets/2cde8b92-53d4-4cc5-a40d-c973f5e59eec)

2. **Choose Authentication Method**:
   - Select **HTTP** or **HTTPS** based on your configuration and click on **Next**.

![image](https://github.com/user-attachments/assets/d8797389-1297-488b-94b5-cbf651158fc6)

3. **Select VHDs**:
   - Choose the virtual hard drives (VHDs) to replicate and then click on **Next**.

![image](https://github.com/user-attachments/assets/ea09fda8-da20-433e-ac51-597b56b5a9dd)

4. **Configure Replication Frequency**:
   - Options: Every **30 seconds**, **5 minutes**, or **15 minutes**. Then click on **Next**.

![image](https://github.com/user-attachments/assets/d66cbf3d-1221-4b4b-b31b-b82c6eac0446)

5. **Initial Replication Method**:
   - Options:
     - **Send initial copy over the network**.
     - **Use an existing VM backup** if the VM already exists on the replica server.
6. **Schedule Initial Replication**:
   - Choose to start the replication immediately or schedule it for later.

![image](https://github.com/user-attachments/assets/05f6f280-ccec-492c-a91c-213de5651596)

7. Review the settings and click **Finish** to start replication.

![image](https://github.com/user-attachments/assets/3d09a903-08b4-45ff-8b6c-a9aa515b7d88)

---

#### **6. Monitor Replication**
1. In **Hyper-V Manager**, select the **Win Server 2016** VM.
2. In the lower pane, switch to the **Replication** tab.
3. Verify that the initial replication completes successfully.

![image](https://github.com/user-attachments/assets/df665832-c7e9-43ba-b6d1-5889f3f39351)

---

#### **7. Test the Failover**
1. On **SERVERDC22**, open **Hyper-V Manager**.
2. Right-click on the replicated VM and select **Test Failover**.

![image](https://github.com/user-attachments/assets/8e0cb1de-7cdf-4dc2-b9a6-01fba8529ed0)

3. Choose a recovery point and click **Test Failover**.
   
![image](https://github.com/user-attachments/assets/c99e4ed5-0653-4b2b-92d2-6598e3dd0bff)


4. Verify that the VM starts successfully on **SERVERDC22**.

![image](https://github.com/user-attachments/assets/30b63ecb-9491-462c-81c8-6ce63debbe76)

---

#### **8. Perform Planned Failover (Optional)**
To migrate the VM permanently to **SERVERDC22**:
1. Shut down the VM on **SERVERDC19**.
2. On **SERVERDC19**, right-click the VM and select **Planned Failover**.

![image](https://github.com/user-attachments/assets/b2061695-7634-4d77-9e04-ce14a7ffd691)

3. Click on **Fail Over** to confirm the failover to **SERVERDC22**.

![image](https://github.com/user-attachments/assets/8a28c4b6-57a3-4d65-88ff-228ad0d56a5f)

4. When you go back to your **SERVERDC22**, you should see your replicated VM running.

![image](https://github.com/user-attachments/assets/1731d184-64b9-4835-992b-79a11b464f5f)

---

### **Conclusion**

By following this guide, you have successfully set up Hyper-V Replica to replicate the **Win Server 2016** virtual machine from **Windows Server 2019 (SERVERDC19)** to **Windows Server 2022 (SERVERDC22)**. This ensures high availability and disaster recovery capabilities for your virtualized environment. 

The replication process not only safeguards your data but also enables seamless failover, ensuring business continuity in case of server failures. Regularly monitor replication health and test failovers to verify the integrity of your replication setup.

With Hyper-V Replica, you can confidently manage your virtual machines, ensuring robust performance and minimal downtime in your infrastructure.
