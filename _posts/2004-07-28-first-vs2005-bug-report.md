---
layout: post
title: First VS2005 bug report
---
Microsoft has recently launched a [Product Feedback](http://lab.msdn.microsoft.com/ProductFeedback/) site, which includes a bug reporting section. This turns out to be a great place to report bugs they introduced in VS2002/2003 and still haven't fixed.

An example of this would be a subtle, insidious bug in compiler-generated code that was introduced with ATL 7.0 in Visual Studio 2002. This was the problem that caused LDLS to crash if Dragon Naturally Speaking was running. It's easy enough to work around, but it would be nice to have the auto-generated code be bug free. Here's my [bug report](http://lab.msdn.microsoft.com/ProductFeedback/viewfeedback.aspx?feedbackid=b1f188b6-5e35-4284-8e7e-57daf2a66be0); let's see what happens with it&hellip;

**Update 2011-09-09:** The bug is still tracked on [Connect](https://connect.microsoft.com/VisualStudio/feedback/details/100561/incorrect-logic-in-dllcanunloadnow-for-vc-atl-project) all these years later. And Microsoft fixed it!
