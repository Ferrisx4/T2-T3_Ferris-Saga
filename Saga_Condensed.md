# Tripal 2 to 3 Migration - Condensed
This will be ***the*** document to follow along if you just want to do the migration. No fluff!

### Table of Contents
1) [Pre-migration scripts](#1-pre-migration-scripts)
2) [Pre-migration manual steps](#2-pre-migration-manual-steps)
3) [The migration](#3-the-migration)
4) [Post-migration scripts](#4-post-migration-scripts)
5) [Post-migration manual steps](#5-post-migration-steps)

## 1. Pre-migration Scripts
1. Install the Migration Module (i5k). In the modules directory:
`git clone git@github.com:NAL-i5K/T2-to-T3-Upgrade-Guide.git`
2. Run the scripts (these may be combined eventually)
`sh pre-migration.sh`
`sh pre-migration_step2.sh`

## 2. Pre-migration manual steps
1. Put the site into *Maintenance Mode*
`drush vset site_offline 1` 
2. Disable Tripal 2 and related modules
`drush pm-disable tripal_core -y`
`drush pm-disable i5k_features -y`
3. Navigate to where the Tripal module is
`rm -rf tripal/`
4. Get Tripal 3
`drush pm-download tripal-7.x-3.1`
`drush pm-enable tripal`
`drush pm-enable tripal_chado`
`drush updatedb` just in case (it will complain that you should, but might not have updates)
## 3. The migration
*Most of this takes place on the website*
1. Prepare the site with Tripal and Chado
   - Alerts on the site will have you prepare the site for Tripal and Chado
2. Perform the migration *(mostly follow on-screen instructions)*
   - Navigate to `admin/tripal/storage/chado/migrate`
   - Step 1 (Ignore) - Talks about Legacy modules (not relevant on i5k)
   - Step 2 "Migrate all"
     - May take a while
   - Step 3 (Ignore) - More options for Legacy Templates
   - Step 4 (Only select first two for now)
     - [x] Copy Title over to Tripal v3 Content
     - [x] Migrate URL Alias to Tripal v3 Content
     - [ ] Unpublish Tripal v2 Content
     - [ ] Delete Tripal v2 Content 

## 4. Post-migration scripts
1. `sh post_migration.sh`
2. `sh post_migration_step2.sh`

## 5. Post Migration Steps
##### Tripal Manage Analyses
1. Enable
   - This is already done in the `post_migration.sh` script

2. Add fields
   - Navigate to the T3 Organism 'Manage Fields' page, found here:
   `admin/structure/bio_data`
   Click "Check for new fields"
   Do the same for 'Analysis' type

3. Enable fields
   - For "Organism" and "Analysis" types, navigate to their respective 'Manage Display' tabs and organize

##### HTML
The HTML formats have gotten stale (id vs machine_name). Full description [here](https://github.com/NAL-i5K/general_issues/issues/28#issuecomment-469293011)
1. Navigate to `admin/config/content/formats`
2. Rename text formats:
   - 'Filtered HTML' -> 'Filtered HTML Old'
   - 'Full HTML' -> 'Full HTML Old'
3. Recrease 'Filtered HTML' and 'Full HTML' with same options as the 'Old' versions

##### CSS/Theming
This deals with a customized version of the i5k_bootstrap theme. It styles certain fields to be italic based on biological standards. The second post_migration script removes inline `<i></i>` tags from the database where they don't belong.
Coming soon.