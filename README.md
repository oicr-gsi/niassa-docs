# Niassa Documentation

Hosts the website for [Niassa](https://www.github.com/oicr-gsi/niassa). Much of
the documentation was copied from the earlier 
[SeqWare](https://www.github.com/seqware/seqware) documentation and modified 
heavily to reflect the changes and ways we use Niassa at OICR.

The website is built using [Jekyll](https://jekyllrb.com/) and currently uses
a modification of the default Minima template.


## Build

```
bundle install
```

## Serve (for testing/development) 

We use --incremental build because of some of the Liquid tags we've added that 
take a while to render all pages (can be up to a minute).

```
bundle exec jekyll serve --incremental
```

## Developer documentation

### Creating documentation for a new version

Different versions of documentation should be hosted in a directory 
corresponding to the version number. To start the documentation for a new 
version, recursively copy the entire directory for the previous version into the 
version number of your choice. 

```bash
# to make docs for version 2.0.4
cp -r 2.0.2 2.0.4
```

You do *not* need to do a massive find-replace for version numbers: this should
already be handled by the Liquid tags described below.

### Liquid tags to automatically fill in version numbers

At the top of each page in a version directory, you will see an `include` tag 
for [functions.liquid](_includes/functions.liquid), 
which contains functions to automatically set the URL and version numbers in
versioned subdirectories:

```
{% include functions.liquid %}
```

* **version_url**: When linking within the version directory, use the variable 
  `{{version_url}}` to fill in the absolute path to the root of the version 
  directory. e.g. Rather than linking to the Pipeline page with 
  `[Pipeline]( "/2.0.2/pipeline" | absolute_url)`, use 
  `[Pipeline]({{version_url}}/pipeline)`.
* **version**: fills in the version according to the directory name sitting 
  in basedir. e.g. Rather than `seqware-distribution-2.0.2-full.jar`, use 
  `seqware-distribution-{{version}}-full.jar`.
  
### 'current' redirects

The `current` directory contains redirect pages that will automatically redirect
to subdirectory that matches the `current_version` in 
[_config.yml](_config.yml). The correct relative page needs to exist in 
[current](current) for it to redirect properly. 

### The nav bar

For items to appear in navigation, add `nav: true` to their front matter. Only
pages in the `current_version` directory in [_config.yml](_config.yml) will be
included.

## License

The Niassa website is free software: you can redistribute it and/or modify
it under the terms of the GNU General Public License as published by
the Free Software Foundation, either version 3 of the License, or
(at your option) any later version.

The Niassa website is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
GNU General Public License for more details.

You should have received a copy of the [GNU General Public License](LICENSE)
along with the Niassa website.  If not, see <http://www.gnu.org/licenses/>.

## Contact

The Niassa website is built and maintained by the
[Genome Sequence Informatics](https://gsi.oicr.on.ca) group at
[Ontario Institute for Cancer Research](https://oicr.on.ca). Get in touch by
submitting a question or issue to our
[Github issue tracker](https://github.com/oicr-gsi/niassa-docs/issues).
