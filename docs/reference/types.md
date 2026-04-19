---
title: "Types Reference"
---

## Types Reference

All types defined by the library, grouped by category. Types are shown using Rust as the canonical representation.

### Result Types

#### StructuredDataResult

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `content` | `String` | — | The extracted text content |
| `format` | `str` | — | Format (str) |
| `metadata` | `HashMap<String, String>` | — | Document metadata |
| `text_fields` | `Vec<String>` | — | Text fields |

---

#### ImageOcrResult

Result of OCR extraction from an image with optional page tracking.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `content` | `String` | — | Extracted text content |
| `boundaries` | `Vec<PageBoundary>` | `None` | Character byte boundaries per frame (for multi-frame TIFFs) |
| `page_contents` | `Vec<PageContent>` | `None` | Per-frame content information |

---

#### HtmlExtractionResult

Result of HTML extraction with optional images and warnings.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `markdown` | `String` | — | Markdown |
| `images` | `Vec<ExtractedInlineImage>` | — | Images extracted from the document |
| `warnings` | `Vec<String>` | — | Warnings |

---

#### DocExtractionResult

Result of DOC text extraction.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `text` | `String` | — | Extracted text content. |
| `metadata` | `DocMetadata` | — | Document metadata. |

---

#### PptExtractionResult

Result of PPT text extraction.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `text` | `String` | — | Extracted text content, with slides separated by double newlines. |
| `slide_count` | `usize` | — | Number of slides found. |
| `metadata` | `PptMetadata` | — | Document metadata. |
| `speaker_notes` | `Vec<String>` | — | Speaker notes text per slide (if available). |

---

#### ExtractionResult

General extraction result used by the core extraction API.

This is the main result type returned by all extraction functions.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `content` | `String` | — | The extracted text content |
| `mime_type` | `str` | — | The detected MIME type |
| `metadata` | `Metadata` | — | Document metadata |
| `tables` | `Vec<Table>` | `vec![]` | Tables extracted from the document |
| `detected_languages` | `Vec<String>` | `vec![]` | Detected languages |
| `chunks` | `Vec<Chunk>` | `vec![]` | Text chunks when chunking is enabled. When chunking configuration is provided, the content is split into overlapping chunks for efficient processing. Each chunk contains the text, optional embeddings (if enabled), and metadata about its position. |
| `images` | `Vec<ExtractedImage>` | `vec![]` | Extracted images from the document. When image extraction is enabled via `ImageExtractionConfig`, this field contains all images found in the document with their raw data and metadata. Each image may optionally contain a nested `ocr_result` if OCR was performed. |
| `pages` | `Vec<PageContent>` | `vec![]` | Per-page content when page extraction is enabled. When page extraction is configured, the document is split into per-page content with tables and images mapped to their respective pages. |
| `elements` | `Vec<Element>` | `vec![]` | Semantic elements when element-based result format is enabled. When result_format is set to ElementBased, this field contains semantic elements with type classification, unique identifiers, and metadata for Unstructured-compatible element-based processing. |
| `djot_content` | `Option<DjotContent>` | `Default::default()` | Rich Djot content structure (when extracting Djot documents). When extracting Djot documents with structured extraction enabled, this field contains the full semantic structure including: - Block-level elements with nesting - Inline formatting with attributes - Links, images, footnotes - Math expressions - Complete attribute information The `content` field still contains plain text for backward compatibility. Always `None` for non-Djot documents. |
| `ocr_elements` | `Vec<OcrElement>` | `vec![]` | OCR elements with full spatial and confidence metadata. When OCR is performed with element extraction enabled, this field contains the structured representation of detected text including: - Bounding geometry (rectangles or quadrilaterals) - Confidence scores (detection and recognition) - Rotation information - Hierarchical relationships (Tesseract only) This field preserves all metadata that would otherwise be lost when converting to plain text or markdown output formats. Only populated when `OcrElementConfig.include_elements` is true. |
| `document` | `Option<DocumentStructure>` | `Default::default()` | Structured document tree (when document structure extraction is enabled). When `include_document_structure` is true in `ExtractionConfig`, this field contains the full hierarchical representation of the document including: - Heading-driven section nesting - Table grids with cell-level metadata - Content layer classification (body, header, footer, footnote) - Inline text annotations (formatting, links) - Bounding boxes and page numbers Independent of `result_format` — can be combined with Unified or ElementBased. |
| `quality_score` | `Option<f64>` | `Default::default()` | Document quality score from quality analysis. A value between 0.0 and 1.0 indicating the overall text quality. Previously stored in `metadata.additional["quality_score"]`. |
| `processing_warnings` | `Vec<ProcessingWarning>` | `vec![]` | Non-fatal warnings collected during processing pipeline stages. Captures errors from optional pipeline features (embedding, chunking, language detection, output formatting) that don't prevent extraction but may indicate degraded results. Previously stored as individual keys in `metadata.additional`. |
| `annotations` | `Vec<PdfAnnotation>` | `vec![]` | PDF annotations extracted from the document. When annotation extraction is enabled via `PdfConfig.extract_annotations`, this field contains text notes, highlights, links, stamps, and other annotations found in PDF documents. |
| `children` | `Vec<ArchiveEntry>` | `vec![]` | Nested extraction results from archive contents. When extracting archives, each processable file inside produces its own full extraction result. Set to `None` for non-archive formats. Use `max_archive_depth` in config to control recursion depth. |
| `uris` | `Vec<Uri>` | `vec![]` | URIs/links discovered during document extraction. Contains hyperlinks, image references, citations, email addresses, and other URI-like references found in the document. Always extracted when present in the source document. |
| `structured_output` | `Option<serde_json::Value>` | `Default::default()` | Structured extraction output from LLM-based JSON schema extraction. When `structured_extraction` is configured in `ExtractionConfig`, the extracted document content is sent to a VLM with the provided JSON schema. The response is parsed and stored here as a JSON value matching the schema. |
| `code_intelligence` | `Option<ProcessResult>` | `Default::default()` | Code intelligence results from tree-sitter analysis. Populated when extracting source code files with the `tree-sitter` feature. Contains metrics, structural analysis, imports/exports, comments, docstrings, symbols, diagnostics, and optionally chunked code segments. |
| `llm_usage` | `Vec<LlmUsage>` | `vec![]` | LLM token usage and cost data for all LLM calls made during this extraction. Contains one entry per LLM call. Multiple entries are produced when VLM OCR, structured extraction, and/or LLM embeddings all run during the same extraction. `None` when no LLM was used. |
| `formatted_content` | `Option<String>` | `Default::default()` | Pre-rendered content in the requested output format. Populated during `derive_extraction_result` before tree derivation consumes element data. `apply_output_format` swaps this into `content` at the end of the pipeline, after post-processors have operated on plain text. |
| `ocr_internal_document` | `Option<InternalDocument>` | `Default::default()` | Structured hOCR document for the OCR+layout pipeline. When tesseract produces hOCR output, the parsed `InternalDocument` carries paragraph structure with bounding boxes and confidence scores. The layout classification step enriches these elements before final rendering. |

---

#### XmlExtractionResult

XML extraction result.

Contains extracted text content from XML files along with
structural statistics about the XML document.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `content` | `String` | — | Extracted text content (XML structure filtered out) |
| `element_count` | `usize` | — | Total number of XML elements processed |
| `unique_elements` | `Vec<String>` | — | List of unique element names found (sorted) |

---

#### TextExtractionResult

Plain text and Markdown extraction result.

Contains the extracted text along with statistics and,
for Markdown files, structural elements like headers and links.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `content` | `String` | — | Extracted text content |
| `line_count` | `usize` | — | Number of lines |
| `word_count` | `usize` | — | Number of words |
| `character_count` | `usize` | — | Number of characters |
| `headers` | `Vec<String>` | `None` | Markdown headers (text only, Markdown files only) |
| `links` | `Vec<(String, String)>` | `None` | Markdown links as (text, URL) tuples (Markdown files only) |
| `code_blocks` | `Vec<(String, String)>` | `None` | Code blocks as (language, code) tuples (Markdown files only) |

---

#### PptxExtractionResult

PowerPoint (PPTX) extraction result.

Contains extracted slide content, metadata, and embedded images/tables.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `content` | `String` | — | Extracted text content from all slides |
| `metadata` | `PptxMetadata` | — | Presentation metadata |
| `slide_count` | `usize` | — | Total number of slides |
| `image_count` | `usize` | — | Total number of embedded images |
| `table_count` | `usize` | — | Total number of tables |
| `images` | `Vec<ExtractedImage>` | — | Extracted images from the presentation |
| `page_structure` | `Option<PageStructure>` | `None` | Slide structure with boundaries (when page tracking is enabled) |
| `page_contents` | `Vec<PageContent>` | `None` | Per-slide content (when page tracking is enabled) |
| `document` | `Option<DocumentStructure>` | `None` | Structured document representation |
| `hyperlinks` | `Vec<(String, Option<String>)>` | — | Hyperlinks discovered in slides as (url, optional_label) pairs. |
| `office_metadata` | `HashMap<String, String>` | — | Office metadata extracted from docProps/core.xml and docProps/app.xml. Contains keys like "title", "author", "created_by", "subject", "keywords", "modified_by", "created_at", "modified_at", etc. |

---

#### EmailExtractionResult

Email extraction result.

Complete representation of an extracted email message (.eml or .msg)
including headers, body content, and attachments.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `subject` | `Option<String>` | `None` | Email subject line |
| `from_email` | `Option<String>` | `None` | Sender email address |
| `to_emails` | `Vec<String>` | — | Primary recipient email addresses |
| `cc_emails` | `Vec<String>` | — | CC recipient email addresses |
| `bcc_emails` | `Vec<String>` | — | BCC recipient email addresses |
| `date` | `Option<String>` | `None` | Email date/timestamp |
| `message_id` | `Option<String>` | `None` | Message-ID header value |
| `plain_text` | `Option<String>` | `None` | Plain text version of the email body |
| `html_content` | `Option<String>` | `None` | HTML version of the email body |
| `cleaned_text` | `String` | — | Cleaned/processed text content |
| `attachments` | `Vec<EmailAttachment>` | — | List of email attachments |
| `metadata` | `HashMap<String, String>` | — | Additional email headers and metadata |

---

#### OcrExtractionResult

OCR extraction result.

Result of performing OCR on an image or scanned document,
including recognized text and detected tables.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `content` | `String` | — | Recognized text content |
| `mime_type` | `String` | — | Original MIME type of the processed image |
| `metadata` | `HashMap<String, serde_json::Value>` | — | OCR processing metadata (confidence scores, language, etc.) |
| `tables` | `Vec<OcrTable>` | — | Tables detected and extracted via OCR |
| `ocr_elements` | `Vec<OcrElement>` | `None` | Structured OCR elements with bounding boxes and confidence scores. Available when TSV output is requested or table detection is enabled. |
| `internal_document` | `Option<InternalDocument>` | `None` | Structured document produced from hOCR parsing. Carries paragraph structure, bounding boxes, and confidence scores that the flattened `content` string discards. |

---

#### ChunkingResult

Result of a text chunking operation.

Contains the generated chunks and metadata about the chunking.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `chunks` | `Vec<Chunk>` | — | List of text chunks |
| `chunk_count` | `usize` | — | Total number of chunks generated |

---

#### NormalizeResult

Result of image normalization

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `rgb_data` | `Vec<u8>` | — | Processed RGB image data (height * width * 3 bytes) |
| `dimensions` | `(usize, usize)` | — | Image dimensions (width, height) |
| `metadata` | `ImagePreprocessingMetadata` | — | Preprocessing metadata |

---

#### BatchItemResult

Batch item result for processing multiple files

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `file_path` | `String` | — | File path |
| `success` | `bool` | — | Success |
| `result` | `Option<OcrExtractionResult>` | `None` | Result (ocr extraction result) |
| `error` | `Option<String>` | `None` | Error |

---

#### OrientationResult

Document orientation detection result.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `degrees` | `u32` | — | Detected orientation in degrees (0, 90, 180, or 270). |
| `confidence` | `f32` | — | Confidence score (0.0-1.0). |

---

#### SlanetResult

SLANeXT recognition result for a single table image.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `cells` | `Vec<SlanetCell>` | — | Detected cells with bounding boxes and grid positions. |
| `num_rows` | `usize` | — | Number of rows in the table. |
| `num_cols` | `usize` | — | Maximum number of columns across all rows. |
| `confidence` | `f32` | — | Average structure prediction confidence. |
| `structure_tokens` | `Vec<String>` | — | Raw HTML structure tokens (for debugging). |

---

#### TatrResult

Aggregated TATR recognition result with detections separated by class.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `rows` | `Vec<TatrDetection>` | — | Detected rows, sorted top-to-bottom by `y2`. |
| `columns` | `Vec<TatrDetection>` | — | Detected columns, sorted left-to-right by `x2`. |
| `headers` | `Vec<TatrDetection>` | — | Detected headers (ColumnHeader and ProjectedRowHeader). |
| `spanning` | `Vec<TatrDetection>` | — | Detected spanning cells. |

---

#### DetectionResult

Page-level detection result containing all detections and page metadata.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `page_width` | `u32` | — | Page width |
| `page_height` | `u32` | — | Page height |
| `detections` | `Vec<LayoutDetection>` | — | Detections |

---

#### KMeansResult

Result of KMeans clustering on font sizes.

Contains cluster labels for each block, where cluster index indicates
the hierarchy level: 0=H1, 1=H2, ..., 5=H6, 6+=Body.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `labels` | `Vec<u32>` | — | Cluster label for each block (0-indexed) |

---

#### PageLayoutResult

Layout detection results for a single page.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `page_index` | `usize` | — | Page index |
| `regions` | `Vec<PageLayoutRegion>` | — | Regions |
| `page_width_pts` | `f32` | — | Page width pts |
| `page_height_pts` | `f32` | — | Page height pts |
| `render_width_px` | `u32` | — | Width of the rendered image used for layout detection (pixels). |
| `render_height_px` | `u32` | — | Height of the rendered image used for layout detection (pixels). |

---

#### PdfUnifiedExtractionResult

Result type for unified PDF text and metadata extraction.

Contains text, optional page boundaries, optional per-page content, and metadata.

*Opaque type — fields are not directly accessible.*

---

### Configuration Types

See [Configuration Reference](configuration.md) for detailed defaults and language-specific representations.

#### CancellationToken

A lightweight, cloneable cancellation token.

Create one with `CancellationToken.new`, pass clones to the extraction
call (via `ExtractionConfig.cancel_token`) and to the caller. Call
`CancellationToken.cancel` from the caller side when the operation
should be aborted. The extraction code polls
`CancellationToken.is_cancelled` at safe checkpoints and returns
`KreuzbergError.Cancelled` if set.

Cloning is cheap (increments the `Arc` reference count only).

*Opaque type — fields are not directly accessible.*

---

#### BatchProcessorConfig

Configuration for batch processing with pooling optimizations.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `string_pool_size` | `usize` | `10` | Maximum number of string buffers to maintain in the pool |
| `string_buffer_capacity` | `usize` | `8192` | Initial capacity for pooled string buffers in bytes |
| `byte_pool_size` | `usize` | `10` | Maximum number of byte buffers to maintain in the pool |
| `byte_buffer_capacity` | `usize` | `65536` | Initial capacity for pooled byte buffers in bytes |
| `max_concurrent` | `Option<usize>` | `None` | Maximum concurrent extractions (for concurrency control) |

---

#### BatchProcessor

Batch processor that manages object pools for optimized extraction.

This struct manages the lifecycle of reusable object pools used during
batch extraction. Pools are created lazily on first use and reused across
all documents processed by this batch processor.

# Lazy Initialization

Pools are initialized on demand to reduce memory usage for applications
that may not use batch processing immediately or at all.

*Opaque type — fields are not directly accessible.*

---

#### AccelerationConfig

Hardware acceleration configuration for ONNX Runtime models.

Controls which execution provider (CPU, CoreML, CUDA, TensorRT) is used
for inference in layout detection and embedding generation.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `provider` | `ExecutionProviderType` | `ExecutionProviderType::Auto` | Execution provider to use for ONNX inference. |
| `device_id` | `u32` | — | GPU device ID (for CUDA/TensorRT). Ignored for CPU/CoreML/Auto. |

---

#### ConcurrencyConfig

Controls thread usage for constrained environments.

Set `max_threads` to cap all internal thread pools (Rayon, ONNX Runtime
intra-op) and batch concurrency to a single limit.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `max_threads` | `Option<usize>` | `Default::default()` | Maximum number of threads for all internal thread pools. Caps Rayon global pool size, ONNX Runtime intra-op threads, and (when `max_concurrent_extractions` is unset) the batch concurrency semaphore. When `None`, system defaults are used. |

---

#### ContentFilterConfig

Cross-extractor content filtering configuration.

Controls whether "furniture" content (headers, footers, page numbers,
watermarks, repeating text) is included in or stripped from extraction
results. Applies across all extractors (PDF, DOCX, RTF, ODT, HTML, etc.)
with format-specific implementation.

When `None` on `ExtractionConfig`, each extractor uses its current
default behavior unchanged.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `include_headers` | `bool` | `false` | Include running headers in extraction output. - PDF: Disables top-margin furniture stripping and prevents the layout model from treating `PageHeader`-classified regions as furniture. - DOCX: Includes document headers in text output. - RTF/ODT: Headers already included; this is a no-op when true. - HTML/EPUB: Keeps `<header>` element content. Default: `False` (headers are stripped or excluded). |
| `include_footers` | `bool` | `false` | Include running footers in extraction output. - PDF: Disables bottom-margin furniture stripping and prevents the layout model from treating `PageFooter`-classified regions as furniture. - DOCX: Includes document footers in text output. - RTF/ODT: Footers already included; this is a no-op when true. - HTML/EPUB: Keeps `<footer>` element content. Default: `False` (footers are stripped or excluded). |
| `strip_repeating_text` | `bool` | `true` | Enable the heuristic cross-page repeating text detector. When `True` (default), text that repeats verbatim across a supermajority of pages is classified as furniture and stripped.  Disable this if brand names or repeated headings are being incorrectly removed by the heuristic. Note: when a layout-detection model is active, the model may independently classify page-header / page-footer regions as furniture on a per-page basis. To preserve those regions, set `include_headers = true` and/or `include_footers = true` in addition to disabling this flag. Primarily affects PDF extraction. Default: `True`. |
| `include_watermarks` | `bool` | `false` | Include watermark text in extraction output. - PDF: Keeps watermark artifacts and arXiv identifiers. - Other formats: No effect currently. Default: `False` (watermarks are stripped). |

---

#### EmailConfig

Configuration for email extraction.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `msg_fallback_codepage` | `Option<u32>` | `Default::default()` | Windows codepage number to use when an MSG file contains no codepage property. Defaults to `None`, which falls back to windows-1252. If an unrecognized or invalid codepage number is supplied (including 0), the behavior silently falls back to windows-1252 — the same as when the MSG file itself contains an unrecognized codepage. No error or warning is emitted. Users should verify output when supplying unusual values. Common values: - 1250: Central European (Polish, Czech, Hungarian, etc.) - 1251: Cyrillic (Russian, Ukrainian, Bulgarian, etc.) - 1252: Western European (default) - 1253: Greek - 1254: Turkish - 1255: Hebrew - 1256: Arabic - 932:  Japanese (Shift-JIS) - 936:  Simplified Chinese (GBK) |

---

#### ExtractionConfig

Main extraction configuration.

This struct contains all configuration options for the extraction process.
It can be loaded from TOML, YAML, or JSON files, or created programmatically.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `use_cache` | `bool` | `true` | Enable caching of extraction results |
| `enable_quality_processing` | `bool` | `true` | Enable quality post-processing |
| `ocr` | `Option<OcrConfig>` | `None` | OCR configuration (None = OCR disabled) |
| `force_ocr` | `bool` | `false` | Force OCR even for searchable PDFs |
| `force_ocr_pages` | `Vec<usize>` | `None` | Force OCR on specific pages only (1-indexed page numbers, must be >= 1). When set, only the listed pages are OCR'd regardless of text layer quality. Unlisted pages use native text extraction. Ignored when `force_ocr` is `True`. Only applies to PDF documents. Duplicates are automatically deduplicated. An `ocr` config is recommended for backend/language selection; defaults are used if absent. |
| `disable_ocr` | `bool` | `false` | Disable OCR entirely, even for images. When `True`, OCR is skipped for all document types. Images return metadata only (dimensions, format, EXIF) without text extraction. PDFs use only native text extraction without OCR fallback. Cannot be `True` simultaneously with `force_ocr`. *Added in v4.7.0.* |
| `chunking` | `Option<ChunkingConfig>` | `None` | Text chunking configuration (None = chunking disabled) |
| `content_filter` | `Option<ContentFilterConfig>` | `None` | Content filtering configuration (None = use extractor defaults). Controls whether document "furniture" (headers, footers, watermarks, repeating text) is included in or stripped from extraction results. See `ContentFilterConfig` for per-field documentation. |
| `images` | `Option<ImageExtractionConfig>` | `None` | Image extraction configuration (None = no image extraction) |
| `pdf_options` | `Option<PdfConfig>` | `None` | PDF-specific options (None = use defaults) |
| `token_reduction` | `Option<TokenReductionOptions>` | `None` | Token reduction configuration (None = no token reduction) |
| `language_detection` | `Option<LanguageDetectionConfig>` | `None` | Language detection configuration (None = no language detection) |
| `pages` | `Option<PageConfig>` | `None` | Page extraction configuration (None = no page tracking) |
| `postprocessor` | `Option<PostProcessorConfig>` | `None` | Post-processor configuration (None = use defaults) |
| `html_options` | `Option<ConversionOptions>` | `None` | HTML to Markdown conversion options (None = use defaults) Configure how HTML documents are converted to Markdown, including heading styles, list formatting, code block styles, and preprocessing options. |
| `html_output` | `Option<HtmlOutputConfig>` | `None` | Styled HTML output configuration. When set alongside `output_format = OutputFormat.Html`, the extraction pipeline uses `StyledHtmlRenderer` which emits stable `kb-*` CSS class hooks on every structural element and optionally embeds theme CSS or user-supplied CSS in a `<style>` block. When `None`, the existing plain comrak-based HTML renderer is used. |
| `extraction_timeout_secs` | `Option<u64>` | `None` | Default per-file timeout in seconds for batch extraction. When set, each file in a batch will be canceled after this duration unless overridden by `FileExtractionConfig.timeout_secs`. `None` means no timeout (unbounded extraction time). |
| `max_concurrent_extractions` | `Option<usize>` | `None` | Maximum concurrent extractions in batch operations (None = (num_cpus × 1.5).ceil()). Limits parallelism to prevent resource exhaustion when processing large batches. Defaults to (num_cpus × 1.5).ceil() when not set. |
| `result_format` | `OutputFormat` | `OutputFormat::Plain` | Result structure format Controls whether results are returned in unified format (default) with all content in the `content` field, or element-based format with semantic elements (for Unstructured-compatible output). |
| `security_limits` | `Option<SecurityLimits>` | `None` | Security limits for archive extraction. Controls maximum archive size, compression ratio, file count, and other security thresholds to prevent decompression bomb attacks. When `None`, default limits are used (500MB archive, 100:1 ratio, 10K files). |
| `output_format` | `OutputFormat` | `OutputFormat::Plain` | Content text format (default: Plain). Controls the format of the extracted content: - `Plain`: Raw extracted text (default) - `Markdown`: Markdown formatted output - `Djot`: Djot markup format (requires djot feature) - `Html`: HTML formatted output When set to a structured format, extraction results will include formatted output. The `formatted_content` field may be populated when format conversion is applied. |
| `layout` | `Option<LayoutDetectionConfig>` | `None` | Layout detection configuration (None = layout detection disabled). When set, PDF pages and images are analyzed for document structure (headings, code, formulas, tables, figures, etc.) using RT-DETR models via ONNX Runtime. For PDFs, layout hints override paragraph classification in the markdown pipeline. For images, per-region OCR is performed with markdown formatting based on detected layout classes. Requires the `layout-detection` feature. |
| `include_document_structure` | `bool` | `false` | Enable structured document tree output. When true, populates the `document` field on `ExtractionResult` with a hierarchical `DocumentStructure` containing heading-driven section nesting, table grids, content layer classification, and inline annotations. Independent of `result_format` — can be combined with Unified or ElementBased. |
| `acceleration` | `Option<AccelerationConfig>` | `None` | Hardware acceleration configuration for ONNX Runtime models. Controls execution provider selection for layout detection and embedding models. When `None`, uses platform defaults (CoreML on macOS, CUDA on Linux, CPU on Windows). |
| `cache_namespace` | `Option<String>` | `None` | Cache namespace for tenant isolation. When set, cache entries are stored under `{cache_dir}/{namespace}/`. Must be alphanumeric, hyphens, or underscores only (max 64 chars). Different namespaces have isolated cache spaces on the same filesystem. |
| `cache_ttl_secs` | `Option<u64>` | `None` | Per-request cache TTL in seconds. Overrides the global `max_age_days` for this specific extraction. When `0`, caching is completely skipped (no read or write). When `None`, the global TTL applies. |
| `email` | `Option<EmailConfig>` | `None` | Email extraction configuration (None = use defaults). Currently supports configuring the fallback codepage for MSG files that do not specify one. See `crate.core.config.EmailConfig` for details. |
| `concurrency` | `Option<ConcurrencyConfig>` | `None` | Concurrency limits for constrained environments (None = use defaults). Controls Rayon thread pool size, ONNX Runtime intra-op threads, and (when `max_concurrent_extractions` is unset) the batch concurrency semaphore. See `crate.core.config.ConcurrencyConfig` for details. |
| `max_archive_depth` | `usize` | — | Maximum recursion depth for archive extraction (default: 3). Set to 0 to disable recursive extraction (legacy behavior). |
| `tree_sitter` | `Option<TreeSitterConfig>` | `None` | Tree-sitter language pack configuration (None = tree-sitter disabled). When set, enables code file extraction using tree-sitter parsers. Controls grammar download behavior and code analysis options. |
| `structured_extraction` | `Option<StructuredExtractionConfig>` | `None` | Structured extraction via LLM (None = disabled). When set, the extracted document content is sent to an LLM with the provided JSON schema. The structured response is stored in `ExtractionResult.structured_output`. |
| `cancel_token` | `Option<CancellationToken>` | `None` | Cancellation token for this extraction (None = no external cancellation). Pass a `CancellationToken` clone here and call `CancellationToken.cancel` from another thread / task to abort the extraction in progress. The extractor checks the token at safe checkpoints (before lock acquisition, between pages, between batch items) and returns `KreuzbergError.Cancelled` when set. The field is excluded from serialization because `CancellationToken` is a runtime handle, not a configuration value. |

