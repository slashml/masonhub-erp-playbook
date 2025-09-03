# Orders

The Orders API manages the complete order lifecycle from creation to fulfillment. It supports various order types, complex routing policies, and asynchronous update operations.

## Endpoints

### Get Orders

**GET** `/orders`

Retrieve orders with filtering and pagination support.

#### Query Parameters

| Parameter | Type | Description | Default |
|-----------|------|-------------|---------|
| `id` | array of UUIDs | MasonHub UUIDs [1..30] | - |
| `offset` | integer | Pagination offset | 0 |
| `limit` | integer | Number of results [1..100] | 30 |
| `list_type` | string | `detail` or `summary` | `detail` |
| `sdt` | string | Start date for updated_at filter | - |

#### Example Request

```http
GET /orders?limit=50&list_type=detail&sdt=2018-08-01T00:00:00Z
```

### Create Orders

**POST** `/orders`

Create one or more orders with comprehensive configuration options.

#### Request Body Schema

Array of order objects with the following fields:

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `customer_identifier` | string | Yes | Unique order identifier |
| `order_type` | string | Yes | Order type (e.g., "customer") |
| `priority` | integer | No | Order priority (higher = more urgent) |
| `shipping_provider` | string | Yes | Shipping provider name |
| `shipper_service_level` | string | Yes | Service level (ground, express, etc.) |
| `value_added_services` | array | No | Additional services |
| `special_instructions` | string | No | Handling instructions |
| `gift_message` | string | No | Gift message for customer |
| `shipping_address_*` | string | Yes | Shipping address fields |
| `billing_address_*` | string | Yes | Billing address fields |
| `submitted_at` | string | Yes | Order submission time (RFC3339) |
| `line_items` | array | Yes | Order line items |
| `order_localization` | object | No | Localization settings |
| `backorder_policy` | string | No | Backorder handling |
| `routing_policy` | string | No | Fulfillment routing |
| `split_policy` | string | No | Order splitting rules |

#### Address Fields

Both shipping and billing addresses require:

```json
{
  "shipping_address_name": "John Jacob JingleHeimer-Schmidt III",
  "shipping_address_street_line_one": "234 House Lane",
  "shipping_address_street_line_two": "Apt 16",
  "shipping_address_city": "Little Falls", 
  "shipping_address_locale": "NJ",
  "shipping_address_postal_code": "07972",
  "shipping_address_country_code": "US",
  "shipping_address_phone_number": "973-999-3333",
  "shipping_address_type": "residential"
}
```

#### Line Items Array

```json
[
  {
    "sku_customer_id": "shirts872340",
    "quantity": 2,
    "promised_delivery_date": "2018-08-15T17:32:28Z",
    "estimated_delivery_date": "2018-08-15T17:32:28Z",
    "pick_from": [
      {
        "match_type": "purchase_order",
        "match_value": ["PO23423", "PO42322"],
        "match_style": "hard"
      }
    ]
  }
]
```

#### Example Request

```json
[
  {
    "customer_identifier": "129374",
    "order_type": "customer",
    "priority": 100,
    "shipping_provider": "masonhub", 
    "shipper_service_level": "ground",
    "value_added_services": ["Complimentary Handkerchief"],
    "special_instructions": "Triple Fold the Sleeves and wrap in Tissue Paper.",
    "gift_message": "Happy Birthday Freddie!",
    "shipping_address_name": "John Jacob JingleHeimer-Schmidt III",
    "shipping_address_street_line_one": "234 House Lane",
    "shipping_address_city": "Little Falls",
    "shipping_address_locale": "NJ", 
    "shipping_address_postal_code": "07972",
    "shipping_address_country_code": "US",
    "shipping_address_phone_number": "973-999-3333",
    "shipping_address_type": "residential",
    "billing_address_name": "John Jacob JingleHeimer-Schmidt III",
    "billing_address_street_line_one": "234 House Lane",
    "billing_address_city": "Little Falls",
    "billing_address_locale": "NJ",
    "billing_address_postal_code": "07972", 
    "billing_address_country_code": "US",
    "billing_address_phone_number": "973-999-3333",
    "billing_address_type": "residential",
    "submitted_at": "2018-08-01T00:00:00Z",
    "line_items": [
      {
        "sku_customer_id": "shirts872340",
        "quantity": 2,
        "promised_delivery_date": "2018-08-15T17:32:28Z",
        "estimated_delivery_date": "2018-08-15T17:32:28Z"
      }
    ]
  }
]
```

