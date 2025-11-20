# DIAGRAM
### 1. Use Case Diagram Sistem Penjualan Tiket Desktop

```plantuml
@startuml
skinparam packageStyle rectangle
skinparam handwritten false
skinparam shadow false
skinparam usecaseBackgroundColor #F0F8FF
skinparam usecaseBorderColor #4682B4
skinparam arrowColor #2F4F4F

left to right direction

actor "Petugas Penjualan" as Sales <<User>>
actor "Petugas Check-in" as CheckIn <<User>>
actor "Admin Event" as Admin <<User>>

rectangle "Sistem Penjualan Tiket Konser (Desktop)" {
    
    package "Autentikasi" {
        usecase "Login" as UC_Login
        usecase "Logout" as UC_Logout
    }

    package "Manajemen Event" {
        usecase "Kelola Data Event" as UC_Event
        usecase "Kelola Kategori Tiket" as UC_Category
        usecase "Kelola Data User" as UC_User
    }

    package "Transaksi Penjualan" {
        usecase "Penjualan Tiket\n(Input Pembeli & Cetak)" as UC_Sell
        usecase "Lihat Riwayat Transaksi" as UC_History
        usecase "Cetak Ulang Struk" as UC_Reprint
    }

    package "Operasional Hari H" {
        usecase "Check-in Tiket\n(Validasi & Update Status)" as UC_CheckIn
    }

    package "Pelaporan" {
        usecase "Lihat Laporan Penjualan" as UC_RepSales
        usecase "Lihat Laporan Kehadiran" as UC_RepCheckIn
    }
}

' Relations
Sales --> UC_Login
CheckIn --> UC_Login
Admin --> UC_Login

Sales --> UC_Sell
Sales --> UC_History
Sales --> UC_Reprint
Sales --> UC_RepSales

CheckIn --> UC_CheckIn
CheckIn --> UC_RepCheckIn

' Admin inherits specific access or has full access
Admin --> UC_Event
Admin --> UC_Category
Admin --> UC_User
Admin --> UC_Sell
Admin --> UC_History
Admin --> UC_CheckIn
Admin --> UC_RepSales
Admin --> UC_RepCheckIn

' Include Logout
UC_Login .> UC_Logout : <<include>>

@enduml
```

---

### 2. Activity Diagram Proses Penjualan Tiket

```plantuml
@startuml
skinparam backgroundColor white
skinparam activity {
  BackgroundColor #E6F2FF
  BorderColor #336699
  DiamondBackgroundColor #FFEBCC
  DiamondBorderColor #CC6600
}

title Activity Diagram: Proses Penjualan Tiket

|Petugas Penjualan|
start
:Login ke Aplikasi;
:Pilih Menu Penjualan;
:Pilih Event & Kategori Tiket;

|Aplikasi Desktop|
:Cek Kuota Tiket Kategori;

if (Kuota Tersedia?) then (Ya)
  |Petugas Penjualan|
  :Input Data Pembeli\n(Nama, NIK, No HP);
  :Input Jumlah Tiket;
  :Klik Simpan Transaksi;
  
  |Aplikasi Desktop|
  :Hitung Total Harga;
  :Validasi Data Input;
  
  |Database|
  :Simpan Data Pembeli;
  :Simpan Header Transaksi;
  :Generate Kode Booking Unik\n(Loop per tiket);
  :Simpan Detail Tiket;
  :Update/Kurangi Kuota Kategori;
  
  |Aplikasi Desktop|
  :Tampilkan Sukses & Ringkasan;
  
  |Petugas Penjualan|
  :Konfirmasi Cetak Bukti;
  
  |Aplikasi Desktop|
  :Generate Struk Transaksi;
  :Kirim ke Printer;
  stop
else (Tidak)
  |Aplikasi Desktop|
  :Tampilkan Pesan "Kuota Habis";
  stop
endif

@enduml
```

---

### 3. Activity Diagram Proses Check-in Tiket

