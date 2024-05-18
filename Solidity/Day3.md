# SOLIDITY DAY 3

Ở bài này thì chúng ta tạo 1 app book hotel. App này có tính năng:

- Book phòng
- Thanh toán ( Nhận và chuyển tiền ether )
- Kiểm tra xem phòng có người book chưa
- Log thông tin 1 giao dịch

Ở bài này chúng ta cần chuẩn bị:

- Remix Ethereum IDE để test Smart Contract
- 2 tài khoản ethererum test
  - Khi deploy Smart Contract contructor sẽ lấy tài khoản hiện tại để làm owner ( Chủ khách sạn )
  - Khi Booking mình sẽ chuyển sang tài khoản khác để chuyển tiền về owner

OK chiến nào, khởi tạo 1 contract có tên là HotelRoom:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract HotelRoom {
    // Vacant: Còn trống | value = 0
    // Occupied: Đã chiếm chỗ | value = 1

    enum Statuses {
        Vacant,
        Occupied
    }
    Statuses public currentStatus;
    // Khởi tạo 1 biến chưa status hiện tại của khách sạn

    address payable public owner;
    // Khởi tạo 1 biến chứa address của chủ khách sạn

    constructor() {
        owner = payable (msg.sender);
        // Gán Address hiện tại
        currentStatus = Statuses.Vacant;
    }
}
```

### Lưu ý:

Thuộc tính payalbe trong `address payable public owner;` cho phép address này có thể nhận Ether

---

## Chức năng Book phòng

Ok sau khi thiết lập được ai đã là chủ khách sạn, chúng ta tiến hành làm chức năng booking !

Flow code của chúng ta sẽ là:

```
- Nếu bạn chuyển đủ số tiền, tui cho bạn đặt phòng
  - Nếu hotel còn phòng, tui cho bạn đặt
    - Khách đã đặt phòng thì chuyển status về Occupied
    - Owner nhận tiền từ khách hàng
  - Nếu không quăng ra lỗi "Currently occupied."
- Nếu không quăng ra lỗi "Not enough ether provided."
```

Để phục cho chuyện kiểm tra ngoài If Else thì chúng ta còn có thằng require: `require(condition, "error_message")`:

- Nếu điều kiện đúng thì code đi tiếp.
- Nếu điều kiện sai thì code ko đi tiếp, quăng ra lỗi !

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract HotelRoom {
    .
    .
    .
    function book () payable public {
        // Check price
        require(msg.value >= 2 ether, "Not enough ether provided.");
        // Check status
        require(currentStatus == Statuses.Vacant, "Currently occupied.");

        currentStatus = Statuses.Occupied;

        owner.transfer(msg.value);
    }
}
```

### Lưu ý:

- Chúng ta phải để payable cho function để thông báo rằng, hàm này chúng ta dùng để nhận tiền đó !

---

## Modifier

Nếu viết như vầy thì cũng OK rồi, nhưng mà nếu chúng ta muốn tái sử dụng các logic như check tiền hoặc là check phòng còn trống hay không thì chúng ta phải làm sao ?
=> Chúng ta có 1 khái niệm đó là modifier, nếu bạn nào đã học backend thì nó tương đương với middleware !

Tiến hành viết 1 modifier nào:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract HotelRoom {
    .
    .
    modifier onlyWhileVacant {
        // Check status
        require(currentStatus == Statuses.Vacant, "Currently occupied.");
        _;
    }

    modifier costs(uint _amount){
        // Check price
        require(msg.value >= _amount, "Not enough ether provided.");
        _;
    }

    function book () payable public onlyWhileVacant costs(2 ether){
        currentStatus = Statuses.Occupied;
        owner.transfer(msg.value);
    }
}
```

Đơn giản chỉ là tạo 1 modifier và copy các logic bỏ vào bên trong !

### Lưu ý

- Ở cuối 1 modifier nhất định phải có \_;
- Nó tượng trưng cho vị trí của những đoạn code phía sau !
  - Ví dụ:
  ```
  require(currentStatus == Statuses.Vacant, "Currently occupied.");
  _;
  ```
  Thì bây giờ \_ sẽ tương trưng ( thế chỗ ) cho
  ```
  currentStatus = Statuses.Occupied;
  owner.transfer(msg.value);
  ```

---

## Tối Ưu Code:

Ở cách trên thì chúng ta dùng transfer để chuyển tiền. Thì chúng ta dùng call thì ngon cơm hơn !

- Nó trả ra kết quả khi chuyển tiền. Ta có thể dùng nó để làm điều kiện

```solidity
// SPDX-License-Identifier: MIT
pragma solidity  ^0.8.0;

contract HotelRoom {
    .
    .
    .
    function book () payable public onlyWhileVacant costs(2 ether){

        currentStatus = Statuses.Occupied;

        (bool sent, bytes memory data) = owner.call{value: msg.value}("");
        require(sent, "Fail to send Ether");
    }
}
```

Ngoài ra chúng ta có thể log một sự kiện khi có ai đó Book phòng chẳng hạn

```solidity
// SPDX-License-Identifier: MIT
pragma solidity  ^0.8.0;

contract HotelRoom {
    .
    .
    event Occupy(address _occupant, uint _value);
    .
    .
    .
    function book () payable public onlyWhileVacant costs(2 ether){

        currentStatus = Statuses.Occupied;

        (bool sent, bytes memory data) = owner.call{value: msg.value}("");
        require(sent);

        emit Occupy(msg.sender, msg.value);
    }
}

```

---

## Tổng quát 1 bài chúng ta sẽ có:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity  ^0.8.0;

contract HotelRoom {
    enum Statuses {
        Vacant,
        Occupied
    }
    Statuses public currentStatus;

    address payable public owner;
    // payable: let this address receive token

    event Occupy(address _occupant, uint _value);

    constructor() {
        owner = payable (msg.sender);
        currentStatus = Statuses.Vacant;
    }

    modifier onlyWhileVacant {
        // Check status
        require(currentStatus == Statuses.Vacant, "Currently occupied.");
        _;
    }

    modifier costs(uint _amount){
        // Check price
        require(msg.value >= _amount, "Not enough ether provided.");
        _;
    }

    function book () payable public onlyWhileVacant costs(2 ether){

        currentStatus = Statuses.Occupied;

        (bool sent, bytes memory data) = owner.call{value: msg.value}("");
        require(sent);

        emit Occupy(msg.sender, msg.value);
    }
}
```
