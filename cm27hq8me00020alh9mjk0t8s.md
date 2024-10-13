---
title: "Deployment of Azure Bastion Host and Service with Terraform for Secure VM Access"
datePublished: Sun Oct 13 2024 11:16:55 GMT+0000 (Coordinated Universal Time)
cuid: cm27hq8me00020alh9mjk0t8s
slug: deployment-of-azure-bastion-host-and-service-with-terraform-for-secure-vm-access
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1728817859641/c53af12d-e276-4e27-85d9-e357de7dd52e.png
tags: azure, devops, infrastructure, terraform, infrastructure-as-code, azure-devops, devops-articles, devops-journey, 90daysofdevops, cloudautomation, terraformwithazure

---

A Bastion Host or Bastion Service is essential for securing remote access to your virtual machines (VMs) in the cloud without exposing them to the public internet. Traditionally, accessing VMs required public IPs, increasing the risk of attacks from malicious actors. With a bastion host, remote desktop (RDP) and SSH connections are securely routed over a private connection, reducing the attack surface.

In our previous article, we exposed the public IP of the virtual machine and established an SSH connection directly using that public IP. While this method works, it introduces potential security risks by exposing the VM to the public internet. To refine our approach and enhance the security of our infrastructure, we will now eliminate the need to attach a public IP. Instead, we’ll leverage Azure Bastion Host and Bastion Service to securely access the VM without exposing it to the internet, ensuring a more secure and controlled environment for remote access.

# Bastion Host vs Azure Bastion Service

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1728610264952/ad3c7b56-eb58-418f-86e8-db7992f0d75c.png align="center")

## Azure Bastion Service

Azure Bastion eliminates the need to set up VPNs, jump servers, or additional security layers by providing a managed, browser-based RDP and SSH connection over SSL. It enables administrators to manage their VMs without needing a public IP address, making the network more secure and less vulnerable to threats. This approach simplifies secure access while maintaining a robust security posture.

## Bastion Host

Setting up a Bastion Host follows a traditional approach where a dedicated virtual machine is deployed in an isolated subnet, separate from the main workload VMs. This Bastion Host has a public IP exposed to the user, enabling secure access to this isolated VM. From the Bastion Host, the user can then connect to the internal workload VMs, ensuring secure connectivity. Once the Bastion Host is configured, the public IP of the main resources is removed, minimizing exposure to the public internet while maintaining secure, internal access to the VMs.

# Resources to be Used in Terraform

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1728610425398/d1400a76-40e5-4114-840b-26b09b572687.png align="center")

In this setup, we’ll utilize several Terraform resources to implement a secure Bastion Host to connect to our infrastructure. The following resources will be used to achieve this:

* **azurerm\_public\_ip**: To allocate a public IP for the Bastion Host.
    
* **azurerm\_bastion\_host**: To provision the Bastion service for secure, browser-based access to VMs.
    
* **azurerm\_network\_interface**: To configure network connectivity for the Bastion Host.
    
* **azurerm\_linux\_virtual\_machine**: For deploying the Bastion Host VM, which also creates Azure-managed disks.
    
* **azurerm\_network\_security\_group** and **azurerm\_network\_security\_rule**: To manage security rules that control inbound and outbound traffic for the Bastion Host.
    
* **azurerm\_network\_interface\_security\_group\_association**: To associate the security group with the network interface.
    
* **azurerm\_subnet**: To configure the subnet where the Bastion Host resides, ensuring it is isolated for enhanced security.
    

This resource configuration ensures a secure setup for managing access to the main workload VMs.

# Deploy Linux Bastion Host (*Traditional Approach*)

In this section, we will deploy a Linux VM in the previously created Bastion subnet, which will serve as a **Bastion server** or **jump server**. This server will act as a secure intermediary, allowing us to establish a safe connection to the workload VM located in the Web-Tier.

1. **Provisioning a Public IP for the Bastion Host**  
    The first step involves creating a **static public IP** for the Bastion Host using the `azurerm_public_ip` resource. This ensures that the Bastion Host has a persistent public IP, making it accessible externally. The IP is allocated with the "Standard" SKU for enhanced availability.
    
2. **Creating a Network Interface for the Bastion Host**  
    Next, we create a **Network Interface** using `azurerm_network_interface`, which binds the public IP to the Bastion Host and connects it to the previously configured Bastion subnet. The private IP is dynamically allocated within this subnet, providing internal network connectivity.
    
