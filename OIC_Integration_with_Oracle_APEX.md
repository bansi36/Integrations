# OIC Integration with Oracle APEX

This guide explains how to integrate **Oracle Integration Cloud (OIC)** with **Oracle APEX** using REST APIs secured with OAuth 2.0.

## 1) Typical Integration Patterns

1. **APEX calls OIC**
   - APEX page process or PL/SQL package invokes an OIC REST endpoint.
   - OIC orchestrates downstream services (ERP, HCM, SaaS, DB, custom APIs).

2. **OIC calls APEX REST APIs**
   - OIC pushes data into APEX-enabled REST modules (ORDS).
   - Useful for near-real-time sync.

3. **Bidirectional event-based flow**
   - APEX creates/updates business events.
   - OIC handles transformation/routing and callback updates.

## 2) High-Level Architecture

- **APEX app** (UI + PL/SQL business logic)
- **Oracle REST Data Services (ORDS)** exposing APEX REST endpoints (if needed)
- **OIC integration** (trigger + mapping + adapters + fault handling)
- **Security provider** (OCI IAM / OAuth identity provider)
- **Monitoring** via OIC Instance Tracking and APEX activity logs

## 3) Prerequisites

- Oracle APEX workspace and app access
- OIC instance with Integration role
- ORDS configured (if OIC needs to invoke APEX REST)
- OAuth 2.0 client credentials configured in OIC and/or APEX endpoint security
- Network rules and allowlists configured between environments

## 4) Create an OIC REST Integration

1. In OIC, create an **App-Driven Orchestration** integration.
2. Add a **REST trigger** with request/response JSON schema.
3. Add business flow logic:
   - data mapping
   - optional enrichment from adapters (ERP/SOAP/DB/FTP)
   - error handling with fault paths
4. Configure connection security (OAuth 2.0 recommended).
5. Activate integration and copy endpoint URL.

## 5) Invoke OIC from APEX (PL/SQL)

Use `APEX_WEB_SERVICE` to call the OIC endpoint from an APEX process/package.

```sql
DECLARE
    l_url           VARCHAR2(4000) := 'https://<oic-host>/ic/api/integration/v1/flows/rest/<flow>/1.0/submit';
    l_response      CLOB;
    l_request_body  CLOB := '{"customerId": 1001, "status": "NEW"}';
BEGIN
    -- Optional: set OAuth bearer token if not using static credentials in Web Credentials
    apex_web_service.g_request_headers(1).name  := 'Content-Type';
    apex_web_service.g_request_headers(1).value := 'application/json';

    l_response := apex_web_service.make_rest_request(
        p_url         => l_url,
        p_http_method => 'POST',
        p_body        => l_request_body
    );

    -- Parse/use response JSON as needed
    dbms_output.put_line(substr(l_response,1,4000));
END;
/
```

## 6) Secure Authentication

### Recommended

- Use **OAuth 2.0 Client Credentials**.
- Store secrets in:
  - APEX **Web Credentials**
  - OCI Vault / secure secret store
- Avoid hardcoding tokens or passwords in PL/SQL.

### Additional Controls

- Enforce TLS 1.2+
- Validate scopes/audience in tokens
- Apply IP restrictions and rate limits

## 7) Error Handling & Observability

- In OIC:
  - use **Scope + Fault Handler** for controlled error responses
  - enable **business identifiers** for traceability
  - monitor through **Tracking** and **Activity Stream**
- In APEX:
  - capture integration response/status code
  - log correlation ID returned by OIC
  - show user-friendly errors while retaining detailed server logs

## 8) Performance Best Practices

- Keep payloads minimal and version your API contracts
- Use asynchronous integrations for long-running flows
- Apply retries with backoff for transient failures
- Add idempotency keys for safe reprocessing

## 9) Example Use Cases

- APEX order submission to ERP via OIC
- APEX form approvals routed through OIC workflows
- OIC master data sync back into APEX reporting schema

## 10) Go-Live Checklist

- [ ] Endpoint authentication tested in lower environments
- [ ] OIC fault policies defined and validated
- [ ] APEX timeout/retry behavior tuned
- [ ] Monitoring dashboards and alerts configured
- [ ] Rollback strategy documented

---

If you want, I can also provide:
1. a concrete **APEX package** for token management + API invocation,
2. an **OIC sample integration design** (trigger/request/response schemas), and
3. a **troubleshooting playbook** for common HTTP 401/403/500 issues.
