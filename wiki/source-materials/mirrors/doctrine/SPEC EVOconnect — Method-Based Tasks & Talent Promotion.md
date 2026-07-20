#connect
## Core Idea

> A Method must prove reliability through user-confirmed success before becoming a Talent.

Alice does not automatically convert Methods into Talents.

She must earn it through:
- repeated execution
- explicit user validation

---

## New Task Status

### Complete — Waiting on Final Review

This status is unique to:

> **Alice Method-based tasks**

---

### Meaning

- Task execution finished  
- Outcome delivered  
- Awaiting user validation  

---

## User Validation Mechanism

After task completion, the user is shown:

- 👍 Thumbs Up (successful)  
- 👎 Thumbs Down (unsuccessful)  

---

## Success Rules

### Promotion Requirement

A Method becomes a Talent only if:

> **User gives 3 consecutive 👍 approvals**

---

### Failure Rule

If the user gives:

> **👎 even once**

Then:

- success counter resets to 0  
- process must restart  

---

### Important

- “Done” ≠ “Successful”  
- Only 👍 counts toward promotion  

---

## Behavior Before Promotion

Before reaching 3 consecutive 👍:

- Method remains:
  - reusable  
  - executable  
- BUT:
  - not a Talent  
  - not shareable  
  - not promoted  

---

### Execution Behavior

Alice may:

- continue using the Method  
- repeat the task indefinitely  

But it remains:

> **Unverified Method, not a Talent**

---

## User Flexibility

### Editable Feedback

Before Talent creation:

- User can change previous 👍/👎 decisions  

---

### After Talent Creation

Once promoted:

- Feedback is locked  
- To remove:

> User must click **“Forget Talent”**

---

## Forget Talent

### Behavior

When user selects:

> “Forget Talent”

System:

- removes Talent  
- resets validation state  
- optionally preserves Method for retraining  

---

## State Model

### Method Lifecycle

1. Method Created  
2. Method Executed  
3. Complete — Waiting on Final Review  
4. User Feedback (👍 / 👎)  
5. Success Counter Updated  

---

### Promotion Path

- 👍 → count +1  
- 👍 → count +1  
- 👍 → count +1 → **Promote to Talent**

---

### Failure Path

- 👍 → count +1  
- 👎 → reset to 0  

---

## Why This Matters

### 1. Trust

Talents are:
- proven  
- user-verified  
- reliable  

---

### 2. Safety

Prevents:
- premature automation  
- incorrect behavior scaling  
- bad Talents spreading  

---

### 3. Learning Loop

Alice can:
- refine Methods  
- retry tasks  
- improve before promotion  

---

### 4. User Control

User is the final authority:

- defines success  
- approves promotion  
- can revoke at any time  

---

## UX Requirements

- Feedback must be:
  - simple  
  - immediate  
  - unobtrusive  

- Status must be visible:
  - “Waiting for your feedback”  

- Progress indicator:
  - e.g. “2/3 successful runs”  

---

## Example Flow

1. Alice executes Method  
2. Task completes  
3. Status:
   → “Complete — Waiting on Final Review”  

4. User taps 👍  
   → Progress: 1/3  

5. Repeat process  

6. On 3rd 👍:

> “I can do this for you automatically now. Want me to save this as a Talent?”

---

## Core Takeaway

> Methods must be proven.  
> Talents must be earned.

Alice does not automate by assumption.

She automates by:
- repetition  
- validation  
- trust