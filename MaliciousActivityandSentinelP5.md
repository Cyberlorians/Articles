## Part-5: Malicious traffic in Sentinel - Summary Rules. ## 

Flow logs will become verbose, especially when an allowed port is open and malicious activity is allowed IN. We can leverage Summary Rules to summarize only the Allowed Malicious IN activity. What are Summary Rules?

Per [Microsoft](https://learn.microsoft.com/en-us/azure/sentinel/summary-rules):

Use summary rules in Microsoft Sentinel to aggregate large sets of data in the background for a smoother security operations experience across all log tiers. Summary data is precompiled in custom log tables and provide fast query performance, including queries run on data derived from low-cost log tiers. Summary rules can help optimize your data for:

- Analysis and reports, especially over large data sets and time ranges, as required for security and incident analysis, month-over-month or annual business reports, and so on.

-Cost savings on verbose logs, which you can retain for as little or as long as you need in a less expensive log tier, and send as summarized data only to an Analytics table for analysis and reports.

-Security and data privacy, by removing or obfuscating privacy details in summarized shareable data and limiting access to tables with raw data.

Let’s create a Summary Rule to focus on only ‘Allowed-IN’

1 - ![](https://github.com/Cyberlorians/uploadedimages/blob/main/sr1.png)

![](https://github.com/Cyberlorians/uploadedimages/blob/main/sr2.png)

![](https://github.com/Cyberlorians/uploadedimages/blob/main/sr3.png)