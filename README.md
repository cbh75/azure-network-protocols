<p align="center">
<img src="https://i.imgur.com/Ua7udoS.png" alt="Traffic Examination"/>
</p>

<h1>Network Security Groups (NSGs) and Inspecting Traffic Between Azure Virtual Machines</h1>
In this tutorial, we observe various network traffic to and from Azure Virtual Machines with Wireshark as well as experiment with Network Security Groups. <br />


<h2>Environments and Technologies Used</h2>

- Microsoft Azure (Virtual Machines/Compute)
- Remote Desktop
- Various Command-Line Tools
- Various Network Protocols (SSH, RDH, DNS, HTTP/S, ICMP)
- Wireshark (Protocol Analyzer)

<h2>Operating Systems Used </h2>

- Windows 11 (22H2)
- Ubuntu Server 24.04

<br />

<h2>Creating the Virtual Machines</h2>

Go to the Azure portal (https://portal.azure.com) and ensure you are signed into your Microsoft Azure account. Once you have done so, nativagate to your resource groups. An easy way to do this is to find the search bar at the top of the page and type in "resource groups".

<br />

![image](https://github.com/user-attachments/assets/37531ce6-7d74-4779-8364-b08b9a844659)

<br />

Once on the resource groups page, click **Create**. This will take you to a new page where you can enter a name for your new resource group. In this guide, we will call it "rg-network-activities". Then click **Review + create** at the bottom-left corner of the page. Once you see "Validation passed", click **Create**.

<br />

![image](https://github.com/user-attachments/assets/62cc9195-ae0b-4f22-a817-c7b9352c7a92)

<br />

After the resource group has been created, navigate to your virtual machines. An easy way to do this is to type in "virtual machines" in the search bar.

<br />

![1](https://github.com/cbh75/osticket-prereqs/assets/62080815/1c9898b8-14cf-42ff-8322-9586a87f2ba5)

<br />

After clicking on **Virtual Machines** in the search bar, you will be taken to your Virtual Machines page listing all of your VMs. You probably won't have any right now, so click on **Create** in the top left corner. This will produce a drop-down box with the different types of VMs you can create. For this tutorial, we will use the first option, **Azure virtual machine**.

<br />

![2](https://github.com/cbh75/osticket-prereqs/assets/62080815/1cb56175-eceb-480e-9c5a-24c006139e86)

<br />

On the next screen, make sure to choose the resource group we just created in the **Resource group** box.<br />
For the virtual machine name, type "windows-vm" as this virtual machine will be running Windows 11.<br />
For **Image**, choose **Windows 11 Pro**.<br />
The size does not matter too much, but be sure to choose one that will have enough resources to run Wireshark smoothly.<br />
For our administrator account, choose a username and password that fits the criteria. **Be sure to remember these!**

<br />

![image](https://github.com/user-attachments/assets/36a8ce63-8a56-4501-a445-6e39a55c1ad4)
![image](https://github.com/user-attachments/assets/dead7fb2-9081-41b1-903a-1e6de384214f)

<br />

Be sure to check the box at the bottom confirming you have an eligible Windows license.<br />

<br />

Once that is complete, click **Next** until you get to the **Networking** tab. Under the **Virtual network** box, click the blue **Create new** text.

<br />

![image](https://github.com/user-attachments/assets/d20f0c4e-5f61-4143-9231-7143b4c5d397)

<br />

This will open up a new sidebar where we can create a new virtual network. Azure could have done this automatically for us, but since we are using this virtual network for more than one virtual machine, we want to give it a more general name to avoid confusion. In this case, we can title it "vm-vnet", then click **OK**.

<br />

![image](https://github.com/user-attachments/assets/21cc11be-9267-4893-9c3b-63d0809e4b66)

<br />

Once that is done, we can use the remaining default settings, so go ahead and click **Review + create**. Once validation passes, click **Create**. Wait a couple minutes for the virtual machine to deploy, then navigate back to your virtual machine list. You should see **windows-vm** as the only one in the list.

<br />

![image](https://github.com/user-attachments/assets/2df838b1-3520-4e5a-b651-4f97cb010029)

<br />

Once you see **windows-vm** in your virtual machines, we want to go ahead and create a second virtual machine, so click **Create** and then **Azure virtual machine** once again.<br />
On the virtual machine creation screen, make sure **rg-network-activities** is selected for our resource group, just like in the last one.<br />
For our virtual machine name, we want to call this one "linux-vm", as we will be using Ubuntu for this virtual machine.<br />
For image, we want to select **Ubuntu Server 24.04 LTS**.<br />

<br />

![image](https://github.com/user-attachments/assets/79f77730-e442-4da1-92d2-a9b193c20e6d)

<br />

Since we are using Ubuntu, there will be an additional option under **Administrator account** called **Authentication type**. Click on the **Password** bubble, then create a user for your linux virtual machine. This can be whatever you want, but for the sake of simplicity this guide will be using the same username and password as the one for the windows machine. Once you have a username and password, click **Next** until you get to the **Networking** tab. Make sure the **virtual network** box has our **vm-vnet** network that we created when making our other virtual machine selected.

<br />

![image](https://github.com/user-attachments/assets/85c0f189-f179-40f2-a6b9-39b374ef0c6d)

<br />

Once you have made sure of that, we can leave the remaining options as their defaults, so click **Review + create** at the bottom. Once validation passes, click **Create**. Give Azure a few minutes to create the Linux machine, then go back to the virtual machines page and ensure that both the Windows machine and the Linux machine are there. Pay attention to the **Operating system** column to ensure you have created both machines correctly.

<br />

![image](https://github.com/user-attachments/assets/5d4c31fa-5ed5-4162-b730-a842fb4fd573)

<br />

<h2>Connecting to the Virtual Machines and Viewing Network Traffic</h2>

On the virtual machine page in the Azure portal, click on your Windows machine and copy the **Public** IP address. We will use this address to connect to our virtual machine using Remote Desktop Connection.

<br />

![image](https://github.com/user-attachments/assets/a19434db-fa81-4e6a-8c6b-baa478f7adcf)

<br />

With our public address copied, open Remote Desktop Connection by searching for it in the Windows Start menu.

<br />

![image](https://github.com/user-attachments/assets/86f40912-1a46-4202-818f-0f39ad13da54)

<br />

In the **Computer** box, paste the IP address we just copied, then click **Connect**. In the box that pops up, click on **More choices**, **Use a different account**, then type in the username and password you set up when creating the machine, then click **OK**. If you get a pop-up box saying the identity of the remote computer cannot be verified, click **Yes**. We just created this virtual machine, so we know its identity.

<br />

![image](https://github.com/user-attachments/assets/34e03bff-6576-4b01-9754-691950292989)

<br />

Once you have logged in, give the virtual machine a couple seconds to log in, then you will be asked to choose privacy settings. The default settings are fine, so click **Next**, then **Accept**. Afterwards, you will be greeted by the Windows desktop.<br />

Next, download [Wireshark](www.wireshark.org) onto the virtual machine. Once downloaded, navigate to your **Downloads** folder through Windows Explorer or click the **Downloads** tab in Microsoft Edge, and open the Wireshark installer we just downloaded.

<br />

![image](https://github.com/user-attachments/assets/d1e6c778-ee61-4349-8b79-02e32e05f15c)

<br />

Once opened, navigate through the install wizard. The default settings are fine, so there is no need to worry about choosing the wrong ones. The wizard will install additional software called Npcap; this is totally normal. Once the installer finishes, type "wireshark" into the Windows search bar, then open Wireshark.

<br />

![image](https://github.com/user-attachments/assets/048a0da9-290c-48a3-a2b7-d7831c2a821c)

<br />

Once Wireshark is open, click the blue shark fin icon in the top-left corner. This will start capturing all the network packets being sent and received by the virtual machine.

<br />

![image](https://github.com/user-attachments/assets/a1c5b6f7-0feb-417d-9a7e-07d27ac8dcb8)

<br />

Once clicked, you will immediately notice all the packets being flooded onto the screen. This is normal, as it is tracking every single packet used by the machine.<br />

<br />

To make things actually readable, we are going to apply display filter so only certain protocols will be displayed by Wireshark. The first one we will use is the Internet Control Message Protocol, or ICMP for short. This is the protocol used by the ping command, so that is how we will test out Wireshark. In the top box where it says **Apply a display filter**, type "icmp". You will notice the box turns from red to green, as Wireshark recognizes that as a valid filter. Press enter.

<br />

![image](https://github.com/user-attachments/assets/c8f44b2c-cb16-42c1-a7d9-e41b326092d6)

<br />

The first thing you will notice is that the entire screen within Wireshark got cleared. This is because of all of the packets being sent and received, none of them are using ICMP. To change this, we can use Windows Powershell commands to send packets using ICMP. Open a Powershell window by searching in the Windows Search bar.

<br />

![image](https://github.com/user-attachments/assets/ac0cccfd-c024-44bb-8896-c26b7fd82b8b)

For the first example, we are going to ping our Ubuntu virtual machine. To do this, we will need our Ubuntu machine's **Internal** IP address. We use the Internal address instead of the External address because we configured both of our virtual machines to be on the same network, **vm-vnet**. To find our Ubuntu machine's internal address, navigate back to the virtual machines page in the Azure portal, then click on **linux-vm**. On the linux-vm page, find the **Private IP address** and copy it. This is another name for the internal IP address, along with **Public IP address** being another name for the external address. 

<br />

![image](https://github.com/user-attachments/assets/18baf281-7f3b-4d48-8438-7a760b7491a8)

<br />

Once you have copied the private IP address, navigate back into the Windows virtual machine and in the Powershell window we opened, type in "ping (private ip address)", then hit enter.

<br />

![image](https://github.com/user-attachments/assets/83a70f90-99ec-48b7-942d-413e968306fe)

<br />

As shown in the above image, the packets sent from our ping command show up in Wireshark because said packets are using ICMP. You can see that for each reply we got back from our Linux machine, we got a corresponding packet displayed in Wireshark, along with a request packet for each. You can also use ping for external IP addresses, and by extension any website. Back in the Powershell window, type in "ping www.google.com". As long as Google is not down (I really hope not!), we will see the replies from www.google.com, along with our request packets.

<br />

![image](https://github.com/user-attachments/assets/509fbd4c-911a-4d75-8d6e-ea9a94531d03)

<br />

And with that, we have observed our first network layer protocol!

<br />

<h2>Viewing More Network Traffic</h2>