```plantuml
@startuml
skinparam backgroundColor white
skinparam activity {
  BackgroundColor #E6FFE6
  BorderColor #339933
  DiamondBackgroundColor #FFEBCC
  DiamondBorderColor #CC6600
}

title Activity Diagram: Proses Check-in Tiket

|Petugas Check-in|
start
:Buka Menu Check-in;
:Input Kode Booking / Scan Barcode\n(atau Cari via Nama/NIK);
:Klik Validasi;

|Aplikasi Desktop|
:Query Data Tiket ke DB;

|Database|
:Select Ticket By Criteria;

|Aplikasi Desktop|
if (Tiket Ditemukan?) then (Ya)
    :Cek Status Tiket;
    if (Status == TERJUAL?) then (Ya)
        |Petugas Check-in|
        :Konfirmasi Check-in;
        
        |Aplikasi Desktop|
        :Set Status = CHECKIN;
        :Catat Waktu & Petugas;
        
        |Database|
        :Update Status Tiket;
        :Insert Checkin Log;
        
        |Aplikasi Desktop|
        :Tampilkan "Check-in BERHASIL"\n(Hijau);
    else (Sudah Check-in)
        :Tampilkan "GAGAL: Tiket Sudah Digunakan"\n(Merah);
    endif
else (Tidak)
    :Tampilkan "Data Tidak Ditemukan";
endif

stop
@enduml
```

---

### 4. Activity Diagram Proses Kelola Event & Kategori

```plantuml
@startuml
skinparam backgroundColor white
skinparam activity {
  BackgroundColor #FFF0F5
  BorderColor #C71585
}

title Activity Diagram: Kelola Event & Kategori

|Admin Event|
start
:Login sebagai Admin;
:Pilih Menu Manajemen Event;

fork
    :Tambah Event Baru;
    :Input Nama, Tanggal, Lokasi;
    :Set Status (DRAFT);
fork again
    :Edit Event Existing;
end fork

|Database|
:Simpan/Update Data Event;

|Admin Event|
:Pilih Event di Tabel;
:Masuk Menu Kategori Tiket;
:Tambah/Edit Kategori;
:Set Nama Kategori (VIP, Reguler);
:Set Kuota Total & Harga;

|Database|
:Simpan Data Kategori;

|Admin Event|
:Verifikasi Data;
:Ubah Status Event -> AKTIF;

|Database|
:Update Status Event;

stop
@enduml
```

---

### 5. Sequence Diagram Proses Penjualan Tiket

Diagram teknis yang menunjukkan interaksi antar objek (UI, Service, DAO).

```plantuml
@startuml
skinparam sequence {
    ActorBorderColor black
    LifeLineBorderColor #333333
    ParticipantBorderColor #333333
    ParticipantBackgroundColor #DDEEFF
}

actor "Petugas Penjualan" as User
participant "FormPenjualan" as UI
participant "PenjualanService" as Service
participant "TicketCategoryDAO" as CatDAO
participant "TransactionDAO" as TransDAO
participant "TicketDAO" as TicketDAO
database "MySQL DB" as DB

User -> UI : Pilih Event & Kategori
UI -> Service : getQuota(categoryId)
Service -> CatDAO : selectQuota()
CatDAO -> DB : query
DB --> CatDAO : resultSet
CatDAO --> Service : int quota
Service --> UI : return quota

User -> UI : Input Data Pembeli & Jml Tiket
User -> UI : Klik Simpan

UI -> Service : prosesPenjualan(dto)
activate Service

Service -> CatDAO : checkAvailability(qty)
alt Kuota Cukup
    Service -> TransDAO : createTransaction(header)
    TransDAO -> DB : insert transaction
    
    loop for i=1 to qty
        Service -> Service : generateBookingCode()
        Service -> TicketDAO : createTicket(ticketData)
        TicketDAO -> DB : insert ticket
    end
    
    Service -> CatDAO : updateQuota(-qty)
    CatDAO -> DB : update ticket_categories
    
    Service --> UI : Transaksi Sukses (List KodeBooking)
    UI -> User : Tampilkan Pesan Sukses
    UI -> User : Cetak Struk
else Kuota Kurang
    Service --> UI : Throw Exception (Kuota Habis)
    UI -> User : Tampilkan Error
end

deactivate Service

@enduml
```

---

### 6. Sequence Diagram Proses Check-in Tiket

