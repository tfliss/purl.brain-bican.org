# BICAN PURLs

This repository provides tools for managing BICAN Permanent URLs (PURLs). 

This repository is cloned and customized from [OBO Foundry Permanent URLs](https://github.com/OBOFoundry/purl.obolibrary.org/). Like <https://github.com/perma-id/w3id.org> per-directory Apache configuration files (`.htaccess` files) are utilized, each of which uses `RedirectMatch` directives to redirect PURL requests to their proper targets. Unlike w3id.org, the Apache configuration files aren't manually edited by hand. Instead, simple YAML configuration format is used, and those YAML configuration are translated into Apache configurations. The YAML files are easier to read and write, and allow us to validate and test PURLs automatically.

## Adding and Updating PURLs

Please use one of these four options to make changes to the PURLs:

1. [Create a new issue](https://github.com/brain-bican/purl.bican.org/issues/new) describing the change you require.

2. [Browse to the configuration file you want to change](https://github.com/brain-bican/purl.bican.org/tree/master/config) and click the "pencil" icon to edit it.

3. [Add a new configuration file](https://github.com/brain-bican/purl.bican.org/new/master/config).

4. [Fork this repository](https://help.github.com/articles/fork-a-repo/) and [make a pull request](https://help.github.com/articles/using-pull-requests/).

All changes are reviewed before they are merged into the `master` branch. Once merged, updated PURLs will be active within 20 minutes.

## Configuration Format

Four namespaces are available: 

- `taxonomy:` Eg. [http://purl.brain-bican.org/taxonomy/CCN201912131/CCN201912131.json](http://purl.brain-bican.org/taxonomy/CCN201912131/CCN201912131.json)
- `ontology:` Eg. [http://purl.brain-bican.org/ontology/pcl/pcl.owl](http://purl.brain-bican.org/ontology/pcl/pcl.owl)
- `data:` Eg. [http://purl.brain-bican.org/data/CCN201912131/single_nuclei_SMART-Seq_v4](http://purl.brain-bican.org/data/CCN201912131/single_nuclei_SMART-Seq_v4)
- `bican:` Eg. [http://purl.brain-bican.org/bican/home.html](http://purl.brain-bican.org/bican/home.html)

Each project using this service gets a [YAML](http://yaml.org) configuration file in the related `config/` folder. That YAML configuration file is used to generate an Apache `.htaccess` file for that ontology. That Apache configuration will apply to all PURLs for that project.

Every YAML configuration file must have these fields:

- `idspace:` the project's IDSPACE, case-sensitive, usually uppercase
- `base_url:` the part of a PURL that comes after the domain, usually lowercase
- `products:` a list of primary files for the resources and the URLs to redirect them

`BICAN_URL ::= http://purl.brain-bican.org/{base_url}/{product}`

Optional fields include:

- `example_terms:` a list of one or more term IDs for automated testing
- `base_redirect:` If your project redirects its `base_url`, then you will need a `base_redirect:` entry. So `base_redirect: https://knowledge.brain-map.org/celltypes/CCN201912131` will redirect <http://purl.brain-bican.org/taxonomy/CCN201912131> to <https://knowledge.brain-map.org/celltypes/CCN201912131>.
- `entries:` a list of other PURLs under the `base_url`, see below

Here's an example adapted from the [`config/taxonomy/CCN201912131.yml`](config/taxonomy/CCN201912131.yml) file:

    idspace: CCN201912131
    base_url: /taxonomy/CCN201912131

    products:
    - CCN201912131.json: https://raw.githubusercontent.com/AllenInstitute/MOp_taxonomies_ontology/main/humanM1_CCN201912131/updated_dendrogram_CCN201912131.json

    base_redirect: https://knowledge.brain-map.org/celltypes/CCN201912131

    entries:
    - exact: /CCN201912131.json
      replacement: https://raw.githubusercontent.com/AllenInstitute/MOp_taxonomies_ontology/main/humanM1_CCN201912131/updated_dendrogram_CCN201912131.json

Most of these fields are straightforward, but the `entries:` need some more explanation.


### Entries

Each YAML configuration file contains the keyword `entries:` followed by a list of entries. Each entry defines an Apache [RedirectMatch](https://httpd.apache.org/docs/2.4/mod/mod_alias.html#redirectmatch) directive for matching URLs and redirecting to new URLs. Every entry begins with a `- `, followed by keywords and values on indented lines. There are three types of entries:

1. **exact**: The simplest entry matches an exact URL and returns an exact replacement
2. **prefix**: These entries match the first part of a URL and replace just that prefix part
3. **regex**: These entries use powerful regular expressions, and should be avoided unless absolutely necessary.

The `#` character indicates a comment, which is not considered part of the configuration.

See the [`tools/examples/test1/test1.yml`](tools/examples/test1/test1.yml) and [`tools/examples/test1/test1.htaccess`](tools/examples/test1/test1.htaccess) for examples.


#### Exact

In the most common case, your PURL should match a unique URL and redirect to a unique URL. Here's an example from the `config/ontology/pcl.yml` file:

     - exact: /pcl-base.owl 
        replacement: https://raw.githubusercontent.com/obophenotype/provisional_cell_ontology/master/pcl-base.owl

This entry will match exactly the URL `http://purl.brain-bican.org/ontology/pcl/pcl-base.owl`, and it will redirect to exactly `https://raw.githubusercontent.com/obophenotype/provisional_cell_ontology/master/pcl-base.owl`. The matched domain name is fixed `http://purl.bican.org`; the next part is defined by `base_url` `/ontology/pcl/` (namespace should be compatible with the folder `ontology` that the config file resides); the final part is taken from the entry `/pcl-base.owl`. The replacement is expected to be a valid, absolute URL, starting with `http`.

Behind the scenes, the entry is translated into a case-insensitive Apache RedirectMatch directive in `ontology/pcl/.htaccess` by escaping special characters and "anchoring" with initial `^`, the project's base URL, and final `$`:

    RedirectMatch temp "(?i)^/ontology/pcl/pcl-base\.owl$" "https://raw.githubusercontent.com/obophenotype/provisional_cell_ontology/master/pcl-base.owl"


#### Prefix

You can also match and replace just the first part of a URL, leaving the rest unchanged. This allows you to define one entry that redirects many URLs matching a common prefix. Another example from `config/ontology/pcl.yml`:

    - prefix: /releases/
      replacement: https://raw.githubusercontent.com/obophenotype/provisional_cell_ontology/v

This entry will match the URL `http://purl.brain-bican.org/ontology/pcl/releases/2022-01-24/pcl.owl` (for example), replace the first part `http://purl.bican.org/ontology/pcl/releases/` with `https://raw.githubusercontent.com/obophenotype/provisional_cell_ontology/v`, resulting in `https://raw.githubusercontent.com/obophenotype/provisional_cell_ontology/v2022-01-24/pcl.owl`. Effectively, the `pcl.owl` is appended to the replacement.

The translation is similar, with the addition of `(.*)` wildcard and a `$1` "backreference" at the ends of the given strings:

    RedirectMatch temp "(?i)^/releases/(.*)$" "https://raw.githubusercontent.com/obophenotype/provisional_cell_ontology/v$1"


#### Regex

Regular expression entries should only be needed very rarely, and should always be used very carefully.

For the regular expression type, the value of the `regex:` and `replacement:` keywords should contain regular expressions in exactly the format expected by Apache [RedirectMatch](https://httpd.apache.org/docs/2.4/mod/mod_alias.html#redirectmatch). The values will be quoted, but no other changes will be made to them. Consider using `(?i)` to make the match case-insensitive.


#### Tests

Every `prefix` or `regex` entry should also have a `tests:` keyword, with a list of additional URLs to check. Each test requires a `from:` value (like `exact:`) and a `to:` value (like `replacement:`). Here's an example:

    - prefix: /releases/
      replacement: https://raw.githubusercontent.com/obophenotype/provisional_cell_ontology/v
      tests:
      - from: /releases/2022-01-24/pcl.owl
        to: https://raw.githubusercontent.com/obophenotype/provisional_cell_ontology/v2022-01-24/pcl.owl


#### Order of Entries

Apache RedirectMatch directives are processed in the [order that they appear](https://httpd.apache.org/docs/2.4/mod/mod_alias.html#order) in the configuration file. Be careful that your `prefix` and `regex` entries do not conflict with your other entries. The YAML-to-Apache translation preserves the order of entries, so you can control the order of processing, but it's best to avoid conflicts.

## Deployment

See Docker based deployment instructions from [here](docker/README.md) 