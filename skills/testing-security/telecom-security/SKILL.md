---
name: telecom-security
description: Assess telecommunications infrastructure security including VoIP/SIP, SS7/Diameter, cellular networks, SMS-based authentication, and telephony-integrated applications. Identifies vulnerabilities in phone-based verification, call routing, and telecom protocol implementations. Use when auditing SMS 2FA, VoIP systems, IVR applications, or any telephony-dependent security controls.
---

# Telecom Security Audit Skill

## When to Use

Activate this skill when the engagement involves:

- **SMS-based 2FA/MFA** — assessing OTP delivery, SIM swap risk, interception vectors
- **SIP/VoIP infrastructure** — auditing PBX, softphones, SIP trunks, WebRTC gateways
- **IVR systems** — interactive voice response security, DTMF handling, menu traversal
- **Call center authentication** — knowledge-based auth, callback verification, social engineering resistance
- **Telephony APIs** — Twilio, Vonage, Bandwidth, Plivo, Telnyx integration security
- **Cellular security** — baseband, IMSI catchers, roaming, carrier interconnects
- **Carrier-grade infrastructure** — SS7/SIGTRAN, Diameter, GTP, SIM provisioning

## Historical Context

The name **2600** comes from a 2600 Hz tone — the exact frequency that, when played into an AT&T
long-distance trunk line, seized control of the switch. This was in-band signaling: control data
traveled on the same channel as voice, so anyone who could produce the tone could command the network.

- **Blue box**: generated MF (multi-frequency) tones to route calls after seizing a trunk with 2600 Hz
- **Red box**: simulated coin deposit tones on payphones
- **Black box**: manipulated line voltage to prevent billing on incoming calls

The fundamental lesson: **when control signaling shares a channel with user data, users become
operators.** AT&T eventually moved to out-of-band signaling (SS7), but SS7 introduced its own
trust model vulnerability — carriers implicitly trust each other, and that trust is now exploitable
by anyone with an SS7 interconnect.

## Attack Surface Taxonomy

### SS7/MAP (Legacy Signaling)
- **Location tracking**: `SendRoutingInfo` / `ProvideSubscriberInfo` queries reveal cell tower location
- **SMS interception**: `UpdateLocation` re-registers victim to attacker-controlled MSC
- **Call interception**: `InsertSubscriberData` redirects call forwarding
- **Authentication bypass**: obtain IMSI, trigger re-authentication to capture triplets

### Diameter (4G/5G Signaling)
- Roaming scenarios inherit SS7's implicit trust between carriers
- S6a/S6d interfaces expose subscriber data during inter-PLMN handoffs
- Diameter Edge Agents (DEA) provide filtering, but coverage is inconsistent

### SIP/VoIP
- **Registration hijacking**: unauthenticated REGISTER, digest auth brute force
- **Eavesdropping**: RTP without SRTP, SIP without TLS
- **Toll fraud**: compromised endpoints used for premium-rate number dialing
- **Caller ID spoofing**: `From` header trivially set to any value
- **SRTP key negotiation**: SRTP-SDES sends keys in plaintext SDP (use DTLS-SRTP instead)

### SMS/OTP
- **SS7 interception**: redirect SMS at the network level (proven in real attacks)
- **SIM swap**: social engineering carrier support to port number to attacker SIM
- **Smishing**: SMS phishing with shortened/obfuscated URLs
- **OTP interception**: malware with SMS read permission on Android

### IVR/DTMF
- **Brute force**: PIN/account number guessing via rapid DTMF input
- **Menu traversal**: undocumented options, debug menus, operator escape sequences
- **Information disclosure**: reading back account details, SSN, balances without strong auth
- **Recording exposure**: call recordings stored without encryption or access control

### Caller ID / ANI
- **Caller ID (CNAM)**: trivially spoofable — no cryptographic verification in legacy PSTN
- **STIR/SHAKEN**: adds certificate-based attestation but adoption is still incomplete
- **ANI vs Caller ID**: ANI is harder to spoof but not immune in VoIP-to-PSTN transitions

### SIM Security
- **SIM swap**: social engineering carrier reps, insider threats, automated porting attacks
- **SIM cloning**: legacy Comp128v1 vulnerability (historical, mostly patched)
- **eSIM provisioning**: QR code interception, SM-DP+ server compromise
- **SIM toolkit (STK)**: Simjacker-style attacks via crafted OTA SMS

### Telephony APIs
- **Webhook spoofing**: attacker sends fake status callbacks to application endpoints
- **Credential leakage**: API keys/auth tokens in client-side code, logs, or VCS history
- **Rate limiting gaps**: no throttle on verification SMS sends (cost amplification, spam)
- **Number enumeration**: carrier lookup APIs reveal active numbers and carrier info

