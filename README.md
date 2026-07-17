# Esker API Authentication and Table Query

This project demonstrates how to authenticate with the Esker API and query records from an Esker table using Python.

The workflow consists of two operations:

1. Request an OAuth access token using an Esker API key.
2. Use the access token to retrieve records from a specified Esker table.

## Features

* Requests an Esker access token through the API Key endpoint
* Uses Bearer authentication for subsequent requests
* Queries an Esker table through the REST API
* Selects specific fields
* Sorts returned records
* Limits the maximum number of results
* Displays API errors when requests fail

## Requirements

* Python 3.8 or later
* `requests`

Install the dependency with:

```bash
pip install requests
```

In Google Colab:

```python
!pip install requests
```

## Project Structure

```text
esker-api-example/
├── get_token.py
├── query_table.py
├── requirements.txt
└── README.md
```

The authentication and table-query examples can be saved as separate scripts or combined into a single script.

## Esker Environment

The examples use the following Esker environment:

```text
https://cc6.ondemand.esker.com
```

Replace this URL when working with another Esker environment or tower.

Example:

```python
base_url = "https://cc6.ondemand.esker.com"
```

---

# 1. Obtaining an Access Token

## Token Endpoint

The authentication request is sent to:

```text
POST /api/v1/token/apikey
```

Complete endpoint:

```text
https://cc6.ondemand.esker.com/api/v1/token/apikey
```

## Required Authentication Information

The request requires:

* An application authorization key
* An API Key Tag

The authorization key is sent through the `Authorization` header:

```python
headers = {
    "Content-Type": "application/x-www-form-urlencoded",
    "Authorization": "<APP_AUTHORIZATION_KEY>"
}
```

The API Key Tag is sent in the request body:

```python
data = {
    "ApiKeyTag": "<API_KEY_TAG>"
}
```

## Token Request Example

```python
import requests


base_url = "https://cc6.ondemand.esker.com"
token_url = f"{base_url}/api/v1/token/apikey"

headers = {
    "Content-Type": "application/x-www-form-urlencoded",
    "Authorization": "<APP_AUTHORIZATION_KEY>"
}

payload = {
    "ApiKeyTag": "<API_KEY_TAG>"
}

response = requests.post(
    token_url,
    headers=headers,
    data=payload,
    timeout=30
)

if response.status_code == 200:
    access_token = response.json().get("access_token")

    if access_token:
        print("Access token successfully generated.")
    else:
        print("The response did not contain an access token.")
else:
    print(
        "Token request failed:",
        response.status_code,
        response.text
    )
```

## Successful Response

A successful request should return a JSON object containing an access token.

Example:

```json
{
  "access_token": "<ACCESS_TOKEN>",
  "token_type": "Bearer",
  "expires_in": 3600
}
```

The token is retrieved using:

```python
access_token = response.json().get("access_token")
```

## Important Security Note

Do not print real access tokens in shared notebooks, production logs, screenshots, or public repositories.

Avoid:

```python
print("Token:", access_token)
```

Prefer:

```python
print("Access token successfully generated.")
```

Access tokens, API Key Tags, and application authorization keys must be treated as secrets.

---

# 2. Querying an Esker Table

## Table Endpoint

The table-query endpoint follows this format:

```text
GET /api/v1/table/<TABLE_NAME>
```

Example table:

```text
SO - Materials__
```

Complete endpoint:

```text
https://cc6.ondemand.esker.com/api/v1/table/SO%20-%20Materials__
```

When `requests` is used, URL characters such as spaces are generally encoded automatically.

The endpoint can be constructed with:

```python
table_name = "SO - Materials__"

table_url = (
    f"https://cc6.ondemand.esker.com/"
    f"api/v1/table/{table_name}"
)
```

## Authorization Header

The access token is sent using Bearer authentication:

```python
headers = {
    "Authorization": f"Bearer {access_token}"
}
```

The word `Bearer` must be followed by one space and the access token.

## Query Parameters

The example uses the following parameters:

```python
params = {
    "fields": "Number__,Description__,Warehouse_Code__",
    "orderBy": "Description__ ASC",
    "limit": 50
}
```

### `fields`

Defines which table attributes should be returned.

```text
Number__
Description__
Warehouse_Code__
```

The fields are provided as a comma-separated string:

```python
"fields": "Number__,Description__,Warehouse_Code__"
```

### `orderBy`

Defines the sorting order.

```python
"orderBy": "Description__ ASC"
```

In this example, records are sorted by description in ascending order.

Another possible value is:

```python
"orderBy": "Description__ DESC"
```

### `limit`

Defines the maximum number of records returned:

```python
"limit": 50
```

## Table Query Example

