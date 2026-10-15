---
name: constraint-driven-architecture
description: Design or evaluate distributed systems by starting from the goal, deriving the properties the system needs, and choosing interaction constraints that induce those properties. Use for APIs, agent systems, marketplaces, platforms, protocols, multi-service architectures, and other systems where independent components must cooperate, scale, evolve, or establish trust without sharing the same internal implementation.
---

# Constraint-Driven Architecture

Use this skill to reason about architecture from the outside in.

The central principle is:

> Do not begin with the machinery. Begin with the goal, the environment, and the properties the system must exhibit. Then derive the constraints on interaction that make those properties more likely.

This skill is inspired by Roy Fielding's REST dissertation, Christopher Alexander's pattern-oriented architectural thinking, and the broader problem of designing environments in which independent components can cooperate without requiring identical internals.

REST is a case study and reasoning model here, not a universal answer.

## Core mental model

Architecture is not merely a diagram of components.

Architecture is a set of constraints on how components may interact. Those constraints shape behavior and induce system-level properties.

A useful reasoning chain is:

**Goal → Environment → Desired Properties → Constraints → Components → Interfaces → State → Resources/Representations → Intermediaries → Failure Modes → Evolution**

Always reason in that direction before choosing a technology stack.

## Foundational principles

### Constraints can create freedom

A constraint removes some local freedom from individual components, but can create greater freedom for the system as a whole.

Examples:

- Stateless interaction limits hidden server-side conversational memory, but improves visibility, recoverability, and horizontal scalability.
- A uniform interface limits application-specific interaction styles, but enables independent implementations to interoperate.
- Layering limits how much each component can see, but contains complexity and permits intermediaries and substitutions.
- Cache rules constrain reuse, but can dramatically reduce latency and load.

When evaluating a proposed constraint, always ask:

1. What freedom does this remove?
2. What property does that restriction induce?
3. Is the tradeoff worth it for this system?

### Separate internals from interfaces

Do not require interacting components to know one another's internal implementation unless the goal truly requires it.

Focus on what must be visible at the boundary:

- identity
- intent
- capabilities
- permissions
- message semantics
- resource identifiers
- representation formats
- state transitions
- errors
- provenance
- trust signals

Internal implementation may change independently as long as the interface contract remains coherent.

A good architecture standardizes the meaning of interaction more than the private machinery behind it.

### Uniformity does not mean sameness

Independent components do not need to use the same programming language, database, model, operating system, vendor, or internal architecture.

They need a sufficiently common way to meet at the boundary.

The objective is interoperability without forced internal uniformity.

### Architecture shapes behavior

Treat architecture as the design of an environment, not merely the assembly of parts.

Ask:

> What kinds of behavior will this environment naturally make easy, hard, cheap, expensive, visible, invisible, safe, or dangerous?

A well-designed environment induces useful behavior through structure rather than constant centralized control.

### Stable identity, changing representation

Distinguish the thing being identified from the way it is currently represented.

A **resource** is the conceptual thing or relation being identified.

A **representation** is a transferable description of the resource's current or intended state.

A stable resource can have:

- changing state
- multiple representations
- different storage backends
- different implementations over time

Do not bind external identity unnecessarily to internal storage details.

### State must live somewhere

"Stateless" does not mean "no state."

It means that a component should not depend on hidden conversational state elsewhere when the interaction model says the message should be independently understandable.

Always identify:

- resource state
- application/session state
- workflow state
- identity/authentication state
- authorization/delegation state
- cache state
- durable historical state

Then decide deliberately where each form of state belongs.

### Self-descriptive interaction increases visibility

A message is more architecturally useful when an authorized observer can understand enough of its semantics to route, cache, validate, audit, transform, or reject it without requiring private knowledge of the sender's internal code.

This is especially important when intermediaries matter.

### Layering bounds knowledge

A component should generally need to understand the layer or counterpart it directly interacts with, not the entire path or implementation chain.

Layering can enable:

- gateways
- proxies
- firewalls
- caches
- load balancers
- policy enforcement
- protocol translation
- legacy encapsulation
- organizational boundaries

