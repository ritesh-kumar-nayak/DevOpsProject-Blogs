---
title: "Simplifying Auto-Scaling with Terraform: Deploy VMSS Efficiently"
datePublished: Sun Oct 27 2024 09:54:15 GMT+0000 (Coordinated Universal Time)
cuid: cm2rexuy1000809mi4awg5x6o
slug: simplifying-auto-scaling-with-terraform-deploy-vmss-efficiently
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1729925725605/3217717f-1ca1-4c55-94ed-a442c7c354fc.png
tags: azure, devops, terraform, azure-devops, autoscaling, create-a-vm-scale-set-vmss-from-existing-vm-image-stored-compute-gallery, azure-cloudcomputing-devops-webdevelopment-tech-scalability-appservices-vmss-github-linkedinlearning, terraformwithazure

---

**Azure Virtual Machine Scale Sets (VMSS)** – a powerful Azure service designed to automatically manage, scale, and balance multiple virtual machines to support varying workloads. Azure VMSS is a service that allows you to deploy and manage a set of identical virtual machines (VMs) that are grouped. These VMs are automatically distributed across fault and update domains, ensuring high availability. VMSS is ideal for situations where you need to support varying application loads without manual intervention, offering a seamless approach to scale-out (adding instances) and scale-in (removing instances) operations.

We’ll explore the deployment of VMSS in a web-tier environment using Terraform, configuring both manual and autoscaling options to illustrate the flexibility and control VMSS brings to cloud applications.

# Manual Scaling vs Auto-Scaling

* **Manual Scaling**: With manual scaling, you set a predefined instance count for the scale set. VMSS will then deploy that specific number of VMs, which is ideal for consistent, predictable workloads. This allows for precise resource planning and control over your deployment size.
    
* **Autoscaling**: In autoscaling, VMSS dynamically adjusts the number of VM instances based on specific metrics such as CPU utilization, memory usage, or custom metrics defined by the application. This enables you to optimize costs by only using the resources you need when demand is high, then automatically scaling down during low-demand periods.
    

# Manual Scaling - VMSS

In this section, we’ll dive into **manual scaling with Azure Virtual Machine Scale Set (VMSS)**, where you explicitly define the number of virtual machines (VMs) the scale set should deploy. Here, instead of provisioning individual Linux VMs for our web tier, we will create a VMSS configured to spin up **two VMs**.

Once the VMSS is deployed, it will be paired with an **Azure Standard Load Balancer**, which will direct incoming traffic to the backend pool associated with the VM instances. This setup ensures that traffic is evenly distributed across the VMs in the scale set, optimizing availability and reliability for web applications.

By defining the VM count in advance, we take full advantage of VMSS to manage scalability without the need to manage individual VM resources.

```yaml
# Resource VMSS(Virtual machine scale sets)

resource "azurerm_linux_virtual_machine_scale_set" "web_vmss" {
  name     = "${local.resource_name_prefix}-web-vmss"
  admin_username = var.linux_admin_username
  computer_name_prefix = "vmss-web"
  location = azurerm_resource_group.rg.location
  sku = "Standard_DS1_v2"
  resource_group_name = azurerm_resource_group.rg.name

  instances = 2 # Manually defining the number of instances. Hence, it is called manual scaling

  upgrade_mode = "Automatic"  # VMs will be auto upgraded after associated with LB

  admin_ssh_key {
    username   = var.linux_admin_username
    public_key = file("${path.module}/ssh-keys/terraform-azure.pub")
  
  }
  os_disk {
    caching              = "ReadWrite"
    storage_account_type = "Standard_LRS"
  }
  source_image_reference {
    publisher = "RedHat"
    offer     = "RHEL"
    sku       = "83-gen2"
    version   = "latest"
  }
  network_interface {
    name = "vmss-nic"
    primary = true
    network_security_group_id = azurerm_network_security_group.web_vmss_nsg.id

    ip_configuration {
      name = "internal"
      primary = true
      subnet_id = azurerm_subnet.web_subnet.id
      # here the VMSS is now associated with LoadBalancer.
      load_balancer_backend_address_pool_ids = [azurerm_lb_backend_address_pool.lb_backend_address_pool.id]
    }

  }
  custom_data = filebase64("${path.module}/app-script/webvm.sh") # One way of passing custom script

  
}
```

This Terraform configuration creates a **Virtual Machine Scale Set (VMSS)** on Azure for the web tier, with key settings for instance scaling, network configuration, and custom automation.

1. **VM Scale Set Basics**:
    
    * A VMSS named `web_vmss` is defined, set to manually scale to **two instances**. Each instance uses a Linux OS and a specified SKU (`Standard_DS1_v2`) for consistent performance.
        
