---
name: max-finder
description: AI-powered research discovery and synthesis engine for academic and market research
version: 2.4.1
author: Max Finder Team
tags:
  - research
  - ai
  - automation
  - literature-review
  - data-extraction
dependencies:
  - python>=3.9
  - openai>=4.0.0
  - anthropic>=0.8.0
  - beautifulsoup4>=4.12.0
  - requests>=2.31.0
  - pymupdf>=1.23.0
  - pandas>=2.0.0
  - sentence-transformers>=2.2.0
  - chromadb>=0.4.0
  - arxiv>=1.4.0
  - scholarly>=1.7.0
  - crossrefapi>=3.0.0
  - pyzotero>=1.5.0
  - Bio>=1.78.0 (for biomedical research)
  - googlesearch-python>=1.2.0
required_env_vars:
  - OPENAI_API_KEY or ANTHROPIC_API_KEY
  - MAX_FINDER_MODEL (default: gpt-4-turbo-preview)
  - MAX_FINDER_EMBEDDING_MODEL (default: all-MiniLM-L6-v2)
  - MAX_FINDER_CHROMA_PATH (default: ~/.maxfinder/chroma_db)
  - MAX_FINDER_CACHE_TTL (default: 86400)
  - MAX_FINDER_MAX_TOKENS (default: 4000)
  - MAX_FINDER_TEMPERATURE (default: 0.3)
  - MAX_FINDER_SEARCH_LIMIT (default: 50)
  - MAX_FINDER_CITATION_STYLE (default: apa)
---

# Max Finder Skill

## Purpose

Max Finder is an AI-powered research discovery and synthesis engine designed to automate and enhance academic and market research workflows. It combines multiple search APIs, semantic embedding search, and LLM synthesis to find, analyze, and organize research materials at scale.

### Real Use Cases

1. **Systematic Literature Reviews**: Automatically search arXiv, PubMed, Google Scholar, and CrossRef for papers matching specific criteria, extract key findings, methodologies, and generate summarized tables.

2. **Competitive Intelligence Monitoring**: Set up daily monitoring of news sources, tech blogs, and patents for specific technologies or companies, with AI-generated trend analysis.

3. **Grant Proposal Research**: Rapidly identify relevant prior work, conflicting studies, and research gaps for grant applications with citation-ready summaries.

4. **Technology Landscape Mapping**: Build comprehensive maps of technology stacks, frameworks, or methodologies by analyzing papers, documentation, and forum discussions.

5. **Expert Finding**: Identify and rank researchers, institutions, or companies working on specific topics using embedding similarity across publication histories.

6. **Research Gap Detection**: Analyze existing literature to identify under-explored areas, contradictory findings, or methodological limitations.

7. **Multi-Modal Research Synthesis**: Extract and correlate information from papers, presentations, datasets, and code repositories into unified knowledge graphs.

8. **Automated Meta-Analysis**: Aggregate statistical findings across multiple studies to compute effect sizes and identify patterns.

## Scope

### Available Commands

```
maxfinder search <query> [--source=<source>] [--date-range=<start>:<end>] [--limit=<n>] [--expand]
maxfinder search-papers <query> [--database=<db>] [--cited-by=<paper_id>] [--citing-papers] [--min-citations=<n>]
maxfinder analyze <paper_id|url> [--extract=<sections>] [--summarize] [--compare-with=<other_ids>]
maxfinder synthesize <query> [--sources=<list>] [--output=<format>] [--citation-style=<style>]
maxfinder embeddings build <name> [--input=<file_or_dir>] [--chunk-size=<n>] [--overlap=<n>]
maxfinder embeddings search <name> <query> [--top-k=<n>] [--threshold=<float>]
maxfinder monitor <query> [--sources=<list>] [--schedule=<cron>] [--alert-email=<email>] [--webhook=<url>]
maxfinder export <query_id> [--format=<json|bib|csv|md>] [--include-embeddings] [--deduplicate]
maxfinder zotero sync [--collection=<path>] [--library-id=<id>] [--api-key=<key>]
maxfinder verify-facts <claim> [--evidence-level=<strict|moderate|lenient>] [--max-sources=<n>]
maxfinder crossref-metadata <doi> [--include=<fields>]
maxfinder scholar-profile <author_query> [--include=<metrics|publications|citations>]
maxfinder health-check
maxfinder cache clear [--older-than=<days>] [--type=<papers|embeddings|results>]
maxfinder config set <key> <value>
maxfinder config get <key>
maxfinder config list
```

### Supported Sources