```python
import requests


base_url = "https://cc6.ondemand.esker.com"
table_name = "SO - Materials__"
access_token = "<ACCESS_TOKEN>"

table_url = f"{base_url}/api/v1/table/{table_name}"

params = {
    "fields": "Number__,Description__,Warehouse_Code__",
    "orderBy": "Description__ ASC",
    "limit": 50
}

headers = {
    "Authorization": f"Bearer {access_token}",
    "Accept": "application/json"
}

response = requests.get(
    table_url,
    headers=headers,
    params=params,
    timeout=30
)

if response.status_code == 200:
    response_data = response.json()
    print("API response:")
    print(response_data)
else:
    print(
        "Table request failed:",
        response.status_code,
        response.text
    )
```

---

# Complete Authentication and Query Example

The following version generates the access token and immediately uses it to query the Esker table.

```python
import os

import requests


BASE_URL = "https://cc6.ondemand.esker.com"
TOKEN_ENDPOINT = f"{BASE_URL}/api/v1/token/apikey"

TABLE_NAME = "SO - Materials__"
TABLE_ENDPOINT = f"{BASE_URL}/api/v1/table/{TABLE_NAME}"


def get_access_token(
    authorization_key,
    api_key_tag
):
    headers = {
        "Content-Type": "application/x-www-form-urlencoded",
        "Authorization": authorization_key
    }

    payload = {
        "ApiKeyTag": api_key_tag
    }

    try:
        response = requests.post(
            TOKEN_ENDPOINT,
            headers=headers,
            data=payload,
            timeout=30
        )

        response.raise_for_status()

        response_data = response.json()
        access_token = response_data.get("access_token")

        if not access_token:
            raise ValueError(
                "The token response did not contain "
                "'access_token'."
            )

        return access_token

    except requests.exceptions.Timeout as error:
        raise RuntimeError(
            "The token request timed out."
        ) from error

    except requests.exceptions.ConnectionError as error:
        raise RuntimeError(
            "Could not connect to the Esker API."
        ) from error

    except requests.exceptions.HTTPError as error:
        raise RuntimeError(
            f"Token request failed with HTTP "
            f"{response.status_code}: {response.text}"
        ) from error

    except requests.exceptions.JSONDecodeError as error:
        raise RuntimeError(
            "The token endpoint returned invalid JSON."
        ) from error


def query_table(
    access_token,
    table_name,
    fields=None,
    order_by=None,
    limit=50
):
    endpoint = (
        f"{BASE_URL}/api/v1/table/{table_name}"
    )

    headers = {
        "Authorization": f"Bearer {access_token}",
        "Accept": "application/json"
    }

    params = {
        "limit": limit
    }

    if fields:
        params["fields"] = ",".join(fields)

    if order_by:
        params["orderBy"] = order_by

    try:
        response = requests.get(
            endpoint,
            headers=headers,
            params=params,
            timeout=30
        )

        response.raise_for_status()

        return response.json()

    except requests.exceptions.Timeout as error:
        raise RuntimeError(
            "The table query timed out."
        ) from error

    except requests.exceptions.ConnectionError as error:
        raise RuntimeError(
            "Could not connect to the Esker API."
        ) from error

    except requests.exceptions.HTTPError as error:
        raise RuntimeError(
            f"Table query failed with HTTP "
            f"{response.status_code}: {response.text}"
        ) from error

    except requests.exceptions.JSONDecodeError as error:
        raise RuntimeError(
            "The table endpoint returned invalid JSON."
        ) from error


def main():
    authorization_key = os.getenv(
        "ESKER_AUTHORIZATION_KEY"
    )

    api_key_tag = os.getenv(
        "ESKER_API_KEY_TAG"
    )

    if not authorization_key:
        raise RuntimeError(
            "The ESKER_AUTHORIZATION_KEY environment "
            "variable is not configured."
        )

    if not api_key_tag:
        raise RuntimeError(
            "The ESKER_API_KEY_TAG environment variable "
            "is not configured."
        )

    access_token = get_access_token(
        authorization_key=authorization_key,
        api_key_tag=api_key_tag
    )

    print("Access token successfully generated.")

    result = query_table(
        access_token=access_token,
        table_name=TABLE_NAME,
        fields=[
            "Number__",
            "Description__",
            "Warehouse_Code__"
        ],
        order_by="Description__ ASC",
        limit=50
    )

    print("Table query completed successfully.")
    print(result)


if __name__ == "__main__":
    main()
```

---

# Environment Variables

Sensitive credentials should be stored in environment variables instead of being written directly in the source code.

## Linux and macOS

```bash
export ESKER_AUTHORIZATION_KEY="your-authorization-key"
export ESKER_API_KEY_TAG="your-api-key-tag"
```

Run the script:

```bash
python esker_api_query.py
```

## Windows PowerShell

```powershell
$env:ESKER_AUTHORIZATION_KEY="your-authorization-key"
$env:ESKER_API_KEY_TAG="your-api-key-tag"
```

Run:

```powershell
python esker_api_query.py
```

## Google Colab

```python
import os

os.environ["ESKER_AUTHORIZATION_KEY"] = (
    "your-authorization-key"
)

os.environ["ESKER_API_KEY_TAG"] = (
    "your-api-key-tag"
)
```