2. **Auto-Upgrade & OS Settings**:
    
    * The VMSS is configured to automatically upgrade instances, so any changes apply without manual intervention.
        
    * The OS is **Red Hat Enterprise Linux** (RHEL), with settings for disk caching and storage.
        
3. **Network Configuration**:
    
    * Each VM in the scale set connects to an existing **subnet** and is secured by a Network Security Group (NSG).
        
    * It also associates with an **Azure Load Balancer** backend pool, distributing traffic across VMs for better reliability and load management.
        
4. **Custom Initialization Script**:
    
    * A custom script ([`webvm.sh`](http://webvm.sh)) initializes each VM instance with the required settings or software. The script is encoded in base64 for direct use in Terraform.
        

This setup creates a flexible and scalable backend for applications, allowing easy scaling, security, and traffic distribution across multiple instances in the web tier.

## Resource Verification on Azure Portal post Apply

After applying the Terraform configuration, the **Virtual Machine Scale Set (VMSS)** reached the desired state, with two virtual machines successfully up and running.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1729956026058/4b776e82-d52f-4d7a-a8dd-4b9aff32df2c.png align="center")

As discussed, **manual scaling** has been implemented here, where the number of instances is explicitly defined to maintain consistency and control over resources.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1729957201686/af81c5d4-208a-4130-9539-6d1a10086447.png align="center")

Additionally, the **VMSS is linked to the Azure Load Balancer**, effectively distributing traffic between the instances. This setup enables access to the VMs through the Load Balancer's public IP, enhancing availability and reliability for end-users.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1729956183225/06c453fd-5e53-4be9-bf5a-fdc25d38e942.png align="center")

Finally, here is the **content served by the two VMs** managed by the VMSS, showcasing the setup's functionality and the seamless distribution across instances.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1729956290838/28e387fc-cbe3-4564-9a57-e0a5fc936e49.png align="center")

# Auto-Scaling in Azure VMSS

**Auto scaling** in Azure is a feature that dynamically adjusts the number of resources—such as virtual machines (VMs) in a scale set or App Service instances—based on real-time demand and pre-defined criteria. Auto scaling helps optimize costs and ensures application availability by automatically scaling out to handle high loads or scaling in to reduce resources during low demand periods.

Auto-scaling is a stand-alone resource in Azure that can be attached to VMSS or even to app services and other resources. Here in this section, we’ll implement Auto-Scaling with VMSS

## Auto-Scaling Profiles

There are 3 auto-scaling profiles available in Azure

* ### Auto-Scaling Default Profile
    
    This profile allows you to configure automatic scaling based on performance metrics. It adjusts the number of instances in response to real-time resource usage, ensuring that your application can handle fluctuating traffic without manual intervention.
    
* ### Auto-Scaling Recurrence Profile
    
    With this profile, you can set up a schedule for scaling your resources at specific times. This is particularly useful for applications that experience predictable traffic patterns, allowing you to allocate resources efficiently during peak hours and scale down during off-peak times.
    
* ### Auto-Scaling Fixed Profile
    
    This profile enables you to define a static number of instances for your application. It provides a consistent level of resources, making it ideal for workloads that require steady performance without the need for scaling based on demand.
    

## Auto-Scaling Rules Metrics

Below are the metric rules we can create while setting up profiles in Autoscaling.

```yaml
1. Percentage CPU Metric Rules
    1. Scale-Up Rule: Increase VMs by 1 when CPU usage is greater than 75%
    2. Scale-In Rule: Decrease VMs by 1when CPU usage is lower than 25%
2. Available Memory Bytes Metric Rules
    1. Scale-Up Rule: Increase VMs by 1 when Available Memory Bytes is less than 1GB in bytes
    2. Scale-In Rule: Decrease VMs by 1 when Available Memory Bytes is greater than 2GB in bytes
3. LB SYN Count Metric Rules (JUST FOR firing Scale-Up and Scale-In Events for Testing and also knowing in addition to current VMSS Resource, we can also create Autoscaling rules for VMSS based on other Resource usage like Load Balancer)
    1. Scale-Up Rule: Increase VMs by 1 when LB SYN Count is greater than 10 Connections (Average)
    2. Scale-Up Rule: Decrease VMs by 1 when LB SYN Count is less than 10 Connections (Average)
```

# Auto-Scaling Profile Creation

By leveraging the `azurerm_monitor_autoscale_setting` resource, we will establish a flexible scaling strategy based on several key metrics. Here we will create all 3 types of profiles as discussed above one by one.

## Profile-1: Default Profile