## Audit Methodology

1. **Enumerate telephony-dependent auth paths**
   - Map every flow that uses phone numbers: login, password reset, transaction verification
   - Identify which are SMS-only vs. offering TOTP/WebAuthn alternatives

2. **Assess SMS 2FA for known bypass vectors**
   - Can an attacker SIM-swap the target? (carrier policy, account PIN, port-out protection)
   - Is the application vulnerable if SMS is intercepted? (session fixation, race conditions)
   - Are OTPs time-limited and single-use? Are they sufficiently long (6+ digits)?

3. **Test caller ID verification**
   - Can a spoofed caller ID bypass IVR authentication?
   - Does the system use callback verification for sensitive operations?
   - Is STIR/SHAKEN attestation level checked for incoming SIP calls?

4. **Audit VoIP configuration**
   - SIP: TLS enabled? Certificate validation? Digest auth or mutual TLS?
   - Media: SRTP enforced? DTLS-SRTP or SDES? Key management reviewed?
   - Registration: rate limiting on REGISTER? Fail2ban or equivalent?

5. **Review IVR security**
   - Lockout after N failed PIN attempts?
   - Sensitive data readback requires step-up authentication?
   - Debug/admin menus accessible from external callers?

6. **Test telephony API integration**
   - Webhook endpoints validate request signatures (e.g., Twilio `X-Twilio-Signature`)?
   - Replay protection via timestamp validation?
   - API credentials rotated, scoped to minimum permissions?

7. **Assess carrier/provider resilience**
   - Single carrier dependency? Failover path?
   - Geographic number portability risks?
   - Provider SLA and incident response capability?

## Code Review Patterns

Look for these anti-patterns during source review:

```python
# FINDING: SMS as sole second factor — no TOTP/WebAuthn alternative
if user.mfa_method == "sms":
    send_otp(user.phone_number)  # No fallback offered

# FINDING: Phone number as primary account identifier
user = User.objects.get(phone=request.data["phone"])  # SIM swap = account takeover

# FINDING: Caller ID trusted for authentication
if call.caller_id == expected_number:
    grant_access()  # Trivially spoofable

# FINDING: SIP credentials in plaintext config
SIP_PASSWORD = "oops-plaintext"  # Should use secrets manager

# FINDING: No SRTP — voice traffic sent unencrypted
media_encryption = "none"  # Should be "srtp" with DTLS key exchange

# FINDING: Telephony webhook without signature verification
@app.route("/twilio/status", methods=["POST"])
def status_callback():
    process(request.form)  # No X-Twilio-Signature check

# FINDING: No rate limiting on phone verification endpoint
@app.route("/verify/send", methods=["POST"])
def send_verification():
    send_sms(request.json["phone"])  # Unlimited sends = cost amplification

# FINDING: Phone number enumeration via error differences
if not user_exists(phone):
    return {"error": "User not found"}  # vs. "Invalid credentials"
```

## Remediation Quick Reference

| Issue | Remediation |
|-------|-------------|
| SMS as sole 2FA | Offer TOTP (RFC 6238) or WebAuthn as alternatives |
| SIM swap risk | Enable carrier port-out PIN, number lock, SIM swap detection alerts |
| Caller ID trusted | Never authenticate on caller ID alone; use callback verification |
| SIP without TLS | Enforce TLS 1.2+ for signaling, validate certificates |
| RTP without SRTP | Enforce SRTP with DTLS-SRTP key exchange (not SDES) |
| IVR PIN brute force | Lockout after 3-5 failures, exponential backoff, CAPTCHA for web-initiated calls |
| Webhook spoofing | Validate HMAC signatures, check timestamps, reject replays >5 min |
| Number enumeration | Uniform error responses, rate limit lookup endpoints |
| API key exposure | Use environment variables or secrets manager, rotate regularly, scope permissions |
| No OTP expiry | OTPs expire in 5-10 minutes, single-use, minimum 6 digits |

## Tools Reference

- **SIPVicious** (`sipvicious`): SIP scanning, enumeration, password cracking
- **Ohrwurm**: RTP fuzzer for VoIP
- **Mr.SIP**: SIP-based audit and attack tool
- **SigPloit**: SS7/Diameter/GTP pentesting framework
- **Owasp VoIPAudit**: VoIP security testing methodology
- **Owasp Owasp Telephony cheatsheet**: telephony security reference
- **Owasp Testing Guide**: relevant sections on OTP, 2FA bypass

## Related Skills

- `social-engineering-audit` — SIM swap and call center pretexting vectors
- `entry-point-analyzer` — mapping telephony-dependent authentication surfaces
- `static-security-analyzer` — scanning for hardcoded SIP credentials and API keys
