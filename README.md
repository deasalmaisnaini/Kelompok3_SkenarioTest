# Kelompok3_SkenarioTest
Repo ini merupakan skenario test dan juga pengujian yang dilakukan pada aplikasi ftgo untuk memenuhi tugas mata kuliah PPLBO

# Test Scenario
| TC ID| Name | Test Data |Response| Result | Keterangan
| ---- | ---- | ---- | ---- | ---- | ---- |
| TC01 |Membuat Order dengan Consumer terdaftar | {"consumerId": 1,"deliveryAddress": {    "city": "bandung",    "state": "jabar",    "street1": "antapani",    "street2": "jatiwangi",    "zip": "40291"  },  "deliveryTime": "2024-04-05T07:01:35.615Z",  "lineItems": [    {      "menuItemId": "1",      "quantity": 1    }  ],  "restaurantId": 1} |- | PASS | -
| TC02 |
| TC03 |
| TC04 |

# Perubahan Kode
| TC ID| Kesalahan | Letak Kesalahan |Perubahan Kode|
| ---- | ---- | ---- | ---- |
|-|Dapat melakukan order dengan kuantitas suatu menu minus | Pada Class OrderService pada method makeOrderLineItems tidak terdapat pengecekan apakah kuantitas item kurang dari 1|Menambah error handling untuk menangani kuantitas suatu menu kurang dari 1 pada method makeOrderLineItems di OrderService.java
