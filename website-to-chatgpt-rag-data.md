---
title: "How to turn a whole website into data for ChatGPT, custom GPTs and RAG"
description: "Crawl every page of a website (and its PDFs) into clean Markdown or JSON you can upload to a custom GPT, Claude Project or vector database. Three ways: free code, a no-code crawler, or done for you."
permalink: /website-to-chatgpt-rag-data/
---

# How to turn a whole website into data for ChatGPT and RAG

A support bot, custom GPT or internal assistant is only as good as the content you give it. Raw HTML is full of menus, footers and cookie banners that waste tokens and confuse retrieval. The goal is **one clean Markdown document per page**, with headings preserved so chunking keeps context.

## Option 1: No-code crawler on Apify

[Website Content Crawler](https://apify.com/apify/website-content-crawler?fpr=kyk4x5) (about 178,000 users) crawls a site from a start URL, renders JavaScript when needed, strips boilerplate and outputs Markdown or text for every page. It also connects directly to Pinecone, Qdrant and LangChain.

1. [Create a free Apify account](https://apify.com?fpr=kyk4x5); the free plan's monthly credit covers small sites.
2. Open [Website Content Crawler](https://apify.com/apify/website-content-crawler?fpr=kyk4x5), paste your start URL and set the maximum pages.
3. Run it and download the dataset as JSON, or send it straight to your vector database with an integration.

## Option 2: Free Python (sitemap + Markdown extraction)

Most sites list every page in `sitemap.xml`. Read it, then extract the main content of each page:

```python
import re, httpx, trafilatura

site = "https://example.com"
xml = httpx.get(f"{site}/sitemap.xml", follow_redirects=True).text
urls = re.findall(r"<loc>(.*?)</loc>", xml)[:200]
for url in urls:
    html = httpx.get(url, follow_redirects=True).text
    md = trafilatura.extract(html, output_format="markdown", include_tables=True, favor_precision=True)
    if md:
        open(re.sub(r"[^a-z0-9]+", "-", url.lower())[:100] + ".md", "w").write(md)
```

Pitfalls: sitemap indexes point to more sitemaps (follow them), some pages render only in the browser, and PDFs need a separate text extractor. See [web page to Markdown](web-page-to-markdown.md), [extract URLs from a sitemap](extract-urls-from-sitemap.md) and [PDF URL to text](pdf-url-to-text.md) for each piece. For single pages, [Article Extractor](https://apify.com/signalfeed/article-extractor?fpr=kyk4x5) does it at $0.50 per 1,000 pages.

## Option 3: Done for you

I convert websites, help centers and PDF libraries into chunk-ready Markdown and JSONL for custom GPTs, Claude Projects and vector databases: **[AI-ready data from your website and PDFs on Fiverr](https://www.fiverr.com/arabinowe/convert-your-website-and-pdfs-into-ai-ready-markdown-for-chatgpt-or-rag)**. PDFs only? See [PDFs to clean Markdown or JSON](https://www.fiverr.com/arabinowe/convert-your-pdfs-into-clean-markdown-or-json-for-rag-and-llms).

## Tips for better answers

- Keep one document per page with its URL, so the assistant can cite sources.
- Drop navigation, footers and legal boilerplate; they show up in every page and drown out real content.
- Preserve headings and tables: chunkers split on headings, and tables carry facts.
- Re-crawl on a schedule so the assistant doesn't answer from stale pages.

*Disclosure: Apify links are affiliate links (I earn a commission if you upgrade to a paid plan, at no cost to you); Article Extractor and the Fiverr service are mine.*

[← Home](README.md)