---

#### FileExtractionConfig

Per-file extraction configuration overrides for batch processing.

All fields are `Option<T>` — `None` means "use the batch-level default."
This type is used with `crate.batch_extract_file` and
`crate.batch_extract_bytes` to allow heterogeneous
extraction settings within a single batch.

# Excluded Fields

The following `super.ExtractionConfig` fields are batch-level only and
cannot be overridden per file:
- `max_concurrent_extractions` — controls batch parallelism
- `use_cache` — global caching policy
- `acceleration` — shared ONNX execution provider
- `security_limits` — global archive security policy

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `enable_quality_processing` | `Option<bool>` | `Default::default()` | Override quality post-processing for this file. |
| `ocr` | `Option<OcrConfig>` | `Default::default()` | Override OCR configuration for this file (None in the Option = use batch default). |
| `force_ocr` | `Option<bool>` | `Default::default()` | Override force OCR for this file. |
| `force_ocr_pages` | `Vec<usize>` | `vec![]` | Override force OCR pages for this file (1-indexed page numbers). |
| `disable_ocr` | `Option<bool>` | `Default::default()` | Override disable OCR for this file. |
| `chunking` | `Option<ChunkingConfig>` | `Default::default()` | Override chunking configuration for this file. |
| `content_filter` | `Option<ContentFilterConfig>` | `Default::default()` | Override content filtering configuration for this file. |
| `images` | `Option<ImageExtractionConfig>` | `Default::default()` | Override image extraction configuration for this file. |
| `pdf_options` | `Option<PdfConfig>` | `Default::default()` | Override PDF options for this file. |
| `token_reduction` | `Option<TokenReductionOptions>` | `Default::default()` | Override token reduction for this file. |
| `language_detection` | `Option<LanguageDetectionConfig>` | `Default::default()` | Override language detection for this file. |
| `pages` | `Option<PageConfig>` | `Default::default()` | Override page extraction for this file. |
| `postprocessor` | `Option<PostProcessorConfig>` | `Default::default()` | Override post-processor for this file. |
| `html_options` | `Option<ConversionOptions>` | `Default::default()` | Override HTML conversion options for this file. |
| `result_format` | `Option<OutputFormat>` | `Default::default()` | Override result format for this file. |
| `output_format` | `Option<OutputFormat>` | `Default::default()` | Override output content format for this file. |
| `include_document_structure` | `Option<bool>` | `Default::default()` | Override document structure output for this file. |
| `layout` | `Option<LayoutDetectionConfig>` | `Default::default()` | Override layout detection for this file. |
| `timeout_secs` | `Option<u64>` | `Default::default()` | Override per-file extraction timeout in seconds. When set, the extraction for this file will be canceled after the specified duration. A timed-out file produces an error result without affecting other files in the batch. |
| `tree_sitter` | `Option<TreeSitterConfig>` | `Default::default()` | Override tree-sitter configuration for this file. |
| `structured_extraction` | `Option<StructuredExtractionConfig>` | `Default::default()` | Override structured extraction configuration for this file. When set, enables LLM-based structured extraction with a JSON schema for this specific file. The extracted content is sent to a VLM/LLM and the response is parsed according to the provided schema. |

---

#### ImageExtractionConfig

Image extraction configuration.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `extract_images` | `bool` | — | Extract images from documents |
| `target_dpi` | `i32` | — | Target DPI for image normalization |
| `max_image_dimension` | `i32` | — | Maximum dimension for images (width or height) |
| `inject_placeholders` | `bool` | — | Whether to inject image reference placeholders into markdown output. When `True` (default), image references like `![Image 1](embedded:p1_i0)` are appended to the markdown. Set to `False` to extract images as data without polluting the markdown output. |
| `auto_adjust_dpi` | `bool` | — | Automatically adjust DPI based on image content |
| `min_dpi` | `i32` | — | Minimum DPI threshold |
| `max_dpi` | `i32` | — | Maximum DPI threshold |

---

#### TokenReductionOptions

Token reduction configuration.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `mode` | `String` | — | Reduction mode: "off", "light", "moderate", "aggressive", "maximum" |
| `preserve_important_words` | `bool` | — | Preserve important words (capitalized, technical terms) |

---

#### LanguageDetectionConfig

Language detection configuration.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `enabled` | `bool` | — | Enable language detection |
| `min_confidence` | `f64` | — | Minimum confidence threshold (0.0-1.0) |
| `detect_multiple` | `bool` | — | Detect multiple languages in the document |

---

#### HtmlOutputConfig

Configuration for styled HTML output.

When set on `ExtractionConfig.html_output` alongside
`output_format = OutputFormat.Html`, the pipeline builds a
`StyledHtmlRenderer` instead of
the plain comrak-based renderer.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `css` | `Option<String>` | `None` | Inline CSS string injected into the output after the theme stylesheet. Concatenated after `css_file` content when both are set. |
| `css_file` | `Option<PathBuf>` | `None` | Path to a CSS file loaded once at renderer construction time. Concatenated before `css` when both are set. |
| `theme` | `HtmlTheme` | `HtmlTheme::Unstyled` | Built-in colour/typography theme. Default: `HtmlTheme.Unstyled`. |
| `class_prefix` | `String` | — | CSS class prefix applied to every emitted class name. Default: `"kb-"`. Change this if your host application already uses classes that start with `kb-`. |
| `embed_css` | `bool` | `true` | When `True` (default), write the resolved CSS into a `<style>` block immediately after the opening `<div class="{prefix}doc">`. Set to `False` to emit only the structural markup and wire up your own stylesheet targeting the `kb-*` class names. |

---

#### LayoutDetectionConfig

Layout detection configuration.

Controls layout detection behavior in the extraction pipeline.
When set on `ExtractionConfig`, layout detection
is enabled for PDF extraction.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `confidence_threshold` | `Option<f32>` | `None` | Confidence threshold override (None = use model default). |
| `apply_heuristics` | `bool` | `true` | Whether to apply postprocessing heuristics (default: true). |
| `table_model` | `TableModel` | `TableModel::Tatr` | Table structure recognition model. Controls which model is used for table cell detection within layout-detected table regions. Defaults to `TableModel.Tatr`. |
| `acceleration` | `Option<AccelerationConfig>` | `None` | Hardware acceleration for ONNX models (layout detection + table structure). When set, controls which execution provider (CPU, CUDA, CoreML, TensorRT) is used for inference. Defaults to `None` (auto-select per platform). |

---

#### LlmConfig

Configuration for an LLM provider/model via liter-llm.

Each feature (VLM OCR, VLM embeddings, structured extraction) carries
its own `LlmConfig`, allowing different providers per feature.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `model` | `String` | — | Provider/model string using liter-llm routing format. Examples: `"openai/gpt-4o"`, `"anthropic/claude-sonnet-4-20250514"`, `"groq/llama-3.1-70b-versatile"`. |
| `api_key` | `Option<String>` | `Default::default()` | API key for the provider. When `None`, liter-llm falls back to the provider's standard environment variable (e.g., `OPENAI_API_KEY`). |
| `base_url` | `Option<String>` | `Default::default()` | Custom base URL override for the provider endpoint. |
| `timeout_secs` | `Option<u64>` | `Default::default()` | Request timeout in seconds (default: 60). |
| `max_retries` | `Option<u32>` | `Default::default()` | Maximum retry attempts (default: 3). |
| `temperature` | `Option<f64>` | `Default::default()` | Sampling temperature for generation tasks. |
| `max_tokens` | `Option<u64>` | `Default::default()` | Maximum tokens to generate. |

---

#### StructuredExtractionConfig

Configuration for LLM-based structured data extraction.

Sends extracted document content to a VLM with a JSON schema,
returning structured data that conforms to the schema.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `schema` | `serde_json::Value` | — | JSON Schema defining the desired output structure. |
| `schema_name` | `String` | — | Schema name passed to the LLM's structured output mode. |
| `schema_description` | `Option<String>` | `None` | Optional schema description for the LLM. |
| `strict` | `bool` | — | Enable strict mode — output must exactly match the schema. |
| `prompt` | `Option<String>` | `None` | Custom Jinja2 extraction prompt template. When `None`, a default template is used. Available template variables: - `{{ content }}` — The extracted document text. - `{{ schema }}` — The JSON schema as a formatted string. - `{{ schema_name }}` — The schema name. - `{{ schema_description }}` — The schema description (may be empty). |
| `llm` | `LlmConfig` | — | LLM configuration for the extraction. |

---

#### OcrQualityThresholds

Quality thresholds for OCR fallback decisions and pipeline quality gating.

All fields default to the values that match the previous hardcoded behavior,
so `OcrQualityThresholds.default()` preserves existing semantics exactly.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `min_total_non_whitespace` | `usize` | `64` | Minimum total non-whitespace characters to consider text substantive. |
| `min_non_whitespace_per_page` | `f64` | `32` | Minimum non-whitespace characters per page on average. |
| `min_meaningful_word_len` | `usize` | `4` | Minimum character count for a word to be "meaningful". |
| `min_meaningful_words` | `usize` | `3` | Minimum count of meaningful words before text is accepted. |
| `min_alnum_ratio` | `f64` | `0.3` | Minimum alphanumeric ratio (non-whitespace chars that are alphanumeric). |
| `min_garbage_chars` | `usize` | `5` | Minimum Unicode replacement characters (U+FFFD) to trigger OCR fallback. |
| `max_fragmented_word_ratio` | `f64` | `0.6` | Maximum fraction of short (1-2 char) words before text is considered fragmented. |
| `critical_fragmented_word_ratio` | `f64` | `0.8` | Critical fragmentation threshold — triggers OCR regardless of meaningful words. Normal English text has ~20-30% short words. 80%+ is definitive garbage. |
| `min_avg_word_length` | `f64` | `2` | Minimum average word length. Below this with enough words indicates garbled extraction. |
| `min_words_for_avg_length_check` | `usize` | `50` | Minimum word count before average word length check applies. |
| `min_consecutive_repeat_ratio` | `f64` | `0.08` | Minimum consecutive word repetition ratio to detect column scrambling. |
| `min_words_for_repeat_check` | `usize` | `50` | Minimum word count before consecutive repetition check is applied. |
| `substantive_min_chars` | `usize` | `100` | Minimum character count for "substantive markdown" OCR skip gate. |
| `non_text_min_chars` | `usize` | `20` | Minimum character count for "non-text content" OCR skip gate. |
| `alnum_ws_ratio_threshold` | `f64` | `0.4` | Alphanumeric+whitespace ratio threshold for skip decisions. |
| `pipeline_min_quality` | `f64` | `0.5` | Minimum quality score (0.0-1.0) for a pipeline stage result to be accepted. If the result from a backend scores below this, try the next backend. |

---

#### OcrPipelineConfig

Multi-backend OCR pipeline with quality-based fallback.

Backends are tried in priority order (highest first). After each backend
produces output, quality is evaluated. If it meets `quality_thresholds.pipeline_min_quality`,
the result is accepted. Otherwise the next backend is tried.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `stages` | `Vec<OcrPipelineStage>` | — | Ordered list of backends to try. Sorted by priority (descending) at runtime. |
| `quality_thresholds` | `OcrQualityThresholds` | — | Quality thresholds for deciding whether to accept a result or try the next backend. |

---

#### OcrConfig

OCR configuration.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `enabled` | `bool` | `true` | Whether OCR is enabled. Setting `enabled: false` is a shorthand for `disable_ocr: true` on the parent `ExtractionConfig`. Images return metadata only; PDFs use native text extraction without OCR fallback. Defaults to `True`. When `False`, all other OCR settings are ignored. |
| `backend` | `String` | — | OCR backend: tesseract, easyocr, paddleocr |
| `language` | `String` | — | Language code (e.g., "eng", "deu") |
| `tesseract_config` | `Option<TesseractConfig>` | `None` | Tesseract-specific configuration (optional) |
| `output_format` | `Option<OutputFormat>` | `None` | Output format for OCR results (optional, for format conversion) |
| `paddle_ocr_config` | `Option<serde_json::Value>` | `None` | PaddleOCR-specific configuration (optional, JSON passthrough) |
| `element_config` | `Option<OcrElementConfig>` | `None` | OCR element extraction configuration |
| `quality_thresholds` | `Option<OcrQualityThresholds>` | `None` | Quality thresholds for the native-text-to-OCR fallback decision. When None, uses compiled defaults (matching previous hardcoded behavior). |
| `pipeline` | `Option<OcrPipelineConfig>` | `None` | Multi-backend OCR pipeline configuration. When set, enables weighted fallback across multiple OCR backends based on output quality. When None, uses the single `backend` field (same as today). |
| `auto_rotate` | `bool` | `false` | Enable automatic page rotation based on orientation detection. When enabled, uses Tesseract's `DetectOrientationScript()` to detect page orientation (0/90/180/270 degrees) before OCR. If the page is rotated with high confidence, the image is corrected before recognition. This is critical for handling rotated scanned documents. |
| `vlm_config` | `Option<LlmConfig>` | `None` | VLM (Vision Language Model) OCR configuration. Required when `backend` is `"vlm"`. Uses liter-llm to send page images to a vision model for text extraction. |
| `vlm_prompt` | `Option<String>` | `None` | Custom Jinja2 prompt template for VLM OCR. When `None`, uses the default template. Available variables: - `{{ language }}` — The document language code (e.g., "eng", "deu"). |

---

#### PageConfig

Page extraction and tracking configuration.

Controls how pages are extracted, tracked, and represented in the extraction results.
When `None`, page tracking is disabled.

Page range tracking in chunk metadata (first_page/last_page) is automatically enabled
when page boundaries are available and chunking is configured.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `extract_pages` | `bool` | `false` | Extract pages as separate array (ExtractionResult.pages) |
| `insert_page_markers` | `bool` | `false` | Insert page markers in main content string |
| `marker_format` | `String` | `"

<!-- PAGE {page_num} -->

"` | Page marker format (use {page_num} placeholder) Default: "\n\n<!-- PAGE {page_num} -->\n\n" |

---

#### PdfConfig

PDF-specific configuration.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `backend` | `PdfBackend` | `PdfBackend::Pdfium` | PDF extraction backend. Default: `Pdfium`. |
| `extract_images` | `bool` | `false` | Extract images from PDF |
| `passwords` | `Vec<String>` | `None` | List of passwords to try when opening encrypted PDFs |
| `extract_metadata` | `bool` | `true` | Extract PDF metadata |
| `hierarchy` | `Option<HierarchyConfig>` | `None` | Hierarchy extraction configuration (None = hierarchy extraction disabled) |
| `extract_annotations` | `bool` | `false` | Extract PDF annotations (text notes, highlights, links, stamps). Default: false |
| `top_margin_fraction` | `Option<f32>` | `None` | Top margin fraction (0.0–1.0) of page height to exclude headers/running heads. Default: 0.06 (6%) |
| `bottom_margin_fraction` | `Option<f32>` | `None` | Bottom margin fraction (0.0–1.0) of page height to exclude footers/page numbers. Default: 0.05 (5%) |
| `allow_single_column_tables` | `bool` | `false` | Allow single-column pseudo tables in extraction results. By default, tables with fewer than 2 columns (layout-guided) or 3 columns (heuristic) are rejected. When `True`, the minimum column count is relaxed to 1, allowing single-column structured data (glossaries, itemized lists) to be emitted as tables. Other quality filters (density, sparsity, prose detection) still apply. |

---

#### HierarchyConfig

Hierarchy extraction configuration for PDF text structure analysis.

Enables extraction of document hierarchy levels (H1-H6) based on font size
clustering and semantic analysis. When enabled, hierarchical blocks are
included in page content.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `enabled` | `bool` | `true` | Enable hierarchy extraction |
| `k_clusters` | `usize` | `3` | Number of font size clusters to use for hierarchy levels (1-7) Default: 6, which provides H1-H6 heading levels with body text. Larger values create more fine-grained hierarchy levels. |
| `include_bbox` | `bool` | `true` | Include bounding box information in hierarchy blocks |
| `ocr_coverage_threshold` | `Option<f32>` | `None` | OCR coverage threshold for smart OCR triggering (0.0-1.0) Determines when OCR should be triggered based on text block coverage. OCR is triggered when text blocks cover less than this fraction of the page. Default: 0.5 (trigger OCR if less than 50% of page has text) |

---

#### PostProcessorConfig

Post-processor configuration.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `enabled` | `bool` | `true` | Enable post-processors |
| `enabled_processors` | `Vec<String>` | `None` | Whitelist of processor names to run (None = all enabled) |
| `disabled_processors` | `Vec<String>` | `None` | Blacklist of processor names to skip (None = none disabled) |
| `enabled_set` | `Option<AHashSet>` | `None` | Pre-computed AHashSet for O(1) enabled processor lookup |
| `disabled_set` | `Option<AHashSet>` | `None` | Pre-computed AHashSet for O(1) disabled processor lookup |

---

#### ChunkingConfig

Chunking configuration.

Configures text chunking for document content, including chunk size,
overlap, trimming behavior, and optional embeddings.

Use `..the default constructor` when constructing to allow for future field additions:

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `max_characters` | `usize` | `1000` | Maximum size per chunk (in units determined by `sizing`). When `sizing` is `Characters` (default), this is the max character count. When using token-based sizing, this is the max token count. Default: 1000 |
| `overlap` | `usize` | `200` | Overlap between chunks (in units determined by `sizing`). Default: 200 |
| `trim` | `bool` | `true` | Whether to trim whitespace from chunk boundaries. Default: true |
| `chunker_type` | `ChunkerType` | `ChunkerType::Text` | Type of chunker to use (Text or Markdown). Default: Text |
| `embedding` | `Option<EmbeddingConfig>` | `None` | Optional embedding configuration for chunk embeddings. |
| `preset` | `Option<String>` | `None` | Use a preset configuration (overrides individual settings if provided). |
| `sizing` | `ChunkSizing` | `ChunkSizing::Characters` | How to measure chunk size. Default: `Characters` (Unicode character count). Enable `chunking-tiktoken` or `chunking-tokenizers` features for token-based sizing. |
| `prepend_heading_context` | `bool` | `false` | When `True` and `chunker_type` is `Markdown`, prepend the heading hierarchy path (e.g. `"# Title > ## Section\n\n"`) to each chunk's content string. This is useful for RAG pipelines where each chunk needs self-contained context about its position in the document structure. Default: `False` |
| `topic_threshold` | `Option<f32>` | `None` | Optional cosine similarity threshold for semantic topic boundary detection. Only used when `chunker_type` is `Semantic` and an `EmbeddingConfig` is provided. You almost never need to set this. When omitted, defaults to `0.75` which works well for most documents. Lower values detect more topic boundaries (more, smaller chunks); higher values detect fewer. Range: `0.0..=1.0`. |

---

#### EmbeddingConfig

Embedding configuration for text chunks.

Configures embedding generation using ONNX models via the vendored embedding engine.
Requires the `embeddings` feature to be enabled.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `model` | `EmbeddingModelType` | `EmbeddingModelType::Preset` | The embedding model to use (defaults to "balanced" preset if not specified) |
| `normalize` | `bool` | `true` | Whether to normalize embedding vectors (recommended for cosine similarity) |
| `batch_size` | `usize` | `32` | Batch size for embedding generation |
| `show_download_progress` | `bool` | `false` | Show model download progress |
| `cache_dir` | `Option<PathBuf>` | `None` | Custom cache directory for model files Defaults to `~/.cache/kreuzberg/embeddings/` if not specified. Allows full customization of model download location. |
| `acceleration` | `Option<AccelerationConfig>` | `None` | Hardware acceleration for the embedding ONNX model. When set, controls which execution provider (CPU, CUDA, CoreML, TensorRT) is used for inference. Defaults to `None` (auto-select per platform). |

---

#### TreeSitterConfig

Configuration for tree-sitter language pack integration.

Controls grammar download behavior and code analysis options.

# Example (TOML)

```toml
[tree_sitter]
languages = ["python", "rust"]
groups = ["web"]

[tree_sitter.process]
structure = true
comments = true
docstrings = true
```

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `enabled` | `bool` | `true` | Enable code intelligence processing (default: true). When `False`, tree-sitter analysis is completely skipped even if the config section is present. |
| `cache_dir` | `Option<PathBuf>` | `None` | Custom cache directory for downloaded grammars. When `None`, uses the default: `~/.cache/tree-sitter-language-pack/v{version}/libs/`. |
| `languages` | `Vec<String>` | `None` | Languages to pre-download on init (e.g., `["python", "rust"]`). |
| `groups` | `Vec<String>` | `None` | Language groups to pre-download (e.g., `["web", "systems", "scripting"]`). |
| `process` | `TreeSitterProcessConfig` | — | Processing options for code analysis. |

---

#### TreeSitterProcessConfig

Processing options for tree-sitter code analysis.

Controls which analysis features are enabled when extracting code files.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `structure` | `bool` | `true` | Extract structural items (functions, classes, structs, etc.). Default: true. |
| `imports` | `bool` | `true` | Extract import statements. Default: true. |
| `exports` | `bool` | `true` | Extract export statements. Default: true. |
| `comments` | `bool` | `false` | Extract comments. Default: false. |
| `docstrings` | `bool` | `false` | Extract docstrings. Default: false. |
| `symbols` | `bool` | `false` | Extract symbol definitions. Default: false. |
| `diagnostics` | `bool` | `false` | Include parse diagnostics. Default: false. |
| `chunk_max_size` | `Option<usize>` | `None` | Maximum chunk size in bytes. `None` disables chunking. |
| `content_mode` | `CodeContentMode` | `CodeContentMode::Chunks` | Content rendering mode for code extraction. |

---

#### ServerConfig

API server configuration.