But every layer can also add latency, failure points, and opacity. Add layers for a reason.

### The most efficient request may be the one you never send

Caching, locality, reuse, precomputation, and appropriate replication can reduce network work.

But reuse must preserve semantics. A cached answer is useful only when it is valid for the request being answered.

Always ask:

- Is this response reusable?
- Under what dimensions may it vary?
- How fresh must it be?
- Does this action require an authoritative answer?
- Would reuse violate access control or user-specific semantics?

### Design for partial failure

Networks fail partially.

A component can be healthy while another is slow, unreachable, stale, inconsistent, or returning an intermediary-generated error.

Do not reason about a networked system as if it were one local function call.

Consider:

- timeout
- retry
- duplicate execution
- stale data
- partial completion
- lost response
- unavailable dependency
- split authority
- recovery after restart

### Design for independent evolution

Distributed systems often span teams, organizations, vendors, versions, and deployment schedules.

Assume components will change at different times.

Prefer interfaces that allow:

- old and new implementations to coexist
- gradual deployment
- backward-compatible evolution
- replacement of internal machinery
- independent upgrades

Avoid unnecessary global coordination.

## Workflow

When asked to design, critique, or explain an architecture, follow this sequence.

### 1. State the goal

Write the goal in functional terms.

Bad:

> Build an agent platform with MCP, A2A, Postgres, and Kubernetes.

Better:

> Enable independently built agents to discover one another, establish permission, delegate tasks, exchange results, and recover from partial failure.

Do not choose technologies yet.

### 2. Define the environment

Identify the forces that exist whether the designer likes them or not.

Examples:

- network latency
- unreliable connectivity
- multiple organizations
- untrusted participants
- heterogeneous implementations
- regulation
- human approval requirements
- cost limits
- privacy boundaries
- limited compute
- physical-world constraints
- legacy systems

Treat these as real forces, not inconveniences to hand-wave away.

### 3. Identify desired properties

Ask what the system must be good at.

Common properties include:

- scalability
- reliability
- visibility
- evolvability
- interoperability
- latency
- privacy
- security
- auditability
- autonomy
- locality
- availability
- consistency
- simplicity
- reversibility
- trust

Do not treat all properties as equally important. Rank them.

### 4. Derive constraints from properties

For each important property, ask what restrictions on component behavior could induce it.

| Desired property | Candidate constraint | Benefit | Cost |
|---|---|---|---|
| Scalability | Stateless requests | Easier horizontal distribution | Repeated context |
| Evolvability | Stable interface semantics | Internals can change independently | Less app-specific optimization |
| Low latency | Cacheable responses | Reuse nearby results | Staleness risk |
| Visibility | Self-descriptive messages | Intermediaries can reason about traffic | More explicit metadata |
| Modularity | Layered interaction | Bounds component knowledge | Added hops/complexity |

Never introduce a constraint without naming the property it is meant to produce.

### 5. Define component roles

Identify the actors and their responsibilities.

Examples:

- client / user agent
- service / origin
- cache
- proxy
- gateway
- resolver
- policy layer
- identity provider
- coordinator
- agent
- tool
- human approver

Avoid giving one component responsibilities that belong at multiple architectural layers unless there is a compelling reason.

### 6. Draw the interface boundary

For every interaction, specify:

- who initiates
- who receives
- what is identified
- what intent is expressed
- what context must travel
- what may remain private
- what response semantics exist
- what errors mean
- what intermediaries are allowed to do

Ask:

> What must the other side know?

Then ask:

> What does it not need to know?

The second question protects independence.

### 7. Place state deliberately

Create a state map.

For each kind of state, record:

- owner
- authority
- lifetime
- persistence
- replication policy
- visibility
- recovery strategy

If state is hidden in a place that other components implicitly depend on, flag it.

### 8. Model resources and representations

When the system exposes identifiable things, separate:

**Resource identity**  
What concept does the identifier mean over time?

**Resource state**  
What facts are true about it now?