* **Introduction to VMSS Auto-Scaling**: An overview of Azure auto-scaling as a means to dynamically adjust the number of virtual machines (VMs) based on real-time resource utilization, traffic patterns, and specific performance **thresholds**.
    
* **Creating the Auto-Scale Resource**:
    
    * **Notification Block**: Setting up automatic email notifications for subscription administrators and co-administrators to keep them informed of scale actions.
        
* **Defining the Default Scaling Profile**:
    
    * **Manual Scaling Capacity**: Initializing with a default of 2 instances and setting bounds for minimum (2) and maximum (6) instances, providing flexibility as demand changes.
        
    * **CPU Usage-Based Scaling Rules**: Configuring scale-in and scale-out actions based on CPU utilization:
        
        * Scale-out is triggered if CPU usage exceeds 75% (adds 1 instance).
            
        * Scale-in occurs when CPU usage drops below 25% (removes 1 instance).
            
* **Memory Availability-Based Scaling Rules**:
    
    * Scale-out adds an instance if available memory is under 1GB.
        
    * Scale-in removes an instance when available memory exceeds 2GB.
        
* **Load Balancer SYN Count Rules**:
    
    * Testing scale actions by monitoring SYN requests to the Load Balancer.
        
    * Scaling actions are triggered based on incoming requests, demonstrating how demand-based metrics can trigger VMSS scaling.
        

```yaml
resource "azurerm_monitor_autoscale_setting" "web_vmss_autoscale" {
    name = "${local.resource_name_prefix}-web-vmss-autoscale-profiles"
    resource_group_name = azurerm_resource_group.rg.name
    location = azurerm_resource_group.rg.location
    target_resource_id = azurerm_linux_virtual_machine_scale_set.web_vmss.id

    #### Notification Block#####
    notification {
      email {
        send_to_subscription_administrator = true
        send_to_subscription_co_administrator = true
        
      }
    }
    #### Profile-1( Default Profile Block ) #####
    profile {
      name = "Default Profile"
      capacity {
        default = 2
        minimum = 2
        maximum = 6
      }

    ### Percentage CPU Metric Rule Begins #####
    ### Scale-Out Rule  ####
    rule {
    scale_action {
        direction = "Increase" # Scale-out means always increase
        type = "ChangeCount"
        value = 1
        cooldown = "PT5M"
    }
    metric_trigger {
        metric_name = "Percentage CPU"
        metric_resource_id = azurerm_linux_virtual_machine_scale_set.web_vmss.id
        metric_namespace = "microsoft.compute/virtualmachinescalesets"
        time_grain = "PT1M"
        statistic = "Average"
        time_window = "PT5M"
        time_aggregation = "Average"
        operator = "GreaterThan"
        threshold = 75
        }
    }

    ### Scale-In Rule ####
    rule {
      scale_action {
        direction = "Decrease"
        type = "ChangeCount"
        value = 1
        cooldown = "PT5M"
      }
      metric_trigger {
        metric_name = "Percentage CPU"
        metric_resource_id = azurerm_linux_virtual_machine_scale_set.web_vmss.id
        metric_namespace = "microsoft.compute/virtualmachinescalesets"
        time_grain = "PT1M"
        statistic = "Average"
        time_window = "PT5M"
        time_aggregation = "Average"
        operator = "LessThan"
        threshold = 25
      }
    }

    ### END OF Percentage CPU Metric Rule ###

  ###########  START: Available Memory Bytes Metric Rules  ###########    
  ## Scale-Out 
    rule {
      scale_action {
        direction = "Increase"
        type      = "ChangeCount"
        value     = 1
        cooldown  = "PT5M"
      }            
      metric_trigger {
        metric_name        = "Available Memory Bytes"
        metric_resource_id = azurerm_linux_virtual_machine_scale_set.web_vmss.id
        metric_namespace   = "microsoft.compute/virtualmachinescalesets"        
        time_grain         = "PT1M"
        statistic          = "Average"
        time_window        = "PT5M"
        time_aggregation   = "Average"
        operator           = "LessThan"
        threshold          = 1073741824 # Increase 1 VM when Memory In Bytes is less than 1GB
      }
    }
    ## Scale-In 
    rule {
      scale_action {
        direction = "Decrease"
        type      = "ChangeCount"
        value     = 1
        cooldown  = "PT5M"
      }        
      metric_trigger {
        metric_name        = "Available Memory Bytes"
        metric_resource_id = azurerm_linux_virtual_machine_scale_set.web_vmss.id
        metric_namespace   = "microsoft.compute/virtualmachinescalesets"                
        time_grain         = "PT1M"
        statistic          = "Average"
        time_window        = "PT5M"
        time_aggregation   = "Average"
        operator           = "GreaterThan"
        threshold          = 2147483648 # Decrease 1 VM when Memory In Bytes is Greater than 2GB
      }
    }
    ###########  END: Available Memory Bytes Metric Rules  ###########  

  ###########  START: LB SYN Count Metric Rules - Just to Test scale-in, scale-out  ###########    
  ## Scale-Out 
    rule {
      scale_action {
        direction = "Increase"
        type      = "ChangeCount"
        value     = 1
        cooldown  = "PT5M"
      }      
      metric_trigger {
        metric_name        = "SYNCount"
        metric_resource_id = azurerm_lb.web_lb.id 
        metric_namespace   = "Microsoft.Network/loadBalancers"        
        time_grain         = "PT1M"
        statistic          = "Average"
        time_window        = "PT5M"
        time_aggregation   = "Average"
        operator           = "GreaterThan"
        threshold          = 10 # 10 requests to an LB
      }
    }
    ## Scale-In 
    rule {
      scale_action {
        direction = "Decrease"
        type      = "ChangeCount"
        value     = 1
        cooldown  = "PT5M"
      }      
      metric_trigger {
        metric_name        = "SYNCount"
        metric_resource_id = azurerm_lb.web_lb.id
        metric_namespace   = "Microsoft.Network/loadBalancers"                
        time_grain         = "PT1M"
        statistic          = "Average"
        time_window        = "PT5M"
        time_aggregation   = "Average"
        operator           = "LessThan"
        threshold          = 10
      }
    }
    ###########  END: LB SYN Count Metric Rules  ###########    

    }   # End of Default Profile Block -1

    
}
```

