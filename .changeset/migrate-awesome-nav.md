---
"authifi-docs": patch
---

Switch navigation to mkdocs-awesome-nav (no nav content change). Navigation is now derived from the file tree plus `docs/.nav.yml` instead of the `nav:` block in `mkdocs.yml`, so future synced docs auto-append to the correct section. The rendered nav and sitemap are unchanged.
