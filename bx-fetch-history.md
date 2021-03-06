Get list of output points, values, and spends for a payment address.
```sh
$ bx fetch-history --help
```
```
Usage: bx fetch-history [-h] [--config VALUE] [--format VALUE]           
[PAYMENT_ADDRESS]                                                        

Info: Get list of output points, values, and spends for a payment        
address. Requires a Libbitcoin server connection.                

Options (named):

-c [--config]        The path to the configuration settings file.        
-f [--format]        The output format. Options are 'info', 'json', and  
                     'xml', defaults to 'info'.                          
-h [--help]          Get a description and instructions for this command.

Arguments (positional):

PAYMENT_ADDRESS      The payment address. If not specified the address is
                     read from STDIN.
```
This command supports [configuration settings](Configuration-Settings).

Version 3 (and later) does not provide unconfirmed history. A confirmation of at least one block on the strong chain is required for a value to be included.

A `value` of 18446744073709551615 indicates that a spend is uncorrelated to a receipt. This will occur if the spend is indexed but the receipt is not, which can occur if the server is configured to start indexing at a height between the two transactions. Correlation failure can also occur due to an extremely low probability hash collision (1 in 2^49). The caller should always check for this condition.

### Example 1
[134HfD2fdeBTohfx8YANxEpsYXsv5UoWyz](https://blockchain.info/address/134HfD2fdeBTohfx8YANxEpsYXsv5UoWyz)
```sh
$ bx fetch-history 134HfD2fdeBTohfx8YANxEpsYXsv5UoWyz
```
```
transfers
{
    transfer
    {
        received
        {
            hash 97e06e49dfdd26c5a904670971ccf4c7fe7d9da53cb379bf9b442fc9427080b3
            height 247683
            index 1
        }
        spent
        {
            hash b7354b8b9cc9a856aedaa349cffa289ae9917771f4e06b2386636b3c073df1b5
            height 247742
            index 0
        }
        value 100000
    }
}
```

> The `spent` property indicates that the received amount has been spent. The `spent.height` property indicates the block height at which the spend transaction is confirmed.

> The `received.height` property indicates the block height at which the receive transaction is confirmed.

### Example 2
[13Ft7SkreJY9D823NPm4t6D1cBqLYTJtAe](https://blockchain.info/address/13Ft7SkreJY9D823NPm4t6D1cBqLYTJtAe)
```sh
$ bx fetch-history 13Ft7SkreJY9D823NPm4t6D1cBqLYTJtAe
```
```js
transfers
{
    transfer
    {
        received
        {
            hash b7354b8b9cc9a856aedaa349cffa289ae9917771f4e06b2386636b3c073df1b5
            height 247742
            index 0
        }
        value 90000
    }
}
```

> A missing `spent` property indicates that the output is unspent.