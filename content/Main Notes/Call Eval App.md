[[Google Spreadsheet Code for Git hub]]

Status:

Tags:[[Streamlit]] [[python]] [[Github]] [[Google Script]]


# Call Eval App

The call eval app flow: Transcripts are uploaded by the user to [[Google|google]] drive. It then is sent to a private google drive and using [[Google workplace studio]] it asks a gem, gets graded by our latest call eval ai gem, then it is sent to a private spreadsheet. The private spread sheet uses two google scripts two organize it. Once it is organized it sends it to a shared sheet. 

We now can go two different routes, one keeping it all in the spreadsheet [[Google Spreadsheet Code for Git hub]] this route send the google spreadsheet to an App for further insights. Eventually we will run out of room in the google spreadsheet.

The 2nd option uses supabase. This uses postgresql as a secure data base. So a google script sends it to the [[Database|database]], then is uploaded to the app. This allows us to delete and clear the spreadsheet without losing past data.



### References
[[Supabase]] 
[[Google workplace studio]]
[[Google Spreadsheet Code for Git hub]]
[[Supabase Streamlit Code Call eval]]
[[gs sync sheet with itself]]
[[moves files to private folder]]
[[gs sync sheet to supabase]]
[[gs private for data transfer]]
[[coaching feedback data transfer gs]]
