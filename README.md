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
- Go to VM2's Overview page and copy the PRIVATE IP address (this example uses **10.0.0.5**).
- Return to VM2, press the Windows Key/Button and seach for "CMD" or "PowerShell".
- Type in `ping -t <Private IP address>` (this example would use command **ping -t 10.0.0.5**).
  - On Wireshark, you should be able to see the results of packets being perpetually sent and received.
<p align="center">
<img width="775" alt="Screen Shot 2024-04-06 at 6 13 01 PM" src="https://github.com/TerikaJ/azure-network-protocols/assets/136477450/e6599114-2602-4bfa-9337-baeb0a4c5b5a">
<img width="1521" alt="Screen Shot 2024-04-06 at 6 14 19 PM" src="https://github.com/TerikaJ/azure-network-protocols/assets/136477450/7516e8a8-837e-4c9f-973c-6ffd63637f6b">
</p>

_While that is infinitely pinging, we'll try to deny those packets and observe what happens next:_
- Minimize the virtual machine to the Azure Portal.
- In the Search Box at the top header, type and select "Network Security Groups".
- Click on "VM2-nsg".
- Go to "Inbound security rules".
- Click "Add"
<p align="center">
<img width="439" alt="Screen Shot 2024-04-06 at 6 16 00 PM" src="https://github.com/TerikaJ/azure-network-protocols/assets/136477450/82a13686-c962-4ede-bd5b-5ded74c84543">
<img width="1052" alt="Screen Shot 2024-04-06 at 6 16 55 PM" src="https://github.com/TerikaJ/azure-network-protocols/assets/136477450/91e46276-c390-49d4-9d0b-4ae97e57deff">
<img width="1048" alt="Screen Shot 2024-04-06 at 6 20 43 PM" src="https://github.com/TerikaJ/azure-network-protocols/assets/136477450/f45e43e3-e37c-46fb-a3e8-81c896c6ce9c">
</p>

- Change the protocol to "ICMP".
- Change the Action to "Deny" (_we are trying to stop any packet requests from Client-01_).
- Change the Priorty to a lower number than the lowest one already set (this example uses **200**).
  - _A lower number means it performs the task before any higher number after it._
- You can change the Name if you desire, but not needed (this example uses **DENY_ICMP_PING_FROM_ANYWHERE**).
- Click "Add".
- Wait for a bit to take effect, but return to VM1 and observe the requests time out.
<p align="center">
<img src="https://i.imgur.com/NC8PHxH.jpg" height="60%" width="60%" alt="Disk Sanitization Steps"/>
<img src="https://i.imgur.com/Q9nha7H.jpg" height="60%" width="60%" alt="Disk Sanitization Steps"/>
<img src="https://i.imgur.com/bCN1O9d.jpg" height="100%" width="100%" alt="Disk Sanitization Steps"/>
</p>

_Now that we've observed the denial of packets, let's try allow it again, however, instead of deleting the added rule, we can simply edit the Action to "Allow"._
<p align="center">
<img src="https://i.imgur.com/Mzxx3Pk.jpg" height="100%" width="100%" alt="Disk Sanitization Steps"/>
<img src="https://i.imgur.com/oIGcjGU.jpg" height="100%" width="100%" alt="Disk Sanitization Steps"/>
<img src="https://i.imgur.com/wOJxDon.jpg" height="100%" width="100%" alt="Disk Sanitization Steps"/>
</p>

- Once done, you can press Control+C to stop the pinging in PowerShell.

<h3>Observe SSH Traffic using Wireshark</h3>

- From Wireshark, type "SSH" in the search bar and press ENTER (there should be no activity).
  - A more direct way is typing "tcp.port == 22".
<p align="center">
<img src="https://i.imgur.com/TBnNka8.jpg" height="100%" width="100%" alt="Disk Sanitization Steps"/>
</p>
- From PowerShell, type `ssh <VM2 username@Private IP address>` (this example would use **ssh linuser@10.0.0.5**).
- When it asks if you want to continue connecting, just type "yes", then ENTER.
- It will then ask you for the password for VM2.
  - When typing the password, there will be no visual indicator of you typing, but inputs are being read.
