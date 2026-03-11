# DuckDB FineType Extension

A DuckDB extension for semantic type classification powered by [FineType](https://github.com/meridian-online/finetype) — a tiered CharCNN model that detects 168 semantic types from raw string values.

## Installation

```sql
INSTALL finetype FROM community;
LOAD finetype;
```

## Functions

### `finetype(value VARCHAR) → VARCHAR`

Classify a single value into one of 168 semantic types.

```sql
SELECT finetype('https://example.com');
-- technology.internet.url

SELECT finetype('user@example.com');
-- identity.person.email

SELECT finetype('2024-01-15');
-- datetime.date.iso

SELECT finetype('550e8400-e29b-41d4-a716-446655440000');
-- technology.cryptographic.uuid

SELECT finetype('true');
-- representation.boolean.terms
```

### `finetype_detail(value VARCHAR) → VARCHAR`

Classify with full detail — returns JSON with type, confidence, and recommended DuckDB type.

```sql
SELECT finetype_detail('192.168.1.1');
-- {"type":"technology.internet.ip_v4","confidence":0.9998,"duckdb_type":"INET"}
```

### `finetype_cast(value VARCHAR) → VARCHAR`

Normalize a value for safe `TRY_CAST()` to its detected DuckDB type.

Returns `VARCHAR` intentionally — a single SQL function can't return different types (DATE, INET, BOOLEAN) for different inputs. Use the two-step pattern: `finetype_cast` normalizes, then `TRY_CAST` converts:

```sql
-- Step 1: finetype_cast normalizes the string
SELECT finetype_cast('01/15/2024');
-- 2024-01-15  (US date → ISO format, still VARCHAR)

-- Step 2: TRY_CAST converts to the native DuckDB type
SELECT TRY_CAST(finetype_cast('01/15/2024') AS DATE);
-- 2024-01-15  (DATE type)
```

More examples:

```sql
SELECT finetype_cast('Yes');
-- true  (boolean normalization)

SELECT finetype_cast('550E8400-E29B-41D4-A716-446655440000');
-- 550e8400-e29b-41d4-a716-446655440000  (UUID lowercase)
```

### `finetype_unpack(json VARCHAR) → VARCHAR`

Recursively classify all scalar values in a JSON document.

```sql
SELECT finetype_unpack('{"email": "user@test.com", "url": "https://example.com"}');
-- Returns annotated JSON with type/confidence/duckdb_type for each field
```

### `finetype_version() → VARCHAR`

Returns the extension version.

```sql
SELECT finetype_version();
-- finetype 0.2.0
```

## Type Taxonomy

FineType classifies values into a three-level taxonomy: `domain.category.type`

| Domain | Categories | Example Types |
|---|---|---|
| **datetime** | date, time, timestamp, epoch, duration, component, offset | `datetime.date.iso`, `datetime.timestamp.rfc_3339` |
| **technology** | internet, cryptographic, development, hardware, code | `technology.internet.url`, `technology.cryptographic.uuid` |
| **geography** | coordinate, location, address, contact, transportation | `geography.location.country`, `geography.coordinate.latitude` |
| **identity** | person, academic, medical, payment | `identity.person.email`, `identity.payment.credit_card_number` |
| **representation** | boolean, discrete, code, numeric, text, file, scientific | `representation.numeric.decimal_number`, `representation.boolean.terms` |
| **container** | object, array, key_value | `container.object.json`, `container.array.comma_separated` |

See the [full type list](https://github.com/meridian-online/finetype#type-taxonomy) for all 168 types.

## DuckDB Type Mapping

Each finetype label maps to a recommended DuckDB type via `finetype_detail()`:

| FineType Label | DuckDB Type |
|---|---|
| `technology.internet.ip_v4` | `INET` |
| `technology.cryptographic.uuid` | `UUID` |
| `datetime.date.*` | `DATE` |
| `datetime.timestamp.*` | `TIMESTAMP` / `TIMESTAMPTZ` |
| `technology.development.boolean` | `BOOLEAN` |
| `representation.numeric.integer_number` | `BIGINT` |
| `representation.numeric.decimal_number` | `DOUBLE` |
| `geography.coordinate.latitude` | `DOUBLE` |
| `container.object.json` | `JSON` |

## Use Cases

**Data profiling** — Automatically detect column types in messy CSV/Parquet data:

```sql
SELECT column_name, finetype(value) AS detected_type, count(*) AS cnt
FROM my_table UNPIVOT (value FOR column_name IN (*))
GROUP BY column_name, detected_type
ORDER BY column_name, cnt DESC;
```

**Schema inference** — Get DuckDB-native type recommendations:

```sql
SELECT column_name,
       finetype_detail(value)::JSON->>'duckdb_type' AS recommended_type
FROM my_table UNPIVOT (value FOR column_name IN (*))
GROUP BY ALL;
```

**Data normalization** — Clean values before casting:

```sql
SELECT TRY_CAST(finetype_cast(messy_date_col) AS DATE) AS clean_date
FROM raw_data;
```

## Building from Source

```bash
git clone --recurse-submodules https://github.com/meridian-online/duckdb-finetype
cd duckdb-finetype
make configure release
```

## License

MIT
