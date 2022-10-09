This file contains light modifications of 3 Solidity Smart Contracts.

1. Create [multisign-contract](https://github.com/gnosis/MultiSigWallet/blob/master/contracts/MultiSigWallet.sol) withdrawal upper-bound per one transaction that equals 66 ETH.

```
    @@ -91,6 +91,11 @@ contract MultiSigWallet {
         _;
     }

+    modifier notBiggerThen(uint256 value, uint256 maxValue) {
+        require(value <= maxValue);
+        _;
+    }
+
     /// @dev Fallback function allows to deposit ether.
     function() payable {
         if (msg.value > 0) Deposit(msg.sender, msg.value);
@@ -182,7 +187,7 @@ contract MultiSigWallet {
         address destination,
         uint256 value,
         bytes data
-    ) public returns (uint256 transactionId) {
+    ) public notBiggerThen(value, 66 ether) returns (uint256 transactionId) {
         transactionId = addTransaction(destination, value, data);
         confirmTransaction(transactionId);
     }
```

2. Restrict [token](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/f2112be4d8e2b8798f789b948f2a7625b2350fe7/contracts/token/ERC20/ERC20.sol) transfers on Saturday.

```solidity
    contract ERC20 is Context, IERC20 {
+    uint256 constant DAY_IN_SECONDS = 86400;
+
     mapping(address => uint256) private _balances;

     mapping(address => mapping(address => uint256)) private _allowances;
@@ -264,6 +266,10 @@ contract ERC20 is Context, IERC20 {
     ) internal virtual {
         require(sender != address(0), "ERC20: transfer from the zero address");
         require(recipient != address(0), "ERC20: transfer to the zero address");
+        require(
+            uint8((block.timestamp / DAY_IN_SECONDS + 4) % 7) != 5,
+            "ERC20: transfer not allowed on Saturday"
+        );

         _beforeTokenTransfer(sender, recipient, amount);
```

3. Make a payment for the [token](https://github.com/mixbytes/solidity/blob/076551041c420b355ebab40c24442ccc7be7a14a/contracts/token/DividendToken.sol) accepted only trough a function with a (bytes[32]) comment. Restrict any other type of receiving.

```solidity
@@ -33,7 +33,7 @@ contract DividendToken is StandardToken, Ownable {
         );
     }

-    function() external payable {
+    function deposit(bytes[32] comment) external payable {
         if (msg.value > 0) {
             emit Deposit(msg.sender, msg.value);
             m_totalDividends = m_totalDividends.add(msg.value);
```
