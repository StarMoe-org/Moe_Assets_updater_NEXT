# HIP completion and recovery

In HIP poller mode, a local download/export record is not evidence that the
gateway committed a bundle. The server-authoritative filter sees all candidates
selected by the configured region filters, even if the legacy JSON contains the
same fingerprint. The HIP executor does not persist newly exported bundles into
that record. Non-HIP executions retain their existing record behavior.

A region advances its watermark only when:

- no download/export failed and every queued task finished;
- every selected HIP candidate is accounted for by an acknowledged upload batch
  or by HIP CHECK skip followed by the required metadata commit;
- the uploader completed without an error or an unknown commit result.

Empty queues alone do not establish completion. Partial acknowledged progress is
kept, but an incomplete region leaves its version watermark unchanged for retry.

## Existing installations

Watermarks now include `hip_verified`. Older files deserialize this field as
`false`; HIP mode does not use them to skip the same version. The next poll
reconciles candidates against HIP, ignoring legacy download records, and writes a
verified watermark only after all completion checks pass. No bulk deletion of
local records, exported files, or gateway data is needed for this migration.

These checks repair completion bookkeeping. They do not shorten gateway COMMIT
transactions, increase the HIP timeout, serialize SQLite writers, or guarantee a
successful repair while the gateway continues timing out. Those performance and
protocol changes require separate fixes. Deployment must still be followed by
candidate/fingerprint reconciliation and file-read checks on the affected regions.
