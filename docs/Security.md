# Security

 The following documentation provides context, reasoning, and examples of methods and constants found in `openzeppelin/security/`.

 > Expect this module to evolve.

## Table of Contents

* [Initializable](#initializable)
* [Pausable](#pausable)
* [Reentrancy Guard](#Reentrancy-Guard)
* [SafeMath](#safemath)
  * [SafeUint256](#safeuint256)

## Initializable

The Initializable library provides a simple mechanism that mimics the functionality of a constructor. More specifically, it enables logic to be performed once and only once which is useful to setup a contract's initial state when a constructor cannot be used.

The recommended pattern with Initializable is to include a check that the Initializable state is `False` and invoke `initialize` in the target function like this:

```cairo
from openzeppelin.security.initializable.library import Initializable

@external
func foo{
        syscall_ptr : felt*, 
        pedersen_ptr : HashBuiltin*,
        range_check_ptr
    }():
    let (initialized) = Initializable.initialized()
    assert initialized = FALSE

    Initializable.initialize()
    return ()
end
```

> Please note that this Initializable pattern should only be used on one function.

## Pausable

The Pausable library allows contracts to implement an emergency stop mechanism. This can be useful for scenarios such as preventing trades until the end of an evaluation period or having an emergency switch to freeze all transactions in the event of a large bug.

To use the Pausable library, the contract should include `pause` and `unpause` functions (which should be protected). For methods that should be available only when not paused, insert `assert_not_paused`. For methods that should be available only when paused, insert `assert_paused`. For example:

```cairo
from openzeppelin.security.pausable.library import Pausable

@external
func whenNotPaused{
        syscall_ptr: felt*,
        pedersen_ptr: HashBuiltin*,
        range_check_ptr
    }():
    Pausable.assert_not_paused()

    # function body
    return ()
end

@external
func whenPaused{
        syscall_ptr: felt*,
        pedersen_ptr: HashBuiltin*,
        range_check_ptr
    }():
    Pausable.assert_paused()

    # function body
    return ()
end
```

> Note that `pause` and `unpause` already include these assertions. In other words, `pause` cannot be invoked when already paused and vice versa.

For a list of full implementations utilizing the Pausable library, see:

* [ERC20Pausable](../src/openzeppelin/token/erc20/presets/ERC20Pausable.cairo)
* [ERC721MintablePausable](../src/openzeppelin/token/erc721/presets/ERC721MintablePausable.cairo)

## Reentrancy Guard

A [reentrancy attack](https://gus-tavo-guim.medium.com/reentrancy-attack-on-smart-contracts-how-to-identify-the-exploitable-and-an-example-of-an-attack-4470a2d8dfe4) occurs when the caller is able to obtain more resources than allowed by recursively calling a target’s function.

Since Cairo does not support modifiers like Solidity, the [`reentrancy_guard`](../src/openzeppelin/security/reentrancyguard/library.cairo) library exposes two methods `_start` and `_end` to protect functions against reentrancy attacks. The protected function must call `ReentrancyGuard._start` before the first function statement, and `ReentrancyGuard._end` before the return statement, as shown below:

```cairo
from openzeppelin.security.reentrancyguard.library import ReentrancyGuard

func test_function{
        syscall_ptr : felt*,
        pedersen_ptr : HashBuiltin*,
        range_check_ptr
    }():
   ReentrancyGuard._start()
   # function body
   ReentrancyGuard._end()
   return ()
end
```

## SafeMath

### SafeUint256

The SafeUint256 namespace in the [SafeMath library](../src/openzeppelin/security/safemath/library.cairo) offers arithmetic for unsigned 256-bit integers (uint256) by leveraging Cairo's Uint256 library and integrating overflow checks. Some of Cairo's Uint256 functions do not revert upon overflows. For instance, `uint256_add` will return a bit carry when the sum exceeds 256 bits. This library includes an additional assertion ensuring values do not overflow.

Using SafeUint256 methods is rather straightforward. Simply import SafeUint256 and insert the arithmetic method like this:

```cairo
from openzeppelin.security.safemath.library import SafeUint256

func add_two_uints{
        syscall_ptr: felt*,
        pedersen_ptr: HashBuiltin*,
        range_check_ptr
    } (a: Uint256, b: Uint256) -> (c: Uint256):
    let (c: Uint256) = SafeUint256.add(a, b)
    return (c)
end
```
