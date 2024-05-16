# SOLIDITY DAY 2

Ở bài này thì chúng ta sẽ tìm hiểu về biến ( variables )

## Variables

Ở trong 1 Smart Contract thì sẽ có 2 kiểu variables:

- State Variable
- Local Variable

#### Local Variable

Những biến nằm ở trong hàm, chỉ có thể được sử dụng trong scope function. Ngoài scope không xài được !

```solidity
function getValue() public pure returns(uint) {
    // pure: does not read from or modify the contract's state.
    uint value = 1;
    return value;
}
```

#### State Variable

Những biến được khai báo ở Smart Contract, có thể được access ở bên ngoài Smart Contract ( Nếu khai báo là public ), xài ở hàm.

Một số biến thông thường:

```solidity
int256 myInt = 1;

uint256 public myUnit256  = 1;

uint public  myUint = 1;
// Giống như myUnit256, shorthand của nó !

uint8 public myUint8 = 1;
```

Tại sao cần phải quan tâm về các biến thể của Int:
=> Bởi vì lưu trữ trên blockchain rất tốn tiền

các loại biến khác:

```solidity
string public myString = "Hello, world!";

bytes32 public myBytes32 = "Hello, world!";

address public myAddress = 0xc7549A116ed67Ef11299e2D5c14DF69e42084E18;
```

Tạo ra 1 kiểu dữ liệu của riêng mình, đối với các ngôn ngữ khác là Interface:

```solidity
struct MyStruct {
    uint256 myUnit256;
    string myString;
}

MyStruct public myStruct = MyStruct(1, "Hello, world!");
```

## Arrays

```solidity
uint[] public uintArray = [1,2,3];
// 1 Mảng uint

string[] public stringArray = ["apple", "banana", "carrot"];
// 1 Mảng string

string[] public values;

uint256[][] public array2D = [[1,2,3], [4,5,6]];
// Mảng 2 chiều
```

Tương tác với mảng

```solidity
function addValue(string memory _value) public {
    // memory: biến chỉ tồn tại trong quá trình gọi hàm
    values.push(_value);
}

function valueCount() public view returns(uint){
    // view: meaning it will not modify the state of the contract.
    return values.length;
}
```

## Mapping

Cấu trúc giống như 1 Map ở các ngôn ngữ khác thôi !

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract MyContract {
   //Mapping
   mapping(uint => string) public names;
   mapping(uint => Book) public books;
   mapping(address => mapping(uint => Book)) public myBooks;

   struct Book {
        string title;
        string author;
   }

   constructor() {
        names[1] = "Adam";
        names[2] = "Bruce";
        names[3] = "Carl";
   }

   function addBook(uint _id, string memory _title, string memory _author) public {
        books[_id] = Book(_title, _author);
   }

   function addMyBook(uint _id, string memory _title, string memory _author) public {
        myBooks[msg.sender][_id] = Book(_title, _author);
   }
}
```

## Loop And Conditionals

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract MyContract {
   // Conditionals
   // Loops
   uint[] public numbers = [1,2,3,4,5,6,7,8,9,10];

   address public owner;
   // Lưu lại thông tin của người sender

   constructor() {
    owner = msg.sender;
   }

   function countEventNumbers() public view returns(uint) {
    uint count = 0;
    for(uint i = 0; i < numbers.length; i++){
        if(isEventNumber(numbers[i])){
            count++;
        }
    }
    return count;
   }

   function isEventNumber(uint _number) public pure returns(bool){
    if(_number % 2 == 0){
        return true;
    } else {
        return false;
    }
   }

   function isOwner() public view returns (bool){
    return msg.sender == owner;
   }

}
```
