# Short-circuiting nodes

## Introduction

In our field....

oneTBB already supports messages whose types are nullable.

Short-circuiting is desired for two reasons:

1. A node has more than one input, and receiving one null message is sufficient to emit a null output message.
2. A node has one input, but if the input message is null, the function body need not be invoked at all, and a null output message is emitted.

We suggest creating new node types/templates that can support short-circuiting behavior, when receiving nullable messages.
Nodes that would be candidates for this include:

- `async_node`
- `composite_node`
- `function_node`
- `join_node`
- `limiter_node`
- `multifunction_node`

We are not arguing to change the behavior of existing types....

Some nodes are not candidates for short-circuiting behavior:

1. The `input_node` receives no input, so it cannot react to a nullable message.

2. Most nodes without a user-supplied function body are not candidates because short-circuiting refers to skipping the execution of the user-supplied body when a message is null.
   Exceptions to this are the `join_node` and `split_node`, for which analogous nodes of short-circuiting behavior would be useful (see below).

3. For a `join_node` using the key-matching policy, when one input port receives a null message, the joined message may be emitted without waiting for the key-matched messages on the other input ports.
   The output of a short-circuiting join node would be a nullable tuple (e.g.): `join(nullable<int>, nullable<double>) -> nullable<tuple<int, double>>`.
   We do not currently see a use for short-circuiting behavior for a `join_node` that uses a policy other than key-matching, but perhaps others will.
   We also see no reason for a short-circuiting `join_node` to limited to a key-matching policy.

4. Note that the complement to the `join_node` is the `split_node`.
   Because the `split_node` does not satisfy the short-circuiting conditions above, it is not suitable for short-circuiting.

### Discussion regarding input node

- `input_node` (perhaps supports a `std::variant<tbb::flow::stop, std::optional<T>>`)

### Discussion regarding composite nodes

We imagine composite nodes can usefully provide short-circuiting behavior.
If a composite node's external input ports receive null messages, the encapsulated nodes need not execute, but a null message could be directly emitted from the composite node's external output ports.

- [Provide two concrete examples based on Omnigraffle diagrams]

### Discussion regarding limiter node

Two use cases:

- In one mode, don't ratchet up the count if a null message is received but just forward it (this is useful if the output is small for the null input)
- In the other mode, discard a null message before the counter is even reached (this is useful if the output is large for the null input)

> Short description of the idea proposed with explained motivation.
>
> The motivation could be:
> - Improved users experience for API changes and extensions. Code snippets to
>   showcase the benefits would be nice here.
> - Performance improvements with the data, if available.
> - Improved engineering practices.
>
> Introduction may also include any additional information that sheds light on
> the proposal, such as history of the matter, links to relevant issues and
> discussions, etc.

## Proposal

> A full and detailed description of the proposal with highlighted consequences.
>
> Depending on the kind of the proposal, the description should cover:
>
> - New use cases supported by the extension.
> - The expected performance benefit for a modification.
> - The interface of extensions including class definitions or function
> declarations.
>
> A proposal should clearly outline the alternatives that were considered,
> along with their pros and cons. Each alternative should be clearly separated
> to make discussions easier to follow.
>
> Pay close attention to the following aspects of the library:
> - API and ABI backward compatibility. The library follows semantic versioning
>   so if any of those interfaces are to be broken, the RFC needs to state that
>   explicitly.
> - Performance implications, as performance is one of the main goals of the library.
> - Changes to the build system. While the library's primary building system is
>   CMake, there are some frameworks that may build the library directly from the sources.
> - Dependencies and support matrix: does the proposal bring any new
>   dependencies or affect the supported configurations?
>
> Some other common subsections here are:
> - Discussion: some people like to list all the options first (as separate
>   subsections), and then have a dedicated section with the discussion.
> - List of the proposed API and examples of its usage.
> - Testing aspects.
> - Short explanation and links to the related sub-proposals, if any. Such
>   sub-proposals could be organized as separate standalone RFCs, but this is
>   not mandatory. If the change is insignificant or doesn't make any sense
>   without the original proposal, you can have it in the RFC.
> - Execution plan (next steps), if approved.

## Open Questions

> For new proposals (i.e., those in the `rfcs/proposed` directory), list any
> open questions.
