---
title: "How to decode Cloudflare email protection ([email protected]) in Python and JavaScript"
description: "Why scraped pages show [email protected] instead of the real address, and a few lines of Python or JavaScript that decode data-cfemail and /cdn-cgi/l/email-protection links back into the email."
permalink: /decode-cloudflare-email-protection/
---

# How to decode Cloudflare email protection (`[email protected]`)

You fetch a page with `requests` or `curl`, look for the contact email, and find this instead:

```html
<a href="/cdn-cgi/l/email-protection#1c75727a735c7d7f7179327573">
  <span class="__cf_email__" data-cfemail="1c75727a735c7d7f7179327573">[email&#160;protected]</span>
</a>
```

That is Cloudflare's Email Address Obfuscation. The site's real address is still in the page, lightly encoded, and a small script decodes it in every visitor's browser. Your HTTP client doesn't run that script, so you see the placeholder.

## How the encoding works

The hex string is a list of bytes. **The first byte is a key; every following byte is one character of the email XOR-ed with that key.** There is no secret involved: the key ships with the data, because the page has to be readable by any browser.

## Python

```python
import re

def decode_cfemail(hexed: str) -> str:
    raw = bytes.fromhex(hexed)
    return bytes(b ^ raw[0] for b in raw[1:]).decode("utf-8", "ignore")

CFEMAIL = re.compile(r"""(?:data-cfemail=["']|/cdn-cgi/l/email-protection#)([0-9a-f]{6,400})""", re.I)

def cloudflare_emails(html: str) -> set[str]:
    return {decode_cfemail(h) for h in CFEMAIL.findall(html)}

print(decode_cfemail("1c75727a735c7d7f7179327573"))   # info@acme.io
```

The regular expression catches both places the value appears: the `data-cfemail` attribute and the `/cdn-cgi/l/email-protection#…` link.

## JavaScript

```javascript
function decodeCfEmail(hex) {
  const key = parseInt(hex.slice(0, 2), 16);
  let out = "";
  for (let i = 2; i < hex.length; i += 2) {
    out += String.fromCharCode(parseInt(hex.slice(i, i + 2), 16) ^ key);
  }
  return out;
}
```

## Other places emails hide

Decoding Cloudflare is one of four things a contact scraper needs to do to match what a person sees:

1. **`mailto:` links**, with HTML entities and URL-encoding undone (`info&#64;acme.io`, `info%40acme.io`).
2. **Cloudflare-protected addresses**, as above.
3. **schema.org markup**: `"email": "info@acme.io"` inside `<script type="application/ld+json">`. Most scrapers strip scripts and miss it.
4. **Written-out obfuscation**: `info [at] acme [dot] io`.

Then filter the false positives: image names like `logo@2x.png`, placeholders like `name@example.com`, and error-tracking addresses.

On a test of 45 small-business websites, adding steps 2 and 3 to a plain-text email scraper found an address on three sites that otherwise showed none.

## Use it responsibly

This reads an address the site publishes to every visitor; it doesn't bypass a login or a CAPTCHA. The owner still chose to make the address harder to harvest, so treat it accordingly: business role inboxes only, relevant one-to-one messages, and follow the email rules that apply to you (CAN-SPAM, GDPR, CASL).

## Skip the build

- **[Website Email Scraper](https://apify.com/signalfeed/website-contact-extractor?fpr=kyk4x5)** does all four steps for any list of websites and returns one row per site with emails, phones and social profiles. $2.50 per 1,000 sites; sites that fail to load are free. *(Built by the author of this guide.)*
- Want a finished list instead? I build [B2B lead lists from company websites](https://www.fiverr.com/arabinowe/build-a-b2b-lead-list-with-emails-and-phones-from-company-websites) to order.

Related: [extract emails, phones and social links from a website](extract-emails-from-website.md)

[← Home](README.md)
