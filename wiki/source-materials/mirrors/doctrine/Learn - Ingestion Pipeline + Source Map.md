# Learn – Ingestion Pipeline + Source Map

## Inputs (User Provided)
- PDF (book pages, worksheets, handouts)
- Photos/scans of pages
- Typed notes
- Optional transcription text (manual upload)

## Pipeline
1. Normalize content into chunks (page/section/paragraph blocks)
2. Tag chunks with metadata:
   - doc_id, page, section headers
   - topic tags (lightweight)
3. Build local index for retrieval (on-device)
4. Maintain "Source Map":
   - concept_tag -> {doc_id, anchors[]}

## Retrieval Anchors
Anchors are learned "where-to-look" pointers:
- concept_tag -> top passages + offsets
- frequently used definitions + locations

If student finds the answer first during source browsing:
- promote that location into anchor list
- increase weight for future retrieval

Goal:
Make SA faster at finding grounded answers over time without changing Templates.