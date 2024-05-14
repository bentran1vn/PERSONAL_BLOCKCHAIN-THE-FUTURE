# SOLIDITY DAY 1

Khi tạo 1 file solidity ta cần khai báo phiên bản solidity mà chúng ta sẽ xài.

- Bằng cách khai báo như sau `pragma solidity ^0.8.0`

Vì là ngôn ngữ hướng đối tượng nên nó cũng có nhưng tính chấn hướng đối tượng tương tự như bên Java, C#, C++. Bên Java, C# có "Class" thì bên đây có "contract".

- Tạo ra 1 contract (class) bằng `contract Your_contract_name { }`

### Biến và Method

Tạo 1 biến count trong contract Counter thử: `unit count;`
Tạo thử method Get Count và Increase Count: `getCount()` và `incrementCount()`

```solidity
pragma solidity ^0.8.0;

contract Counter {
    //Code goes here...
    uint count; // 1,2,3

    //public cho phép chúng ta truy cập vào hàm kể cả khi
        //bên ngoài contract
    function getCount() public view returns(uint) {
        return count;
    }
    // Định nghĩa kiểu trả về bằng returns(uint)

    //public cho phép khi trên blockchain bên ngoài contract
    function incrementCount() public {
        count = count + 1;
    }

}
```

## Lưu ý

- Method Read, lấy thông tin từ blockchain thì không tốn Gas.
- Method Write, thêm hoặc chỉnh sửa thông tin từ blockchain thì tốn Gas.

#### Set Default Value cho Variable bằng Constructor

```solidity
pragma solidity ^0.8.0;

contract Counter {
    uint count;

    constructor() public {
        count = 0;
    }

    function getCount() public view returns(uint) {
        return count;
    }

    function incrementCount() public {
        count = count + 1;
    }

}
```

Constructor sẽ chạy 1 lần duy nhất khi:

- Contract Inititalization
- Deploy Smart Contract

Viết xong Smart Contract thì chúng ta khoan vội vui mừng, để xài được trên blockchain thì ta cần phải deployed nó lên blockchain.

--> Nhưng để lên được blockchain thì nó cần phải qua 1 giai đoạn đó là compilation. Chuyển đống smart contract đó thành bytecode.
--> Quy trình: Smart Contract -> Compilation ( ByteCode ) -> Deployed Blockchain.

## Lưu ý

- Quá trình Deployed Smart Contract thì ta tạo ra 1 transaction.
- Tất cả data ở trên blockchain thì nằm ở transaction, được nhóm lại với nhau bởi các records gọi là block, block được chain với nhau để tạo thành 1 public ledger.
- Mỗi lần chúng ta gửi tiền từ tài khoản này sang tài khoản khác, call function từ smart contract, deployed smart contract lên blockchain thì ta đều tạo ra 1 transaction và quá trình này đều phải trả phí Gas.

### Refactor and Improve Code

Khi chúng ta khai báo biến, mà để access modifier là public thì mặc định chúng ta sẽ có 1 hàm Get.
Chúng ta có thể gán giá trị mặc định cho biến mà không cần dùng đến constructor !

Final Code:

```solidity
pragma solidity ^0.8.0;

contract Counter {
    //getCount will auto generate !
    uint public count = 0;

    function incrementCount() public {
        count++;
    }

}
```