Avoid including credentials in notebooks that will be shared.

## `.env` Alternative

A `.env` file can also be used:

```text
ESKER_AUTHORIZATION_KEY=your-authorization-key
ESKER_API_KEY_TAG=your-api-key-tag
```

Install `python-dotenv`:

```bash
pip install python-dotenv
```

Load the values:

```python
from dotenv import load_dotenv

load_dotenv()
```

Add `.env` to `.gitignore`:

```gitignore
.env
```

---

# Response Handling

## Successful Query

A successful table request returns JSON data.

The exact structure depends on the Esker API response.

Example:

```json
{
  "value": [
    {
      "Number__": "MAT-001",
      "Description__": "Example material",
      "Warehouse_Code__": "WH01"
    },
    {
      "Number__": "MAT-002",
      "Description__": "Second material",
      "Warehouse_Code__": "WH02"
    }
  ]
}
```

The returned records can be accessed with:

```python
records = response_data.get("value", [])
```

Example:

```python
for record in records:
    print(
        record.get("Number__"),
        record.get("Description__"),
        record.get("Warehouse_Code__")
    )
```

## Creating a DataFrame

The returned records can be converted to a pandas DataFrame:

```python
import pandas as pd


records = response_data.get("value", [])

dataframe = pd.DataFrame(records)

print(dataframe.head())
```

They can also be exported to CSV:

```python
dataframe.to_csv(
    "materials.csv",
    index=False,
    encoding="utf-8-sig"
)
```

---

# Error Handling

The basic examples check the HTTP status code:

```python
if response.status_code == 200:
```

A more robust implementation can use:

```python
response.raise_for_status()
```

This raises an exception for unsuccessful HTTP responses.

## Common Status Codes

### `200 OK`

The request completed successfully.

### `400 Bad Request`

The request may contain:

* An invalid field name
* An invalid table name
* An invalid query parameter
* A malformed request body

### `401 Unauthorized`

Possible causes include:

* Invalid application authorization key
* Invalid access token
* Expired access token
* Incorrect Bearer header

### `403 Forbidden`

The authenticated application or user may not have permission to access the requested table.

### `404 Not Found`

Possible causes include:

* Incorrect Esker environment
* Incorrect endpoint
* Incorrect table name

### `429 Too Many Requests`

The API request limit may have been exceeded.

The client should wait and retry according to the API response headers.

### `500 Internal Server Error`

An unexpected error occurred on the Esker server.

---

# Access Token Expiration

Access tokens are temporary.

When a token expires, subsequent API requests may return:

```text
401 Unauthorized
```

The application should request a new token instead of permanently storing an expired one.

A common workflow is:

```text
Request access token
        │
        ▼
Call Esker API
        │
        ▼
Token expired?
   │           │
  No          Yes
   │           │
   ▼           ▼
Continue    Request new token
```

---

# URL and Table Names

Esker table names may contain:

* Spaces
* Hyphens
* Underscores
* Double underscores

Example:

```text
SO - Materials__
```

For explicit URL encoding, use:

```python
from urllib.parse import quote


encoded_table_name = quote(
    table_name,
    safe=""
)

endpoint = (
    f"{BASE_URL}/api/v1/table/"
    f"{encoded_table_name}"
)
```

For the table:

```text
SO - Materials__
```

the encoded value becomes:

```text
SO%20-%20Materials__
```

---

# Field Names

The example requests:

```text
Number__
Description__
Warehouse_Code__
```

These names must match the exact technical field names configured in the Esker table.

A displayed field label may be different from its technical API attribute name.

For example:

| Display name   | Technical name     |
| -------------- | ------------------ |
| Number         | `Number__`         |
| Description    | `Description__`    |
| Warehouse Code | `Warehouse_Code__` |

Invalid technical field names may cause the API to reject the request or omit the field.

---

# Security Recommendations

* Never commit access tokens to Git.
* Never commit API Key Tags or authorization keys.
* Do not include credentials in screenshots.
* Do not print tokens in production logs.
* Use environment variables or a secrets manager.
* Grant only the permissions required by the application.
* Rotate credentials when exposure is suspected.
* Use HTTPS for every request.
* Configure request timeouts.
* Avoid storing access tokens longer than necessary.

Example `.gitignore`:

```gitignore
.env
*.log
__pycache__/
.ipynb_checkpoints/
```

## Exposed Credentials

When a real credential has been accidentally published or shared, it should be revoked or rotated instead of merely removing it from the source code.

---

# Limitations

* The example queries only one table.
* It retrieves a maximum of 50 records.
* Pagination is not implemented.
* Advanced table filters are not included.
* Token refresh is not automatic.
* Retry behavior is not implemented.
* The script prints the complete JSON response.
* The exact response structure may vary according to the endpoint and Esker configuration.
* Table and field names are specific to the configured Esker environment.

## License

This project is intended for internal Esker API integration, master-data retrieval, testing, and educational purposes.
