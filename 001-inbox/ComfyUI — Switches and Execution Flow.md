---
type: reference
status: active
created: 2026-10-18
source: "https://github.com/yolain/ComfyUI-Easy-Use"
tags:
  - comfyui
  - image-generation
  - workflow
  - switches
  - lazy-evaluation
  - execution-flow
  - subgraphs
  - comfyui-easy-use
  - impact-pack
  - krea-2
---

> [!abstract] Overview  
> Switch nodes provide conditional routing within ComfyUI workflows. However, routing values and controlling execution are not necessarily the same operation.
> 
> A switch may select one value from several inputs, distribute a value to selected outputs, or influence which upstream nodes are required to execute.
> 
> Our Krea 2 workflow uses a **shared-index inverse-switch/index-switch architecture**, originally developed using Impact Pack nodes.
> 
> Following problems with the Impact switch implementations, we replaced those nodes with **Any Inversed Switch** and **Any Index Switch** from ComfyUI-Easy-Use, preserving the established architecture.
> 
> This article explains switch behaviour, lazy evaluation, execution blocking, dependency graphs, circular connections, and the design of our reusable three-route image-refinement pipeline.

## 1. Why Use Switches?

Our Krea 2 generation workflow eventually supported three alternative refinement paths:

1. Base image without additional refinement.
    
2. Automated Eye Detailer.
    
3. Manual Masked Detailer.
    

All three paths operate on image data, but only one result is normally required for a particular generation.

Without conditional routing, a workflow may perform unnecessary processing or require separate output nodes for different operations.

The objective was to provide a simple selector while preserving the independent refinement implementations.

Conceptually:

```
                    Base Image
                        |
           +------------+------------+
           |            |            |
           v            v            v
         Base       Eye Detailer   Masked Detailer
           |            |            |
           +------------+------------+
                        |
                        v
                  Output Selector
                        |
                        v
                    Save Image
```

This is the logical architecture, independent of the particular switch-node implementation.

## 2. ComfyUI Is a Dependency Graph

ComfyUI workflows are constructed from nodes connected by typed inputs and outputs.

At execution time, the system determines which nodes and dependencies are required to produce requested outputs.

A simple chain looks like:

```
A -> B -> C -> Save Image
```

For `Save Image` to receive its input, the preceding nodes must provide their results.

A branching workflow introduces additional possibilities:

```
          +-> B -> D
          |
          A
          |
          +-> C -> E
```

Whether both branches execute depends on the requested outputs, dependency relationships, caching, and any conditional or lazy execution mechanisms.

### Directed acyclic graphs

A conventional ComfyUI execution graph is expected to have a valid dependency order.

A directed cycle creates a problem:

```
A -> B -> C
^         |
|_________|
```

Here, A depends on C, while C ultimately depends on A.

The graph cannot be resolved through an ordinary dependency ordering.

> [!important] Fundamental principle  
> A visual connection is also a dependency relationship.
> 
> When constructing conditional routes, consider not only where data travels but also which nodes depend on which other nodes.

## 3. Routing Data vs Controlling Execution

A basic switch can select one of several available values.

However, this does not automatically establish that unselected upstream branches will be skipped.

There are two separate concerns.

### Data selection

Determines which value appears at the switch output.

### Execution control

Determines which inputs and upstream computations are required to produce that output.

An implementation may provide data selection without lazy evaluation.

Alternatively, a switch may be specifically designed to request only its selected input.

### Why this distinction matters

Consider three image-processing branches:

```
Base Image ------------------+
                             |
Eye Detailer ----------------+--> Switch --> Save Image
                             |
Masked Detailer -------------+
```

If every upstream branch must execute, selecting the base image may still trigger unnecessary refinement.

If the switch supports lazy evaluation, the selected input can be requested without automatically evaluating the other branches.

The implementation of the switch therefore matters as much as its visual appearance.

## 4. Lazy Evaluation

Lazy evaluation means postponing computation until its result is actually required.

In programming, a conditional expression may evaluate only the selected branch.

Conceptually:

```
if mode == 0:
    result = base_image()
elif mode == 1:
    result = eye_detailer()
else:
    result = masked_detailer()
```

This differs from calculating every result before selecting one.

```
base = base_image()
eyes = eye_detailer()
masked = masked_detailer()

result = [base, eyes, masked][mode]
```

The first example expresses conditional computation.

The second expresses selection from values that have already been computed.

ComfyUI switch nodes can exhibit different behaviour depending on how they interact with the execution engine.

### Lazy inputs