- Academic: arXiv, PubMed, IEEE Xplore, ACM Digital Library, SpringerLink, Wiley, ScienceDirect,bioRxiv, medRxiv
- Preprint: arXiv, bioRxiv, medRxiv, SSRN
- Gray Literature: Google Scholar, Semantic Scholar, Microsoft Academic (via API), CORE
- Patents: Google Patents, USPTO, EPO
- News/Media: Google News, NewsAPI, RSS feeds
- Code: GitHub, GitLab, arXiv源码链接
- Datasets: Kaggle, HuggingFace, Zenodo, Figshare
- Theses: ProQuest, institutional repositories
- Clinical Trials: ClinicalTrials.gov, EU Clinical Trials Register

## Detailed Work Process

### Phase 1: Query Understanding and Expansion
1. Parse user query using NER to extract:
   - Technical terms, acronyms, and domain-specific vocabulary
   - Named entities (authors, institutions, locations, organizations)
   - Temporal constraints (years, periods)
   - Methodological keywords (qualitative, quantitative, randomized, etc.)
   - Relationship types (comparison, causation, correlation, survey)

2. Generate query variants:
   - Synonym expansion using WordNet and domain-specific thesauri
   - Acronym expansion and definition
   - Boolean query formulation with optimized operators
   - Field-specific query patterns (e.g., "title:...", "abstract:...", "author:...")

3. Classify query type:
   - Exploratory (broad, undefined scope)
   - Targeted (known papers/authors)
   - Comparative (A vs B)
   - Systematic (strict inclusion/exclusion)
   - Trend analysis (temporal)

4. Estimate search scope and set appropriate limits:
   - Calculate expected result count from each source
   - Adjust database-specific parameters
   - Set timeout and pagination strategy

### Phase 2: Multi-Source Discovery

Execute parallel searches across selected sources with source-specific optimizations:

**arXiv Search**:
- Use arXiv API with category filtering
- Parse LaTeX sources when available
- Extract: title, authors, abstract, subjects, DOI, PDF URL, version history

**PubMed/MEDLINE**:
- Use E-utilities API
- Apply MeSH term expansion
- Extract: PMID, PMCID, DOI, abstract, publication types, grant info

**Google Scholar**:
- Use scholarly library with proxy rotation
- Handle CAPTCHA detection and backoff
- Extract: citations, year, venue, author affiliations

**CrossRef**:
- Query by DOI, ISSN, or metadata
- Extract: references, funders, licenses, container-title

**Semantic Scholar**:
- Use API for embedding vectors when available
- Extract: influential citations, key phrases, TLDR

For each result:
- Deduplicate by DOI, title similarity (Jaccard > 0.85), or author+year combination
- Normalize metadata fields to internal schema
- Store raw response for audit trail
- Assign confidence scores based on source reliability and metadata completeness

### Phase 3: Content Retrieval and Parsing

For each unique paper:

1. **Priority-based retrieval**:
   - First: Open Access PDFs from publisher (if CORS allows)
   - Second: arXiv/ institutional repository PDFs
   - Third: Unpaywall/alternative sources
   - Fourth: HTML abstracts with CSS selectors
   - Fifth: Metadata-only fallback

2. **PDF parsing**:
   - Use PyMuPDF for text extraction with layout preservation
   - Detect and preserve section headings (h1, h2 patterns)
   - Extract figures and tables as base64 images (store separately)
   - Parse references section into structured list
   - Identify supplementary materials

3. **HTML parsing**:
   - Remove navigation, ads, sidebars
   - Extract from common article templates (Springer, Elsevier, IEEE, etc.)
   - Reconstruct reference list

4. **Text preprocessing**:
   - Normalize whitespace, Unicode
   - Remove page numbers, headers/footers
   - Detect and label sections: Abstract, Introduction, Methods, Results, Discussion, Conclusion, References
   - Split into semantic chunks (max 1000 tokens) with 200 token overlap using sliding window
   - Preserve chunk metadata (section, page, source position)

### Phase 4: Embedding and Indexing

1. **Generate embeddings** for each chunk:
   - Use sentence-transformers model (configurable)
   - Batch process (max 32 chunks per batch)
   - Normalize vectors for cosine similarity
   - Cache embeddings by content hash to avoid recomputation

2. **Store in ChromaDB**:
   - Collection name: `<query_hash>_<timestamp>`
   - Metadata per chunk: {paper_id, chunk_id, section, page, source_file, timestamp}
   - Persist to disk at `~/.maxfinder/chroma_db/`

3. **Build document index**:
   - Store paper-level metadata in local SQLite database `~/.maxfinder/papers.db`
   - Create full-text search index with FTS5
   - Maintain mapping: embedded chunk → paper → full metadata

### Phase 5: Semantic Search and Synthesis

1. **Hybrid search**:
   - Vector similarity search (top 50 chunks)
   - Keyword BM25 search (top 50 results)
   - Combine with Reciprocal Rank Fusion (RRF)
   - Filter by metadata (date range, source type, author)

