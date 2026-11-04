---
copilot_chat: obsidian://copilot-chat?vault=Dispatch-Wiki&id=epoch%3A1793099431555
---
# Dispatch Mesh naming recommendations

Date: 2026-10-27
Status: Naming exploration; no final name selected.

Related: [[04_Discovery/git-sync/README]] and [[requirements and architecture]].

## Product positioning

The product synchronizes files over the network as easily as Google Drive, with Git-backed traceability of who changed what and why. It supports teams and autonomous agents maintaining shared context, including an LLM Wiki, while remaining useful for other kinds of files and work.

For LLM edits, commit messages preserve the user's intent from the associated prompt and summarize the actual changes. For direct document edits without recorded intent, they preferably summarize changes; any inferred intent is identified as inferred.

The preferred name should have a technical or agent-oriented character. A relationship to Dispatch or shared context is desirable, but the name should allow use beyond Dispatch and LLM Wikis. The earlier courier, post, and chronicle direction felt too much like a newspaper and was rejected.

## Recommended shortlist

| Name | What it communicates | Tradeoff |
|---|---|---|
| **Dispatch Mesh** | A synchronization layer connecting people, agents, devices, and workspaces | May imply peer-to-peer networking; overlaps with the graphics API term `DispatchMesh` |
| **IntentReplica** | Replicating files together with the intent behind their changes | Technical and distinctive in tone, but less immediately approachable; recorded intent is not available for every edit |
| **ContextConverge** | Bringing distributed knowledge and edits into a shared state | Strong fit for an LLM Wiki, but longer and less explicit about file synchronization |
| **Dispatch Flux** | A continuous flow of changes through the system | Flux is crowded in software naming; nearby wording also appears in unrelated businesses |

## Recommendation

**Dispatch Mesh** is the first choice if the product should clearly belong to the Dispatch family. It conveys connected workspaces and collaboration between people and agents. The mesh metaphor is a branding choice, not a requirement to implement a peer-to-peer network.

**IntentReplica** is the first choice for an independent product. It combines the synchronization mechanism with the differentiator: preserving the reason behind a change when that reason is available.

**ContextConverge** is the alternative if shared knowledge should be more prominent than file replication.

Dispatch Flux remains a secondary option because of its crowded terminology.

### Example presentation

> **Dispatch Mesh**  
> File synchronization with history and intent.  
> For teams and agents.

Or, as an independent product:

> **IntentReplica**  
> File synchronization with history and intent.  
> For teams and agents. Built for Dispatch.

## Preliminary naming checks

Exact-name and software-related web searches were performed on 2026-10-27. These are preliminary search results, not confirmation of name availability.

- **Dispatch Mesh:** a direct technical overlap exists with `DispatchMesh`, a graphics API term. This could affect searchability. [AMD GPUOpen explanation](https://gpuopen.com/learn/mesh_shaders/mesh_shaders-from_vertex_shader_to_mesh_shader/)
- **IntentReplica:** no obvious exact-name product match appeared in the searches performed.
- **ContextConverge:** no obvious exact-name product match appeared in the searches performed.
- **Dispatch Flux:** the wording is close to the existing unrelated business Flux Dispatch. [Flux Dispatch](https://fluxdispatch.com/)

Domain registrations, package registries, executable-name collisions, and trademark availability have not been verified. Check these for the selected finalist before adopting the name publicly.

## Names excluded during exploration

| Name | Reason |
|---|---|
| git-sync | Existing [Kubernetes git-sync project](https://github.com/kubernetes/git-sync) |
| Dispatch Relay | Existing [DispatchRelay product](https://www.catalystsyndicate.io/dispatchrelay/) |
| Dispatch Link | Existing [fleet management product](https://www.dispatchlinksolutions.com/) |
| Weave | Existing [Git merge driver](https://github.com/Ataraxy-Labs/weave) and other nearby software uses |
| ContextTrail | Existing [agent project-history tool](https://pypi.org/project/contexttrail/) |
| TracePlane | Existing [agent continuity product](https://traceplane.cc/) and other software uses |
| EditHarbor | Emphasizes editing and safekeeping more than effortless team synchronization |
| ContextCourier, ContextPost, Dispatch Chronicle | The editorial/postal naming direction did not match the desired technical character |

The next decision is whether to use a Dispatch-family name or an independent brand; **Dispatch Mesh** and **IntentReplica** are the respective recommendations.
