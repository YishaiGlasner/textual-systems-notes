# Beyond Links: Towards a Taxonomy of Relationships Between Texts

While working on Sefaria, I found myself thinking about a deceptively simple question:

**What kind of relationship exists between two texts?**

At first glance, this seems like an easy question. Software systems that work with texts often represent relationships through links: one text points to another. But once we look more carefully, the word "link" hides many very different possibilities.

A quotation from another text is a link. A mention of another text is a link. A commentary on another text is a link. Two texts that independently preserve the same tradition are also related, even though neither one points to the other.

Are these really the same kind of relationship?

At Sefaria, relationships between texts are represented by `Link` objects connecting two references. Links may also contain a `type` field. Some values, such as `commentary`, are already used as conventions throughout the library. However, because the field is intentionally unrestricted, it does not provide a complete or consistent taxonomy of textual relationships.

A more precise classification could be useful for many purposes: better search, navigation, visualization, scholarly analysis, and automatic discovery of new relationships.

But before discussing possible applications, we need to ask a more basic question:

**What are the different kinds of relationships between texts?**

---

The first distinction that naturally comes to mind is the distinction between **verbal** and **thematic** relationships.

There is a significant difference between quoting a passage from *The Divine Comedy*, mentioning it without quoting it, echoing its language with changes, and simply discussing the same subject.

Likewise, two texts may be connected because they use similar words, or because they engage with the same ideas while using completely different language.

This distinction is clearly important.

However, after considering more examples, it becomes clear that it is not the first question we should ask.

Before asking *what kind of similarity exists*, we should ask two more fundamental questions:

1. Is the relationship dependent?
2. Is the relationship direct?

---

## Dependent and independent relationships

Many relationships between texts exist because one author knew another text.

But this is not always the case.

Leibniz and Newton developed calculus independently.

In this example, the similarity is not a result of one mathematician influencing the other.

The same phenomenon appears in textual traditions.

Even verbal similarity does not necessarily prove dependence. In rabbinic literature, parallel passages in the Babylonian and Jerusalem Talmuds may preserve similar wording because both traditions inherited earlier material, not because one text borrowed from the other.

This suggests a fundamental distinction:

- **Dependent relationship** — one text is related to another text as a result of literary awareness.
- **Independent relationship** — two texts are related without direct literary dependence.

Only after making this distinction does the verbal/thematic distinction become useful for independent relationships:

- **Verbal parallel** — similar wording or expressions without clear dependence.
- **Thematic parallel** — similar ideas, narratives, or motifs without clear dependence.

---

## Direct and indirect relationships

Among dependent relationships, another important distinction appears:

Is the relationship direct?

A later author may refer to an earlier text explicitly, but they may also engage with it indirectly.

### Indirect relationships

The clearest example is **allusion**: a later text echoes an earlier one without explicitly identifying it.

However, indirect relationships do not need to rely on shared language. A later text may engage with another text's ideas—supporting, challenging, or interpreting them—without any recognizable verbal echo.

I will call this **dialogue**.

At this point, further classification becomes increasingly uncertain. Identifying an allusion already involves interpretation; determining the exact purpose of the author involves even more speculation.

Therefore, I think a taxonomy of textual relationships should focus on observable relationships between texts, rather than attempting to classify authorial intentions.

---

## Direct relationships

Direct relationships are different.

Here the connection is not inferred: the text itself provides evidence that one work refers to another.

Because of this, the question of verbal versus thematic similarity becomes less central. The relationship does not exist because the two texts happen to resemble each other; it exists because one text explicitly engages with the other.

The important question becomes:

**How is the direct relationship established?**

Several categories emerge.

---

## Structural relationships

Some texts are structurally organized around another work.

### Commentary

A commentary follows another text and explains, interprets, expands, or discusses it.

This includes traditional commentaries, self-commentaries, critical commentaries (*hasagot*), and similar works.

One might suggest dividing commentaries further into explanatory, grammatical, philosophical, critical, and other types. However, I do not include such distinctions here because they classify the nature of the commentary itself, rather than the relationship between the commentary and its source text.

The question here is not:

> What kind of commentary is this?

but:

> What kind of relationship exists between these two texts?

### Super-commentary

A commentary on another commentary.

### Paraphrase

A text that systematically rewrites another text, such as certain expansive translations or Midrashic works where each verse receives a retelling or expansion.

### Essay

A work that discusses another work without following it continuously.

For example, a scholarly work analyzing Shakespeare's works or a study of a specific literary text.

Translation is another structural relationship. However, many systems—including Sefaria—often represent translations as versions of the same text rather than as separate links.

---

## Referential relationships

Other direct relationships are established through explicit reference.

### Citation

Another text is explicitly identified but not quoted.

### Quotation

Words from another text are reproduced verbatim or nearly verbatim, whether or not accompanied by a full citation.

---

This produces a first approximation of the taxonomy:

```text
Textual Relationships

Dependent
├── Direct
│   ├── Structural
│   │   ├── Commentary
│   │   ├── Super-commentary
│   │   ├── Essay
│   │   ├── Paraphrase
│   │   └── Translation
│   └── Referential
│       ├── Citation
│       └── Quotation
└── Indirect
    ├── Allusion
    └── Dialogue

Independent
├── Verbal Parallel
└── Thematic Parallel
```

This taxonomy is not intended to be definitive. Some categories may be merged, others subdivided, and borderline cases will always exist.

The goal is to identify meaningful distinctions between relationships, rather than to provide a perfect classification of every possible case.

---

## From taxonomy to practical use

A good taxonomy should first represent meaningful conceptual distinctions.

However, practical considerations also matter. A classification that is theoretically elegant but impossible to apply consistently has limited value.

One interesting consequence of this taxonomy is that many of these distinctions correspond to observable features.

Direct relationships can often be identified with reasonable effort once texts and their structures are properly modeled. Determining dependence may require historical knowledge, especially regarding chronology, but this is often a property of pairs of works rather than individual passages.

For independent relationships, verbal parallels can be approached through textual comparison methods, while thematic parallels require semantic analysis.

The important point is that these practical possibilities emerge from the conceptual model; they are not the reason for choosing the categories.

A good representation creates possibilities.

Instead of asking only whether two texts are connected, we can ask:

**How are they connected?**

Once that information becomes part of the data itself rather than hidden in an unrestricted label or rediscovered separately by every application, richer forms of exploration become possible.

Perhaps the first step toward building better software for texts is recognizing that not every connection means the same thing.