2. **Context assembly**:
   - Select top 10-15 most relevant chunks
   - Sort by relevancy score descending
   - Deduplicate by paper (max 2 chunks per paper)
   - Ensure diversity: at least 3 different sources
   - Add paper-level abstracts for context

3. **LLM synthesis**:
   ```
   System: You are a research synthesis AI. Analyze provided excerpts and produce objective, evidence-based synthesis.
   Constraints:
   - Extract specific claims with supporting evidence
   - Note contradictory findings
   - Identify research gaps
   - Cite using [N] format referencing the provided context list
   - Distinguish between findings, opinions, and hypotheses
   - Report statistical significance when present
   - Do not hallucinate beyond provided context
   - If insufficient context, state explicitly
   
   User: {query}
   Context:
   [1] Paper: {title} ({year}) - {authors}
   Content: {chunk_text}
   
   [2] ...
   
   Provide synthesis in markdown with sections:
   ## Overview
   ## Key Findings
   ## Methodologies
   ## Contradictions/Agreements
   ## Gaps & Future Directions
   ## References
   ```

4. **Generate structured output**:
   - Markdown report with citations
   - Bibliography in configured style (APA, MLA, Chicago, Harvard, Vancouver, IEEE)
   - Summary table: paper, main claim, methodology, sample size, key limitations
   - JSON export for downstream processing

### Phase 6: Quality Assurance and Verification

1. **Fact-check claims**:
   - Verify each synthesized claim against original source chunks
   - Ensure no misattribution or out-of-context quoting
   - Detect LLM hallucinations by checking for entity consistency

2. **Source validation**:
   - Check for predatory journal indicators
   - Verify author affiliations (if available)
   - Cross-check citation counts from multiple sources

3. **Bias detection**:
   - Analyze author demographics (if inferable)
   - Check publication date distribution (avoid outdated evidence)
   - Identify funding sources and potential conflicts

4. **Completeness check**:
   - Ensure at least 3 independent sources cited
   - Verify coverage of query dimensions
   - Flag results requiring human review

### Phase 7: Output and Integration

1. Generate final markdown report with TOC
2. Create BibTeX/ RIS file for reference managers
3. Export to Obsidian/Notion/Roam format if requested
4. Push to Zotero collection if configured
5. Send email/notification if monitor triggered
6. Log to `~/.maxfinder/logs/` with query_id for audit

## Golden Rules

1. **Never hallucinate**: If the source doesn't explicitly state something, mark it as "unclear" or "not found". Do not infer beyond provided text.

2. **Preserve provenance**: Every synthesized claim must be traceable to specific chunks with paper IDs and chunk indices. Store source mapping in output metadata.

3. **Respect rate limits**: Implement exponential backoff for all APIs. Never hammer servers. Cache aggressively. Default delays: arXiv (3s), PubMed (0.34s), Google Scholar (60s+), Crossref (1s).

4. **Handle paywalls gracefully**: Never attempt to bypass DRM. If open access unavailable, use metadata and abstract only. Do not log or share full copyrighted content without permission.

5. **Cite properly**: Always include DOIs when available. Use configured citation style exactly. Do not invent citations.

6. **Privacy protection**: Never store personal data from author affiliations (individual emails, phone numbers). Anonymize in logs unless explicitly needed for expert finding.

7. **Quota management**: Track API usage per provider. Alert if approaching limits. Default monthly limits: OpenAI ($20), Semantic Scholar (100/day), arXiv (unlimited but respectful).

8. **Version pinning**: Lock dependency versions in `~/.maxfinder/frozen_requirements.txt`. Do not auto-update without explicit `maxfinder upgrade` command.

9. **Cache invalidation**: Use content-based hashing (SHA256) for PDFs. Re-embed only when content changes. Cache TTL respects `MAX_FINDER_CACHE_TTL`.

10. **Fail open**: If any single source fails, continue with others. Log errors but don't abort entire search. Only fail if all sources unavailable.

## Examples

### Example 1: Systematic Literature Review

**User command**:
```bash
maxfinder search-papers "transformer attention mechanisms in vision tasks" \
  --database=arxiv,ieee,acm \
  --date-range=2020:2024 \
  --min-citations=10 \
  --limit=100 \
  --cited-by=2303.17671
```

**Internal process**:
- Expands to: ("transformer" OR "vision transformer" OR "ViT") AND ("attention" OR "self-attention" OR "multi-head attention") AND ("image classification" OR "object detection" OR "semantic segmentation")
- Searches arXiv cs.CV, cs.LG, cs.AI categories
- Queries IEEE Xplore with same terms, filters for "Conference" and "Journal"
- Filters results: citations >= 10, year >= 2020
- For each result, fetches full PDF if available
- Extracts: attention patterns, computational complexity, accuracy metrics, datasets used

