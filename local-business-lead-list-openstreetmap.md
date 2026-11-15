---
title: "How to build a free local business lead list with OpenStreetMap (Python)"
description: "Get dentists, lawyers, restaurants or any business type in a city with name, website, phone and address from OpenStreetMap, free and legally reusable. Python code and the pitfalls that cause 403s and empty results."
permalink: /local-business-lead-list-openstreetmap/
---

# How to build a free local business lead list with OpenStreetMap

Google Maps is the obvious source for local business leads, but scraping it breaks Google's terms and gets expensive. **OpenStreetMap (OSM)** is a free, open alternative: millions of businesses with name, website, phone and address, licensed under the ODbL so you may reuse the data with attribution.

Two free APIs do the work: **Nominatim** turns "Austin, Texas" into an area, and **Overpass** returns every business with a given tag in it.

```python
import csv, httpx

UA = {"User-Agent": "my-lead-list/1.0 (+https://your-site.com)"}  # identify yourself; placeholder agents get a 403

def area_id(place):
    hits = httpx.get("https://nominatim.openstreetmap.org/search",
                     params={"q": place, "format": "json", "limit": 5}, headers=UA).json()
    rel = next(h for h in hits if h["osm_type"] == "relation")
    return 3_600_000_000 + int(rel["osm_id"])  # Overpass area id for an OSM relation

def businesses(tag, place, limit=200):
    key, value = tag.split("=")
    query = f'[out:json][timeout:90];area({area_id(place)})->.a;nwr["{key}"="{value}"](area.a);out tags {limit};'
    data = httpx.post("https://overpass-api.de/api/interpreter", data={"data": query}, headers=UA, timeout=120).json()
    for el in data["elements"]:
        t = el.get("tags", {})
        if t.get("name"):
            yield {"name": t["name"], "website": t.get("website") or t.get("contact:website", ""),
                   "phone": t.get("phone") or t.get("contact:phone", ""), "email": t.get("email") or t.get("contact:email", ""),
                   "city": t.get("addr:city", "")}

rows = list(businesses("amenity=dentist", "Austin, Texas"))
with open("leads.csv", "w", newline="") as f:
    w = csv.DictWriter(f, fieldnames=rows[0].keys()); w.writeheader(); w.writerows(rows)
print(len(rows), "businesses,", sum(1 for r in rows if r["website"]), "with a website")
```

Running it for dentists in Austin returned **122 businesses, 56 with a website** (November 2026).

## Common business tags

| Niche | OSM tag |
|---|---|
| Dentists | `amenity=dentist` |
| Doctors / clinics | `amenity=doctors`, `amenity=clinic` |
| Lawyers | `office=lawyer` |
| Accountants | `office=accountant` |
| Real estate agents | `office=estate_agent` |
| Restaurants / cafes | `amenity=restaurant`, `amenity=cafe` |
| Gyms | `leisure=fitness_centre` |
| Hair salons | `shop=hairdresser` |
| Car repair | `shop=car_repair` |
| Plumbers / electricians | `craft=plumber`, `craft=electrician` |

## Pitfalls

- **403 from Nominatim:** it blocks generic or placeholder user agents (including `you@example.com`). Send one that identifies your project, and keep to 1 request per second.
- **Pick a relation, not a point:** the area id only works for boundaries (`osm_type == "relation"`). Searching "Austin" alone can match the wrong place; add the state or country.
- **Coverage varies by niche.** Healthcare, legal, real estate and restaurants are well mapped (Texas has 700+ dentists with websites; Chicago has 1,500+ restaurants). Trades are thin (Colorado lists only about 30 plumbers with websites), so widen the area to a state or combine sources.
- **Use the public Overpass server politely:** cache results, avoid hammering it in loops, and keep the timeout realistic for large areas.
- **Credit the data:** "© OpenStreetMap contributors" is required by the ODbL when you share or publish the list.
- **Emails are rarely in OSM.** Most businesses publish their email on their own website instead, usually on the contact page.

## Adding emails and social profiles

The next step is visiting each business's website and pulling its public contact details. See [how to extract emails, phone numbers and social links from a website](extract-emails-from-website.md) for the code, or use **[Website Contact Extractor](https://apify.com/signalfeed/website-contact-extractor?fpr=kyk4x5)** to do it for the whole list at $2.50 per 1,000 websites. *(Built by the author of this guide.)*

Don't want to do it yourself? I build these lists to order, deduplicated and delivered as a spreadsheet: **[B2B lead list from company websites on Fiverr](https://www.fiverr.com/arabinowe/build-a-b2b-lead-list-with-emails-and-phones-from-company-websites)**.

[← Home](README.md)
