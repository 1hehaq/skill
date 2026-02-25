# Bug Bounty Recon Methodology

## Subdomain Enumeration

### Passive
```bash
subfinder -d target.com -silent -all -recursive -o subs.txt
```
```bash
assetfinder -subs-only target.com | tee -a subs.txt
```
```bash
findomain -t target.com -o findomain.txt
```
```bash
github-subdomains -d target.com -t GITHUB_TOKEN -o github.txt
```

### Active
```bash
shuffledns -d target.com -list passive_subs.txt -r resolvers.txt -o active_subs.txt
```
```bash
cat passive_subs.txt active_subs.txt | sort -u | anew all_subs.txt
```
```bash
dnsx -l all_subs.txt -resp -o resolved_subs.txt
```

---

## HTTP Probing
```bash
httpx -l resolved_subs.txt -p 80,443,8080,8443 -silent -title -sc -ip -o live_websites.txt
```
```bash
cat live_websites.txt | grep -E "login|admin|dashboard|api" | tee interesting.txt
```

---

## JavaScript Analysis
```bash
gau target.com | grep "\.js$" | sort -u | httpx -mc 200 | tee js_files.txt
```
```bash
waybackurls target.com | grep "\.js$" | sort -u | httpx -mc 200 | tee -a js_files.txt
```
```bash
linkfinder -i js_files.txt -o js_endpoints.txt
```
```bash
cat js_files.txt | gf aws-keys | tee aws_keys.txt
```
```bash
cat js_files.txt | gf tokens | tee tokens.txt
```
```bash
cat js_files.txt | gf urls | tee sensitive_urls.txt
```
```bash
echo "https://target.com/app.js" | sed 's/\.js$/.map/' | httpx -mc 200
```

---

## URL Discovery & Crawling
```bash
katana -list live_websites.txt -jc -o katana_urls.txt
```
```bash
gospider -s "https://target.com" -d 3 -o gospider_output/
```
```bash
cat gospider_output/*.txt | sort -u | tee all_urls.txt
```

---

## Archive Enumeration
```bash
gau --subs target.com | tee gau_urls.txt
```
```bash
waybackurls target.com | tee wayback_urls.txt
```
```bash
curl 'https://web.archive.org/cdx/search/cdx?url=*.target.com/*&collapse=urlkey&output=text&fl=original&filter=original:.*\.(xls|xml|xlsx|json|pdf|sql|doc|docx|pptx|txt|zip|tar|gz|tgz|bak|7z|rar|log|cache|secret|db|backup|yml|config|csv|yaml|md|md5|exe|dll|bin|ini|bat|sh|deb|rpm|iso|img|apk|msi|dmg|tmp|crt|pem|key|pub|asc)$'
```

---

## Parameter Discovery
```bash
arjun -u "https://target.com/page" -m GET -o params.json
```
```bash
paramspider.py --domain target.com --exclude woff,css,js,png,ico,jpg --output params.txt
```
```bash
ffuf -u "https://target.com/page?FUZZ=value" -w /usr/share/wordlists/seclists/Discovery/Web-Content/burp-parameter-names.txt -o ffuf_params.txt
```

---

## Content Discovery
```bash
ffuf -u "https://target.com/FUZZ" -w /usr/share/wordlists/raft-large-words.txt -mc 200,204,301,302,307,401,403 -t 50 -o ffuf_content.txt
```

---

## Cloud Asset Enumeration
```bash
cloud_enum -k target.com -o cloud_results.txt
```
```bash
amass intel -asn ASN_NUMBER -o asn_ips.txt
```

---

## Subdomain Takeover

### Automation
```bash
subjack -w resolved_subs.txt -ssl -a -m takeover_results.json
```
```bash
tnscored -w resolved_subs.txt -o takeover.txt
```

### Manual
```bash
dnsx -l resolved_subs.txt -cname -resp-only -o cnames.txt
```

**Services:** GitHub Pages, Heroku, AWS S3, Azure, Cloudflare, Vercel, Netlify, Shopify, Fastly, Tumblr, WordPress, SendGrid, Stripe

**Indicators:** NXDOMAIN, 404, "NoSuchBucket", "Not Found", abandoned service error
