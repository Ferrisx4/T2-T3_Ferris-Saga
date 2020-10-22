# Tripal 2 to 3 Migration - Condensed
This will be ***the*** document to follow along if you just want to do the migration. No fluff!

### Table of Contents
1) [Pre-migration scripts](#1-pre-migration-scripts)
2) [Pre-migration manual steps](#2-pre-migration-manual-steps)
3) [The migration](#3-the-migration)
4) [Post-migration scripts](#4-post-migration-scripts)
5) [Post-migration manual steps](#5-post-migration-steps)

<hr>

## 1. Pre-migration Scripts
1. Install the Migration Module (i5k). In the modules directory:

   `git clone git@github.com:NAL-i5K/T2-to-T3-Upgrade-Guide.git`
2. Run the scripts (these may be combined eventually)
   - `cd T2-to-T3-Upgrade-Guide/`
   - `chmod +x *.sh`
   - `./pre_migration.sh`
   - `./pre_migration_step2.sh`

<hr>

## 2. Pre-migration manual steps
1. Put the site into *Maintenance Mode*

   - `drush vset site_offline 1` 

2. Delete all Views integrations
   - Navigate to `admin/tripal/views-integration`
   - Click **Delete ALL Integrations** and confirm

3. Disable Tripal 2 and related modules
   - `drush pm-disable tripal_core -y`
   - `drush pm-disable i5k_features -y`
4. Navigate to where the Tripal module is

   `rm -rf tripal/`
5. Get Tripal 3

   - ~~`drush pm-download tripal-7.x-3.3` OR `git clone https://github.com/tripal/tripal.git`~~
   - `git clone https://github.com/Ferrisx4/tripal` (until this fork gets accepted)
   - `drush pm-enable tripal`
   - `drush pm-enable tripal_chado`
   - `drush pm-enable tripal_core, tripal_views, tripal_db, tripal_cv, tripal_analysis, tripal_organism, tripal_feature, tripal_pub, tripal_stock, tripal_ds`
   - `drush updatedb` just in case (it will complain that you should, but might not have updates)

6. Directory Creation
   - ensure that the directory  `sites/default/files/tripal_organism` exists and is writable by the webserver within the Drupal installation
## 3. The migration
*Most of this takes place on the website. When prompted, you should spawn jobs via Drush on the command line. This provides the benefit of seeing the results/errors of jobs as they run. The Tripal Jobs page is not reliable for reporting job activity and progress.*
1. Prepare the site with Tripal and Chado
   - Alerts on the site will have you prepare the site for Tripal and Chado, or navigate to `admin/tripal/storage/chado/prepare` and follow the on-screen instructions
2. Upgrade the site with Chado 1.3 (Currently 1.2)
   - Upgrade page: `admin/tripal/storage/chado/install`

3. Database Cleanup (Manual)
   - Log into the database (psql, phppgadmin, etc.)
   - Delete from the `tripal_entity` table any rows where the title has "test" or TEST"
     - *These have been causing errors on certain versions of the database. We may be able to remove this step later on*
4. Perform the migration *(mostly follow on-screen instructions)*
   - Navigate to `admin/tripal/storage/chado/migrate`
   - Step 1 (Ignore) - Talks about Legacy modules (not relevant on i5k)
   - Step 2 "Migrate all"
     - May take a while
   - Step 3 (Ignore) - More options for Legacy Templates
   - Step 4 (Only select first two of "All" for now) - *Note: this may take very long*
     - [x] Copy Title over to Tripal v3 Content
     - [x] Migrate URL Alias to Tripal v3 Content
     - [ ] Unpublish Tripal v2 Content
     - [ ] Delete Tripal v2 Content 

<hr>

## 4. Post-migration scripts
1. `./post_migration.sh`
2. `./post_migration_step2.sh`

<hr>

## 5. Post Migration Steps
#### Tripal Manage Analyses
1. Enable
   - This is already done in the `post_migration.sh` script

2. Add fields for Organism, Analysis, Gene, mRNA, Project
   - Navigate to the T3 fields page, `admin/structure/bio_data`, and update the fields for Organism, Analysis, Gene, and mRNA
     - Scroll to 'Organism', click 'Manage Fields', and click "Check for new fields" on the new page.
     - Return to the previous page, scroll to Analysis, click 'Manage Fields', and click "Check for new fields" on the new page.

3. Enable fields
   - For each of these content types, navigate to their respective 'Manage Display' tabs and organize the fields. This is purely a design choice at this point. An outline for this is provided in the [Tripal Content Type configuration](#Tripal-Content-Type-configuration) section.

4. Apply/reapply Tripal Default Layout
   - For certain content types, the Tripal default needs to be applied.  On the 'Manage Display' tab for those content types, click *Apply Tripal Default Layout*. 

#### Gene JBrowse Fields
This module makes a field appear on Gene/Feature pages that consists of an iframe linking to the related jbrowse instance, configured by the module.
1. Install the module into your modules file
   - `git clone git@github.com:NAL-i5K/gene_jbrowse_field.git`
   - `drush pm-enable gene_jbrowse_field`
2. Navigate to the configuration page - Tripal -> Extensions -> Gene Jbrowse Field Configuration
   - In the URL Template field, paste the following:
    `https://apollo.nal.usda.gov/apollo/[Genus]%20[species]/jbrowse/?loc=[gene name]-RA` 
    and click Save. 
3. Navigate to `admin/structure/bio_data` and click on "manage fields" for the Gene type.
   - Click on ```+ Check for new fields```.
   - `local__jbrowse_link` should be added, along with a few others.

Gene pages should now render with an iframe featuring the corresponding Apollo instance for that organism.

<hr>

#### Tripal 3 Species Glossary
This module provides a glossary of all species on the site. This gets created as a view.
1. Navigate to the directory where you store custom/contributed modules
2. `git clone git@github.com:NAL-i5K/t3_species_glossary.git`
3. `drush pm-enable t3_species_glossary -y`
This creates a page at `/t3_species_glossary` (this can be changed later). Now we must point the main menu "Organisms" button to this page instead of `/species`.
4. Navigate to `/admin/structure/menu/manage/main-menu`
5. Click on the "edit" link for "Organisms" (second in list)
6. Change the Path to `t3_species_glossary` and click Save

<hr>

#### Tripal Content Type configuration
In order to make our pages look presentable, we need to modify the current way they are configured.

The settings for the page configuration can be found at `admin/structure/bio_data`.
Click on the **Manage Fields** and **Manage Display** links/tabs for each content type below. New groups may have to be added in order to satisfy the structure outlined below, which can be done on the corresponding Manage Display tab.

 1. On the *Manage Display* page, the layout at the bottom may not be set correctly. **If and only if** this is the case, perform the following two steps
    1. Choose 'Tripal Feature Layout' from the dropdown and Save
    2. At the top, after saving, click `+ Apply Default Tripal Layout (will reset current layout)`
 2. Manage Display tab: Use the following hierarchies to define what fields should be shown in what order and under what category (category is first level).
 3. Manage Display tab: Drag any fields not wanted to the area below the "Disabled" section (Manage Display).

##### Organism
 - Summary
   - Abbreviation (?)
   - Genus
   - Species
   - Common name
   - Description
   - Image
   - Image credit and license/attribution
 - Analyses
   - Link
 - Assembly Stats
   - Contig N50
   - Scaffold N50
   - Number of Genes
   - GC Content
 - Other information
   - Community contact
   - External links

##### Analysis
 - Summary
   - Analysis name
   - Software
   - Materials and Methods
   - Organism

##### Gene
 - Content goes here

##### mRNA
 - Content goes here

##### Biosamples (from EUtils)
 *This content type may need to have its layout reset. If this is the case, perform the following steps:*
  - On the 'Manage Display' tab for this content type, click `Apply Default Tripal Layout (will reset current layout)` followed by `Yes, apply layout`.*
  - If after clearing the cache and refreshing, the layout is still not correct (for instance only the Summary tab appears), disabling all fields in 'Manage Display' tab, saving, and re-enabling the fields may work. 

 Work may be required to get Biosample pages to link to the associated Analysis page (they already link to the associated Organism).

#### HTML
The HTML formats have gotten stale (id vs machine_name). Full description [here](https://github.com/NAL-i5K/general_issues/issues/28#issuecomment-469293011)
1. Navigate to `admin/config/content/formats`
2. Rename text formats:
   - 'Filtered HTML' -> 'Filtered HTML Old'
   - 'Full HTML' -> 'Full HTML Old'
3. Recrease 'Filtered HTML' and 'Full HTML' with same options as the 'Old' versions

#### CSS/Theming
This deals with a customized version of the i5k_bootstrap theme. It styles certain fields to be italic based on biological standards. The second post_migration script removes inline `<i></i>` tags from the database where they don't belong.
The CSS used for this site will be responsible for applying italics to these strings.

#### Permissions
By default, Tripal does not automatically set content types to be viewable by anybody except for admin and tripal admin users. For content types that we want site visitors to see, we need to set these permissions. 
 1. Navigate to `admin/people/permissions` in the browser. 
 2. In the **Tripal** section, check the "ANONYMOUS USER" box for any content types you'd like users to be able to view (or otherwise access - be careful).
    - *Organism:* View Content
