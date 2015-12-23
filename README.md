# gas-change-ownership

This is a Google Apps Script to logically change ownership of files and folders by making copies and moving the old ones aside.

## Introduction 

At work we are a Google Apps campus, and we collaborate heavily using the Google Apps platform. Our IT group uses a shared folder on Google Drive to organize our work which is owned by a departmental service account. We want content in this area to be owned by the service account and not individual users so that when people come and go, our documents stay. 

Google Drive has two characteristics that make it hard to manage this area over time.

1. New objects (folders and files) are always owned by the user that creates them, even if the service account owns the containing folder. 
2. Only the owner of a document or folder (or the site superadmin) can reassign ownership of that object to someone else.  It cannot be "taken" by the service account. 

We needed a way to keep content in this area attached to the service account and not individual users. The scenarios that this script is primarily designed to address are the following:

> Joe User is moving on to his next job opportunity after years working in your organization. He owns hundreds of documents in the
> shared area, including several critical pieces of institutional knowledge. If his account is purged or archived when he leaves the
> organization, all of those documents owned by Joe are at risk of disappearing.

or even worse

> Jane Doe was fired with cause.  You want to quickly take steps to preserve critical institutional knowledge and avoid any fallout
> from potential malicious acts.

We could just run the script when someone is leaving on an ad-hoc basis, but for reasons I'll explain later you probably waht this script to run via trigger to keep the area clean on an ongoing basis.

## Details

The key configuration parameters are the id of a starting folder and optionally a targeted user's email address.  It then traverses that folder tree and copies any files that the targeted user owns (or any files that anyone other than the user running the script owns, if no targeted user was provided). It will then make a copy of the file, re-create all of the sharing permissions (optionally adding the old owner as an editor on the new file), and archive the old file away in a separate archiving folder.  It will then traverse the tree again, this time looking for folders that are owned by the targeted user (or any user other than the one running the script, if a targeted user was not provided).  In this case, it will create a new folder, move all the files from old to new, and archive the old folder. 

Caveats: 
1. The user running the script (our service account in our case) must have access to these files and folders. If the service account owns the top-level folder then the service account gets Can Edit permission on anything new created under it.  However, the owner could then revoke access to the file or folder.  At that point, the service account won't even see it, so there's nothing we can do about that.
2. The way I've written this, the service account actually needs Can Edit access, not just access.  That's because I rewrite the old document's title and move the old document out of the way.  

Because of these caveats, you probably don't want to wait until someone is leaving to run this script. You probably want to catch wrongly-owned documents in the shared area soon after creation, before an outgoing employee could do irreparable harm (I use "irreparable" loosely here, because there is a site superadmin with broad powers, but at an institution of 50,000+ people we're not counting on someone in central IT being able to retrieve deleted files efficiently). 

The script has some niceties in it, such as it will log activity to a Google Sheet for later review, and it will check the time and avoid hitting a Maximum Execution Time exception (Google will kill Apps Scripts after 6 minutes).

Read the comments in the code, there's a lot of good stuff in there.

## INSTALLATION

1. Cut and paste this into your Google Apps Script development environment (script.google.com, or Eclipse with the Google plugin, or what have you...).
1. Create a Google Sheet for logging.
1. Set the configuration parameters in the configuration area at the top of the script.
1. Run the script. Set up triggers if you'd like

## Known Bugs

1. If it takes more than 6 minutes to enumerate the folders and files you need to traverse, this script will never do anything. (This can likely be fixed using the IteratorTokens that GAS provides.)  However, it's ok if the script cannot process all the affected files and folders in one run, as long as it can enumerate them within one run.

## Unknown Bugs

I'm certain there are plenty.  Did I mention this was my first ever GAS script?  Please let me know if/when you find more bugs.