### Verifying Profile-1 on Azure Portal

Now after applying, the custom auto-scale has been chosen

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1730018751649/31abf745-9d9d-4829-80e2-66929c5167ba.png align="center")

The respective rules have been added to the profile as follows

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1730018928194/9709f88d-c068-40aa-8e99-b8251fe2887a.png align="center")

## Profile-2: Recurrence Profile

A recurrence profile for auto-scaling with settings specific to weekdays (Monday to Friday) and CPU, memory, and load balancer metrics for scaling actions.

```yaml
  ##### Profile-2: Recurrence Profile - Week Days
  profile {
    name = "profile-2-weekdays"
  # Capacity Block     
    capacity {
      default = 4
      minimum = 4
      maximum = 20
    }
  # Recurrence Block for Week Days (5 days)
    recurrence {
      timezone = "India Standard Time"
      days = ["Monday", "Tuesday", "Wednesday", "Thursday", "Friday"]
      hours = [0]
      minutes = [0]      
    }    
###########  START: Percentage CPU Metric Rules  ###########    
  ## Scale-Out 
    rule {
      scale_action {
        direction = "Increase"
        type      = "ChangeCount"
        value     = 1
        cooldown  = "PT5M"
      }            
      metric_trigger {
        metric_name        = "Percentage CPU"
        metric_resource_id = azurerm_linux_virtual_machine_scale_set.web_vmss.id
        metric_namespace   = "microsoft.compute/virtualmachinescalesets"        
        time_grain         = "PT1M"
        statistic          = "Average"
        time_window        = "PT5M"
        time_aggregation   = "Average"
        operator           = "GreaterThan"
        threshold          = 75
      }
    }

  ## Scale-In 
    rule {
      scale_action {
        direction = "Decrease"
        type      = "ChangeCount"
        value     = 1
        cooldown  = "PT5M"
      }        
      metric_trigger {
        metric_name        = "Percentage CPU"
        metric_resource_id = azurerm_linux_virtual_machine_scale_set.web_vmss.id
        metric_namespace   = "microsoft.compute/virtualmachinescalesets"                
        time_grain         = "PT1M"
        statistic          = "Average"
        time_window        = "PT5M"
        time_aggregation   = "Average"
        operator           = "LessThan"
        threshold          = 25
      }
    }
###########  END: Percentage CPU Metric Rules   ###########    

###########  START: Available Memory Bytes Metric Rules  ###########    
  ## Scale-Out 
    rule {
      scale_action {
        direction = "Increase"
        type      = "ChangeCount"
        value     = 1
        cooldown  = "PT5M"
      }            
      metric_trigger {
        metric_name        = "Available Memory Bytes"
        metric_resource_id = azurerm_linux_virtual_machine_scale_set.web_vmss.id
        metric_namespace   = "microsoft.compute/virtualmachinescalesets"        
        time_grain         = "PT1M"
        statistic          = "Average"
        time_window        = "PT5M"
        time_aggregation   = "Average"
        operator           = "LessThan"
        threshold          = 1073741824 # Increase 1 VM when Memory In Bytes is less than 1GB
      }
    }

  ## Scale-In 
    rule {
      scale_action {
        direction = "Decrease"
        type      = "ChangeCount"
        value     = 1
        cooldown  = "PT5M"
      }        
      metric_trigger {
        metric_name        = "Available Memory Bytes"
        metric_resource_id = azurerm_linux_virtual_machine_scale_set.web_vmss.id
        metric_namespace   = "microsoft.compute/virtualmachinescalesets"                
        time_grain         = "PT1M"
        statistic          = "Average"
        time_window        = "PT5M"
        time_aggregation   = "Average"
        operator           = "GreaterThan"
        threshold          = 2147483648 # Decrease 1 VM when Memory In Bytes is Greater than 2GB
      }
    }
###########  END: Available Memory Bytes Metric Rules  ###########  


###########  START: LB SYN Count Metric Rules - Just to Test scale-in, scale-out  ###########    
  ## Scale-Out 
    rule {
      scale_action {
        direction = "Increase"
        type      = "ChangeCount"
        value     = 1
        cooldown  = "PT5M"
      }      
      metric_trigger {
        metric_name        = "SYNCount"
        metric_resource_id = azurerm_lb.web_lb.id 
        metric_namespace   = "Microsoft.Network/loadBalancers"        
        time_grain         = "PT1M"
        statistic          = "Average"
        time_window        = "PT5M"
        time_aggregation   = "Average"
        operator           = "GreaterThan"
        threshold          = 10 # 10 requests to an LB
      }
    }
  ## Scale-In 
    rule {
      scale_action {
        direction = "Decrease"
        type      = "ChangeCount"
        value     = 1
        cooldown  = "PT5M"
      }      
      metric_trigger {
        metric_name        = "SYNCount"
        metric_resource_id = azurerm_lb.web_lb.id
        metric_namespace   = "Microsoft.Network/loadBalancers"                
        time_grain         = "PT1M"
        statistic          = "Average"
        time_window        = "PT5M"
        time_aggregation   = "Average"
        operator           = "LessThan"
        threshold          = 10
      }
    }
###########  END: LB SYN Count Metric Rules  ###########    
  } 
  ##### End of Profile-2
```

