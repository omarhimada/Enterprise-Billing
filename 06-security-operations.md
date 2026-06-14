# Security, Operations, and Reliability

## Security Controls

| Area | Control |
|---|---|
| Authentication | OAuth2/OIDC for users, mTLS or signed tokens for services |
| Authorization | Tenant-aware RBAC and least-privilege service roles |
| Data Protection | Encryption at rest and in transit |
| Payment Data | Prefer tokenized payment methods; avoid storing PAN data |
| Audit | Immutable append-only audit events |
| Secrets | Centralized secret manager and rotation |
| Privacy | Data retention, deletion workflows, and access logging |

## Reliability Requirements

| Capability | Requirement |
|---|---|
| Invoice generation | Durable, resumable, idempotent jobs |
| Payment collection | Retryable with processor-safe idempotency keys |
| Usage ingestion | Exactly-once effect through deduplication |
| Ledger posting | Immutable, balanced debit/credit entries |
| Webhooks | At-least-once delivery with signed payloads |
| Reconciliation | Daily comparison between billing, payments, and ERP |

## Observability

Track these metrics:

- Billing run duration
- Invoice finalization failures
- Payment authorization and capture success rates
- Usage ingestion lag
- Rating error count
- Tax provider latency and failure rate
- Ledger posting imbalance count
- Dunning recovery rate

## C# Pseudocode — Idempotent Command Handler

```csharp
public sealed class IdempotentCommandHandler<TCommand, TResult>
{
    private readonly IIdempotencyStore _store;
    private readonly ICommandHandler<TCommand, TResult> _inner;

    public async Task<TResult> HandleAsync(
        TCommand command,
        string idempotencyKey,
        CancellationToken ct)
    {
        var requestHash = HashRequest(command);
        var existing = await _store.GetAsync(idempotencyKey, ct);

        if (existing is not null)
        {
            if (existing.RequestHash != requestHash)
            {
                throw new InvalidOperationException(
                    "Idempotency key was reused with a different request.");
            }

            return existing.GetResult<TResult>();
        }

        var result = await _inner.HandleAsync(command, ct);

        await _store.SaveAsync(new IdempotencyRecord
        {
            Key = idempotencyKey,
            RequestHash = requestHash,
            SerializedResult = Serialize(result),
            CreatedAt = DateTimeOffset.UtcNow
        }, ct);

        return result;
    }

    private static string HashRequest(TCommand command)
    {
        var json = Serialize(command);
        return Convert.ToHexString(System.Security.Cryptography.SHA256.HashData(
            System.Text.Encoding.UTF8.GetBytes(json)));
    }

    private static string Serialize<T>(T value)
    {
        return System.Text.Json.JsonSerializer.Serialize(value);
    }
}
```

## Disaster Recovery

- Define RPO/RTO by bounded context.
- Backup operational databases continuously.
- Store invoice PDFs and accounting exports in versioned object storage.
- Rebuild projections from the event log.
- Reconcile restored balances against immutable ledger entries.

## Compliance Notes

- Use immutable invoice numbering after finalization.
- Preserve historical price, tax, and contract terms used for each invoice.
- Separate invoice correction from invoice mutation; use credit memos or void/reissue workflows.
- Maintain audit logs for financial state transitions.