**Synthesis prompt** (auto-generated):
```
Synthesize findings on transformer attention mechanisms in vision tasks from 2020-2024. 
Focus on: 1) attention pattern innovations, 2) computational efficiency improvements, 
3) accuracy trade-offs, 4) dataset dependencies. 
Exclude: pure NLP transformers, non-attention architectures.
```

**Output** (`./transformer_attention_synthesis_20240315.md`):
```markdown
# Transformer Attention Mechanisms in Vision Tasks: 2020-2024 Synthesis

**Query ID**: `arxiv_search_20240315_abc123`
**Sources**: 47 papers from arXiv (32), IEEE (10), ACM (5)
**Date Range**: 2020-2024

## Overview
Research on transformer attention for vision has evolved from naive adaptation of NLP transformers to specialized mechanisms addressing computational inefficiency and visual feature heterogeneity.

## Key Findings

### 1. Attention Pattern Innovations
- **Windowed attention** (Swin Transformer, 2021) reduces complexity from O(n²) to O(n) by limiting self-attention to local windows [1].
- **Axial attention** (Axial DeepLab, 2021) processes height and width dimensions separately [2].
- **Cross-attention between scales** (FPN-Transformer, 2022) enables multi-scale feature fusion [3].
- **Sparse attention patterns** with learnable patterns (Sparse Transformer, 2021) achieve 40% FLOPs reduction with <1% accuracy drop on ImageNet [4].

[References continue...]

## Methodological Trends
- 78% of papers use ImageNet-1K/21K as primary benchmark
- 92% report FLOPs and parameters; only 45% report inference latency
- 67% use COCO for detection, ADE20K for segmentation
- Attention heatmap visualization: 84% use Grad-CAM variants

## Contradictions & Open Questions
- **Efficiency vs accuracy trade-off**: Some studies claim windowed attention loses global context [5], while others compensate with cross-window connections [6]. No consensus on optimal window size.
- **Training data requirements**: Some claim ImageNet-21K pre-training essential [7]; others achieve SOTA with 1K [8].
- **_positional encoding for vision_: Relative vs absolute: no clear winner across tasks [9,10].

## Research Gaps
1. Attention mechanisms for video understanding remain under-explored (only 3 papers in dataset).
2. Few studies examine attention robustness to adversarial attacks.
3. Limited work on attention for low-resolution imagery (<64x64).

## Bibliographic Summary
| Paper | Year | Venue | Citations | Core Contribution |
|-------|------|-------|-----------|-------------------|
| Swin Transformer | 2021 | ICCV | 5,234 | Hierarchical shifted windows |
| PVT / PVTv2 | 2021/2022 | ICCV | 2,156 | Pyramidal structure |
| Focal Transformer | 2021 | CVPR | 987 | Multi-modal fusion |
| ... | ... | ... | ... | ... |

## Full Bibliography (APA 7th)
[1] Liu, Z., et al. (2021). Swin transformer: Hierarchical vision transformer using shifted windows. *ICCV 2021*.
[2] ... (full list exportable as .bib)
```

### Example 2: Automated Monitoring

**User command**:
```bash
maxfinder monitor "GPT-4 multimodal capabilities" \
  --sources=arxiv,techcrunch,openai-blog \
  --schedule="0 9 * * *" \
  --alert-email=research-team@company.com \
  --webhook=https://hooks.slack.com/services/...
```

**Cron job created**: `~/.maxfinder/cron/gpt4_multimodal_20240315`
- Runs daily at 9 AM
- Search query variations: 
  - "GPT-4V vision capabilities"
  - "GPT-4 multimodal"
  - "GPT-4 visual reasoning"
  - "GPT-4 image understanding"
- On new results (>5 new papers/articles), generate synthesis and send:
  - Email with markdown summary and key findings
  - Slack webhook with bullet points and links
  - Update Zotero collection "AI Monitors"

**Log output** (`~/.maxfinder/logs/monitor_20240316.log`):
```
[2024-03-16 09:00:01] Starting monitor: GPT-4 multimodal capabilities
[2024-03-16 09:00:05] Query expansion: 4 variants generated
[2024-03-16 09:00:10] Searching: arXiv (success, 12 results), TechCrunch (success, 3), OpenAI Blog (success, 1)
[2024-03-16 09:01:30] Deduped to 14 unique items (3 new since 2024-03-15)
[2024-03-16 09:03:45] Retrieved PDFs: 8/12 available
[2024-03-16 09:05:20] Synthesis complete. Key новости:
  - OpenAI releases GPT-4V(ision) technical report
  - Independent evaluation shows 75% accuracy on VQA v2
  - Prompt injection vulnerabilities identified in image mode
  - Microsoft integrates GPT-4V into Copilot
[2024-03-16 09:05:25] sent email to research-team@company.com
[2024-03-16 09:05:30] sent slack notification to #research-alerts
[2024-03-16 09:05:35] updated Zotero collection (5 new entries)
[2024-03-16 09:05:40] Monitor completed successfully. Next run: 2024-03-17 09:00:00
```

