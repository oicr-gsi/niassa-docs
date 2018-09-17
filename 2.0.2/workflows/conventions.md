---
layout : page
title: Workflow Conventions
---
{% include functions.liquid %}

In order for Niassa Pipeline workflows to work well within complex workflows we need to agree on some conventions.

### Exit Status

The authoritative object that includes the exit status codes is net.sourceforge.seqware.common.module.ReturnValue.  Please look there for the latest list but this should be fairly complete:

```
// generally it's a good idea to offset by 10 so if new ones need to be added
// they can be added "between" existing constants
public static final int NULL = -99;
public static final int NOTIMPLEMENTED = -1;
public static final int SUCCESS = 0;
public static final int PROGRAMFAILED = 1;
public static final int INVALIDPARAMETERS = 2;
public static final int DIRECTORYNOTREADABLE = 3;
public static final int FILENOTREADABLE = 4;
public static final int FILENOTWRITABLE = 5;
public static final int RUNTIMEEXCEPTION = 6;
public static final int INVALIDFILE = 7;
public static final int METADATAINVALIDIDCHAIN = 8; // Problem either getting
// parentID or setting
// processingID to a file
// for the next job
public static final int INVALIDARGUMENT = 9;
public static final int FILENOTEXECUTABLE = 10;
public static final int DIRECTORYNOTWRITABLE = 11;
public static final int FILEEMPTY = 12;
public static final int SETTINGSFILENOTFOUND = 13;
public static final int ENVVARNOTFOUND = 14;
public static final int FAILURE = 15;
public static final int FREEMARKEREXCEPTION = 70;
public static final int DBCOULDNOTINITIALIZE = 80;
public static final int DBCOULDNOTDISCONNECT = 81;
public static final int SQLQUERYFAILED = 82;
public static final int STDOUTERR = 90; // Problem when trying to redirect
// stdout to a file
public static final int RUNNERERR = 91; // Some problem internal to the runner
// these can be used to indicate a module is queued or currently running
public static final int PROCESSING = 100;
public static final int QUEUED = 101;
public static final int RETURNEDHELPMSG = 110;
public static final int INVALIDPLUGIN = 120;
```


## File Metatypes

Niassa Pipeline workflows consume input files and output files as 
the result. We use file metatypes to document the inputs and outputs and to
permit automation.

In order to get this to work we need to agree on the metatypes used in Niassa 
workflows. The following are used by one or more workflows. Please check this 
list before you write a new workflow to see if input/outputs can be annotated 
with one of these metatypes.  Only add to this list if you're certain the  
metatype does not already exist below.

For a directory of standard metatypes (or metatypes) (you should use a standard one if it exists!) see [here](http://www.feedforall.com/mime-types.htm), [here](http://silk.nih.gov/public/zzyzzap.@www.silk.types.html) and [here](http://www.iana.org/assignments/media-types/index.html).  Wikipedia has more to say [here](http://en.wikipedia.org/wiki/MIME).

For the current list of metatypes, please see the [current list](https://docs.google.com/spreadsheet/ccc?key=0An-x7dcdlF7AdGhjdjRTU0toZkJXNlNRb1NROXdfLWc).
If you wish to add a new MIME type, use the form which contributes to this list [here](https://docs.google.com/spreadsheet/viewform?formkey=dGhjdjRTU0toZkJXNlNRb1NROXdfLWc6MQ).

