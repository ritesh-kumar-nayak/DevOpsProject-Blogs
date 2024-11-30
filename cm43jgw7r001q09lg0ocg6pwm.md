---
title: "Azure Application Gateway Implementation via Terraform"
datePublished: Sat Nov 30 2024 02:13:58 GMT+0000 (Coordinated Universal Time)
cuid: cm43jgw7r001q09lg0ocg6pwm
slug: azure-application-gateway-implementation-via-terraform
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1732932599255/3aa49eca-40a3-4af8-81e3-b4ab995d1939.png
tags: azure, devops, infrastructure-as-code, azure-devops, azure-application-gateway, devops-articles, terraformwithazure

---

**Azure Application Gateway** is a web traffic load balancer that helps you manage traffic to your web applications. Unlike traditional load balancers, it operates at the **application layer (Layer 7)** of the OSI model.

## Use-Case

1. **Hosting Multi-Tier Applications**  
    Route requests to different services within your application based on the URL path.  
    Example: `/auth` routes to the authentication service, while `/orders` routes to the order management system.
    
2. **Protecting Applications with WAF**  
    Use the **WAF** feature to secure your applications against web vulnerabilities, especially when hosting sensitive data or user-facing applications.
    
3. **Serving Multiple Applications on a Single Gateway**  
    Host multiple websites or applications using different domain names or subdomains with a single Application Gateway instance.
    
4. **Modernizing Legacy Applications**  
    Implement SSL offloading and URL-based routing to improve the performance of legacy systems that lack such capabilities.
    
5. **Scaling Secure Web Applications**  
    Use autoscaling and load balancing to ensure availability and performance for applications experiencing variable or high traffic loads.
    
6. **Global Applications with Centralized Security**  
    Centralize security policies for globally distributed applications using a single WAF-enabled Application Gateway.
    

# Project Overview

In this project, we are extending our network architecture by introducing a dedicated **Application Gateway Subnet**, similar to the previously configured **App Subnet** and **DB Subnet**.

### **Key Configurations for Azure Application Gateway:**

1️⃣ **Application Gateway Backend Pool:**

* Associate the pool with the Web **VMSS (Virtual Machine Scale Set)** for handling application traffic.
    

2️⃣ **Frontend IP Configuration:**

* Link the **Frontend IP object** with a public IP to enable external access.
    

3️⃣ **Listeners Configuration:**

* Configure **listeners** to monitor incoming requests on **port 80**.
    

4️⃣ **Backend HTTP Settings:**

* Define settings to connect the **backend pool** (Web VMSS) with appropriate HTTP configurations.
    

5️⃣ **Routing Rules:**

* Establish routing **rules** to associate listeners with backend pools and HTTP settings.
    

6️⃣ **Health Probe Configuration:**

* Enable **health probes** to monitor and ensure the availability of backend instances.
    

By the end of this project, the **Azure Application Gateway** will efficiently route and manage application traffic, ensuring optimal performance and availability.

# Provision Application Gateway Subnet

This Terraform code configures the **Application Gateway Subnet (AG Subnet)**, associates it with a **Network Security Group (NSG)**, and creates inbound rules to allow traffic essential for the Application Gateway.

This setup ensures that the Application Gateway operates securely within its dedicated subnet while allowing only essential traffic. The NSG rules allow HTTP (80), HTTPS (443), and required ephemeral ports (65200-65535), ensuring proper communication and functionality.

```yaml
# Create AG Subnet
resource "azurerm_subnet" "ag_subnet" {
  name                 = "${azurerm_virtual_network.vnet.name}-${var.ag_subnet_name}"
  virtual_network_name = azurerm_virtual_network.vnet.name
  address_prefixes     = var.ag_subnet_address
  resource_group_name  = azurerm_resource_group.rg.name

}

# Create NSG for Subnet
resource "azurerm_network_security_group" "ag_snet_nsg" {

  name                = "${azurerm_subnet.ag_subnet.name}-nsg"
  location            = azurerm_resource_group.rg.location
  resource_group_name = azurerm_resource_group.rg.name

}

# Associate ag_subnet with ag_snet_nsg

resource "azurerm_subnet_network_security_group_association" "associate_agnet_agnsg" {
  depends_on                = [azurerm_network_security_rule.application_gtw_nsg_rules_inbound]
  subnet_id                 = azurerm_subnet.ag_subnet.id
  network_security_group_id = azurerm_network_security_group.ag_snet_nsg.id

}

# Locals block for security rules
locals {
  ag_inbound_port_map = {
    # priority:port
    "100" : "80"
    "110" : "443"
    "130" : "65200-65535"
  }
}

# Create NSG Rules using azurerm_network_security_rule resource

resource "azurerm_network_security_rule" "application_gtw_nsg_rules_inbound" {
  for_each = local.ag_inbound_port_map

  name                        = "Rule_Port_${each.value}"
  access                      = "Allow"
  direction                   = "Inbound"
  network_security_group_name = azurerm_network_security_group.ag_snet_nsg.name
  priority                    = each.key
  protocol                    = "Tcp"
  source_port_range           = "*"
  destination_port_range      = each.value
  source_address_prefix       = "*"
  destination_address_prefix  = "*"
  resource_group_name         = azurerm_resource_group.rg.name

}
```

