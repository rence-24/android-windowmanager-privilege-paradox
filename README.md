# Android WindowManager Privacy-State Synchronization Observation

A Technical Case Study of Protected Application Content and Multimodal Assistant Processing

Author: Lawrence Bernales

Initial Observation: May 25, 2026

Follow-up Diagnostics: September 2026

Platforms Tested: Android 14, Android 15, Android 16

Abstract

This research documents an observed privacy-boundary issue involving Android application-level visual protection, system-level window and capture state, and multimodal assistant processing.

The investigation does not oppose Gemini Live, screen awareness, or multimodal assistant functionality. These features can provide legitimate accessibility and productivity capabilities.

The security question examined here is narrower:

When an application declares visual content as protected, does that privacy restriction remain consistently enforced across system-level capture, snapshot, and downstream multimodal processing during application-state transitions?

During the original testing, Gemini Live was observed to identify sensitive visual information displayed within a tested application context. Subsequent testing and Android framework diagnostics were used to investigate the system state surrounding the behavior.

The research does not claim access to AOSP source code, Google's internal Gemini implementation, or internal telemetry. Accordingly, the underlying architectural explanation is presented as a technical hypothesis based on observed behavior, rather than a confirmed source-code root cause.

# I. SECURITY PRINCIPLE

Android applications can use mechanisms such as FLAG_SECURE to communicate that sensitive visual content should receive protection from screenshots and related capture mechanisms.

This protection is particularly important for applications handling:

financial information;

authentication information;

personal information;

account data;

other sensitive visual content.

The purpose of this research is not to argue that Gemini Live should be prevented from seeing screens generally.

Instead:

Multimodal functionality should continue to operate normally where permitted, while application-declared privacy restrictions should remain effective when protected content is encountered.

The security concern therefore exists at the interaction boundary between otherwise legitimate components.

# II. OBSERVED BEHAVIOR

A. Original Testing

The original testing was performed during May–August 2026 using Android devices and controlled application scenarios.

Evidence included:

screen recordings;

screenshots;

application-state observations;

comparison between protected and non-protected contexts;

Gemini Live visual/overlay behavior.

The strongest impact evidence is behavioral.

During one documented reproduction, Gemini Live was able to identify a sensitive financial value displayed within the tested application context.

For public disclosure, actual credentials, OTPs, authentication values, and other secrets are intentionally omitted.

B. Protected vs. Non-Protected Contexts

Testing showed that application behavior differed depending on the privacy mechanisms implemented by the application.

Protected application contexts could cause Gemini's visual access to become unavailable or restricted.

Other application contexts did not necessarily provide the same protection.

This difference motivated investigation into whether application privacy state and system-level visual processing state remain synchronized during transitions.

# III. EVIDENCE CHRONOLOGY

May–August 2026 — Original Evidence

The original report and reproduction were based primarily on:

controlled device testing;

screen recordings;

screenshots;

observed Gemini Live behavior;

application privacy behavior;

comparison between different applications and Android environments.

No claim is made that later ADB/Linux diagnostics were available during this original testing period.

September 2026 — Follow-Up Diagnostics

A laptop became available later in the investigation.

Additional ADB/Linux diagnostics were then performed, including examination of:

WindowManager task snapshots;

TaskSnapshot;

HardwareBuffer;

mIsRealSnapshot;

MediaProjection state;

foreground application/task transitions.

These diagnostics are presented as subsequent technical validation, not as evidence that existed during the original report.

# IV. ANDROID FRAMEWORK OBSERVATIONS

A. WindowManager State

Observed framework telemetry included states such as:



mBaseLayer=21000
mSnapshot=android.hardware.HardwareBuffer@...
mIsRealSnapshot=true

The 21000 value is treated only as an observed WindowManager layer/state value.

It is not presented as proof that the layer itself represents an unauthorized privilege.

Similarly, mIsRealSnapshot=true is treated as framework telemetry describing snapshot state.

It is not treated as standalone proof that a particular protected frame was delivered to Gemini.

B. MediaProjection State

Later diagnostics also showed:



Media Projection:
(com.google.android.googlequicksearchbox, uid=10116):
TYPE_SCREEN_CAPTURE

This demonstrates the presence of a MediaProjection-related screen-capture state associated with the Google application package during testing.

However:

The presence of a MediaProjection session alone does not establish that protected application pixels were delivered to the multimodal model.

