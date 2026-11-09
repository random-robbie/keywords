# keywords

A collection of wordlists for hunting exposed secrets, API keys and credentials in public sources such as GitHub code search, pastebins and other indexed code hosting sites.

The keyword lists were originally taken from [Shashank-In/TravisLeaks](https://github.com/Shashank-In/TravisLeaks), mixed with additional patterns gathered from real-world Kubernetes secrets (environment variable names, config keys and key-value patterns commonly seen in `Secret` and `ConfigMap` objects).

## Contents

| File | Entries | Description |
|------|---------|-------------|
| `keywords.txt` | ~1,600 | One keyword per line: credential-related variable names and config keys (`password=`, `--token=`, `api_key`, private key PEM headers such as `-----BEGIN RSA PRIVATE KEY-----`, and similar). Useful as grep/regex needles when scanning code or HTTP responses for secrets. |
| `tocheck.txt` | ~1,800 | Pre-built GitHub code search URLs of the form `https://github.com/search?q=AKIA...&type=Code`, mostly targeting AWS access key ID prefixes (`AKIA`). Useful as a checklist when manually reviewing leaked AWS keys. |

Both files are plain text, one entry per line, with no headers — they can be fed directly into tools that accept a wordlist.

## Usage

### grep a directory of cloned repos

```bash
while read -r kw; do
  grep -rniF -- "$kw" ./repos/ && echo ">> $kw"
done < keywords.txt
```

### Scan responses with ffuf-style request loops

Match keyword lines against HTTP responses (for example responses from exposed `.env`, config or debug endpoints):

```bash
while read -r kw; do
  curl -s "https://target.example/.env" | grep -qiF -- "$kw" && echo "HIT: $kw"
done < keywords.txt
```

### nuclei-style matchers

Turn lines from `keywords.txt` into nuclei word matchers:

```yaml
id: exposed-secrets-keywords

matchers-condition: or
matchers:
  - type: word
    part: body
    condition: or
    words:
      - "-----BEGIN RSA PRIVATE KEY-----"
      - "password="
      - "api_key"
```

### Work through the GitHub search checklist

Open the pre-built search URLs in `tocheck.txt` in a browser, or fetch them programmatically:

```bash
while read -r url; do
  echo "Checking: $url"
  # open in browser, or query the GitHub API with your own token
done < tocheck.txt
```

Note that GitHub code search requires authentication for API access and rate limits unauthenticated requests.

## Disclaimer / Authorised use

This repository is for authorised security testing and responsible disclosure only. Do not use these lists to access, exfiltrate or misuse credentials that do not belong to you. If you find exposed keys belonging to others, report them responsibly to the affected party or a coordinated disclosure programme.

## Author

Robbie Wiggins — [What-Security](https://www.what-security.co.uk) · [GitHub](https://github.com/random-robbie)
