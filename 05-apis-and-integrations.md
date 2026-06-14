# APIs and Integrations

## External API Design

APIs should be versioned, idempotent, and scoped by tenant or organization.

### Example Resources

| Resource | Example Endpoint |
|---|---|
| Customers | `POST /v1/customers` |
| Accounts | `POST /v1/accounts` |
| Subscriptions | `POST /v1/subscriptions` |
| Usage Records | `POST /v1/usage-records` |
| Invoices | `GET /v1/invoices/{invoiceId}` |
| Payments | `POST /v1/payments` |
| Credit Memos | `POST /v1/credit-memos` |

## C# Pseudocode — Usage Ingestion

```csharp
[ApiController]
[Route("v1/usage-records")]
public sealed class UsageRecordsController : ControllerBase
{
    private readonly IUsageIngestionService _usage;

    [HttpPost]
    public async Task<IActionResult> CreateAsync(
        [FromHeader(Name = "Idempotency-Key")] string idempotencyKey,
        [FromBody] CreateUsageRecordRequest request,
        CancellationToken ct)
    {
        if (string.IsNullOrWhiteSpace(idempotencyKey))
        {
            return BadRequest("Idempotency-Key header is required.");
        }

        var result = await _usage.RecordAsync(request, idempotencyKey, ct);

        return result.WasAlreadyProcessed
            ? Ok(result.UsageRecord)
            : CreatedAtAction(nameof(GetAsync), new { id = result.UsageRecord.Id }, result.UsageRecord);
    }

    [HttpGet("{id}")]
    public async Task<IActionResult> GetAsync(Guid id, CancellationToken ct)
    {
        var usageRecord = await _usage.GetAsync(id, ct);
        return usageRecord is null ? NotFound() : Ok(usageRecord);
    }
}
```

## Integration Patterns

| Integration | Pattern |
|---|---|
| CRM / CPQ | Contract and customer sync via events and APIs |
| ERP / GL | Batch export with reconciliation reports |
| Payment Processor | Synchronous authorization, asynchronous webhook confirmation |
| Tax Provider | Real-time tax calculation during invoice finalization |
| Product Apps | High-throughput usage ingestion |
| Data Warehouse | Event streaming and nightly dimensional models |

## Webhook Events

```json
{
  "eventId": "evt_123",
  "eventType": "invoice.finalized",
  "occurredAt": "2026-06-13T12:00:00Z",
  "tenantId": "tenant_001",
  "data": {
    "invoiceId": "inv_123",
    "accountId": "acct_123",
    "amountDue": 12500,
    "currency": "USD"
  }
}
```

## Idempotency Requirements

- All externally callable write APIs must accept an idempotency key.
- Idempotency records should include request hash, response payload, status, and expiration.
- Reusing a key with a different request hash should return a conflict.
