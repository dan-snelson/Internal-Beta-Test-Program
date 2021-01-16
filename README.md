# Your Internal Beta Test Program: Opt-in / Opt-out via Self Service

## Scripts
- [Extension Attribute Update.bash](Extension%20Attribute%20Update.bash)
<!-- - [Client-side Functions](https://github.com/dan-snelson/Jamf-Pro-Scripts/tree/master/Client-side%20Functions) -->

![Screenshot of Self Service policy](images/Screen%20Shot%202018-06-29%20at%2010.00.27%20PM.png)

---

## Background

Inspired by @elliotjordan's plea to obtain user feedback, we've been using a pop-up menu Computer Extension Attribute called "Testing Level" which has three options:
- Alpha (i.e., bleeding-edge test machines)
- Beta (i.e., direct team members)
- Gamma (i.e., opt-in testers from various teams)

![Screenshot of Testing Level Extenstion Attribute](images/Screen%20Shot%202018-06-29%20at%2010.03.06%20PM.png)

We then have Smart Computer Groups for each of the three levels and a fourth for "none" so we can more easily scope policies.
- Testing: Alpha Group
- Testing: Beta Group
- Testing: Gamma Group
- Testing: None

![Screenshot of Testing: None Smart Group](images/Screen%20Shot%202018-06-30%20at%205.07.54%20PM.png)

This has been working well, but has required a Jamf Pro administrator to manually edit each computer record and specify the desired Testing Level.

After being challenged by @mike.paul and @kenglish to leverage the API, a search revealed @seansb's [Updating Pop-Up Extension Attribute Value via API](https://www.jamf.com/jamf-nation/discussions/18307/) post and @mm2270's reply about [Results of single extension attribute via API](https://www.jamf.com/jamf-nation/discussions/15258/results-of-single-extension-attribute-via-api#responseChild93856) we had exactly what we needed.

---

## API Permissions for Computer Extension Attributes

In my rather frustated testing, the API read / write account needs (at least) the following Jamf Pro Objects "Read" and "Update" permissions:

- Computer Extension Attributes
- Computers
- User Extension Attributes
- Users

---

## Script: Extension Attribute Update

You'll need the [Client-side Functions](https://github.com/dan-snelson/Jamf-Pro-Scripts/tree/master/Client-side%20Functions) installed on each Mac and you'll need to update the "apiURL" in the [Extension Attribute Update.bash](https://github.com/dan-snelson/Jamf-Pro-Scripts/blob/master/Extension%20Attribute%20Update.bash) script which leverages parameters 4 though 7 for:

- API Username
- API Password
- EA Name (i.e., "Testing Level")
- EA Value (i.e., "Gamma" or "None")

![Screenshot of Extension Attribute Update.bash](images/Screen%20Shot%202018-06-30%20at%206.06.30%20PM.png)

---

## Opt-in Beta Test Program Self Service Policy

Create an ongoing Self Sevice policy, scoped to "Testing: None" which includes a single Scripts option of "Update Extension Attribute" and specify:
- API Username (Read / Write)
- API Password (Read / Write)
- EA Name (i.e., "Testing Level")
- EA Value (i.e., "Gamma" or "None")

---

## Opt-out Beta Test Program Self Service Policy

Clone your Opt-in policy and change EA Value to "None" to unset a computer's Testing Level; scope to your testing groups.

---

## Refresh Self Service when opting-in / opting-out

### Opt-in > Files and Processes > Execute Command

We've added the following one-liner to the **Files and Processes > Execute Command** Payload for your Opt-in policy to force Self Service to refresh:

`/usr/local/bin/jamf manage -verbose ; /usr/bin/su \- "`/usr/bin/stat -f%Su /dev/console`" -c "/usr/bin/osascript -e 'tell application \"Self Service\" to activate' -e 'tell application \"System Events\" to key code 53' -e 'tell application \"System Events\" to keystroke \"r\" using {command down}'"`

![Screenshot of Files and Processes > Execute Command](images/Screen%20Shot%202017-11-09%20at%208.26.16%20PM.png)

### Opt-out > Files and Processes > Execute Command

A slight variation on the opt-in one-liner, if a user opts-out of our interal Beta Test program, we'll _also_ remove them from Apple's. (Bbbbbuuuwwwahahahah!!!)

Add to the **Files and Processes > Execute Command** Payload for your Opt-out policy:

`/System/Library/PrivateFrameworks/Seeding.framework/Versions/A/Resources/seedutil unenroll ; /bin/rm -v /Library/Application\ Support/JAMF/Receipts/macOSDeveloperBetaAccessUtility.pkg ; /usr/bin/su \- "`/usr/bin/stat -f%Su /dev/console`" -c "/usr/bin/osascript -e 'tell application \"Self Service\" to activate' -e 'tell application \"System Events\" to key code 53' -e 'tell application \"System Events\" to keystroke \"r\" using {command down}'" ; /usr/local/bin/jamf recon`

(Special thanks to Kyle Flater @kfbbt) for his racing-stripe of including the Escape key in case the user was viewing the Self Service Description.)

### Privacy Preferences Policy Control settings

You'll also need the following Privacy Preferences Policy Control settings nowadays:

- **Identifier:** `com.jamf.management.service`
- **Identifier Type:** Bundle ID
- **Code Requirement:** `anchor apple generic and identifier "com.jamf.management.service" and (certificate leaf[field.1.2.840.113635.100.6.1.9] /* exists */ or certificate 1[field.1.2.840.113635.100.6.2.6] /* exists */ and certificate leaf[field.1.2.840.113635.100.6.1.13] /* exists */ and certificate leaf[subject.OU] = "483DWKW443")`
- **App or Service:** Accessibility
- **Access:** Allow

![Screenshot of Privacy Preferences Policy Control](images/Screen%20Shot%202021-01-16%20at%2010.11.01%20AM.png)

---

## Presentation: JNUC 2019

- [YouTube](https://www.youtube.com/watch?v=AhYPVvO7LwM)
