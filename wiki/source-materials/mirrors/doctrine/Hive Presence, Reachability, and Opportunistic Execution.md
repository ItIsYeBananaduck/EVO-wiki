## **Core problem**


Right now Hive appears to depend too much on:

- devices being powered on
    
- EVO being open
    
- nodes staying effectively always-on
    
- manual certainty that another device is available
    


That creates a brittle model where the system only works when every node is already visibly alive.

  

This conflicts with the intended Hive role as a coordination layer responsible for node discovery, capability detection, task distribution, and safe leader reassignment. Hive is supposed to coordinate nodes that can enter and exit safely, not require a permanently visible cluster to exist first. 

  

## **Why this matters**

  

This is not just a Connect issue. It affects both:

- **EVOtraining**
    
- **EVOconnect**
    

  

But it matters more for Connect because Connect is the orchestration surface. Connect is meant to be the bounded automation environment and hive orchestrator for Alice across devices, tools, and compute resources.

  

If Hive cannot build a reliable snapshot of reachable devices, then several core promises weaken:

- cross-device task execution
    
- opportunistic compute borrowing
    
- delegated work routing
    
- swarm acceleration
    
- desktop-to-mobile continuity
    

  

## **Core insight**

  

You do **not** actually need every device to be always on.

  

You need three separate concepts that are currently getting conflated:

  

### **1. Registered node**

  

A known device that belongs to the user and has joined the ecosystem before.

  

### **2. Reachable node**

  

A device that can respond right now or within a short response window.

  

### **3. Runnable node**

  

A reachable device that currently has the permissions, battery state, thermal state, app/runtime state, and capability set required for a specific task.

  

Those are different.

  

A device can be:

- registered but unreachable
    
- reachable but not runnable
    
- runnable for one task but not another
    

  

That distinction is the key to making Hive feel real instead of fragile.

---

# **Proposed model**

  

## **Node state model**

  

Every node should expose a small presence model.

  

### **Identity state**

- node id
    
- device type
    
- device name
    
- platform
    
- trust/ownership status
    

  

### **Reachability state**

- last seen timestamp
    
- currently reachable yes/no
    
- last successful handshake timestamp
    
- expected wake behavior
    
- background contact channel availability
    

  

### **Capability state**

- model/runtime availability
    
- compute class
    
- battery state
    
- charging state
    
- thermal state
    
- network quality
    
- permissions granted
    
- domain eligibility
    

  

### **Execution state**

- idle
    
- busy
    
- suspended
    
- unavailable
    
- low-power restricted
    
- requires foreground
    
- requires approval