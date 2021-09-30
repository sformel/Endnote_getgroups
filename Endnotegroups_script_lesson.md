## Get Group Metadata From Endnote

#### Using Apple Scripts

_last updated Sep 27, 2021_

## Why?

Endnote allows you to create metadata about your referencel library by the creation of groups, but doesn't make it easy to export the group information.  I've been managing a shared library that is used by about 30 people and I needed a way to create a daily log of which records were in which groups.  Why? Well, sometimes mistakes happen, and if a record is put in the trash, it loses all of it's associations with group, even if you "restore" it.  Many people reccomend using custom fields to track groups, but I haven't found a way to automatically update those fields as records are added to and removed from groups.  Lastly, I also needed to generate summary statistics to understand how many unique records were in each group, how many records were shared by groups, and which groups they were shared by.

Endnote has an API that allows you to get some record information, but the documentation is extremely sparse, and no matter how hard I tried, I couldn't get the download link to appear to just begin to learn to use it.  They also offer some funciontality with Apple Scripts, which I had never used before.  Endnote offers a few bare bones examples in the help documentation but, like the API, doesn't offer much support (i.e. there are a lot of unanswered questions on the messageboards).  

In the end, my need for a solution (after several very tedious repairs to the library) overwhelmed my need to use my time for something other than learning Apple Scripts. Hopefully I help make someone else's life easier by sharing this script.

Here are some links that helped me out: 

- [Apple Script Dictionaries](https://apple.stackexchange.com/questions/46521/how-do-i-find-out-the-applescript-commands-available-for-a-particular-app)
- [How to write function to write to text file](https://macosxautomation.com/applescript/sbrt/sbrt-09.html)

#### A few notes
* I've successfiully used this in Endnote X9, I wouldn't be surprised if it doesn't work for all versions.
* I was unable to automate this with "launch agents", another mac tool for automation.  Endnote didn't like the commands being run from the terminal as opposed to the script editor and would reject the attempt to open Endnote.
* I bring this output into R for comparison and stats, but you could just as easily work with it in Python or other languages.
* I tried to run this on the shared library while I was logged in as another user instead of as the owner.  Endnote won't allow this, you must be the library owner to run these queries.


#### Script Walk through (complete script at end)

_Note that there are two lines in the script where you will need to replace "yourusername" with your real username so the file path can be properly read._

Set the default delimiter to newline

```
set AppleScript's text item delimiters to "\n"
```

Set date for filename

```
set dateObj to (current date)
set theMonth to text -1 thru -2 of ("0" & (month of dateObj as number))
set theDay to text -1 thru -2 of ("0" & day of dateObj)
set theYear to year of dateObj
set dateStamp to "" & theYear & theMonth & theDay
```

Open Endnote and open library named "My Endnote Library" in path: Macintosh HD/Users/yourusername/Documents/My Endnote Library.enl
 
```
tell application "EndNote X9"
open "Macintosh HD:Users:yourusername:Documents:My Endnote Library.enl"
```

Set variable "myGroups" to the list of "Custom" groups.  Window 3 refers to the 3rd window in Endnote, according to Apple Scripts.

```
set myGroups to get groups of type "CUSTOM" in window 3
```
Make empty array named "finalprod"
```     
        set finalprod to {}
```     
For loop. For each of the elements in the variable "myGroups", get records for that group and assign to variable "recordIDs.  For each of the records in recordIDs, 

1. assign the "Title" field to the "title" variable
2. assign the "unformatted record" to the variable "ufRecord"

Add a line to the end of the array "final product" that consists of 4 elements concatenated together with the "|" symbol:

1. The groupname
2. the Record ID
3. the unformatted record
4. the title

```
repeat with thegroup in myGroups
        set recordIDs to get records in thegroup in window 3
        repeat with theRecord in recordIDs
                set title to field "Title" of record theRecord
                set ufRecord to unformatted record theRecord
                set the end of finalprod to {(thegroup as text) & "|" & (theRecord as text) & "|" & (ufRecord as text) & "|" & (title as text)} as text
               end repeat
        end repeat
```
        
Convert finalprod to text

```
set finalprod to finalprod as text
```

Make variables that feed into function

```     
set this_data to finalprod
        set this_file to ((("Macintosh HD:Users:yourusername:Documents:MDBC Endnote Logs:MDBC_Endnote_group_records_" & dateStamp & ".txt") as string))
        my write_to_file(this_data, this_file, false)
```     
Quit Endnote

```
quit 
```     

End Script

```
end tell
```

Subroutine (AKA function) to write data to text file.  Used at end of script above.

```
on write_to_file(this_data, target_file, append_data)
        try
                set the target_file to the target_file
                set the open_target_file to open for access file target_file with write permission
                if append_data is false then set eof of the open_target_file to 0
                write this_data to the open_target_file starting at eof
                close access the open_target_file
                return true
        on error
                try
                        close access file target_file
                end try
                return false
        end try
end write_to_file
```


#### Entire script

Copy this into a the Mac Program, "Script Editor" (Applications > Utilities). Press play to run. To view progress: View > Show log

```
#https://apple.stackexchange.com/questions/46521/how-do-i-find-out-the-applescript-commands-available-for-a-particular-app
#https://macosxautomation.com/applescript/sbrt/sbrt-09.html

#Get groups for all references in Endnote Library

set AppleScript's text item delimiters to "
"
#https://apple.stackexchange.com/questions/46521/how-do-i-find-out-the-applescript-commands-available-for-a-particular-app
#https://macosxautomation.com/applescript/sbrt/sbrt-09.html

#Get groups for all references in Endnote Library

set AppleScript's text item delimiters to "
"

## Set date for filename
set dateObj to (current date)
set theMonth to text -1 thru -2 of ("0" & (month of dateObj as number))
set theDay to text -1 thru -2 of ("0" & day of dateObj)
set theYear to year of dateObj
set dateStamp to "" & theYear & theMonth & theDay

tell application "EndNote X9"
        open "Macintosh HD:Users:yourusername:Documents:My Endnote Library.enl"
        set myGroups to get groups of type "CUSTOM" in window 3
        set finalprod to {}
        
        repeat with thegroup in myGroups
                set recordIDs to get records in thegroup in window 3
                repeat with theRecord in recordIDs
                        set title to field "Title" of record theRecord
                        set ufRecord to unformatted record theRecord
                        set the end of finalprod to {(thegroup as text) & "|" & (theRecord as text) & "|" & (ufRecord as text) & "|" & (title as text)} as text
                        
                end repeat
        end repeat
        
        ##return (finalprod)
        set finalprod to finalprod as text
        
        set this_data to finalprod
        set this_file to ((("Macintosh HD:Users:yourusername:Documents:MDBC Endnote Logs:MDBC_Endnote_group_records_" & dateStamp & ".txt") as string))
        my write_to_file(this_data, this_file, false)
        
        quit #quite Endnote
        
end tell



##Subroutine to write to text file

on write_to_file(this_data, target_file, append_data)
        try
                set the target_file to the target_file
                set the open_target_file to open for access file target_file with write permission
                if append_data is false then set eof of the open_target_file to 0
                write this_data to the open_target_file starting at eof
                close access the open_target_file
                return true
        on error
                try
                        close access file target_file
                end try
                return false
        end try
end write_to_file
```