The inspected upstream implementation of ComfyUI-Easy-Use's `Any Index Switch` declares its selectable inputs as lazy.

It uses a `check_lazy_status` method to request the input corresponding to the selected index.

Conceptually, when `index = 1`, the node requests `value1`.

This is an implementation-level mechanism for selecting only the required input dependency.

### Limits of lazy execution

Lazy input selection does not mean the entire workflow executes only one node.

Shared prerequisites may still need to execute.

For example, our Eye Detailer and Masked Detailer both depend on the base generated image.

That shared image still needs to exist before either refinement can operate.

Caching and additional output nodes may also influence what executes.

> [!tip] Practical verification  
> Use execution highlighting, timing, and intermediate previews to confirm which nodes run.
> 
> Do not assume that a switch is lazy simply because only one output is visible.

## 5. Multiplexers and Demultiplexers

Our established design uses two complementary routing operations.

### Demultiplexer

A demultiplexer accepts one value and directs it to one of several outputs.

Conceptually:

```
                    +--> Output 0
                    |
Input --> DEMUX ----+--> Output 1
                    |
                    +--> Output 2
```

The selected index determines which output receives the value.

### Multiplexer

A multiplexer accepts several possible inputs and returns the selected one.

```
Input 0 --+
          |
Input 1 --+--> MUX --> Output
          |
Input 2 --+
```

The selected index determines which input is returned.

### Combining the two

Using a demultiplexer before alternative processing branches and a multiplexer afterward produces a conditional processing architecture.

```
                    +--> Processing A --+
                    |                   |
Input --> DEMUX ----+--> Processing B --+--> MUX --> Output
                    |                   |
                    +--> Processing C --+
```

Both routing nodes receive the same selection index.

This is the architecture we originally established using Impact Pack switches and subsequently retained when replacing them with ComfyUI-Easy-Use nodes.

## 6. Evolution of Our Routing Architecture

The routing implementation evolved through several stages.

### Stage 1 — Output selection

Our initial workflow used an Impact Switch to select between the base image and refinement results.

The switch was positioned after the processing branches.

This was a relatively simple way to expose one selected output.

### Stage 2 — Shared-index inverse/index routing

We subsequently developed a more structured routing implementation using complementary Impact Pack switches.

The architecture used:

- An inverse switch to distribute a value according to a selected index.
    
- A corresponding index switch to select the matching route.
    
- A shared integer control value supplied to both switches.
    

This established the demultiplexer/multiplexer architecture.

**The shared-index design was already part of the Impact Pack implementation.**

It was not introduced by ComfyUI-Easy-Use.

### Stage 3 — Impact Pack compatibility problems

Following ComfyUI updates, we encountered problems with Impact Pack switch behaviour.

These affected the reliability of our established routing implementation.

The problems were associated with the installed versions and observed workflow behaviour, rather than a demonstrated flaw in the shared-index architecture itself.

### Stage 4 — Targeted node replacement

We replaced the Impact Pack switches with:

- `Any Inversed Switch`
    
- `Any Index Switch`
    

From ComfyUI-Easy-Use.

The existing shared integer control was retained.

The surrounding routing architecture remained the same.

### Architectural significance

This was a **dependency replacement**, not a workflow redesign.

The switch implementations changed, while the logical routing structure and its external responsibilities were preserved.

> [!important] Lesson learned  
> When an implementation becomes unreliable, replacing the failing component does not necessarily require redesigning the architecture around it.
> 
> A well-defined interface makes targeted replacement possible.

## 7. The ComfyUI-Easy-Use Switch Nodes

The replacement nodes are provided by:

