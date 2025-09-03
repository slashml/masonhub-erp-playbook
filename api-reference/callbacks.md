# Callbacks

The Callbacks API manages webhook subscriptions for real-time event notifications from MasonHub. Register callback URLs to receive immediate notifications when events occur in the system.

## Endpoints

### Get Callback URLs

**GET** `/callbacks`

Retrieve registered callback URLs filtered by message type.

#### Query Parameters

| Parameter | Type | Description | Default |
|-----------|------|-------------|---------|
| `message_type` | string | Filter by message type or "all" | "all" |

#### Available Message Types

- **skuInventoryChange**: Inventory level changes
- **orderEvent**: Order status updates  
- **orderUpdateResolution**: Order update request results
- **orderCancelResolution**: Order cancel request results
- **inboundShipmentEvent**: Inbound shipment status changes
- **rmaEvent**: Return status updates
- **snapshotReady**: Inventory snapshot completion

#### Example Request

```http
GET /callbacks?message_type=skuInventoryChange
```

#### Example Response

```json
[
  {
    "id": "callback-uuid-123",
    "url": "https://api.clienturl.com/api/inventory_event",
    "message_type": "skuInventoryChange",
    "api_version": "1.0",
    "token": "encrypted_verification_token"
  }
]
```

### Create Callback URLs

**POST** `/callbacks`

Register new callback URLs for event notifications.

#### Request Body Schema

Array of callback objects:

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `url` | string | Yes | HTTPS endpoint URL |
| `message_type` | string | Yes | Event type to subscribe to |
| `api_version` | string | No | API version (default: "1.0") |
| `token` | string | No | Optional verification token |

#### Example Request

```json
[
  {
    "url": "https://api.clienturl.com/api/skuInventoryChange",
    "message_type": "skuInventoryChange",
    "api_version": "1.0",
    "token": "your_secure_verification_token"
  },
  {
    "url": "https://api.clienturl.com/api/orderEvent", 
    "message_type": "orderEvent",
    "api_version": "1.0"
  }
]
```

### Delete Callback URLs

**DELETE** `/callbacks`

Remove callback subscriptions by ID.

#### Request Body Schema

Array of callback IDs to delete:

```json
[
  "callback-uuid-123",
  "callback-uuid-456"
]
```

## Callback Security

### HTTPS Required

All callback URLs must use HTTPS for security. HTTP URLs will be rejected.

### Verification Tokens

Include optional tokens in callback registrations for verification:

```json
{
  "url": "https://api.clienturl.com/webhook",
  "message_type": "orderEvent",
  "token": "your_encrypted_verification_token"
}
```

The token will be included in callback requests for verification.

### Authentication

Callback requests from MasonHub include:
- User-Agent identification
- Optional verification token (if configured)
- Consistent payload format

## Callback Management

### Callback Holds

Temporarily suspend callback delivery using the Callback Holds API:

#### Get Callback Holds

**GET** `/callback_holds`

List current callback holds by message type.

#### Create Callback Holds  

**POST** `/callback_holds`

Place holds on specific message types:

```json
[
  "skuInventoryChange",
  "orderEvent"
]
```

#### Delete Callback Holds

**DELETE** `/callback_holds`

Remove holds to resume callback delivery:

```json
[
  "skuInventoryChange", 
  "orderEvent"
]
```

## Event Delivery

### Delivery Guarantees

- **At-least-once delivery**: Events may be delivered multiple times
- **Retry logic**: Failed deliveries are retried with exponential backoff
- **Timeout handling**: Requests timeout after 30 seconds
- **Error logging**: Delivery failures are logged for troubleshooting

### Payload Format

All callbacks use consistent payload structure:

```json
{
  "callback_url": "https://client.com/api/webhook",
  "message_type": "eventType",
  "message_id": "unique-event-id",
  "api_version": "1.0",
  "data": [
    {
      // Event-specific data
    }
  ]
}
```

### Response Requirements

Your webhook endpoints should:
- Return HTTP 200 status for successful processing
- Respond within 30 seconds
- Handle duplicate events gracefully
- Validate message authenticity using tokens

## Testing and Development  

### Sandbox Environment

Use sandbox environment for testing:
- `https://sandbox.masonhub.co/{account}/api/v1/callbacks`
- All callback types available
- Use DataFactory to trigger test events

### Development Best Practices

1. **Idempotency**: Handle duplicate event delivery
2. **Authentication**: Verify callback authenticity
3. **Error Handling**: Return appropriate HTTP status codes
4. **Logging**: Log all callback events for debugging
5. **Monitoring**: Monitor webhook endpoint availability

### Example Webhook Implementation

```python
@app.route('/webhook/masonhub', methods=['POST'])
def handle_masonhub_webhook():
    try:
        payload = request.get_json()
        
        # Verify token if configured
        if expected_token and payload.get('token') != expected_token:
            return 'Unauthorized', 401
            
        message_type = payload['message_type']
        message_id = payload['message_id']
        data = payload['data']
        
        # Process based on message type
        if message_type == 'skuInventoryChange':
            handle_inventory_change(data)
        elif message_type == 'orderEvent':
            handle_order_event(data)
            
        return 'OK', 200
        
    except Exception as e:
        logger.error(f"Webhook error: {e}")
        return 'Error', 500
```

## Troubleshooting

### Common Issues

1. **HTTPS Required**: Ensure callback URLs use HTTPS
2. **Timeouts**: Respond within 30 seconds
3. **Status Codes**: Return 200 for successful processing
4. **Duplicates**: Handle duplicate event delivery
5. **Token Verification**: Verify tokens when configured

### Monitoring Delivery

Track callback delivery through:
- HTTP response codes from your endpoints
- MasonHub delivery logs (available on request)
- Callback hold status
- Event processing latency

### Support

For callback-related issues:
- Check callback registration status
- Verify endpoint accessibility
- Review delivery logs
- Contact: support@masonhub.co