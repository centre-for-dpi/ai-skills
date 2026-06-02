---
name: dpi-ai-use-cases
description: AI use cases across DPI sectors — health, welfare, agriculture, financial inclusion, governance. Use when exploring how AI applies to specific DPI services or government programmes.
---

# DPI AI Use Cases

> **Reference**: https://digitalpublicinfrastructure.ai/

## The DPI-AI Value Framework

AI amplifies each DPI layer:

| DPI Layer | Without AI | With AI |
|-----------|-----------|---------|
| **Identity** | Manual enrollment, form-filling, in-person verification | AI-guided enrollment, document OCR, multilingual support, exception handling |
| **Payments** | Rule-based fraud detection, batch processing, manual reconciliation | Real-time ML fraud scoring, predictive cash flow, automated anomaly detection |
| **Data Exchange** | Citizen must know what data exists and request it manually | AI proactively identifies relevant data, explains consent, aggregates intelligently |
| **Credentials** | Credential issuance triggered by manual process | AI-triggered issuance based on life events, automated renewal reminders |
| **Registries** | Manual data quality checks, batch deduplication | Real-time quality scoring, AI-assisted deduplication, exception flagging |

## Use Case 1: Inclusive Enrollment (Identity)

### Problem
Citizens in remote areas face barriers to digital identity enrollment: distance, language, literacy, disability, document availability.

### DPI-AI Solution

```
AI-Guided Mobile Enrollment Agent
    ↓
Step 1: Citizen speaks in local language
[ASR Block] → text in local language
[Translation Block] → standardized text

Step 2: Capture documents
[Computer Vision Block] → extract structured data from any document type
[Quality Check Block] → "Please move closer, image is blurry"

Step 3: Handle exceptions
[LLM Reasoning] → "Citizen has no birth certificate; recommend alternative proofs per policy"

Step 4: Submit to DPI identity API
[Identity Enrollment API] → enrollment record created

Step 5: Confirm in local language + dialect
[TTS Block] → spoken confirmation
```

**AI blocks used**: ASR, translation, OCR, computer vision, liveness detection, LLM reasoning, TTS

**Impact**: Reach populations that formal enrollment centers exclude; reduce enrollment abandonment; reduce cost per enrollment.

## Use Case 2: Real-Time Payment Fraud Prevention

### Problem
Open payment rails attract fraud: account takeover, merchant fraud, money mules, SIM swap.

### DPI-AI Solution

```
Payment initiation event
    ↓
[Feature Engineering] → velocity, device, geography, behavior pattern
    ↓
[ML Fraud Scoring] → risk score (0-100) in < 50ms
    ↓
[Decision Engine]:
  score < 20  → auto-approve
  20-70       → step-up auth (biometric challenge)
  > 70        → block + notify
    ↓
[Graph Analysis] (async) → network of connected accounts
    ↓
[LLM Case Narrative] → human-readable explanation for analyst review
```

**Signals used**: Transaction amount vs. historical average, time of day, device fingerprint, geolocation velocity, beneficiary reputation, peer group behavior, account age

**Privacy note**: Fraud scoring must not use protected characteristics (gender, ethnicity, religion) — regular bias audits required.

## Use Case 3: Intelligent Benefit Eligibility and Routing

### Problem
Citizens don't know which government programs they qualify for; eligibility rules are complex and programs are fragmented.

### DPI-AI Solution — "Benefits Discovery Agent"

```python
async def benefits_discovery_agent(citizen_id: str) -> list[BenefitRecommendation]:
    # Step 1: Authenticate
    auth = await dpi.identity.authenticate(citizen_id)
    
    # Step 2: Build citizen profile from consented DPI data
    profile = await asyncio.gather(
        dpi.data_exchange.fetch(citizen_id, "income"),      # Financial data
        dpi.registry.get(citizen_id, "household"),          # Household size
        dpi.registry.get(citizen_id, "current_enrollments") # Already enrolled programs
    )
    
    # Step 3: AI reasons against policy catalog (RAG over all programs)
    all_programs = await vector_store.search(
        query=f"Income: {profile.income}, Household: {profile.household}",
        collection="benefit_programs"
    )
    
    recommendations = []
    for program in all_programs:
        eligibility = await llm.assess_eligibility(profile, program.eligibility_rules)
        if eligibility.likely_eligible and program not in profile.current_enrollments:
            recommendations.append(BenefitRecommendation(
                program=program,
                confidence=eligibility.confidence,
                explanation=eligibility.plain_language_explanation
            ))
    
    return sorted(recommendations, key=lambda r: r.confidence, reverse=True)
```

