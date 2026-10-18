---
type: reference
status: active
created: 2026-10-17
source:
tags:
  - obsidian
  - knowledge-management
  - metadata
---
# Article Metadata Conventions
## Purpose

Establish a lightweight, consistent approach to classifying and organizing articles in the knowledge base.

The objective is to make information easy to capture, discover and maintain without introducing unnecessary administrative overhead.

**Core principle: Tags describe subjects. Properties describe the document itself.**

## Metadata structure

Articles can use YAML frontmatter at the beginning of their Markdown files.

```
---
type: reference
status: active
tags:
  - krea2
  - lighting
  - cinematic
  - image-generation
  - photography
---
```

Obsidian recognizes these properties and provides an interface for editing them.

The metadata remains readable and editable outside Obsidian.

## Document type

The `type` property describes the purpose of an article.

|Type|Purpose|
|---|---|
|`reference`|Reusable information, explanations and technical knowledge|
|`how-to`|Instructions for completing a task|
|`troubleshooting`|Investigation, diagnosis and resolution of problems|
|`decision`|Architectural or technical decisions and their rationale|
|`idea`|Thoughts, proposals and concepts not yet developed|

These values are initial conventions, not an enforced schema. Additional types should be introduced only when there is a demonstrated need.

## Document status

The `status` property describes the article's lifecycle.

|Status|Meaning|
|---|---|
|`draft`|Incomplete or awaiting further development|
|`active`|Current and usable|
|`archived`|Retained for historical reference but no longer current|

Status describes the document, not the operational state of the system or project it discusses.

## Subject tags

Tags describe the subjects covered by an article.

Examples:

- `krea2`
    
- `lighting`
    
- `cinematic`
    
- `image-generation`
    
- `photography`
    
- `docker`
    
- `raspberry-pi`
    
- `ollama`
    

Guidelines:

- Use lowercase tags.
    
- Use hyphens for multiword subjects.
    
- Prefer existing tags where appropriate.
    
- Avoid synonyms and duplicate spellings.
    
- Add only tags that are useful for discovery.
    
- Do not use tags to represent document type or lifecycle status.
    
- Avoid hierarchical tags until a genuine need emerges.
    

Tags are optional for quick-capture notes and should not become an obstacle to recording information.

## Folders, tags and links

These serve different purposes.

**Folders — Where does the article live?**

Use broad organizational categories such as `projects/`, `reference/` and `00-inbox/`.

**Tags — What is the article about?**

Use subject-based metadata to support discovery across folder boundaries.

**Links — How does the article relate to other knowledge?**

Use Obsidian internal links, such as `[[Charon - Obsidian Setup]]`, to connect related documents.

An article should not need to be duplicated across folders simply because it covers several subjects.

## Capture workflow

Immediate capture takes priority over classification.

1. Capture thoughts and information in `00-inbox`.
    
2. Add a meaningful title where practical.
    
3. Do not require metadata during initial capture.
    
4. Review and classify articles later.
    
5. Move articles into an appropriate location when their long-term purpose becomes clear.
    
6. Add links to related knowledge where useful.
    

## Future agent integration

Automated knowledge-processing agents may eventually propose:

- Document type and status
    
- Relevant subject tags
    
- Related existing articles
    
- Appropriate destination folders
    
- Duplicate or overlapping content
    

Agent-generated classifications should initially be reviewable before publication.

The metadata convention should remain independent of any particular AI model or agent framework.

## Maintenance

Review these conventions as the knowledge base grows.

Avoid expanding the metadata schema simply because additional properties are technically possible.

**The success criterion is improved retrieval with minimal effort, not perfectly classified documentation.**