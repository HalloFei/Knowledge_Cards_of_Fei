---
name: feedback_term_teacher
description: Term Teacher Agent should create files directly in /Terms directory
type: feedback
---

When using Term Teacher Agent to create knowledge cards for terms, create the corresponding .md file directly in the current directory (under the appropriate subfolder: Method/, Concept/, or Tool/), rather than just outputting the content in the response.

**Why:** The user wants terms to be immediately archived in the knowledge base, not just displayed. The output should be persisted as a file so it can be referenced later.

**How to apply:** After generating the JSON card and Markdown content, use write_file to create the file at the storage_path (e.g., `Method/some_term.md`) with the Markdown content. The JSON card metadata can be included as frontmatter or at the top of the file.