#### Command Line

BX uses source code generation and Boost's [program_options](http://www.boost.org/doc/libs/1_50_0/doc/html/program_options/overview.html) library to bind command line parameters to strongly-typed command class properties.

Command headers are generated from metadata during development. The metadata include full definition for all command parameters, including name, shortcut, data type, order, cardinality, optionality, default value, help description, file input delegation, fallback to STDIN and definition for localized messages.
```xml
<command symbol="hd-private" category="WALLET">
    <option name="help" description="Derive a child HD (BIP32) private key from another HD private key." />
    <option name="hard" shortcut="d" description="Signal to create a hardened key." />
    <option name="index" type="uint32_t" description="The HD index, defaults to zero." />
    <argument name="HD_PRIVATE_KEY" stdin="true" type="hd_private" description="The parent HD private key.  If not specified the key is read from STDIN." />
</command>
```
Input processing is handled in shared code and generated headers. All values are available to command implementation via strongly-typed getters on the command class:
```c++
// command implementation
console_result invoke(ostream& output, ostream& error)
{
    // bound getters
    auto hard = get_hard_option();
    auto index = get_index_option();
    auto& secret = get_hd_private_key_argument();

     /* command logic omitted */

    return console_result::okay;
}
```
Corresponding setters enable library consumers to execute BX methods directly. This is the access technique used by all tests:
```c++
BOOST_AUTO_TEST_CASE(hd_private__invoke__mainnet_vector2_m_0_2147483647h_1_2147483646h__okay_output)
{
    BX_DECLARE_COMMAND(hd_private);
    
    // corresponding setters
    command.set_hard_option(true);
    command.set_index_option(2147483646);
    command.set_hd_private_key_argument({ "xprv9zFnWC6h2cLgpmSA46vutJzBcfJ8yaJGg8cX1e5StJh45BBciYTRXSd25UEPVuesF9yog62tGAQtHjXajPPdbRCHuWS6T8XA2ECKADdw4Ef" });
    
    BX_REQUIRE_OKAY(command.invoke(output, error));
    BX_REQUIRE_OUTPUT("xprvA1RpRA33e1JQ7ifknakTFpgNXPmW2YvmhqLQYMmrj4xJXXWYpDPS3xz7iAxn8L39njGVyuoseXzU6rcxFLJ8HFsTjSyQbLYnMpCqE2VbFWc\n");
}
```

#### Standard and File Input

In most commands the option is available to load the primary input parameter via [STDIN](http://wikipedia.org/wiki/Standard_streams#Standard_input_.28stdin.29). In many cases the input value can optionally be read from STDIN. Multi-valued inputs supported in STDIN treat any [whitespace](http://en.wikipedia.org/wiki/Whitespace_character) as a separator.

#### Configuration Settings

BX uses Boost's [program_options](http://www.boost.org/doc/libs/1_50_0/doc/html/program_options/overview.html) library to bind configuration settings to strongly-typed application level properties. Settings are populated by shared code into properties generated from metadata.
```xml
  <configuration section="general">
    <!-- Only hd-new and stealth-encode currently use the testnet distinction, apart from swapping servers. -->
    <setting name="network" default="mainnet" description="The network to use, either 'mainnet' or 'testnet'. Defaults to 'mainnet'." />
    <setting name="retries" type="base10" description="Number of times to retry contacting the server before giving up." />
    <setting name="wait" default="2000" type="uint32_t" description="Milliseconds to wait for a response from the server." />
  </configuration>
    
  <configuration section="mainnet">
    <setting name="url" type="uri" default="tcp://obelisk.airbitz.co:9091" description="The URL of the Obelisk mainnet server." />
  </configuration>
  
  <configuration section="testnet">
    <setting name="url" type="uri"  default="tcp://obelisk-testnet.airbitz.co:9091" description="The URL of the Obelisk testnet server." />
  </configuration>
```
The implementation supports a two level hierarchy of settings using "sections" to group settings, similar to an `.ini` file:
```ini
# Example Bitcoin Explorer (BX) configuration file.

[general]

# Only hd-new and stealth-encode currently use the testnet distinction, apart from swapping servers.
# The network to use, either 'mainnet' or 'testnet'. Defaults to 'mainnet'.
network = mainnet

# Number of times to retry contacting the server before giving up.
retries = 0

# Milliseconds to wait for a response from the server.
wait = 2000

[mainnet]

# The URL of the default mainnet Obelisk server.
url = tcp://obelisk.airbitz.co:9091

[testnet]

# The URL of the default testnet Obelisk server.
url = tcp://obelisk-testnet.airbitz.co:9091
```
The path to the configuration settings file is specified by the `--config` command line option or otherwise the `BX_CONFIG` environment variable. If the file is specified by either method, and is not found or contains invalid setttings, an error is returned via STDERR.

Configuration settings are generated from metadata during development. The metadata includes full definition for all settings, including section, name, data type, default value and help description. If the file, or a given value within the file, is not specified the value defaults according to its metadata default value. The BX `settings` command shows the current value of all configuration settings.

#### Environment Variables

BX uses Boost's [program_options](http://www.boost.org/doc/libs/1_50_0/doc/html/program_options/overview.html) library to bind environment variables. All BX environment variables are prefixed with `BX_`. Currently environment variables are bound explicitly (i.e. bindings are not generated from metadata).

Currently `BX_CONFIG` is the only bound environment variable. BX uses a Boost feature to tie the environment variable and the command line option of the same identity (i.e. `--config`). In BX the command line option takes precedence.