### Example 3: Expert Finding

**User command**:
```bash
maxfinder scholar-profile "Yoshua Bengio" \
  --include=publications,citations,affiliations \
  --output=json > bengio_profile_2024.json
```

**Output**:
```json
{
  "query": "Yoshua Bengio",
  "profile": {
    "name": "Yoshua Bengio",
    "affiliations": [
      "Universit\u00e9 de Montr\u00e9al",
      "Mila - Quebec AI Institute"
    ],
    "h_index": 124,
    "total_citations": 195432,
    "recent_citations_5y": 58432,
    "i10_index": 310,
    "publications_count": 425,
    "top_venues": [
      {"venue": "NeurIPS", "count": 87},
      {"venue": "ICML", "count": 65},
      {"venue": "ICLR", "count": 43}
    ],
    "research_areas": [
      "deep learning",
      "natural language processing",
      "generative models",
      "attention mechanisms",
      "representation learning"
    ],
    "collaborators": [
      {"name": "Ian Goodfellow", "papers": 23},
      {"name": "Aaron Courville", "papers": 41},
      {"name": "R phosphatidyl", "papers": 18}
    ],
    "recent_papers": [
      {
        "title": "A Neural Conversational Model",
        "year": 2024,
        "venue": "ICLR",
        "citations": 15,
        "doi": "10.48550/arxiv.2401.12345"
      }
    ]
  },
  "sources_used": ["Google Scholar", "Semantic Scholar", "DBLP"],
  "retrieved_at": "2024-03-15T14:30:22Z"
}
```

### Example 4: Fact Verification

**User command**:
```bash
maxfinder verify-facts "ResNet-50 achieves 76.4% top-1 accuracy on ImageNet" \
  --evidence-level=strict \
  --max-sources=20 \
  --output=report.html
```

**Output** (`./resnet50_accuracy_verification_20240315.html`):
```html
<h2>Fact Verification Report</h2>
<p><strong>Claim:</strong> "ResNet-50 achieves 76.4% top-1 accuracy on ImageNet"</p>

<h3>Verdict: <span style="color:green">VERIFIED</span> (Confidence: 0.93)</h3>

<h4>Evidence Found:</h4>
<ol>
  <li><strong>Original ResNet paper (He et al., 2015)</strong>: 
    <ul>
      <li>Top-1 accuracy: 76.4% on ImageNet validation set (single crop)</li>
      <li>Top-5 accuracy: 93.2%</li>
      <li>Source: <a href="https://arxiv.org/abs/1512.03385">arXiv:1512.03385</a></li>
      <li>Context excerpt: "...ResNet-50 obtains 76.4% top-1 accuracy..."</li>
    </ul>
    <small>Chunk: arxiv_151203385_chunk_12, lines 45-47</small>
  </li>
  <li><strong>PyTorch benchmarks (2020)</strong>:
    <ul>
      <li>Replicated result: 76.3% ± 0.1% (single GPU, 224x224)</li>
      <li>Source: PyTorch Image Models documentation</li>
    </ul>
  </li>
  <li><strong>Comprehensive ResNet analysis (2022)</strong>:
    <ul>
      <li>Confirms original numbers across multiple implementations</li>
      <li>Notes that multi-crop evaluation increases to 78.6%</li>
    </ul>
  </li>
</ol>

<h4>Context Conditions:</h4>
<ul>
  <li>Single-crop evaluation at 224×224 resolution</li>
  <li>Standard ImageNet validation set (50,000 images)</li>
  <li>No external data augmentation beyond training protocol</li>
  <li>Original pre-training on ImageNet-1K</li>
</ul>

<h4>Potential Confounders:</h4>
<p>Some later implementations report 76.3% due to random seed variations or slight preprocessing differences. The 76.4% figure is the original reported value.</p>

<h4>Sources Consulted:</h4>
<ul>
  <li>arXiv:1512.03385</li>
  <li>pytorch.org/vision/stable/models.html</li>
  <li>arXiv:2203.06866</li>
  <li>Papers With Code resnet50 benchmark</li>
  <li>Wikipedia: ResNet (verified citations)</li>
</ul>

<p><small>Generated by Max Finder Fact Checker v2.4.1 | Mode: strict | Timestamp: 2024-03-15T14:45:12Z</small></p>
```

### Example 5: Compare Methodologies

**User command**:
```bash
maxfinder analyze <paper_id1> <paper_id2> \
  --extract=methods,results \
  --summarize \
  --compare-with=statistical_tests,sample_sizes,limitations
```