#### **1\. Create an Application Gateway Subnet**

The `azurerm_subnet` resource creates a subnet dedicated to the Application Gateway.

* **Key Attributes:**
    
    * `address_prefixes`: Specifies the address range for the subnet.
        
    * `virtual_network_name`: Associates the subnet with the existing Virtual Network.
        
    * `resource_group_name`: Links the subnet to the resource group.
        

#### **2\. Create Network Security Group (NSG)**

The `azurerm_network_security_group` resource sets up an NSG for securing the Application Gateway Subnet.

* **Key Attributes:**
    
    * `location`: Specifies the region of the NSG.
        
    * `resource_group_name`: Associates the NSG with the same resource group as the subnet.
        

#### **3\. Associate NSG with the Subnet**

The `azurerm_subnet_network_security_group_association` resource links the Application Gateway Subnet with its NSG.

* **Key Attributes:**
    
    * `subnet_id`: Identifies the Application Gateway Subnet.
        
    * `network_security_group_id`: Associates the NSG with the subnet.
        
* **Dependency:** This association is dependent on the creation of NSG rules to ensure proper configuration.
    

#### **4\. Define Local Map for Port Rules**

The `locals` block contains a map of inbound ports with their respective priorities for the Application Gateway.

* **Example:**
    
    * Port 80 for HTTP (priority 100).
        
    * Port 443 for HTTPS (priority 110).
        
    * Ports 65200–65535 for internal communication (priority 130).
        

#### **5\. Create NSG Rules**

The `azurerm_network_security_rule` resource dynamically generates inbound security rules using the `for_each` loop based on the port map defined in the `locals` block.

* **Key Attributes:**
    
    * `priority`: Ensures the order of rule evaluation.
        
    * `destination_port_range`: Specifies the target port(s) allowed.
        
    * `protocol`: Restricts to TCP traffic.
        
    * `direction`: Allows only inbound traffic.
        
    * `access`: Permits traffic matching the rule.
        

### Ephemeral Ports

Ephemeral ports, also known as **dynamic ports**, are short-lived ports automatically allocated by the operating system to facilitate communication between a client and a server. These ports are used as the **source ports** for outbound traffic in a TCP/IP connection and are released once the communication is complete.

### **Why Are Ephemeral Ports Used in Application Gateway?**

For Azure Application Gateway, ephemeral ports are crucial for managing backend connections. Here's why:

1. **Backend Pool Communication**: When the Application Gateway forwards client requests to backend servers (e.g., VMs or VMSS), it uses ephemeral ports for each session to establish a secure connection.
    
2. **Load Balancing**: Helps maintain multiple simultaneous connections to different backend servers, improving performance.
    
3. **Avoiding Port Conflicts**: Using a large range of dynamic ports reduces the risk of port conflicts between connections.
    

### **In Context of the NSG Rules**

In the provided Terraform configuration, the rule to allow ephemeral ports (**65200–65535**) ensures:

1. **Communication Between App Gateway and Backend Pool**: This range allows the Application Gateway to connect to resources in its backend pool, such as VMs or VMSS.
    
2. **Smooth Functioning of HTTP/HTTPS Traffic**: Without this rule, the Application Gateway might fail to establish connections with backend instances.
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1732589878606/29e69e4a-81fb-4c43-8e06-77ae0044dc8b.png align="center")

# Provision Application Gateway

Provisioning the application gateway has multiple sub-components which will be explained below

### **1\. Create a Public IP for the Application Gateway**

This public IP will be used for front-end communication.

```yaml
# Required Resource: Azure Application Gateway Public IP
resource "azurerm_public_ip" "web_application_gateway_publicip" {
    name                = "${local.resource_name_prefix}-web-ag-publicip"
    allocation_method   = "Static"
    resource_group_name = azurerm_resource_group.rg.name
    location            = azurerm_resource_group.rg.location
    sku                 = "Standard"
}
```

---

### **2\. Define Local Variables**

These variables simplify naming conventions and support multiple applications with context-path-based routing.

