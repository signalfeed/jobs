---
title: "How to get Indeed job listings as data (CSV or JSON), and a fresher alternative"
description: "Export Indeed job postings by keyword and location without coding, what it costs per 1,000 jobs, and how to get the same jobs earlier and cleaner straight from employers' career pages."
permalink: /indeed-jobs-data/
---

# How to get Indeed job listings as data

Indeed has no open public API for job search, so job boards, recruiters and researchers usually export listings with a scraper. Here's the quickest route, and an alternative that's often better.

## Option 1: Indeed Scraper on Apify

[Indeed Scraper](https://apify.com/misceres/indeed-scraper?fpr=kyk4x5) (about 33,000 users) exports listings for any keyword and location: title, company, location, salary when shown, posting date, description and the job URL.

1. [Create a free Apify account](https://apify.com?fpr=kyk4x5); the monthly free credit covers a test run.
2. Open [Indeed Scraper](https://apify.com/misceres/indeed-scraper?fpr=kyk4x5), enter a position (e.g. "data engineer"), a location and a maximum number of results.
3. Run it and download CSV, Excel or JSON, or schedule it daily.

**Cost:** about **$6 per 1,000 jobs** (November 2026).

## Option 2: Go straight to the employer (fresher, cleaner, cheaper)

Indeed mostly re-posts jobs that companies publish on their own career pages, which usually run on Greenhouse, Lever, Ashby or Workday. Those systems expose **public JSON endpoints**, so you can read the original posting directly:

- **Fresher:** jobs appear on the employer's board first, often days before aggregators.
- **No duplicates or sponsored noise:** one posting per job, from the source.
- **Better fields:** structured department, workplace type and, often, salary ranges.

The free guide shows each endpoint with code: [Greenhouse](greenhouse-jobs-api.md), [Lever](lever-jobs-api.md), [Ashby](ashby-jobs-api.md), [Workday](workday-jobs-api.md) and more.

To search thousands of these boards at once by keyword, use [Tech & Startup Job Search](https://apify.com/signalfeed/tech-job-search?fpr=kyk4x5) (4,400+ company career pages, $2 per 1,000 jobs), or see [this week's companies hiring software engineers](companies-hiring-this-week.md).

## Which should you use?

| Need | Best option |
|---|---|
| Every job in a city, including small local employers | Indeed Scraper |
| Tech and startup jobs, freshest possible | Career-page APIs or Tech & Startup Job Search |
| Salary data | Career pages (Ashby and Greenhouse often publish ranges); see [salary data from job postings](salary-data-from-job-postings.md) |
| Leads: which companies are hiring for a role | [Hiring Companies Finder](https://apify.com/signalfeed/hiring-companies-finder?fpr=kyk4x5) |

*Disclosure: Apify links are affiliate links (I earn a commission if you upgrade to a paid plan, at no cost to you); the signalfeed tools are mine.*

[← Home](README.md)
