## Onboarding

### 1. Dry Run / Create Token

```php
/** 
 * Uji coba echo Token yang sesuai dan di DB akan tersimpan
 * Pastikan sudah mengisi konfigurasi di .env
*/

$client = new Satusehat\Integration\OAuth2Client;
echo $client->token(); // OAuth2Token anda akan muncul
```

Selamat, anda berhasil menyelesaikan langkah pertama akses platform SATUSEHAT!