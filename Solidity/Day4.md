# SOLIDITY DAY 4

Ở bài này chúng ta sẽ học về Inheritance và Interaction giữa các Smart Contract.

Bây giờ mình có 1 Contract MyContract đơn giản như sau:

```solidity
contract MyContract {
    address owner;
    string secret;

    constructor(string memory _secret)  {
        owner = msg.sender;
        secret = _secret;
    }

    modifier onlyOnwer() {
        require(msg.sender == owner, "Must be owner");
        _;
    }

    function getSecret() public view onlyOnwer returns(string memory){
        return secret;
    }
}
```

Ok viết như vầy cũng ngon ngon rồi đó, nhưng mà nếu chúng ta muốn tái sử dụng logic xác thực owner mới được xài thì sao. Trong lập trình hướng đối tượng ta có khái niệm đó là kế thừa.

## Inheritance

Ok vậy bây giờ mình sẽ tách Owner ra để làm 1 Base Contract để có thể tái sử dụng logic xác thực ở các Contract khác:

```solidity
contract Ownable {
    address owner;

    modifier onlyOnwer() {
        require(msg.sender == owner, "Must be owner");
        _;
    }

    constructor()  {
        owner = msg.sender;
    }
}


contract MyContract is Ownable{
    string secret;

    constructor(string memory _secret)  {
        secret = _secret;
        super;
    }

    function getSecret() public view onlyOnwer returns(string memory){
        return secret;
    }
}
```

- Với đoạn code trên thì chúng ta đã cho MyContract kế thừa Ownable với từ khoá `is`.
- Ngoài khai báo từ khoá `is` thì chúng ta còn phải new mới base contract bằng từ khoá `super` ở trong Constructor ở Derived Contract.
  => Vậy là chúng ta đã có thể kế thừa logic xác thực bằng cách kế thừa 1 Contract.

---

## Composition

Nếu các bạn nào đã quen với lập trình OOP thì kĩ thuật composition đã không còn quá xa lạ đối với các bạn.

Composition là kĩ thuật, sử dụng 1 class là atribute trong 1 class khác, từ đó chúng ta có thể tái sử dụng các logic ( Ngoài cách kế thừa ). Vậy chúng ta thử Apply vào Solidity, Contract.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity  ^0.8.0;

contract Ownable {
    address owner;

    modifier onlyOnwer() {
        require(msg.sender == owner, "Must be owner");
        _;
    }

    constructor()  {
        owner = msg.sender;
    }
}

contract SecretVault {
    string secret;

    constructor(string memory _secret)  {
        secret = _secret;
    }

    function getSecret() public view returns(string memory){
        return secret;
    }
}

contract MyContract is Ownable {
    address secretVault;

    constructor(string memory _secret)  {
        SecretVault _secretVault = new SecretVault(_secret);
        secretVault = address(_secretVault);
        super;
    }

    function getSecret() public view onlyOnwer returns(string memory){
        return SecretVault(secretVault).getSecret();
    }
}
```

[Tìm hiểu thêm ép kiểu từ Contract sang Address](https://docs.soliditylang.org/en/latest/types.html#contract-types)
[Tìm hiểu thêm về Contract](https://docs.soliditylang.org/en/latest/contracts.html#contracts)

