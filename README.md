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
local(vsl_ctx)

# This is inserted only if running the script remotely
remote(vsl_ctx)
# End of inserted bootstrap code
```

The `local()`, `remote()` and `remote_target` value is used to determine:
- `local`: What gets run locally
- `remote` and `remote_target`: What gets run remotely and where

Any templated string (`"{{ foo }}"`) will be rendered using an environment
provided by the user written in [TOML](https://toml.io/en/).


### The `ctx` argument

Each function, `local` and `remote`, accepts an argument called `ctx`, which
provides various utilities to make common tasks easier:
- `ctx.copy(local_path, remote_path)`


### Running

```bash
vessel -e env.toml script.xsh
```
