# 1. Contracts
* Solidity's code is encapsulated in contracts. A `contract` is the fundamental building block of Ethereum applications — all variables and functions belong to a contract.

# 2. Version Pragma
* All solidity source code should start with a `version pragma` — a declaration of the version of the Solidity compiler this code should use. This is to prevent issues with future compiler versions potentially introducing changes that would break your code.

# 3. State Variables
* State variables are permanently stored in contract storage. This means they're written to the Ethereum blockchain.

# 4. `uint`
* In Solidity, uint is actually an alias for `uint256`, a 256-bit unsigned integer. You can declare `uint` with less bits — `uint8`, `uint16`, `uint32`, etc.. But in general you want to simply use `uint`. There's also an `int` data type for signed integers.

# 5. `public`
* `public` is an access modifier. It means that Solidity automatically creates a function that allows you to access the variable from outside of the contract.
```solidity
// example public array
Person[] public people;
```

# 6. Arrays
* There are two types of arrays in Solidity: fixed arrays and dynamic arrays:
    * Fixed arrays: `uint[3]`
    * Dynamic arrays: `uint[]`
* `array.push()` returns a uint of the new length of the array

# 7. Functions
* A function declaration in solidity looks like the following:
```solidity
function eatHamburgers(string memory _name, uint _amount) public {}
```
* Note that we're specifying the function visibility as `public`. We're also providing instructions about where the `_name` variable should be stored - in `memory`.
* It's convention to start private function names and any function parameters with an underscore `_`:
```solidity
function _addToArray(uint _number) private {
  numbers.push(_number);
}
```
* **Return Values**: Functions can return values. You can specify the type of the return value by adding it after the parameter list and access modifier:
```solidity
string greeting = "What's up dog";

function sayHello() public returns (string memory) {
  return greeting;
}
```
* **View Functions**: If a function is only reading data from the blockchain, you can mark it as `view`. This will save you some gas:
```solidity
function sayHello() public view returns (string memory) {}
```
* **Pure Functions**: If a function doesn't read from the blockchain at all (i.e., it only depends on its parameters), you can mark it as `pure`. This will save you even more gas:
```solidity
function _multiply(uint a, uint b) private pure returns (uint) {
  return a * b;
}
```

# 8. Keccak256 and Typecasting
* Keccak256 is a hash function used in Ethereum. It takes in a variable number of `bytes` and returns a 256-bit hash (32 bytes). It takes in just one parameter of type `bytes`. This means that we have to `"pack"` any parameters before calling keccak256:
```solidity
//6e91ec6b618bb462a4a6ee5aa2cb0e9cf30f7a052bb467b0ba58b8748c00d2e5
keccak256(abi.encodePacked("aaaab"));
```
* Typecasting is the process of converting one data type to another. In Solidity, we can typecast a variable by putting the type we want to convert to in `parentheses` before the variable:
```solidity
uint8 a = 5;
uint b = 6;
// throws an error because a * b returns a uint, not uint8:
uint8 c = a * b;
// we have to typecast b as a uint8 to make it work:
uint8 c = a * uint8(b);
```

# 9. Events
* Events allow you to emit log data to the Ethereum blockchain. You can subscribe to these events with client-side code. Events are a way for your contract to communicate that something happened on the blockchain to your app front-end, which can be 'listening' for certain events and take action when they happen:
```solidity
// declare the event
event IntegersAdded(uint x, uint y, uint result);

function add(uint _x, uint _y) public returns (uint) {
  uint result = _x + _y;
  // fire an event to let the app know the function was called:
  emit IntegersAdded(_x, _y, result);
  return result;
}
```
* Your app front-end could then listen for the event. A JavaScript implementation would look something like:
```javascript
YourContract.IntegersAdded(function(error, result) {
  // do something with the result
})
```