### Update Orders

**POST** `/order_update_requests`

Submit asynchronous order update requests. Updates require full order objects and are processed asynchronously.

#### Request Body Schema

Same schema as Create Orders. The system performs full replacement of order data.

#### Response

```json
{
  "records_submitted": 1,
  "records_processed": 1,
  "records_failed": 0,
  "records_succeeded": 1,
  "results": [
    {
      "system_id": "0b744e29-b668-4486-85dd-82528b5da0dd",
      "customer_identifier": "12345",
      "status": "success"
    }
  ]
}
```

### Get Order Updates

**GET** `/order_update_requests`

Retrieve order update request status.

#### Query Parameters

| Parameter | Type | Description | Default |
|-----------|------|-------------|---------|
| `id` | array of UUIDs | Update request UUIDs [1..30] | - |
| `offset` | integer | Pagination offset | 0 |
| `limit` | integer | Number of results [1..100] | 30 |

### Cancel Orders

**POST** `/order_cancel_requests`

Submit asynchronous order cancellation requests.

#### Request Body Schema

```json
[
  {
    "customer_identifier": "129374"
  },
  {
    "order_cancel_request_id": "uuid-here"
  }
]
```

### Get Order Cancels

**GET** `/order_cancel_requests`

Retrieve order cancellation request status.

#### Query Parameters

Same as Get Order Updates.

## Order Statuses

### Primary Statuses

- **open**: Order created, awaiting fulfillment
- **atWarehouse**: Order received at distribution center
- **inProcess**: Order being picked and packed
- **packed**: Order packed, ready to ship
- **fulfilled**: Order shipped to customer
- **delivered**: Order delivered to customer
- **canceled**: Order canceled

### Status Transitions

Orders progress through statuses automatically as they move through the fulfillment process.

## Advanced Features

### Pick From Constraints

Control inventory allocation with pick_from rules:

```json
{
  "pick_from": [
    {
      "match_type": "purchase_order",
      "match_value": ["PO23423", "PO42322"], 
      "match_style": "hard"
    },
    {
      "match_type": "lot_number",
      "match_value": ["LOT001"],
      "match_style": "soft"
    }
  ]
}
```

### Order Splitting

Configure how orders can be split across shipments:

- **single_shipment**: Never split orders
- **allow_splits**: Allow splits for efficiency
- **minimize_splits**: Prefer single shipments but allow splits

### Backorder Policies

Handle out-of-stock scenarios:

- **cancel_shorts**: Cancel unavailable items
- **backorder**: Hold order until inventory available
- **partial_ship**: Ship available items immediately

## Order Events and Callbacks

The system generates **orderEvent** callbacks for status changes:

```json
{
  "callback_url": "https://client.com/api/orderEvent",
  "message_type": "orderEvent", 
  "message_id": "0896116f-e54b-4756-9d3e-1b0c4a25d821",
  "data": [
    {
      "order_id": "8f1316d5-859f-4802-98b0-63d03a9517ac",
      "customer_identifier": "12345",
      "status": "fulfilled",
      "shipments": [
        {
          "shipment_id": "88888",
          "shipping_provider": "UPS",
          "shipper_service_level": "ground",
          "tracking_number": "1z324897234nferg45",
          "tracking_url": "http://ups.com/tracking/1z324897234nferg45",
          "shipment_date_time": "2018-08-06T15:11:19Z",
          "shipment_line_items": [
            {
              "sku_customer_id": "shirts872340",
              "quantity": 2
            }
          ]
        }
      ],
      "shorts": [
        {
          "sku_customer_id": "pants3422",
          "quantity": 1,
          "reason": "damage"
        }
      ]
    }
  ]
}
```

## Update and Cancel Resolution

Asynchronous operations generate resolution callbacks:

### Order Update Resolution

**orderUpdateResolution** callbacks indicate success/failure of update requests.

### Order Cancel Resolution

**orderCancelResolution** callbacks indicate success/failure of cancel requests.

## Best Practices

1. **Complete Address**: Always provide complete shipping and billing addresses with country codes
2. **Asynchronous Updates**: Monitor callback events for update/cancel confirmations
3. **SKU Validation**: Ensure all line item SKUs exist before creating orders
4. **Status Tracking**: Use orderEvent callbacks for real-time status updates
5. **Batch Operations**: Process multiple orders in single requests when possible