3. **Deploying the Linux Virtual Machine as the Bastion Host**  
    The **Bastion Host Linux VM** is provisioned using `azurerm_linux_virtual_machine`. Key configurations include specifying the admin username, SSH key-based password-less authentication, VM size, and network interface association. Additionally, the OS disk settings and RedHat image are defined, ensuring that the Bastion Host is ready to securely handle remote access requests.
    

```yaml
# In some regions bastion azure native bastion service is not available or not conveinient to use. In those cases we can provision our own bastion host

# 1- Public IP for Linux Bastion Host VM
resource "azurerm_public_ip" "bastion_host_publicip" {
    name = "${local.resource_name_prefix}-${var.linux_bastionhost_publicip}"
    resource_group_name = azurerm_resource_group.rg.name
    location = azurerm_resource_group.rg.location
    allocation_method = "Static"
    sku = "Standard"
}

# 2- Network Interface 
resource "azurerm_network_interface" "linux_bastionhost_NIC" {
    name = "${local.resource_name_prefix}-${var.linux_bastionhost_nic_name}"
    location = azurerm_resource_group.rg.location
    resource_group_name = azurerm_resource_group.rg.name
    ip_configuration {
      name = "bastionhost_ip_1"
      subnet_id = azurerm_subnet.bastion_subnet.id
      private_ip_address_allocation = "Dynamic"
      public_ip_address_id = azurerm_public_ip.bastion_host_publicip.id
    }
  
}

# 3- Create Bastion Host Linux Virtual Machine

resource "azurerm_linux_virtual_machine" "bastion_host_linuxvm" {

    name = "${local.resource_name_prefix}-${var.bastionhost_vm_name}"
    computer_name = var.bastionhost_vm_hostname
    admin_username = var.linux_admin_username
    size = var.vm_size
    resource_group_name = azurerm_resource_group.rg.name
    location = azurerm_resource_group.rg.location
    disable_password_authentication = true

    network_interface_ids =[
        azurerm_network_interface.linux_bastionhost_NIC.id
    ]

    admin_ssh_key {
      username = var.linux_admin_username
      public_key = file("${path.module}/ssh-keys/terraform-azure.pub")
    }
    os_disk {
      caching = "ReadWrite"
      storage_account_type = "Standard_LRS"
    }

    source_image_reference {
      publisher = "RedHat"
      offer = "RHEL"
      sku = "83-gen2"
      version = "latest"
    }   

}
```

## Input Variables

The input variables below are being referenced in the above manifest

```yaml
variable "bastion_service_subnet_name" {
    default = "AzureBastionSubnet"
    description = "Dedicated Subnet for Azure Bastion Service"
}
variable "bastion_service_address_prefixes" {
    default = ["10.0.100.0/27"]
    description = "Bastion Service Address Space"
  
}

variable "bastionhost_vm_name" {
    default = "linuxbastion"
    description = "Bastion Host VM Name"
  
}
variable "bastionhost_vm_hostname" {
    default = "linuxbastion"
    description = "Bastion Host VM Name"
  
}

variable "linux_bastionhost_publicip" {
    default = "linux_bastionhost_publicip"
    type = string
    description = "Linux VM BastionHost Public IP"
  
}
variable "linux_bastionhost_nic_name" {
    default = "bastionhost_linuxvm_nic"
    description = "value"
  
}
```

## Moving Private Keys to Bastion Host

Since this Linux VM will function as the Bastion server, it requires the necessary credentials to establish secure connections to the workload VM. To achieve this, we need to transfer the previously generated SSH private keys to the Bastion host. These keys will allow the Bastion server to authenticate against the workload VM, ensuring secure and password-less SSH connections when accessing the workload VM through the Bastion server.

```yaml
# Movoing the private ssh key is important as we are going to take connect from this bastion host vm to our actual workload vm in web tier
resource "null_resource" "null_copy_ssh_privatekey_to_bastionhost" {

    depends_on = [ azurerm_linux_virtual_machine.bastion_host_linuxvm ] # This resource block needs to be executed only after vm creation
    connection {
      host = azurerm_linux_virtual_machine.bastion_host_linuxvm.public_ip_address
      type = "ssh"
      user = azurerm_linux_virtual_machine.bastion_host_linuxvm.admin_username
      private_key = file("${path.module}/ssh-keys/terraform-azure.pem")
    }
    provisioner "file" {
        source = "ssh-keys/terraform-azure.pem"
        destination = "/tmp/terraform-azure.pem"
    }
    ## Remote Exec provisioner 
    provisioner "remote-exec" {
        inline = [ 
            "sudo chmod 400"
        ]
         
    }
  
}
```

