# 1. Manually Create Token

```php
/** 
 * Uji coba echo Token yang sesuai dan di DB akan tersimpan
 * Pastikan sudah mengisi konfigurasi di .env
*/

$client = new Satusehat\Integration\OAuth2Client;
echo $client->token(); // OAuth2Token anda akan muncul
```

# 2. KYC (Verifikasi SATUSEHAT Centang Biru)
*Note : Wajib dilakukan pada konfigurasi .env ```SATUEHAT_ENV=PROD```

```php
/** 
 * Pastikan sudah mengisi konfigurasi di .env
 * Proses KYC tidak perlu lagi menggunakan deklarasi OAuth2Client->token()
*/
<?php

use Satusehat\Integration\KYC;

$kyc = new KYC;

// Isi nama verifikator & NIK verifikator untuk mendapatkan link KYC
$json = $kyc->generateUrl('{nama_verifikator}', '{nik_verifikator}');
$kyc_link = json_decode($json, true);

/** 
 * Melakukan route redirect ke link KYC
 * saat ini hanya bisa dibuka pada tab baru / pop-up
 * tidak bisa melalui iframe
*/
return redirect($kyc_link['data']['url']);
```

# 3. FHIR Resource Agnostic Operation
- Fungsi dasar GET, POST, PUT sudah ditambahkan pada kelas OAuth2Client.
- Proses GET / POST / PUT, tidak perlu lagi menggunakan deklarasi ```OAuth2Client->token()```
- Hasil balikan dari GET adalah kode status HTTP dan response body (JSON)

## Contoh GET by ID

```php
/** 
 * Hasil balikan dari GET / POST / PUT adalah kode status HTTP dan response body (JSON)
*/
<?php

use Satusehat\Integration\OAuth2Client;

$client = new OAuth2Client;

// Organization
[$statusCode, $response] = $client->get('Organization', '{id}');

// Location
[$statusCode, $response] = $client->get('Location', '{id}');

// Patient
[$statusCode, $response] = $client->get('Patient', '{id}');

// Practitioner
[$statusCode, $response] = $client->get('Practitioner', '{id}');

// Encounter
[$statusCode, $response] = $client->get('Encounter', '{id}');

// Condition
[$statusCode, $response] = $client->get('Condition', '{id}');
```

## GET by NIK

```php
// Patient
[$statusCode, $response] = $client->get_by_nik('Patient', '{NIK Pasien}');

// Practitioner
[$statusCode, $response] = $client->get_by_nik('Practitioner', '{NIK Dokter}');
```

## Agnostic POST & PUT : using self build JSON object

```php
$client = new OAuth2Client;
$body = ...... ; // JSON Object
$resource = ......; // FHIR Resource (Organization, Location, Patient, Practitioner, Encounter, Condition)

// Format Agnostic POST
[$statusCode, $response] = $client->ss_post($resource, $body);
echo $statusCode, $response;

// Format Agnostic PUT
$id = ...... ; // SATUSEHAT response ID
[$statusCode, $response] = $client->ss_put($resource, $id, $body);
echo $statusCode, $response;
```

# 4. Menggunakan FHIR Class Object
Untuk mempermudah pembuatan FHIR Object data type, dapat menggunakan class object yang sudah disediakan.<br>
Pertama-tama import class dari FHIR Class Object :
```php
<?php

use Satusehat\Integration\FHIR\Organization;
use Satusehat\Integration\FHIR\Location;
use Satusehat\Integration\FHIR\Encounter;
use Satusehat\Integration\FHIR\Condition;
```

## Organization
```php
// Organization
$organization = new Organization;
$organization->addIdentifier('{kode_unik_suborganisasi}'); // unique string free text (increments / UUID / inisial)
$organization->setName('{nama_suborganisasi}'); // string free text
dd($organization->json());
```

## Location
```php
// Location
$location = new Location;
$location->addIdentifier('{kode_unik_lokasi}'); // unique string free text (increments / UUID / inisial)
$location->setName('{nama_lokasi}'); // string free text
$location->addPhysicalType('{tipe_lokasi}'); // ro = ruangan, bu = bangunan, wi = sayap gedung, ve = kendaraan, ho = rumah, ca = kabined, rd = jalan, area = area. Default bila tidak dideklarasikan = ruangan
dd($location->json());
```

## Encounter
```php
// Encounter
$encounter = new Encounter;

/** 
 * timestamp_kedatangan, timestamp_pemeriksaan, timestamp_pulang = format Y-m-dTH:i:s+UTC
 * Contoh : 31 Desember 2022 15:45 WIB : 2022-12-31T15:45:00+07:00
*/
$statusHistory = ['arrived' => '{timestamp_kedatangan}', 
                    'in-progress' => '{timestamp_pemeriksaan}', 
                    'finished' => '{timestamp_pulang}']; 

$encounter->addRegistrationId('{kode_registrasi}'); // unique string free text (increments / UUID)
$encounter->addStatusHistory($statusHistory); // array of timestamp
$encounter->setConsultationMethod('{metode_konsultasi}'); // RAJAL, IGD, RANAP, HOMECARE, TELEKONSULTASI
$encounter->setSubject('{id_patient}', '{nama_pasien}'); // ID SATUSEHAT Pasien dan Nama SATUSEHAT
$encounter->addParticipant('{id_practitioner}', '{nama_dokter}'); // ID SATUSEHAT Dokter, Nama Dokter
$encounter->addLocation('{id_location}', '{nama_poli}'); // ID SATUSEHAT Location, Nama Poli
$encounter->addDiagnosis('{id_condition}', '{kode_icd_10}'); // ID SATUSEHAT Condition, Kode ICD10
$dd($encounter->json());
```

## Condition
```php
// Condition
$condition = new Condition;
$condition->addClinicalStatus('{status_klinis}'); // active, inactive, resolved. Default bila tidak dideklarasi = active
$condition->addCategory('{kategori}'); // Diagnosis, Keluhan. Default : diagnosis
$condition->addCode('{kode_icd_10}'); // Kode ICD10
$condition->setSubject('{id_patient}', '{nama_pasien}'); // ID SATUSEHAT Pasien dan Nama SATUSEHAT
$condition->setEncounter('{id_encounter}'); // ID SATUSEHAT Encounter
$condition->setOnsetDateTime('{timestamp_onset}'); // timestamp onset. Timestamp sekarang
$condition->setRecordedDate('{timestamp_recorded}'); // timestamp recorded. Timestamp sekarang
$condition->json();

```

# 5. POST / PUT menggunakan FHIR Class Object
Proses GET / POST / PUT melalui FHIR class Object, tidak perlu lagi menggunakan deklarasi kelas OAuth2Client.<br>
Hasil balikan dari POST & PUT adalah kode status HTTP dan response body (JSON)

```php
// Organization
[$statusCode, $response] = $organization->post();
[$statusCode, $response] = $organization->put('{organization_id}');

// Location
[$statusCode, $response] = $location->post();
[$statusCode, $response] = $location->put('{location_id}');

// Encounter
[$statusCode, $response] = $encounter->post();
[$statusCode, $response] = $encounter->put('{encounter_id}');

// Condition
[$statusCode, $response] = $condition->post();
[$statusCode, $response] = $condition->put('{condition_id}');
```