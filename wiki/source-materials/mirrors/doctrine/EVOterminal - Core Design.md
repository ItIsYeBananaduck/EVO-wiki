#connect
## Purpose
Provide a governed, sandboxed execution surface for Computer Alice to interact with local tools and environments.

---

## What EVOterminal Is

- Internal terminal interface  
- Sandboxed execution environment  
- Governed by Delegator  
- Fully logged  
- Scoped to approved actions  

---

## What EVOterminal Is NOT

- Unrestricted system terminal  
- Root-level access tool  
- Arbitrary script executor  
- Hidden automation layer  

---

## Capabilities

- Run approved commands  
- Inspect local repositories  
- Execute bounded scripts  
- Launch controlled workflows  
- Interface with local tools  

---

## Governance Rules

All actions must:
- Bind to a Method  
- Be approved when required  
- Be logged  
- Respect scope boundaries  

---

## Execution Model

1. Task enters execution phase  
2. Delegator validates allowed actions  
3. EVOterminal executes within sandbox  
4. Output is captured  
5. Result is logged and returned  

---

## Logging

Each execution records:
- command or action  
- timestamp  
- task id  
- result  
- success/failure  

---

## Safety

- No arbitrary filesystem access  
- No silent background execution  
- No external data access without approval  