**Output**:
```markdown
# Methodological Comparison

## Paper 1
**Title**: "Attention is All You Need" (2017)
**Authors**: Vaswani et al.
**Methods**: Transformer encoder-decoder with multi-head self-attention, no recurrence or convolutions.
**Sample**: WMT 2014 English-to-German (4.5M sentence pairs), English-to-French (36M sentence pairs).

## Paper 2
**Title**: "Swin Transformer: Hierarchical Vision Transformer using Shifted Windows" (2021)
**Authors**: Liu et al.
**Methods**: Hierarchical Swin Transformer with shifted windows, token propagation via patch merging.
**Sample**: ImageNet-1K (1.2M images), COCO (118K images), ADE20K (20K images).

## Comparative Analysis

### Architectural Approach
| Dimension | Transformer (2017) | Swin (2021) |
|-----------|-------------------|-------------|
| Attention scope | Global (O(n²)) | Local windows (O(n)) |
| Position encoding | Sinusoidal, absolute | Relative, learnable |
| Feature resolution | Fixed | Hierarchical, multi-scale |
| Memory footprint | Quadratic | Linear |

### Experimental Rigor
- **Sample size**: NLP: millions of tokens; Vision: 1.2M images (adequate)
- **Baselines**: Transformer: RNNseq2seq, ConvS2S; Swin: ResNet, ViT, DeiT (both strong)
- **Statistical tests**: Neither paper includes significance testing (common in ML)
- **Reproducibility**: Both provide code (Transformer in Tensor2Tensor, Swin in PyTorch)

### Key Difference: Computational Strategy
Transformer uses global attention at single scale; Swin uses shifted local attention with hierarchical feature maps to capture global context efficiently. Swin's linear complexity enables high-resolution vision tasks.

### Limitations Noted
- Transformer: "Self-attention is computationally expensive for long sequences" (Section 3.2)
- Swin: "Window-based self-attention limits interactions across windows" (Section 3.2) - mitigated by shifted windows.

## Synthesis
Swin can be viewed as an adaptation of Transformer principles to vision, addressing computational constraints via locality and hierarchy while maintaining the core attention mechanism.
```

## Rollback Commands

### 1. Revert to Previous Query Results

```bash
# List query history
maxfinder history --limit=20

# Restore previous search state
maxfinder restore <query_id>
# Example: maxfinder restore arxiv_search_20240315_abc123
# This reloads cached results, embeddings, and synthesis from that timestamp
```

### 2. Clear Cache Safely

```bash
# Dry-run: see what would be deleted
maxfinder cache --dry-run --older-than=7

# Clear specific cache types only
maxfinder cache clear --type=embeddings --older-than=30  # keep synthesis cache
maxfinder cache clear --type=results --older-than=90    # keep embedding vectors

# Full cache reset (preserves config)
maxfinder cache clear --all --confirm
# Prompts: "Delete 2.4GB in ~/.maxfinder/cache? (yes/no): "
```

### 3. Undo Zotero Sync

```bash
# Preview what would be deleted from Zotero
maxfinder zotero sync --dry-run --action=preview

# Rollback last sync (remove items added in last sync)
maxfinder zotero rollback --sync-id=20240315_abc123

# Remove all Max Finder items from Zotero (dangerous, requires confirmation)
maxfinder zotero cleanup --all --confirm
```

### 4. Disable/Restore Monitor Jobs

```bash
# List active monitors
maxfinder monitor list

# Temporarily disable (preserves config)
maxfinder monitor disable <query_id>

# Re-enable
maxfinder monitor enable <query_id>

# Permanently delete monitor
maxfinder monitor delete <query_id> --confirm
# Also removes associated cron job and log rotation config
```

### 5. Restore Configuration

```bash
# List config changes history (stored in ~/.maxfinder/config/history/)
maxfinder config history

# Restore specific version
maxfinder config restore --timestamp=2024-03-10T14:30:00

# Reset to factory defaults
maxfinder config reset --confirm
# WARNING: clears API keys, model preferences, citation style
```

### 6. Database Recovery

```bash
# Repair corrupted SQLite database
maxfinder db repair

# Export full database for backup
maxfinder db export ~/backup/maxfinder_$(date +%Y%m%d).sql

# Restore from backup (stops any running queries)
maxfinder db restore ~/backup/maxfinder_20240310.sql --confirm
```

### 7. ChromaDB Migration/Rollback

```bash
# Backup current embeddings
maxfinder embeddings backup --output=~/backups/chroma_$(date +%Y%m%d).tar.gz

# Rebuild embeddings with different model (preserves old collection)
maxfinder embeddings rebuild --old-collection=<name> --new-model=all-mpnet-base-v2

# Restore specific collection from backup
maxfinder embeddings restore --backup=chroma_20240310.tar.gz --collection-name=<name>
```

