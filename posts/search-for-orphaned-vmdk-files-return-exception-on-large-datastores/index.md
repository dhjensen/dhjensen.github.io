<!--
.. title: Search for Orphaned vmdk files return exception on large Datastores
.. slug: search-for-orphaned-vmdk-files-return-exception-on-large-datastores
.. date: 2015-08-13 09:16:00 UTC+02:00
.. tags: VDI, View, ESXi, Datastore, Powershell, timeout, VMware, ESXCLI, vCenter, PowerCLI
.. category: 
.. link: 
.. description: 
.. type: text
-->

One of the challenges working with VMware vCenter is keeping clean and healthy datastores. Making sure only vmdk files registered with active VMs is stored on Datastores is one example.  
For this purposes there are numerous scripts around that will compare the files of registered VMs with the actual list of vmdk files found in a Datastore  
Examples:  

- <https://communities.vmware.com/message/1527123>
- <http://virtuallyjason.blogspot.dk/2013/08/orphaned-vmdk-files.html>
- <http://www.lucd.info/2011/04/25/orphaned-files-and-folders-spring-cleaning/>

There is however one problem no one seem to be able to address at all across all these scripts. When you search "very" large datastores and the ESXCLI / PowerCLI have been running for 15 minutes the script get an exception. (Our VDI linked clone datastore is very large)  

```Powershell
Exception calling "SearchDatastoreSubFolders" with "2" argument(s): "An error o
ccurred while communicating with the remote host."
At xxxxx
+     $searchResult = $dsBrowser.SearchDatastoreSubFolders <<<< ($rootPath, $se
archSpec)
    + CategoryInfo          : NotSpecified: (:) [], MethodInvocationException
    + FullyQualifiedErrorId : DotNetMethodException".
```

I noticed the exception message was about the remote host so I figured it must be vCenter / ESXi related.  
  
Based on above finding I found this [VMware KB article](http://kb.vmware.com/selfservice/microsites/search.do?language=en_US&cmd=displayKC&externalId=1017253) that describe how to increase the timeout for tasks to vCenter and ESXCLI to much more than the default 15 minutes. I implemented the KB article on my vCenter server and on all the ESXi hosts in the cluster. This did the trick for me and I can now complete the datastore search. More specific I'm now able to complete this line of code from the scripts without the exception:  

```Powershell
$xxx.SearchDatastoreSubFolders($rootPath, $searchSpec)
```

I will not list my specific actions as they are covered in the KB article. However the article doesn't mention that the `vpxa.cfg` file is read only on the ESXi 5.5 host so you need to run the following to get write access  

```bash
chmod u+w /etc/vmware/vpxa/vpxa.cfg
```

I hope this makes sense for everyone.
Please comment if you have any questions or feedback.
