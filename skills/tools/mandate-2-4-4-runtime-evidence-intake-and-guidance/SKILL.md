---
name: mandate-2-4-4-runtime-evidence-intake-and-guidance
description: Collect natural-language system context and generate supporting evidence guidance for Mandate 2.4.4 Secure Inter-Agent Communication, including mTLS, certificate lifecycle, and network policy proof.
---

# Mandate 2.4.4 Runtime Evidence Intake and Guidance Skill

## Mandate
- ID: 2.4.4
- Title: Secure Inter-Agent Communication

## Mitigates
- ASI07 Insecure Inter-Agent Communication

## Objective
Gather communication topology context and produce a complete evidence plan proving encryption, mutual authentication, and channel integrity across agent interactions.

## Natural-Language Intake Workflow
1. Ask which agents/services communicate and over which protocols.
2. Ask whether service mesh or custom TLS is used.
3. Ask how certificates are issued, rotated, and revoked.
4. Ask how trust anchors/CA chains are managed.
5. Ask how unauthorized endpoints are blocked.
6. Ask how replay/spoofing/eavesdropping risks are tested.

## Context Normalization Schema
Capture and normalize answers into:
- `comm_topology`: agent pairs, protocols, ports, transport paths.
- `crypto_posture`: TLS versions, cipher policies, mTLS requirements.
- `identity_trust_chain`: CA, cert issuance workflow, SAN validation.
- `cert_lifecycle`: rotation cadence, revocation process, expiration handling.
- `network_enforcement`: firewall/network policy/service mesh authz rules.
- `integrity_controls`: signatures/MAC/nonces/replay defense controls.
- `security_validation`: penetration tests and abuse simulations.

## Supporting Documents to Request
1. Service mesh or communication architecture diagrams.
2. TLS/mTLS configuration and certificate authority details.
3. Certificate lifecycle logs (issuance, rotation, revocation).
4. Inter-service firewall/network policy exports.
5. Security test reports for spoofing/eavesdropping/replay.

## Guidance to Generate Supporting Evidence
1. Generate communication topology evidence.
   - Export service maps, endpoint inventories, and traffic policy maps.
2. Generate TLS/mTLS posture evidence.
   - Export active TLS and peer-authentication policies.
3. Generate certificate lifecycle evidence.
   - Export CA inventory and cert issuance/rotation/revocation logs.
4. Generate network policy evidence.
   - Export inter-service allow/deny rules proving least-connectivity.
5. Generate security validation evidence.
   - Export test reports showing resistance to spoofing/replay/eavesdropping.

## System Command Templates (Adapt to Environment)
- Kubernetes service and policy map:
  - `kubectl get svc -A -o wide > services_export.txt`
  - `kubectl get networkpolicy -A -o yaml > network_policies.yaml`
- Istio/mesh security policies (if applicable):
  - `kubectl get peerauthentication -A -o yaml > peerauthentication.yaml`
  - `kubectl get destinationrule -A -o yaml > destinationrules.yaml`
  - `istioctl authn tls-check <pod>.<namespace>`
- TLS certificate inspection:
  - `openssl s_client -connect <host>:<port> -showcerts </dev/null > tls_chain_<host>_<port>.txt`
- AWS certificate inventory (if applicable):
  - `aws acm list-certificates > acm_certificates.json`
  - `aws acm describe-certificate --certificate-arn <arn> > acm_certificate_detail.json`
- GCP certificate inventory (if applicable):
  - `gcloud certificate-manager certificates list --format=json > gcp_certificates.json`

## Evidence Completeness Rules
- Require at least one artifact per control domain: encryption, mutual auth, cert lifecycle, network restriction, testing.
- Reject artifacts that do not map to specific agent communication paths.
- Mark artifacts stale if they do not reflect current deployment.

## Final Assessment Readiness
- Ready only when all agent communication paths have matching cryptographic, identity, and network control evidence.
- Not ready if any path is undocumented or lacks proof of control enforcement.

## Output Contract
Return:
- `context_profile`
- `communication_control_matrix`
- `required_artifacts_checklist`
- `artifact_generation_steps`
- `command_templates`
- `evidence_status_matrix`
- `assessment_readiness`
- `remaining_gaps`

## Guardrails
- Keep evidence collection non-invasive and read-only by default.
- Request explicit environment scoping before generating commands.
- Flag unmanaged communication channels immediately as high risk.