1. **Profile-2 Overview**:
    
    * **Name**: Profile-2 is designated as the "profile-2-weekdays."
        
    * **Capacity Constraints**: Configures the scale set to maintain a minimum of 4 VMs, a maximum of 20, and a default of 4 VMs, adjusting dynamically within these boundaries.
        
2. **Recurrence Block**:
    
    * Sets the auto-scaling action to apply from **Monday through Friday** at **00:00 hours** in **India Standard Time**.
        
    * Ensures scaling actions only trigger during specified weekday hours, optimizing resource allocation based on anticipated load patterns.
        
3. **Scaling Rules for CPU Usage (Percentage CPU Metric Rules)**:
    
    * **Scale-Out Rule**: Increases the instance count by 1 if average CPU usage exceeds 75%.
        
    * **Scale-In Rule**: Decreases the instance count by 1 if average CPU usage falls below 25%.
        
    * Each rule has a 5-minute cooldown period, allowing sufficient time for load adjustment before further scaling.
        
4. **Scaling Rules for Memory Availability (Available Memory Bytes Metric Rules)**:
    
    * **Scale-Out Rule**: Triggers scaling out by adding 1 instance when available memory falls below 1GB.
        
    * **Scale-In Rule**: Triggers scaling in by removing 1 instance when available memory exceeds 2GB.
        
    * This approach ensures resource availability by adding VMs as memory demand increases and reducing VMs when memory is ample.
        
5. **SYN Count-Based Scaling Rules (Load Balancer Metric Rules)**:
    
    * Monitors load balancer **SYN request count** for scale adjustments, beneficial for high-traffic scenarios.
        
    * **Scale-Out Rule**: Adds 1 VM if the SYN request count exceeds 10.
        
    * **Scale-In Rule**: Removes 1 VM if the SYN request count falls below 10, stabilizing resource usage during traffic fluctuations.
        

Each section within Profile 2 enables adaptive scaling based on real-time traffic and workload patterns, enhancing availability, optimizing costs, and aligning resource usage with demand.

### Verifying Profile-2 on Azure Portal

Now post applying the Recurrence profile i.e. profile 2 is now in action where all the rules have been added respectively

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1730019761675/f4b4577d-34db-4242-bb79-46e9c1176895.png align="center")

While Profile-2 is in effect, it takes precedence, meaning that the Default Profile will not execute concurrently. This ensures that scaling adjustments occur according to the current profile’s configuration, based on weekday and demand metrics, optimizing resource alignment with operational needs.