**Representation**  
What data is being transferred to describe or manipulate that state?

**Implementation**  
Where and how is the resource actually stored or computed?

Prefer stable identifiers whose semantics survive backend changes.

### 9. Evaluate intermediaries

Ask whether middle layers should be able to:

- cache
- route
- transform
- authorize
- filter
- observe
- audit
- retry
- translate
- balance
- enforce policy

If yes, ensure enough interaction semantics are visible for them to do so safely.

If messages are opaque, intermediaries can usually act only as routers.

### 10. Analyze latency and communication cost

Count meaningful interactions, not just computation.

Ask:

- How many network round trips are required?
- Can useful work begin before the whole response arrives?
- Can the system send the likely-best answer first?
- Can some data be reused locally?
- Is negotiation adding extra trips?
- Is the architecture chatty when it should be coarse-grained?

For network systems, communication structure often matters more than local computation cost.

### 11. Model failure explicitly

Run failure scenarios:

- The server crashes after accepting a request.
- The response is lost.
- The client retries.
- An intermediary returns stale data.
- A dependency is unavailable.
- Two participants disagree about state.
- Authorization changes mid-workflow.
- A long-running task is interrupted.
- A message arrives twice.
- A participant upgrades while others do not.

Determine whether the interface makes recovery understandable.

### 12. Test independent evolution

Ask:

- Can the client change without the server changing?
- Can the server change without the client changing?
- Can an intermediary be inserted or removed?
- Can implementations be replaced while identifiers remain stable?
- Can multiple versions coexist?
- Can different organizations deploy on different schedules?

If every change requires coordinated reconstruction, the architecture is tightly coupled.

### 13. Look for architectural mismatch

Compare the stated goals to the actual design.

Examples:

- Claims scalability but requires global shared session state.
- Claims decentralization but every action requires one central authority.
- Claims privacy but exposes unnecessary internal data at interfaces.
- Claims independent evolution but all participants must use one SDK implementation.
- Claims visibility but messages hide essential semantics.
- Claims interoperability but uses application-specific operations for every resource.
- Claims resilience but assumes remote calls fail like local function calls.

Name the mismatch clearly.

### 14. Only then choose technologies

After the structure is clear, evaluate protocols, databases, frameworks, runtimes, and vendors.

Technology should satisfy the architecture, not define it by default.

## REST as a diagnostic lens

Use REST's constraints as questions, not commandments.

### Client-server

Is there a useful separation of concerns between requester and provider?

What becomes easier to evolve independently because of that separation?

### Statelessness

Can each interaction be understood without relying on hidden conversational memory at the receiver?

If not, is the retained state necessary, explicit, bounded, recoverable, and correctly owned?

### Cache

Can prior results be reused safely?

What makes a response fresh, stale, private, authoritative, or variant-specific?

### Uniform interface

Are interaction semantics consistent enough that independent components and intermediaries can understand them?

Is the interface describing generic intent or leaking application-specific internal procedures?

### Layered system

Can components operate without knowing the entire chain behind their immediate counterpart?

Can useful intermediaries exist without breaking semantics?

### Code-on-demand

Would sending executable behavior to another component improve extensibility enough to justify reduced visibility and greater security complexity?

Treat this as optional.

## Agentic systems extension

Autonomous agents raise the same architectural questions at a higher level of capability.

Do not assume that smarter components eliminate the need for architecture.

Often the opposite is true.

When agents can plan, act, spend, delegate, negotiate, or modify external systems, explicitly design for:

### Identity

Who or what is this agent?

Can that identity be authenticated across organizations?

### Capability discovery

What can this agent do?

How are capabilities described without exposing private internals?

### Authority

What is this agent allowed to do?

Authority should be explicit, scoped, and revocable.

### Delegation

Can an agent grant another agent limited authority?

Track:

- who delegated
- what was delegated
- scope
- duration
- budget
- constraints
- revocation
- responsibility

### Intent

What outcome is the agent attempting to produce?

Separate intent from implementation.

Two agents may satisfy the same intent using completely different internal strategies.

