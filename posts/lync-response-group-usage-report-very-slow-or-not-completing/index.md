<!--
.. title: Lync Response Group Usage Report very slow or not completing
.. slug: lync-response-group-usage-report-very-slow-or-not-completing
.. date: 2014-12-16 15:41:00 UTC+02:00
.. tags: Response Group Usage Report, SQL, Lync, SQL Reporting Services, Lync reporting
.. category: 
.. link: 
.. description: 
.. type: text
-->

In Lync 2013 it's possibly to generate reports in SQL reporting services.
This works fine for all except the "Response Group Usage Report".

The problem is that the report never finish unless it's given a very short time
window. I posted this in the Lync forum: [here](https://social.technet.microsoft.com/Forums/office/en-US/7e472c38-35ac-42cb-ad4a-a683eb0becac/response-group-usage-report-not-working?forum=lyncinterop) and a very kind person who
been in contact with Microsoft support
posted the SQL script below.

The script creates two index that help speed up the stored procedure that fetch
the data for the report.

Someone asked if this can be made more visibly so I created this blog / post.
Enjoy!!  
Please provide any feedback as this is my first blog / post about anything.

```SQL
/*
USE [LcsCDR]
GO
DROP INDEX [IX_SessionDetails_CorrelationId_SessionIdTime] ON [dbo].[SessionDetails]
GO
*/

CREATE NONCLUSTERED INDEX [IX_SessionDetails_CorrelationId_SessionIdTime] ON [dbo].[SessionDetails]
(
 [CorrelationId] ASC,
 [SessionIdTime] ASC,
 [ReplacesDialogIdTime] ASC,
 [ReplacesDialogIdSeq] ASC,
 [CallFlag] ASC,
 [MediaTypes] ASC,
 [User1ClientVerId] ASC,
 [User2ClientVerId] ASC,
 [SessionIdSeq] ASC,
 [SessionStartedById] ASC,
 [User1Id] ASC,
 [User2Id] ASC,
 [ReferredById] ASC
)
INCLUDE (  [TargetUserId],
 [ResponseTime],
 [ResponseCode],
 [SessionEndTime]) WITH (SORT_IN_TEMPDB = OFF, DROP_EXISTING = OFF, ONLINE = OFF) ON [PRIMARY]
go

/*
USE [LcsCDR]
GO
DROP INDEX [IX_SessionDetails_ReplacesDialogIdTime_SessionIdTime] ON [dbo].[SessionDetails]
GO
*/

CREATE NONCLUSTERED INDEX [IX_SessionDetails_ReplacesDialogIdTime_SessionIdTime] ON [dbo].[SessionDetails]
(
 [ReplacesDialogIdTime] ASC,
 [SessionIdTime] ASC,
 [ReplacesDialogIdSeq] ASC,
 [CallFlag] ASC,
 [MediaTypes] ASC,
 [User1ClientVerId] ASC,
 [User2ClientVerId] ASC,
 [SessionIdSeq] ASC,
 [SessionStartedById] ASC,
 [User1Id] ASC,
 [User2Id] ASC,
 [CorrelationId] ASC,
 [ReferredById] ASC
)
INCLUDE (  [TargetUserId],
 [ResponseTime],
 [ResponseCode],
 [SessionEndTime]) WITH (SORT_IN_TEMPDB = OFF, DROP_EXISTING = OFF, ONLINE = OFF) ON [PRIMARY]
go
```