### 8. API Quota Emergency Stop

If you've exceeded API limits and need to pause all operations:

```bash
# Emergency stop: prevents all new searches, only serves cached results
maxfinder emergency-stop

# Resume normal operations
maxfinder emergency-resume
# Also check: maxfinder quota status
```

### 9. Rollback Specific Operation by Type

```bash
# Undo last 'search-papers' operation (ignores other types)
maxfinder rollback --operation=search-papers

# Undo all operations from specific date
maxfinder rollback --date=2024-03-14

# Interactive rollback browser
maxfinder rollback --interactive
# Shows numbered list of recent operations with previews, you select which to undo
```

### 10. Docker/Container Reset (if using containerized deployment)

```bash
# Stop container and remove volume (WARNING: loses all local data)
docker stop maxfinder && docker rm maxfinder
docker volume rm maxfinder_data

# Reinitialize with fresh database
maxfinder init --force

# Restore from most recent backup
maxfinder restore --latest-backup
```

---

## Troubleshooting

### Issue: "Embedding generation failed: CUDA out of memory"

**Solution**:
```bash
# Reduce batch size and chunk size in config
maxfinder config set embedding_batch_size 16
maxfinder config set chunk_size 500

# Or force CPU (slower but guaranteed to work)
export MAX_FINDER_DEVICE=cpu
maxfinder embeddings build ...
```

### Issue: "arXiv API 429: Too Many Requests"

**Solution**:
```bash
# Increase default delay
maxfinder config set arxiv_delay 10.0

# Or throttle globally
maxfinder config set max_concurrent_sources 2

# Check current usage
maxfinder quota arxiv
# Output: "Used: 3,000/10,000 requests today (resets in 2h)"
```

### Issue: "Semantic Scholar rate limit exceeded"

**Solution**:
```bash
# Semantic Scholar free tier: 100 requests/day
# Your current usage: 102 requests
#
# Solutions:
# 1. Subscribe to Academic API tier
# 2. Reduce search limit: add --limit=20 instead of 100
# 3. Cache aggressively: set MAX_FINDER_CACHE_TTL=604800 (7 days)
# 4. Use alternative sources: switch to arXiv+Google Scholar
```

### Issue: "PDF extraction returns garbled text"

**Diagnosis**:
```bash
# Check if PDF is scanned/image-based
maxfinder qpdf --check <file.pdf>  # if encrypted, need password
pdftoppm -png <file.pdf> /tmp/test_page  # generates images, indicates scanned

# Solution for scanned PDFs:
# Install OCR dependencies:
sudo apt-get install tesseract-ocr tesseract-ocr-eng poppler-utils
maxfinder config set ocr_enabled true
maxfinder config set ocr_language eng
```

### Issue: "Synthesis is repetitive or generic"

**Cause**: Context window too large or low-quality sources.

**Fixes**:
```bash
# 1. Filter by citation count before synthesis
maxfinder search-papers ... --min-citations=50 --sort-by=citations

# 2. Limit synthesis to papers from top venues only
maxfinder config set venue_priority "NeurIPS,ICML,ICCV,CVPR,ACL"

# 3. Use higher-quality model
export MAX_FINDER_MODEL=gpt-4-turbo-preview  # instead of gpt-3.5-turbo

# 4. Post-process: filter synthesized output
maxfinder synthesize <query> --deduplicate-claims --min-evidence=2
```

### Issue: "Monitor jobs don't run"

**Checklist**:
```bash
# 1. Verify cron is active
systemctl status cron || systemctl status crond

# 2. List Max Finder's cron entries
crontab -l | grep maxfinder

# 3. Check log rotation
ls -la ~/.maxfinder/logs/ | grep rotate

# 4. Test run manually
maxfinder monitor run --query-id=<id> --dry-run

# 5. Ensure environment variables loaded in cron
# Add to crontab:
# SHELL=/bin/bash
# MAX_FINDER_HOME=/home/user/.maxfinder
# PATH=/usr/local/bin:/usr/bin:/bin
```

### Issue: "Zotero sync fails: 403 Forbidden"

**Solution**:
```bash
# 1. Verify API key has write permissions
maxfinder zotero check-permissions

# 2. Ensure library type matches (group library vs personal)
maxfinder config set zotero_library_type group  # or 'user'

# 3. Check collection exists and is writable
maxfinder zotero list-collections

# 4. Don't exceed rate limit: Zotero 10 requests/second
maxfinder config set zotero_batch_size 20  # smaller batches
```

### Issue: "ChromaDB query returns no results after embedding"

**Diagnosis and repair**:
```bash
# 1. Check collection exists
maxfinder embeddings list-collections

# 2. Count documents in collection
maxfinder embeddings count <name>

# 3. If empty, rebuild:
maxfinder embeddings rebuild <name> --force

# 4. If corrupted:
rm -rf ~/.maxfinder/chroma_db/
maxfinder init  # reinitialize database
```