## Profile-3: Recurrence Profile (Weekend)

This profile manages auto-scaling configurations specifically for weekends, ensuring optimal resource allocation during lower usage periods with adjustments triggered by CPU, memory, and load balancer metrics.

```yaml
  profile {
    name = "profile-3-weekends"
  # Capacity Block     
    capacity {
      default = 3
      minimum = 3
      maximum = 6
    }
  # Recurrence Block for Weekends (2 days)
    recurrence {
      timezone = "India Standard Time"
      days = ["Saturday", "Sunday"]
      hours = [0]
      minutes = [0]      
    }    
###########  START: Percentage CPU Metric Rules  ###########    
  ## Scale-Out 
    rule {
      scale_action {
        direction = "Increase"
        type      = "ChangeCount"
        value     = 1
        cooldown  = "PT5M"
      }            
      metric_trigger {
        metric_name        = "Percentage CPU"
        metric_resource_id = azurerm_linux_virtual_machine_scale_set.web_vmss.id
        metric_namespace   = "microsoft.compute/virtualmachinescalesets"        
        time_grain         = "PT1M"
        statistic          = "Average"
        time_window        = "PT5M"
        time_aggregation   = "Average"
        operator           = "GreaterThan"
        threshold          = 75
      }
    }

  ## Scale-In 
    rule {
      scale_action {
        direction = "Decrease"
        type      = "ChangeCount"
        value     = 1
        cooldown  = "PT5M"
      }        
      metric_trigger {
        metric_name        = "Percentage CPU"
        metric_resource_id = azurerm_linux_virtual_machine_scale_set.web_vmss.id
        metric_namespace   = "microsoft.compute/virtualmachinescalesets"                
        time_grain         = "PT1M"
        statistic          = "Average"
        time_window        = "PT5M"
        time_aggregation   = "Average"
        operator           = "LessThan"
        threshold          = 25
      }
    }
###########  END: Percentage CPU Metric Rules   ###########    

###########  START: Available Memory Bytes Metric Rules  ###########    
  ## Scale-Out 
    rule {
      scale_action {
        direction = "Increase"
        type      = "ChangeCount"
        value     = 1
        cooldown  = "PT5M"
      }            
      metric_trigger {
        metric_name        = "Available Memory Bytes"
        metric_resource_id = azurerm_linux_virtual_machine_scale_set.web_vmss.id
        metric_namespace   = "microsoft.compute/virtualmachinescalesets"        
        time_grain         = "PT1M"
        statistic          = "Average"
        time_window        = "PT5M"
        time_aggregation   = "Average"
        operator           = "LessThan"
        threshold          = 1073741824 # Increase 1 VM when Memory In Bytes is less than 1GB
      }
    }

  ## Scale-In 
    rule {
      scale_action {
        direction = "Decrease"
        type      = "ChangeCount"
        value     = 1
        cooldown  = "PT5M"
      }        
      metric_trigger {
        metric_name        = "Available Memory Bytes"
        metric_resource_id = azurerm_linux_virtual_machine_scale_set.web_vmss.id
        metric_namespace   = "microsoft.compute/virtualmachinescalesets"                
        time_grain         = "PT1M"
        statistic          = "Average"
        time_window        = "PT5M"
        time_aggregation   = "Average"
        operator           = "GreaterThan"
        threshold          = 2147483648 # Decrease 1 VM when Memory In Bytes is Greater than 2GB
      }
    }
###########  END: Available Memory Bytes Metric Rules  ###########  

###########  START: LB SYN Count Metric Rules - Just to Test scale-in, scale-out  ###########    
  ## Scale-Out 
    rule {
      scale_action {
        direction = "Increase"
        type      = "ChangeCount"
        value     = 1
        cooldown  = "PT5M"
      }      
      metric_trigger {
        metric_name        = "SYNCount"
        metric_resource_id = azurerm_lb.lb_web.id 
        metric_namespace   = "Microsoft.Network/loadBalancers"        
        time_grain         = "PT1M"
        statistic          = "Average"
        time_window        = "PT5M"
        time_aggregation   = "Average"
        operator           = "GreaterThan"
        threshold          = 10 # 10 requests to an LB
      }
    }
  ## Scale-In 
    rule {
      scale_action {
        direction = "Decrease"
        type      = "ChangeCount"
        value     = 1
        cooldown  = "PT5M"
      }      
      metric_trigger {
        metric_name        = "SYNCount"
        metric_resource_id = azurerm_lb.lb_web.id
        metric_namespace   = "Microsoft.Network/loadBalancers"                
        time_grain         = "PT1M"
        statistic          = "Average"
        time_window        = "PT5M"
        time_aggregation   = "Average"
        operator           = "LessThan"
        threshold          = 10
      }
    }
###########  END: LB SYN Count Metric Rules  ###########    
} # End of Profile-3
```

