---
content_title: ABI/Code generator attributes
link_text: ABI/Code generator attributes
---

The new ABI generator tool uses C++11 or GNU style attributes to mark `actions` and `tables`.

## [[eosio::action]]
This attribute marks a method as an action.
Example (four ways to declare an action for ABI generation):
```cpp
// this is the C++11 and greater style attribute
[[eosio::action]]
void testa( name n ) {
   // do something
}

// this is the GNU style attribute, this can be used in C code and prior to C++ 11
__attribute__((eosio_action))
void testa( name n ){
   // do something
}

struct [[eosio::action]] testa {
   name n;
   EOSLIB_SERIALIZE( testa, (n) )
};

struct __attribute__((eosio_action)) testa {
   name n;
   EOSLIB_SERIALIZE( testa, (n) )
};
```

If your action name is not a valid [FullOn name](../02_naming-conventions.md) you can explicitly specify the name in the attribute ```c++ [[eosio::action("<valid action name>")]]```

## [[eosio::table]]
Example (two ways to declare a table for ABI generation):
```cpp
struct [[eosio::table]] testtable {
   uint64_t owner;
   /* all other fields */
};

struct __attribute__((eosio_table)) testtable {
   uint64_t owner;
   /* all other fields */
};

typedef eosio::multi_index<"tablename"_n, testtable> testtable_t;
```

If you don't want to use the multi-index you can explicitly specify the name in the attribute ```c++ [[eosio::table("<valid action name>")]]```.

## [[eosio::contract("ANY_NAME_YOU_LIKE")]]
```cpp
class [[eosio::contract("ANY_NAME_YOU_LIKE")]] test_contract : public eosio::contract {
};
```

The code above will mark this `class` as being an `FullOn` contract, this allows for namespacing of contracts, i.e. you can include headers like `eosio::token` and not have `eosio::token`'s actions/tables wind up in you ABI or generated dispatcher.

## [[eosio::on_notify("VALID_EOSIO_ACCOUNT_NAME::VALID_EOSIO_ACTION_NAME")]]
```cpp
[[eosio::on_notify("eosio.token::transfer")]]
void on_token_transfer(name from, name to, assert quantity, std::string memo) {
   // do something on eosio.token contract's transfer action from any account to the account where the contract is deployed.
}

[[eosio::on_notify("*::transfer")]]
void on_any_transfer(name from, name to, assert quantity, std::string memo) {
   // do something on any contract's transfer action from any account to the account where the contract is deployed.
}
```

## [[eosio::wasm_entry]]
```cpp
[[eosio::wasm_entry]]
void some_function(...) {
   // do something
}
```

The code above will mark an arbitrary function as an entry point, which will then wrap the function with global constructors (ctors) and global destructors (dtors).  This will allow for the CDT toolchain to produce WASM binaries for other ecosystems.

## [[eosio::wasm_import]]
```cpp
extern "C" {
   __attribute__((eosio_wasm_import))
   void some_intrinsic(...);
}
```

The code above will mark a function declaration as being a WebAssembly import.  This allows for other compilation modes to specify which functions are import only (i.e. do not link) without having to maintain a secondary file with duplicate declarations.

## [[eosio::action, eosio::read-only]]
The `read-only` attribute marks a method which has been defined as an action as a read-only action.

Example:

```cpp
[[eosio::action, eosio::read-only]]
std::vector<my_struct> get() {
   std::vector<my_struct> ret;
   // retrieve blockchain state and populate the ret vector
   return ret; 
}
```

Contract actions tagged read-only:
* Cannot call insert/update (write) functions on the `Multi-Index API`, nor the `Key Value API`.
* Cannot call `deferred transactions`.
* Cannot call `inline actions`.
* The amount of data returned by read-only queries is limited by the action return value size. By default these are set to 256 bytes by `default_max_action_return_value_size`.

The `cdt-cpp` and `cdt-cc` tools will generate an error and terminate compilation if an action tagged read-only attempts to call insert/update (write) functions, `deferred transactions` or `inline actions`. However, if the command-line override option `--warn-action-read-only` is used, the `cdt-cpp` and `cdt-cc` tools will issue a warning and continue compilation.
