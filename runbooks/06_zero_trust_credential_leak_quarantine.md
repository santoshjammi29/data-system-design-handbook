# 🚨 Production Runbook: Lakehouse Credential Compromise & Fast Quarantine

> **Severity**: `SEV-1`  
> **Domain**: Data Systems Design & Enterprise Architecture  
> **Key Focus**: Revoke compromised cloud storage keys, invalidate active tokens, and rotate credentials.

---

## 1. Emergency Decision Tree

```mermaid
flowchart TD
    Alert["🚨 Lakehouse Credential Compromise & Fast Quarantine (SEV-1)"] --> TriageCategory{"Identify Root Cause Category"}
    
    TriageCategory -->|"Configuration or Schema Drift"| ConfigTriage["Inspect Pipeline Schema & Type Contracts"]
    TriageCategory -->|"Resource Starvation or OOM"| ResourceTriage["Inspect Memory Pools, Off-Heap State & GC"]
    TriageCategory -->|"Network or Storage Timeout"| NetworkTriage["Inspect Downstream Latency & Deadlocks"]

    ConfigTriage --> FixConfig["Remediation: Pin Schema Contracts & Apply Reversion"]
    ResourceTriage --> FixResource["Remediation: Increase Worker Memory & Throttle Batch Slices"]
    NetworkTriage --> FixNetwork["Remediation: Trigger Failover & Backoff Retry Strategy"]
```

---

## 2. Immediate Diagnostic Procedures (First 5 Minutes)

### Step 1: Query Error Signatures and Pipeline Metrics
Run the following inspection commands to isolate the fault:
```bash
# Check pipeline task status and recent error traces
echo "Inspecting error status for pipeline..."
grep -E "ERROR|CRITICAL|Exception|Timeout" /var/log/pipeline.log | tail -n 25
```

### Step 2: Inspect Worker and State Health
Check memory, disk saturation, and open file descriptors:
```bash
# Verify worker container resource utilization
ps aux --sort=-%mem | head -n 10
```

---

## 3. Root Cause Analysis & Detailed Remediation

### Primary Root Cause
In production systems, this incident typically originates from sudden traffic surges, upstream breaking schema mutations, or resource limits being breached without backpressure.

### Remediation & Recovery Strategy:
1. **Apply Emergency Throttle**: Restrict incoming batch size or pause non-critical consumers.
2. **Apply Hotfix Configuration**:
   ```python
   # Emergency throttle configuration
   throttle_rate_per_sec = 5000
   retry_backoff_multiplier = 2.0
   max_retry_limit = 5
   ```
3. **Execute Recovery Step**: Re-run the validation test suite and verify consumer convergence.

---

## 4. Verification & Prevention Checklist

- [ ] Pipeline metrics return to baseline with 0 active error alerts.
- [ ] Processing latency conforms to standard production SLAs.
- [ ] Postmortem root cause documented and shared with architecture leads.


---

## 5. Mosaic AI Gateway Token Hijack & Prompt Exfiltration Triage

### 5.1 Incident Symptoms & Root Cause
- **Symptoms**: AI Gateway billing spikes dramatically ($10\times$ in 1 hour), or security audit logs alert on prompt injection attempts attempting to dump internal database connection strings.
- **Root Cause**: Compromised API tokens or malicious external user prompts bypassing naive application-level prompt sanitization.

### 5.2 Emergency Quarantine & AI Gateway Hardening
1. **Immediate Revocation of Gateway Route Credentials**:
```bash
# Rotate downstream foundation model keys immediately
databricks secrets put-secret --json '{
  "scope": "genai_secrets",
  "key": "external_llm_key",
  "string_value": "NEW_ROTATED_CREDENTIAL_VALUE"
}'
```

2. **Enable In-Line PII Redaction and Toxicity Blocking**:
```python
# Hardening AI Gateway guardrails via Python SDK
import mlflow.deployments

client = mlflow.deployments.get_deploy_client("databricks")
client.update_endpoint(
    endpoint="enterprise-genai-gateway",
    config={
        "ai_gateway": {
            "guardrails": {
                "input": {"pii": {"behavior": "mask"}},
                "output": {"toxicity": {"behavior": "block"}}
            }
        },
        "rate_limits": [{"calls": 20, "key": "user", "renewal_period": "minute"}]
    }
)
```
