# YouTube License Checker

Check the Creative Commons license-label signal for a YouTube video and export the API result as JSON and JSONL.

Powered by the [FlashData YouTube License Metadata API](https://flashdata.dev/docs/sources/youtube_trainability). permitted is a license-label signal, not legal training permission or creator third-party training settings.

## Quick start

Add a [FlashData API key](https://flashdata.dev/signup) as the repository secret `FLASHDATA_API_KEY`. API calls require FlashData Credits ([pricing](https://flashdata.dev/pricing)). Then add this workflow:

```yaml
name: YouTube License Checker
on:
  workflow_dispatch: {}
permissions:
  contents: read
jobs:
  collect:
    runs-on: ubuntu-latest
    steps:
      - name: YouTube License Checker
        id: data
        uses: rocosen/flashdata-youtube-trainability-action@v0.1.1
        with:
          api-key: ${{ secrets.FLASHDATA_API_KEY }}
          video-id: dQw4w9WgXcQ
          options-json: "{}"
      - name: Upload results
        uses: actions/upload-artifact@v4
        with:
          name: youtube-trainability
          path: ${{ steps.data.outputs.artifact-directory }}
          retention-days: 7
```

Run it from the **Actions** tab. Download the exported files from the run’s **Artifacts**, or use the output paths in later steps. Requires an up-to-date runner with Node 24 support; no dependency installation is needed. For production, pin actions to a reviewed commit SHA.

## Inputs

| Input | Required | Description |
| --- | --- | --- |
| `api-key` | Yes | FlashData API Key. Pass secrets.FLASHDATA_API_KEY; requests use paid Credits. |
| `video-id` | Yes | 11-character YouTube video ID. |
| `options-json` | No | Optional JSON object with only source-specific options listed in the README. Never include credentials here. |

### Options

Pass these fields in `options-json`:

No additional options. The API source name is youtube_trainability, but the returned label does not establish permission to train models.

Invalid options fail before an API request. One paid query per run; no automatic pagination or retries.

## Outputs

- `response.json`: API response, including available result fields and usage.
- `records.jsonl`: result objects, one per line.
- `manifest.json`: source, operation, timestamps, record count and export warnings, plus query/options and reported usage.

Returned API fields: results[]: videoId and license-label indicator permitted, with available kind/etag; usage.credits.

Empty results produce an empty JSONL file and `result-count: 0`. Optional fields may be absent.

| Step output | Description |
| --- | --- |
| `artifact-directory` | Absolute directory with response.json, records.jsonl and manifest.json. |
| `results-file` | Absolute path to the original response JSON (sanitized lifecycle fields for download jobs). |
| `records-file` | Absolute path to JSONL records for downstream steps. Empty file for an empty result. |
| `result-count` | Number of exported records; not the requested result count. |
| `source` | The fixed FlashData source used by this action. |

## Pricing and errors

The action is free to install. FlashData API calls use paid Credits; GitHub runner/artifact costs are separate. [Current pricing](https://flashdata.dev/pricing).

401: check the API key. 402: add Credits. 429: rate/quota limit. After a timeout, check [FlashData Jobs](https://flashdata.dev) before rerunning: a request may already have been accepted. Paid POST requests are not retried. Requests have a 65-second timeout and a 16-MiB response limit.

## Development

`npm ci` → `npm run check` → `npm test` → `npm run build`. Tests use offline fixtures. Commit the bundle with runtime changes.
