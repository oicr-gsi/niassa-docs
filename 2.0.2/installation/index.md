---
layout: page
title: Installation
nav: true
---
{% include functions.liquid %}

Niassa is designed to work in a 64-bit Linux environment. Depending on your 
requirements, there are several ways to install and use Niassa.

## Option 0 - Client Only

If you are within an environment that already has Niassa configured and set up
(hi OICR!), you will only need to configure your environment to interact with 
the system. As long as you have access to the Niassa webservice host and the 
appropriate permissions, you can write new workflows, install those workflows on 
the remote Niassa system, schedule workflows remotely, etc. 

Most sites do not have Niassa already configured and you will 
typically need to use other options below.


## Option 1 - Docker Whitestar (Pipeline only)

You can run Niassa on your local machine using SeqWare Whitestar and Docker. 
Whitestar is an alternative engine for Niassa that has no metadata writeback, no
database, no Oozie, and no cluster. It takes your workflow, throws it into a 
bash script, and executes it locally. Whitestar also uses an older version of 
SeqWare, the predecessor of Niassa, and so not all functionality may be available.

If your work depends at all on the presence of the cluster, a metaDB, or web
service, you can't use this technique.

To install and run Docker on your local machine, go to the 
[download page](https://store.docker.com/search?type=edition&offering=community) 
and follow instructions for your operating system.

Whitestar is available from the 
[seqware_whitestar](https://github.com/SeqWare/seqware_whitestar) repository on 
Github. Full instructions are given in the README.md file.

## Option 2 - Installing from Scratch

This is not a trivial undertaking and should only be attempted by the brave.

Follow the instructions at [Bindle](https://github.com/CloudBindle/Bindle).

## Next Steps

Now that you have seen how to launch either a local or cloud-based Niassa 
instance follow the "[Getting Started]({{ version_url }}/getting-started)" guide 
to see how to run the sample HelloWorld workflow.
