
## Purpose
Define when Alice should stop local execution and escalate.

---

## Trigger Categories

### Failure-Based
- Multiple failed execution attempts  
- Repeated incorrect outputs  
- Tool execution errors  

### Confidence-Based
- Low confidence in reasoning  
- Ambiguous or underspecified task  
- High uncertainty in output quality  

### Conflict-Based
- Disagreement between agents  
- Conflicting tool outputs  
- Inconsistent analysis results  

### Complexity-Based
- Task exceeds local model capability  
- Requires long-context reasoning  
- Cross-domain synthesis required  

### Risk-Based
- High-impact decision  
- Potential destructive action  
- Sensitive system interaction  

---

## Rules

- Escalation must be explicit and logged  
- Triggers should be pattern-detectable  
- User can override escalation behavior  
- Thresholds can adapt per domain  

#connect