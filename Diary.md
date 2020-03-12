# Attempts at Migration
Several have gone before. This is one that started on

## February 18, 2020

Thing
 - [x] Clone the repository
 - [x] Manually install all of the missing modules
 - [x] Copy `sites/all/files` that weren't included
 - [ ] Some stuff ()
 - [x] Prepare Chado - complain about Analysis
 - [x] Migrate ALL content types (GUI Step 2)- no errors
 - [ ] GUI Step 3 - Optional
   - Don't do this. Tested it with the Organism type, the pages turn out blank.
 - [ ] GUI Step 4
   - [x] ALL - Copy Title over to Tripal v3 Content
     - No change?
   - [x] ALL - Migrate URL Alias to Tripal v3 Content
   - [ ] ALL - Unpublish Tripal v2 Content
   - [ ] ALL - Delete Tripal v2 Content

## February 28, 2020

General Notes
 - It's not complaininga bout all the missing modules like last time.

Progress
 - [x] Installed database, copied over `sites/default/files` from the archive
 - [ ] Some other things

## Regular site images
Some images were provided by Monica (hero images, about us portraits, etc).
##### Still missing About Us portraits:
 - Chris
 - Leo Hsiao
 - LiMei Chiang
 - Susan McCarthy

##### Hero Images
For some reason, Hero images (typically wide banners near top of page) were mis-named in the database. For example:
 - On disk: `aboutus.png`
 - In ddb:  `aboutus_1.png`

Sometimes the thumbnail on the 'edit' screen was correct, but the one used for the actual page view was not correct.
*shrug* 
|Page   |Expected file (db)   |On disk|
|-|-|-|
|Tools|sites/default/files/tools_0.png|tools.png|
|About Us|sites/default/files/aboutus_2.png|aboutus.png, aboutus_0.png|
|Contact us|**sites/all/themes/i5k_bootstrap/images/contactus.png* (ok)|contactus.png|
|Data|sites/default/files/data_0.png|data.png|
|Organisms|sites/default/files/organisms_3.png|organisms.png|
|Tutorials|sites/default/files/tutorials_0.png|tutorials.png|
**this and the searchbg.jpg (nice diamond tile image) are the only files here*