This distinction is important to the interpretation of the evidence.

# V. SNAPSHOT AND TRANSITION OBSERVATIONS

Testing demonstrated that different tasks could have different snapshot states.

For example, during protected browsing transitions, a Chrome task was observed with:



mIsRealSnapshot=false

while a Gemini-associated task could independently show:



mIsRealSnapshot=true

This demonstrates why mIsRealSnapshot cannot be treated as a direct indicator of whether Gemini can or cannot see protected content.

The relevant security question is broader:

Does the privacy state of the protected application remain consistently represented across the components participating in visual capture and multimodal processing?

# VI. OBSERVED PRIVACY IMPACT

The central security observation is not the WindowManager layer number or an individual snapshot field.

It is the observed handling of sensitive application-derived visual information.

During the documented testing:



Sensitive application content
          ↓
Device visual state
          ↓
Gemini Live visual processing
          ↓
Assistant interpretation

Gemini Live was able to identify a sensitive financial value displayed within the tested application context.

This establishes a visual privacy exposure under the tested conditions.

The behavior can reasonably be described as a form of visual eavesdropping across an application privacy boundary, because information originating inside a privacy-sensitive application context became available to an assistant processing path that the application was expected to restrict.

The public disclosure intentionally does not reproduce the actual sensitive value beyond the minimum evidence necessary to demonstrate the impact.

# VII. CONVERSATIONAL / SESSION HISTORY

Where sensitive application-derived information subsequently appears in an associated conversational or session history, this creates an additional privacy concern.

The important distinction is:

The observation of sensitive content in conversation history establishes persistence of the observed information at the session/application layer; it does not by itself establish the underlying storage format or backend storage architecture.

Therefore, this research does not classify the behavior as CWE-312 solely because the information appeared in conversation history.

A separate cleartext-storage determination would require evidence establishing how that information is actually stored and protected.

The practical concern remains that:



Protected application content
          ↓
Assistant processing
          ↓
Conversation/session representation

can extend the privacy exposure beyond the original application screen.

# VIII. TECHNICAL INTERPRETATION

Cross-Component Privacy-State Synchronization

The current working hypothesis is a possible capture/snapshot/processing state-reconciliation gap.

The relevant components can be represented as:



Application privacy state
          │
          ▼
WindowManager lifecycle state
          │
          ▼
Capture / snapshot state
          │
          ▼
Assistant visual-processing state
          │
          ▼
Multimodal/session processing

Each component can be individually legitimate while the interaction between their state transitions creates an inconsistent privacy boundary.

The hypothesis is therefore:

During certain application-state transitions, the privacy state of the protected application may not be synchronously reconciled across every downstream component capable of receiving visual or multimodal state.

This is a root-cause hypothesis, not a confirmed AOSP implementation finding.

# IX. WHY FLAG_SECURE / PRIVACY CONTROLS MATTER

The purpose of a mechanism such as FLAG_SECURE is not merely to control screenshots initiated manually by the user.

Its security purpose is to communicate that application content is sensitive and should not be exposed through applicable visual-capture paths.

Therefore, when a protected application transitions between states:



Protected
   ↓
Hidden / background
   ↓
Foreground transition
   ↓
Another application

the relevant capture and processing components should consistently respect the application's privacy state.

The research therefore does not argue against system-level assistant overlays or multimodal processing.

It argues for consistent privacy-state enforcement when protected content is encountered.

## X. REFERENCES & PRIOR ART

Previous Android research has documented security issues involving system assistants and `FLAG_SECURE`.

**CVE-2019-2103** — Android 9 Google Assistant `FLAG_SECURE` screenshot-permission bypass resulting in information disclosure.

**Pankaj Upadhyay (2020)** — *OK Google, Bypass FLAG_SECURE* — prior research documenting the historical interaction between Google Assistant and Android's `FLAG_SECURE` protection.

These references provide historical context for the present research. This case study does **not** claim that the current observation is the same vulnerability as CVE-2019-2103. The present investigation concerns the interaction between modern multimodal assistant processing, application privacy state, and Android framework lifecycle/capture state.

# XI. DEVICE AND ANDROID COVERAGE

Testing included multiple devices and Android versions:

DeviceAndroid



TECNO

Android 14

OPPO

Android 15

Samsung

Android 15

Xiaomi

Android 16

