---
layout: post
title: Code Comments
---
One of our interns left some, uh, interesting comments in the code. Here are some I found just going through one source file. (Comments are word-wrapped here.  In the original source, each comment was on just one line (up to 500 characters long).)

`// the "i" on the end of the [regular expression] won't work if it's in a string format (which it must be to get that i on the end...AAH, CATCH-22, MY ANCIENT NEMESIS)`

`// This is necessary because this main window actually modifies the start date passed in from the properties dialog (global variables at work, oh ya), so when we pass it back in if we decide to open the properties dialog again once we've looked at our calendar, the start date is bad and causes the properties dialog to crash!  I fixed this by simply keeping track of what the properties dialog returns separately from the gStartDate variable.`

`// Get all the g00dz from the source window`

`// we are about to perfom some serious loadification`

`// MAN, I HOPE THEY DON'T EVER DECIDE TO HAVE A DIFFERENT NUMBER OF DAYS IN THE WEEK THAN SEVEN!@#!@3~4112!@`

`// Collect the latest and greatest saving xml from the peerless and altogether unmatched CreateSaveXML function`
[ editor's note: I deleted that function. ]

`// append the goodness to the main document's XML container for subsequent savification`

