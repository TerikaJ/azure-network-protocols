<p align="center">
<img src="https://i.imgur.com/Ua7udoS.png" height="80%" width="80%" alt="Traffic Examination"/>
</p>

<h1>Network Security Groups (NSGs) and Inspecting Traffic Between Azure Virtual Machines</h1>

This demonstration goes over how to observe various network traffic to and from Azure Virtual Machines with Wireshark as well as experiment with Network Security Groups.

_<b>NOTE:</b> This demonstration uses materials created in the previous demonstration, ["Configuring On-premises Active Directory within Azure VMs"](https://github.com/JTYKolesar/configure-ad)._

<h2>Environments and Technologies Used</h2>

- Microsoft Azure (Virtual Machines/Compute)
- Remote Desktop
- Various Command-Line Tools
- Various Network Protocols (SSH, RDH, DNS, HTTP/S, ICMP)
- Wireshark (Protocol Analyzer)

<h2>Operating Systems Used </h2>

- Windows 10 (21H2)
- Ubuntu Server 20.04

<h2>High-Level Steps</h2>

- Setup 2 Virtual Machines within Azure:
  - Virtual Machine #1 (Windows 10)
  - Virtual Machine #2 (Linux Ubuntu) -- using same Resource Group and Vnet as VM1
- Use Remote Desktop (RDP) to VM1 and install Wireshark.
- Use Wireshark and PowerShell to Observe Network Protocols (SSH, RDH, DNS, HTTP/S, ICMP)
  - Add new Inbound Rules to Deny/Allow ICMP protocol.
- Bonus: How to Display and Flush DNS.

<h2>Actions and Observations</h2>

<h3>Creating 2 Virtual Machines</h3>

- In the Search Box at the top header, type and select "Virtual machines".
  - If "Virtual machines" is already listed on the front page, then you can simply click on it, rather than manually searching.
- Click "Create", then select "Azure virtual machine".
<p align="center">
<img src="https://i.imgur.com/tiC5aA4.jpg" height="80%" width="80%" alt="Step 1-1"/>
</p>

- Name your Virtual Machine anyway you want (this example uses **VM1**).
  - Resource Group is automatically given a name when naming the Virtual Machine, but you can change it if you wish (this example uses **VM1_group**).
- Change the Region that best suites your location (this example uses **(US) West US 3**).
- Change the Image to a Windows OS (this example uses **Windows 10 Pro, version 22H2 - x64 Gen2**).
- Make sure the Size is adequate enough to run this server (this example uses **Standard_E2s_v3 - 2 vcpus, 16 GiB memory**).
- Create a username and password of your choice (this example uses **winuser**).
- Skip everything else and click "Review + create".
  - IF there is a Licensing Checkbox at the end, make sure that is CHECKED!
- If Validation passed, click "Create".
<p align="center">
<img src="https://i.imgur.com/JnsNUpw.jpg" height="100%" width="100%" alt="Step 1-2"/>
</p>

_Essentially repeat the same steps from creating the other virtual machine, but using Ubuntu (linux):_
- Set the Resource Group to the same as VM1 (this example uses **VM1_group**).
- Name your Virtual Machine anyway you want (this example uses **VM2**).
- Change Image to Ubuntu Linux (this example uses **Ubuntu Server 20.04 LTS - x64 Gen2**)
- Keep the size the same as the Windows VM (this example uses **Standard_E2s_v3 - 2 vcpus, 16 GiB memory**).
- Change the Authentication type to "Password", and create any username (this example uses the username **linuser**).
- Once done, press "Next:" until you reach "Networking" (or simply click the Networking tab".
<p align="center">
<img src="https://i.imgur.com/X6DMGwE.jpg" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>

- Make sure that the Virtual Network is set as the same as Windows VM (this example uses **VM1-vnet**).
- Set the Public IP to whatever it has automatically assigned to (you might have to confirm the selection).
- Then press "Review + create".
- If Validation passed, click "Create".
<p align="center">
<img src="https://i.imgur.com/snw2Byu.jpg" height="100%" width="100%" alt="Disk Sanitization Steps"/>
</p>

<h3>Connecting to VM1 and Installing Wireshark</h3>

- From Azure Portal, go to VM1's Overview page and copy the Public IP address.
- Press the Windows Key/Button, then type in "Remote Desktop Connection" (RDP).
- Input the IP into RDP and click "Connect" (this example uses **20.171.65.41**).
- Enter the login credentials for VM1, then click "OK" (this example uses **winuser**).
- When the Certificate Error prompt appears, just click "Yes".
- As it boots up, you can disable all privacy settings when prompted, then hit "Accept".
<p align="center">
<img src="https://i.imgur.com/4JNReKH.jpg" height="100%" width="100%" alt="Disk Sanitization Steps"/>
<img src="https://i.imgur.com/GyC1qlp.jpg" height="100%" width="100%" alt="Disk Sanitization Steps"/>
</p>

- On VM1, open Microsoft Edge (or any internet browser), then go to the Wireshark download page.
  - You can simply Google Search it, or copy the link shown.
- Click on "Windows Intel Installer" to start downloading the executable.
- Once downloaded, click "Open file" to run the .exe file (you can also find this inside your Downloads folder within Windows Explorer)
<p align="center">
<img src="https://i.imgur.com/chUmI2q.jpg" height="100%" width="100%" alt="Disk Sanitization Steps"/>
</p>

- The installation prompt will appear, hit "Next".
- When the installation prompt appears, leave everything by default and keep pressing "Next" until you start Installing.
- If any agreement prompts appear during installation, just agree to them and click install (without checkmarking anything).
- After all installations are complete, click "Finish".
<p align="center">
<img src="https://i.imgur.com/kZJSHgb.jpg" height="100%" width="100%" alt="Disk Sanitization Steps"/>
</p>

<h3>Observe ICMP Traffic using Wireshark</h3>

- While in the virtual machine, run Wireshark.
- Click the first button at the top (blue shark fin) to start capturing activity on the VM.
  - You can see there is activity constantly going in the background of the VM, despite you not doing anything.
<p align="center">
<img src="https://i.imgur.com/vVm7yX4.jpg" height="30%" width="30%" alt="Disk Sanitization Steps"/>
<img src="https://i.imgur.com/ypuqGby.jpg" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<img src="https://i.imgur.com/vo6Iafl.jpg" height="100%" width="100%" alt="Disk Sanitization Steps"/>
</p>

- Click in the search box above, type in "ICMP", then press ENTER to confirm.
  - You should then see all boxes blank (this is due to having no activity under the ICMP protocol)
<p align="center">
<img src="https://i.imgur.com/a4jfqVg.jpg" height="100%" width="100%" alt="Disk Sanitization Steps"/>
</p>

_Note: The 'ping' tool within Command Prompt (cmd) / PowerShell uses protocol ICMPv4._
- Minimize the virtual machine to the Azure Portal.
- Go to VM2's Overview page and copy the PRIVATE IP address (this example uses **10.0.0.5**).
- Return to VM2, press the Windows Key/Button and seach for "CMD" or "PowerShell".
- Type in `ping -t <Private IP address>` (this example would use command **ping -t 10.0.0.5**).
  - On Wireshark, you should be able to see the results of packets being perpetually sent and received.
<p align="center">
<img src="https://i.imgur.com/a4jfqVg.jpg" height="100%" width="100%" alt="Disk Sanitization Steps"/>
<img src="https://i.imgur.com/QlLMACP.jpg" height="30%" width="30%" alt="Disk Sanitization Steps"/>
<img src="https://i.imgur.com/XnGJJ8b.jpg" height="60%" width="60%" alt="Disk Sanitization Steps"/>
</p>

_While that is infinitely pinging, we'll try to deny those packets and observe what happens next:_
- Minimize the virtual machine to the Azure Portal.
- In the Search Box at the top header, type and select "Network Security Groups".
- Click on "VM2-nsg".
- Go to "Inbound security rules".
- Click "Add"
<p align="center">
<img src="https://i.imgur.com/wrZKCE1.jpg" height="100%" width="100%" alt="Disk Sanitization Steps"/>
</p>

- Change the protocol to "ICMP".
- Change the Action to "Deny" (_we are trying to stop any packet requests from VM1_).
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

<h1><p align=center>(ﾉ^ヮ^)ﾉ*:・ﾟ✧ COMPLETE! ✧ﾟ・:*╰(^ヮ^╰)</p></h1>

<p align=right>DELETE **EVERYTHING!** IN AZURE TO SAVE CREDITS!<br>
If you don't know how to, click <a href="https://github.com/JTYKolesar/azure-start/blob/main/README.md#bonus-delete-all-resources-in-azure">HERE</a>
</p>