**Key innovation**: Citizen does not need to know what programs exist — the agent finds them based on verified DPI data.

## Use Case 4: Parametric Agricultural Insurance

### Problem
Traditional crop insurance requires field verification (expensive, slow, fraud-prone). Smallholders cannot afford it.

### DPI-AI Solution

```
Policy Issuance:
  Farmer authenticates via DPI identity rail
  Land holding verified from land registry (DPI)
  Policy terms set by AI based on: crop type, acreage, historical yields, weather risk
  Premium calculated and paid via DPI payment rail
  Policy stored as Verifiable Credential in farmer's wallet

Claim Triggering (Automated):
  [Satellite Imagery Analysis] → NDVI (crop health index) computed weekly
  [Weather Station Data] → rainfall, temperature, humidity IoT feeds
  [AI Anomaly Detection] → crop failure confirmed when index < threshold
  [Payment Trigger] → DPI payment API called automatically
  [Credential Update] → insurance VC updated with claim record
  Farmer receives SMS notification of payment
```

**No manual claim filing required.** Claim is triggered by objective, verifiable data.

## Use Case 5: Multilingual Government Service Assistant

### Problem
Citizens face language and literacy barriers when accessing digital government services; helplines are overloaded.

### DPI-AI Solution

```python
# Multilingual DPI Service Agent
async def citizen_service_agent(message: str, channel: str, session: Session) -> Response:
    
    # Detect language
    language = await ai.detect_language(message)
    
    # Convert voice to text if needed (USSD/IVR channel)
    if channel == "voice":
        text = await asr.transcribe(message, language=language)
    else:
        text = message
    
    # Understand intent with context
    intent = await llm.understand(
        text=text,
        conversation_history=session.history,
        available_services=await dpi.registry.list_services()
    )
    
    # Execute with DPI tool access
    result = await execute_intent(intent, session)
    
    # Generate response in detected language, appropriate literacy level
    response_text = await llm.generate_response(
        result=result,
        language=language,
        literacy_level=session.citizen_profile.literacy,
        channel=channel
    )
    
    # Convert back to voice if needed
    if channel == "voice":
        return await tts.synthesize(response_text, language=language)
    
    return Response(text=response_text, language=language)
```

**Impact**: Citizens in any language, on any channel (SMS, WhatsApp, IVR, web), can access the same quality of government service.

## Use Case 6: Health Care Navigation and Pre-Authorization

### Problem
Patients don't know which facility to go to; insurers require manual pre-authorization that delays care.

### DPI-AI Solution

```
Step 1: Patient describes symptoms via chat/voice
[Medical NLU] → structured clinical concepts
[Triage Agent] → urgency classification

Step 2: Find appropriate facility
[Registry Query] → health facilities with required specialties, location
[Real-time Capacity Query] → current wait times, bed availability

Step 3: Pre-authorization
[Patient Health Record] → fetched via DPI data exchange with consent (FHIR)
[AI Pre-Auth Engine] → matches symptoms/diagnosis to insurer policy
[Pre-Auth Decision] → approve/deny/query in minutes instead of days

Step 4: Appointment
[Registry Update] → appointment booked in health facility registry
[Calendar Invite + Reminder] → sent via DPI notification API
[Health Credential] → updated in patient's wallet
```

## Use Case 7: Digital KYC for Financial Inclusion

### Problem
Traditional KYC costs $5-25 per customer and requires in-person document submission, excluding billions from formal financial services.

### DPI-AI Solution

```
Customer Journey (3 minutes, fully digital):

1. Citizen opens bank app
2. Enters national ID number
3. [OTP sent via DPI identity API]
4. Citizen enters OTP
5. [DPI eKYC API returns: name, DOB, address, photo]
6. [AI face match: selfie vs. ID photo]
7. [AI liveness check: blink, turn head]
8. [AI risk scoring: fraud signal check]
9. Account opened, linked to DPI payment rail
10. [Verifiable Credential issued: "KYC Verified" with timestamp]
```

