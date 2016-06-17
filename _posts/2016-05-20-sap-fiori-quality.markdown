---
layout: post
comments: true
title:  "SAP's Fiori Applications - A quality question"
excerpt: "SAP provides a number of SAP Fiori applications for customers; however from experience in implementing and using these application I have serious concerns over their quality and SAP's goals..."
date:   2016-05-20 09:00:00
mathjax: false
---

I've been implementing and using SAP's Fiori applications for a couple of years now and over that time I've come to the startling conclusion that they are very poorly crafted. SAP have made an economic decision to go with the cheapest bidder for software development and it is REALLY showing.

My customers now prefer that we custom build applications instead of using the "SAP rubbish" and the SAP name has become synonymous with poor quality, particularly in the UI space.

The applications often require a large amount of OSS notes to be applied and it is not clearly documented which ones are needed. Like finding a needle in a haystack. If you raise an incident - expect a resolution to be 6 months away as that is typical in my experience - even for HIGH priority incidents and at Max Attention customers. Unbelievable!

To fix allot of the issues your simply on your own to modify the application yourself given the time-frames of working with SAP support.

Before I go into any details I do need to say that SAPUI5 (and OpenUI5) are actually top quality libraries with an awesome and approachable development team behind them (*the new SAP*). However when it came to building the Fiori applications for customers to use, SAP have dropped the ball and I can only assume they have sent the work off to ABAP developers or some other group with no experience in web applications.

Lets take look at just a couple of the standard applications.

## My Timesheets (V2)
This application is an abomination. I only wish I new before we implemented it. There have been so many bugs in the coding I simply can't describe them all.
We have had to make 3 enhancements (2 x BAdI's and 1 x implicit enhancement spot) as well as enhancing the JavaScript controllers for each page just to get it functioning as a reasonable person would expect.

### A few UI issues
1. Cost Objects that you time-write to do not display properly. In fact the standard application just shows the controlling area instead. How did this pass quality assurance. If a competent, modern software engineering team was involved this simple would not get through. It renders the application useless for the users.
To fix this required an implicit enhancement in the backend OData service handler class: `CL_HCM_TIMESHEET_MAN_DPC_EXT` is required. At then end of method `TIMEDATALIST_GET_ENTITYSET()` you can insert an implicit enhancement; loop over the data fields and derive the correct text for each cost object and activity type.

2. All the value help are completely un-intuitive and essentially un-usable. If you want your users to have any chance at all of finding cost objects to time-write to you will have to write your own value help code in EVERY case.
Luckily there is a BAdI to allow this: `HCM_B_TSH_PICKLIST_TXTFILTER`. If you implement this BAdI you can use method `if_hcm_tsh_picklist_txtfilter~search_by_text()` to code handler for each value help via the field name.
I've re-written each one (i.e. WBS, Cost Center, Unit and so on) so that by default it either shows all results (which are paged of course) or if the user starts typing it provides a hierarchical lookup, first by the object Id. If it doesn't find anything then by the short text. Finally it search by the long text.
(The rules are slightly different for each field. For example with UNIT I just provide a small configurable list as users can normally only time-write in Hours and maybe a few other units for mileage.)

3. Field Ranking. The fields on the time-writing overview screen each have a ranking assigned to determine what is the important information (which is shown in bold) and so on. There is a bug with the WBS field that the application assigns a ranking based on field name POSID,  but it compares to CATSDB that uses the field name RPROJ. Therefore the WBS entries display weird with the activity being the main text instead of the WBS text.
Again we are lucky that there is a BAdI available so that we can fix this. `HCM_B_TSH_FIELDRANKINGS` can be used with method `if_hcm_tsh_fieldrankings~assign_fieldrankings()` to override and provide our own field rankings - just copying the SAP code but fixing the WBS field name.

4. Layout on mobile devices. Once you have saved some time entries and wish to submit them for approval, the Submit button is pushed off the screen when working on a phone. To fix this you need to replace the `S3.controller.js controller.js` controller and change function `getHeaderFooterOptions`. Unfortunately there is no Fiori extension points for this. In this method you can resort the array of buttons to ensure Submit is always visible and doesn't take multiple clicks to find it.

5. Half-baked solution for clock times. The application "half" works for allowing the user to enter their time as either a duration or clock times (start time and finish time.). You can fix this with a small modification to controller `S31.controller.js` in the `initializeView()` function. Just add the following to the end of the function:
```
	if (this.clockEntry) {
		this.byId("ClkTimeDurationEle").setVisible(true);
	} else {
    	this.byId("ClkTimeDurationEle").setVisible(false);
	}
```

## Approve Timesheets (V2)
This one has similar issues to My Timesheets (above) where the cost objects do not display correctly rendering the application useless as this data is one of the main decision points for manager in approving time.

## Conclusion
__Build your own applications__. They can be completely tailored to your situation and believe it or not the cost of implementation will be little different from using any of the SAP provided Fiori applications. _Speaking from experience_, I built a suite of custom HR Fiori applications in less time than it took to get the Leave processing and time-sheet applications from SAP working in a reasonable fashion. No joke!

__Use skilled software engineers__. You think it sounds good getting in a cheap resource... You will just pay 5 times over; not only in the quality of the build and time of the build but in continual maintenance headaches.


*ps. My experiences that I speak of above have been with the core set of Fiori applications that run on Any-DB. These include all the MM Approvals applications and all the HR ESS-style applications like Leave, Time-sheets, etc. I have read that the CRM applications (as an example) are of a much better coding quality.*