```plantuml
@startuml
skinparam sequence {
    ActorBorderColor black
    ParticipantBackgroundColor #E6FFE6
}

actor "Petugas Check-in" as User
participant "FormCheckIn" as UI
participant "CheckInService" as Service
participant "TicketDAO" as TicketDAO
participant "CheckInLogDAO" as LogDAO
database "MySQL DB" as DB

User -> UI : Input Kode Booking
User -> UI : Klik Cari

UI -> Service : findTicket(kode)
activate Service
Service -> TicketDAO : getTicketByCode(kode)
TicketDAO -> DB : select * form tickets
DB --> TicketDAO : result
TicketDAO --> Service : TicketModel
Service --> UI : Ticket Details
deactivate Service

UI -> User : Tampilkan Detail (Event, Nama, Status)

opt Status == TERJUAL
    User -> UI : Klik Validasi Check-in
    UI -> Service : processCheckIn(ticketId, userId)
    activate Service
    
    Service -> TicketDAO : updateStatus(ticketId, "CHECKIN")
    TicketDAO -> DB : update tickets
    
    Service -> LogDAO : insertLog(ticketId, timestamp)
    LogDAO -> DB : insert checkin_logs
    
    Service --> UI : Success
    deactivate Service
    
    UI -> User : Show "Check-in OK"
else Status == CHECKIN
    UI -> User : Show "Warning: Already Used"
end

@enduml
```

---

### 7. ERD (Entity Relationship Diagram) Lengkap

ERD ini menunjukkan struktur database MySQL dan kardinalitasnya.

```plantuml
@startuml
skinparam linetype ortho
skinparam classAttributeIconSize 0
skinparam classBackgroundColor #FFF8DC
skinparam classBorderColor #8B4513

entity "events" as events {
  *event_id : INT <<PK>>
  --
  nama : VARCHAR(100)
  tanggal : DATE
  lokasi : VARCHAR(150)
  status : VARCHAR(20)
}

entity "ticket_categories" as categories {
  *category_id : INT <<PK>>
  --
  *event_id : INT <<FK>>
  nama_kategori : VARCHAR(50)
  kuota_total : INT
  kuota_terjual : INT
  harga : DECIMAL
}

entity "users" as users {
  *user_id : INT <<PK>>
  --
  username : VARCHAR(50)
  password_hash : VARCHAR(255)
  role : VARCHAR(20)
}

entity "buyers" as buyers {
  *buyer_id : INT <<PK>>
  --
  nama : VARCHAR(100)
  nomor_identitas : VARCHAR(50)
  nomor_hp : VARCHAR(20)
}

entity "transactions" as transactions {
  *transaction_id : INT <<PK>>
  --
  kode_transaksi : VARCHAR(50)
  *buyer_id : INT <<FK>>
  *event_id : INT <<FK>>
  *user_id_petugas : INT <<FK>>
  total_harga : DECIMAL
  waktu_transaksi : DATETIME
}

entity "tickets" as tickets {
  *ticket_id : INT <<PK>>
  --
  *event_id : INT <<FK>>
  *category_id : INT <<FK>>
  *buyer_id : INT <<FK>>
  *transaction_id : INT <<FK>>
  kode_booking : VARCHAR(50) <<Unique>>
  status : VARCHAR(20)
  waktu_penjualan : DATETIME
}

entity "checkin_logs" as logs {
  *checkin_id : INT <<PK>>
  --
  *ticket_id : INT <<FK>>
  *user_id_petugas : INT <<FK>>
  waktu_checkin : DATETIME
}

' Relationships
events ||..|{ categories : "memiliki"
events ||..|{ tickets : "terkait"
events ||..|{ transactions : "bagian dari"

categories ||..|{ tickets : "mendefinisikan"

buyers ||..|{ transactions : "melakukan"
buyers ||..|{ tickets : "memiliki"

users ||..|{ transactions : "memproses"
users ||..|{ logs : "memvalidasi"

transactions ||..|{ tickets : "berisi"

tickets ||..|{ logs : "dicatat"

@enduml
```

---

### 8. Diagram Navigasi / Hierarki Menu