The "KYC Verified" VC can be reused at any other financial institution — citizen does not repeat the process.

## Use Case 8: Proactive Social Protection

### Problem
Vulnerable citizens who need benefits most are often the least likely to navigate bureaucratic application processes.

### DPI-AI Solution — Push Model

Traditional: Citizen finds program → applies → waits → receives benefit (if they ever apply)

AI-DPI Push Model:
```
[Life Event Detected]:
  Birth registered in CRVS → triggers maternity benefit outreach
  Job loss detected (employment registry change) → triggers unemployment benefit
  Elderly citizen reaches 65 (population registry) → triggers pension enrollment
  Disability assessment complete → triggers disability benefit

[Outreach Agent]:
  AI generates personalized SMS/WhatsApp in citizen's language
  "Congratulations on your new baby! You may be eligible for X benefit.
   Reply YES to apply or call 1234 for help."

[Application Agent]:
  Citizen replies YES → agent handles application automatically
  Fetches required data via consented DPI data exchange
  Completes enrollment if eligible
  Initiates first payment via DPI payment rail

No government office visit required.
```

## Use Case 9: AI-Assisted Grievance Resolution

### Problem
Government complaint systems are slow, opaque, and unresponsive. Citizens lose trust.

### DPI-AI Solution

```python
# AI Grievance Triage and Routing
async def handle_grievance(complaint: CitizenComplaint) -> GrievanceResult:
    
    # Classify and assess urgency
    classification = await ai.classify_grievance(complaint.text)
    
    # Fetch citizen's case history from DPI registry
    history = await dpi.registry.get_complaints(complaint.citizen_id)
    
    # If duplicate/resolved issue, inform citizen immediately
    if similar := await find_similar_resolved(complaint, history):
        return await ai.generate_resolution_info(similar)
    
    # If urgent (benefits stopped, medical, child welfare), escalate immediately
    if classification.urgency == "URGENT":
        await escalate_to_human(complaint, classification, sla_hours=4)
        return await ai.generate_acknowledgment("urgent", complaint.language)
    
    # Attempt AI resolution for common issues
    if resolution := await ai.auto_resolve(classification, complaint):
        # Execute resolution via DPI (restart payment, reactivate benefit, etc.)
        await dpi_action(resolution.action, resolution.params)
        return await ai.generate_resolution_confirmation(resolution, complaint.language)
    
    # Route to appropriate department with AI-prepared summary
    await route_to_department(classification.department, complaint, ai.summarize(complaint))
    return await ai.generate_tracking_message(complaint.id, sla=classification.expected_sla)
```

**Impact**: Measurable reduction in resolution time; consistent treatment; citizen sees status updates.

## Use Case 10: Cross-Border Credential Recognition

### Problem
Migrants and workers cannot have their credentials (degrees, professional licenses, health records) recognized across borders quickly.

### DPI-AI Solution

```
Source country credential:
  [Verifiable Credential] in W3C VC or SD-JWT format
  Issued by trusted authority in source country's trust registry

Target country verification:
  [AI Document Translation] → translate human-readable fields
  [Trust Registry Lookup] → is source country's issuer in target's trusted list?
  [Credential Verification] → cryptographic signature valid
  [Equivalency Assessment] → AI maps source qualification to target country's framework
    "BSc Computer Science (3-year) from University X is equivalent to BSc (Hons) in Computing, Level 6"
  [Professional License Decision] → AI assists human assessor with equivalency analysis

Output:
  Recognition decision in hours (vs. months manually)
  Verifiable Credential issued confirming recognized equivalency
```

## Impact Measurement for DPI-AI

Key metrics to track when deploying AI on DPI:

| Metric | Description |
|--------|-------------|
| **Inclusion rate** | % of population that can use AI-DPI services vs. manual alternative |
| **Accuracy** | AI decision quality vs. human benchmark (precision, recall, F1) |
| **Bias parity** | Outcome equality across demographic groups |
| **Cost per service** | Cost reduction vs. non-AI DPI baseline |
| **Time to service** | End-to-end time reduction |
| **Escalation rate** | % of cases AI can handle autonomously vs. human escalation |
| **Citizen satisfaction** | NPS / CSAT for AI-assisted services |
| **Audit completion** | % of AI decisions with complete audit trail |
