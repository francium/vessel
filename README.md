# Vessel

Prototype of an [Ansible](https://www.ansible.com/) replacement that uses
[Xonsh](https://xon.sh/) to run scripts over SSH.


## Design

PyPI and other non-standard packages will not be supported.

User writes a Xonsh script,

```xsh
import random
import some_standard_python_module

some_constant = "{{ foo }}"

def local(ctx):
    ...

def remote(ctx):
    ...
```

Preprocessor turns it into a bootstrapped standalone script that can be run on
local and remote machine,

```xsh
# Inserted preamble
vsl_util_a = ...
vsl_util_b = ...
vsl_ctx = ..
# end of inserted preamble

import random
import some_standard_python_module

some_constant = "42"

def local(ctx):
    ...

def remote(ctx):
    ...

# Start of inserted bootstrap code
# This is inserted only if running the script locally
if __name__ == "__main__":
    local(vsl_ctx)

# This is inserted only if running the script remotely
if __name__ == "__main__":
    remote(vsl_ctx)
# End of inserted bootstrap code
```

The `local()`, `remote()` and `remote_target` value is used to determine:
- `local`: What gets run locally
- `remote` and `remote_target`: What gets run remotely and where

Any templated string (`"{{ foo }}"`) will be rendered using an environment
provided by the user written in [TOML](https://toml.io/en/).


### Environment
A script requires an environment to be able to run. The environment is like a
configuration that configures vessel and the script for a specific target
machine.

The environment is specified using the `-e`/`--env` flag and is written in a
TOML file.

Due to implementation details and constraints, the parsing is done using an INI
parser, rather than a proper TOML parser. Therefore, only a subset of TOML
syntax is supported.

A minimal environment file looks like,
```toml
[default]
ssh = user@127.0.0.1
```

Additional fields can be specified in the `[default]` section if required. Any
templated string in the script will be rendered using this environment. If a
templated string can not be rendered due to a missing environment value, vessel
will exit with an error


### Environment Values

All the values must be specified in the `[default]` section.

- `ssh`: SSH target (`user@address`)
- `ssh_port`: SSH port (optional, defaults to `22` if not specified)

All other values can be specified as needed in the script.


### The `ctx` argument

Each function, `local` and `remote`, accepts an argument called `ctx`, which
provides various utilities to make common tasks easier:
- `ctx.copy(local_path, remote_path)`


### Running

```bash
vessel -e env.toml script.xsh
```