### State

Agent workflows may be long-running and stateful.

Do not force statelessness where durable task state is necessary.

Instead distinguish:

- conversational context
- task state
- resource state
- authorization state
- memory
- durable audit history

### Provenance

Where did a claim, artifact, decision, or action come from?

Preserve enough lineage to distinguish:

- retrieved fact
- user instruction
- agent inference
- transformed data
- generated content
- delegated work
- external tool result

### Trust

Trust should not be a vague score.

Decompose it into observable properties such as:

- authenticated identity
- verified authority
- provenance
- reputation
- policy compliance
- transaction history
- cryptographic proof
- human approval
- bounded liability

### Negotiation

Agents may need richer negotiation than traditional clients.

Design explicit semantics for:

- offers
- requirements
- constraints
- alternatives
- cost
- quality
- deadlines
- cancellation
- partial acceptance

### Failure and recovery

An intelligent agent can still fail.

Plan for:

- hallucinated assumptions
- invalid tool outputs
- contradictory instructions
- lost delegation chains
- duplicated actions
- stale context
- revoked access
- interrupted workflows
- counterpart non-cooperation

### Human boundaries

Specify when humans must remain in the loop.

Do not leave human review as an informal afterthought.

## Pattern thinking

Use a pattern when the same relationship between forces, constraints, and behavior repeatedly appears across contexts.

A pattern should describe:

- the recurring problem
- the forces in tension
- the structural response
- the property it tends to create
- the tradeoffs
- the contexts where it fails

Do not copy a pattern mechanically.

A pattern is a reusable relationship, not a template that ignores context.

## Architecture review questions

Use these questions to strip a system down to the studs:

- What are we trying to make possible?
- Who are the independent actors?
- What forces in the environment cannot be ignored?
- Which system properties matter most?
- What constraints induce those properties?
- What freedom does each constraint remove?
- What is the interface?
- What semantics must be visible at the interface?
- What can remain private?
- Where does state live?
- Who owns and authoritatively controls that state?
- What are the stable resource identities?
- What representations cross the boundary?
- Which intermediaries should be intelligent participants rather than mere routers?
- What can be cached or reused?
- What requires an authoritative source?
- What happens during partial failure?
- Can components evolve independently?
- Which assumptions depend on one implementation, vendor, or library?
- Where does the proposed implementation contradict the stated architectural goal?
- What can be removed without harming the desired properties?

## Default output

Unless the user asks for another format, produce a concise architecture analysis with:

1. **Goal**
2. **Environment and forces**
3. **Desired properties**
4. **Derived constraints**
5. **Interface and state model**
6. **Failure/evolution analysis**
7. **Architectural mismatches**
8. **Recommended direction**

For exploratory conversations, do not force this structure. Use it internally and explain the system naturally.

## Interaction style

When teaching:

- Start with the simple mechanism.
- Use a concrete analogy only when it clarifies the architecture.
- Clearly mark where an analogy stops being technically exact.
- Distinguish the original architectural concept from broader philosophical extrapolation.
- Prefer "why this constraint exists" over memorizing vocabulary.
- Connect each technical detail back to the system property it induces.

When designing:

- Do not jump directly to tools or frameworks.
- Do not assume REST is always correct.
- Do not equate distributed calls with local function calls.
- Do not confuse resource identity with storage location.
- Do not confuse state with "memory" in general.
- Do not treat uniform interfaces as identical implementations.
- Do not hide important semantics in opaque side channels.
- Do not optimize one component at the expense of the system goal without naming the tradeoff.

## Compact heuristic

When time is short, use this:

> **Goal → Properties → Constraints → Interface → State → Failure → Evolution**

Then ask:

> Does the design actually induce the properties we said we wanted?

If not, identify the mismatch before adding more machinery.

## Final principle

As intelligence and code become cheaper, architecture becomes more important, not less.

The scarce capability may increasingly be the ability to arrange capable components well:

**not controlling every move, but designing the constraints, interfaces, and environments that give capable things useful direction.**
