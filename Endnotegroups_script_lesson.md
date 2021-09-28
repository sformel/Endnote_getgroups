## Get Group Metadata From Endnote

####Using Apple Scripts

Endnote allows you to create rich metadata by the creation of groups, but doesn't make it easy to export the group information.  I've been managing a shared library that is being used by about 30 people. I needed a way to create a daily log of which records were in which groups so I could make repairs to the library when mistakes were made.  I also needed to be able to generate sumary statistics to understand how many unqiue records were in each group, and how many records were shared, and which groups they were shared by.

Endnote has an API to get some record information, but the documentation is extremely sparse, and no matter how hard I tried, I couldn't get the download link to appear.  They also have apple scripts, which I had never used before.  Endnote offers a few bare bones examples in their help documentation but like the API, doesn't seem interested in answering questions or supporting the use of Apple Scripts.  

In the end, my need for a solution (after several very tedious repairs to the library) overwhelmed my need to use my time for something other than learning apple scripts. Hopefully I help make someone else's life easier by sharing this script.

Here are some links that helped me out: 

- [Apple Script Dictionaries](https://apple.stackexchange.com/questions/46521/how-do-i-find-out-the-applescript-commands-available-for-a-particular-app)
- [How to write function to write to text file](https://macosxautomation.com/applescript/sbrt/sbrt-09.html)

#### A few notes
* I've successfiully used this in Endnote X9, I wouldn't be surprised if it doesn't work for all versions.
* I was unable to automate this with "launch agents", another mac tool for automation.  Endnote didn't like the commands being run from the terminal as opposed to the script editor and would reject the attempt to open Endnote.
* I bring this output into R for comparison and stats, but you could just as easily work with it in Python or other languages.
* I tried to run this on the shared library while I was logged in as another user instead of as the owner.  Endnote won't allow this, you must be the library owner to run these queries.

Set the default delimiter to newline

```set AppleScript's text item delimiters to "\n"```

Set date for filename

```set dateObj to (current date)
set theMonth to text -1 thru -2 of ("0" & (month of dateObj as number))
set theDay to text -1 thru -2 of ("0" & day of dateObj)
set theYear to year of dateObj
set dateStamp to "" & theYear & theMonth & theDay
```

Open Endnote and open library named "My Endnote Library" in path: Macintosh HD/Users/stephenformelnoaa/Documents/My Endnote Library.enl
 
```
tell application "EndNote X9"
open "Macintosh HD:Users:stephenformelnoaa:Documents:My Endnote Library.enl"
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
	set this_file to ((("Macintosh HD:Users:stephenformelnoaa:Documents:MDBC Endnote Logs:MDBC_Endnote_group_records_" & dateStamp & ".txt") as string))
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


Entire script.  Run in Mac Program named "Script Editor" (Applicatoins > Utilities).  To view progress: View > Show log

```
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
	open "Macintosh HD:Users:stephenformelnoaa:Documents:My Endnote Library.enl"
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
	set this_file to ((("Macintosh HD:Users:stephenformelnoaa:Documents:MDBC Endnote Logs:MDBC_Endnote_group_records_" & dateStamp & ".txt") as string))
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
