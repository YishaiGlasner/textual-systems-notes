# Designing the Ref API: Navigating Sefaria's Textual Graph

In Sefaria, a `Ref` is the representation of a location in a textual structure. It can refer to a whole work, a part of a work, a section, a segment, a range, or other kinds of locations in our textual data.

Refs are also the basic mechanism for navigating Sefaria's texts. A client that wants to move through the library therefore needs to be able to understand a Ref and find its place in the textual structure. Without an API for this, an external client would need to implement much of Sefaria's Ref logic itself — and in practice, each client would end up implementing it slightly differently, producing inconsistencies between apps that are hard to trace back to a single cause.

For example, a client receiving `Beit Yosef, Orach Chayim 1` might want to understand where this Ref sits in the structure, find the first available section, or continue navigating from it. Similarly, given a Ref such as `Siddur Ashkenaz, Weekday, Shacharit, Amidah`, a client might want to know whether it represents a section containing segments, whether it has child Refs such as Avot and Gevurot, or how to navigate from it to Mincha. This was the motivation for the Ref API.

Sefaria already had a Python `Ref` class with many methods for working with these locations. The idea of a Ref was therefore not something I needed to design. My task was to use this existing abstraction to define a public API: what should a client be able to know about a Ref, and what should it be able to do with it?

I find it useful to think about a Ref as a vertex in a graph. The API should expose enough information about that vertex to let a client navigate the graph without having to load Sefaria's entire internal structure and reproduce its logic.

## What should a Ref API return?

The existing `Ref` class has many methods, but not everything that can be obtained while resolving a Ref should necessarily be part of its API.

For example, a Ref belongs to an Index, and the Index has categories. We can also find Versions that have a specific text for the Ref. Both pieces of information may be useful to a client, but they are not really properties of the Ref itself. The Index and Version APIs are better places for them.

This distinction is important because it would be easy to make the Ref API into a general-purpose endpoint that returns everything available while resolving a Ref. That would make the API harder to understand and would blur the boundaries between the different parts of Sefaria's data model.

So the question was not simply what information is available, but what information belongs in an API whose subject is a Ref.

## Different kinds of locations

One of the reasons this is not completely straightforward is that Refs can exist at different levels and point to different kinds of structures.

For example:

```text
Genesis
Genesis 1
Genesis 1:1
```

are all Refs, but they represent different levels of the same textual structure.

A `SchemaNode` is a node used to organize a work's structure. It can contain titled parts of a work that are not themselves simply numbered text, such as an introduction or a particular named section. `Beit Yosef, Orach Chayim` sits at this level: it is a named part of the structure, not yet a numbered section.

A `JaggedArrayNode` is a node containing text organized into a fixed number of nested levels. For example, a biblical book can have chapters, verses, and segments, giving it a depth of three.

Other Refs point to structures that behave differently. A Ref can point to a dictionary entry, or to a sheet.

The API therefore has a common set of information for Refs in general, and additional information depending on the kind of structure being referenced.

For a `SchemaNode`, for example, the API returns its direct children.

For a `JaggedArrayNode`, it returns information about the depth of the structure, its address types and section names, and the sections represented by the Ref.

For a dictionary entry, it returns information about the lexicon and headword.

This makes the response reflect the structure that the Ref points to rather than forcing every kind of Ref into the same representation.

## Navigating a Ref

Since the main purpose of the API is navigation, the response includes several ways to understand where a Ref is in its structure.

First, it returns the shortest path from the Ref to the root. This lets a client understand the hierarchy without having to load the entire Index tree.

It also returns the direct children of a Ref when it represents a node containing other nodes.

Another useful case is a Ref that represents a large or structural part of a work, where the client wants to start reading actual text. For these Refs, the API returns a `first_available_section_ref`: the first section containing available text from which the client can start reading.

### What should "next" and "previous" mean?

The question becomes more complicated when we try to define next and previous navigation.

For a segment, the meaning is clear:

```text
Genesis 1:1 → Genesis 1:2
```

A segment has a next and previous segment in the reading order.

The same is true for a section: the client can ask for the next or previous section.

The problem appears above the section level.

Consider:

```text
Ramban on Genesis 1
```

The Genesis part of the Ramban is a three-level structure: an array of sections, with segments inside them. The node before it in the schema may be an introduction, but that introduction is at a lower depth.

If we return the introduction as the previous Ref, the result is surprising: if the client asks for the previous Ref of a depth-three location, it receives a depth-two Ref. But if we return `null`, the client may reasonably interpret that as meaning that this is the first Ref in the whole structure.

So I chose to define next and previous only where the meaning is unambiguous.

A segment-level Ref gets:

```text
prev_segment_ref
next_segment_ref
```

and a section-level Ref gets:

```text
prev_section_ref
next_section_ref
```

Refs above the section level do not get next/previous navigation.

An important property of this definition is that the navigation can cross structural nodes. The next section or segment does not have to belong to the same node as the current one. This is useful for clients that want to let a reader simply continue reading, without needing to understand where one internal node ends and another begins.

For a higher-level Ref, `first_available_section_ref` provides a different way into the readable structure.

## Commentary: when a Link looks like navigation

There is another kind of edge in the textual graph.

The structural relationships discussed so far—such as parent and child, ancestor and descendant, and next and previous—are part of the structure represented by Refs. `Link`s also connect two Ref vertices, but they represent external relationships between them. They are therefore a different kind of edge. Classifying what kind of external relationship a given Link represents is itself a hard question, one I explored separately in [Beyond Links: Towards a Taxonomy of Relationships Between Texts](classification-of-connections-between-texts.md). The question here is a different, prior one: whether a relationship belongs to a Ref's own structure at all, or is external to it and therefore a Link.

Commentary raises an interesting question about this distinction.

In Sefaria's data model, commentary relationships are represented by `Link`s. But conceptually, commentary can look very different from many other Links.

For example:

```text
Rashi on Genesis 1:1
Genesis 1:1
```

A user may naturally want to move between these two locations as part of navigating the text. In a physical book, commentary is often printed alongside the base text, so this kind of movement can feel like navigation within the same reading structure rather than following a related reference.

This suggests that a Ref API could potentially expose a commentary relationship as something like:

```json
{
  "base_ref": "Genesis 1:1"
}
```

There is a complication, however. The current data model does not directly store the exact base Ref as a property of the commentary Ref. We can know at the Index level that a work is a commentary, but the exact relationship between a particular commentary Ref and a particular base Ref has to be recovered from Links.

So adding this to the API would require additional work to obtain the information in a reliable way. It is not a matter of simply exposing another property of the Python `Ref` object.

I did not include this in the initial API, but I think it is a worthwhile direction for future work: commentary is currently represented as a Link, but it has some of the characteristics of structural navigation. Resolving it well would mean asking the same scoping question that shaped the rest of this API: not simply whether a base ref can be recovered, but whether it belongs on the Ref at all, or whether commentary deserves a navigational vocabulary of its own.

## Summary

The final API is deliberately centered on navigation rather than on exposing everything that can be learned about a Ref.

Given a Ref string, it determines whether it is valid and, if it is, provides the information needed to understand its location: its normalized forms, its position in the structural hierarchy, its node-specific structure, its children where applicable, and navigation to the next readable section or segment.

The endpoint is therefore not simply an HTTP wrapper around the Python `Ref` class. The class provided the underlying capabilities; the API defines which of those capabilities form a useful public interface.

An API like this is not a transcription of what its underlying class can do. It is a decision about what a caller is allowed to know — and, just as importantly, a record of what was deliberately left out, like commentary navigation, until it can be done properly.