This struct holds all configuration options for the Kreuzberg API server,
including host/port settings, CORS configuration, and upload limits.

# Defaults

- `host`: "127.0.0.1" (localhost only)
- `port`: 8000
- `cors_origins`: empty vector (allows all origins)
- `max_request_body_bytes`: 104_857_600 (100 MB)
- `max_multipart_field_bytes`: 104_857_600 (100 MB)

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `host` | `String` | — | Server host address (e.g., "127.0.0.1", "0.0.0.0") |
| `port` | `u16` | — | Server port number |
| `cors_origins` | `Vec<String>` | `vec![]` | CORS allowed origins. Empty vector means allow all origins. If this is an empty vector, the server will accept requests from any origin. If populated with specific origins (e.g., ["<https://example.com">]), only those origins will be allowed. |
| `max_request_body_bytes` | `usize` | — | Maximum size of request body in bytes (default: 100 MB) |
| `max_multipart_field_bytes` | `usize` | — | Maximum size of multipart fields in bytes (default: 100 MB) |

---

#### JsonExtractionConfig

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `extract_schema` | `bool` | `false` | Extract schema |
| `max_depth` | `usize` | `20` | Maximum depth |
| `array_item_limit` | `usize` | `500` | Array item limit |
| `include_type_info` | `bool` | `false` | Include type info |
| `flatten_nested_objects` | `bool` | `true` | Flatten nested objects |
| `custom_text_field_patterns` | `Vec<String>` | `vec![]` | Custom text field patterns |

---

#### HwpDocument

An extracted HWP document, consisting of one or more body-text sections.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `sections` | `Vec<Section>` | `vec![]` | All sections from all BodyText/SectionN streams. |

---

#### Section

A body-text section containing a flat list of paragraphs.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `paragraphs` | `Vec<Paragraph>` | `vec![]` | Paragraphs |

---

#### Paragraph

A single paragraph; may or may not carry a text payload.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `text` | `Option<ParaText>` | `Default::default()` | Text (para text) |

---

#### Drawing

A drawing object extracted from `<w:drawing>`.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `drawing_type` | `DrawingType` | `DrawingType::Inline` | Drawing type (drawing type) |
| `extent` | `Option<Extent>` | `Default::default()` | Extent (extent) |
| `doc_properties` | `Option<DocProperties>` | `Default::default()` | Doc properties (doc properties) |
| `image_ref` | `Option<String>` | `Default::default()` | Image ref |

---

#### Extent

Size in EMUs (English Metric Units, 1 inch = 914400 EMU).

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `cx` | `i64` | — | Cx |
| `cy` | `i64` | — | Cy |

---

#### DocProperties

Document properties from `<wp:docPr>`.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `id` | `Option<String>` | `Default::default()` | Unique identifier |
| `name` | `Option<String>` | `Default::default()` | The name |
| `description` | `Option<String>` | `Default::default()` | Human-readable description |

---

#### AnchorProperties

Properties for anchored drawings.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `behind_doc` | `bool` | — | Behind doc |
| `layout_in_cell` | `bool` | — | Layout in cell |
| `relative_height` | `Option<i64>` | `Default::default()` | Relative height |
| `position_h` | `Option<Position>` | `Default::default()` | Position h (position) |
| `position_v` | `Option<Position>` | `Default::default()` | Position v (position) |
| `wrap_type` | `WrapType` | `WrapType::None` | Wrap type (wrap type) |

---

#### Position

Horizontal or vertical position.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `relative_from` | `String` | — | Relative from |
| `offset` | `Option<i64>` | `Default::default()` | Offset |

---

#### Document

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `paragraphs` | `Vec<Paragraph>` | `vec![]` | Paragraphs |
| `tables` | `Vec<Table>` | `vec![]` | Tables extracted from the document |
| `headers` | `Vec<HeaderFooter>` | `vec![]` | Headers |
| `footers` | `Vec<HeaderFooter>` | `vec![]` | Footers |
| `footnotes` | `Vec<Note>` | `vec![]` | Footnotes |
| `endnotes` | `Vec<Note>` | `vec![]` | Endnotes |
| `numbering_defs` | `AHashMap` | — | Numbering defs (a hash map) |
| `elements` | `Vec<DocumentElement>` | `vec![]` | Document elements in their original order. |
| `style_catalog` | `Option<StyleCatalog>` | `Default::default()` | Parsed style catalog from `word/styles.xml`, if available. |
| `theme` | `Option<Theme>` | `Default::default()` | Parsed theme from `word/theme/theme1.xml`, if available. |
| `sections` | `Vec<SectionProperties>` | `vec![]` | Section properties parsed from `w:sectPr` elements. |
| `drawings` | `Vec<Drawing>` | `vec![]` | Drawing objects parsed from `w:drawing` elements. |
| `image_relationships` | `AHashMap` | — | Image relationships (rId → target path) for image extraction. |

---

#### Run

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `text` | `String` | — | Text |
| `bold` | `bool` | — | Bold |
| `italic` | `bool` | — | Italic |
| `underline` | `bool` | — | Underline |
| `strikethrough` | `bool` | — | Strikethrough |
| `subscript` | `bool` | — | Subscript |
| `superscript` | `bool` | — | Superscript |
| `font_size` | `Option<u32>` | `Default::default()` | Font size in half-points (from `w:sz`). |
| `font_color` | `Option<String>` | `Default::default()` | Font color as "RRGGBB" hex (from `w:color`). |
| `highlight` | `Option<String>` | `Default::default()` | Highlight color name (from `w:highlight`). |
| `hyperlink_url` | `Option<String>` | `Default::default()` | Hyperlink url |
| `math_latex` | `Option<(String, bool)>` | `Default::default()` | LaTeX math content: (latex_source, is_display_math). When set, this run represents an equation and `text` is ignored. |

---

#### TableRow

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `cells` | `Vec<TableCell>` | `vec![]` | Cells |
| `properties` | `Option<RowProperties>` | `Default::default()` | Properties (row properties) |

---

#### HeaderFooter

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `paragraphs` | `Vec<Paragraph>` | `vec![]` | Paragraphs |
| `tables` | `Vec<Table>` | `vec![]` | Tables extracted from the document |
| `header_type` | `HeaderFooterType` | `HeaderFooterType::Default` | Header type (header footer type) |

---

#### PageMargins

Page margins in twips (twentieths of a point).

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `top` | `Option<i32>` | `Default::default()` | Top margin in twips. |
| `right` | `Option<i32>` | `Default::default()` | Right margin in twips. |
| `bottom` | `Option<i32>` | `Default::default()` | Bottom margin in twips. |
| `left` | `Option<i32>` | `Default::default()` | Left margin in twips. |
| `header` | `Option<i32>` | `Default::default()` | Header offset in twips. |
| `footer` | `Option<i32>` | `Default::default()` | Footer offset in twips. |
| `gutter` | `Option<i32>` | `Default::default()` | Gutter margin in twips. |

---

#### PageMarginsPoints

Page margins converted to points (1/72 inch).

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `top` | `Option<f64>` | `Default::default()` | Top |
| `right` | `Option<f64>` | `Default::default()` | Right |
| `bottom` | `Option<f64>` | `Default::default()` | Bottom |
| `left` | `Option<f64>` | `Default::default()` | Left |
| `header` | `Option<f64>` | `Default::default()` | Header |
| `footer` | `Option<f64>` | `Default::default()` | Footer |
| `gutter` | `Option<f64>` | `Default::default()` | Gutter |

---

#### ColumnLayout

Column layout configuration.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `count` | `Option<i32>` | `Default::default()` | Number of columns. |
| `space_twips` | `Option<i32>` | `Default::default()` | Space between columns in twips. |
| `equal_width` | `Option<bool>` | `Default::default()` | Whether columns have equal width. |

---

#### SectionProperties

DOCX section properties parsed from `w:sectPr` element.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `page_width_twips` | `Option<i32>` | `Default::default()` | Page width in twips (from `w:pgSz w:w`). |
| `page_height_twips` | `Option<i32>` | `Default::default()` | Page height in twips (from `w:pgSz w:h`). |
| `orientation` | `Option<Orientation>` | `Default::default()` | Page orientation (from `w:pgSz w:orient`). |
| `margins` | `PageMargins` | — | Page margins (from `w:pgMar`). |
| `columns` | `ColumnLayout` | — | Column layout (from `w:cols`). |
| `doc_grid_line_pitch` | `Option<i32>` | `Default::default()` | Document grid line pitch in twips (from `w:docGrid w:linePitch`). |

---

#### RunProperties

Run-level formatting properties (bold, italic, font, size, color, etc.).

All fields are `Option` so that inheritance resolution can distinguish
"not set" (`None`) from "explicitly set" (`Some`).

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `bold` | `Option<bool>` | `Default::default()` | Bold |
| `italic` | `Option<bool>` | `Default::default()` | Italic |
| `underline` | `Option<bool>` | `Default::default()` | Underline |
| `strikethrough` | `Option<bool>` | `Default::default()` | Strikethrough |
| `color` | `Option<String>` | `Default::default()` | Hex RGB color, e.g. `"2F5496"`. |
| `font_size_half_points` | `Option<i32>` | `Default::default()` | Font size in half-points (`w:sz` val). Divide by 2 to get points. |
| `font_ascii` | `Option<String>` | `Default::default()` | ASCII font family (`w:rFonts w:ascii`). |
| `font_ascii_theme` | `Option<String>` | `Default::default()` | ASCII theme font (`w:rFonts w:asciiTheme`). |
| `vert_align` | `Option<String>` | `Default::default()` | Vertical alignment: "superscript", "subscript", or "baseline". |
| `font_h_ansi` | `Option<String>` | `Default::default()` | High ANSI font family (w:rFonts w:hAnsi). |
| `font_cs` | `Option<String>` | `Default::default()` | Complex script font family (w:rFonts w:cs). |
| `font_east_asia` | `Option<String>` | `Default::default()` | East Asian font family (w:rFonts w:eastAsia). |
| `highlight` | `Option<String>` | `Default::default()` | Highlight color name (e.g., "yellow", "green", "cyan"). |
| `caps` | `Option<bool>` | `Default::default()` | All caps text transformation. |
| `small_caps` | `Option<bool>` | `Default::default()` | Small caps text transformation. |
| `shadow` | `Option<bool>` | `Default::default()` | Text shadow effect. |
| `outline` | `Option<bool>` | `Default::default()` | Text outline effect. |
| `emboss` | `Option<bool>` | `Default::default()` | Text emboss effect. |
| `imprint` | `Option<bool>` | `Default::default()` | Text imprint (engrave) effect. |
| `char_spacing` | `Option<i32>` | `Default::default()` | Character spacing in twips (from w:spacing w:val). |
| `position` | `Option<i32>` | `Default::default()` | Vertical position offset in half-points (from w:position w:val). |
| `kern` | `Option<i32>` | `Default::default()` | Kerning threshold in half-points (from w:kern w:val). |
| `theme_color` | `Option<String>` | `Default::default()` | Theme color reference (e.g., "accent1", "dk1"). |
| `theme_tint` | `Option<String>` | `Default::default()` | Theme color tint modification (hex value). |
| `theme_shade` | `Option<String>` | `Default::default()` | Theme color shade modification (hex value). |

---

#### ParagraphProperties

Paragraph-level formatting properties (alignment, spacing, indentation, etc.).

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `alignment` | `Option<String>` | `Default::default()` | `"left"`, `"center"`, `"right"`, `"both"` (justified). |
| `spacing_before` | `Option<i32>` | `Default::default()` | Spacing before paragraph in twips. |
| `spacing_after` | `Option<i32>` | `Default::default()` | Spacing after paragraph in twips. |
| `spacing_line` | `Option<i32>` | `Default::default()` | Line spacing in twips or 240ths of a line. |
| `spacing_line_rule` | `Option<String>` | `Default::default()` | Line spacing rule: "auto", "exact", or "atLeast". |
| `indent_left` | `Option<i32>` | `Default::default()` | Left indentation in twips. |
| `indent_right` | `Option<i32>` | `Default::default()` | Right indentation in twips. |
| `indent_first_line` | `Option<i32>` | `Default::default()` | First-line indentation in twips. |
| `indent_hanging` | `Option<i32>` | `Default::default()` | Hanging indentation in twips. |
| `outline_level` | `Option<u8>` | `Default::default()` | Outline level 0-8 for heading levels. |
| `keep_next` | `Option<bool>` | `Default::default()` | Keep with next paragraph on same page. |
| `keep_lines` | `Option<bool>` | `Default::default()` | Keep all lines of paragraph on same page. |
| `page_break_before` | `Option<bool>` | `Default::default()` | Force page break before paragraph. |
| `widow_control` | `Option<bool>` | `Default::default()` | Prevent widow/orphan lines. |
| `suppress_auto_hyphens` | `Option<bool>` | `Default::default()` | Suppress automatic hyphenation. |
| `bidi` | `Option<bool>` | `Default::default()` | Right-to-left paragraph direction. |
| `shading_fill` | `Option<String>` | `Default::default()` | Background color hex value (from w:shd w:fill). |
| `shading_val` | `Option<String>` | `Default::default()` | Shading pattern value (from w:shd w:val). |
| `border_top` | `Option<String>` | `Default::default()` | Top border style (from w:pBdr/w:top w:val). |
| `border_bottom` | `Option<String>` | `Default::default()` | Bottom border style (from w:pBdr/w:bottom w:val). |
| `border_left` | `Option<String>` | `Default::default()` | Left border style (from w:pBdr/w:left w:val). |
| `border_right` | `Option<String>` | `Default::default()` | Right border style (from w:pBdr/w:right w:val). |

---

#### ResolvedStyle

Fully resolved (flattened) style after walking the inheritance chain.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `paragraph_properties` | `ParagraphProperties` | — | Paragraph properties (paragraph properties) |
| `run_properties` | `RunProperties` | — | Run properties (run properties) |

---

#### StyleCatalog

Catalog of all styles parsed from `word/styles.xml`, plus document defaults.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `styles` | `AHashMap` | — | Styles (a hash map) |
| `default_paragraph_properties` | `ParagraphProperties` | — | Default paragraph properties (paragraph properties) |
| `default_run_properties` | `RunProperties` | — | Default run properties (run properties) |

---

#### TableProperties

Table-level properties from `<w:tblPr>`.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `style_id` | `Option<String>` | `Default::default()` | Style id |
| `width` | `Option<TableWidth>` | `Default::default()` | Width (table width) |
| `alignment` | `Option<String>` | `Default::default()` | Alignment |
| `layout` | `Option<String>` | `Default::default()` | Layout |
| `look` | `Option<TableLook>` | `Default::default()` | Look (table look) |
| `borders` | `Option<TableBorders>` | `Default::default()` | Borders (table borders) |
| `cell_margins` | `Option<CellMargins>` | `Default::default()` | Cell margins (cell margins) |
| `indent` | `Option<TableWidth>` | `Default::default()` | Indent (table width) |
| `caption` | `Option<String>` | `Default::default()` | Caption |

---

#### TableLook

Table look bitmask/flags controlling conditional formatting bands.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `first_row` | `bool` | — | First row |
| `last_row` | `bool` | — | Last row |
| `first_column` | `bool` | — | First column |
| `last_column` | `bool` | — | Last column |
| `no_h_band` | `bool` | — | No h band |
| `no_v_band` | `bool` | — | No v band |

---

#### TableBorders

Borders for a table (6 borders: top, bottom, left, right, insideH, insideV).

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `top` | `Option<BorderStyle>` | `Default::default()` | Top (border style) |
| `bottom` | `Option<BorderStyle>` | `Default::default()` | Bottom (border style) |
| `left` | `Option<BorderStyle>` | `Default::default()` | Left (border style) |
| `right` | `Option<BorderStyle>` | `Default::default()` | Right (border style) |
| `inside_h` | `Option<BorderStyle>` | `Default::default()` | Inside h (border style) |
| `inside_v` | `Option<BorderStyle>` | `Default::default()` | Inside v (border style) |

---

#### CellMargins

Cell margins (used for both table-level defaults and per-cell overrides).

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `top` | `Option<i32>` | `Default::default()` | Top |
| `bottom` | `Option<i32>` | `Default::default()` | Bottom |
| `left` | `Option<i32>` | `Default::default()` | Left |
| `right` | `Option<i32>` | `Default::default()` | Right |

---

#### RowProperties

Row-level properties from `<w:trPr>`.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `height` | `Option<i32>` | `Default::default()` | Height |
| `height_rule` | `Option<String>` | `Default::default()` | Height rule |
| `is_header` | `bool` | — | Whether header |
| `cant_split` | `bool` | — | Cant split |

---

#### CellProperties

Cell-level properties from `<w:tcPr>`.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `width` | `Option<TableWidth>` | `Default::default()` | Width (table width) |
| `grid_span` | `Option<u32>` | `Default::default()` | Grid span |
| `v_merge` | `Option<VerticalMerge>` | `Default::default()` | V merge (vertical merge) |
| `borders` | `Option<CellBorders>` | `Default::default()` | Borders (cell borders) |
| `shading` | `Option<CellShading>` | `Default::default()` | Shading (cell shading) |
| `margins` | `Option<CellMargins>` | `Default::default()` | Margins (cell margins) |
| `vertical_align` | `Option<String>` | `Default::default()` | Vertical align |
| `text_direction` | `Option<String>` | `Default::default()` | Text direction |
| `no_wrap` | `bool` | — | No wrap |

---

#### CellBorders

Per-cell borders (4 sides).

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `top` | `Option<BorderStyle>` | `Default::default()` | Top (border style) |
| `bottom` | `Option<BorderStyle>` | `Default::default()` | Bottom (border style) |
| `left` | `Option<BorderStyle>` | `Default::default()` | Left (border style) |
| `right` | `Option<BorderStyle>` | `Default::default()` | Right (border style) |

---

#### CellShading

Cell shading/background.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `fill` | `Option<String>` | `Default::default()` | Fill |
| `color` | `Option<String>` | `Default::default()` | Color |
| `val` | `Option<String>` | `Default::default()` | Val |

---

#### ColorScheme

Color scheme containing all 12 standard Office theme colors.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `name` | `String` | — | Color scheme name. |
| `dk1` | `Option<ThemeColor>` | `Default::default()` | Dark 1 (dark background) color. |
| `lt1` | `Option<ThemeColor>` | `Default::default()` | Light 1 (light background) color. |
| `dk2` | `Option<ThemeColor>` | `Default::default()` | Dark 2 color. |
| `lt2` | `Option<ThemeColor>` | `Default::default()` | Light 2 color. |
| `accent1` | `Option<ThemeColor>` | `Default::default()` | Accent color 1. |
| `accent2` | `Option<ThemeColor>` | `Default::default()` | Accent color 2. |
| `accent3` | `Option<ThemeColor>` | `Default::default()` | Accent color 3. |
| `accent4` | `Option<ThemeColor>` | `Default::default()` | Accent color 4. |
| `accent5` | `Option<ThemeColor>` | `Default::default()` | Accent color 5. |
| `accent6` | `Option<ThemeColor>` | `Default::default()` | Accent color 6. |
| `hlink` | `Option<ThemeColor>` | `Default::default()` | Hyperlink color. |
| `fol_hlink` | `Option<ThemeColor>` | `Default::default()` | Followed hyperlink color. |

---

#### FontScheme

Font scheme containing major (heading) and minor (body) fonts.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `name` | `String` | — | Font scheme name. |
| `major_latin` | `Option<String>` | `Default::default()` | Major (heading) font - Latin script. |
| `major_east_asian` | `Option<String>` | `Default::default()` | Major (heading) font - East Asian script. |
| `major_complex_script` | `Option<String>` | `Default::default()` | Major (heading) font - Complex script. |
| `minor_latin` | `Option<String>` | `Default::default()` | Minor (body) font - Latin script. |
| `minor_east_asian` | `Option<String>` | `Default::default()` | Minor (body) font - East Asian script. |
| `minor_complex_script` | `Option<String>` | `Default::default()` | Minor (body) font - Complex script. |

---

#### Theme

Complete theme with color scheme and font scheme.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `name` | `String` | — | Theme name (e.g., "Office Theme"). |
| `color_scheme` | `Option<ColorScheme>` | `Default::default()` | Color scheme (12 standard colors). |
| `font_scheme` | `Option<FontScheme>` | `Default::default()` | Font scheme (major and minor fonts). |

---

#### DocxAppProperties

Application properties from docProps/app.xml for DOCX

Contains Word-specific document statistics and metadata.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `application` | `Option<String>` | `Default::default()` | Application name (e.g., "Microsoft Office Word") |
| `app_version` | `Option<String>` | `Default::default()` | Application version |
| `template` | `Option<String>` | `Default::default()` | Template filename |
| `total_time` | `Option<i32>` | `Default::default()` | Total editing time in minutes |
| `pages` | `Option<i32>` | `Default::default()` | Number of pages |
| `words` | `Option<i32>` | `Default::default()` | Number of words |
| `characters` | `Option<i32>` | `Default::default()` | Number of characters (excluding spaces) |
| `characters_with_spaces` | `Option<i32>` | `Default::default()` | Number of characters (including spaces) |
| `lines` | `Option<i32>` | `Default::default()` | Number of lines |
| `paragraphs` | `Option<i32>` | `Default::default()` | Number of paragraphs |
| `company` | `Option<String>` | `Default::default()` | Company name |
| `doc_security` | `Option<i32>` | `Default::default()` | Document security level |
| `scale_crop` | `Option<bool>` | `Default::default()` | Scale crop flag |
| `links_up_to_date` | `Option<bool>` | `Default::default()` | Links up to date flag |
| `shared_doc` | `Option<bool>` | `Default::default()` | Shared document flag |
| `hyperlinks_changed` | `Option<bool>` | `Default::default()` | Hyperlinks changed flag |

---

#### XlsxAppProperties

Application properties from docProps/app.xml for XLSX

Contains Excel-specific document metadata.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `application` | `Option<String>` | `Default::default()` | Application name (e.g., "Microsoft Excel") |
| `app_version` | `Option<String>` | `Default::default()` | Application version |
| `doc_security` | `Option<i32>` | `Default::default()` | Document security level |
| `scale_crop` | `Option<bool>` | `Default::default()` | Scale crop flag |
| `links_up_to_date` | `Option<bool>` | `Default::default()` | Links up to date flag |
| `shared_doc` | `Option<bool>` | `Default::default()` | Shared document flag |
| `hyperlinks_changed` | `Option<bool>` | `Default::default()` | Hyperlinks changed flag |
| `company` | `Option<String>` | `Default::default()` | Company name |
| `worksheet_names` | `Vec<String>` | `vec![]` | Worksheet names |

---

#### PptxAppProperties

Application properties from docProps/app.xml for PPTX

Contains PowerPoint-specific document metadata.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `application` | `Option<String>` | `Default::default()` | Application name (e.g., "Microsoft Office PowerPoint") |
| `app_version` | `Option<String>` | `Default::default()` | Application version |
| `total_time` | `Option<i32>` | `Default::default()` | Total editing time in minutes |
| `company` | `Option<String>` | `Default::default()` | Company name |
| `doc_security` | `Option<i32>` | `Default::default()` | Document security level |
| `scale_crop` | `Option<bool>` | `Default::default()` | Scale crop flag |
| `links_up_to_date` | `Option<bool>` | `Default::default()` | Links up to date flag |
| `shared_doc` | `Option<bool>` | `Default::default()` | Shared document flag |
| `hyperlinks_changed` | `Option<bool>` | `Default::default()` | Hyperlinks changed flag |
| `slides` | `Option<i32>` | `Default::default()` | Number of slides |
| `notes` | `Option<i32>` | `Default::default()` | Number of notes |
| `hidden_slides` | `Option<i32>` | `Default::default()` | Number of hidden slides |
| `multimedia_clips` | `Option<i32>` | `Default::default()` | Number of multimedia clips |
| `presentation_format` | `Option<String>` | `Default::default()` | Presentation format (e.g., "Widescreen", "Standard") |
| `slide_titles` | `Vec<String>` | `vec![]` | Slide titles |

---

#### CoreProperties

Dublin Core metadata from docProps/core.xml

