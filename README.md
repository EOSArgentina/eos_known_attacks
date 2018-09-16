# Example Vulnerable Code



## Not checking properly code against eosio.token on transfer

Vulnerable code was first posted [on stack exchange](https://eosio.stackexchange.com/questions/421/how-to-do-something-when-your-contract-is-an-action-notification-recipient-like) and used by several dapps which got hacked. 

```cpp

// extend from EOSIO_ABI
#define EOSIO_ABI_EX( TYPE, MEMBERS )
extern "C" {
   void apply( uint64_t receiver, uint64_t code, uint64_t action ) {
      auto self = receiver;
      if( action == N(onerror)) {
         /* onerror is only valid if it is for the "eosio" code account and authorized by "eosio"'s "active permission */
         eosio_assert(code == N(eosio), "onerror action's are only valid from the \"eosio\" system account");
      }
      if( code == self || code == N(eosio.token) || action == N(onerror) ) {
         TYPE thiscontract( self );
         switch( action ) {
            EOSIO_API( TYPE, MEMBERS )
         }
         /* does not allow destructor of thiscontract to run: eosio_exit(0); */ \
      }
   }
}

EOSIO_ABI_EX(eosio::charity, (hi)(transfer))
``` 

An improved version was later added.

```cpp
if( ((code == self && action != N(transfer)) || (code == N(eosio.token) && action == N(transfer)) || action == N(onerror)) ) { 
``` 
