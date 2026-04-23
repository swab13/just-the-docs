---
title: Quickly retrieving data from Exchange Online mailboxes
parent: Microsoft 365
---

# Quickly retrieving data from Exchange Online mailboxes
{: .d-inline-block }

**14 June 2017**
{: .label .label-grey }

Often we need to perform searches over many mailboxes to find just those few which match certain criteria.

One that I recently had was to find users which have a forward on their mailbox (set on the mailbox attribute) in Microsoft 365.

One option I had was I could run:

```
get-mailbox -resultsize unlimited | where ($_.ForwardingSmtpAddress -like '*')
```

This will work.  However, if you've got a lot of users (e.g. over 100,000 users) it can be very slow.  But why?

As Microsoft 365 (Exchange Online) is a shared platform, Microsoft throttle PowerShell commands.  They throttle PowerShell in a number of ways.  In this instance, by the amount of data that is sent back to your local PowerShell session.

But if I've only got 5 matching users out of thousands, surely that can't be much data.. why is it throttled?

This is where it becomes important to know where the work for the PowerShell command is being done and when the data is being transferred.  As you are connecting to Office 365, some parts of the PowerShell command execute on remote server (Office 365 in our case), some is executed locally where you lauched PowerShell from and they pass data between them.

In the above example, the get-mailbox -resultsize unlimited retrieves that data from every mailbox.  This is done on Office 365 remote server and then it is all sent back to your local PowerShell session.  Once it arrives at your local PowerShell, it will execute the where ($_.ForwardingSmtpAddress -like '*') on the data.  The data is being transferred between the remote Office 365 and local server contains all the mailbox data, and of course due to the size of the data it gets throttled and slows down the command, taking longer to get the results.

# We can speed this up!

The where executes locally, so we really want this to be executed on the remote server, decreasing the amount of data transferred back to your PowerShell session.  So how do we do that?  We can filter the results of the get-mailbox command using the -filter parameter and the command will only return what matches the filter and it performs it all on the remote server.

So we could change this example

```
get-mailbox -resultsize unlimited | where ($_.ForwardingSmtpAddress -like '*')
```

to

```
get-mailbox -resultsize unlimited -filter "ForwardingSmtpAddress -like '*'"
```

# Prove it..

Unfortunately due to way these commands work, we can't use -resultsize parameter to prove that these commands are faster. If we did use the -resultssize parameter e.g. -resultsize 100, the first example the command would only search the first 100 mailboxes while the updated version above would search mailboxes and return only first 100 forward results.

So in order to show the results, I'll run this on a tenancy which contains over 300,000 mailboxes.  The original PowerShell command took over **2.5 hours** while the new command took just over **10 minutes**.

As you can see, the larger the number of mailboxes being searched the more benefit using ```-filter``` will give you.

If you have a small amount of users in your tenancy, then you probably won't see very much difference in using the ```-filter``` parameter, however if you're company expands or your scripts are used on larger Office 365 tenancies, using -filter may help your scripts get quicker results.

