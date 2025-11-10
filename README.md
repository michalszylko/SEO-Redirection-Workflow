# SEO-Redirection-Workflow
A repository of the created workflow for creating the new URL structure, along with the new URL's based on the internal RAG.

Built an n8n + LLM + RAG workflow that automatically maps thousands of legacy marketplace URLs to a new multi-country URL structure with validation, turning a multi-week manual task into a repeatable, machine-audited system. 
The solution encodes complex business rules (clusters, markets, cuisines, brands) directly into vector stores and AI agents, ensuring consistency and protecting organic revenue during migrations. 

# Redirection Tool Workflow

Automated SEO redirection & URL mapping engine built on **n8n + LLMs + Google Sheets**.

This repository contains an n8n workflow (`Redirection Tool Workflow.json`) that:

1. Reads legacy URLs from Google Sheets
2. Parses and classifies them (market, page type, entities)
3. Generates a **single-line CSV mapping** for each URL based on strict cluster & structure rules
4. Validates generated redirect destinations against predefined **Structure Rules** & **Cuisine Mapping**
5. Outputs ready-to-use CSV rows for redirect plans

It is designed for large marketplaces / multi-market platforms (e.g. food delivery) migrating to a new URL structure while keeping redirects **consistent, multilingual, and rule-based**.

---

## Features

* 🔁 **Automated redirect mapping** from legacy URLs to new structured URLs
* 🌍 **Cluster-aware logic** (CH, BE, LU, DE clusters + standalone markets)
* 🍕 **Cuisine mapping engine** (multilingual cuisine slug resolver via vector store)
* 🏙️ Stable handling of **city**, **brand**, and **cuisine** entities
* ✅ **LLM-based QA layer** verifying each redirect against:

  * Page type rules
  * Market & language patterns
  * URL path templates
  * Cuisine slug translations
* 📄 Output as **single-line CSV rows**, ready for export or implementation

---

## High-Level Architecture

The workflow is defined in `Redirection Tool Workflow.json` and consists of:

1. **Manual Trigger**

   * Node: `When clicking 'Execute workflow'`
   * Starts the process on demand inside n8n.

2. **Source Data Loader**

   * Node: `Get row(s) in sheet`
   * Reads URLs and metadata from a Google Sheet.
   * Uses:

     * `documentId` → your redirect input sheet
     * `sheetName` → target sheet/tab (e.g. `V2`)

3. **Batch Processor**

   * Node: `Loop Over Items` (`splitInBatches`)
   * Iterates through each input row.

4. **AI Redirect Generator (Parser & Architect)**

   * Node: `AI Agent` (backed by `OpenRouter Chat Model` + `Simple Vector Store`)
   * Responsibilities:

     * Parse `source_url` & `market`
     * Detect entities: `city`, `cuisine`, `brand`
     * Determine `Page Type`
     * Resolve **Market Cluster** based on:

       * CH-Cluster, BE-Cluster, LU-Cluster, DE-Cluster, Standalone markets, etc.
     * Use **Structure Rules Table** to construct correct destination paths
     * Use **Cuisine Mapping Table** via vector store to:

       * Match longest cuisine slug
       * Translate cuisine slugs per target language
     * Output: **single CSV line** (no header) in format:

       ```text
       {Source URL},{Cluster Name},{Page Type},{Entity Names},{Destination 1},{Destination 2},{Destination 3},{Destination 4}
       ```

5. **AI QA Validator**

   * Node: `AI Agent1` (backed by `OpenRouter Chat Model1` + `Simple Vector Store1`)
   * Responsibilities:

     * Take generated CSV line as input
     * Validate each destination URL against:

       * Structure Rules
       * Cuisine Mapping
       * Page type expectations
     * Mark & correct invalid patterns
     * Return **corrected single-line CSV** as final output
   * Enforced constraints:

     * No markdown
     * No explanations
     * Output = raw CSV line only

6. **File Export**

   * Node: `Convert to File`
   * Converts final CSV line output into a file-like binary (for download/export from n8n).

7. **Vector Stores & Embeddings**

   * Nodes:

     * `Simple Vector Store`, `Simple Vector Store1`
     * `Embeddings OpenAI`, `Embeddings OpenAI1`
   * Store and expose:

     * Structure rules
     * Cuisine mapping table
   * Used as tools by AI Agents for **RAG-style lookups** (no external guessing).

---

## Prerequisites

To run this workflow, you need:

* **n8n** (self-hosted or cloud)
* **Google Sheets API credentials**

  * Configured as `Google Sheets account` in n8n
* **OpenRouter API key**

  * Saved as `OpenRouter account` credential
* **OpenAI API key**

  * Saved as `OpenAi account 2` credential
* Access to the **input Google Sheet** with:

  * Legacy URLs
  * Markets / language codes (if required by your variant)

---

## Installation

1. Export/download `Redirection Tool Workflow.json` from this repository.
2. In your n8n instance:

   * Go to **Workflows → Import from file**
   * Select `Redirection Tool Workflow.json`.
3. Configure credentials:

   * Set your **Google Sheets OAuth2** credential.
   * Set your **OpenRouter** credential (for `gpt-4o-mini` or compatible).
   * Set your **OpenAI** credential for embeddings.
4. Update node parameters where needed:

   * `documentId` → your Sheet ID
   * `sheetName` / `gid` → your tab
   * Confirm vector store prompt text matches your actual structure & cuisine tables.

---

## How It Works (Step-by-Step)

1. You click **Execute Workflow** in n8n.
2. Workflow pulls all rows from the configured Google Sheet.
3. For each row:

   * `AI Agent`:

     * Parses the URL.
     * Identifies market & cluster.
     * Resolves entities & page type.
     * Builds cluster-wide destination URLs.
     * Returns a single CSV line.
4. `AI Agent1`:

   * Validates the CSV line using:

     * Structure Rules Table
     * Cuisine Mapping Table
   * Fixes any mismatches.
   * Returns a **clean, final CSV line**.
5. `Convert to File`:

   * Packages output in a file format suitable for download/logging.

The end result: **consistent, audited, machine-generated redirect rows** ready for implementation.

---

## CSV Output Specification

Final output row (no headers):

```text
SourceURL,ClusterName,PageType,EntityNames,Destination1,Destination2,Destination3,Destination4
```

Where:

* `SourceURL` – original legacy URL
* `ClusterName` – e.g. `CH-Cluster`, `BE-Cluster`, `DA-DK`
* `PageType` – e.g. `Location`, `Location Cuisines`, `Brands`, `City Brands`
* `EntityNames` – semicolon-separated, e.g. `basel;pizza`, `antwerpen;mcdonalds`
* `Destination1..4` – ordered by the cluster market order; trailing empty if fewer markets

All logic is **fully driven by the rules encoded in the system prompts & vector stores**.

---

## Customization

You can adapt this workflow to your stack by:

* Updating:

  * Market clusters
  * Structure Rules table
  * Cuisine Mapping table
* Pointing to a different Google Sheet
* Swapping models (e.g. different OpenRouter/OpenAI models)
* Extending the QA layer with additional checks (status codes, duplicates, etc.)

---

## Limitations

* Assumes:

  * City and brand slugs stay consistent across markets.
  * Some of the cuisines require translation/mapping.
    
* Depends on:

  * Correctly formatted input sheet.
  * Accurate rules encoded in prompts/vector stores.
*No live crawling or external validation: all decisions are rule-based + provided knowledge.

---