- Once you think you typed your password correctly, press ENTER.
  - You should then see the VM2's username, but colored Green.
    - Because VM2 uses Ubuntu, commands must now be in Linux format.
<p align="center">
<img src="https://i.imgur.com/QZbMzg2.jpg" height="100%" width="100%" alt="Disk Sanitization Steps"/>
<img src="https://i.imgur.com/STDQ3b0.jpg" height="100%" width="100%" alt="Disk Sanitization Steps"/>
</p>

- Now accessed to VM2, from PowerShell, type "id", then ENTER.
  - This will give you the indentity group information for VM2's user.
- Observe the new traffic on Wireshark.
- Type in "exit" to close the linked connection and return to VM1's control.
<p align="center">
<img src="https://i.imgur.com/GGXC4Ry.jpg" height="100%" width="100%" alt="Disk Sanitization Steps"/>
</p>

<h3>Observe DHCP, DNS, and RDP Traffic using Wireshark</h3>

- From Wireshark, search for "dhcp", then ENTER (there should be no activity).
- From PowerShell, type `ipconfig /renew`, then ENTER.
  - The virtual machine will briefly lose connection, but will return shortly.
- Observe the new activity in Wireshark.
<p align="center">
<img src="https://i.imgur.com/s3OoQOI.jpg" height="100%" width="100%" alt="Disk Sanitization Steps"/>
</p>

_Next to observe DNS traffic activity:_
- From Wireshark, search for "dns", then ENTER (there should be a lot of traffic).
  - A more direct way is typing "udp.port == 53".
- Clear the boxes by pressing the "Restart current capture" button (green shark fin).
- From PowerShell, type `nslookup www.google.com`, observe the new activity in Wireshark.
<p align="center">
<img src="https://i.imgur.com/sKPc1ap.jpg" height="100%" width="100%" alt="Disk Sanitization Steps"/>
<img src="https://i.imgur.com/xvQtTBC.jpg" height="100%" width="100%" alt="Disk Sanitization Steps"/>
<img src="https://i.imgur.com/bDKkcYl.jpg" height="100%" width="100%" alt="Disk Sanitization Steps"/>
</p>

_Finally to observe DNS traffic activity:_
- From Wireshark, search for "rdp", then ENTER (there should be a lot of traffic, non-stop).
  - A more direct way is typing "tcp.port == 3389".
_Because we are currently using RDP to run the virtual machine, anything and everything done while in the VM is captured into Wireshark._
<p align="center">
<img src="https://i.imgur.com/fvClqHS.jpg" height="100%" width="100%" alt="Disk Sanitization Steps"/>
</p>

<h3>BONUS: Display and Flush DNS</h3>

- From PowerShell, type `ipconfig /displaydns`, the ENTER.
  - _You should see many domain names to other websites with information below them._
  - _The saved data here allows your system to remember information a website that was already visited without and have access to it without making requesting for new info._
<p align="center">
<img src="https://i.imgur.com/dLrnksv.jpg" height="100%" width="100%" alt="Disk Sanitization Steps"/>
</p>

- Type `ipconfig /flushdns`, then ENTER.
  - _This will essentially delete all entries within the cache, making your system require to make requests from the site for information as if were visiting the first time, which is then saved in the cache._
- Type `ipconfig /displaydns` to see how everything has been cleared out and nothing to display.
<p>
<img src="https://i.imgur.com/bfOrHLb.jpg" height="100%" width="100%" alt="Disk Sanitization Steps"/>
</p>
<hr>

<h1><p align=center> Done! Good job! </p></h1>

<p align=right> Please delete & clean up your Azure resources when finished! <br>
If you're unsure of how to do this, please click <a href="https://github.com/terikaj/azure-begin?tab=readme-ov-file">HERE</a>
</p>

