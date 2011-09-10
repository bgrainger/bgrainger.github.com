---
layout: post
title: Changes to Asynchronous Pluggable Protocol Handlers
---
After dealing with yet another customer (through tech support) who was having problems directly attributable to a full/corrupted Temporary Internet Files folder, I decided to change the implementation of our **lbxfile:** and **lbxres:** protocols. (We're alpha testing, after all :-).)

## Current Approach

Even though we can open a stream directly on an embedded file, our protocols work by saving the data to a temp file (so that the embedded file isn't locked while IE is accessing it). Until now, this temp file has been stored in the Temporary Internet Files folder (TIF). 

## Problems with Current Approach

There appear to be numerous problems with using the UrlCache functions to store this data in TIF:

* The size of this folder seems to grow without limit, even though there is a limit set in the IE > Tools > Internet Options > General > Temporary Internet Files/Settings dialog. D.D. just reported deleting 10GB of files. Just accessing a few reports can write a megabyte of data to the cache (Common.js is 184KB).
* The files are never cleaned up properly. This is a minor point, but many megabytes of source code are easily accessible in TIF.
* When the cache gets full, new files are deleted without warning. A symptom of this is when View > Source in IE opens an empty Notepad window. The file is successfully written to the cache, but is deleted immediately before the filename of the cache file is returned to the client. This manifests itself as users' command bars failing to load (because the images for the buttons are inaccessible).
* Filling the cache breaks other components/programs. IE's View > Source is one example, but Libronix Update also uses the cache to download files. Way back in the original Alpha days, we had a machine that couldn't launch Libronix Update because the .lbxupd file itself was deleted before Libronix Update could open it.
* Other weird errors. NewVerseTemplate.xslt mysteriously failing to load was almost certainly a TIF problem; changing `.Item` to `.OpenStream` (in BibleUtility.js) fixed it.

## New Approach

It still seems advantageous to write the data to a temporary file (to avoid locking problems). We're now writing it to a location we control: a subfolder of the user's temp folder. lbxfile: writes files to `%TEMP%\lbxfile` and lbxres: writes files to `%TEMP%\lbxres`.

## Problems with New Approach

There is still the thorny issue of file lifetime management. The protocols create an object (that implements IStream) that reads from the temp file. It would be nice if we could delete the temp file as soon as this object is released. However, as part of the protocol operations, we call `ReportProgress(BINDSTATUS_CACHEFILENAMEAVAILABLE)`, which lets the caller know the name of our temp file. If we omit this call, IE will still load dialogs and reports properly, but `UrlDownloadToCacheFile` or `UrlDownloadToFile`, which are used by CommandBars and TrayIcons, will fail. However, by reporting the filename, we're implicitly granting clients the right to access that file later on. Our current solution is to have a worker thread that runs in the background. Every five minutes it wakes up, scans the temp folder and deletes files that were last accessed over ten minutes ago. The folders get cleaned when the thread is started (when FileProt.dll or ResProt.dll is loaded) but are left intact when the app shuts down (since there isn't enough time to set a DllBeingUnloaded event then wait for the thread to wake up and delete everything when the DLL is being unloaded). (If we wanted maximum purity, the app itself could delete the known folders after waiting for threads to shut down.)
The result of this is that at most fifteen minutes worth of data (probably several megabytes) could be left in the user's temp folder in between runs of the program.

## So?

This will be in the release after 2.1 Alpha 3, which should probably be an alpha (instead of a beta) release just because of this change. The code's still fairly untested under heavy use (I zipped the shell and some addins on my system, and it worked well enough), so it will be good to have some reports that it's working correctly. There probably won't be any amazing success stories with it (maybe one or two users who were having weird problems will be fixed?), but we shouldn't see any more problems that I currently attribute to TIF (unless there's a bug in the new implementation).
