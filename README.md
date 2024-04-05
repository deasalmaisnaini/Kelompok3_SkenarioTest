# Kelompok3_SkenarioTest
Repo ini merupakan skenario test dan juga pengujian yang dilakukan pada aplikasi ftgo untuk memenuhi tugas mata kuliah PPLBO

# Test Scenario
| TC ID|Name| Precondition | Test Data |Response| Result | Keterangan
| ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| TC01 |Membuat Order dengan Consumer terdaftar |Sudah ada restoran yang tersimpan, Sudah ada menu pada restoran terpilih yang tersimpan, Consumer sudah terdaftar| {"consumerId": 1,"deliveryAddress": {    "city": "bandung",    "state": "jabar",    "street1": "antapani",    "street2": "jatiwangi",    "zip": "40291"  },  "deliveryTime": "2024-04-05T07:01:35.615Z",  "lineItems": [    {      "menuItemId": "1",      "quantity": 1    }  ],  "restaurantId": 1} |{  "orderId": 1} | PASS | -
| TC02 |Membuat Order dengan Consumer tidak terdaftar |Sudah ada restoran yang tersimpan, Sudah ada menu pada restoran terpilih yang tersimpan| {  "consumerId": 8,  "deliveryAddress": {    "city": "cimahi",    "state": "jabar",    "street1": "antapani",    "street2": "jatiwangi",    "zip": "40291"  },  "deliveryTime": "2024-04-05T17:01:35.615Z",  "lineItems": [    {      "menuItemId": "2",      "quantity": 50    }  ],  "restaurantId": 1} | create Order:<br>{  "orderId": 6,  "state": "REJECTED",  "orderTotal": "800000.00"}<br><br>get Order:<br>{  "orderId": 6,  "state": "REJECTED",  "orderTotal": "800000.00"}|PASS|-
| TC03 |Membuat Order dengan Restaurant terdaftar|Sudah ada restoran yang tersimpan, Sudah ada menu pada restoran terpilih yang tersimpan, Consumer sudah terdaftar|{  "consumerId": 1,  "deliveryAddress": {    "city": "Cimahi",    "state": "Jawa Barat",    "street1": "Cipageran",    "street2": "Cimahi Utara",  },  "deliveryTime": "2024-04-05T08:36:16.858Z",  "lineItems": [    {      "menuItemId": "1",      "quantity": 1    }  ],  "restaurantId": 2}|{  "timestamp": "2024-04-05T09:07:55.338+0000",  "status": 500,  "error": "Internal Server Error",  "message": "Restaurant not found with id 2",  "path": "/orders"}|PASS|-
| TC04 |Membuat Order dengan Menu tidak terdaftar|Sudah ada restoran yang tersimpan, Sudah ada menu pada restoran terpilih yang tersimpan, Consumer sudah terdaftar|{  "consumerId": 1,  "deliveryAddress": {    "city": "cimahi",    "state": "jabar",    "street1": "antapani",    "street2": "jatiwangi",    "zip": "40291"  },  "deliveryTime": "2024-04-05T17:01:35.615Z",  "lineItems": [    {      "menuItemId": "3",      "quantity": 50    }  ],  "restaurantId": 1}|{  "timestamp": "2024-04-05T09:03:38.056+0000",  "status": 500,  "error": "Internal Server Error",  "message": "Invalid menu item id 3",  "path": "/orders"}|PASS|-
|TC05|Membuat Order tanpa memilih Menu apapun |Sudah ada restoran yang tersimpan, Sudah ada menu pada restoran terpilih yang tersimpan, Consumer sudah terdaftar| {  "consumerId": 1,  "deliveryAddress": {    "city": "cimahi",    "state": "jabar",    "street1": "antapani",    "street2": "jatiwangi",    "zip": "40291"  },  "deliveryTime": "2024-04-05T17:01:35.615Z",  "lineItems": [  ],  "restaurantId": 1}|{  "orderId": 8}|FAIL|-
|TC06|Memesan Menu dengan jumlah kurang dari 1|Sudah ada restoran yang tersimpan, Sudah ada menu pada restoran terpilih yang tersimpan, Consumer sudah terdaftar|{  "consumerId": 1,  "deliveryAddress": {    "city": "cimahi",    "state": "jabar",    "street1": "antapani",    "street2": "jatiwangi",    "zip": "40291"  },  "deliveryTime": "2024-04-05T17:01:35.615Z",  "lineItems": [    {      "menuItemId": "1",      "quantity": -50    }  ],  "restaurantId": 1}|{  "orderId": 9}|FAIL|-
|TC07|Mendapatkan Order yang sudah terdaftar |Sudah ada Order yang tersimpan| orderId = 1|{  "orderId": 1,  "state": "APPROVED",  "orderTotal": "15000.00"}|PASS|-
|TC08|Mendapatkan Order yang tidak terdaftar |-| orderId = 100|-|PASS|-
|TC09|Mengubah order dengan orderId dan id menu yang terdaftar pada sistem|Sudah Membuat order | orderId = 1 {"revisedOrderLineItems": [{"menuItemId": "2","quantity": 10}]}| {"orderId" : 3, "state" : "APPROVAL_PENDING","orderTotal":"300000"}|PASS| karena statusnya Approval_Pending sehinggga orderTotal masih mengandung total dari pesanan sebelumnya| 


