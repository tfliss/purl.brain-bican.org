# BICAN PURLs

This repository provides tools for managing BICAN Permanent URLs (PURLs). 

This repository is cloned and customized from [OBO Foundry Permanent URLs](https://github.com/OBOFoundry/purl.obolibrary.org/). Like <https://github.com/perma-id/w3id.org> per-directory Apache configuration files (`.htaccess` files) are utilized, each of which uses `RedirectMatch` directives to redirect PURL requests to their proper targets. Unlike w3id.org, the Apache configuration files aren't manually edited by hand. Instead, simple YAML configuration format is used, and those YAML configuration are translated into Apache configurations. The YAML files are easier to read and write, and allow us to validate and test PURLs automatically.

## Adding and Updating PURLs

Please use one of these four options to make changes to the PURLs:

1. [Create a new issue](https://github.com/hkir-dev/purl.bican.org/issues/new) describing the change you require.

2. [Browse to the configuration file you want to change](https://github.com/hkir-dev/purl.bican.org/tree/master/config) and click the "pencil" icon to edit it.

3. [Add a new configuration file](https://github.com/hkir-dev/purl.bican.org/new/master/config).

4. [Fork this repository](https://help.github.com/articles/fork-a-repo/) and [make a pull request](https://help.github.com/articles/using-pull-requests/).

All changes are reviewed before they are merged into the `master` branch. Once merged, updated PURLs will be active within 20 minutes.
