In SX it was common for a command to invoke an internal [Pseudo Random Number Generator](http://wikipedia.org/wiki/Pseudorandom_number_generator). As a weak random number generator can introduce cryptographic weakness this technique has been obsoleted. Any BX command that requires a random number obtains that value as an argument. This places the responsibility of ensuring random number strength on the end-user and also helps them understand the potential for problems.

The "seed" command is provided as a convenience, and is the only command that generates randomness. The `seed` command accepts a bit length argument, and has default and minimum value of 128.
```sh
$ bx seed 256
```
```
e4d28a5972ce0785477f39f58e424c5ef643b26894c50f8e024601f87736b8fe 
```