Contains standard metadata fields defined by the Dublin Core standard
and Office-specific extensions.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `title` | `Option<String>` | `Default::default()` | Document title |
| `subject` | `Option<String>` | `Default::default()` | Document subject/topic |
| `creator` | `Option<String>` | `Default::default()` | Document creator/author |
| `keywords` | `Option<String>` | `Default::default()` | Keywords or tags |
| `description` | `Option<String>` | `Default::default()` | Document description/abstract |
| `last_modified_by` | `Option<String>` | `Default::default()` | User who last modified the document |
| `revision` | `Option<String>` | `Default::default()` | Revision number |
| `created` | `Option<String>` | `Default::default()` | Creation timestamp (ISO 8601) |
| `modified` | `Option<String>` | `Default::default()` | Last modification timestamp (ISO 8601) |
| `category` | `Option<String>` | `Default::default()` | Document category |
| `content_status` | `Option<String>` | `Default::default()` | Content status (Draft, Final, etc.) |
| `language` | `Option<String>` | `Default::default()` | Document language |
| `identifier` | `Option<String>` | `Default::default()` | Unique identifier |
| `version` | `Option<String>` | `Default::default()` | Document version |
| `last_printed` | `Option<String>` | `Default::default()` | Last print timestamp (ISO 8601) |

---

#### OdtProperties

OpenDocument metadata from meta.xml

Contains metadata fields defined by the OASIS OpenDocument Format standard.
Uses Dublin Core elements (dc:) and OpenDocument meta elements (meta:).

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `title` | `Option<String>` | `Default::default()` | Document title (dc:title) |
| `subject` | `Option<String>` | `Default::default()` | Document subject/topic (dc:subject) |
| `creator` | `Option<String>` | `Default::default()` | Current document creator/author (dc:creator) |
| `initial_creator` | `Option<String>` | `Default::default()` | Initial creator of the document (meta:initial-creator) |
| `keywords` | `Option<String>` | `Default::default()` | Keywords or tags (meta:keyword) |
| `description` | `Option<String>` | `Default::default()` | Document description (dc:description) |
| `date` | `Option<String>` | `Default::default()` | Current modification date (dc:date) |
| `creation_date` | `Option<String>` | `Default::default()` | Initial creation date (meta:creation-date) |
| `language` | `Option<String>` | `Default::default()` | Document language (dc:language) |
| `generator` | `Option<String>` | `Default::default()` | Generator/application that created the document (meta:generator) |
| `editing_duration` | `Option<String>` | `Default::default()` | Editing duration in ISO 8601 format (meta:editing-duration) |
| `editing_cycles` | `Option<String>` | `Default::default()` | Number of edits/revisions (meta:editing-cycles) |
| `page_count` | `Option<i32>` | `Default::default()` | Document statistics - page count (meta:page-count) |
| `word_count` | `Option<i32>` | `Default::default()` | Document statistics - word count (meta:word-count) |
| `character_count` | `Option<i32>` | `Default::default()` | Document statistics - character count (meta:character-count) |
| `paragraph_count` | `Option<i32>` | `Default::default()` | Document statistics - paragraph count (meta:paragraph-count) |
| `table_count` | `Option<i32>` | `Default::default()` | Document statistics - table count (meta:table-count) |
| `image_count` | `Option<i32>` | `Default::default()` | Document statistics - image count (meta:image-count) |

---

#### PptxExtractionOptions

Options for PPTX content extraction.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `extract_images` | `bool` | `true` | Whether to extract embedded images. |
| `page_config` | `Option<PageConfig>` | `None` | Optional page configuration for boundary tracking. |
| `plain` | `bool` | `false` | Whether to output plain text (no markdown). |
| `include_structure` | `bool` | `false` | Whether to build the `DocumentStructure` tree. |
| `inject_placeholders` | `bool` | `true` | Whether to emit `![alt](target)` references in markdown output. |

---

#### CodeExtractor

Source code extractor using tree-sitter language pack.

Detects the programming language from the file extension or shebang line,
then uses tree-sitter to parse and extract structural information.

*Opaque type — fields are not directly accessible.*

---

#### CsvExtractor

CSV/TSV extractor with proper field parsing.

Replaces raw text passthrough with structured CSV parsing,
producing space-separated text output and populated `tables` field.

*Opaque type — fields are not directly accessible.*

---

#### StructuredExtractor

Structured data extractor supporting JSON, JSONL/NDJSON, YAML, and TOML.

*Opaque type — fields are not directly accessible.*

---

#### PlainTextExtractor

Plain text extractor.

Extracts content from plain text files (.txt).

*Opaque type — fields are not directly accessible.*

---

#### DjotExtractor

Djot markup extractor with metadata and table support.

Parses Djot documents with YAML frontmatter, extracting:
- Metadata from YAML frontmatter
- Plain text content
- Tables as structured data
- Document structure (headings, links, code blocks)

*Opaque type — fields are not directly accessible.*

---

#### SecurityLimits

Configuration for security limits across extractors.

All limits are intentionally conservative to prevent DoS attacks
while still supporting legitimate documents.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `max_archive_size` | `usize` | — | Maximum uncompressed size for archives (500 MB) |
| `max_compression_ratio` | `usize` | `100` | Maximum compression ratio before flagging as potential bomb (100:1) |
| `max_files_in_archive` | `usize` | `10000` | Maximum number of files in archive (10,000) |
| `max_nesting_depth` | `usize` | `100` | Maximum nesting depth for structures (100) |
| `max_entity_length` | `usize` | `32` | Maximum entity/string length (32) |
| `max_content_size` | `usize` | — | Maximum string growth per document (100 MB) |
| `max_iterations` | `usize` | `10000000` | Maximum iterations per operation |
| `max_xml_depth` | `usize` | `100` | Maximum XML depth (100 levels) |
| `max_table_cells` | `usize` | `100000` | Maximum cells per table (100,000) |

---

#### ImageExtractor

Image extractor for various image formats.

Supports: PNG, JPEG, WebP, BMP, TIFF, GIF.
Extracts dimensions, format, and EXIF metadata.
Optionally runs OCR when configured.
When layout detection is also enabled, uses per-region OCR with
markdown formatting based on detected layout classes.

*Opaque type — fields are not directly accessible.*

---

#### ZipExtractor

ZIP archive extractor.

Extracts file lists and text content from ZIP archives.

*Opaque type — fields are not directly accessible.*

---

#### TarExtractor

TAR archive extractor.

Extracts file lists and text content from TAR archives.

*Opaque type — fields are not directly accessible.*

---

#### SevenZExtractor

7z archive extractor.

Extracts file lists and text content from 7z archives.

*Opaque type — fields are not directly accessible.*

---

#### GzipExtractor

Gzip archive extractor.

Decompresses gzip files and extracts text content from the compressed data.

*Opaque type — fields are not directly accessible.*

---

#### EmailExtractor

Email message extractor.

Supports: .eml, .msg

*Opaque type — fields are not directly accessible.*

---

#### PstExtractor

PST file extractor.

Supports: .pst (Microsoft Outlook Personal Folders)

*Opaque type — fields are not directly accessible.*

---

#### ExcelExtractor

Excel spreadsheet extractor using calamine.

Supports: .xlsx, .xlsm, .xlam, .xltm, .xls, .xla, .xlsb, .ods

# Limitations

- **Hyperlinks**: calamine (v0.34) does not expose cell hyperlink data in its
  public API. Excel files may contain hyperlinks via the `HYPERLINK()` formula
  or via the relationships XML, but neither is accessible through the crate.
  This would require either a calamine upstream change or manual OOXML parsing.

*Opaque type — fields are not directly accessible.*

---

#### HwpExtractor

Extractor for Hangul Word Processor (.hwp) files.

Supports HWP 5.0 format, the standard document format in South Korea.

*Opaque type — fields are not directly accessible.*

---

#### KeynoteExtractor

Apple Keynote presentation extractor.

Supports `.key` files (modern iWork format, 2013+).

Extracts slide text and speaker notes from the IWA container:
ZIP → Snappy → protobuf text fields.

*Opaque type — fields are not directly accessible.*

---

#### NumbersExtractor

Apple Numbers spreadsheet extractor.

Supports `.numbers` files (modern iWork format, 2013+).

Extracts cell string values and sheet names from the IWA container:
ZIP → Snappy → protobuf text fields. Output is formatted as plain text
with one text token per line (representing cell values and labels).

*Opaque type — fields are not directly accessible.*

---

#### PagesExtractor

Apple Pages document extractor.

Supports `.pages` files (modern iWork format, 2013+).

Extracts all text content from the document by parsing the IWA
(iWork Archive) container: ZIP → Snappy → protobuf text fields.

*Opaque type — fields are not directly accessible.*

---

#### HtmlExtractor

HTML document extractor using html-to-markdown.

*Opaque type — fields are not directly accessible.*

---

#### BibtexExtractor

BibTeX bibliography extractor.

Parses BibTeX files and extracts structured bibliography data including
entries, authors, publication years, and entry type distribution.

*Opaque type — fields are not directly accessible.*

---

#### CitationExtractor

Citation format extractor for RIS, PubMed/MEDLINE, and EndNote XML formats.

Parses citation files and extracts structured bibliography data including
entries, authors, publication years, and format-specific metadata.

*Opaque type — fields are not directly accessible.*

---

#### DocExtractor

Native DOC extractor using OLE/CFB parsing.

This extractor handles Word 97-2003 binary (.doc) files without
requiring LibreOffice, providing ~50x faster extraction.

*Opaque type — fields are not directly accessible.*

---

#### DbfExtractor

Extractor for dBASE (.dbf) files.

Reads all records and formats them as a markdown table with
column headers derived from field names.

*Opaque type — fields are not directly accessible.*

---

#### DocxExtractor

High-performance DOCX extractor.

This extractor provides:
- Fast text extraction via streaming XML parsing
- Comprehensive metadata extraction (core.xml, app.xml, custom.xml)

*Opaque type — fields are not directly accessible.*

---

#### EpubExtractor

EPUB format extractor using permissive-licensed dependencies.

Extracts content and metadata from EPUB files (both EPUB2 and EPUB3)
using native Rust parsing without GPL-licensed dependencies.

*Opaque type — fields are not directly accessible.*

---

#### FictionBookExtractor

FictionBook document extractor.

Supports FictionBook 2.0 format with proper section hierarchy and inline formatting.

*Opaque type — fields are not directly accessible.*

---

#### MarkdownExtractor

Markdown extractor with metadata and table support.

Parses markdown documents with YAML frontmatter, extracting:
- Metadata from YAML frontmatter
- Plain text content
- Tables as structured data
- Document structure (headings, links, code blocks)
- Images from data URIs

*Opaque type — fields are not directly accessible.*

---

#### MdxExtractor

MDX extractor with JSX stripping and Markdown processing.

Strips MDX-specific syntax (imports, exports, JSX component tags,
inline expressions) and processes the remaining content as Markdown,
extracting metadata from YAML frontmatter and tables.

*Opaque type — fields are not directly accessible.*

---

#### RstExtractor

Native Rust reStructuredText extractor.

Parses RST documents using document tree parsing and extracts:
- Metadata from field lists
- Document structure (headings, sections)
- Text content and inline formatting
- Code blocks and directives
- Tables and lists

*Opaque type — fields are not directly accessible.*

---

#### LatexExtractor

LaTeX document extractor

*Opaque type — fields are not directly accessible.*

---

#### JupyterExtractor

Jupyter Notebook extractor.

Extracts content from Jupyter notebook JSON files, including:
- Notebook metadata (kernel, language, nbformat version)
- Cell content (code and markdown)
- Cell outputs (text, HTML, etc.)
- Cell-level metadata (tags, execution counts)

*Opaque type — fields are not directly accessible.*

---

#### OrgModeExtractor

Org Mode document extractor.

Provides native Rust-based Org Mode extraction using the `org` library,
extracting structured content and metadata.

*Opaque type — fields are not directly accessible.*

---

#### OdtExtractor

High-performance ODT extractor using native Rust XML parsing.

This extractor provides:
- Fast text extraction via roxmltree XML parsing
- Comprehensive metadata extraction from meta.xml
- Table extraction with row and cell support
- Formatting preservation (bold, italic, strikeout)
- Support for headings, paragraphs, and special elements

*Opaque type — fields are not directly accessible.*

---

#### OpmlExtractor

OPML format extractor.

Extracts outline structure and metadata from OPML documents using native Rust parsing.

*Opaque type — fields are not directly accessible.*

---

#### TypstExtractor

Typst document extractor

*Opaque type — fields are not directly accessible.*

---

#### JatsExtractor

JATS document extractor.

Supports JATS (Journal Article Tag Suite) XML documents in various versions,
handling both the full article structure and minimal JATS subsets.

*Opaque type — fields are not directly accessible.*

---

#### NativeTextStats

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `non_whitespace` | `usize` | — | Non whitespace |
| `alnum` | `usize` | — | Alnum |
| `meaningful_words` | `usize` | — | Meaningful words |
| `alnum_ratio` | `f64` | — | Alnum ratio |
| `garbage_char_count` | `usize` | — | Count of Unicode replacement characters (U+FFFD) indicating encoding failures. |
| `fragmented_word_ratio` | `f64` | — | Fraction of whitespace-delimited words that are 1-2 characters (0.0-1.0). High values indicate fragmented/garbled text extraction. |
| `consecutive_repeat_ratio` | `f64` | — | Fraction of consecutive word pairs that are identical (0.0-1.0). High values indicate column scrambling where text is duplicated. |
| `avg_word_length` | `f64` | — | Average word length (by chars). Very low values indicate garbled extraction. |
| `word_count` | `usize` | — | Total word count (whitespace-delimited). |

---

#### PdfExtractor

PDF document extractor using pypdfium2 and playa-pdf.

*Opaque type — fields are not directly accessible.*

---

#### PptExtractor

Native PPT extractor using OLE/CFB parsing.

This extractor handles PowerPoint 97-2003 binary (.ppt) files without
requiring LibreOffice, providing ~50x faster extraction.

*Opaque type — fields are not directly accessible.*

---

#### PptxExtractor

PowerPoint presentation extractor.

Supports: .pptx, .pptm, .ppsx

*Opaque type — fields are not directly accessible.*

---

#### RtfExtractor

Native Rust RTF extractor.

Extracts text content, metadata, and structure from RTF documents

*Opaque type — fields are not directly accessible.*

---

#### XmlExtractor

XML extractor.

Extracts text content from XML files, preserving element structure information.

*Opaque type — fields are not directly accessible.*

---

#### DocbookExtractor

DocBook document extractor.

Supports both DocBook 4.x (no namespace) and 5.x (with namespace) formats.

*Opaque type — fields are not directly accessible.*

---

#### DocumentExtractorRegistry

Registry for document extractor plugins.

Manages extractors with MIME type and priority-based selection.

# Thread Safety

The registry is thread-safe and can be accessed concurrently from multiple threads.

*Opaque type — fields are not directly accessible.*

---

#### OcrBackendRegistry

Registry for OCR backend plugins.

Manages OCR backends with backend type and language-based selection.

# Thread Safety

The registry is thread-safe and can be accessed concurrently from multiple threads.

*Opaque type — fields are not directly accessible.*

---

#### PostProcessorRegistry

Registry for post-processor plugins.

Manages post-processors organized by processing stage.

*Opaque type — fields are not directly accessible.*

---

#### RendererRegistry

Registry for document renderer plugins.

Manages renderers that convert `InternalDocument` to output format strings.

# Thread Safety

The registry is thread-safe and can be accessed concurrently from multiple threads.

*Opaque type — fields are not directly accessible.*

---

#### ValidatorRegistry

Registry for validator plugins.

Manages validators with priority-based execution order.

*Opaque type — fields are not directly accessible.*

---

#### TokenReductionConfig

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `level` | `ReductionLevel` | `ReductionLevel::Moderate` | Level (reduction level) |
| `language_hint` | `Option<String>` | `None` | Language hint |
| `preserve_markdown` | `bool` | `false` | Preserve markdown |
| `preserve_code` | `bool` | `true` | Preserve code |
| `semantic_threshold` | `f32` | `0.3` | Semantic threshold |
| `enable_parallel` | `bool` | `true` | Enable parallel |
| `use_simd` | `bool` | `true` | Use simd |
| `custom_stopwords` | `HashMap<String, Vec<String>>` | `None` | Custom stopwords |
| `preserve_patterns` | `Vec<String>` | `vec![]` | Preserve patterns |
| `target_reduction` | `Option<f32>` | `None` | Target reduction |
| `enable_semantic_clustering` | `bool` | `false` | Enable semantic clustering |

---

#### DocumentStructureBuilder

Builder for constructing `DocumentStructure` trees with automatic
heading-driven section nesting.

The builder maintains an internal section stack: when you push a heading,
it automatically creates a `Group` container and nests subsequent content
under it. Higher-level headings pop deeper sections off the stack.

*Opaque type — fields are not directly accessible.*

---

#### Attributes

Element attributes in Djot.

