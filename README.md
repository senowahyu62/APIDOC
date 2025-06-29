# API Server Documentation - Endpoints

## 📋 Daftar Isi
1. [Overview](#overview)
2. [Authentication](#authentication)
3. [Base URL & Configuration](#base-url--configuration)
4. [Response Format](#response-format)
5. [Error Handling](#error-handling)
6. [Endpoints](#endpoints)
   - [General Endpoints](#general-endpoints)
   - [User Management](#user-management)
   - [Transaction Endpoints](#transaction-endpoints)
   - [Product Endpoints](#product-endpoints)
   - [Utility Endpoints](#utility-endpoints)
7. [Rate Limiting](#rate-limiting)
8. [Examples](#examples)

---

## 📖 Overview

API Server menyediakan RESTful API untuk integrasi dengan sistem panel pulsa dan data. Server berjalan pada port `4444` dan mendukung berbagai operasi termasuk manajemen user, transaksi, topup, dan utilitas lainnya.

**Base URL**: `http://0.0.0.0:4444`
**Protocol**: HTTP/HTTPS
**Content-Type**: `application/json`
**Encoding**: UTF-8

---

## 🔐 Authentication

### API Key Authentication
Sebagian besar endpoint memerlukan autentikasi menggunakan API Key yang dikirim melalui header.

**Header Required**:
```
apikey: YOUR_API_KEY_HERE
```

### Alternative Authentication Methods
API Key juga dapat dikirim melalui:
- **Body parameter**: `apikey` dalam request body
- **Query parameter**: `?apikey=YOUR_API_KEY`
- **Authorization header**: `Authorization: YOUR_API_KEY`

### Mendapatkan API Key
- API Key digenerate otomatis saat user terdaftar via Telegram Bot
- Dapat di-regenerate melalui bot dengan command `/start` → "🔑 Generate APIKey"
- Format: 32 karakter alfanumerik random

---

## 🌐 Base URL & Configuration

```
Server Host: 0.0.0.0
Server Port: 4444
Max Request Size: 10MB
Timeout: 30 seconds
CORS: Enabled (all origins)
```

### CORS Headers
```
Access-Control-Allow-Origin: *
Access-Control-Allow-Methods: GET, POST, PUT, DELETE, OPTIONS
Access-Control-Allow-Headers: Origin, X-Requested-With, Content-Type, Accept, Authorization, apikey
```

---

## 📝 Response Format

### Success Response
```json
{
  "status": true,
  "message": "Operation successful",
  "data": {
    // Response data object
  },
  "timestamp": "2024-01-15 10:30:45"
}
```

### Error Response
```json
{
  "status": false,
  "message": "Error description",
  "error": "ERROR_CODE",
  "details": "Detailed error information (development only)"
}
```

### Common Error Codes
| Code | Description |
|------|-------------|
| `MISSING_API_KEY` | API key tidak disertakan |
| `INVALID_API_KEY` | API key tidak valid |
| `MISSING_PARAMETER` | Parameter required tidak ada |
| `INVALID_FORMAT` | Format data tidak valid |
| `DATABASE_ERROR` | Kesalahan database |
| `SERVER_ERROR` | Kesalahan internal server |
| `NOT_FOUND` | Resource tidak ditemukan |
| `REQUEST_TIMEOUT` | Request timeout |

---

## 📡 Endpoints

### General Endpoints

#### `GET /`
**Deskripsi**: Informasi API dan dokumentasi endpoints

**Authentication**: ❌ Tidak diperlukan

**Request**:
```bash
curl -X GET http://0.0.0.0:4444/
```

**Response**:
```json
{
  "status": true,
  "message": "Profil API Server",
  "version": "1.0.0",
  "server_time": "15/01/2024 10.30.45",
  "endpoints": {
    // List semua endpoints dengan dokumentasi
  },
  "example_usage": {
    // Contoh penggunaan setiap endpoint
  }
}
```

---

#### `GET /health`
**Deskripsi**: Health check server dan database

**Authentication**: ❌ Tidak diperlukan

**Request**:
```bash
curl -X GET http://0.0.0.0:4444/health
```

**Response Success**:
```json
{
  "status": true,
  "message": "API server is running",
  "server_time": "15/01/2024 10.30.45",
  "uptime_seconds": 3600,
  "memory_usage": {
    "used": "45 MB",
    "total": "128 MB"
  },
  "database": {
    "status": "connected",
    "error": null
  },
  "node_version": "v18.17.0"
}
```

**Response Error** (503):
```json
{
  "status": false,
  "message": "Database connection error",
  "database": {
    "status": "error",
    "error": "Connection timeout"
  }
}
```

---

### User Management

#### `GET /profil`
**Deskripsi**: Mendapatkan informasi profil user

**Authentication**: ✅ Required (API Key)

**Headers**:
```
apikey: YOUR_API_KEY_HERE
```

**Request**:
```bash
curl -X GET http://0.0.0.0:4444/profil \
  -H "apikey: YOUR_API_KEY"
```

**Response Success**:
```json
{
  "status": true,
  "message": "Profil berhasil diambil",
  "data": {
    "user_id": 123456789,
    "username": "John Doe",
    "balance": 50000
  },
  "timestamp": "15/01/2024 10.30.45"
}
```

**Response Error** (401):
```json
{
  "status": false,
  "message": "API key tidak valid atau user tidak ditemukan",
  "error": "INVALID_API_KEY"
}
```

---

### Transaction Endpoints

#### `POST /tembak/v1`
**Deskripsi**: Melakukan pembelian produk

**Authentication**: ❌ Tidak diperlukan (API key dalam body)

**Headers**:
```
Content-Type: application/json
```

**Request Body**:
```json
{
  "apikey": "YOUR_API_KEY_HERE",
  "package_id": "PACKAGE_ID_HERE",
  "msisdn": "081234567890"
}
```

**Request Example**:
```bash
curl -X POST http://0.0.0.0:4444/tembak/v1 \
  -H "Content-Type: application/json" \
  -d '{
    "apikey": "abc123def456ghi789jkl012mno345pq",
    "package_id": "XL_5GB_30D",
    "msisdn": "081234567890"
  }'
```

**Response Success**:
```json
{
  "status": true,
  "message": "Pembelian berhasil dibuat",
  "data": {
    "order_id": "a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6",
    "package_id": "XL_5GB_30D",
    "nama_produk": "XL Internet 5GB 30 Hari",
    "msisdn": "081234567890",
    "harga": 25000,
    "status": "PENDING",
    "date": "2024-01-15T03:30:45.123Z"
  },
  "timestamp": "15/01/2024 10.30.45"
}
```

**Response Error** (400):
```json
{
  "status": false,
  "message": "MSISDN harus 10-15 digit",
  "error": "INVALID_MSISDN_LENGTH"
}
```

**Validation Rules**:
- `apikey`: Required, string, valid API key
- `package_id`: Required, string, must exist in database
- `msisdn`: Required, string, 10-15 digits, numeric only

---

#### `POST /tembak/history`
**Deskripsi**: Melihat detail transaksi pembelian berdasarkan order_id

**Authentication**: ❌ Tidak diperlukan

**Headers**:
```
Content-Type: application/json
```

**Request Body**:
```json
{
  "order_id": "ORDER_ID_HERE"
}
```

**Request Example**:
```bash
curl -X POST http://0.0.0.0:4444/tembak/history \
  -H "Content-Type: application/json" \
  -d '{
    "order_id": "a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6"
  }'
```

**Response Success**:
```json
{
  "status": true,
  "message": "Data transaksi berhasil ditemukan",
  "data": {
    "order_id": "a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6",
    "package_id": "XL_5GB_30D",
    "nama_produk": "XL Internet 5GB 30 Hari",
    "msisdn": "081234567890",
    "harga": 25000,
    "status": "SUCCESS",
    "sn": "SN123456789",
    "message": "Transaksi berhasil diproses",
    "channel": "v1",
    "date": "2024-01-15T03:30:45.123Z",
    "date_formatted": "15/01/2024 10.30.45"
  },
  "timestamp": "15/01/2024 10.30.45"
}
```

**Response Error** (404):
```json
{
  "status": false,
  "message": "Order ID tidak ditemukan",
  "error": "ORDER_NOT_FOUND",
  "order_id": "invalid_order_id"
}
```

**Validation Rules**:
- `order_id`: Required, string, tidak boleh kosong

---

#### `POST /topup`
**Deskripsi**: Melakukan topup saldo via QRIS

**Authentication**: ✅ Required (API Key)

**Headers**:
```
Content-Type: application/json
apikey: YOUR_API_KEY_HERE
```

**Request Body**:
```json
{
  "amount": 10000
}
```

**Request Example**:
```bash
curl -X POST http://0.0.0.0:4444/topup \
  -H "Content-Type: application/json" \
  -H "apikey: YOUR_API_KEY" \
  -d '{
    "amount": 10000
  }'
```

**Response Success**:
```json
{
  "status": true,
  "message": "Topup berhasil dibuat",
  "data": {
    "trx_id": "abc123def456",
    "qris": "00020101021126670016COM.NOBUBANK.WWW...",
    "amount_requested": 10000,
    "amount_actual": 10543,
    "actual_balance": 10469,
    "balance_before": 0,
    "status": "PENDING",
    "expired_in": "30 menit"
  },
  "timestamp": "15/01/2024 10.30.45"
}
```

**Response Error** (400):
```json
{
  "status": false,
  "message": "Amount harus antara 1.000 dan 450.000",
  "error": "INVALID_AMOUNT_RANGE"
}
```

**Validation Rules**:
- `amount`: Required, integer, 1000-450000

**QRIS Flow**:
1. User input amount → amount_requested
2. System add unique ticket (100-1000) → amount_actual
3. Generate QRIS dengan amount_actual
4. User bayar sesuai amount_actual
5. System potong fee 0.7% → actual_balance
6. Update user balance dengan actual_balance

---

#### `POST /topup/history`
**Deskripsi**: Melihat detail transaksi topup berdasarkan trx_id

**Authentication**: ❌ Tidak diperlukan

**Headers**:
```
Content-Type: application/json
```

**Request Body**:
```json
{
  "trx_id": "TRX_ID_HERE"
}
```

**Request Example**:
```bash
curl -X POST http://0.0.0.0:4444/topup/history \
  -H "Content-Type: application/json" \
  -d '{
    "trx_id": "abc123def456"
  }'
```

**Response Success**:
```json
{
  "status": true,
  "message": "Data transaksi topup berhasil ditemukan",
  "data": {
    "trx_id": "abc123def456",
    "amount_requested": 10000,
    "amount_actual": 10543,
    "actual_balance": 10469,
    "status": "SUCCESS",
    "balance_before": 0,
    "qris": "00020101021126670016COM.NOBUBANK.WWW...",
    "date": "2024-01-15T03:30:45.123Z",
    "date_formatted": "15/01/2024 10.30.45",
    "fee_amount": 74,
    "ticket_amount": 543,
    "status_description": "Pembayaran berhasil - saldo sudah ditambahkan"
  },
  "timestamp": "15/01/2024 10.30.45"
}
```

**Status Descriptions**:
| Status | Description |
|--------|-------------|
| `PENDING` | Menunggu pembayaran - silakan scan QRIS |
| `SUCCESS` | Pembayaran berhasil - saldo sudah ditambahkan |
| `FAILED` | Pembayaran gagal |
| `EXPIRED` | Pembayaran expired - sudah lebih dari 30 menit |

---

### Product Endpoints

#### `GET /list/v1`
**Deskripsi**: Mendapatkan daftar produk dengan harga termasuk markup user

**Authentication**: ✅ Required (API Key)

**Headers**:
```
apikey: YOUR_API_KEY_HERE
```

**Request**:
```bash
curl -X GET http://0.0.0.0:4444/list/v1 \
  -H "apikey: YOUR_API_KEY"
```

**Response Success**:
```json
{
  "status": true,
  "message": "Daftar produk berhasil diambil",
  "data": [
    {
      "package_id": "XL_5GB_30D",
      "nama_produk": "XL Internet 5GB 30 Hari",
      "harga_jual": 27000,
      "deskripsi": "Paket internet XL 5GB berlaku 30 hari",
      "channel": "v1",
      "status": "Berbayar"
    },
    {
      "package_id": "TSEL_10GB_30D",
      "nama_produk": "Telkomsel Internet 10GB 30 Hari", 
      "harga_jual": 0,
      "deskripsi": "Paket internet Telkomsel 10GB berlaku 30 hari",
      "channel": "v2",
      "status": "Gratis"
    }
  ],
  "statistics": {
    "total_products": 150,
    "produk_gratis": 5,
    "produk_berbayar": 145,
    "global_markup": 1500,
    "custom_markup_count": 12
  },
  "timestamp": "15/01/2024 10.30.45"
}
```

**Markup Logic**:
1. **Product-specific markup** (prioritas tertinggi)
2. **Global markup** (fallback)
3. **No markup** (harga dasar)

**Product Status**:
- `Gratis`: Produk dengan harga dasar 0
- `Berbayar`: Produk dengan harga dasar > 0

---

#### `GET /stok`
**Deskripsi**: Mengecek status stok produk dari panel.khfy-store.com

**Authentication**: ❌ Tidak diperlukan

**Request**:
```bash
curl -X GET http://0.0.0.0:4444/stok
```

**Response Success**:
```json
{
  "status": true,
  "message": "Data stok berhasil diambil",
  "data": {
    // Raw response dari panel.khfy-store.com API
    "XL 5GB": "25 tersedia",
    "Telkomsel 10GB": "0 tersedia",
    "Indosat 15GB": "50 tersedia"
  }
}
```

**Response Error** (500):
```json
{
  "status": false,
  "message": "Gagal mengambil data stok: Request timeout after 10 seconds",
  "error": "STOCK_CHECK_FAILED"
}
```

**External API**: `https://panel.khfy-store.com/api/api-xl-v7/cek_stock_akrab`
**Timeout**: 10 seconds

---

### Utility Endpoints

#### `POST /dompul`
**Deskripsi**: Cek detail paket kartu dengan menjalankan dompul.js

**Authentication**: ❌ Tidak diperlukan

**Headers**:
```
Content-Type: application/json
```

**Request Body**:
```json
{
  "msisdn": "087711498204"
}
```

**Request Example**:
```bash
curl -X POST http://0.0.0.0:4444/dompul \
  -H "Content-Type: application/json" \
  -d '{
    "msisdn": "087711498204"
  }'
```

**Response Success**:
```json
{
  "status": true,
  "message": "SUCCESS",
  "data": {
    "msisdn": "6287711498204",
    "hasil": "📃 RESULT:<br><br>MSISDN: 6287711498204<br><br>Tipe Kartu: XL Prepaid<br>Status 4G: Active<br>Status Dukcapil: Verified<br>Umur Kartu: 2 tahun<br>Masa Aktif: 30 hari<br>===========================<br>🎁 Quota: Internet 5GB<br>🍂 Aktif Hingga: 2024-02-15 23:59:59<br>===========================<br>",
    "data_sp": {
      "prefix": {
        "success": true,
        "value": "XL Prepaid"
      },
      "dukcapil": {
        "success": true,
        "value": "Verified"
      },
      "quotas": {
        "success": true,
        "value": [
          {
            "packages": {
              "name": "Internet 5GB",
              "expDate": "2024-02-15T23:59:59"
            },
            "benefits": [
              {
                "type": "DATA",
                "bname": "Internet",
                "quota": "5 GB",
                "remaining": "3.2 GB"
              }
            ]
          }
        ]
      }
    }
  },
  "timestamp": "15/01/2024 10.30.45"
}
```

**Response Error**:
```json
{
  "status": false,
  "message": "ERROR",
  "data": {
    "msisdn": "6287711498204",
    "hasil": "📃 RESULT:<br><br>MSISDN: 6287711498204<br><br>❌ Nomor tidak ditemukan atau tidak aktif<br>",
    "data_sp": {}
  }
}
```

**Validation Rules**:
- `msisdn`: Required, string, 10-15 digits, numeric only

**External Script**: `dompul.js`
**Timeout**: 25 seconds

---

#### `POST /area`
**Deskripsi**: Cari informasi area kuota AKRAB XL & AXIS berdasarkan kota

**Authentication**: ❌ Tidak diperlukan

**Headers**:
```
Content-Type: application/json
```

**Request Body**:
```json
{
  "kota": "bandung"
}
```

**Request Example**:
```bash
curl -X POST http://0.0.0.0:4444/area \
  -H "Content-Type: application/json" \
  -d '{
    "kota": "bandung"
  }'
```

**Response Success**:
```json
{
  "status": "success",
  "keyword": "bandung",
  "total": 3,
  "results": [
    {
      "pulau": "Jawa",
      "provinsi": "Jawa Barat",
      "kabupaten": "Kab. Bandung",
      "area": "Area 1"
    },
    {
      "pulau": "Jawa", 
      "provinsi": "Jawa Barat",
      "kabupaten": "Kab. Bandung Barat",
      "area": "Area 1"
    },
    {
      "pulau": "Jawa",
      "provinsi": "Jawa Barat", 
      "kabupaten": "Kota Bandung",
      "area": "Area 1"
    }
  ],
  "timestamp": "15/01/2024 10.30.45"
}
```

**Response Not Found**:
```json
{
  "status": "not_found",
  "keyword": "invalidcity",
  "total": 0,
  "message": "Tidak ada kabupaten/kota yang mengandung \"invalidcity\".",
  "results": []
}
```

**Validation Rules**:
- `kota`: Required, string, tidak boleh kosong

**External Script**: `area.js`
**Timeout**: 25 seconds

**Area Coverage**:
- Pulau: Jawa, Sumatera, Kalimantan, Sulawesi, Nusa Tenggara, Maluku, Papua
- Area: Area 1, Area 2, Area 3, Area 4
- Support fuzzy search (partial matching)

---

## ⚡ Rate Limiting

### Current Implementation
- **No explicit rate limiting** implemented
- **Timeout protection**: 25-30 seconds per request
- **Concurrent request handling**: Supported
- **Memory limits**: 10MB per request

### Recommended Client-Side Limits
- **API calls**: Max 10 requests per second
- **File uploads**: Max 1 concurrent upload
- **Long-running operations**: Wait for response before next request

---

## 📚 Examples

### Complete User Journey

#### 1. Get User Profile
```bash
curl -X GET http://0.0.0.0:4444/profil \
  -H "apikey: abc123def456ghi789jkl012mno345pq"
```

#### 2. Check Available Products
```bash
curl -X GET http://0.0.0.0:4444/list/v1 \
  -H "apikey: abc123def456ghi789jkl012mno345pq"
```

#### 3. Check Stock Status
```bash
curl -X GET http://0.0.0.0:4444/stok
```

#### 4. Check Card Details (Optional)
```bash
curl -X POST http://0.0.0.0:4444/dompul \
  -H "Content-Type: application/json" \
  -d '{"msisdn": "081234567890"}'
```

#### 5. Check Area Information (Optional)
```bash
curl -X POST http://0.0.0.0:4444/area \
  -H "Content-Type: application/json" \
  -d '{"kota": "jakarta"}'
```

#### 6. Topup Balance (If Needed)
```bash
curl -X POST http://0.0.0.0:4444/topup \
  -H "Content-Type: application/json" \
  -H "apikey: abc123def456ghi789jkl012mno345pq" \
  -d '{"amount": 50000}'
```

#### 7. Check Topup Status
```bash
curl -X POST http://0.0.0.0:4444/topup/history \
  -H "Content-Type: application/json" \
  -d '{"trx_id": "topup_trx_id_here"}'
```

#### 8. Purchase Product
```bash
curl -X POST http://0.0.0.0:4444/tembak/v1 \
  -H "Content-Type: application/json" \
  -d '{
    "apikey": "abc123def456ghi789jkl012mno345pq",
    "package_id": "XL_5GB_30D",
    "msisdn": "081234567890"
  }'
```

#### 9. Check Purchase Status
```bash
curl -X POST http://0.0.0.0:4444/tembak/history \
  -H "Content-Type: application/json" \
  -d '{"order_id": "purchase_order_id_here"}'
```

### Error Handling Example

#### JavaScript Implementation
```javascript
class APIClient {
  constructor(baseURL, apiKey) {
    this.baseURL = baseURL;
    this.apiKey = apiKey;
  }

  async request(endpoint, options = {}) {
    const config = {
      method: 'GET',
      headers: {
        'Content-Type': 'application/json',
        'apikey': this.apiKey,
        ...options.headers
      },
      ...options
    };

    try {
      const response = await fetch(`${this.baseURL}${endpoint}`, config);
      const data = await response.json();
      
      if (!response.ok) {
        throw new Error(`HTTP ${response.status}: ${data.message || 'Unknown error'}`);
      }
      
      if (!data.status) {
        throw new Error(`API Error: ${data.message} (${data.error})`);
      }
      
      return data;
    } catch (error) {
      console.error(`API Request failed:`, error);
      throw error;
    }
  }

  async getProfile() {
    return this.request('/profil');
  }

  async getProducts() {
    return this.request('/list/v1');
  }

  async purchaseProduct(packageId, msisdn) {
    return this.request('/tembak/v1', {
      method: 'POST',
      body: JSON.stringify({
        apikey: this.apiKey,
        package_id: packageId,
        msisdn: msisdn
      })
    });
  }

  async topupBalance(amount) {
    return this.request('/topup', {
      method: 'POST',
      body: JSON.stringify({ amount })
    });
  }
}

// Usage
const client = new APIClient('http://0.0.0.0:4444', 'your_api_key_here');

try {
  const profile = await client.getProfile();
  console.log('User balance:', profile.data.balance);
  
  const products = await client.getProducts();
  console.log('Available products:', products.data.length);
  
  const purchase = await client.purchaseProduct('XL_5GB_30D', '081234567890');
  console.log('Purchase order ID:', purchase.data.order_id);
} catch (error) {
  console.error('Operation failed:', error.message);
}
```

#### Python Implementation
```python
import requests
import json
from typing import Dict, Any, Optional

class APIClient:
    def __init__(self, base_url: str, api_key: str):
        self.base_url = base_url
        self.api_key = api_key
        self.session = requests.Session()
        self.session.headers.update({
            'Content-Type': 'application/json',
            'apikey': api_key
        })

    def _request(self, endpoint: str, method: str = 'GET', data: Optional[Dict] = None) -> Dict[str, Any]:
        url = f"{self.base_url}{endpoint}"
        
        try:
            if method.upper() == 'GET':
                response = self.session.get(url, timeout=30)
            else:
                response = self.session.post(url, json=data, timeout=30)
            
            response.raise_for_status()
            result = response.json()
            
            if not result.get('status', False):
                raise Exception(f"API Error: {result.get('message', 'Unknown error')} ({result.get('error', 'UNKNOWN')})")
            
            return result
        
        except requests.RequestException as e:
            raise Exception(f"Request failed: {str(e)}")
        except json.JSONDecodeError as e:
            raise Exception(f"Invalid JSON response: {str(e)}")

    def get_profile(self) -> Dict[str, Any]:
        return self._request('/profil')

    def get_products(self) -> Dict[str, Any]:
        return self._request('/list/v1')

    def purchase_product(self, package_id: str, msisdn: str) -> Dict[str, Any]:
        data = {
            'apikey': self.api_key,
            'package_id': package_id,
            'msisdn': msisdn
        }
        return self._request('/tembak/v1', 'POST', data)

    def topup_balance(self, amount: int) -> Dict[str, Any]:
        data = {'amount': amount}
        return self._request('/topup', 'POST', data)

    def check_stock(self) -> Dict[str, Any]:
        return self._request('/stok')

# Usage
client = APIClient('http://0.0.0.0:4444', 'your_api_key_here')

try:
    profile = client.get_profile()
    print(f"User balance: Rp {profile['data']['balance']:,}")
    
    products = client.get_products()
    print(f"Available products: {len(products['data'])}")
    
    stock = client.check_stock()
    print("Stock status retrieved successfully")
    
    purchase = client.purchase_product('XL_5GB_30D', '081234567890')
    print(f"Purchase order ID: {purchase['data']['order_id']}")
    
except Exception as e:
    print(f"Operation failed: {str(e)}")
```

---

## 🔧 Development Notes

### Adding New Endpoints
1. **Define route** dengan method yang sesuai
2. **Add authentication** jika diperlukan
3. **Validate input** parameters
4. **Implement business logic**
5. **Handle errors** dengan proper response codes
6. **Add logging** untuk debugging
7. **Update documentation**

### Best Practices
- **Always validate input** sebelum processing
- **Use database transactions** untuk operasi yang kompleks
- **Implement proper error handling** dengan meaningful messages
- **Log semua operations** untuk debugging dan monitoring
- **Use connection pooling** untuk database efficiency
- **Handle timeouts** untuk external API calls
- **Sanitize user input** untuk security

### Security Considerations
- **API Key validation** untuk setiap protected endpoint
- **Input sanitization** untuk mencegah injection attacks
- **Rate limiting** untuk mencegah abuse
- **HTTPS enforcement** di production
- **Error message filtering** untuk tidak expose sensitive info
- **Database query parameterization** untuk mencegah SQL injection

---

*Dokumentasi ini akan terus diupdate seiring dengan pengembangan fitur baru.*
