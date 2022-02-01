# ns8-kickstart

This is a template module for [NethServer 8](https://github.com/NethServer/ns8-core).
To start a new module from it:

1. Click on [Use this template](https://github.com/NethServer/ns8-kickstart/generate).
   Name your repo with `ns8-` prefix (e.g. `ns8-mymodule`)

1. Search and replace all `kickstart` occurrences in this repository with
   your module name (e.g. `mymodule`)

1. Rename `imageroot/systemd/user/kickstart.service` to reflect your
   module name (e.g. `mymodule.service`)

1. Fix this `README.md` file, by replacing this section with your module
   description.

## Install

Instantiate the module with:

    add-module ghcr.io/nethserver/kickstart:latest 1

The output of the command will return the instance name.
Output example:

    {"module_id": "kickstart1", "image_name": "kickstart", "image_url": "ghcr.io/nethserver/kickstart:latest"}

## Configure

Let's assume that the kickstart instance is named `kickstart1`.

Launch `configure-module`, by setting the following parameters:
- `<MODULE_PARAM1_NAME>`: <MODULE_PARAM1_DESCRIPTION>
- `<MODULE_PARAM2_NAME>`: <MODULE_PARAM2_DESCRIPTION>
- ...

Example:

    api-cli run module/kickstart1/configure-module --data '{}'

The above command will:
- start and configure the kickstart instance
- (describe configuration process)
- ...

Send a test HTTP request to the kickstart backend service:

    curl http://127.0.0.1/kickstart/

## Uninstall

To uninstall the instance:

    remove-module --no-preserve kickstart1