[https://github.com/yolain/ComfyUI-Easy-Use](https://github.com/yolain/ComfyUI-Easy-Use)

Our current implementation uses:

- `Any Inversed Switch`
    
- `Any Index Switch`
    

Both are generic routing nodes rather than image-specific switches.

### Any Inversed Switch

The node accepts an arbitrary input value and a numerical index.

It exposes outputs labelled:

`out0`, `out1`, `out2`, and so on.

The selected output receives the input value.

In the inspected upstream implementation, unselected outputs return an `ExecutionBlocker` rather than the input value.

This is significant because the node provides explicit execution-blocking results for unselected routes.

### Any Index Switch

The node accepts:

`value0`, `value1`, `value2`, and so on.

It also accepts an integer index.

The output is named:

`value`

In the inspected upstream implementation, the selectable inputs are declared lazy.

The node requests the input corresponding to the selected index.

### Indexing convention

Both nodes use zero-based indexing.

|Index|Inverse output|Index-switch input|
|---|---|---|
|0|`out0`|`value0`|
|1|`out1`|`value1`|
|2|`out2`|`value2`|

This correspondence is central to the paired architecture.

### Supported index range

In the inspected source, the index input is defined with a minimum of 0 and a maximum of 9.

Additional sockets may be visible beyond that selectable range.

For example, our screenshot shows outputs through `out19`.

The presence of a socket does not automatically mean it can be selected by the current index schema.

Actual permitted values should be checked against the installed node implementation.

## 8. Our Current Routing Subgraph

The screenshot from 18 October 2026 shows the current implementation using ComfyUI-Easy-Use.

Its external inputs are:

|Input|Meaning|
|---|---|
|`base_input`|Base image input|
|`detailer_input`|Automated Eye Detailer input|
|`inpaint_input`|Manual Masked Detailer input|
|`switch`|Shared route selection|

Its exposed outputs are:

|Output|Meaning|
|---|---|
|`detailer_output`|Detailer-related output|
|`inpaint_output`|Inpainting-related output|
|`base_output`|Base-image-related output|

The screenshot shows the shared selection value being supplied through an `Int` node to both switches.

### Logical architecture

```
                    switch
                       |
                     INT
                       |
              +--------+--------+
              |                 |
              v                 v
       Any Inversed         Any Index
          Switch              Switch
              |                 ^
              |                 |
        out0, out1, out2   value0, value1, value2
              |                 ^
              +---- routing ----+
```

The exact wiring includes the surrounding input/output connections and reroute points.

The significant feature is that both switches receive the same integer selection value.

### Why retain the shared index?

The shared index provides one authoritative control for the route.

It prevents the two routing operations from being independently configured to different selections.

For example:

`index = 1`

Selects `out1` on the inverse switch and `value1` on the index switch.

### What changed during migration?

The implementation of the two switch nodes.

### What did not change?

- The shared-index architecture.
    
- The routing responsibilities.
    
- The overall subgraph interface.
    
- The separation between generation, detailing, and routing.
    
- The intention to avoid unnecessary processing.
    

This is an example of preserving a working design while replacing a problematic dependency.

## 9. Understanding the Screenshot Correctly

The screenshot represents a reusable routing subgraph.

It exposes several inputs and outputs while encapsulating the switching implementation.

It should not be interpreted as the complete image-generation workflow.

The surrounding workflow determines how the routed values connect to the base image, Eye Detailer, Masked Detailer, and final outputs.

### Why this distinction matters

A subgraph interface exposes the responsibilities of a component.

It does not necessarily reveal every operation performed by its callers.

The routing subgraph can remain stable even if the parent workflow changes how images are generated or refined.

See [[ComfyUI — Subgraph Architecture]].

## 10. Impact Pack Regression and Troubleshooting History

Our original switching implementation used nodes from the Impact Pack ecosystem.

After a ComfyUI update, we observed unexpected behaviour.

### Initial symptoms

During our earlier investigation:

- A previously functioning normal Impact Switch stopped producing its expected output.
    
- The inverse switch continued to operate.
    
- Creating a new normal switch initially worked.
    
- Duplicating a working switch reproduced the failure.
    
- The surrounding sampling and VAE processing continued producing valid intermediate images.
    

The change coincided with ComfyUI Core v0.40.0.

At the time, Impact Pack had not received a corresponding recent update in our installation.

### What we established

The workflow failure occurred at the switch boundary.

The base generation and decoding stages were still functioning.

We did not establish the complete upstream root cause.

The relationship to the Core update was a compatibility hypothesis supported by the timing and symptoms.

### Separate workflow-loading issue

We subsequently encountered another Impact Pack compatibility problem that prevented some workflows from opening.

A locally applied patch from an open pull request addressed that separate issue.

However, the switch execution problem remained.

### Final response

Rather than continue modifying the established routing architecture, we replaced the problematic switch implementations with their ComfyUI-Easy-Use equivalents.

The shared-index design was retained.

### Lesson

A custom-node package can have multiple independent compatibility problems.

Fixing workflow loading does not necessarily fix execution behaviour.

It is important to distinguish:

- Workflow loading failures
    
- Node validation failures
    
- Runtime execution failures
    
- Incorrect output routing
    

These should be investigated separately.

## 11. Circular Dependencies and Routing Experiments

During development, we encountered a circular-connection error while experimenting with inverse and normal switch routing.

This was an earlier wiring or execution-dependency problem.

It should not be confused with the later shared-index architecture or the subsequent replacement of its switch implementations.

### Why cycles matter

A valid execution graph requires resolvable dependencies.

If one node depends on another that ultimately depends on the first, the graph becomes cyclic.

Conceptually:

```
A -> B -> C
^         |
|_________|
```

The execution engine cannot resolve this through ordinary dependency ordering.

### Important distinction

The earlier circular-connection failure did not establish that inverse/index switch architectures are inherently invalid.

The subsequent shared-index design was retained through the switch implementation migration.

### General lesson

When a workflow reports a cycle, investigate the actual dependency relationships.

Do not assume that the visual left-to-right layout accurately describes execution dependencies.

## 12. Execution Blocking

The ComfyUI-Easy-Use inverse switch uses an execution-blocking mechanism.

In the inspected implementation, unselected outputs return:

`ExecutionBlocker(None)`

### Conceptual behaviour

With `index = 1`:

```
                     out0 -> BLOCKED
                    /
Input -> Inverse --- out1 -> Input value
                    \
                     out2 -> BLOCKED
```

This means the node does not simply duplicate the input across all outputs.

Unselected outputs receive a special result intended to prevent downstream processing through those paths.

### Execution blocking vs lazy evaluation

These mechanisms are related but distinct.

**Execution blocking** prevents downstream processing from proceeding through a blocked output.

**Lazy evaluation** allows a consumer to request only the input it actually needs.

The Easy-Use pair uses both mechanisms.

### Important limitation

Execution blockers do not guarantee that unrelated nodes elsewhere in the workflow will be skipped.

Other requested outputs, shared dependencies, or execution-engine behaviour may still require additional processing.

The actual execution path should be verified through testing.

## 13. Comparing the Routing Implementations

|Implementation|Architecture|Our experience|
|---|---|---|
|Separate output paths|Independent branches|Simple but less convenient|
|Impact output switch|Output selection|Initially worked; later encountered regression|
|Impact shared-index inverse/index pair|Demultiplexer/multiplexer|Established routing architecture|
|Easy-Use shared-index inverse/index pair|Same demultiplexer/multiplexer architecture|Current replacement implementation|

The key distinction is between the **routing design** and the **nodes implementing it**.

The design survived the migration.

## 14. Why Generic Any-Type Switches Are Useful

The Easy-Use switches operate on generic values rather than being restricted to `IMAGE`.

This allows the same switching concepts to be applied to different ComfyUI data types.

Examples include:

- Images
    
- Latents
    
- Conditioning
    
- Models
    
- Other supported node values
    

### Architectural benefit

Routing logic can be encapsulated independently of the data being routed.

A reusable routing component does not necessarily need to know whether it is selecting an image, latent, or model.

### Trade-off

Generic routing provides flexibility but also places responsibility on the workflow designer.

Downstream nodes still expect compatible data types.

> [!warning] Type safety  
> An Any-type connection does not make incompatible values interchangeable.
> 
> All selectable branches must produce values appropriate for their consumers.

## 15. Testing the Routing Subgraph

A routing implementation should be tested independently before being relied upon in a larger generation workflow.

### Test 1 — Index mapping

Verify each route individually.

|Index|Intended selection|
|---|---|
|0|First configured route|
|1|Second configured route|
|2|Third configured route|

The exact mapping should be taken from the actual subgraph wiring rather than inferred from labels.

### Test 2 — Shared selection

Confirm that both switches receive the same integer value.

### Test 3 — Selected output

Verify that the selected route produces the intended result.

### Test 4 — Unselected branches

Observe whether unselected processing branches execute.

Use previews, execution indicators, or timing.

### Test 5 — Repeated runs

Test switching between modes without modifying other workflow parameters.

This helps reveal caching, execution-order, or stale-output behaviour.

### Test 6 — Subgraph boundaries

Verify the routing behaviour after encapsulation.

A component that works independently should also be tested through its parent workflow interface.

## 16. Troubleshooting Switch Problems

|Symptom|Investigation|
|---|---|
|Wrong output selected|Index mapping and socket ordering|
|No output|Selected socket, missing input, or execution blocker|
|Unexpected branch executes|Lazy dependency handling and independent outputs|
|Workflow reports a circular connection|Dependency cycle|
|Switch previously worked but now fails|Core and custom-node version changes|
|Only one saved workflow fails|Saved node state or serialization differences|
|Duplicated node behaves differently|Node metadata, version-specific behaviour, or serialized configuration|
|Generic output rejected downstream|Data-type incompatibility|
|Switch works outside subgraph but not inside|Subgraph dependency handling and interface wiring|

### Recommended diagnostic sequence

1. Identify whether the failure occurs during loading, validation, or execution.
    
2. Inspect the selected index.
    
3. Verify the corresponding input and output connections.
    
4. Preview intermediate data where possible.
    
5. Test a minimal reproduction.
    
6. Compare installed versions and upstream changes if the failure followed an update.
    
7. Preserve the known-good workflow before modifying its architecture.
    

Avoid replacing unrelated generation components when the evidence points specifically to routing.

## 17. Designing Replaceable Routing Components

Our routing migration demonstrates several useful software-engineering principles.

### Single responsibility

The routing subgraph controls routing.

It does not need to understand how the Eye Detailer performs segmentation or how the Masked Detailer performs inpainting.

### Shared control state

Both switches receive the same selection value.

This avoids inconsistent configuration.

### Encapsulation

The parent workflow interacts with a small set of meaningful inputs and outputs.

The internal switching implementation can change without requiring a redesign of the parent workflow.

### Replaceability

The transition from Impact Pack to ComfyUI-Easy-Use demonstrates dependency substitution.

The logical architecture remained intact.

Only the underlying switch nodes were replaced.

### Observability

Intermediate previews and execution indicators help determine whether failures originate in processing or routing.

This becomes especially useful when implementation details are hidden inside reusable subgraphs.

See [[ComfyUI — Subgraph Architecture]].

## 18. Key Lessons

### 1. Architecture and implementation are separate concerns

Our shared-index routing architecture was established using Impact Pack and preserved when the switch implementations were replaced.

### 2. Switching values and controlling execution are different responsibilities

A switch may provide selection without necessarily avoiding upstream computation.

### 3. Lazy evaluation is an implementation detail worth verifying

The Easy-Use index switch explicitly declares lazy inputs.

### 4. Execution blockers provide conditional downstream control

The Easy-Use inverse switch returns execution blockers on unselected outputs.

### 5. Zero-based indexing must remain consistent

The selected inverse output and corresponding index-switch input must use the same index.

### 6. Dependency graphs matter more than visual flow

A graph that looks sensible can still contain invalid dependencies.

### 7. Version-related regressions should be investigated precisely

An observed failure following a ComfyUI update does not automatically establish a universal incompatibility.

### 8. Replace failing dependencies before redesigning working architecture

Our migration preserved the established routing design while substituting the problematic switch nodes.

### 9. Subgraphs provide useful abstraction boundaries

Encapsulation made it possible to retain the same external responsibilities while changing the internal implementation.

---

## Related Notes

- [[ComfyUI — Understanding Latent Space]]
    
- [[ComfyUI — Masks and Compositing]]
    
- [[ComfyUI — Masks and Compositing]]
    
- [[ComfyUI — Automated Detailing]]
    
- [[ComfyUI — Subgraph Architecture]]
    
- [[Krea 2 — Prompt Engineering]]
    
- [[Krea 2 — Practical Workflow Architecture]]
    

## References

### ComfyUI-Easy-Use

Repository: [https://github.com/yolain/ComfyUI-Easy-Use](https://github.com/yolain/ComfyUI-Easy-Use)

Switch implementation: [https://github.com/yolain/ComfyUI-Easy-Use/blob/main/py/nodes/logic.py](https://github.com/yolain/ComfyUI-Easy-Use/blob/main/py/nodes/logic.py)

The upstream source inspected on 18 October 2026 defines `anythingInversedSwitch` and `anythingIndexSwitch`.

The inverse switch returns `ExecutionBlocker(None)` on unselected outputs.

The index switch declares lazy selectable inputs and requests the input corresponding to the chosen index.

These findings describe the inspected upstream source. They are not a substitute for checking the exact version installed locally if behaviour changes.

### ComfyUI Impact Pack

Repository: [https://github.com/ltdrdata/ComfyUI-Impact-Pack](https://github.com/ltdrdata/ComfyUI-Impact-Pack)

Referenced for the original switch implementation and associated troubleshooting history.

## Origin

Developed from practical Krea 2 workflow experiments in ComfyUI Desktop.

The workflow initially used Impact Pack output switching to select between the original generated image, automated eye refinement, and manual masked repair.

We subsequently established a **shared-index inverse-switch/index-switch routing architecture using Impact Pack nodes**.

Following update-associated problems affecting those switches, we replaced the two implementations with **Any Inversed Switch** and **Any Index Switch** from ComfyUI-Easy-Use.

The shared-index routing architecture, surrounding workflow responsibilities, and subgraph interface were preserved.

The 18 October 2026 screenshot documents the replacement nodes within the existing architecture.

The central architectural lesson is that **a reliable abstraction allows a failing implementation to be replaced without redesigning the system around it**.