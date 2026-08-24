---
title: Learn - Ingestion Pipeline + Source Map
type: concept
tags: ["EVO","doctrine"]
sources:
  - source-materials/mirrors/doctrine/Learn - Ingestion Pipeline + Source Map.md
updated: 2026-07-24
---

# Learn - Ingestion Pipeline + Source Map
[Learn – Ingestion Pipeline + Source Map](https://www.notion.so/33ec72bad0138149a414d235925ee59c)
Inputs (User Provided)
PDF (book pages, worksheets, handouts)
Photos/scans of pages
Typed notes
Optional transcription text (manual upload)
Pipeline
Normalize content into chunks (page/section/paragraph blocks)
Tag chunks with metadata:
doc_id, page, section headers
topic tags (lightweight)
Build local index for retrieval (on-device)
Maintain “Source Map”:
concept_tag -> {doc_id, anchors[]}
Retrieval Anchors
Anchors are learned “where-to-look” pointers: - concept_tag -> top passages + offsets - frequently used definitions + locations
If student finds the answer first during source browsing: - promote that location into anchor list - increase weight for future retrieval
Goal: Make SA faster at finding grounded answers over time without changing Templates.

## Related

^[source-materials/mirrors/doctrine/Learn - Ingestion Pipeline + Source Map.md]
