
## **Core Idea**

  

> Alice can be present everywhere, but she only learns when explicitly invited.

  

This defines the separation between:

- **Awareness (always on)**
    
- **Learning (explicit only)**
    

---

## **Awareness vs Learning**

  

### **Awareness (Default State)**

  

Alice is present across all surfaces:

- Chat
    
- Task Manager
    
- EVObrowser
    
- EVOterminal
    
- Other Hive nodes
    

  

She can:

- understand context
    
- answer questions
    
- assist in real-time
    
- explain what is happening
    

  

She cannot:

- record workflows
    
- infer repeatable patterns
    
- create Methods
    
- create Talents
    

  

> Awareness is passive understanding, not data capture.

---

### **Learning (Training Mode Only)**

  

Learning is only enabled when the user explicitly enters:

  

> **Talent Training Mode**

  

In this state, Alice can:

- observe actions
    
- interpret intent
    
- construct a Method
    
- prepare a reusable workflow
    

  

After execution:

1. Alice generates a Method
    
2. Method is presented to the user
    
3. Permissions are clearly defined
    
4. User approves or rejects
    
5. Talent is created if approved
    

  

> No Training Mode → No learning.

---

## **Execution Surfaces**

  

### **EVObrowser**

- Alice can see user-driven workflows
    
- Can assist and explain
    
- Cannot learn unless Training Mode is active
    

---

### **EVOterminal**

- Alice can execute commands through a governed environment
    
- Can assist with system-level tasks
    
- Can be used for Talent Training
    

  

All actions must:

- bind to a Method
    
- be governed by Delegator
    
- be logged
    

---

## **Privileged Execution (sudo / elevated actions)**

  

### **Problem**

  

Some real-world tasks require elevated permissions:

- system cleanup
    
- package installation
    
- environment configuration
    
- device-level operations
    

  

These require:

  

> **sudo-level access**

---

## **Vault-Based Credential Access**

  

Sensitive credentials (like system passwords) are stored in:

  

> **User-controlled secure vault**

  

Alice:

- does not have direct access to raw credentials
    
- cannot retrieve or expose them
    
- must request permission to use them
    

---

## **Permission Model for Privileged Actions**

  

### **Default Behavior (Strict)**

  

For any action requiring sudo:

1. Alice detects elevated permission requirement
    
2. Alice pauses execution
    
3. Alice presents:
    
    - what command will run
        
    - why elevated access is required
        
    - what system areas are affected
        
    
4. User must explicitly approve:
    
    - the action
        
    - the use of vault credentials
        
    

---

### **Approval Options**

  

User can choose:

  

#### **1. One-Time Approval**

- Approve this execution only
    
- Credentials used once
    
- No persistence
    

  

#### **2. Session Approval**

- Allow for current session
    
- Expires automatically
    

  

#### **3. Always Allow (Explicit Opt-In)**

- Whitelisted for this specific Method/Talent
    
- Still logged
    
- Still governed
    
- Can be revoked at any time
    

  

> Default is ALWAYS: One-Time Approval

---

## **Delegator Enforcement**

  

Delegator ensures:

- no silent privilege escalation
    
- no background sudo execution
    
- no credential exposure
    
- no execution outside approved scope
    

  

All privileged actions must:

- bind to a Method
    
- be visible before execution
    
- be logged
    
- be interruptible
    

---

## **Example — System Cleanup Talent**

  

### **Scenario**

  

User teaches Alice:

  

> Clean Xcode DerivedData and Simulator files

---

### **Method (Simplified)**

1. Analyze storage usage
    
2. Identify safe-to-remove developer artifacts
    
3. Execute removal commands via EVOterminal
    
4. Report recovered space
    

---

### **Privileged Step**

  

Some commands may require:

```
sudo rm -rf [protected directories]
```

---

### **Execution Flow**

1. Alice proposes Method
    
2. User approves Method + permissions
    
3. During execution:
    
    Alice says:
    
    > “This step requires elevated access to remove system-protected files.”
    
4. User selects:
    
    - Approve once
        
    - Allow for session
        
    - Always allow for this Talent
        
    
5. Delegator authorizes vault usage
    
6. EVOterminal executes command
    
7. Action is logged
    

---

## **Recurring Task Integration**

  

Once a Talent is created, user can:

  

> “Run this once a week”

  

Execution behavior:

- If privileged access is required:
    
    - respects stored approval level
        
    - re-prompts if necessary
        
    
- If no persistent approval:
    
    - pauses and requests confirmation
        
    

---

## **Security Guarantees**

- No silent credential usage
    
- No raw password exposure
    
- No background privileged execution
    
- No cross-task permission leakage
    
- Full audit trail for all elevated actions
    

---

## **Design Principles**

  

### **1. Presence without surveillance**

  

Alice can see context, but does not learn without consent.

---

### **2. Power with friction (intentional)**

  

Privileged actions require explicit approval.

---

### **3. Transparency first**

  

Every action is:

- explained
    
- reviewable
    
- interruptible
    

---

### **4. User ownership**

  

User controls:

- what is learned
    
- what is executed
    
- what is allowed
    

---

## **Core Takeaway**

  

> Alice is always present, but never invasive.

> She only learns when invited, and only acts with permission—

> especially when real system power is involved.

#connect