### Issue: "High API costs despite caching"

**Investigation**:
```bash
# Check cache hit rates
maxfinder cache stats

# Expected output:
# Embeddings cache: 94% hit rate (saved $12.30 in OpenAI costs)
# Search results cache: 87% hit rate
# Synthesis cache: 76% hit rate

# If low:
# 1. Cache TTL too short:
maxfinder config set cache_ttl_days 30

# 2. Cache key misses due to query variation:
maxfinder config set normalize_queries true  # lowercases, removes punctuation

# 3. Not caching failures:
maxfinder config set cache_negative_results true
```

### Issue: "Citation style format incorrect for journal X"

**Fix**: Use custom CSL (Citation Style Language) file.

```bash
# 1. Get journal-specific CSL from Zotero Styles repo
curl -o ~/.maxfinder/styles/custom_journal.csl \
  https://raw.githubusercontent.com/citation-style-language/styles/master/...

# 2. Configure:
maxfinder config set citation_style_file ~/.maxfinder/styles/custom_journal.csl

# 3. Test:
maxfinder synthesize <query> --citation-style=custom_journal.csl --dry-run
```

### Issue: "Multi-process queries interfere"

**Problem**: Simultaneous maxfinder commands corrupt SQLite.

**Solution**: Ensure only one instance runs.

```bash
# Add flock wrapper to your scripts:
flock -n /tmp/maxfinder.lock maxfinder search ...

# Or configure Max Finder to use file locking:
maxfinder config set enable_file_lock true
maxfinder config set lock_timeout_seconds 300
```

### Issue: "Search returns 0 results from all sources"

**Debug steps**:
```bash
# 1. Check query expansion results
maxfinder search "test query" --verbose=3 | grep -i "query variants"

# 2. Test each source individually
maxfinder search-papers "quantum computing" --source=arxiv --limit=1
maxfinder search-papers "quantum computing" --source=google-scholar --limit=1

# 3. Verify network connectivity
curl -I https://export.arxiv.org/  # should return 200
curl -I https://api.semanticscholar.org/  # 200

# 4. Check API keys validity
maxfinder health-check
# Output includes: "✓ OpenAI API key valid", "✓ Semantic Scholar API key valid"
```

## Verification Steps

After installation or upgrade:

```bash
# 1. Health check (all dependencies)
maxfinder health-check
# Expected: ✓ python 3.9+, ✓ chromadb, ✓ openai api, ✓ all source APIs

# 2. Test single search
maxfinder search-papers "deep learning" --limit=5 --source=arxiv --output=json | head -20
# Expected: 5 papers with title, authors, abstract, arxiv_id

# 3. Test embedding+search
maxfinder embeddings build test_collection --input="tests/sample_papers/"
maxfinder embeddings search test_collection "attention mechanisms"
# Expected: list of matching chunks with similarity scores

# 4. Test synthesis with small query
maxfinder synthesize "What is transfer learning?" --sources=arxiv,semantic-scholar --limit=10 --output=md
# Expected: ./synthesis_*.md with 300-500 words and citations

# 5. Test monitor scheduling
maxfinder monitor test "machine learning" --sources=arxiv
# Expected: Immediate run, report in ~/.maxfinder/monitor_outputs/

# 6. Verify database integrity
maxfinder db check
# Expected: "No corruption detected. X papers indexed, Y embeddings stored."
```

## Performance Benchmarks

Typical performance on moderate hardware (8 CPU, 16GB RAM, moderate SSD):

| Task | Time (first run) | Time (cached) | Memory |
|------|------------------|---------------|--------|
| Search 50 papers (3 sources) | 60-90s | 5-10s | 200MB |
| PDF extraction (50 papers) | 2-5min | 30s | 500MB |
| Embedding generation (500 chunks) | 3-5min | 0s (cached) | 1GB |
| Synthesis (50 sources) | 30-90s | 10-20s | 800MB |
| Full pipeline (search→extract→synthesize) | 10-15min | 2-4min | 2GB peak |

Expected caching improvements: With `MAX_FINDER_CACHE_TTL=604800`, repeat queries on same topic within 7 days complete in <30 seconds.

---

**Version**: 2.4.1  
**Last Updated**: 2024-03-15  
**Maintainer**: Max Finder Team <support@maxfinder.ai>  
**License**: AGPLv3  
**Repository**: https://github.com/maxfinder-ai/maxfinder  
**Issue Tracker**: https://github.com/maxfinder-ai/maxfinder/issues  
**Documentation**: https://docs.maxfinder.ai  
**Changelog**: https://github.com/maxfinder-ai/maxfinder/releases
```