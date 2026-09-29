**Similar Incidents, Different Causes: Filtering Hindsight Recall Before the LLM**

When an incident happens, I want more than a generic troubleshooting checklist. I want to know whether we have seen something similar before, what we tried, and whether that resolution actually worked.

That is the problem I built IncidentIQ to address. It uses persistent organizational memory to give engineers historical context during incident investigation, while keeping diagnosis and verification under human control.

The core workflow is:

**Retain → Recall → Filter → Reason → Verify → Retain**

The interesting engineering problem was not simply giving the system memory. It was deciding how much of that memory should be trusted before passing it to an LLM.

**How IncidentIQ fits together**
IncidentIQ has a deliberately small architecture.

The frontend uses HTML, CSS, and JavaScript to provide the incident dashboard. The backend is written in Python with FastAPI. Hindsight provides persistent incident memory, while Groq generates troubleshooting guidance using the current incident and selected historical context. Python’s requests library handles HTTP communication with Hindsight, and SequenceMatcher is used for application-level duplicate filtering.

The main application endpoints are:

POST /api/retain
POST /api/recall
POST /api/suggest
The architecture separates four responsibilities:

Engineer
↓
IncidentIQ Frontend
↓
FastAPI Backend
↓
Hindsight Recall
↓
Memory Filtering
↓
Groq Reasoning
↓
Human Verification
↓
Hindsight Retain

I use Hindsight for persistent agent memory as the memory layer rather than treating the LLM’s context as long-term organizational memory. The Hindsight documentation and API reference describe the Retain and Recall APIs and the broader memory capabilities behind the system.

The important boundary is simple:

Hindsight provides memory. Groq provides reasoning. IncidentIQ controls the workflow between them.

There is no automatic production change in this workflow. An engineer reviews the generated troubleshooting guidance, investigates the issue, and verifies the result.

**Retaining incidents as organizational memory**
When an engineer saves an incident, IncidentIQ records more than the original description. It stores the resolution, outcome, and whether the result was verified.

The application builds the memory content like this:

ccontent = (
f"Incident: {incident.description}\n"
f"Resolution: {incident.resolution or 'Not recorded'}\n"
f"Outcome: {incident.outcome}\n"
f"Verified: {'Yes' if incident.verified else 'No'}"
)
That record is sent to the incidentiq Hindsight memory bank using its HTTP API, along with context and a generated document ID.

This distinction matters. A description tells me that an incident happened. A description combined with a resolution, outcome, and verification status captures what we actually learned from it.


The broader idea of persistent memory is important here. Vectorize’s explanation of what agent memory is and why it matters provides useful context for understanding why an application that operates across separate incidents needs memory outside the immediate LLM interaction.

**Recall is not reasoning**
When a new incident is submitted, IncidentIQ sends its description to Hindsight Recall.

The relevant request is straightforward:

response = requests.post(
url,
json={
"query": description,
"max_tokens": 2000,
},
timeout=120,
)
Hindsight returns memories that are relevant to the current incident.

But I deliberately keep retrieval separate from reasoning.

The flow is:

Current incident
↓
Hindsight Recall
↓
Retrieved memories
↓
IncidentIQ filtering
↓
Selected historical context
↓
Groq
Two incidents can have nearly identical symptoms but completely different causes.

For example:

Login API returned HTTP 503 after deployment.

Root cause: authentication service URL was incorrect.

and:

Login API returned HTTP 503 after deployment.

Root cause: database connection configuration was invalid.

Both could be relevant to a new login failure. Passing both directly to an LLM may provide useful context, but it also introduces conflicting evidence that the application can partially handle before reasoning begins.

Filtering memory before the LLM sees it
IncidentIQ performs its own filtering after Recall.

The first step is near-duplicate detection using Python’s SequenceMatcher:

similarity = SequenceMatcher(
None,
text.lower(),
old_text.lower(),
).ratio()
if similarity >= 0.82:
duplicate = True
A similarity score of 0.82 or higher is treated as a near-duplicate by the application.

This is an IncidentIQ-level filtering step, not something I attribute to Hindsight itself.

I also use a small cause-signature heuristic for explicitly recognized categories such as database configuration and authentication-service URLs. It is not intended to determine the root cause. Its purpose is to prevent obviously different known cause categories from being collapsed simply because their incident descriptions look similar.

After filtering, IncidentIQ passes at most six selected memories to Groq.

That gives the system a controlled boundary between retrieval and reasoning.
What an investigation looks like
Consider an incident entered into the application:

After a deployment, the login API returns HTTP 503 errors.

Users cannot sign in. The service worked normally before deployment.

IncidentIQ sends this description to Hindsight and retrieves related historical incidents.

Suppose the history contains previous login failures caused by different configuration problems. The system does not assume that the newest incident has the same root cause.

Instead, the relevant memories provide context for the troubleshooting process.

Groq receives the current incident and the selected historical context and produces prioritized diagnostic checks. The engineer then investigates those checks and verifies the result.

If the issue is resolved, the engineer saves the resolution, outcome, and verification status. IncidentIQ retains that information in Hindsight, making it available to future investigations.

The resulting learning loop is:

Incident
↓
Recall
↓
Filter
↓
Troubleshoot
↓
Verify
↓
Retain
↓
Future Recall

**What I learned**
1. Memory quality matters as much as memory availability
Adding a memory layer does not automatically improve troubleshooting.

If retrieval produces duplicated or conflicting records, the model gets more context but not necessarily better context. Filtering became an important part of the application because of that.

2. Retrieval is not reasoning
Keeping Hindsight and Groq separate makes the system easier to understand.

Hindsight answers:

What relevant experience do we have?

Groq answers:

Given this incident and that experience, what should we investigate?

That boundary also means retrieval behavior can evolve without redesigning the reasoning layer.

3. Similarity is not root cause
The SequenceMatcher threshold is useful for identifying similar text. It is not evidence that two incidents share a root cause.

That distinction is important enough to encode in the system rather than relying on the model to infer it every time.

4. Previous fixes need verification
A historical resolution is evidence, not an instruction.

The current incident still needs investigation, and the resulting fix still needs verification before it becomes useful historical knowledge.

5. The real value is the loop
The first troubleshooting response is only one part of the system.

The more important behavior is what happens afterward: an engineer verifies the outcome and records it so that the next incident can benefit from that experience.

**What I would improve next**
I would capture richer incident metadata such as affected services, timestamps, environments, deployment identifiers, and components. That would provide additional retrieval signals beyond free-form incident text.

I would also evaluate retrieval against a controlled collection of historical incidents instead of relying only on individual examples. The useful question is not whether the system can recall something similar, but whether it consistently recalls information that helps distinguish between plausible causes.

Hindsight also provides capabilities beyond the basic Retain and Recall flow I use here. I would evaluate those capabilities based on actual memory-quality requirements rather than adding them simply because they are available.

The architectural principle would remain the same:

IncidentIQ remembers. Hindsight retrieves. Groq reasons. The engineer verifies.

That separation is what makes the system useful without turning historical memory or generated text into unquestioned truth.