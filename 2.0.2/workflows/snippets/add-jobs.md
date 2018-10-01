```java
@Override
public void buildWorkflow() {
    // a bash job to call mkdir
    // note that this job uses the system's mkdir (which depends on the system being *nix)
    Job mkdirJob = this.getWorkflow().createBashJob("bash_mkdir");
    mkdirJob.getCommand().addArgument("mkdir test1");      
   
    // get the relative location of the input file previously defined so that it 
    // can be used in the next Job
    String inputFilePath = this.getFiles().get("file_in_0").getProvisionedPath();

    // a bash job to cat a file into a test file
    // the file is not saved to the metadata database
    Job copyJob1 = this.getWorkflow().createBashJob("bash_cp").setMaxMemory("3000");
    copyJob1.setCommand(catPath + " " + inputFilePath + "> test1/test.out");
    copyJob1.addParent(mkdirJob);
    
    // a bash job to echo to an output file and concat an input file
    // the file IS saved to the metadata database
    Job copyJob2 = this.getWorkflow().createBashJob("bash_cp").setMaxMemory("3000");
    copyJob2.getCommand().addArgument(echoPath).addArgument(greeting).addArgument(" > ").addArgument("dir1/output");
    copyJob2.getCommand().addArgument(";");
    copyJob2.getCommand().addArgument(catPath + " " +inputFilePath+ " >> dir1/output");
    copyJob2.addParent(mkdirJob);
    SqwFile outputFile = createOutputFile("dir1/output", "txt/plain", manualOutput);
    
    copyJob2.addFile(outputFile);
}
```