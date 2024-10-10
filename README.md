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
