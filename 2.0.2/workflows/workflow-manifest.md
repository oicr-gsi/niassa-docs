The workflow manifest (`metadata.xml`) includes the workflow name,
version, description, test command, and enough information so that the SeqWare
tools can test, execute, and install the workflow. Here is an example from the
MyHelloWorld workflow:

```
<bundle version="1.0">
  <workflow name="MyHelloWorld" version="1.0" seqware_version="<%= seqware_release_version %>"
  basedir="${workflow_bundle_dir}/Workflow_Bundle_MyHelloWorld/1.0">
    <description>Add a description of the workflow here.</description>
    <workflow_class path="${workflow_bundle_dir}/Workflow_Bundle_MyHelloWorld/1.0/classes/com/github/seqware/MyHelloWorldWorkflow.java"/>
    <config path="${workflow_bundle_dir}/Workflow_Bundle_MyHelloWorld/1.0/config/MyHelloWorldWorkflow.ini"/>
    <requirements compute="single" memory="20M" network="local"  workflow_engine="Pegasus,Oozie" workflow_type="java"/>
  </workflow>
</bundle>
```

As mentioned above, you can edit the description and workflow name in the workflow.properties file.
