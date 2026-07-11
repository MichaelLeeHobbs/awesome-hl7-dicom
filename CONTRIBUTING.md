# Contributing

Thanks for wanting to add to the list. This is a *curated* list, not a directory — the bar is
"a working integration engineer would be glad someone showed them this."

## What gets accepted

- **It's genuinely useful to someone building HL7 or DICOM integrations.** Libraries, engines,
  plugins, viewers, validators, specs, and the rare article that actually teaches something.
- **It's maintained, or it's the best thing that exists anyway.** Dormant-but-canonical is fine
  (say so). Dead-and-superseded belongs in the "Dead ends" section, not the main list.
- **The link works and the description is honest.** One line, no marketing copy.

## What gets rejected

- **Self-promotion from a brand-new account with no track record.** Not a hard no — but if you're
  submitting your own product, say so in the PR, and expect it to be checked.
- **"Open source" claims with no source.** If you say a tool is open source, the repo must actually
  contain the code.
- **Tools that handle PHI without a verifiable privacy story.** A de-identifier or message viewer
  that processes real patient data is a *trust* dependency. If it claims client-side-only
  processing, that claim needs to be verifiable (published source, or at minimum a clear
  explanation). Healthcare is not the place for "just trust me."
- **SEO bait, link farms, and paywalled content dressed up as free.**

## Format

One bullet, alphabetical-ish within its section, sorted by usefulness where that's clearer:

```markdown
- [Name](https://example.com) - What it is, in one line, ending without a period.
```

Mark commercial products `(commercial)` and unmaintained-but-notable ones `(unmaintained)`. Be
straight about it — readers can handle nuance, and hiding it wastes their afternoon.

## How

1. Fork, branch, edit `README.md`.
2. Open a PR explaining *why* it belongs and *what your relationship to it is*.
3. Expect questions. They're not personal.
