---
title: "NSX Troubleshooting, what changed in the firewall?"
date: 2022-09-06T00:10:00+01:00
draft: false
---

I found a neat feature that I didn't know in the NSX Manager during a late night.
Every publish/change makes a configuration point, that you can see what changed from then -> now.

This can be good for troubleshooting, something that stops working, that might be due to a DFW Configuration issue.

Go to the DFW, over the categories click "Actions" -> Under Drafts click "View"
You will be presented with the saved configurations:

[![NSX-T DFW Changes](/blog/img/NSX-T_DFW-FWChanges.png)](/blog/img/NSX-T_DFW-FWChanges.png)

So lets go into troubleshooting mode, and lets say something stopped working at 10:32. I can find the date in the above screenshot and point at the dots and see the timestamps - look below:

<!--more-->

[![NSX-T DFW Changes 0609](/blog/img/NSX-T_DFW-FWChanges-0609.png)](/blog/img/NSX-T_DFW-FWChanges-0609.png)

If I open that, i can see the sections that contains changes, and what changed that will be, added/modified/removed if i load that config.

[![NSX-T Full Draft](/blog/img/NSX-T_DFW-FWChanges-FullDraft.png)](/blog/img/NSX-T_DFW-FWChanges-FullDraft.png)

So whats visible here?
- What saved it (Would properly be System)
- The user that did it
- Modified time

What are the draft changes?
- The sections that contain changes.
- If you expand the section you can see the "Red/Green/Orange" symbol to the left that tells if the rule is getting created/removed.

Remember it dosent show that a rule was created as "Added" but "Deleted". That's because you will delete the rule when you load the config.

[![NSX-T Draft Diffs](/blog/img/NSX-T_DFW-FWChanges-DraftDifferences.png)](/blog/img/NSX-T_DFW-FWChanges-DraftDifferences.png)

If we click load as you can see below you will be prompted and warned that you will do a full replace of the FW. 

[![NSX-T Draft Load](/blog/img/NSX-T_DFW-FWChanges-Load.png)](/blog/img/NSX-T_DFW-FWChanges-Load.png)

Doing that will as the warning say put you back to the DFW - where you can decide to Publish, make some changes or revert.
