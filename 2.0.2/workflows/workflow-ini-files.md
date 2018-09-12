Each workflow uses an INI file to record which variables it accepts,
their types, and default values. The ini file(s) follow the general pattern of:

```
# comment/specification
key=value
```

For example:

```
input_file=${workflow_bundle_dir}/Workflow_Bundle_MyHelloWorld/${workflow-version}/data/input.txt
greeting=Testing

#either absolute file system paths or relative paths to scripts/applications inside bundles
cat=${workflow_bundle_dir}/Workflow_Bundle_MyHelloWorld/${workflow-version}/bin/gnu-coreutils-5.67/cat
echo=${workflow_bundle_dir}/Workflow_Bundle_MyHelloWorld/${workflow-version}/bin/gnu-coreutils-5.67/echo

# the output directory is a convention used in many workflows to specify a relative output path
output_dir=seqware-results

# the output_prefix is a convention used to specify the root of the absolute output path or an S3 bucket name 
# you should pick a path that is available on all cluster nodes and can be written by your user
output_prefix=./

# manual output determines whether or not SeqWare should enforce the uniqueness of the final directory or not. 
# If false, SeqWare places files in a directory specified by output_prefix/output_dir/workflowname_version/RANDOM/<files>
# where RANDOM is an integer. If true, SeqWare places the files at output_prefix/output_dir and may overwrite existing
# files
manual_output=false

# Optional: This controls the default number of lines of stdout and stderr that jobs in a workflow will report as metadata
# Otherwise, the default in GenericCommandRunner will be used (currently, 10)
seqware-output-lines-number=20
```

You access these variables in the Java workflow using the
`getProperty()` method. When installing the workflow the ini file is
parsed and extra metadata about each parameter is examined. This gives the
system information about the type of the variable (integer, string, etc) and
any default values.

### Required INI Entries

There are (currently) two required entries that all workflows should define
in their ini files. These are related to output file provisioning. In your workflow,
if you produce output files and use the file provisioning mechanism built into
workflows these two entries are used to construct the output location for the
output file.

* output_dir
* output_prefix

For example, if you have a `SqwFile` object can call
`file.setIsOutput(true);` the workflow engine constructs an output path
for this file using the following:

```
<output_prefix>/<output_dir>/<file_name>
```

You can use `s3://bucketname/` or a local path as the prefix.

**Note:** While the above entries are required, it is STRONGLY suggested that workflow developers no longer rely on them to decide the output path of a provisioned file.  Instead we recommend explicitly providing in the ini file whatever paths you may require, possibly using the variables described below, and then assigning that path to the output file via `SqwFile.setOutputPath(String path)`

#### Reserved INI Entries

There are a number of entries that are used by SeqWare and should be avoided in your own workflows. They are:

* parent_accessions
* parent-accessions
* parent_accession
* workflow-run-accession
* workflow_run_accession
* metadata
* workflow_bundle_dir

### INI Variables

The ini files support variables, in the format `$(variable-name}`, that will be replaced when the workflow run is launched. The variable name can refer to another entry in the ini file, or can refer to the following SeqWare generated values:

* `sqw.bundle-dir`: the path to the directory of this workflow's bundle. Support for the legacy version of this variable, `workflow_bundle_dir`, may be removed in a future version.
* `sqw.date`: the current date in ISO 8601 format, e.g., 2013-10-31.  Support for the legacy version of this variable, `date`, may be removed in a future version.
* `sqw.datetime`: the current datetime in ISO 8601 format, e.g., 2013-10-31T16:45:30.
* `sqw.random`: a randomly generated integer from 0 to 2147483647.  Support for the legacy version of this variable, `random`, may be removed in a future version.
* `sqw.timestamp`: the current number of milliseconds since January 1, 1970.
* `sqw.uuid`: a randomly generated [universally unique identifier](http://en.wikipedia.org/wiki/Universally_unique_identifier#Version_4_.28random.29).
* `sqw.bundle-seqware-version`: the version of seqware that this workflow was built with. You should not have to use this often, but it may be useful if you want to trigger different behaviour based on a version of seqware.

Each instance of the above `sqw.*` variables in an ini file will be replaced with a separately resolved value, e.g., multiple instances of `${sqw.uuid}` will each resolve to different values. If you desire to reuse the same generated value, do somthing akin to the following:

```
dirname=output
filename=${sqw.random}
text_file=${dirname}/${filename}.txt
json_file=${dirname}/${filename}.json
```

Thus if `filename` resolved to a value of `12345`, then `text_file` will have a value of `output/12345.txt` and `json_file` will have a value of `output/12345.json`.