Represents the attributes attached to elements using {.class #id key="value"} syntax.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `id` | `Option<String>` | `Default::default()` | Element ID (#identifier) |
| `classes` | `Vec<String>` | `vec![]` | CSS classes (.class1 .class2) |
| `key_values` | `Vec<(String, String)>` | `vec![]` | Key-value pairs (key="value") |

---

#### NodeId

Deterministic node identifier.

Generated from a hash of `node_type + text + page`. The same document
always produces the same IDs, making them useful for diffing, caching,
and external references.

*Opaque type — fields are not directly accessible.*

---

#### DocumentStructure

Top-level structured document representation.

A flat array of nodes with index-based parent/child references forming a tree.
Root-level nodes have `parent: None`. Use `body_roots()` and `furniture_roots()`
to iterate over top-level content by layer.

# Validation

Call `validate()` after construction to verify all node indices are in bounds
and parent-child relationships are bidirectionally consistent.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `nodes` | `Vec<DocumentNode>` | `vec![]` | All nodes in document/reading order. |
| `source_format` | `Option<String>` | `Default::default()` | Origin format identifier (e.g. "docx", "pptx", "html", "pdf"). Allows renderers to apply format-aware heuristics when converting the document tree to output formats. |
| `relationships` | `Vec<DocumentRelationship>` | `vec![]` | Resolved relationships between nodes (footnote refs, citations, anchor links, etc.). Populated during derivation from the internal document representation. Empty when no relationships are detected. |

---

#### TableGrid

Structured table grid with cell-level metadata.

Stores row/column dimensions and a flat list of cells with position info.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `rows` | `u32` | — | Number of rows in the table. |
| `cols` | `u32` | — | Number of columns in the table. |
| `cells` | `Vec<GridCell>` | `vec![]` | All cells in row-major order. |

---

#### LlmUsage

Token usage and cost data for a single LLM call made during extraction.

Populated when VLM OCR, structured extraction, or LLM-based embeddings
are used. Multiple entries may be present when multiple LLM calls occur
within one extraction (e.g. VLM OCR + structured extraction).

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `model` | `String` | — | The LLM model identifier (e.g. "openai/gpt-4o", "anthropic/claude-sonnet-4-20250514"). |
| `source` | `String` | — | The pipeline stage that triggered this LLM call (e.g. "vlm_ocr", "structured_extraction", "embeddings"). |
| `input_tokens` | `Option<u64>` | `Default::default()` | Number of input/prompt tokens consumed. |
| `output_tokens` | `Option<u64>` | `Default::default()` | Number of output/completion tokens generated. |
| `total_tokens` | `Option<u64>` | `Default::default()` | Total tokens (input + output). |
| `estimated_cost` | `Option<f64>` | `Default::default()` | Estimated cost in USD based on the provider's published pricing. |
| `finish_reason` | `Option<String>` | `Default::default()` | Why the model stopped generating (e.g. "stop", "length", "content_filter"). |

---

#### ElementId

Unique identifier for semantic elements.

Wraps a string identifier that is deterministically generated
from element type, content, and page number.

*Opaque type — fields are not directly accessible.*

---

#### BoundingBox

Bounding box coordinates for element positioning.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `x0` | `f64` | — | Left x-coordinate |
| `y0` | `f64` | — | Bottom y-coordinate |
| `x1` | `f64` | — | Right x-coordinate |
| `y1` | `f64` | — | Top y-coordinate |

---

#### ImagePreprocessingConfig

Image preprocessing configuration for OCR.

These settings control how images are preprocessed before OCR to improve
text recognition quality. Different preprocessing strategies work better
for different document types.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `target_dpi` | `i32` | `300` | Target DPI for the image (300 is standard, 600 for small text). |
| `auto_rotate` | `bool` | `true` | Auto-detect and correct image rotation. |
| `deskew` | `bool` | `true` | Correct skew (tilted images). |
| `denoise` | `bool` | `false` | Remove noise from the image. |
| `contrast_enhance` | `bool` | `false` | Enhance contrast for better text visibility. |
| `binarization_method` | `String` | `"otsu"` | Binarization method: "otsu", "sauvola", "adaptive". |
| `invert_colors` | `bool` | `false` | Invert colors (white text on black → black on white). |

---

#### TesseractConfig

Tesseract OCR configuration.

Provides fine-grained control over Tesseract OCR engine parameters.
Most users can use the defaults, but these settings allow optimization
for specific document types (invoices, handwriting, etc.).

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `language` | `String` | `"eng"` | Language code (e.g., "eng", "deu", "fra") |
| `psm` | `i32` | `3` | Page Segmentation Mode (0-13). Common values: - 3: Fully automatic page segmentation (default) - 6: Assume a single uniform block of text - 11: Sparse text with no particular order |
| `output_format` | `String` | `"markdown"` | Output format ("text" or "markdown") |
| `oem` | `i32` | `3` | OCR Engine Mode (0-3). - 0: Legacy engine only - 1: Neural nets (LSTM) only (usually best) - 2: Legacy + LSTM - 3: Default (based on what's available) |
| `min_confidence` | `f64` | `0` | Minimum confidence threshold (0.0-100.0). Words with confidence below this threshold may be rejected or flagged. |
| `preprocessing` | `Option<ImagePreprocessingConfig>` | `None` | Image preprocessing configuration. Controls how images are preprocessed before OCR. Can significantly improve quality for scanned documents or low-quality images. |
| `enable_table_detection` | `bool` | `true` | Enable automatic table detection and reconstruction |
| `table_min_confidence` | `f64` | `0` | Minimum confidence threshold for table detection (0.0-1.0) |
| `table_column_threshold` | `i32` | `50` | Column threshold for table detection (pixels) |
| `table_row_threshold_ratio` | `f64` | `0.5` | Row threshold ratio for table detection (0.0-1.0) |
| `use_cache` | `bool` | `true` | Enable OCR result caching |
| `classify_use_pre_adapted_templates` | `bool` | `true` | Use pre-adapted templates for character classification |
| `language_model_ngram_on` | `bool` | `false` | Enable N-gram language model |
| `tessedit_dont_blkrej_good_wds` | `bool` | `true` | Don't reject good words during block-level processing |
| `tessedit_dont_rowrej_good_wds` | `bool` | `true` | Don't reject good words during row-level processing |
| `tessedit_enable_dict_correction` | `bool` | `true` | Enable dictionary correction |
| `tessedit_char_whitelist` | `String` | `""` | Whitelist of allowed characters (empty = all allowed) |
| `tessedit_char_blacklist` | `String` | `""` | Blacklist of forbidden characters (empty = none forbidden) |
| `tessedit_use_primary_params_model` | `bool` | `true` | Use primary language params model |
| `textord_space_size_is_variable` | `bool` | `true` | Variable-width space detection |
| `thresholding_method` | `bool` | `false` | Use adaptive thresholding method |

---

#### ImageDpiConfig

Image extraction DPI configuration (internal use).

**Note:** This is an internal type used for image preprocessing.
For the main extraction configuration, see `crate.core.config.ExtractionConfig`.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `target_dpi` | `i32` | `300` | Target DPI for image normalization |
| `max_image_dimension` | `i32` | `4096` | Maximum image dimension (width or height) |
| `auto_adjust_dpi` | `bool` | `true` | Whether to auto-adjust DPI based on content |
| `min_dpi` | `i32` | `72` | Minimum DPI threshold |
| `max_dpi` | `i32` | `600` | Maximum DPI threshold |

---

#### OcrConfidence

Confidence scores for an OCR element.

Separates detection confidence (how confident that text exists at this location)
from recognition confidence (how confident about the actual text content).

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `detection` | `Option<f64>` | `Default::default()` | Detection confidence: how confident the OCR engine is that text exists here. PaddleOCR provides this as `box_score`, Tesseract doesn't have a direct equivalent. Range: 0.0 to 1.0 (or None if not available). |
| `recognition` | `f64` | — | Recognition confidence: how confident about the text content. Range: 0.0 to 1.0. |

---

#### OcrElement

A unified OCR element representing detected text with full metadata.

This is the primary type for structured OCR output, preserving all information
from both Tesseract and PaddleOCR backends.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `text` | `String` | — | The recognized text content. |
| `geometry` | `OcrBoundingGeometry` | `OcrBoundingGeometry::Rectangle` | Bounding geometry (rectangle or quadrilateral). |
| `confidence` | `OcrConfidence` | — | Confidence scores for detection and recognition. |
| `level` | `OcrElementLevel` | `OcrElementLevel::Line` | Hierarchical level (word, line, block, page). |
| `rotation` | `Option<OcrRotation>` | `Default::default()` | Rotation information (if detected). |
| `page_number` | `usize` | — | Page number (1-indexed). |
| `parent_id` | `Option<String>` | `Default::default()` | Parent element ID for hierarchical relationships. Only used for Tesseract output which has word -> line -> block hierarchy. |
| `backend_metadata` | `HashMap<String, serde_json::Value>` | `HashMap::new()` | Backend-specific metadata that doesn't fit the unified schema. |

---

#### OcrElementConfig

Configuration for OCR element extraction.

Controls how OCR elements are extracted and filtered.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `include_elements` | `bool` | — | Whether to include OCR elements in the extraction result. When true, the `ocr_elements` field in `ExtractionResult` will be populated. |
| `min_level` | `OcrElementLevel` | `OcrElementLevel::Line` | Minimum hierarchical level to include. Elements below this level (e.g., words when min_level is Line) will be excluded. |
| `min_confidence` | `f64` | — | Minimum recognition confidence threshold (0.0-1.0). Elements with confidence below this threshold will be filtered out. |
| `build_hierarchy` | `bool` | — | Whether to build hierarchical relationships between elements. When true, `parent_id` fields will be populated based on spatial containment. Only meaningful for Tesseract output. |

---

#### LayoutRegion

A detected layout region on a page.

When layout detection is enabled, each page may have layout regions
identifying different content types (text, pictures, tables, etc.)
with confidence scores and spatial positions.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `class` | `String` | — | Layout class name (e.g. "picture", "table", "text", "section_header"). |
| `confidence` | `f64` | — | Confidence score from the layout detection model (0.0 to 1.0). |
| `bounding_box` | `BoundingBox` | — | Bounding box in document coordinate space. |
| `area_fraction` | `f64` | — | Fraction of the page area covered by this region (0.0 to 1.0). |

---

#### Table

Extracted table structure.

Represents a table detected and extracted from a document (PDF, image, etc.).
Tables are converted to both structured cell data and Markdown format.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `cells` | `Vec<Vec<String>>` | `vec![]` | Table cells as a 2D vector (rows × columns) |
| `markdown` | `String` | — | Markdown representation of the table |
| `page_number` | `usize` | — | Page number where the table was found (1-indexed) |
| `bounding_box` | `Option<BoundingBox>` | `Default::default()` | Bounding box of the table on the page (PDF coordinates: x0=left, y0=bottom, x1=right, y1=top). Only populated for PDF-extracted tables when position data is available. |

---

#### TableCell

Individual table cell with content and optional styling.

Future extension point for rich table support with cell-level metadata.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `content` | `String` | — | Cell content as text |
| `row_span` | `usize` | — | Row span (number of rows this cell spans) |
| `col_span` | `usize` | — | Column span (number of columns this cell spans) |
| `is_header` | `bool` | — | Whether this is a header cell |

---

#### PoolMetrics

Metrics tracking for pool allocations and reuse patterns.

These metrics help identify pool efficiency and allocation patterns.
Only available when the `pool-metrics` feature is enabled.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `total_acquires` | `AtomicUsize` | — | Total number of acquire calls on this pool |
| `total_cache_hits` | `AtomicUsize` | — | Total number of cache hits (reused objects from pool) |
| `peak_items_stored` | `AtomicUsize` | — | Peak number of objects stored simultaneously in this pool |
| `total_creations` | `AtomicUsize` | — | Total number of objects created by the factory function |

---

#### PoolConfig

Configuration for the string buffer pool.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `max_buffers_per_size` | `usize` | `4` | Maximum buffers per size bucket |
| `initial_capacity` | `usize` | `4096` | Initial capacity for new buffers |
| `max_capacity_before_discard` | `usize` | `65536` | Maximum capacity before discarding |

---

#### ExtractionService

A `tower.Service` that dispatches extraction requests to the kreuzberg
core library.

This service is cheap to clone and can be shared across handlers.
Concurrency and timeouts are managed by composing Tower layers on top
(see `super.ExtractionServiceBuilder`).

*Opaque type — fields are not directly accessible.*

---

#### TracingLayer

A `tower.Layer` that wraps each extraction in a semantic tracing span.

*Opaque type — fields are not directly accessible.*

---

#### MetricsLayer

A `tower.Layer` that records service-level extraction metrics.

*Opaque type — fields are not directly accessible.*

---

#### ExtractionServiceBuilder

Builder for composing an extraction service with Tower middleware layers.

Layers are applied in the order: Tracing → Metrics → Timeout → ConcurrencyLimit → Service.

*Opaque type — fields are not directly accessible.*

---

#### ApiSizeLimits

API server size limit configuration.

Controls maximum sizes for request bodies and multipart uploads.
Default limits are set to 100 MB to accommodate typical document processing workloads.

# Default Values

- `max_request_body_bytes`: 100 MB (104,857,600 bytes)
- `max_multipart_field_bytes`: 100 MB (104,857,600 bytes)

# Configuration via Environment Variables

You can override the defaults using these environment variables:

```bash
# Modern approach (in bytes):
export KREUZBERG_MAX_REQUEST_BODY_BYTES=104857600     # 100 MB
export KREUZBERG_MAX_MULTIPART_FIELD_BYTES=104857600  # 100 MB
```

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `max_request_body_bytes` | `usize` | — | Maximum size of the entire request body in bytes. This applies to the total size of all uploaded files and form data in a single request. Default: 100 MB (104,857,600 bytes). |
| `max_multipart_field_bytes` | `usize` | — | Maximum size of a single multipart field in bytes. This applies to individual files in a multipart upload. Default: 100 MB (104,857,600 bytes). |

---

#### ErrorResponse

Error response.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `error_type` | `String` | — | Error type name |
| `message` | `String` | — | Error message |
| `traceback` | `Option<String>` | `Default::default()` | Stack trace (if available) |
| `status_code` | `u16` | — | HTTP status code |

---

#### WarmRequest

Cache warm request.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `all_embeddings` | `bool` | — | Download all embedding model presets |
| `embedding_model` | `Option<String>` | `Default::default()` | Specific embedding model preset to download |

---

#### KreuzbergMcp

Kreuzberg MCP server.

Provides document extraction capabilities via MCP tools.

The server loads a default extraction configuration from kreuzberg.toml/yaml/json
via discovery. Per-request OCR settings override the defaults.

*Opaque type — fields are not directly accessible.*

---

#### YakeParams

YAKE-specific parameters.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `window_size` | `usize` | `2` | Window size for co-occurrence analysis (default: 2). Controls the context window for computing co-occurrence statistics. |

---

#### RakeParams

RAKE-specific parameters.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `min_word_length` | `usize` | `1` | Minimum word length to consider (default: 1). |
| `max_words_per_phrase` | `usize` | `3` | Maximum words in a keyword phrase (default: 3). |

---

#### KeywordConfig

Keyword extraction configuration.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `algorithm` | `KeywordAlgorithm` | `KeywordAlgorithm::Yake` | Algorithm to use for extraction. |
| `max_keywords` | `usize` | `10` | Maximum number of keywords to extract (default: 10). |
| `min_score` | `f32` | `0` | Minimum score threshold (0.0-1.0, default: 0.0). Keywords with scores below this threshold are filtered out. Note: Score ranges differ between algorithms. |
| `ngram_range` | `(usize, usize)` | — | N-gram range for keyword extraction (min, max). (1, 1) = unigrams only (1, 2) = unigrams and bigrams (1, 3) = unigrams, bigrams, and trigrams (default) |
| `language` | `Option<String>` | `Default::default()` | Language code for stopword filtering (e.g., "en", "de", "fr"). If None, no stopword filtering is applied. |
| `yake_params` | `Option<YakeParams>` | `None` | YAKE-specific tuning parameters. |
| `rake_params` | `Option<RakeParams>` | `None` | RAKE-specific tuning parameters. |

---

#### OcrCacheStats

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `total_files` | `usize` | — | Total files |
| `total_size_mb` | `f64` | — | Total size mb |

---

#### LanguageRegistry

Language support registry for OCR backends.

Maintains a mapping of OCR backend names to their supported language codes.
This is the single source of truth for language support across all bindings.

*Opaque type — fields are not directly accessible.*

---

#### TesseractBackend

Native Tesseract OCR backend.

This backend wraps the OcrProcessor and implements the OcrBackend trait,
allowing it to be used through the plugin system.

# Thread Safety

Uses Arc for shared ownership and is thread-safe (Send + Sync).

*Opaque type — fields are not directly accessible.*

---

#### PaddleOcrBackend

PaddleOCR backend using ONNX Runtime.

Maintains a pool of OCR engines keyed by script family. Each family has its own
recognition model and character dictionary, while detection and classification
models are shared across all families.

# Thread Safety

The backend is `Send + Sync` and can be used across threads safely via `Arc`.
Each engine in the pool has its own mutex, so concurrent OCR on different
script families does not block.

*Opaque type — fields are not directly accessible.*

---

#### PaddleOcrConfig

Configuration for PaddleOCR backend.

Configures PaddleOCR text detection and recognition with multi-language support.
Uses a builder pattern for convenient configuration.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `language` | `String` | — | Language code (e.g., "en", "ch", "jpn", "kor", "deu", "fra") |
| `cache_dir` | `Option<PathBuf>` | `Default::default()` | Optional custom cache directory for model files |
| `use_angle_cls` | `bool` | — | Enable angle classification for rotated text (default: false). Can misfire on short text regions, rotating crops incorrectly before recognition. |
| `enable_table_detection` | `bool` | — | Enable table structure detection (default: false) |
| `det_db_thresh` | `f32` | — | Database threshold for text detection (default: 0.3) Range: 0.0-1.0, higher values require more confident detections |
| `det_db_box_thresh` | `f32` | — | Box threshold for text bounding box refinement (default: 0.5) Range: 0.0-1.0 |
| `det_db_unclip_ratio` | `f32` | — | Unclip ratio for expanding text bounding boxes (default: 1.6) Controls the expansion of detected text regions |
| `det_limit_side_len` | `u32` | — | Maximum side length for detection image (default: 960) Larger images may be resized to this limit for faster inference |
| `rec_batch_num` | `u32` | — | Batch size for recognition inference (default: 6) Number of text regions to process simultaneously |
| `padding` | `u32` | — | Padding in pixels added around the image before detection (default: 10). Large values can include surrounding content like table gridlines. |
| `drop_score` | `f32` | — | Minimum recognition confidence score for text lines (default: 0.5). Text regions with recognition confidence below this threshold are discarded. Matches PaddleOCR Python's `drop_score` parameter. Range: 0.0-1.0 |
| `model_tier` | `String` | — | Model tier controlling detection/recognition model size and accuracy trade-off. - `"mobile"` (default): Lightweight models (~4.5MB detection, ~16.5MB recognition), fast download and inference - `"server"`: Large, high-accuracy models (~88MB detection, ~84MB recognition), best for GPU or complex documents |

---

#### LayoutEngineConfig

Full configuration for the layout engine.

Provides fine-grained control over model selection, thresholds, and
postprocessing.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `backend` | `ModelBackend` | `ModelBackend::RtDetr` | Which model backend to use. |
| `confidence_threshold` | `Option<f32>` | `None` | Confidence threshold override (None = use model default). |
| `apply_heuristics` | `bool` | `true` | Whether to apply postprocessing heuristics. |
| `cache_dir` | `Option<PathBuf>` | `None` | Custom cache directory for model files (None = default). |
| `acceleration` | `Option<AccelerationConfig>` | `None` | Hardware acceleration for ONNX inference. |

---

#### DetectTimings

Granular timing breakdown for a single `detect()` call.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `preprocess_ms` | `f64` | — | Time spent in image preprocessing (resize, letterbox, normalize, tensor allocation). |
| `onnx_ms` | `f64` | — | Time for the ONNX `session.run()` call (actual neural network computation). |
| `model_total_ms` | `f64` | — | Total time from start of model call to end of raw output decoding. |
| `postprocess_ms` | `f64` | — | Time spent in postprocessing heuristics (confidence filtering, overlap resolution). |

---

#### PageRenderOptions

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `target_dpi` | `i32` | `300` | Target dpi |
| `max_image_dimension` | `i32` | `65536` | Maximum image dimension |
| `auto_adjust_dpi` | `bool` | `true` | Auto adjust dpi |
| `min_dpi` | `i32` | `72` | Minimum dpi |
| `max_dpi` | `i32` | `600` | Maximum dpi |

---

### Metadata Types

#### ListItemMetadata

Metadata about a detected list item.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `list_type` | `ListType` | — | Type of list (Bullet, Numbered, etc.) |
| `byte_start` | `usize` | — | Starting byte offset in the content string |
| `byte_end` | `usize` | — | Ending byte offset in the content string |
| `indent_level` | `u32` | — | List item indent level |

---

#### DocMetadata

Metadata extracted from DOC files.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `title` | `Option<String>` | `Default::default()` | Title |
| `subject` | `Option<String>` | `Default::default()` | Subject |
| `author` | `Option<String>` | `Default::default()` | Author |
| `last_author` | `Option<String>` | `Default::default()` | Last author |
| `created` | `Option<String>` | `Default::default()` | Created |
| `modified` | `Option<String>` | `Default::default()` | Modified |
| `revision_number` | `Option<String>` | `Default::default()` | Revision number |

---

#### PptMetadata

Metadata extracted from PPT files.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `title` | `Option<String>` | `Default::default()` | Title |
| `subject` | `Option<String>` | `Default::default()` | Subject |
| `author` | `Option<String>` | `Default::default()` | Author |
| `last_author` | `Option<String>` | `Default::default()` | Last author |

---

#### ParagraphMeta

Metadata for a single paragraph extracted from RTF.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `heading_level` | `u8` | — | Heading level (1-based): 1 = H1, 2 = H2, etc. 0 = not a heading. |
| `list_level` | `Option<u8>` | `Default::default()` | List nesting level (0-based). `None` means not a list item. |
| `list_id` | `Option<u16>` | `Default::default()` | List override ID (\lsN). Used to detect list boundaries. |
| `is_table` | `bool` | — | Whether this paragraph is a table placeholder (text is in tables vec). |
| `ordered` | `bool` | — | Whether this list item is ordered (numbered/lettered). Detected from `\listtext` or `\pntext` content. `False` = unordered (bullet). |

---

#### ChunkMetadata

Metadata about a chunk's position in the original document.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `byte_start` | `usize` | — | Byte offset where this chunk starts in the original text (UTF-8 valid boundary). |
| `byte_end` | `usize` | — | Byte offset where this chunk ends in the original text (UTF-8 valid boundary). |
| `token_count` | `Option<usize>` | `None` | Number of tokens in this chunk (if available). This is calculated by the embedding model's tokenizer if embeddings are enabled. |
| `chunk_index` | `usize` | — | Zero-based index of this chunk in the document. |
| `total_chunks` | `usize` | — | Total number of chunks in the document. |
| `first_page` | `Option<usize>` | `None` | First page number this chunk spans (1-indexed). Only populated when page tracking is enabled in extraction configuration. |
| `last_page` | `Option<usize>` | `None` | Last page number this chunk spans (1-indexed, equal to first_page for single-page chunks). Only populated when page tracking is enabled in extraction configuration. |
| `heading_context` | `Option<HeadingContext>` | `None` | Heading context when using Markdown chunker. Contains the heading hierarchy this chunk falls under. Only populated when `ChunkerType.Markdown` is used. |

---

#### ElementMetadata

Metadata for a semantic element.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `page_number` | `Option<usize>` | `None` | Page number (1-indexed) |
| `filename` | `Option<String>` | `None` | Source filename or document name |
| `coordinates` | `Option<BoundingBox>` | `None` | Bounding box coordinates if available |
| `element_index` | `Option<usize>` | `None` | Position index in the element sequence |
| `additional` | `HashMap<String, String>` | — | Additional custom metadata |

---

#### ImagePreprocessingMetadata

Image preprocessing metadata.

Tracks the transformations applied to an image during OCR preprocessing,
including DPI normalization, resizing, and resampling.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `original_dimensions` | `(usize, usize)` | — | Original image dimensions (width, height) in pixels |
| `original_dpi` | `(f64, f64)` | — | Original image DPI (horizontal, vertical) |
| `target_dpi` | `i32` | — | Target DPI from configuration |
| `scale_factor` | `f64` | — | Scaling factor applied to the image |
| `auto_adjusted` | `bool` | — | Whether DPI was auto-adjusted based on content |
| `final_dpi` | `i32` | — | Final DPI after processing |
| `new_dimensions` | `Option<(usize, usize)>` | `None` | New dimensions after resizing (if resized) |
| `resample_method` | `String` | — | Resampling algorithm used ("LANCZOS3", "CATMULLROM", etc.) |
| `dimension_clamped` | `bool` | — | Whether dimensions were clamped to max_image_dimension |
| `calculated_dpi` | `Option<i32>` | `None` | Calculated optimal DPI (if auto_adjust_dpi enabled) |
| `skipped_resize` | `bool` | — | Whether resize was skipped (dimensions already optimal) |
| `resize_error` | `Option<String>` | `None` | Error message if resize failed |

---

#### Metadata

Extraction result metadata.

Contains common fields applicable to all formats, format-specific metadata
via a discriminated union, and additional custom fields from postprocessors.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `title` | `Option<String>` | `Default::default()` | Document title |
| `subject` | `Option<String>` | `Default::default()` | Document subject or description |
| `authors` | `Vec<String>` | `vec![]` | Primary author(s) - always Vec for consistency |
| `keywords` | `Vec<String>` | `vec![]` | Keywords/tags - always Vec for consistency |
| `language` | `Option<String>` | `Default::default()` | Primary language (ISO 639 code) |
| `created_at` | `Option<String>` | `Default::default()` | Creation timestamp (ISO 8601 format) |
| `modified_at` | `Option<String>` | `Default::default()` | Last modification timestamp (ISO 8601 format) |
| `created_by` | `Option<String>` | `Default::default()` | User who created the document |
| `modified_by` | `Option<String>` | `Default::default()` | User who last modified the document |
| `pages` | `Option<PageStructure>` | `Default::default()` | Page/slide/sheet structure with boundaries |
| `format` | `Option<FormatMetadata>` | `Default::default()` | Format-specific metadata (discriminated union) Contains detailed metadata specific to the document format. Serializes with a `format_type` discriminator field. |
| `image_preprocessing` | `Option<ImagePreprocessingMetadata>` | `Default::default()` | Image preprocessing metadata (when OCR preprocessing was applied) |
| `json_schema` | `Option<serde_json::Value>` | `Default::default()` | JSON schema (for structured data extraction) |
| `error` | `Option<ErrorMetadata>` | `Default::default()` | Error metadata (for batch operations) |
| `extraction_duration_ms` | `Option<u64>` | `Default::default()` | Extraction duration in milliseconds (for benchmarking). This field is populated by batch extraction to provide per-file timing information. It's `None` for single-file extraction (which uses external timing). |
| `category` | `Option<String>` | `Default::default()` | Document category (from frontmatter or classification). |
| `tags` | `Vec<String>` | `vec![]` | Document tags (from frontmatter). |
| `document_version` | `Option<String>` | `Default::default()` | Document version string (from frontmatter). |
| `abstract_text` | `Option<String>` | `Default::default()` | Abstract or summary text (from frontmatter). |
| `output_format` | `Option<String>` | `Default::default()` | Output format identifier (e.g., "markdown", "html", "text"). Set by the output format pipeline stage when format conversion is applied. Previously stored in `metadata.additional["output_format"]`. |
| `additional` | `AHashMap` | — | Additional custom fields from postprocessors. **Deprecated**: Prefer using typed fields on `ExtractionResult` and `Metadata` instead of inserting into this map. Typed fields provide better cross-language compatibility and type safety. This field will be removed in a future major version. This flattened map allows Python/TypeScript postprocessors to add arbitrary fields (entity extraction, keyword extraction, etc.). Fields are merged at the root level during serialization. Uses `Cow<'static, str>` keys so static string keys avoid allocation. |

---

#### ExcelMetadata

Excel/spreadsheet metadata.

Contains information about sheets in Excel, OpenDocument Calc, and other
spreadsheet formats (.xlsx, .xls, .ods, etc.).

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `sheet_count` | `usize` | — | Total number of sheets in the workbook |
| `sheet_names` | `Vec<String>` | `vec![]` | Names of all sheets in order |

---

#### EmailMetadata

Email metadata extracted from .eml and .msg files.

Includes sender/recipient information, message ID, and attachment list.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `from_email` | `Option<String>` | `Default::default()` | Sender's email address |
| `from_name` | `Option<String>` | `Default::default()` | Sender's display name |
| `to_emails` | `Vec<String>` | `vec![]` | Primary recipients |
| `cc_emails` | `Vec<String>` | `vec![]` | CC recipients |
| `bcc_emails` | `Vec<String>` | `vec![]` | BCC recipients |
| `message_id` | `Option<String>` | `Default::default()` | Message-ID header value |
| `attachments` | `Vec<String>` | `vec![]` | List of attachment filenames |

---

#### ArchiveMetadata

Archive (ZIP/TAR/7Z) metadata.

Extracted from compressed archive files containing file lists and size information.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `format` | `str` | — | Archive format ("ZIP", "TAR", "7Z", etc.) |
| `file_count` | `usize` | — | Total number of files in the archive |
| `file_list` | `Vec<String>` | `vec![]` | List of file paths within the archive |
| `total_size` | `usize` | — | Total uncompressed size in bytes |
| `compressed_size` | `Option<usize>` | `Default::default()` | Compressed size in bytes (if available) |

---

#### ImageMetadata

Image metadata extracted from image files.

Includes dimensions, format, and EXIF data.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `width` | `u32` | — | Image width in pixels |
| `height` | `u32` | — | Image height in pixels |
| `format` | `String` | — | Image format (e.g., "PNG", "JPEG", "TIFF") |
| `exif` | `HashMap<String, String>` | `HashMap::new()` | EXIF metadata tags |

---

#### XmlMetadata

XML metadata extracted during XML parsing.

Provides statistics about XML document structure.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `element_count` | `usize` | — | Total number of XML elements processed |
| `unique_elements` | `Vec<String>` | `vec![]` | List of unique element tag names (sorted) |

---

#### TextMetadata

Text/Markdown metadata.

Extracted from plain text and Markdown files. Includes word counts and,
for Markdown, structural elements like headers and links.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `line_count` | `usize` | — | Number of lines in the document |
| `word_count` | `usize` | — | Number of words |
| `character_count` | `usize` | — | Number of characters |
| `headers` | `Vec<String>` | `vec![]` | Markdown headers (headings text only, for Markdown files) |
| `links` | `Vec<(String, String)>` | `vec![]` | Markdown links as (text, url) tuples (for Markdown files) |
| `code_blocks` | `Vec<(String, String)>` | `vec![]` | Code blocks as (language, code) tuples (for Markdown files) |

---

#### HeaderMetadata

Header/heading element metadata.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `level` | `u8` | — | Header level: 1 (h1) through 6 (h6) |
| `text` | `String` | — | Normalized text content of the header |
| `id` | `Option<String>` | `None` | HTML id attribute if present |
| `depth` | `usize` | — | Document tree depth at the header element |
| `html_offset` | `usize` | — | Byte offset in original HTML document |

---

#### LinkMetadata

Link element metadata.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `href` | `String` | — | The href URL value |
| `text` | `String` | — | Link text content (normalized) |
| `title` | `Option<String>` | `None` | Optional title attribute |
| `link_type` | `LinkType` | — | Link type classification |
| `rel` | `Vec<String>` | — | Rel attribute values |
| `attributes` | `Vec<(String, String)>` | — | Additional attributes as key-value pairs |

---

#### ImageMetadataType

Image element metadata.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `src` | `String` | — | Image source (URL, data URI, or SVG content) |
| `alt` | `Option<String>` | `None` | Alternative text from alt attribute |
| `title` | `Option<String>` | `None` | Title attribute |
| `dimensions` | `Option<(u32, u32)>` | `None` | Image dimensions as (width, height) if available |
| `image_type` | `ImageType` | — | Image type classification |
| `attributes` | `Vec<(String, String)>` | — | Additional attributes as key-value pairs |

---

#### HtmlMetadata

HTML metadata extracted from HTML documents.

Includes document-level metadata, Open Graph data, Twitter Card metadata,
and extracted structural elements (headers, links, images, structured data).

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `title` | `Option<String>` | `Default::default()` | Document title from `<title>` tag |
| `description` | `Option<String>` | `Default::default()` | Document description from `<meta name="description">` tag |
| `keywords` | `Vec<String>` | `vec![]` | Document keywords from `<meta name="keywords">` tag, split on commas |
| `author` | `Option<String>` | `Default::default()` | Document author from `<meta name="author">` tag |
| `canonical_url` | `Option<String>` | `Default::default()` | Canonical URL from `<link rel="canonical">` tag |
| `base_href` | `Option<String>` | `Default::default()` | Base URL from `<base href="">` tag for resolving relative URLs |
| `language` | `Option<String>` | `Default::default()` | Document language from `lang` attribute |
| `text_direction` | `Option<TextDirection>` | `Default::default()` | Document text direction from `dir` attribute |
| `open_graph` | `HashMap<String, String>` | `HashMap::new()` | Open Graph metadata (og:* properties) for social media Keys like "title", "description", "image", "url", etc. |
| `twitter_card` | `HashMap<String, String>` | `HashMap::new()` | Twitter Card metadata (twitter:* properties) Keys like "card", "site", "creator", "title", "description", "image", etc. |
| `meta_tags` | `HashMap<String, String>` | `HashMap::new()` | Additional meta tags not covered by specific fields Keys are meta name/property attributes, values are content |
| `headers` | `Vec<HeaderMetadata>` | `vec![]` | Extracted header elements with hierarchy |
| `links` | `Vec<LinkMetadata>` | `vec![]` | Extracted hyperlinks with type classification |
| `images` | `Vec<ImageMetadataType>` | `vec![]` | Extracted images with source and dimensions |
| `structured_data` | `Vec<StructuredData>` | `vec![]` | Extracted structured data blocks |

---

#### OcrMetadata

OCR processing metadata.

Captures information about OCR processing configuration and results.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `language` | `String` | — | OCR language code(s) used |
| `psm` | `i32` | — | Tesseract Page Segmentation Mode (PSM) |
| `output_format` | `String` | — | Output format (e.g., "text", "hocr") |
| `table_count` | `usize` | — | Number of tables detected |
| `table_rows` | `Option<usize>` | `Default::default()` | Table rows |
| `table_cols` | `Option<usize>` | `Default::default()` | Table cols |

---

#### ErrorMetadata

Error metadata (for batch operations).

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `error_type` | `String` | — | Error type |
| `message` | `String` | — | Message |

---

#### PptxMetadata

PowerPoint presentation metadata.

Extracted from PPTX files containing slide counts and presentation details.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `slide_count` | `usize` | — | Total number of slides in the presentation |
| `slide_names` | `Vec<String>` | `vec![]` | Names of slides (if available) |
| `image_count` | `Option<usize>` | `Default::default()` | Number of embedded images |
| `table_count` | `Option<usize>` | `Default::default()` | Number of tables |

---

#### DocxMetadata

Word document metadata.

Extracted from DOCX files using shared Office Open XML metadata extraction.
Integrates with `office_metadata` module for core/app/custom properties.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `core_properties` | `Option<CoreProperties>` | `Default::default()` | Core properties from docProps/core.xml (Dublin Core metadata) Contains title, creator, subject, keywords, dates, etc. Shared format across DOCX/PPTX/XLSX documents. |
| `app_properties` | `Option<DocxAppProperties>` | `Default::default()` | Application properties from docProps/app.xml (Word-specific statistics) Contains word count, page count, paragraph count, editing time, etc. DOCX-specific variant of Office application properties. |
| `custom_properties` | `HashMap<String, serde_json::Value>` | `HashMap::new()` | Custom properties from docProps/custom.xml (user-defined properties) Contains key-value pairs defined by users or applications. Values can be strings, numbers, booleans, or dates. |

---

#### CsvMetadata

CSV/TSV file metadata.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `row_count` | `usize` | — | Number of row |
| `column_count` | `usize` | — | Number of column |
| `delimiter` | `Option<String>` | `Default::default()` | Delimiter |
| `has_header` | `bool` | — | Whether header |
| `column_types` | `Vec<String>` | `vec![]` | Column types |

---

#### BibtexMetadata

BibTeX bibliography metadata.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `entry_count` | `usize` | — | Number of entry |
| `citation_keys` | `Vec<String>` | `vec![]` | Citation keys |
| `authors` | `Vec<String>` | `vec![]` | Authors |
| `year_range` | `Option<YearRange>` | `Default::default()` | Year range (year range) |
| `entry_types` | `HashMap<String, usize>` | `HashMap::new()` | Entry types |

---

#### CitationMetadata

Citation file metadata (RIS, PubMed, EndNote).

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `citation_count` | `usize` | — | Number of citation |
| `format` | `Option<String>` | `Default::default()` | Format |
| `authors` | `Vec<String>` | `vec![]` | Authors |
| `year_range` | `Option<YearRange>` | `Default::default()` | Year range (year range) |
| `dois` | `Vec<String>` | `vec![]` | Dois |
| `keywords` | `Vec<String>` | `vec![]` | Keywords |

---

#### FictionBookMetadata

FictionBook (FB2) metadata.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `genres` | `Vec<String>` | `vec![]` | Genres |
| `sequences` | `Vec<String>` | `vec![]` | Sequences |
| `annotation` | `Option<String>` | `Default::default()` | Annotation |

---

#### DbfMetadata

dBASE (DBF) file metadata.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `record_count` | `usize` | — | Number of record |
| `field_count` | `usize` | — | Number of field |
| `fields` | `Vec<DbfFieldInfo>` | `vec![]` | Fields |

---

#### JatsMetadata

JATS (Journal Article Tag Suite) metadata.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `copyright` | `Option<String>` | `Default::default()` | Copyright |
| `license` | `Option<String>` | `Default::default()` | License |
| `history_dates` | `HashMap<String, String>` | `HashMap::new()` | History dates |
| `contributor_roles` | `Vec<ContributorRole>` | `vec![]` | Contributor roles |

---

#### EpubMetadata

EPUB metadata (Dublin Core extensions).

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `coverage` | `Option<String>` | `Default::default()` | Coverage |
| `dc_format` | `Option<String>` | `Default::default()` | Dc format |
| `relation` | `Option<String>` | `Default::default()` | Relation |
| `source` | `Option<String>` | `Default::default()` | Source |
| `dc_type` | `Option<String>` | `Default::default()` | Dc type |
| `cover_image` | `Option<String>` | `Default::default()` | Cover image |

---

#### PstMetadata

Outlook PST archive metadata.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `message_count` | `usize` | — | Number of message |

---

#### OpenWebDocumentMetadata

Metadata for the OpenWebUI external document loader response.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `source` | `String` | — | Original filename |

---

#### PdfMetadata

PDF-specific metadata.

Contains metadata fields specific to PDF documents that are not in the common
`Metadata` structure. Common fields like title, authors, keywords, and dates
are now at the `Metadata` level.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `pdf_version` | `Option<String>` | `Default::default()` | PDF version (e.g., "1.7", "2.0") |
| `producer` | `Option<String>` | `Default::default()` | PDF producer (application that created the PDF) |
| `is_encrypted` | `Option<bool>` | `Default::default()` | Whether the PDF is encrypted/password-protected |
| `width` | `Option<i64>` | `Default::default()` | First page width in points (1/72 inch) |
| `height` | `Option<i64>` | `Default::default()` | First page height in points (1/72 inch) |
| `page_count` | `Option<usize>` | `Default::default()` | Total number of pages in the PDF document |

---

#### PdfExtractionMetadata

Complete PDF extraction metadata including common and PDF-specific fields.

This struct combines common document fields (title, authors, dates) with
PDF-specific metadata and optional page structure information. It is returned
by `extract_metadata_from_document()` when page boundaries are provided.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `title` | `Option<String>` | `None` | Document title |
| `subject` | `Option<String>` | `None` | Document subject or description |
| `authors` | `Vec<String>` | `None` | Document authors (parsed from PDF Author field) |
| `keywords` | `Vec<String>` | `None` | Document keywords (parsed from PDF Keywords field) |
| `created_at` | `Option<String>` | `None` | Creation timestamp (ISO 8601 format) |
| `modified_at` | `Option<String>` | `None` | Last modification timestamp (ISO 8601 format) |
| `created_by` | `Option<String>` | `None` | Application or user that created the document |
| `pdf_specific` | `PdfMetadata` | — | PDF-specific metadata |
| `page_structure` | `Option<PageStructure>` | `None` | Page structure with boundaries and optional per-page metadata |

---

#### CommonPdfMetadata

Common metadata fields extracted from a PDF.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `title` | `Option<String>` | `None` | Title |
| `subject` | `Option<String>` | `None` | Subject |
| `authors` | `Vec<String>` | `None` | Authors |
| `keywords` | `Vec<String>` | `None` | Keywords |
| `created_at` | `Option<String>` | `None` | Created at |
| `modified_at` | `Option<String>` | `None` | Modified at |
| `created_by` | `Option<String>` | `None` | Created by |

---

### Document Structure

#### TableWidth

Width specification used for tables and cells.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `value` | `i32` | — | Value |
| `width_type` | `String` | — | Width type |

---

#### TableValidator

Helper struct for validating table cell counts.

*Opaque type — fields are not directly accessible.*

---

#### DocumentRelationship

A resolved relationship between two nodes in the document tree.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `source` | `u32` | — | Source node index (the referencing node). |
| `target` | `u32` | — | Target node index (the referenced node). |
| `kind` | `RelationshipKind` | — | Semantic kind of the relationship. |

---

#### DocumentNode

A single node in the document tree.

Each node has deterministic `id`, typed `content`, optional `parent`/`children`
for tree structure, and metadata like page number, bounding box, and content layer.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `id` | `NodeId` | — | Deterministic identifier (hash of content + position). |
| `content` | `NodeContent` | — | Node content — tagged enum, type-specific data only. |
| `parent` | `Option<u32>` | `None` | Parent node index (`None` = root-level node). |
| `children` | `Vec<u32>` | — | Child node indices in reading order. |
| `content_layer` | `ContentLayer` | — | Content layer classification. |
| `page` | `Option<u32>` | `None` | Page number where this node starts (1-indexed). |
| `page_end` | `Option<u32>` | `None` | Page number where this node ends (for multi-page tables/sections). |
| `bbox` | `Option<BoundingBox>` | `None` | Bounding box in document coordinates. |
| `annotations` | `Vec<TextAnnotation>` | — | Inline annotations (formatting, links) on this node's text content. Only meaningful for text-carrying nodes; empty for containers. |
| `attributes` | `HashMap<String, String>` | `None` | Format-specific key-value attributes. Extensible bag for data that doesn't warrant a typed field: CSS classes, LaTeX environment names, Excel cell formulas, slide layout names, etc. |

---

#### GridCell

Individual grid cell with position and span metadata.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `content` | `String` | — | Cell text content. |
| `row` | `u32` | — | Zero-indexed row position. |
| `col` | `u32` | — | Zero-indexed column position. |
| `row_span` | `u32` | — | Number of rows this cell spans. |
| `col_span` | `u32` | — | Number of columns this cell spans. |
| `is_header` | `bool` | — | Whether this is a header cell. |
| `bbox` | `Option<BoundingBox>` | `None` | Bounding box for this cell (if available). |

---

#### OcrTable

Table detected via OCR.

Represents a table structure recognized during OCR processing.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `cells` | `Vec<Vec<String>>` | — | Table cells as a 2D vector (rows × columns) |
| `markdown` | `String` | — | Markdown representation of the table |
| `page_number` | `usize` | — | Page number where the table was found (1-indexed) |
| `bounding_box` | `Option<OcrTableBoundingBox>` | `None` | Bounding box of the table in pixel coordinates (from OCR word positions). |

---

#### OcrTableBoundingBox

Bounding box for an OCR-detected table in pixel coordinates.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `left` | `u32` | — | Left x-coordinate (pixels) |
| `top` | `u32` | — | Top y-coordinate (pixels) |
| `right` | `u32` | — | Right x-coordinate (pixels) |
| `bottom` | `u32` | — | Bottom y-coordinate (pixels) |

---

#### InternalDocument

The internal flat document representation.

All extractors output this structure. It is converted to the public
`ExtractionResult` and
`DocumentStructure` in the pipeline.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `elements` | `Vec<InternalElement>` | — | All elements in reading order. Append-only during extraction. |
| `relationships` | `Vec<Relationship>` | — | Relationships between elements (source index → target). Stored separately from elements for cache-friendly iteration. |
| `source_format` | `str` | — | Source format identifier (e.g., "pdf", "docx", "html", "markdown"). |
| `metadata` | `Metadata` | — | Document-level metadata (title, author, dates, etc.). |
| `images` | `Vec<ExtractedImage>` | — | Extracted images (binary data). Referenced by index from `ElementKind.Image`. |
| `tables` | `Vec<Table>` | — | Extracted tables (structured data). Referenced by index from `ElementKind.Table`. |
| `uris` | `Vec<Uri>` | — | URIs/links discovered during extraction (hyperlinks, image refs, citations, etc.). |
| `children` | `Vec<ArchiveEntry>` | `None` | Archive children: fully-extracted results for files within an archive. Only populated by archive extractors (ZIP, TAR, 7z, GZIP) when recursive extraction is enabled. Each entry contains the full `ExtractionResult` for a child file that was extracted through the public pipeline. |
| `mime_type` | `str` | — | MIME type of the source document (e.g., "application/pdf", "text/html"). |
| `processing_warnings` | `Vec<ProcessingWarning>` | — | Non-fatal warnings collected during extraction. |
| `annotations` | `Vec<PdfAnnotation>` | `None` | PDF annotations (links, highlights, notes). |
| `prebuilt_pages` | `Vec<PageContent>` | `None` | Pre-built per-page content (set by extractors that track page boundaries natively). When populated, `derive_extraction_result` uses this directly instead of attempting to reconstruct pages from element-level page numbers. |
| `pre_rendered_content` | `Option<String>` | `None` | Pre-rendered formatted content produced by the extractor itself. When an extractor has direct access to high-quality formatted output (e.g., html-to-markdown produces GFM markdown), it can store that here to bypass the lossy InternalDocument → renderer round-trip. `derive_extraction_result` will use this directly when the requested output format matches `metadata.output_format`. |
| `prebuilt_ocr_elements` | `Vec<OcrElement>` | `None` | Pre-built OCR element list (set by extractors that have direct access to bounding-box element data alongside a separately produced coherent text). When populated, `derive_extraction_result` uses this directly instead of reconstructing `OcrElement`s from `OcrText` `InternalElement`s. This lets the image extractor carry Tesseract/paddle-ocr bounding-box metadata without injecting raw word tokens into the element list (which would otherwise corrupt `render_plain` and page content — issue #706). |
| `llm_usage` | `Vec<LlmUsage>` | `None` | LLM usage records accumulated during extraction (e.g., VLM OCR per page). Populated by extractors that call LLM-backed backends (VLM OCR). `derive_extraction_result` transfers this to `ExtractionResult.llm_usage`. |

---

#### InternalDocumentBuilder

Builder for constructing `InternalDocument` with an ergonomic push-based API.

Tracks nesting depth automatically for list and quote containers,
and generates deterministic element IDs via blake3 hashing.

*Opaque type — fields are not directly accessible.*

---

#### OpenWebDocumentResponse

OpenWebUI "External" engine response format.

Returned by `PUT /process` for the OpenWebUI external document loader.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `page_content` | `String` | — | Extracted text content |
| `metadata` | `OpenWebDocumentMetadata` | — | Document metadata |

---

#### DoclingCompatDocument

Document content in the docling-serve response format.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `md_content` | `String` | — | Markdown content of the converted document |

---

#### RecognizedTable

Pre-computed table markdown for a table detection region.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `detection_bbox` | `BBox` | — | Detection bbox that this table corresponds to (for matching). |
| `cells` | `Vec<Vec<String>>` | — | Table cells as a 2D vector (rows x columns). |
| `markdown` | `String` | — | Rendered markdown table. |

---

#### TableClassifier

PP-LCNet table classifier model.

*Opaque type — fields are not directly accessible.*

---

### OCR Types

#### OcrPipelineStage

A single backend stage in the OCR pipeline.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `backend` | `String` | — | Backend name: "tesseract", "paddleocr", "easyocr", or a custom registered name. |
| `priority` | `u32` | — | Priority weight (higher = tried first). Stages are sorted by priority descending. |
| `language` | `Option<String>` | `None` | Language override for this stage (None = use parent OcrConfig.language). |
| `tesseract_config` | `Option<TesseractConfig>` | `None` | Tesseract-specific config override for this stage. |
| `paddle_ocr_config` | `Option<serde_json::Value>` | `None` | PaddleOCR-specific config for this stage. |
| `vlm_config` | `Option<LlmConfig>` | `None` | VLM config override for this pipeline stage. |

---

#### OcrFallbackDecision

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `stats` | `NativeTextStats` | — | Stats (native text stats) |
| `avg_non_whitespace` | `f64` | — | Avg non whitespace |
| `avg_alnum` | `f64` | — | Avg alnum |
| `fallback` | `bool` | — | Fallback |

---

#### OcrBackend

Trait for OCR backend plugins.

Implement this trait to add custom OCR capabilities. OCR backends can be:
- Native Rust implementations (like Tesseract)
- FFI bridges to Python libraries (like EasyOCR, PaddleOCR)
- Cloud-based OCR services (Google Vision, AWS Textract, etc.)

# Thread Safety

OCR backends must be thread-safe (`Send + Sync`) to support concurrent processing.

*Opaque type — fields are not directly accessible.*

---

#### OcrRotation

Rotation information for an OCR element.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `angle_degrees` | `f64` | — | Rotation angle in degrees (0, 90, 180, 270 for PaddleOCR). |
| `confidence` | `Option<f64>` | `None` | Confidence score for the rotation detection. |

---

#### VlmOcrBackend

VLM-based OCR backend using liter-llm vision models.

This backend sends images to a vision language model (e.g., GPT-4o, Claude)
for text extraction, as an alternative to traditional OCR backends.

*Opaque type — fields are not directly accessible.*

---

#### OcrCache

*Opaque type — fields are not directly accessible.*

---

#### OcrProcessor

*Opaque type — fields are not directly accessible.*

---

### Other Types

#### GenericCache

*Opaque type — fields are not directly accessible.*

---

#### FileBytes

An owned buffer of file bytes.

On non-WASM platforms this may be backed by a memory-mapped file (zero heap
allocation for the file contents) or by a `Vec<u8>` for small files.
On WASM it is always a `Vec<u8>`.

Implements `Deref<Target = [u8]>` so callers can pass `&FileBytes` as `&[u8]`
without any additional copy.

*Opaque type — fields are not directly accessible.*

---

#### SupportedFormat

A supported document format entry.

Represents a file extension and its corresponding MIME type that Kreuzberg can process.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `extension` | `String` | — | File extension (without leading dot), e.g., "pdf", "docx" |
| `mime_type` | `String` | — | MIME type string, e.g., "application/pdf" |

---

#### ParaText

Plain text content decoded from a ParaText record (tag 0x43).

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `content` | `String` | — | The extracted text content |

---

#### FileHeader

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `flags` | `u32` | — | Flags |

---

#### Record

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `tag_id` | `u16` | — | Tag id |
| `data` | `Vec<u8>` | — | Data |

---

#### StreamReader

*Opaque type — fields are not directly accessible.*

---

#### CfbReader

*Opaque type — fields are not directly accessible.*

---

#### ExtractedInlineImage

Extracted inline image with metadata.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `data` | `Vec<u8>` | — | Uses `bytes.Bytes` for cheap cloning of large buffers. |
| `format` | `String` | — | Format |
| `filename` | `Option<String>` | `None` | Filename |
| `description` | `Option<String>` | `None` | Human-readable description |
| `dimensions` | `Option<(u32, u32)>` | `None` | Dimensions ((u32, u32)) |
| `attributes` | `Vec<(String, String)>` | — | Attributes |

---

#### Note

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `id` | `String` | — | Unique identifier |
| `note_type` | `NoteType` | — | Note type (note type) |
| `paragraphs` | `Vec<Paragraph>` | — | Paragraphs |

---

#### StyleDefinition

A single style definition parsed from `<w:style>` in `word/styles.xml`.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `id` | `String` | — | The style ID (`w:styleId` attribute). |
| `name` | `Option<String>` | `None` | Human-readable name (`<w:name w:val="..."/>`). |
| `style_type` | `StyleType` | — | Style type: paragraph, character, table, or numbering. |
| `based_on` | `Option<String>` | `None` | ID of the parent style (`<w:basedOn w:val="..."/>`). |
| `next_style` | `Option<String>` | `None` | ID of the style to apply to the next paragraph (`<w:next w:val="..."/>`). |
| `is_default` | `bool` | — | Whether this is the default style for its type. |
| `paragraph_properties` | `ParagraphProperties` | — | Paragraph properties defined directly on this style. |
| `run_properties` | `RunProperties` | — | Run properties defined directly on this style. |

---

#### BorderStyle

A single border specification.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `style` | `String` | — | Style |
| `size` | `Option<i32>` | `None` | Size in bytes |
| `color` | `Option<String>` | `None` | Color |
| `space` | `Option<i32>` | `None` | Space |

---

#### CustomProperties

Custom properties from docProps/custom.xml

Maps property names to their values. Values are converted to JSON types
based on the VT (Variant Type) specified in the XML.

*Opaque type — fields are not directly accessible.*

---

#### SyncExtractor

Trait for extractors that can work synchronously (WASM-compatible).

This trait defines the synchronous extraction interface for WASM targets and other
environments where async/tokio runtimes are not available or desirable.

# Implementation

Extractors that need to support WASM should implement this trait in addition to
the async `DocumentExtractor` trait. This allows the same extractor to work in both
environments by delegating to the sync implementation.

# MIME Type Validation

The `mime_type` parameter is guaranteed to be already validated.

*Opaque type — fields are not directly accessible.*

---

#### ZipBombValidator

Helper struct for validating ZIP archives for security issues.

*Opaque type — fields are not directly accessible.*

---

#### StringGrowthValidator

Helper struct for tracking and validating string growth.

*Opaque type — fields are not directly accessible.*

---

#### IterationValidator

Helper struct for validating iteration counts.

*Opaque type — fields are not directly accessible.*

---

#### DepthValidator

Helper struct for validating nesting depth.

*Opaque type — fields are not directly accessible.*

---

#### EntityValidator

Helper struct for validating entity/string length.

*Opaque type — fields are not directly accessible.*

---

#### ModelCache

*Opaque type — fields are not directly accessible.*

---

#### PanicContext

Context information captured when a panic occurs.

This struct stores detailed information about where and when a panic happened,
enabling better error reporting across FFI boundaries.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `file` | `String` | — | Source file where the panic occurred |
| `line` | `u32` | — | Line number where the panic occurred |
| `function` | `String` | — | Function name where the panic occurred |
| `message` | `String` | — | Panic message extracted from the panic payload |
| `timestamp` | `SystemTime` | — | Timestamp when the panic was captured |

---

#### Renderer

Trait for document renderers that convert `InternalDocument` to output strings.

Renderers are stateless converters that transform the internal document
representation into a specific output format (Markdown, HTML, Djot, plain text, etc.).

# Thread Safety

Renderers must be `Send + Sync` to support concurrent rendering across threads.

*Opaque type — fields are not directly accessible.*

---

#### PluginHealthStatus

Plugin health status information.

Contains diagnostic information about registered plugins for each type.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `ocr_backends_count` | `usize` | — | Number of registered OCR backends |
| `ocr_backends` | `Vec<String>` | — | Names of registered OCR backends |
| `extractors_count` | `usize` | — | Number of registered document extractors |
| `extractors` | `Vec<String>` | — | Names of registered document extractors |
| `post_processors_count` | `usize` | — | Number of registered post-processors |
| `post_processors` | `Vec<String>` | — | Names of registered post-processors |
| `validators_count` | `usize` | — | Number of registered validators |
| `validators` | `Vec<String>` | — | Names of registered validators |

---

#### Plugin

Base trait that all plugins must implement.

This trait provides common functionality for plugin lifecycle management,
identification, and metadata.

# Thread Safety

All plugins must be `Send + Sync` to support concurrent usage across threads.

*Opaque type — fields are not directly accessible.*

---

#### StyledHtmlRenderer

Styled HTML renderer.

Implements the `Renderer` trait; registered as `"html"` when the
`html` feature is active. Configuration is baked in at
construction time — no per-render allocation for CSS resolution.

*Opaque type — fields are not directly accessible.*

---

#### ExtractionMetrics

Collection of all kreuzberg metric instruments.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `extraction_total` | `Counter` | — | Total extractions (attributes: mime_type, extractor, status). |
| `cache_hits` | `Counter` | — | Cache hits. |
| `cache_misses` | `Counter` | — | Cache misses. |
| `batch_total` | `Counter` | — | Total batch requests (attributes: status). |
| `extraction_duration_ms` | `Histogram` | — | Extraction wall-clock duration in milliseconds (attributes: mime_type, extractor). |
| `extraction_input_bytes` | `Histogram` | — | Input document size in bytes (attributes: mime_type). |
| `extraction_output_bytes` | `Histogram` | — | Output content size in bytes (attributes: mime_type). |
| `pipeline_duration_ms` | `Histogram` | — | Pipeline stage duration in milliseconds (attributes: stage). |
| `ocr_duration_ms` | `Histogram` | — | OCR duration in milliseconds (attributes: backend, language). |
| `batch_duration_ms` | `Histogram` | — | Batch total duration in milliseconds. |
| `concurrent_extractions` | `UpDownCounter` | — | Currently in-flight extractions. |

---

#### TokenReducer

*Opaque type — fields are not directly accessible.*

---

#### QualityProcessor

Post-processor that calculates quality score and cleans text.

This processor:
- Runs in the Early processing stage
- Calculates quality score when `config.enable_quality_processing` is true
- Stores quality score in `metadata.additional["quality_score"]`
- Cleans and normalizes extracted text

*Opaque type — fields are not directly accessible.*

---

#### PdfAnnotation

A PDF annotation extracted from a document page.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `annotation_type` | `PdfAnnotationType` | — | The type of annotation. |
| `content` | `Option<String>` | `None` | Text content of the annotation (e.g., comment text, link URL). |
| `page_number` | `usize` | — | Page number where the annotation appears (1-indexed). |
| `bounding_box` | `Option<BoundingBox>` | `None` | Bounding box of the annotation on the page. |

---

#### DjotContent

Comprehensive Djot document structure with semantic preservation.

This type captures the full richness of Djot markup, including:
- Block-level structures (headings, lists, blockquotes, code blocks, etc.)
- Inline formatting (emphasis, strong, highlight, subscript, superscript, etc.)
- Attributes (classes, IDs, key-value pairs)
- Links, images, footnotes
- Math expressions (inline and display)
- Tables with full structure

Available when the `djot` feature is enabled.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `plain_text` | `String` | — | Plain text representation for backwards compatibility |
| `blocks` | `Vec<FormattedBlock>` | — | Structured block-level content |
| `metadata` | `Metadata` | — | Metadata from YAML frontmatter |
| `tables` | `Vec<Table>` | — | Extracted tables as structured data |
| `images` | `Vec<DjotImage>` | — | Extracted images with metadata |
| `links` | `Vec<DjotLink>` | — | Extracted links with URLs |
| `footnotes` | `Vec<Footnote>` | — | Footnote definitions |
| `attributes` | `Vec<(String, Attributes)>` | — | Attributes mapped by element identifier (if present) |

---

#### FormattedBlock

Block-level element in a Djot document.

Represents structural elements like headings, paragraphs, lists, code blocks, etc.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `block_type` | `BlockType` | — | Type of block element |
| `level` | `Option<usize>` | `None` | Heading level (1-6) for headings, or nesting level for lists |
| `inline_content` | `Vec<InlineElement>` | — | Inline content within the block |
| `attributes` | `Option<Attributes>` | `None` | Element attributes (classes, IDs, key-value pairs) |
| `language` | `Option<String>` | `None` | Language identifier for code blocks |
| `code` | `Option<String>` | `None` | Raw code content for code blocks |
| `children` | `Vec<FormattedBlock>` | — | Nested blocks for containers (blockquotes, list items, divs) |

---

#### InlineElement

Inline element within a block.

Represents text with formatting, links, images, etc.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `element_type` | `InlineType` | — | Type of inline element |
| `content` | `String` | — | Text content |
| `attributes` | `Option<Attributes>` | `None` | Element attributes |
| `metadata` | `HashMap<String, String>` | `None` | Additional metadata (e.g., href for links, src/alt for images) |

---

#### DjotImage

Image element in Djot.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `src` | `String` | — | Image source URL or path |
| `alt` | `String` | — | Alternative text |
| `title` | `Option<String>` | `None` | Optional title |
| `attributes` | `Option<Attributes>` | `None` | Element attributes |

---

#### DjotLink

Link element in Djot.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `url` | `String` | — | Link URL |
| `text` | `String` | — | Link text content |
| `title` | `Option<String>` | `None` | Optional title |
| `attributes` | `Option<Attributes>` | `None` | Element attributes |

---

#### Footnote

Footnote in Djot.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `label` | `String` | — | Footnote label |
| `content` | `Vec<FormattedBlock>` | — | Footnote content blocks |

---

#### TextAnnotation

Inline text annotation — byte-range based formatting and links.

Annotations reference byte offsets into the node's text content,
enabling precise identification of formatted regions.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `start` | `u32` | — | Start byte offset in the node's text content (inclusive). |
| `end` | `u32` | — | End byte offset in the node's text content (exclusive). |
| `kind` | `AnnotationKind` | — | Annotation type. |

---

#### ArchiveEntry

A single file extracted from an archive.

When archives (ZIP, TAR, 7Z, GZIP) are extracted with recursive extraction
enabled, each processable file produces its own full `ExtractionResult`.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `path` | `String` | — | Archive-relative file path (e.g. "folder/document.pdf"). |
| `mime_type` | `String` | — | Detected MIME type of the file. |
| `result` | `ExtractionResult` | — | Full extraction result for this file. |

---

#### ProcessingWarning

A non-fatal warning from a processing pipeline stage.

Captures errors from optional features that don't prevent extraction
but may indicate degraded results.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `source` | `str` | — | The pipeline stage or feature that produced this warning (e.g., "embedding", "chunking", "language_detection", "output_format"). |
| `message` | `str` | — | Human-readable description of what went wrong. |

---

#### Chunk

A text chunk with optional embedding and metadata.

Chunks are created when chunking is enabled in `ExtractionConfig`. Each chunk
contains the text content, optional embedding vector (if embedding generation
is configured), and metadata about its position in the document.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `content` | `String` | — | The text content of this chunk. |
| `chunk_type` | `ChunkType` | — | Semantic structural classification of this chunk. Assigned by the heuristic classifier based on content patterns and heading context. Defaults to `ChunkType.Unknown` when no rule matches. |
| `embedding` | `Vec<f32>` | `None` | Optional embedding vector for this chunk. Only populated when `EmbeddingConfig` is provided in chunking configuration. The dimensionality depends on the chosen embedding model. |
| `metadata` | `ChunkMetadata` | — | Metadata about this chunk's position and properties. |

---

#### HeadingContext

Heading context for a chunk within a Markdown document.

Contains the heading hierarchy from document root to this chunk's section.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `headings` | `Vec<HeadingLevel>` | — | The heading hierarchy from document root to this chunk's section. Index 0 is the outermost (h1), last element is the most specific. |

---

#### HeadingLevel

A single heading in the hierarchy.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `level` | `u8` | — | Heading depth (1 = h1, 2 = h2, etc.) |
| `text` | `String` | — | The text content of the heading. |

---

#### ExtractedImage

Extracted image from a document.

Contains raw image data, metadata, and optional nested OCR results.
Raw bytes allow cross-language compatibility - users can convert to
PIL.Image (Python), Sharp (Node.js), or other formats as needed.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `data` | `Vec<u8>` | — | Raw image data (PNG, JPEG, WebP, etc. bytes). Uses `bytes.Bytes` for cheap cloning of large buffers. |
| `format` | `str` | — | Image format (e.g., "jpeg", "png", "webp") Uses Cow<'static, str> to avoid allocation for static literals. |
| `image_index` | `usize` | — | Zero-indexed position of this image in the document/page |
| `page_number` | `Option<usize>` | `None` | Page/slide number where image was found (1-indexed) |
| `width` | `Option<u32>` | `None` | Image width in pixels |
| `height` | `Option<u32>` | `None` | Image height in pixels |
| `colorspace` | `Option<String>` | `None` | Colorspace information (e.g., "RGB", "CMYK", "Gray") |
| `bits_per_component` | `Option<u32>` | `None` | Bits per color component (e.g., 8, 16) |
| `is_mask` | `bool` | — | Whether this image is a mask image |
| `description` | `Option<String>` | `None` | Optional description of the image |
| `ocr_result` | `Option<ExtractionResult>` | `None` | Nested OCR extraction result (if image was OCRed) When OCR is performed on this image, the result is embedded here rather than in a separate collection, making the relationship explicit. |
| `bounding_box` | `Option<BoundingBox>` | `None` | Bounding box of the image on the page (PDF coordinates: x0=left, y0=bottom, x1=right, y1=top). Only populated for PDF-extracted images when position data is available from pdfium. |
| `source_path` | `Option<String>` | `None` | Original source path of the image within the document archive (e.g., "media/image1.png" in DOCX). Used for rendering image references when the binary data is not extracted. |

---

#### Element

Semantic element extracted from document.

Represents a logical unit of content with semantic classification,
unique identifier, and metadata for tracking origin and position.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `element_id` | `ElementId` | — | Unique element identifier |
| `element_type` | `ElementType` | — | Semantic type of this element |
| `text` | `String` | — | Text content of the element |
| `metadata` | `ElementMetadata` | — | Metadata about the element |

---

#### ExcelWorkbook

Excel workbook representation.

Contains all sheets from an Excel file (.xlsx, .xls, etc.) with
extracted content and metadata.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `sheets` | `Vec<ExcelSheet>` | — | All sheets in the workbook |
| `metadata` | `HashMap<String, String>` | — | Workbook-level metadata (author, creation date, etc.) |

---

#### ExcelSheet

Single Excel worksheet.

Represents one sheet from an Excel workbook with its content
converted to Markdown format and dimensional statistics.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `name` | `String` | — | Sheet name as it appears in Excel |
| `markdown` | `String` | — | Sheet content converted to Markdown tables |
| `row_count` | `usize` | — | Number of rows |
| `col_count` | `usize` | — | Number of columns |
| `cell_count` | `usize` | — | Total number of non-empty cells |
| `table_cells` | `Vec<Vec<String>>` | `None` | Pre-extracted table cells (2D vector of cell values) Populated during markdown generation to avoid re-parsing markdown. None for empty sheets. |

---

#### EmailAttachment

Email attachment representation.

Contains metadata and optionally the content of an email attachment.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `name` | `Option<String>` | `None` | Attachment name (from Content-Disposition header) |
| `filename` | `Option<String>` | `None` | Filename of the attachment |
| `mime_type` | `Option<String>` | `None` | MIME type of the attachment |
| `size` | `Option<usize>` | `None` | Size in bytes |
| `is_image` | `bool` | — | Whether this attachment is an image |
| `data` | `Option<Vec<u8>>` | `None` | Attachment data (if extracted). Uses `bytes.Bytes` for cheap cloning of large buffers. |

---

#### CacheStats

Cache statistics.

Provides information about the extraction result cache,
including size, file count, and age distribution.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `total_files` | `usize` | — | Total number of cached files |
| `total_size_mb` | `f64` | — | Total cache size in megabytes |
| `available_space_mb` | `f64` | — | Available disk space in megabytes |
| `oldest_file_age_days` | `f64` | — | Age of the oldest cached file in days |
| `newest_file_age_days` | `f64` | — | Age of the newest cached file in days |

---

#### InternalElementId

Deterministic element identifier, generated via blake3 hashing.

Format: `"ie-{12 hex chars}"` (48 bits from blake3, ~281 trillion address space).
Same input always produces the same ID, enabling diffing and caching.

*Opaque type — fields are not directly accessible.*

---

#### InternalElement

A single element in the internal flat document.

Elements are appended in reading order during extraction. The `depth` field
and optional container markers enable tree reconstruction in the derivation step.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `id` | `InternalElementId` | — | Deterministic identifier. |
| `kind` | `ElementKind` | — | What kind of content this element represents. |
| `text` | `String` | — | Primary text content. Empty for non-text elements (images, page breaks). |
| `depth` | `u16` | — | Nesting depth (0 = root level). Extractors set this based on heading level, list indent, blockquote depth, etc. The tree derivation step uses depth changes to reconstruct parent-child relationships. |
| `page` | `Option<u32>` | `None` | Page number (1-indexed). `None` for non-paginated formats. |
| `bbox` | `Option<BoundingBox>` | `None` | Bounding box in document coordinates. |
| `layer` | `ContentLayer` | — | Content layer classification (Body, Header, Footer, Footnote). |
| `annotations` | `Vec<TextAnnotation>` | — | Inline annotations (formatting, links) on this element's text content. Byte-range based, reuses the existing `TextAnnotation` type. |
| `attributes` | `Option<AHashMap>` | `None` | Format-specific key-value attributes. Used for CSS classes, LaTeX env names, slide layout names, etc. |
| `anchor` | `Option<String>` | `None` | Optional anchor/key for this element. Used by the relationship resolver to match references to targets. Examples: heading slug `"introduction"`, footnote label `"fn1"`, citation key `"smith2024"`, figure label `"fig:diagram"`. |
| `ocr_geometry` | `Option<OcrBoundingGeometry>` | `None` | OCR bounding geometry (rectangle or quadrilateral). |
| `ocr_confidence` | `Option<OcrConfidence>` | `None` | OCR confidence scores (detection + recognition). |
| `ocr_rotation` | `Option<OcrRotation>` | `None` | OCR rotation metadata. |

---

#### Relationship

A relationship between two elements in the document.

During extraction, targets may be unresolved keys (`RelationshipTarget.Key`).
The derivation step resolves these to indices using the element anchor index.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `source` | `u32` | — | Index of the source element in `InternalDocument.elements`. |
| `target` | `RelationshipTarget` | — | Target of the relationship (resolved index or unresolved key). |
| `kind` | `RelationshipKind` | — | Semantic kind of the relationship. |

---

#### StructuredData

Structured data (Schema.org, microdata, RDFa) block.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `data_type` | `StructuredDataType` | — | Type of structured data |
| `raw_json` | `String` | — | Raw JSON string representation |
| `schema_type` | `Option<String>` | `None` | Schema type if detectable (e.g., "Article", "Event", "Product") |

---

#### YearRange

Year range for bibliographic metadata.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `min` | `Option<u32>` | `None` | Min |
| `max` | `Option<u32>` | `None` | Max |
| `years` | `Vec<u32>` | — | Years |

---

#### DbfFieldInfo

dBASE field information.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `name` | `String` | — | The name |
| `field_type` | `String` | — | Field type |

---

#### ContributorRole

JATS contributor with role.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `name` | `String` | — | The name |
| `role` | `Option<String>` | `None` | Role |

---

#### PageStructure

Unified page structure for documents.

Supports different page types (PDF pages, PPTX slides, Excel sheets)
with character offset boundaries for chunk-to-page mapping.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `total_count` | `usize` | — | Total number of pages/slides/sheets |
| `unit_type` | `PageUnitType` | — | Type of paginated unit |
| `boundaries` | `Vec<PageBoundary>` | `None` | Character offset boundaries for each page Maps character ranges in the extracted content to page numbers. Used for chunk page range calculation. |
| `pages` | `Vec<PageInfo>` | `None` | Detailed per-page metadata (optional, only when needed) |

---

#### PageBoundary

Byte offset boundary for a page.

Tracks where a specific page's content starts and ends in the main content string,
enabling mapping from byte positions to page numbers. Offsets are guaranteed to be
at valid UTF-8 character boundaries when using standard String methods (push_str, push, etc.).

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `byte_start` | `usize` | — | Byte offset where this page starts in the content string (UTF-8 valid boundary, inclusive) |
| `byte_end` | `usize` | — | Byte offset where this page ends in the content string (UTF-8 valid boundary, exclusive) |
| `page_number` | `usize` | — | Page number (1-indexed) |

---

#### PageInfo

Metadata for individual page/slide/sheet.

Captures per-page information including dimensions, content counts,
and visibility state (for presentations).

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `number` | `usize` | — | Page number (1-indexed) |
| `title` | `Option<String>` | `None` | Page title (usually for presentations) |
| `dimensions` | `Option<(f64, f64)>` | `None` | Dimensions in points (PDF) or pixels (images): (width, height) |
| `image_count` | `Option<usize>` | `None` | Number of images on this page |
| `table_count` | `Option<usize>` | `None` | Number of tables on this page |
| `hidden` | `Option<bool>` | `None` | Whether this page is hidden (e.g., in presentations) |
| `is_blank` | `Option<bool>` | `None` | Whether this page is blank (no meaningful text, no images, no tables) A page is considered blank if it has fewer than 3 non-whitespace characters and contains no tables or images. This is useful for filtering out empty pages in scanned documents or PDFs with blank separator pages. |

---

#### PageContent

Content for a single page/slide.

When page extraction is enabled, documents are split into per-page content
with associated tables and images mapped to each page.

# Performance

Uses Arc-wrapped tables and images for memory efficiency:
- `Vec<Arc<Table>>` enables zero-copy sharing of table data
- `Vec<Arc<ExtractedImage>>` enables zero-copy sharing of image data
- Maintains exact JSON compatibility via custom Serialize/Deserialize

This reduces memory overhead for documents with shared tables/images
by avoiding redundant copies during serialization.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `page_number` | `usize` | — | Page number (1-indexed) |
| `content` | `String` | — | Text content for this page |
| `tables` | `Vec<Table>` | — | Tables found on this page (uses Arc for memory efficiency) Serializes as Vec<Table> for JSON compatibility while maintaining Arc semantics in-memory for zero-copy sharing. |
| `images` | `Vec<ExtractedImage>` | — | Images found on this page (uses Arc for memory efficiency) Serializes as Vec<ExtractedImage> for JSON compatibility while maintaining Arc semantics in-memory for zero-copy sharing. |
| `hierarchy` | `Option<PageHierarchy>` | `None` | Hierarchy information for the page (when hierarchy extraction is enabled) Contains text hierarchy levels (H1-H6) extracted from the page content. |
| `is_blank` | `Option<bool>` | `None` | Whether this page is blank (no meaningful text content) Determined during extraction based on text content analysis. A page is blank if it has fewer than 3 non-whitespace characters and contains no tables or images. |
| `layout_regions` | `Vec<LayoutRegion>` | `None` | Layout detection regions for this page (when layout detection is enabled). Contains detected layout regions with class, confidence, bounding box, and area fraction. Only populated when layout detection is configured. |

---

#### PageHierarchy

Page hierarchy structure containing heading levels and block information.

Used when PDF text hierarchy extraction is enabled. Contains hierarchical
blocks with heading levels (H1-H6) for semantic document structure.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `block_count` | `usize` | — | Number of hierarchy blocks on this page |
| `blocks` | `Vec<HierarchicalBlock>` | — | Hierarchical blocks with heading levels |

---

#### HierarchicalBlock

A text block with hierarchy level assignment.

Represents a block of text with semantic heading information extracted from
font size clustering and hierarchical analysis.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `text` | `String` | — | The text content of this block |
| `font_size` | `f32` | — | The font size of the text in this block |
| `level` | `String` | — | The hierarchy level of this block (H1-H6 or Body) Levels correspond to HTML heading tags: - "h1": Top-level heading - "h2": Secondary heading - "h3": Tertiary heading - "h4": Quaternary heading - "h5": Quinary heading - "h6": Senary heading - "body": Body text (no heading level) |
| `bbox` | `Option<(f32, f32, f32, f32)>` | `None` | Bounding box information for the block Contains coordinates as (left, top, right, bottom) in PDF units. |

---

#### Uri

A URI extracted from a document.

Represents any link, reference, or resource pointer found during extraction.
The `kind` field classifies the URI semantically, while `label` carries
optional human-readable display text.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `url` | `String` | — | The URL or path string. |
| `label` | `Option<String>` | `None` | Optional display text / label for the link. |
| `page` | `Option<u32>` | `None` | Optional page number where the URI was found (1-indexed). |
| `kind` | `UriKind` | — | Semantic classification of the URI. |

---

#### PoolMetricsSnapshot

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `total_acquires` | `usize` | — | Total acquires |
| `total_cache_hits` | `usize` | — | Total cache hits |
| `peak_items_stored` | `usize` | — | Peak items stored |
| `total_creations` | `usize` | — | Total creations |

---

#### Recyclable

Trait for types that can be pooled and reused.

Implementing this trait allows a type to be used with `Pool<T>`.
The `reset()` method should clear the object's state for reuse.

*Opaque type — fields are not directly accessible.*

---

#### StringBufferPool

Convenience type alias for a pooled String.

*Opaque type — fields are not directly accessible.*

---

#### ByteBufferPool

Convenience type alias for a pooled Vec<u8>.

*Opaque type — fields are not directly accessible.*

---

#### Pool

*Opaque type — fields are not directly accessible.*

---

#### PoolSizeHint

Hint for optimal pool sizing based on document characteristics.

This struct contains the estimated sizes for string and byte buffers
that should be allocated in the pool to handle extraction without
excessive reallocation.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `estimated_total_size` | `usize` | — | Estimated total string buffer pool size in bytes |
| `string_buffer_count` | `usize` | — | Recommended number of string buffers |
| `string_buffer_capacity` | `usize` | — | Recommended capacity per string buffer in bytes |
| `byte_buffer_count` | `usize` | — | Recommended number of byte buffers |
| `byte_buffer_capacity` | `usize` | — | Recommended capacity per byte buffer in bytes |

---

#### StringBufferPoolMetrics

Metrics for StringBufferPool (only available with `pool-metrics` feature).

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `total_acquires` | `usize` | — | Total number of acquire calls |
| `total_reuses` | `usize` | — | Total number of buffer reuses from pool |
| `hit_rate` | `f64` | — | Hit rate as percentage (0.0-100.0) |

---

#### PooledString

RAII wrapper for a pooled string buffer.

Automatically returns the buffer to the pool when dropped.

*Opaque type — fields are not directly accessible.*

---

#### InternedString

A reference to an interned string stored in an Arc.

This wraps an Arc<String> and provides convenient access to the string content.
Multiple calls with the same string content will share the same Arc, reducing memory usage.

*Opaque type — fields are not directly accessible.*

---

#### Instant

A platform-aware instant for measuring elapsed time.

On native targets this delegates to `std.time.Instant`.
On `wasm32` targets it is a zero-cost no-op to avoid the `unreachable` trap.

*Opaque type — fields are not directly accessible.*

---

#### HocrWord

Represents a word extracted from hOCR (or any source) with position and confidence information.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `text` | `String` | — | Text |
| `left` | `u32` | — | Left |
| `top` | `u32` | — | Top |
| `width` | `u32` | — | Width |
| `height` | `u32` | — | Height |
| `confidence` | `f64` | — | Confidence |

---

#### ExtractionRequest

A request to extract content from a single document.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `source` | `ExtractionSource` | — | Where to read the document from. |
| `config` | `ExtractionConfig` | — | Base extraction configuration. |
| `file_overrides` | `Option<FileExtractionConfig>` | `None` | Optional per-file overrides (merged on top of `config`). |

---

#### ApiError

API-specific error wrapper.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `status` | `StatusCode` | — | HTTP status code |
| `body` | `ErrorResponse` | — | Error response body |

---

#### ApiDoc

OpenAPI documentation structure.

Defines all endpoints, request/response schemas, and examples
for the Kreuzberg document extraction API.

*Opaque type — fields are not directly accessible.*

---

#### HealthResponse

Health check response.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `status` | `String` | — | Health status |
| `version` | `String` | — | API version |
| `plugins` | `Option<PluginStatus>` | `None` | Plugin status (optional) |

---

#### InfoResponse

Server information response.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `version` | `String` | — | API version |
| `rust_backend` | `bool` | — | Whether using Rust backend |

---

#### ExtractResponse

Extraction response (list of results).

*Opaque type — fields are not directly accessible.*

---

#### ApiState

API server state.

Holds the default extraction configuration loaded from config file
(via discovery or explicit path). Per-request configs override these defaults.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `default_config` | `ExtractionConfig` | — | Default extraction configuration |
| `extraction_service` | `Mutex` | — | Tower service for extraction requests. Wrapped in `Arc<Mutex>` because `BoxCloneService` is `Send` but not `Sync`, while `ApiState` must be `Clone + Sync` for Axum's state requirement. The lock is held only long enough to clone the service. |

---

#### CacheStatsResponse

Cache statistics response.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `directory` | `String` | — | Cache directory path |
| `total_files` | `usize` | — | Total number of cache files |
| `total_size_mb` | `f64` | — | Total cache size in MB |
| `available_space_mb` | `f64` | — | Available disk space in MB |
| `oldest_file_age_days` | `f64` | — | Age of oldest file in days |
| `newest_file_age_days` | `f64` | — | Age of newest file in days |

---

#### CacheClearResponse

Cache clear response.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `directory` | `String` | — | Cache directory path |
| `removed_files` | `usize` | — | Number of files removed |
| `freed_mb` | `f64` | — | Space freed in MB |

---

#### EmbedRequest

Embedding request for generating embeddings from text.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `texts` | `Vec<String>` | — | Text strings to generate embeddings for (at least one non-empty string required) |
| `config` | `Option<EmbeddingConfig>` | `None` | Optional embedding configuration (model, batch size, etc.) |

---

#### EmbedResponse

Embedding response containing generated embeddings.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `embeddings` | `Vec<Vec<f32>>` | — | Generated embeddings (one per input text) |
| `model` | `String` | — | Model used for embedding generation |
| `dimensions` | `usize` | — | Dimensionality of the embeddings |
| `count` | `usize` | — | Number of embeddings generated |

---

#### ChunkRequest

Chunk request with text and configuration.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `text` | `String` | — | Text to chunk (must not be empty) |
| `config` | `Option<ChunkingConfigRequest>` | `None` | Optional chunking configuration |
| `chunker_type` | `String` | — | Chunker type (text, markdown, yaml, or semantic) |

---

#### ChunkResponse

Chunk response with chunks and metadata.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `chunks` | `Vec<ChunkItem>` | — | List of chunks |
| `chunk_count` | `usize` | — | Total number of chunks |
| `config` | `ChunkingConfigResponse` | — | Configuration used for chunking |
| `input_size_bytes` | `usize` | — | Input text size in bytes |
| `chunker_type` | `String` | — | Chunker type used for chunking |

---

#### VersionResponse

Version response.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `version` | `String` | — | Kreuzberg version string |

---

#### DetectResponse

MIME type detection response.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `mime_type` | `String` | — | Detected MIME type |
| `filename` | `Option<String>` | `None` | Original filename (if provided) |

---

#### ManifestEntryResponse

Model manifest entry for cache management.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `relative_path` | `String` | — | Relative path within the cache directory |
| `sha256` | `String` | — | SHA256 checksum of the model file |
| `size_bytes` | `u64` | — | Expected file size in bytes |
| `source_url` | `String` | — | HuggingFace source URL for downloading |

---

#### ManifestResponse

Model manifest response.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `kreuzberg_version` | `String` | — | Kreuzberg version |
| `total_size_bytes` | `u64` | — | Total size of all models in bytes |
| `model_count` | `usize` | — | Number of models in the manifest |
| `models` | `Vec<ManifestEntryResponse>` | — | Individual model entries |

---

#### WarmResponse

Cache warm response.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `cache_dir` | `String` | — | Cache directory used |
| `downloaded` | `Vec<String>` | — | Models that were downloaded |
| `already_cached` | `Vec<String>` | — | Models that were already cached |

---

#### StructuredExtractionResponse

Response from structured extraction endpoint.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `structured_output` | `serde_json::Value` | — | Structured data conforming to the provided JSON schema |
| `content` | `String` | — | Extracted document text content |
| `mime_type` | `String` | — | Detected MIME type of the input file |

---

#### DoclingCompatResponse

OpenWebUI "Docling" engine response format.

Returned by `POST /v1/convert/file` for docling-serve compatibility.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `document` | `DoclingCompatDocument` | — | Converted document content |
| `status` | `String` | — | Processing status |

---

#### ExtractFileParams

Request parameters for file extraction.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `path` | `String` | — | Path to the file to extract |
| `mime_type` | `Option<String>` | `None` | Optional MIME type hint (auto-detected if not provided) |
| `config` | `Option<serde_json::Value>` | `None` | Extraction configuration (JSON object) |
| `pdf_password` | `Option<String>` | `None` | Password for encrypted PDFs |
| `response_format` | `Option<String>` | `None` | Wire format for the response: "json" (default) or "toon" |

---

#### ExtractBytesParams

Request parameters for bytes extraction.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `data` | `String` | — | Base64-encoded file content |
| `mime_type` | `Option<String>` | `None` | Optional MIME type hint (auto-detected if not provided) |
| `config` | `Option<serde_json::Value>` | `None` | Extraction configuration (JSON object) |
| `pdf_password` | `Option<String>` | `None` | Password for encrypted PDFs |
| `response_format` | `Option<String>` | `None` | Wire format for the response: "json" (default) or "toon" |

---

#### BatchExtractFilesParams

Request parameters for batch file extraction.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `paths` | `Vec<String>` | — | Paths to files to extract |
| `config` | `Option<serde_json::Value>` | `None` | Extraction configuration (JSON object) |
| `pdf_password` | `Option<String>` | `None` | Password for encrypted PDFs |
| `file_configs` | `Vec<Option<serde_json::Value>>` | `None` | Per-file extraction configuration overrides (parallel array to paths). Each entry is either null (use default) or a FileExtractionConfig JSON object. |
| `response_format` | `Option<String>` | `None` | Wire format for the response: "json" (default) or "toon" |

---

#### DetectMimeTypeParams

Request parameters for MIME type detection.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `path` | `String` | — | Path to the file |
| `use_content` | `bool` | — | Use content-based detection (default: true) |

---

#### CacheWarmParams

Request parameters for cache warm (model download).

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `all_embeddings` | `bool` | — | Download all embedding model presets |
| `embedding_model` | `Option<String>` | `None` | Specific embedding preset name to download (e.g. "balanced", "speed", "quality") |

---

#### EmbedTextParams

Request parameters for embedding generation.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `texts` | `Vec<String>` | — | List of text strings to generate embeddings for |
| `preset` | `Option<String>` | `None` | Embedding preset name (default: "balanced"). Available: "speed", "balanced", "quality" |
| `model` | `Option<String>` | `None` | LLM model for provider-hosted embeddings (e.g., "openai/text-embedding-3-small"). When set, overrides preset and uses liter-llm for embedding generation. |
| `api_key` | `Option<String>` | `None` | API key for the LLM provider (optional, falls back to env). |

---

#### ExtractStructuredParams

Request parameters for LLM-based structured extraction.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `path` | `String` | — | File path to extract from |
| `schema` | `serde_json::Value` | — | JSON schema for structured output |
| `model` | `String` | — | LLM model (e.g., "openai/gpt-4o") |
| `schema_name` | `String` | — | Schema name (default: "extraction") |
| `schema_description` | `Option<String>` | `None` | Schema description for the LLM |
| `prompt` | `Option<String>` | `None` | Custom Jinja2 prompt template |
| `api_key` | `Option<String>` | `None` | API key (optional, falls back to env) |
| `strict` | `bool` | — | Enable strict mode |

---

#### ChunkTextParams

Request parameters for text chunking.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `text` | `String` | — | Text content to split into chunks |
| `max_characters` | `Option<usize>` | `None` | Maximum characters per chunk (default: 2000) |
| `overlap` | `Option<usize>` | `None` | Number of overlapping characters between chunks (default: 100) |
| `chunker_type` | `Option<String>` | `None` | Chunker type: "text", "markdown", "yaml", or "semantic" (default: "text") |
| `topic_threshold` | `Option<f32>` | `None` | Topic threshold for semantic chunking (0.0-1.0, default: 0.75) |

---

#### DetectedBoundary

A detected structural boundary in the text.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `byte_offset` | `usize` | — | Byte offset of the start of the line in the original text. |
| `is_header` | `bool` | — | Whether this boundary looks like a header/section title. |

---

#### ChunkingProcessor

Post-processor that chunks text in document content.

This processor:
- Runs in the Middle processing stage
- Only processes when `config.chunking` is configured
- Stores chunks in `result.chunks`
- Uses configurable chunk size and overlap

*Opaque type — fields are not directly accessible.*

---

#### MergedChunk

A merged chunk produced by `merge_segments`.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `text` | `String` | — | Text |
| `byte_start` | `usize` | — | Byte start |
| `byte_end` | `usize` | — | Byte end |

---

#### EmbeddingEngine

Text embedding model with thread-safe inference.

The `embed()` method takes `&self` instead of `&mut self`, allowing it to
be shared across threads via `Arc<EmbeddingEngine>` without mutex contention.

*Opaque type — fields are not directly accessible.*

---

#### EmbeddingPreset

Preset configurations for common RAG use cases.

Each preset combines chunk size, overlap, and embedding model
to provide an optimized configuration for specific scenarios.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `name` | `String` | — | The name |
| `chunk_size` | `usize` | — | Chunk size |
| `overlap` | `usize` | — | Overlap |
| `model_repo` | `String` | — | HuggingFace repository name for the model. |
| `pooling` | `String` | — | Pooling strategy: "cls" or "mean". |
| `model_file` | `String` | — | Path to the ONNX model file within the repo. |
| `dimensions` | `usize` | — | Dimensions |
| `description` | `String` | — | Human-readable description |

---

#### LanguageDetector

Post-processor that detects languages in document content.

This processor:
- Runs in the Early processing stage
- Only processes when `config.language_detection` is configured
- Stores detected languages in `result.detected_languages`
- Uses the whatlang library for detection

*Opaque type — fields are not directly accessible.*

---

#### KeywordExtractor

Post-processor that extracts keywords from document content.

This processor:
- Runs in the Middle processing stage
- Only processes when `config.keywords` is configured
- Stores extracted keywords in `metadata.additional["keywords"]`
- Uses the configured algorithm (YAKE or RAKE)

*Opaque type — fields are not directly accessible.*

---

#### Keyword

Extracted keyword with metadata.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `text` | `String` | — | The keyword text. |
| `score` | `f32` | — | Relevance score (higher is better, algorithm-specific range). |
| `algorithm` | `KeywordAlgorithm` | — | Algorithm that extracted this keyword. |
| `positions` | `Vec<usize>` | `None` | Optional positions where keyword appears in text (character offsets). |

---

#### TsvRow

Tesseract TSV row data for conversion.

This struct represents a single row from Tesseract's TSV output format.
TSV format includes hierarchical information (block, paragraph, line, word)
along with bounding boxes and confidence scores.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `level` | `i32` | — | Hierarchical level (1=block, 2=para, 3=line, 4=word, 5=symbol) |
| `page_num` | `i32` | — | Page number (1-indexed) |
| `block_num` | `i32` | — | Block number within page |
| `par_num` | `i32` | — | Paragraph number within block |
| `line_num` | `i32` | — | Line number within paragraph |
| `word_num` | `i32` | — | Word number within line |
| `left` | `u32` | — | Left x-coordinate in pixels |
| `top` | `u32` | — | Top y-coordinate in pixels |
| `width` | `u32` | — | Width in pixels |
| `height` | `u32` | — | Height in pixels |
| `conf` | `f64` | — | Confidence score (0-100) |
| `text` | `String` | — | Recognized text |

---

#### TessdataManager

Manages tessdata file downloading, caching, and manifest generation.

*Opaque type — fields are not directly accessible.*

---

#### ResolvedRecModel

Resolved recognition model with engine pool key for sharing.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `model_dir` | `PathBuf` | — | Directory containing model.onnx. |
| `dict_file` | `PathBuf` | — | Path to the character dictionary file. |
| `model_key` | `String` | — | Engine pool key for sharing engines across script families. Multiple families may share the same key (e.g. chinese and japanese both map to "v2:unified_server" when using server tier). |

---

#### SharedModelPaths

Paths to shared models (detection + classification).

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `det_model` | `PathBuf` | — | Path to the detection model directory. |
| `cls_model` | `PathBuf` | — | Path to the classification model directory. |

---

#### RecModelPaths

Paths to a recognition model and its character dictionary.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `rec_model` | `PathBuf` | — | Path to the recognition model directory. |
| `dict_file` | `PathBuf` | — | Path to the character dictionary file. |

---

#### ModelPaths

Combined paths to all models needed for OCR (backward compatibility).

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `det_model` | `PathBuf` | — | Path to the detection model directory. |
| `cls_model` | `PathBuf` | — | Path to the classification model directory. |
| `rec_model` | `PathBuf` | — | Path to the recognition model directory. |
| `dict_file` | `PathBuf` | — | Path to the character dictionary file. |

---

#### ModelManifestEntry

A single model file entry in the cache manifest.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `relative_path` | `String` | — | Relative path within the cache directory (e.g., "paddle-ocr/det/model.onnx"). |
| `sha256` | `String` | — | SHA256 checksum of the model file. |
| `size_bytes` | `u64` | — | Expected file size in bytes. |
| `source_url` | `String` | — | HuggingFace source URL for downloading. |

---

#### ModelManager

Manages PaddleOCR model downloading, caching, and path resolution.

The model manager ensures that PaddleOCR models are available locally,
organized by model type. Shared models (det, cls) are downloaded once,
while recognition models are downloaded per-script-family on demand.

*Opaque type — fields are not directly accessible.*

---

#### DocOrientationDetector

Detects document page orientation using the PP-LCNet model.

Thread-safe: uses unsafe pointer cast for ONNX session (same pattern as embedding engine).
The model is downloaded from HuggingFace on first use and cached locally.

*Opaque type — fields are not directly accessible.*

---

#### LayoutEngine

High-level layout detection engine.

Wraps model loading, inference, and postprocessing into a single
reusable object. Models are downloaded and cached on first use.

*Opaque type — fields are not directly accessible.*

---

#### LayoutModelManager

Manages layout model downloading, caching, and path resolution.

*Opaque type — fields are not directly accessible.*

---

#### RtDetrModel

Docling RT-DETR v2 layout detection model.

This model is NMS-free (transformer-based end-to-end detection).

Input tensors:
  - `images`:            f32 [batch, 3, 640, 640]  (preprocessed pixel data)
  - `orig_target_sizes`: i64 [batch, 2]            ([height, width] of original image)

Output tensors:
  - `labels`: i64 [batch, num_queries]   (class IDs, 0-16)
  - `boxes`:  f32 [batch, num_queries, 4] (bounding boxes in original image coordinates)
  - `scores`: f32 [batch, num_queries]   (confidence scores)

*Opaque type — fields are not directly accessible.*

---

#### SlanetCell

A single cell detected by SLANeXT.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `polygon` | `[f32;8]` | — | Bounding box polygon in image pixel coordinates. Format: [x1, y1, x2, y2, x3, y3, x4, y4] (4 corners, clockwise from top-left). |
| `bbox` | `[f32;4]` | — | Axis-aligned bounding box derived from polygon: [left, top, right, bottom]. |
| `row` | `usize` | — | Row index in the table (0-based). |
| `col` | `usize` | — | Column index within the row (0-based). |

---

#### SlanetModel

SLANeXT table structure recognition model.

Wraps an ORT session for SLANeXT ONNX model and provides preprocessing,
inference, and post-processing in a single `recognize` call.

*Opaque type — fields are not directly accessible.*

---

#### TatrDetection

A single TATR detection result.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `bbox` | `[f32;4]` | — | Bounding box in crop-pixel coordinates: `[x1, y1, x2, y2]`. |
| `confidence` | `f32` | — | Detection confidence score (0.0..1.0). |
| `class` | `TatrClass` | — | Detected class. |

---

#### CellBBox

A cell bounding box within the reconstructed table grid.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `x1` | `f32` | — | X1 |
| `y1` | `f32` | — | Y1 |
| `x2` | `f32` | — | X2 |
| `y2` | `f32` | — | Y2 |

---

#### TatrModel

TATR (Table Transformer) table structure recognition model.

Wraps an ORT session for the TATR ONNX model and provides preprocessing,
inference, and post-processing in a single `recognize` call.

*Opaque type — fields are not directly accessible.*

---

#### YoloModel

YOLO-family layout detection model (YOLOv10, DocLayout-YOLO, YOLOX).

*Opaque type — fields are not directly accessible.*

---

#### LayoutModel

Common interface for all layout detection model backends.

*Opaque type — fields are not directly accessible.*

---

#### BBox

Bounding box in original image coordinates (x1, y1) top-left, (x2, y2) bottom-right.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `x1` | `f32` | — | X1 |
| `y1` | `f32` | — | Y1 |
| `x2` | `f32` | — | X2 |
| `y2` | `f32` | — | Y2 |

---

#### LayoutDetection

A single layout detection result.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `class` | `LayoutClass` | — | Class (layout class) |
| `confidence` | `f32` | — | Confidence |
| `bbox` | `BBox` | — | Bbox (b box) |

---

#### EmbeddedFile

Embedded file descriptor extracted from the PDF name tree.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `name` | `String` | — | The filename as stored in the PDF name tree. |
| `data` | `Vec<u8>` | — | Raw file bytes from the embedded stream. |
| `mime_type` | `Option<String>` | `None` | MIME type if specified in the filespec, otherwise `None`. |

---

#### FontSizeCluster

A cluster of text blocks with the same font size characteristics.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `centroid` | `f32` | — | The centroid (mean) font size of this cluster |
| `members` | `Vec<TextBlock>` | — | The text blocks that belong to this cluster |

---

#### CharData

Character information extracted from PDF with font metrics.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `text` | `String` | — | The character text content |
| `x` | `f32` | — | X position in PDF units |
| `y` | `f32` | — | Y position in PDF units |
| `font_size` | `f32` | — | Font size in points |
| `width` | `f32` | — | Character width in PDF units |
| `height` | `f32` | — | Character height in PDF units |
| `is_bold` | `bool` | — | Whether the font is bold (from pdfium force-bold flag) |
| `is_italic` | `bool` | — | Whether the font is italic |
| `baseline_y` | `f32` | — | Baseline Y position (from character origin, falls back to bounds bottom) |

---

#### TextBlock

A block of text with spatial and semantic information.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `text` | `String` | — | The text content |
| `bbox` | `BoundingBox` | — | The bounding box of the block |
| `font_size` | `f32` | — | The font size of the text in this block |

---

#### HierarchyBlock

A TextBlock with hierarchy level assignment.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `text` | `String` | — | The text content |
| `bbox` | `BoundingBox` | — | The bounding box of the block |
| `font_size` | `f32` | — | The font size of the text in this block |
| `hierarchy_level` | `HierarchyLevel` | — | The hierarchy level of this block (H1-H6 or Body) |

---

#### SegmentData

Text segment data extracted from PDF using pdfium's pre-merged segments.

Pdfium merges characters sharing the same baseline and font settings into segments,
providing correct word boundaries without gap-based heuristics. Each segment contains
the full text run, bounding box, and font metadata sampled from the first character.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `text` | `String` | — | The segment text content (may contain spaces / multiple words) |
| `x` | `f32` | — | Left x position in PDF units |
| `y` | `f32` | — | Bottom y position in PDF units (PDF coordinate system, y=0 at bottom) |
| `width` | `f32` | — | Width of the segment bounding box |
| `height` | `f32` | — | Height of the segment bounding box |
| `font_size` | `f32` | — | Font size in points (from first character) |
| `is_bold` | `bool` | — | Whether the font is bold |
| `is_italic` | `bool` | — | Whether the font is italic |
| `is_monospace` | `bool` | — | Whether the font is monospace (e.g. Courier, Consolas) |
| `baseline_y` | `f32` | — | Baseline Y position (from first character origin, falls back to bounds bottom) |
| `assigned_role` | `Option<u8>` | `None` | Pre-assigned heading level from the PDF structure tree (1-6), or `None` when the heading level is unknown and must be inferred via font-size clustering. |

---

#### PdfImage

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `page_number` | `usize` | — | Page number |
| `image_index` | `usize` | — | Image index |
| `width` | `i64` | — | Width |
| `height` | `i64` | — | Height |
| `color_space` | `Option<String>` | `None` | Color space |
| `bits_per_component` | `Option<i64>` | `None` | Bits per component |
| `filters` | `Vec<String>` | — | Original PDF stream filters (e.g. `["FlateDecode"]`, `["DCTDecode"]`). |
| `data` | `Vec<u8>` | — | The decoded image bytes in a standard format (JPEG, PNG, etc.). |
| `decoded_format` | `String` | — | The format of `data` after decoding: `"jpeg"`, `"png"`, `"jpeg2000"`, `"ccitt"`, or `"raw"`. |

---

#### PdfImageExtractor

*Opaque type — fields are not directly accessible.*

---

#### PdfLayoutBBox

Bounding box in PDF coordinate space (points, y=0 at bottom of page).

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `left` | `f32` | — | Left |
| `bottom` | `f32` | — | Bottom |
| `right` | `f32` | — | Right |
| `top` | `f32` | — | Top |

---

#### PageLayoutRegion

A detected layout region mapped to PDF coordinate space.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `class` | `LayoutClass` | — | Class (layout class) |
| `confidence` | `f32` | — | Confidence |
| `bbox` | `PdfLayoutBBox` | — | Bbox (pdf layout b box) |

---

#### PageTiming

Timing breakdown for a single page.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `render_ms` | `f64` | — | Time to render the PDF page to a raster image (amortized from batch render). |
| `preprocess_ms` | `f64` | — | Time spent in image preprocessing (resize, normalize, tensor construction). |
| `onnx_ms` | `f64` | — | Time for the ONNX model session.run() call (actual neural network inference). |
| `inference_ms` | `f64` | — | Total model inference time (preprocess + onnx), as measured by the engine. |
| `postprocess_ms` | `f64` | — | Time spent in postprocessing (confidence filtering, overlap resolution). |
| `mapping_ms` | `f64` | — | Time to map pixel-space bounding boxes to PDF coordinate space. |

---

#### LayoutTimingReport

Timing breakdown for the entire layout detection run.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `total_ms` | `f64` | — | Total ms |
| `per_page` | `Vec<PageTiming>` | — | Per page |

---

#### PdfPageIterator

Lazy page-by-page PDF renderer.

Reads the file once at construction and yields one PNG-encoded page per
`next()` call. Only one rendered page is held in memory at a time.

The PDFium mutex is acquired and released per page, so other PDF
operations can proceed between iterations. This makes the iterator
safe to use in long-running loops (e.g., sending each page to a vision
model for OCR) without blocking all PDF processing.

Use the iterator when memory is a concern or when you want to process
pages as they are rendered.

*Opaque type — fields are not directly accessible.*

---

#### PdfRenderer

*Opaque type — fields are not directly accessible.*

---

#### PdfTextExtractor

*Opaque type — fields are not directly accessible.*

---
