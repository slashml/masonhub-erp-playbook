# Catalog Management

The Catalog API manages SKUs (Stock Keeping Units), which are the primary catalog items in the MasonHub system. SKUs are never fully removed from the system if inventory exists, ensuring data integrity and historical tracking.

## Endpoints

### Get SKUs on the Item Master

**GET** `/skus`

Retrieve SKUs with flexible search parameters and batch support.

#### Query Parameters

| Parameter | Type | Description | Default |
|-----------|------|-------------|---------|
| `id` | array of UUIDs | MasonHub UUID strings [1..30] | - |
| `cid` | array of strings | Customer identifiers [1..30] | - |
| `list_type` | string | Response detail level: `detail` or `summary` | `detail` |
| `limit` | integer | Number of results [1..100] | 30 |
| `offset` | integer | Pagination offset | 0 |
| `include_counts` | boolean | Include inventory counts (detail only) | false |

#### Example Request

```http
GET /skus?cid=pants4523&list_type=detail&include_counts=true
```

#### Example Response

```json
{
  "include_counts": true,
  "limit": 10,
  "list_type": "detail", 
  "offset": 0,
  "data": [...]
}
```

### Add SKUs to the Item Master

**POST** `/skus`

Create one or more SKUs in the catalog system.

#### Request Body Schema

Array of SKU objects with the following fields:

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `customer_identifier` | string | Yes | Unique customer SKU identifier |
| `customer_bar_code` | string | Yes | Customer barcode |
| `vendor_bar_code` | string | No | Vendor barcode |
| `bar_codes` | array | No | Additional barcodes for the SKU |
| `vendor_style` | string | Yes | Vendor style code [1-40 chars] |
| `pick_style` | string | No | Pick strategy: `FIFO`, `FEFO`, `LIFO` |
| `brand_name` | string | Yes | Brand name |
| `product_name` | string | Yes | Product name |
| `product_category` | string | Yes | Product category |
| `product_sub_category` | string | No | Product subcategory |
| `product_description` | string | No | Product description |
| `dim_weight` | number | Yes | Weight |
| `dim_length` | number | Yes | Length |
| `dim_width` | number | Yes | Width |
| `dim_height` | number | Yes | Height |
| `dim_volume` | number | No | Volume |
| `unit_cost` | number | No | Unit cost |
| `price` | number | No | Retail price |
| `is_kit` | boolean | No | Whether SKU is a kit |
| `kit_skus` | array | No | Component SKUs for kits |
| `special_instructions` | string | No | Special handling instructions |

#### Example Request

```json
[
  {
    "customer_identifier": "pants4523",
    "customer_bar_code": "2345687902", 
    "vendor_bar_code": "39485627389",
    "bar_codes": ["85962198623", "3459189485324"],
    "vendor_style": "FP-pants",
    "brand_name": "Freddie's Pants",
    "product_name": "Fancy Pants",
    "product_category": "Apparel",
    "product_sub_category": "pants",
    "product_description": "Very fancy pants with stripes",
    "dim_weight": 0.75,
    "dim_length": 4,
    "dim_width": 2.2, 
    "dim_height": 0.5,
    "unit_cost": 80,
    "price": 160
  }
]
```

### Update SKUs on the Item Master

**PUT** `/skus`

Update one or more SKUs. Requires full SKU object replacement.

#### Request Body Schema

Same schema as Create SKUs (POST). The system will perform a full replacement of the existing SKU data.

### Remove SKUs from the Item Master

**DELETE** `/skus`

Delete one or more SKUs by customer identifier.

#### Request Body Schema

```json
[
  {
    "customer_identifier": "pants4523"
  },
  {
    "customer_identifier": "shirts450293" 
  }
]
```

## Kit SKUs

MasonHub supports kit SKUs - parent SKUs composed of multiple child SKUs. There are two types:

### Pre-built Kits
- Already assembled at the warehouse
- Child units are not available for individual sale
- Faster processing and shipping

### Kit-to-Ship
- Assembled when orders arrive at the distribution center  
- Child units remain available for individual sale
- More flexible inventory management

#### Kit SKU Example

```json
{
  "customer_identifier": "kit00001",
  "brand_name": "Freddie's",
  "product_name": "Shirt and Pants Combo",
  "product_description": "Complete outfit kit",
  "is_kit": true,
  "kit_skus": [
    {
      "customer_identifier": "shirt-001",
      "quantity": 1
    },
    {
      "customer_identifier": "pants-001", 
      "quantity": 1
    }
  ]
}
```

## Barcode Management

SKUs support multiple barcode associations:

- **customer_bar_code**: Primary customer barcode
- **vendor_bar_code**: Vendor's barcode  
- **bar_codes**: Array of additional barcodes

All barcodes must be unique across the system. The API will return validation errors if duplicate barcodes are detected.