# Perubahan Kode
| TC ID| Kesalahan | Letak Kesalahan |Perubahan Kode|
| ---- | ---- | ---- | ---- |
|TC05|Dapat melakukan order walau tidak memesan menu apapun | Tidak terdapat pengecekan apakah lineItems kosong atau tidak ketika membuat Order|Menambah error handling untuk menangani isi line items kosong pada method CreateOrder class OrderService
```java
  @Transactional
  public Order createOrder(long consumerId, long restaurantId, DeliveryInformation deliveryInformation,
                           List<MenuItemIdAndQuantity> lineItems) {
    if (lineItems.isEmpty()) {
        throw new IllegalArgumentException("List of line items cannot be empty");
    }
    
    Restaurant restaurant = restaurantRepository.findById(restaurantId)
            .orElseThrow(() -> new RestaurantNotFoundException(restaurantId));

    List<OrderLineItem> orderLineItems = makeOrderLineItems(lineItems, restaurant);

    ResultWithDomainEvents<Order, OrderDomainEvent> orderAndEvents =
            Order.createOrder(consumerId, restaurant, deliveryInformation, orderLineItems);

    Order order = orderAndEvents.result;
    orderRepository.save(order);

    orderAggregateEventPublisher.publish(order, orderAndEvents.events);
    OrderDetails orderDetails = new OrderDetails(consumerId, restaurantId, orderLineItems, order.getOrderTotal());
    CreateOrderSagaState data = new CreateOrderSagaState(order.getId(), orderDetails);
    sagaInstanceFactory.create(createOrderSaga, data);

    meterRegistry.ifPresent(mr -> mr.counter("placed_orders").increment());

    return order;
  }
```

| |  |  |  |
| ---- | ---- | ---- | ---- |
|TC06|Dapat melakukan order dengan kuantitas suatu menu minus | Pada Class OrderService pada method makeOrderLineItems tidak terdapat pengecekan apakah kuantitas item kurang dari 1|Menambah error handling untuk menangani kuantitas suatu menu kurang dari 1 pada method makeOrderLineItems di OrderService.java

```java
  private List<OrderLineItem> makeOrderLineItems(List<MenuItemIdAndQuantity> lineItems, Restaurant restaurant) {
    return lineItems.stream().map(li -> {
      if (li.getQuantity() < 1) {
        throw new IllegalArgumentException("Invalid quantity for menu with ID: " + li.getMenuItemId());
      }

      MenuItem om = restaurant.findMenuItem(li.getMenuItemId()).orElseThrow(() -> new InvalidMenuItemIdException(li.getMenuItemId()));
      return new OrderLineItem(li.getMenuItemId(), om.getName(), om.getPrice(), li.getQuantity());
    }).collect(toList());
  }
