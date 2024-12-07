
# İndirim Servisi

**İstek Tipi:** `POST`  
**Endpoint:** `https://test.cepteisler.com/discounts`

## İndirim Servisi cURL

```bash
curl -X 'POST' \
  'https://test.cepteisler.com/discounts' \
  -H 'accept: application/json' \
  -H 'Content-Type: application/json' \
  -d '{
  "barcodes": [
    {
      "amount": 0,
      "barcode": "string",
      "name": "string",
      "price": 0
    }
  ],
  "branchCode": "string",
  "transactionType": 0,
  "userCode": "string"
}'
```

### Parametreler

- `transactionType = 1`: Alışveriş  
- `transactionType = 2`: İade
- `userCode`: Müşteri Kodu

### Örnek İstek Gönderimi (Request Body)

```json
{
  "barcodes": [
    {
      "amount": 1,
      "barcode": "9786057530851",
      "name": "TYT HIZ ve RENK S.B. TÜRKÇE - 2024-25",
      "price": 220
    },
    {
      "amount": 1,
      "barcode": "9786259793627",
      "name": "BİLGİ ARŞİVİ TYT S.B. TARİH - 2024-25",
      "price": 160
    }
  ],
  "branchCode": "78",
  "transactionType": 1,
  "userCode": "723760"
}
```

### Örnek Yanıt (Response)

```json
{
  "success": true,
  "data": [
    {
      "barkod": "9786057530851",
      "discount": 5,
      "discountId": 15,
      "price": 209
    },
    {
      "barkod": "9786259793627",
      "discount": 0,
      "discountId": null,
      "price": 160
    }
  ]
}
```

### Hata Durumları

- **API Key girilmemişse:**
  - **HTTP Status Code:** 403
  - Örnek cevap:
    ```bash
    403 Client Error: Forbidden for url: https://test.cepteisler.com/discounts
    ```
    
- **Hatalı müşteri kodu girilirse (500):**
    ```json
    {
      "error": true,
      "message": "girilen müşteri kodu bulunamadı"
    }
    ```

- **Yanlış transaction type girilirse (400):**
    ```json
    {
      "error": true,
      "message": "Invalid transaction type"
    }
    ```

- **Diğer hatalar (400):**  
  - Fiyat hatası:
    ```json
    {
      "error": true,
      "message": "price in barcodes cannot be zero or empty"
    }
    ```  
  - Barcode array hatası:
    ```json
    {
      "error": true,
      "message": "barcodes array cannot be empty"
    }
    ```

---

# Alışveriş Servisi

**İstek Tipi:** `POST`  
**Endpoint:** `http://test.cepteisler.com/purchase`

## Alışveriş Servisi cURL

```bash
curl -X 'POST' \
  'http://test.cepteisler.com/purchase' \
  -H 'accept: application/json' \
  -H 'Content-Type: application/json' \
  -d '{
  "amountSum": 0,
  "branchCode": "string",
  "cashPayment": 0,
  "itemList": [
    {
      "amount": 0,
      "barcode": "string",
      "discountID": 0,
      "name": "string",
      "price": 0
    }
  ],
  "paracikPayment": 0,
  "transactionType": 0,
  "userCode": "string"
}'
```

### Not

- `discountID` hariç tüm bilgiler zorunludur!  
- `discountID` yoksa `null` gönderilebilir.

### Örnek İstek Gönderimi (Request Body)

```json
{
  "amountSum": 2233,
  "branchCode": "28",
  "cashPayment": 2233,
  "itemList": [
    {
      "amount": 1,
      "barcode": "9786057530851",
      "name": "TYT HIZ ve RENK S.B. TÜRKÇE - 2024-25",
      "price": 220,
      "discountID": 15
    },
    {
      "amount": 1,
      "barcode": "9786259793627",
      "name": "BİLGİ ARŞİVİ TYT S.B. TARİH - 2024-25",
      "price": 160,
      "discountID": null
    }
  ],
  "paracikPayment": 0,
  "transactionType": 1,
  "userCode": "723760"
}
```

### Örnek Yanıt (Response)

```json
{
  "success": true,
  "data": {
    "purchaseItemDetail": [
      "9786057530851||TYT HIZ ve RENK S.B. TÜRKÇE - 2024-25||1||220||22",
      "9786259793627||BİLGİ ARŞİVİ TYT S.B. TARİH - 2024-25||1||160||0"
    ]
  }
}
```

### Hata Durumları

- **Hatalı müşteri kodu (404):**
    ```json
    {
      "error": true,
      "message": "Girilen müşteri kodu bulunamadı"
    }
    ```

- **Müşteri kodu boş ise:**
    ```json
    {
      "error": true,
      "message": "userCode: Müşteri kodu girin."
    }
    ```

- **Hatalı bayi kodu (500):**
    ```json
    {
      "error": true,
      "message": "Girilen bayi kodu geçersiz"
    }
    ```

- **500₺ harcama limiti yetersiz ise:**
    ```json
    {
      "error": true,
      "message": "MÖüşterinin Cepte Puan harcama limiti yetersiz. Lütfen daha küçük bir tutar deneyin."
    }
    ```

- **Birden fazla hata varsa (400 veya 500 tipinde):**
    ```json
    {
      "error": true,
      "message": "itemList: Sepet boş. Lütfen ürün ekleyin.; userCode: Müşteri kodu 6 rakamdan oluşmalıdır."
    }
    ```

- **Öğrencinin bakiyesi yetersizse:**
    ```json
    {
      "error": true,
      "message": "Müşterinin Cepte Puan harcama limiti yetersiz. Lütfen daha küçük bir tutar deneyin."
    }
    ```

---

# İade Senaryosu

**Transaction type = 2** olmalıdır. İstek formatı alışveriş ile aynıdır, sadece `transactionType` değeri `2` olarak gönderilir. Bu sayede, daha önce `transactionType = 1` ile yapılan alışverişlerden kazanılan cepte puanlar iade durumunda bakiyeden düşülebilir.

**İndirim Servisinde de transactionType = 2 gönderilirse**, indirim servisinden de iade işlemine dair indirim bilgisi alınabilir. Böylece iade yapılırken ürünün indirimli alınmış bir tutarı varsa, iade tutarı da bu oranda ayarlanır.

Örneğin;  
1000 TL’lik ürünü %20 indirimle (yani 800 TL’ye) alan bir müşteri, iade ettiğinde kasa 800 TL iade tutarını dikkate alır. Bu özellik şu an test aşamasındadır, ancak gelecekte fiş veya uygulama üzerinden de kontrol sağlanacaktır.
# islercepte
