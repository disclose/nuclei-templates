# disclose.io · Nuclei templates

Enrichment templates that turn a [Nuclei](https://github.com/projectdiscovery/nuclei) scan into a *disclosure workflow*: for every host you scan, resolve **who to report a vulnerability to** — via [lookup.disclose.io](https://lookup.disclose.io).

> These are **enrichment** templates. They report the responsible security-disclosure channel for a host (bug bounty, security.txt, VDP, PSIRT, national CERT). They do **not** detect a vulnerability on the target.

Part of [the disclose.io Project](https://disclose.io) — the open, vendor-neutral infrastructure for vulnerability disclosure.

---

## Templates

| Template | What it does |
|----------|--------------|
| [`http/osint/disclose-io-lookup.yaml`](http/osint/disclose-io-lookup.yaml) | For each scanned host, POSTs it to `lookup.disclose.io/api/lookup` and surfaces the resolved organization + disclosure contacts as an `info` finding. |

## Use it

Clone and point Nuclei at the directory:

```bash
git clone https://github.com/disclose/nuclei-templates disclose-nuclei-templates
nuclei -u example.com -t disclose-nuclei-templates/ -duc
```

Or wire it as a remote template source so `-update-templates` pulls it:

```bash
export GITHUB_TEMPLATE_REPO=disclose/nuclei-templates
nuclei -update-templates
nuclei -u example.com -t github/nuclei-templates/ -duc
```

For a larger scan, raise the rate limit with a free API key (request one at
[hello@disclose.io](mailto:hello@disclose.io)) and keep Nuclei's own rate limiter low —
under `@Host` every scanned target egresses to lookup's single IP:

```bash
nuclei -l hosts.txt -t disclose-nuclei-templates/ -var token=YOUR_KEY -rl 20 -duc
```

## Notes & caveats

- **Unsigned template → you'll see a warning.** These templates aren't signed with ProjectDiscovery's key, so Nuclei prints an "unsigned templates" notice and asks you to allow them; `-duc` (disable update check) keeps runs quiet. `http` templates still execute normally — only `code`-protocol templates are ever *blocked* when unsigned.
- **The pipe is often the better tool.** If you're already producing scan output, [`dio-lookup`](https://github.com/disclose/dio-lookup) enriches it directly and **de-duplicates hosts first** (kinder to the rate limit): `nuclei -u example.com -jsonl | dio-lookup --nuclei`. Reach for the template when you want the disclosure contact **inline in your Nuclei findings**; reach for the pipe for large scans and automation.
- **Data egress.** Each scanned host is sent to lookup.disclose.io, which logs requests. For target lists under NDA, be aware of that before running across a whole scope.
- **Not affiliated with `projectdiscovery/nuclei-templates`.** This is a disclose.io-maintained set; consume it directly from here.

## License

MIT — see [LICENSE](LICENSE). A [disclose.io](https://disclose.io) project.