In this section, we are automating the process of copying the SSH private key to the Bastion Host using the `null_resource` in Terraform. Here’s the breakdown:

1. **Resource Dependency**: The `null_resource` block is set to execute only after the Bastion Host Linux VM (`azurerm_linux_virtual_machine.bastion_host_linuxvm`) has been created using the `depends_on` argument. This ensures the resource execution order.
    
2. **SSH Connection Configuration**: A secure SSH connection is established with the Bastion Host by providing the public IP address, username, and private key (`terraform-azure.pem`). This connection will allow further actions on the Bastion Host.
    
3. **Provisioning File**: Using the `file` provisioner, the SSH private key (`terraform-azure.pem`) is copied from the local machine to the Bastion Host’s `/tmp` directory. This key will be used later for secure SSH authentication to the workload VM.
    
4. **Remote Execution**: A `remote-exec` provisioner is used to change the permissions of the copied private key on the Bastion Host to `chmod 400`, ensuring secure access.
    

## Bastion Host Public IP Output

```yaml
output "bastion_host_publicip" {
    value = azurerm_linux_virtual_machine.bastion_host_linuxvm.public_ip_address
    description = "Output bastion host linux vm ip"
  
}
```

The public IP of this machine will be displayed on the console so that we can use it to connect to SSH.

This setup creates a fully functional Bastion Host isolated within its subnet, equipped with secure SSH access, and capable of routing secure connections to other VMs in the network.

# Deploy Azure Bastion Service

Azure Bastion is a fully managed Platform-as-a-Service (PaaS) offering by Microsoft that provides secure and seamless RDP and SSH access to virtual machines (VMs) over SSL directly from the Azure portal. With Azure Bastion, you don't need to expose your VMs to the internet or use a public IP address for secure access, as it creates a secure gateway to connect to your VMs in a private network.

```yaml
# Azure Bastion Subnet

resource "azurerm_subnet" "az_bastion_service_subnet" {
    name = var.az_bastion_service_subnet_name # Name cant be anything other than "AzureBastionSubnet".
    address_prefixes = var.az_bastion_service_address_prefixes
    resource_group_name = azurerm_resource_group.rg.name
    virtual_network_name = azurerm_virtual_network.vnet.name
  
}

# Azure Bastion Public IP
resource "azurerm_public_ip" "az_bastion_public_ip" {
    name = "${local.resource_name_prefix}-${var.az_bastion_service_public_ip}"
    allocation_method = "Static"
    location = azurerm_resource_group.rg.location
    resource_group_name = azurerm_resource_group.rg.name
    sku = "Standard"
  
}
# Azure Bastion Service Host

resource "azurerm_bastion_host" "az_bastion_host_svc" {
    name = "${local.resource_name_prefix}-${var.az_bastion_service_name}"
    resource_group_name = azurerm_resource_group.rg.name
    location = azurerm_resource_group.rg.location
    sku = "Standard"
    ip_configuration {
      name = "bastion_ip_config"
      subnet_id = azurerm_subnet.az_bastion_service_subnet.id
      public_ip_address_id = azurerm_public_ip.az_bastion_public_ip.id
    }
  
}
```

### 1\. **Azure Bastion Subnet Configuration**:

* The subnet for Azure Bastion must be explicitly named `AzureBastionSubnet`, which is a mandatory naming convention for deploying the service.
    
* This subnet must be created within the same virtual network (VNet) where the workload VMs reside, ensuring secure connectivity within the private network.
    
* The subnet will be assigned a unique address prefix for IP allocation, defined through the `az_bastion_service_address_prefixes` variable.
    

### 2\. **Azure Bastion Public IP**:

* A **Static Public IP** is required for Azure Bastion, which will act as the public-facing endpoint through which users securely connect to their VMs over SSL.
    
* The public IP is created using the `Standard` SKU to support secure and scalable connections.
    