```plantuml
@startuml
skinparam rectangle {
    BackgroundColor #EEE8AA
    BorderColor #DAA520
    RoundCorner 15
}

rectangle "Login Screen" as Login
rectangle "Main Dashboard" as Dash

rectangle "Menu Event" as M_Event
rectangle "  - Data Event  " as S_EventData
rectangle "  - Kategori    " as S_EventCat

rectangle "Menu Penjualan" as M_Sales
rectangle "  - Transaksi Baru " as S_SalesNew
rectangle "  - Riwayat        " as S_SalesHist

rectangle "Menu Check-in" as M_Check
rectangle "  - Validasi Tiket " as S_CheckVal

rectangle "Menu Laporan" as M_Report
rectangle "  - Lap. Penjualan " as S_RepSales
rectangle "  - Lap. Check-in  " as S_RepCheck

rectangle "Menu User" as M_User
rectangle "  - Kelola Admin   " as S_UserMan

rectangle "Logout" as Logout

Login --> Dash : Valid Credentials

Dash --> M_Event
M_Event --> S_EventData
M_Event --> S_EventCat

Dash --> M_Sales
M_Sales --> S_SalesNew
M_Sales --> S_SalesHist

Dash --> M_Check
M_Check --> S_CheckVal

Dash --> M_Report
M_Report --> S_RepSales
M_Report --> S_RepCheck

Dash --> M_User : (Admin Only)
M_User --> S_UserMan

Dash --> Logout
@enduml
```

---

### 9. Wireframes (Antarmuka Aplikasi)

#### 9.1 Wireframe Login

```plantuml
@startsalt
{+
  ^<b>Login Sistem Tiket Konser</b>^
  .
  Username: | "admin_event     "
  Password: | "************    "
  .
  [  Masuk  ] | [  Keluar  ]
  .
  Status: Siap.
}
@endsalt
```

#### 9.2 Wireframe Dashboard Utama

```plantuml
@startsalt
{+
  <b>Halo, Budi (Admin Event)</b> | [Logout]
  ==
  {/ <b>Dashboard</b> | Event | Penjualan | Check-in | Laporan | User }
  
  [ Icon Event ]    [ Icon Jual ]    [ Icon Checkin ]
  
  == Ringkasan ==
  Event Aktif : 3
  Tiket Terjual Hari Ini : 150
  
  == Notifikasi ==
  * Event "Jazz Festival" kuota VIP menipis.
}
@endsalt
```

#### 9.3 Wireframe Manajemen Event

```plantuml
@startsalt
{+
  <b>Manajemen Event</b>
  --
  {+
    Nama Event : | "Konser Amal 2025      "
    Tanggal    : | "2025-10-20            "
    Lokasi     : | "Stadion Madya         "
    Status     : | ^DRAFT^
    [ Simpan ] | [ Batal ] | [ Hapus ]
  }
  --
  {.
  <b>Nama Event</b> | <b>Tanggal</b> | <b>Status</b>
  Konser Amal 2025 | 2025-10-20 | DRAFT
  Indie Night v2   | 2025-11-15 | AKTIF
  }
}
@endsalt
```

#### 9.4 Wireframe Manajemen Kategori Tiket

```plantuml
@startsalt
{+
  <b>Manajemen Kategori Tiket</b>
  Pilih Event: ^Konser Amal 2025^
  --
  {
    Nama Kategori: | "VIP           "
    Kuota Total  : | "100           "
    Harga (IDR)  : | "1000000       "
    [ Tambah/Update ] | [ Hapus ]
  }
  --
  {.
    <b>Kategori</b> | <b>Kuota</b> | <b>Terjual</b> | <b>Harga</b>
    VIP | 100 | 0 | 1.000.000
    Reguler A | 500 | 0 | 500.000
    Reguler B | 500 | 0 | 350.000
  }
}
@endsalt
```

#### 9.5 Wireframe Manajemen User

```plantuml
@startsalt
{+
  <b>Kelola Data User</b>
  --
  Username : | "petugas1        "
  Password : | "*******         "
  Role     : | ^PENJUAL^
  [ Simpan ] | [ Hapus ]
  --
  {.
    <b>Username</b> | <b>Role</b>
    admin | ADMIN
    petugas1 | PENJUAL
    gatekeeper | CHECKIN
  }
}
@endsalt
```

#### 9.6 Wireframe Penjualan Tiket

```plantuml
@startsalt
{+
  <b>Transaksi Penjualan Tiket</b>
  --
  <b>1. Pilih Tiket</b>
  Event    : ^Konser Amal 2025^
  Kategori : ^Reguler A (Sisa: 450)^
  Harga    : Rp 500.000
  Jml Tiket: "2   "
  --
  <b>2. Data Pembeli</b>
  Nama Lengkap : | "Ahmad Yani        "
  No. Identitas: | "3171234567890001  "
  No. HP       : | "08123456789       "
  --
  <b>Total: Rp 1.000.000</b>
  [ PROSES TRANSAKSI ] [ Reset ]
}
@endsalt
```