The purpose of this testing was to determine whether the behavior was limited to one device implementation.

Observed behavior varied depending on application, device, Android version, and application-level privacy implementation.

Therefore, this research does not claim universal reproduction across every Android device.

# XII. EVIDENCE BOUNDARIES

Directly observed

Gemini Live could identify sensitive visual information under the documented test conditions.

Protected and non-protected application contexts behaved differently.

Screen recordings and screenshots documented the original behavior.

Android framework diagnostics later showed WindowManager snapshot states.

mIsRealSnapshot=true and mIsRealSnapshot=false were both observed in different task contexts.

MediaProjection-related screen-capture state was observed.

Multiple application/task states could remain represented during transitions.

Not directly established

This research does not claim to establish:

the exact AOSP source-code root cause;

Gemini's internal capture implementation;

Google's internal multimodal processing architecture;

that mIsRealSnapshot=true itself causes the exposure;

that every MediaProjection session exposes protected content;

that every Android device is affected;

that a particular WindowManager layer number represents unauthorized privilege;

that Google silently implemented or removed a specific mitigation;

the precise backend storage architecture of conversation history.

# XIII. CWE CLASSIFICATION

## Primary — CWE-213

**CWE-213: Exposure of Sensitive Information Due to Incompatible Policies**

This is the primary classification because the documented behavior concerns the interaction of different privacy expectations and policies across components.

The application expects protected content to remain restricted while another system-level component has legitimate visual-processing capabilities.

The security concern therefore occurs at the policy interaction boundary.

## CWE-312 — Not Assigned

CWE-312 is not assigned as a confirmed classification.

Although sensitive information was observed in conversational/session context, the available evidence does not establish that the information was stored in cleartext in the technical sense required for CWE-312.

The persistence concern is therefore documented separately rather than overstating the CWE classification.

# XIV. SECURITY IMPACT

The practical impact is:

Sensitive visual information originating inside an application privacy boundary may become available to a system-level multimodal assistant and may subsequently appear in associated conversational/session processing.

Potentially affected information could include:

financial information;

account information;

personal information;

authentication-related visual content;

other sensitive data displayed by protected applications.

The severity depends on:

the application's privacy mechanism;

the device and Android implementation;

the assistant configuration;

the specific lifecycle transition;

the type of information displayed.

# XV. MITIGATION CONSIDERATIONS

A robust implementation should ensure that privacy state is consistently propagated across:



Application privacy state
        ↓
WindowManager state
        ↓
Capture / snapshot state
        ↓
Assistant visual input
        ↓
Multimodal processing
        ↓
Session/history handling

Possible defensive principles include:

Synchronous privacy-state reconciliation during foreground/background transitions.

Invalidation or restriction of downstream visual state when protected content becomes active.

Consistent enforcement of application-declared visual privacy controls across capture paths.

Avoidance of stale protected visual state remaining available to downstream processing after a privacy transition.

Independent validation of privacy state at the point where visual data enters multimodal processing.

These are defensive design considerations, not claims about Google's current implementation.

# XVI. CONCLUSION

This research does not argue that Gemini Live or multimodal assistant functionality should be disabled.

The central issue is the privacy boundary between protected application content and system-level visual processing.

The strongest evidence is behavioral:

Under the documented test conditions, Gemini Live was able to identify sensitive visual information displayed within a tested application context.

Additional Android framework diagnostics provide supporting evidence for investigating how application privacy state, WindowManager snapshot state, capture state, and downstream multimodal processing interact during lifecycle transitions.

The available evidence is consistent with a possible cross-component capture/snapshot/processing state-reconciliation gap.

However, without access to AOSP internals, proprietary Gemini implementation details, or internal telemetry, the exact root cause cannot be conclusively established from the available evidence.

The appropriate conclusion is therefore:

The observed behavior represents a meaningful visual privacy exposure at the interaction boundary between application-level privacy controls and system-level multimodal processing. Further investigation should focus on ensuring that protected application state remains consistently enforced across the entire visual-processing lifecycle.

# XVII. DISCLOSURE

The original issue was reported through Google's vulnerability reporting process.

The public research intentionally:

omits credentials and OTPs;

redacts sensitive account information;

separates original reproduction evidence from later diagnostic investigation;

distinguishes observed behavior from architectural hypothesis;

does not publish proprietary or confidential vendor communications.

Planned disclosure date: September 30, 2026.
