<p align="center">
<img src="https://i.imgur.com/Ua7udoS.png" height="80%" width="80%" alt="Traffic Examination"/>
</p>

<h1>Configuring Network Protocols, Security Groups, and Inspecting Traffic Between Azure Virtual Machines</h1>

In this demonstration we go over the process for observing network traffic between Azure Virtual Machines within Wireshark. We also play around with configuring Network Security Groups.

_<b>NOTE:</b> This demonstration uses materials created in the previous demonstration, ["Configuring On-premises Active Directory within Azure VMs"](https://github.com/terikaj/configure-ad)._

<h2>Environments and Technologies Used</h2>

- Microsoft Azure (Virtual Machine)
- Microsoft Remote Desktop
- Command-Line (Tools / Commands)
- Network Protocols (DHCP, SSH, RDP, DNS, HTTP/S, ICMP)
- Wireshark (Network Protocol Analyzer)

<h2>Operating Systems Used </h2>

- Windows 10 (21H2)
- Ubuntu Server 20.04 (Linux)
- MacOS

<h2>High-Level Steps</h2>

- Setup 2 Virtual Machines within Azure:
  - Client-01 VM #1 (Windows 10)
  - VM #2 (Linux Ubuntu) -- uses the same Resource Group and Vnet as Client-01.
  - _For this example Client-01 is located in DC-01 which is a remaining resource group/vnet we      had from the previous lab._
  - **Feel free to create a new VM and resource group if you desire.**
- Use Microsoft Remote Desktop (RDP) to access Client-01; Install Wireshark.
- Use Wireshark and PowerShell to Observe Network Protocols (DHCP, SSH, RDP, DNS, HTTP/S, ICMP)
  - Add new Inbound Rules to our linux machine such as Deny/Allow ICMP protocol.
- Bonus: How to Display and Flush DNS.

<h2>Actions and Observations</h2>

<h3>Creating 2 Virtual Machines</h3>

- In the Search Box at the top header, type and select "Virtual Machines".
  - If "Virtual Machines" is already listed on the front page, click there to save yourself time     rather than manually searching.
- Click "Create", then select "Azure Virtual Machine".
<p align="center">
<img width="418" alt="Azure 11" src="https://github.com/TerikaJ/azure-network-protocols/assets/136477450/fc90faf5-2ff1-4fd3-94dc-8b535a9f3fa6">
<img width="416" alt="Azure 12" src="https://github.com/TerikaJ/azure-network-protocols/assets/136477450/8bd444fc-758c-4269-9d14-3e32bbebbc0c">
</p>

- Name your Virtual Machine anyway you want (this example uses **Client-01**).
  - The Resource Group is automatically given a name when naming the VM, but you can change it       if you wish (this example uses **DC-01_group**).
- Change the Region that best suites your location (this example uses **(US) West US 3**).
- Change the Image to a Windows OS (this example uses **Windows 10 Pro, version 22H2 - x64         Gen2**).
- Make sure the Size is adequate enough to run this server (this example uses **Standard_E2s_v3 - 2 vcpus, 16 GiB memory**).
- Create a username and password of your choice (this example uses **clientuser**).
- Skip through the following sections; Click "Review + create".
  - IF you notice a Licensing Checkbox at the end of the page, ensure it's CHECKED!
- If Validation passes, click "Create".
<p align="center">
<img width="495" alt="AD 4 Step 2 Begins" src="https://github.com/TerikaJ/azure-network-protocols/assets/136477450/4a36bb8f-1bec-4b41-92d2-8cd3f359c47e">
</p>

_We repeat the previous steps listed in the creation of Client-01 VM, however, our Operating System will be Ubuntu (Linux):_
- Set the Resource Group to the same group as Client-01 VM (this example uses **DC-01_group**).
- Name your Linux Virtual Machine anything you desire (this example uses **VM2**).
- Change the **Image** to Ubuntu Linux (this example uses **Ubuntu Server 20.04 LTS - x64 Gen2**)
- You may keep the size the same as the Windows VM (However, this example uses **Standard_E2s_v3
  2 vcpus, 8 GiB memory**). As long as you are utilizing at minimum 2 vcpus, and 8gb of RAM you    will be fine.
- Change the **Authentication type** to "Password", and create any username (this example uses the username **linuser**).
- When finished, press "Next:" until you reach "Networking" (or click the Networking tab").
<p align="center">
<img width="474" alt="Screen Shot 2024-04-06 at 4 34 40 PM" src="https://github.com/TerikaJ/azure-network-protocols/assets/136477450/4357ed56-428e-4667-b0c5-4d4303ac1db7">
</p>

- Ensure the Virtual Network is set to the same VNET as your Windows VM (this example uses **DC-01-vnet**).
- Set the Public IP to the automatic selection (you may need to confirm the selection).
- Click "Review + create".
- If Validation passes, click "Create".
<p align="center">
<img width="589" alt="Screen Shot 2024-04-06 at 4 38 44 PM" src="https://github.com/TerikaJ/azure-network-protocols/assets/136477450/94e6f0c9-1138-4d41-8148-7c08a5c56051">
</p>

<h3>Connecting to VM1 and Installing Wireshark</h3>

- Back in the Azure Portal, go to Client-01's Overview page and copy the Public IP address.
- On your PC or Mac click the "Microsoft Remote Desktop Connection" (RDP). (this example uses      MacOS) 
- Click Add PC and input the IP address and click "Connect" (this example uses **20.38.32.244**).
- Enter the login credentials for Client-01, click "Continue" (this example uses **clientuser**).
- You will see a Certificate prompt, just click "Continue".
- In the inital boot, you can disable all privacy settings when prompted, click "Accept".
<p align="center">
<img width="591" alt="Screen Shot 2024-04-06 at 5 34 49 PM" src="https://github.com/TerikaJ/azure-network-protocols/assets/136477450/30710db9-b1b0-497a-bd2e-9735e91b48fa">
<img width="433" alt="Screen Shot 2024-04-06 at 5 36 32 PM" src="https://github.com/TerikaJ/azure-network-protocols/assets/136477450/f60b2b60-93d8-4e16-8f5e-94f2d1683246">
<img width="653" alt="Screen Shot 2024-04-06 at 5 42 53 PM" src="https://github.com/TerikaJ/azure-network-protocols/assets/136477450/aec847d0-3a46-449c-ad1a-681ce9c53c97">
</p>

- On Client-01, open Microsoft Edge (or download Chrome), then navigate to the Wireshark           download page.
  - You can Google search the link, or copy the link HERE https://www.wireshark.org/download.html
- Click "Windows x64 Installer" to begin the download.
- Once the application is downloaded, click "Open file" to run the .exe file (you can also find the file within your Downloads folder on the Windows Explorer page or your File Explorer)
<p align="center">
<img width="653" alt="Screen Shot 2024-04-06 at 5 42 53 PM" src="https://github.com/TerikaJ/azure-network-protocols/assets/136477450/a7478003-90cd-4de3-b2d0-1d000d2426f5">
</p>

- The installation prompt will appear, hit "Next".
- Once the installation prompt appears, leave everything by default and keep pressing "Next"       "Noted" until you reach "Installing".
- If you notice an agreement prompt during installation, click agree & install.
- After the installation is complete, click "Finish".
<p align="center">
<img width="518" alt="Screen Shot 2024-04-06 at 5 47 06 PM" src="https://github.com/TerikaJ/azure-network-protocols/assets/136477450/7c830896-2069-454b-b79f-4212b04d21ab">
<img width="520" alt="Screen Shot 2024-04-06 at 5 48 16 PM" src="https://github.com/TerikaJ/azure-network-protocols/assets/136477450/e1c4de7c-16b8-433a-9038-032c0fad69c9">
<img width="520" alt="Screen Shot 2024-04-06 at 5 53 25 PM" src="https://github.com/TerikaJ/azure-network-protocols/assets/136477450/2c9401d3-ea33-4139-b9fb-b04050639c01">
<img width="519" alt="Screen Shot 2024-04-06 at 5 54 55 PM" src="https://github.com/TerikaJ/azure-network-protocols/assets/136477450/92f9e307-beb3-482b-8eec-9f7ceb4650af">
<img width="517" alt="Screen Shot 2024-04-06 at 5 56 46 PM" src="https://github.com/TerikaJ/azure-network-protocols/assets/136477450/b461809e-9b29-43da-be7d-a851bbe8e5fd">
</p>

<h3>Observe ICMP Traffic using Wireshark</h3>

- Within the virtual machine, navigate to the Wireshark application & click run. 
- Click the first button (blue shark fin) at the top tool bar to begin capturing activity on the   VM.
  - Observe the network activity taking place in the background of the VM.
<p align="center">
<img width="331" alt="Screen Shot 2024-04-06 at 5 57 55 PM" src="https://github.com/TerikaJ/azure-network-protocols/assets/136477450/79dda0ef-6d46-4088-8de4-fcad4b7c3495">
<img width="786" alt="Screen Shot 2024-04-06 at 6 05 55 PM" src="https://github.com/TerikaJ/azure-network-protocols/assets/136477450/2d617a3c-85d6-404a-ba77-562160ffdaad">
</p>

- Click the search box, type "ICMP", then press ENTER to confirm or the blue arrow in the right    corner.
  - All boxes should become blank (since there is currently no activity under the ICMP protocol)
<p align="center">
<img width="793" alt="Screen Shot 2024-04-06 at 6 08 33 PM" src="https://github.com/TerikaJ/azure-network-protocols/assets/136477450/61b4c71e-cd68-48dd-bdea-6789236c46bd">
<img src="https://i.imgur.com/a4jfqVg.jpg" height="100%" width="100%" alt="Disk Sanitization Steps"/>
</p>

_Note: The 'ping' tool within the Command Prompt (cmd) / PowerShell uses protocol ICMPv4._
- Minimize the virtual machine window; Navigate back to the Azure Portal.
- Go to VM2's Overview page and copy the PRIVATE IP address (this example uses **10.0.0.6**).
- Return to VM2, press the Windows Key/Button and search for "CMD" or "PowerShell".
- Type in `ping -t <Private IP address>` (this example uses command **ping -t 10.0.0.6**).
  - Observing Wireshark, you should see an infinite reply from 10.0.0.6.
<p align="center">
<img width="775" alt="Screen Shot 2024-04-06 at 6 13 01 PM" src="https://github.com/TerikaJ/azure-network-protocols/assets/136477450/e6599114-2602-4bfa-9337-baeb0a4c5b5a">
<img width="1521" alt="Screen Shot 2024-04-06 at 6 14 19 PM" src="https://github.com/TerikaJ/azure-network-protocols/assets/136477450/7516e8a8-837e-4c9f-973c-6ffd63637f6b">
</p>

_While the infinite ping is taking place, we'll attempt to deny the reply ping and observe what happens next:_
- Minimize the virtual machine and navigate back to the Azure Portal.
- In the Search Box at the top header, type and select "Network Security Groups".
- Click "VM2-nsg".
- Navigate to "Inbound security rules".
- Click "Add"
<p align="center">
<img width="439" alt="Screen Shot 2024-04-06 at 6 16 00 PM" src="https://github.com/TerikaJ/azure-network-protocols/assets/136477450/82a13686-c962-4ede-bd5b-5ded74c84543">
<img width="1052" alt="Screen Shot 2024-04-06 at 6 16 55 PM" src="https://github.com/TerikaJ/azure-network-protocols/assets/136477450/91e46276-c390-49d4-9d0b-4ae97e57deff">
<img width="1048" alt="Screen Shot 2024-04-06 at 6 20 43 PM" src="https://github.com/TerikaJ/azure-network-protocols/assets/136477450/f45e43e3-e37c-46fb-a3e8-81c896c6ce9c">
</p>

- Change the protocol to "ICMP".
- Change the Action to "Deny" (_we are going to stop any & all ICMP packet requests from Client-   01_).
- Change the Priorty to a lower number than the number currently set (this example uses **200**).
  - _A lower number indicates the task is performed prior to any higher priority number._
- You can change the Name if you desire, but is not necessary (this example uses **DENY_ICMP_PING_FROM_ANYWHERE**).
- Click "Add".
- Please allow a few minutes for the change to take affect; In the meantime, return to Client-01   to observe the time out requests.
<p align="center">
<img width="1523" alt="Screen Shot 2024-04-06 at 6 24 56 PM" src="https://github.com/TerikaJ/azure-network-protocols/assets/136477450/ccf4faf6-6c8a-4b02-b438-bb7a05700b52">
</p>

_Now that we've observed the denial, we can reconfigure back to allow. Remember, there's no need to delete the added rule. We can change the Action back to "Allow"._
<p align="center">
<img width="821" alt="Screen Shot 2024-04-06 at 6 27 40 PM" src="https://github.com/TerikaJ/azure-network-protocols/assets/136477450/a5104f2d-b96a-4de4-9e42-a3e0b2917320">
<img width="828" alt="Screen Shot 2024-04-06 at 6 28 55 PM" src="https://github.com/TerikaJ/azure-network-protocols/assets/136477450/0eca175c-543b-4f43-a80f-ff5eb361e47b">
<img width="1514" alt="Screen Shot 2024-04-06 at 6 35 02 PM" src="https://github.com/TerikaJ/azure-network-protocols/assets/136477450/cd3df419-8362-4942-b9cc-74965f0e815d">
</p>

- When finished, press Control+C to stop the pinging in CMD / PowerShell.

<h3>Observe SSH Traffic using Wireshark</h3>

- Within Wireshark, type "SSH" in the search bar and press ENTER (there should be no activity).
  - A direct pathway is typing "tcp.port == 22".
<p align="center">
<img width="1518" alt="Screen Shot 2024-04-06 at 6 37 52 PM" src="https://github.com/TerikaJ/azure-network-protocols/assets/136477450/aa607e9b-32f7-44e9-a752-8154aafd539d">
</p>
- Inside PowerShell, type `ssh <VM2 username@Private IP address>` (this example would use **ssh linuser@10.0.0.6**).
- When aksed if you would like to continue connecting, type "yes", then ENTER.
- It will ask you for the password for VM2.
  - When typing the password, there will be no visual indicator of you typing, however, inputs       are being recorded.
- Once your password is typed, press ENTER.
  - You should see VM2's username, and colored Green.
    - Since VM2 uses Ubuntu, commands are now in Linux format.
<p align="center">
<img width="694" alt="Screen Shot 2024-04-06 at 6 41 39 PM" src="https://github.com/TerikaJ/azure-network-protocols/assets/136477450/c6821b32-076c-4d01-a72b-d7c6a0de23a5">
<img width="666" alt="Screen Shot 2024-04-06 at 6 44 11 PM" src="https://github.com/TerikaJ/azure-network-protocols/assets/136477450/503b69df-ec82-4682-95ca-9ff7f2bf3202">
</p>

- Now that have access to VM2, within PowerShell or CMD, type "id", then ENTER.
  - This gives us the indentity group information for VM2's user.
- Observe the new traffic appearing on Wireshark.
- Type "exit" to close the linked connection and return to Client-01's CMD.
<p align="center">
<img width="673" alt="Screen Shot 2024-04-06 at 6 46 31 PM" src="https://github.com/TerikaJ/azure-network-protocols/assets/136477450/bb2142e6-06c0-4efd-abd7-89c8d53d6dc3">
<img width="1527" alt="Screen Shot 2024-04-06 at 6 47 39 PM" src="https://github.com/TerikaJ/azure-network-protocols/assets/136477450/20e48894-0749-4273-9d26-774845581645">
</p>

<h3>Observe DHCP, DNS, and RDP Traffic using Wireshark</h3>

- In Wireshark, search for "dhcp", then ENTER (there should be no activity).
- From PowerShell, type `ipconfig /renew`, then ENTER.
  - The virtual machine will briefly lose connection, but will return shortly.
- Observe the new activity in Wireshark.
<p align="center">
<img width="1525" alt="Screen Shot 2024-04-06 at 6 51 58 PM" src="https://github.com/TerikaJ/azure-network-protocols/assets/136477450/06ae1e32-5b8a-4023-bb12-4b91c8d52047">
<img width="1517" alt="Screen Shot 2024-04-06 at 6 54 24 PM" src="https://github.com/TerikaJ/azure-network-protocols/assets/136477450/4e713e0b-027e-4976-8076-a42c446aa18c">
</p>

_Next we observe DNS traffic activity:_
- In Wireshark, search for "dns", then ENTER (there should be more traffic).
  - A direct pathway is typing "udp.port == 53".
- Clear the traffic by pressing the "Restart current capture" button (green shark fin).
- In PowerShell, type `nslookup www.google.com`, observe the new activity in Wireshark.
<p align="center">
<img width="1517" alt="Screen Shot 2024-04-06 at 6 56 47 PM" src="https://github.com/TerikaJ/azure-network-protocols/assets/136477450/d072e708-3048-4ca6-bd73-626aeea452e1">
<img src="https://i.imgur.com/xvQtTBC.jpg" height="100%" width="100%" alt="Disk Sanitization Steps"/>
<img width="1520" alt="Screen Shot 2024-04-06 at 7 01 06 PM" src="https://github.com/TerikaJ/azure-network-protocols/assets/136477450/1c2cc8a1-707e-4e3e-ba50-092e73ba8958">
</p>

_Our final observation is RDP traffic activity:_
- In Wireshark, search for "rdp", then ENTER (you should see continuous traffic).
  - A direct pathway is typing "tcp.port == 3389".
_Because we are currently utilizing RDP to run the virtual machine, everything done while inside the VM is captured into Wireshark._
<p align="center">
<img width="1521" alt="Screen Shot 2024-04-06 at 7 02 11 PM" src="https://github.com/TerikaJ/azure-network-protocols/assets/136477450/1d2f9e48-add4-4cb4-be70-8c784577ffcf">
</p>

<h3>BONUS: Display and Flush DNS</h3>

- In PowerShell, type `ipconfig /displaydns`, the ENTER.
  - _You should see domain names to other websites with additional information._
  - _The saved data in this location allows the system to remember information from websites         that were already visited. This is done without having access and without new request for        information._
<p align="center">
<img width="667" alt="Screen Shot 2024-04-06 at 7 04 59 PM" src="https://github.com/TerikaJ/azure-network-protocols/assets/136477450/778cf6f1-80a4-4e9e-8373-f00264cfca66">
</p>

- Type `ipconfig /flushdns`, then ENTER.
  - _This command deletes all entries within the cache, making our system requiring our system       to make a new request from the site for information as if were visiting the site for the         first time; This will be saved in the cache._
- Type `ipconfig /displaydns` to observe everything cleared out. There should be nothing to        display.
<p>
<img width="666" alt="Screen Shot 2024-04-06 at 7 05 59 PM" src="https://github.com/TerikaJ/azure-network-protocols/assets/136477450/b20286b8-2fe1-40fe-a13d-7797bda372bb">
</p>
<hr>

<h1><p align=center> Done! Good job! </p></h1>

<p align=right> Please delete & clean up your Azure resources when finished! <br>
If you're unsure of how to do this, please click <a href="https://github.com/terikaj/azure-begin?tab=readme-ov-file">HERE</a>
</p>

