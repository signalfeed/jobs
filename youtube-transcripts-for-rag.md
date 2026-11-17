---
title: "How to get YouTube transcripts in bulk for ChatGPT, RAG and search (Python)"
description: "Pull the captions from any list of public YouTube videos, merge them into timestamped passages, and save chunk-ready JSONL for a chatbot or vector database. Working Python, limits, and a no-code option."
permalink: /youtube-transcripts-for-rag/
---

# How to get YouTube transcripts in bulk for ChatGPT and RAG

Video libraries are full of knowledge that never makes it into a chatbot: webinars, product demos, course lessons, conference talks. Most public YouTube videos already have captions (uploaded or auto-generated), so you can turn a channel into searchable text without transcribing anything yourself.

The useful unit for retrieval isn't the raw caption line (2 to 5 seconds of speech) but a **passage of about a minute**, with a link that jumps to that moment in the video so answers can cite their source.

## Python: transcripts to timestamped passages

[youtube-transcript-api](https://pypi.org/project/youtube-transcript-api/) reads the captions YouTube already shows; no API key needed.

```python
# pip install youtube-transcript-api
import json
from youtube_transcript_api import YouTubeTranscriptApi

def transcript_chunks(video_id, seconds=60, languages=("en",)):
    """Merge caption lines into ~60-second passages, each with a timestamped link back to the video."""
    snippets = YouTubeTranscriptApi().fetch(video_id, languages=languages).snippets
    chunks, buf, start = [], [], 0.0
    for s in snippets:
        if not buf:
            start = s.start
        buf.append(s.text.replace("\n", " "))
        if s.start + s.duration - start >= seconds:
            chunks.append({"video": video_id, "start": int(start), "text": " ".join(buf),
                           "url": f"https://www.youtube.com/watch?v={video_id}&t={int(start)}s"})
            buf = []
    if buf:
        chunks.append({"video": video_id, "start": int(start), "text": " ".join(buf),
                       "url": f"https://www.youtube.com/watch?v={video_id}&t={int(start)}s"})
    return chunks

with open("transcripts.jsonl", "w") as f:
    for vid in ["VIDEO_ID_1", "VIDEO_ID_2"]:  # the part after watch?v= in each URL
        for c in transcript_chunks(vid):
            f.write(json.dumps(c, ensure_ascii=False) + "\n")
```

Each line of `transcripts.jsonl` is one passage: the video ID, the start second, the merged text and a deep link such as `https://www.youtube.com/watch?v=VIDEO_ID&t=120s`. Load it into any vector database and pass the `url` through to the answer, so users can click straight to the moment being quoted.

Tested November 2026 with youtube-transcript-api 1.x (`YouTubeTranscriptApi().fetch(...)`; older tutorials use the removed `get_transcript` class method).

## Things that break

- **No captions:** some videos have captions turned off. The library raises an error; skip and log those IDs.
- **Language:** pass `languages=("en", "en-US")` or another code. Auto-generated captions exist for most spoken languages, but quality varies.
- **Music and noise:** auto-captions mark them as `[Music]` or `[♪♪♪]`; drop passages that are mostly bracketed tags.
- **Volume:** YouTube rate-limits and often blocks requests from cloud servers. A few hundred videos from a home connection is usually fine; thousands need pauses, or a hosted tool.
- **Rights:** captions are the creator's content. Transcribing your own channel, licensed courses or videos for private research is fine; republishing someone else's transcript usually isn't.

## Skip the build

- **No-code, any volume:** the [Apify Store has a dozen YouTube transcript scrapers](https://apify.com/store?search=youtube%20transcript&fpr=kyk4x5) that take channel, playlist or video URLs and handle proxies; prices run from under $1 to about $40 per 1,000 videos, so compare before you pick. *(Affiliate link.)*
- **Done for you:** I turn video libraries, websites and PDFs into clean, chunk-ready datasets for chatbots and RAG: [AI-ready knowledge base data on Fiverr](https://www.fiverr.com/arabinowe/convert-your-website-and-pdfs-into-ai-ready-markdown-for-chatgpt-or-rag).

Related: [turn a website into RAG data](website-to-chatgpt-rag-data.md) · [PDFs to clean Markdown for RAG](pdf-to-markdown-for-rag.md)

[← Home](README.md)