1. **Capacity Settings**
    
    * **Default**: 3 instances
        
    * **Minimum**: 3 instances
        
    * **Maximum**: 6 instances
        
2. **Recurrence Settings**
    
    * **Days**: Saturday and Sunday
        
    * **Time**: 00:00 hours (India Standard Time)
        
3. **Scaling Rules**
    
    * **CPU Utilization (Percentage CPU)**
        
        * **Scale-Out**: Increase by 1 instance when average CPU &gt; 75% over a 5-minute period.
            
        * **Scale-In**: Decrease by 1 instance when average CPU &lt; 25% over a 5-minute period.
            
    * **Memory Usage (Available Memory Bytes)**
        
        * **Scale-Out**: Increase by 1 instance when available memory is &lt; 1GB.
            
        * **Scale-In**: Decrease by 1 instance when available memory &gt; 2GB.
            
    * **Load Balancer Requests (SYNCount)**
        
        * **Scale-Out**: Increase by 1 instance when SYN requests exceed 10 over a 5-minute period.
            
        * **Scale-In**: Decrease by 1 instance when SYN requests fall below 10 over a 5-minute period.
            

This weekend-specific profile ensures that resources are efficiently managed with minimal instances during off-peak times, responding dynamically to weekend traffic demands through auto-scaling adjustments.

### Verifying Profile-3 on Azure Portal

The **Weekend Profile** (Profile-3) has now been successfully applied, configured with specific scaling rules to operate exclusively on Saturdays and Sundays. This ensures that the auto-scaling behavior aligns with weekend traffic patterns, repeating every Saturday and Sunday as specified in the recurrence settings.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1730020869354/ed653be4-053f-4e82-9da9-91cea798565c.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1730020748070/2f993bd6-86d4-428a-845e-de32fad452c0.png align="center")

With Profile-3 in action, scaling adjustments will adhere to the defined metrics, keeping resources optimized during weekend periods.

## Profile-4: Fixed Profile (Specific Dates)

It is a targeted scaling profile that activates only on a specified date. It provides flexibility for scaling to meet anticipated needs for a specific day or event.