#### 9.7 Wireframe Riwayat Transaksi

```plantuml
@startsalt
{+
  <b>Riwayat Transaksi</b>
  Cari: | "Ahmad Yani      " | [ Cari ]
  Event: ^Semua Event^
  --
  {.
    <b>Tgl</b> | <b>Kode Trx</b> | <b>Pembeli</b> | <b>Event</b> | <b>Total</b> | <b>Petugas</b>
    20/10 | TRX001 | Ahmad Yani | Konser Amal | 1.000.000 | Budi
    20/10 | TRX002 | Siti Aminah| Konser Amal | 500.000 | Budi
  }
  [ Cetak Ulang Struk ]
}
@endsalt
```

#### 9.8 Wireframe Check-in Tiket

```plantuml
@startsalt
{+
  <b>Check-in Gate System</b>
  --
  Masukkan Kode Booking / Scan:
  | "CA-2025-REG-0091   " | [ CARI TIKET ]
  --
  <b>Detail Tiket:</b>
  Nama Event : Konser Amal 2025
  Nama Peserta : Ahmad Yani
  Kategori : Reguler A
  Status : <b><color:green>TERJUAL</color></b>
  --
  [ VALIDASI MASUK (CHECK-IN) ]
  
  .
  "Log: Tiket valid. Check-in sukses pada 16:30."
}
@endsalt
```

#### 9.9 Layout Laporan Penjualan

```plantuml
@startsalt
{+
  <b>Laporan Penjualan</b>
  Event: ^Konser Amal 2025^
  Periode: "01-10-2025" s/d "31-10-2025"
  [ Tampilkan ] [ Export PDF ]
  --
  {.
    <b>No</b> | <b>Tanggal</b> | <b>Kategori</b> | <b>Qty</b> | <b>Total IDR</b>
    1 | 2025-10-20 | VIP | 10 | 10.000.000
    2 | 2025-10-20 | Reguler A | 50 | 25.000.000
    . | . | . | . | .
    = | <b>TOTAL</b> | <b>ALL</b> | <b>60</b> | <b>35.000.000</b>
  }
}
@endsalt
```

#### 9.10 Layout Laporan Check-in

```plantuml
@startsalt
{+
  <b>Laporan Kehadiran (Check-in)</b>
  Event: ^Konser Amal 2025^
  [ Tampilkan ]
  --
  Ringkasan: 450 / 500 Hadir (90%)
  --
  {.
    <b>Waktu Masuk</b> | <b>Kode Booking</b> | <b>Nama</b> | <b>Kategori</b> | <b>Gate Keeper</b>
    16:01 | CA-REG-001 | Budi | Reg A | Petugas2
    16:05 | CA-VIP-002 | Ani | VIP | Petugas2
  }
}
@endsalt
```

#### 9.11 Layout Struk / Bukti Transaksi

```plantuml
@startsalt
{+
  .
  ^      <b>BUKTI PEMBELIAN TIKET</b>   ^
  ^      ASTRINELIS EVENT HELPER V67    ^
  .
  No. Trx : TRX-20251020-001
  Waktu   : 20/10/2025 14:30
  Petugas : Budi
  .
  Event   : KONSER AMAL 2025
  Lokasi  : Stadion Madya
  Hari/Tgl: Sabtu, 15 Nov 2025
  .
  -------------------------------------
  <b>DATA PEMBELI</b>
  Nama    : Ahmad Yani
  ID      : 3171...0001
  .
  -------------------------------------
  <b>DETAIL TIKET</b>
  Kategori : Reguler A
  Harga    : Rp 500.000
  Qty      : 2
  .
  <b>KODE BOOKING:</b>
  1. CA-2025-REG-8821
  2. CA-2025-REG-8822
  .
  -------------------------------------
  <b>TOTAL BAYAR : Rp 1.000.000</b>
  -------------------------------------
  .
  ^  Simpan struk ini sebagai bukti.  ^
  ^    Tunjukkan saat check-in.       ^
  .
}
@endsalt
```
