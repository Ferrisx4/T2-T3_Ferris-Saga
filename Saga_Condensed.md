# Tripal 2 to 3 Migration - Condensed
This will be ***the*** document to follow along if you just want to do the migration. No fluff!
It was most recently tested with:
 - the "new codebase" (circa late February 2020)
 - a slightly older database dump (July 30, 2019, md5: 
 `77966af4479af6a1dd1b6510d61e2c70`)
 - ElementaryOS (Basically Ubuntu)
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
   - `cd T2-to-T3-Upgrade-Guide/`
   - `chmod +x *.sh`
   - `sh pre_migration.sh`
   - `sh pre_migration_step2.sh`

## 2. Pre-migration manual steps
1. Put the site into *Maintenance Mode*

   `drush vset site_offline 1` 

2. Disable Tripal 2 and related modules

 - `drush pm-disable tripal_core -y`
 - `drush pm-disable i5k_features -y`
3. Navigate to where the Tripal module is

   `rm -rf tripal/`
4. Get Tripal 3

 - ~~`drush pm-download tripal-7.x-3.3` OR `git clone https://github.com/tripal/tripal.git`~~
 - `git clone https://github.com/Ferrisx4/tripal` (until this fork gets accepted)
 - `drush pm-enable tripal`
 - `drush pm-enable tripal_chado`
 - `drush pm-enable tripal_core, tripal_views, tripal_db, tripal_cv, tripal_analysis, tripal_organism, tripal_feature, tripal_pub, tripal_stock`
 - `drush updatedb` just in case (it will complain that you should, but might not have updates)

5. Directory Creation
- ensure that the directory  `sites/default/files/tripal_organism` exists and is writable by the webserver within the Drupal installation
## 3. The migration
*Most of this takes place on the website*
1. Prepare the site with Tripal and Chado
   - Alerts on the site will have you prepare the site for Tripal and Chado, or navigate to `admin/tripal/storage/chado/prepare` and follow the on-screen instructions
2. Upgrade the site with Chado 1.3 (Currently 1.2)
   - Upgrade page: `http://i5k2.local/admin/tripal/storage/chado/install`
3. Perform the migration *(mostly follow on-screen instructions)*
   - Navigate to `admin/tripal/storage/chado/migrate`
   - Step 1 (Ignore) - Talks about Legacy modules (not relevant on i5k)
   - Step 2 "Migrate all"
     - May take a while
   - Step 3 (Ignore) - More options for Legacy Templates
   - Step 4 (Only select first two of "All" for now)
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

2. Add fields for Organism and Analysis
   - Navigate to the T3 fields page, `admin/structure/bio_data`, and update the fields for Organism and Analysis
   - Scroll to 'Organism', click 'Manage Fields', and click "Check for new fields" on the new page.
   - Return to the previous page, scroll to Analysis, click 'Manage Fields', and click "Check for new fields" on the new page.

3. Enable fields
   - For "Organism" and "Analysis" types, navigate to their respective 'Manage Display' tabs and organize the fields. This is purely a design choice at this point.

##### Gene JBrowse Fields
This module makes a field appear on Gene/Feature pages that consists of an iframe linking to the related jbrowse instance, configured by the module.
1. Install the module into your modules file
   - `git clone git@github.com:NAL-i5K/gene_jbrowse_field.git`
   - `drush pm-enable gene_jbrowse_field`
2. Navigate to the configuration page - Tripal -> Extensions -> Gene Jbrowse Field Configuration


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