* The public IP will be allocated to the Bastion host, ensuring that all SSH and RDP connections are routed through this IP address.
    

### 3\. **Azure Bastion Host Service Configuration**:

* The Azure Bastion Host service itself is defined, which is deployed within the previously created `AzureBastionSubnet`.
    
* The service is linked to the public IP and subnet created earlier.
    
* The Bastion host will use the IP configuration block to map the subnet and public IP, enabling secure and seamless access to VMs in the private network.
    
* The SKU of `Standard` ensures high availability and reliability for connections to the VMs.
    

This outline walks through the critical steps of setting up Azure Bastion to provide secure, managed RDP and SSH access to Azure VMs without exposing public IPs, ensuring better security practices for your infrastructure.

# Verification on Portal

## Verify Bastion-Service

**1\. No Public IP associated with the workload VM (hr-dev-web\_azlinux\_vm):**

The workload VM, `hr-dev-web_azlinux_vm`, does not have an associated public IP address. This is crucial for maintaining the security of the infrastructure, as the VM is not directly exposed to the public internet. All access to this VM is now restricted through the private network, with Azure Bastion acting as the intermediary for remote access.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1728815871261/71ed61e8-2818-4fa7-8236-6addadb7c736.png align="center")

**2\. Bastion service deployed in** `hr-dev-az-vnet-default/AzureBastionSubnet` **Subnet:**

Azure Bastion has been deployed in a dedicated subnet named `AzureBastionSubnet` within the virtual network (`hr-dev-az-vnet-default`). This subnet is a secure location specifically designed for Bastion services, ensuring isolation from the workload VM subnets. It provides a secure path to access virtual machines within the VNet.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1728816031189/a6d0e530-f06b-4bcd-87ba-65094d600139.png align="center")

**3\. Explicit Public IP for the Bastion service:**

Unlike the workload VM, the Bastion service has been provisioned with an explicit static public IP address. This public IP is required to allow remote access via the Azure portal's Bastion feature, which facilitates secure RDP/SSH connections to the VMs without the need for a public IP on the workload VM itself.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1728817390214/bf8ffc60-1e96-4305-a86a-9417641c8194.png align="center")

**4\. Taking the connection using the Bastion service:**

Using Azure Bastion, we are able to securely connect to the workload VM (`hr-dev-web_azlinux_vm`) directly from the Azure portal. This connection uses SSL over port 443, which provides encrypted and secure access to the VM through the Bastion service, without exposing the VM to the public internet.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1728817241391/d45e0310-388a-4d04-a922-d3dc2be47915.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1728817286386/aa75ef81-2bbb-43f5-b86e-484ea7374e17.png align="center")

## Verify Bastion Host Linux VM

**1\. The Bastion Host for Linux VM has been deployed as below:**

The Bastion Host VM, which acts as a jump server, has been successfully deployed in the previously configured Bastion subnet. This VM allows secure SSH access to other workload VMs that do not have public IPs. The Bastion Host serves as an entry point for administrative access to private resources.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1728816275387/066116c1-135b-4fdc-9f0a-0a9a7d1a8b8c.png align="center")

**2\. Dedicated network interface for the Bastion Host:**

The Bastion Host VM has its own dedicated network interface (NIC), which is associated with a static public IP. This NIC facilitates the secure connection from external networks to the internal private network via the Bastion VM. All traffic to the workload VM will be routed through this NIC.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1728816196884/5e282e74-d06e-4668-b936-266215b9af99.png align="center")

**3\. Connecting to the Bastion Host VM (Hostname: linuxbastion):**

Using the public IP assigned to the Bastion Host VM, we can establish an SSH connection by using the hostname `linuxbastion`. This connection is essential as it enables access to the private VMs securely without exposing their IPs directly to the public internet.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1728816496903/25756a9b-8c86-4210-9a42-7cde0d122bdc.png align="center")

**4\. Accessing the workload VM (hr-dev-web\_azlinux\_vm) using private IP:**

Once connected to the Bastion Host VM, we can initiate an SSH connection to the workload VM (`hr-dev-web_azlinux_vm`) using its private IP address. This ensures that all connections to the workload VM are made securely within the internal network, without the need for a public IP on the workload VM.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1728817031054/ad15b1d1-ee80-4852-aaff-abd75230701a.png align="center")