```yaml
# Azure AG Locals Block
locals {
  # Generic Configuration
  frontend_port_name               = "${azurerm_virtual_network.vnet.name}-feport"
  frontend_ip_configuration_name   = "${azurerm_virtual_network.vnet.name}-feipconfig"
  listener_name                    = "${azurerm_virtual_network.vnet.name}-httplistener"
  request_routing_rule1_name       = "${azurerm_virtual_network.vnet.name}-requestrouting1"

  # Application-Specific Configuration (App1)
  backend_address_pool_name_app1   = "${azurerm_virtual_network.vnet.name}-bepap_app1"
  http_setting_name_app1           = "${azurerm_virtual_network.vnet.name}-behttpsettung_app1"
  probe_name_app1                  = "${azurerm_virtual_network.vnet.name}-beprobe_app1"
}
```

---

### **3\. Provision the Application Gateway**

This block provisions the gateway with a **Standard\_v2 SKU** and configures autoscaling, frontend IP, listeners, backend pools, and routing rules.

```yaml
# Resource: Azure Application Gateway
resource "azurerm_application_gateway" "web_application_gateway" {
    name                = "${local.resource_name_prefix}-web_application_gateway"
    location            = azurerm_resource_group.rg.location
    resource_group_name = azurerm_resource_group.rg.name

    # Gateway SKU and Autoscaling
    sku {
      name = "Standard_v2"
      tier = "Standard_v2"
    }
    autoscale_configuration {
      min_capacity = 0
      max_capacity = 10 # Max 125 for Standard_v2 SKU
    }

    # Gateway IP Configuration
    gateway_ip_configuration {
      name     = "ag_ip_config"
      subnet_id = azurerm_subnet.ag_subnet.id
    }

    # Frontend Configuration
    frontend_port {
      name = local.frontend_port_name
      port = 80
    }
    frontend_ip_configuration {
      name                 = local.frontend_ip_configuration_name
      public_ip_address_id = azurerm_public_ip.web_application_gateway_publicip.id
      subnet_id            = azurerm_subnet.ag_subnet.id
    }

    # HTTP Listener
    http_listener {
      name                         = local.listener_name
      frontend_ip_configuration_name = local.frontend_ip_configuration_name
      frontend_port_name           = local.frontend_port_name
      protocol                     = "Http"
    }

    # Backend Configuration for App1
    backend_address_pool {
      name = local.backend_address_pool_name_app1
    }
    backend_http_settings {
      name                  = local.http_setting_name_app1
      cookie_based_affinity = "Disabled"
      port                  = 80
      protocol              = "Http"
      request_timeout       = 60
      probe_name            = local.probe_name_app1
    }

    # Health Probe for App1
    probe {
      name                 = local.probe_name_app1
      host                 = "127.0.0.1"
      interval             = 30
      timeout              = 30
      unhealthy_threshold  = 3
      protocol             = "Http"
      port                 = 80
      path                 = "/app1/status.html"
      match {
        body         = "App1"
        status_code  = ["200"]
      }
    }

    # Request Routing Rule
    request_routing_rule {
      name                        = local.request_routing_rule1_name
      rule_type                   = "Basic"
      http_listener_name          = local.listener_name
      backend_address_pool_name   = local.backend_address_pool_name_app1
      backend_http_settings_name  = local.http_setting_name_app1
    }
}
```

---

### **4\. Key Features of the Configuration**

1. **Public IP**:
    
    * Allocated statically and linked to the Application Gateway.
        
2. **Autoscaling**:
    
    * The gateway dynamically scales between 0 to 10 instances (configurable up to 125).
        
3. **Context-Path-Based Routing**:
    
    * Routes traffic based on the URL path (`/app1` goes to App1 VMSS).
        
4. **Health Probes**:
    
    * Ensures backend VMSS health with custom probes targeting [`http://127.0.0.1/app1/status.html`](http://127.0.0.1/app1/status.html).
        
5. **Listener and Rule Configuration**:
    
    * Listens on port 80 and applies a request-routing rule to backend pools.
        

---

### **5\. Attach the Application Gateway with VMSS Backend**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1732932068013/9ce608cd-b35f-40b1-a96a-9da923066283.png align="center")

Previously, the Virtual Machine Scale Set (VMSS) was configured with a standard load balancer for traffic distribution. In this setup, we have transitioned the VMSS to utilize the **Application Gateway's backend address pool**.

### **How It Works**

1. A **public IP** is provisioned for external communication.
    
2. The **Application Gateway** is deployed in a dedicated subnet.
    
3. **Frontend configuration** is set up for HTTP traffic.
    
4. A **listener** receives incoming requests.
    
5. **Backend pools** and **routing rules** direct traffic to the appropriate VMSS based on the URL path.
    
6. **Health probes** ensure backend availability.
    

---

This Terraform configuration enables a scalable and efficient Azure Application Gateway setup with context-path-based routing and dynamic scaling that is suitable for modern applications.