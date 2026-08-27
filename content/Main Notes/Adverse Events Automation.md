2026-08-27 10:50

Status:

Tags: [[Github]][[DP13]] [[Google]] [[Google Script]]


# Adverse Events Automation

This is a code that will fetch transcripts once every hour from five9 ai insights, download them, then upload into a google drive folder. When it hits that folder a google script will take it to a private folder, archiving the old one. When it hits the private folder google studio flows will then take that transcript, then it will "ask a gem" which goes to our latest adverse events tool where it will identify if there are adverse events or product complaints. Then it goes to a google spreadsheet where another google script sends it to a shared sheet to be reviewed by humans. [[Google workplace studio]] only works if everything is private. 



### References
[[fetch_transcripts.py]]
[[gitignore]]
[[requirements for adverse auto]]
[[yaml timer]]
[[Automove gs adverse]]
[[spreadsheet hoper gs adverse]]