```yaml
# Profile-4: Fixed Profile for a Specific Day
  profile {
    name = "profile-4-fixed-profile"
  # Capacity Block     
    capacity {
      default = 5
      minimum = 5
      maximum = 20
    }
  # Fixed Block for a specific day
    fixed_date {
      timezone = "India Standard Time"
      start    = "2021-08-16T00:00:00Z"  # CHANGE TO THE DATE YOU ARE TESTING
      end      = "2021-08-16T23:59:59Z"  # CHANGE TO THE DATE YOU ARE TESTING
    }  
###########  START: Percentage CPU Metric Rules  ###########    
  ## Scale-Out 
    rule {
      scale_action {
        direction = "Increase"
        type      = "ChangeCount"
        value     = 1
        cooldown  = "PT5M"
      }            
      metric_trigger {
        metric_name        = "Percentage CPU"
        metric_resource_id = azurerm_linux_virtual_machine_scale_set.web_vmss.id
        metric_namespace   = "microsoft.compute/virtualmachinescalesets"        
        time_grain         = "PT1M"
        statistic          = "Average"
        time_window        = "PT5M"
        time_aggregation   = "Average"
        operator           = "GreaterThan"
        threshold          = 75
      }
    }

  ## Scale-In 
    rule {
      scale_action {
        direction = "Decrease"
        type      = "ChangeCount"
        value     = 1
        cooldown  = "PT5M"
      }        
      metric_trigger {
        metric_name        = "Percentage CPU"
        metric_resource_id = azurerm_linux_virtual_machine_scale_set.web_vmss.id
        metric_namespace   = "microsoft.compute/virtualmachinescalesets"                
        time_grain         = "PT1M"
        statistic          = "Average"
        time_window        = "PT5M"
        time_aggregation   = "Average"
        operator           = "LessThan"
        threshold          = 25
      }
    }
###########  END: Percentage CPU Metric Rules   ###########    

###########  START: Available Memory Bytes Metric Rules  ###########    
  ## Scale-Out 
    rule {
      scale_action {
        direction = "Increase"
        type      = "ChangeCount"
        value     = 1
        cooldown  = "PT5M"
      }            
      metric_trigger {
        metric_name        = "Available Memory Bytes"
        metric_resource_id = azurerm_linux_virtual_machine_scale_set.web_vmss.id
        metric_namespace   = "microsoft.compute/virtualmachinescalesets"        
        time_grain         = "PT1M"
        statistic          = "Average"
        time_window        = "PT5M"
        time_aggregation   = "Average"
        operator           = "LessThan"
        threshold          = 1073741824 # Increase 1 VM when Memory In Bytes is less than 1GB
      }
    }

  ## Scale-In 
    rule {
      scale_action {
        direction = "Decrease"
        type      = "ChangeCount"
        value     = 1
        cooldown  = "PT5M"
      }        
      metric_trigger {
        metric_name        = "Available Memory Bytes"
        metric_resource_id = azurerm_linux_virtual_machine_scale_set.web_vmss.id
        metric_namespace   = "microsoft.compute/virtualmachinescalesets"                
        time_grain         = "PT1M"
        statistic          = "Average"
        time_window        = "PT5M"
        time_aggregation   = "Average"
        operator           = "GreaterThan"
        threshold          = 2147483648 # Decrease 1 VM when Memory In Bytes is Greater than 2GB
      }
    }
###########  END: Available Memory Bytes Metric Rules  ###########  


###########  START: LB SYN Count Metric Rules - Just to Test scale-in, scale-out  ###########    
  ## Scale-Out 
    rule {
      scale_action {
        direction = "Increase"
        type      = "ChangeCount"
        value     = 1
        cooldown  = "PT5M"
      }      
      metric_trigger {
        metric_name        = "SYNCount"
        metric_resource_id = azurerm_lb.web_lb.id 
        metric_namespace   = "Microsoft.Network/loadBalancers"        
        time_grain         = "PT1M"
        statistic          = "Average"
        time_window        = "PT5M"
        time_aggregation   = "Average"
        operator           = "GreaterThan"
        threshold          = 10 # 10 requests to an LB
      }
    }
  ## Scale-In 
    rule {
      scale_action {
        direction = "Decrease"
        type      = "ChangeCount"
        value     = 1
        cooldown  = "PT5M"
      }      
      metric_trigger {
        metric_name        = "SYNCount"
        metric_resource_id = azurerm_lb.web_lb.id
        metric_namespace   = "Microsoft.Network/loadBalancers"                
        time_grain         = "PT1M"
        statistic          = "Average"
        time_window        = "PT5M"
        time_aggregation   = "Average"
        operator           = "LessThan"
        threshold          = 10
      }
    }
###########  END: LB SYN Count Metric Rules  ###########    
} # End of Profile-4
```

1. **Profile Name**: "profile-4-fixed-profile"
    
2. **Capacity Block**:
    
    * Sets a default, minimum, and maximum capacity for the VM scale set:
        
        * Default: 5 instances
            
        * Minimum: 5 instances
            
        * Maximum: 20 instances
            
3. **Fixed Date Block**:
    
    * Defines the profile's fixed timeframe:
        
        * Time Zone: "India Standard Time"
            
        * Start: The specific start date and time for this profile.
            
        * End: The specific end date and time for this profile.
            
    * Example: `start = "2021-08-16T00:00:00Z"` and `end = "2021-08-16T23:59:59Z"` to activate for a single day.
        
4. **Percentage CPU Metric Rules**:
    
    * Defines conditions for scaling based on CPU usage:
        
        * **Scale-Out Rule**: Adds 1 VM if the average CPU utilization exceeds 75% over 5 minutes.
            
        * **Scale-In Rule**: Removes 1 VM if the average CPU utilization is below 25% over 5 minutes.
            
5. **Available Memory Bytes Metric Rules**:
    
    * Defines conditions for scaling based on available memory:
        
        * **Scale-Out Rule**: Adds 1 VM if available memory falls below 1GB.
            
        * **Scale-In Rule**: Removes 1 VM if available memory is above 2GB.
            
6. **Load Balancer SYN Count Metric Rules**:
    
    * Defines conditions for scaling based on the SYN count of requests to the Load Balancer:
        
        * **Scale-Out Rule**: Adds 1 VM if the SYN count exceeds 10.
            
        * **Scale-In Rule**: Removes 1 VM if the SYN count is below 10.
            

Each of these metric-based rules is applied within the specified fixed date, ensuring that resources are scaled precisely as needed for the chosen timeframe.

### Verifying Profile 4 on Azure Portal

The fixed-date profile, **Profile-4**, has now been updated and set to the current date. As it is configured for a specific day, it has taken precedence and is currently active, overriding the previous profiles.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1730022102381/93fb5088-4bc4-463a-bfa5-c78b684f8cdd.png align="center")