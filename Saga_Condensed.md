# Tripal 2 to 3 Migration - Condensed
This will be ***the*** document to follow along if you just want to do the migration. No fluff!

What this includes:
 - Upgrade Tripal from version 2 to 3
 - Upgrade Chado from version 1.2 to 1.3
 - Modify content types to display appropriate fields
 - Install tripal_manage_analysis
 - Install t3_species_glossary

What this does not include:
 - Webserver configuration/file transfer
 - Install Elasticsearch

### Table of Contents
1) [Pre-migration scripts](#1-pre-migration-scripts)
2) [Pre-migration manual steps](#2-pre-migration-manual-steps)
3) [The migration](#3-the-migration)
4) [Post-migration scripts](#4-post-migration-scripts)
5) [Post-migration manual steps](#5-post-migration-steps)
   - [Additional modules](#additional-modules)
   - [Tripal Content Type configuration](#tripal-content-type-configuration)
   - [Other Configurations](#other-configurations)

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

   - `git clone https://github.com/tripal/tripal.git`
   - Apply patches from [Tripal Install documentation](https://tripal.readthedocs.io/en/latest/user_guide/install_tripal/manual_install/install_tripal.html#apply-patches) (Postgres and Views patches)
     - This step may have to be run slightly later depending on if the `views` module has been installed yet (TESTING)
   - `drush pm-enable tripal`
   - `drush pm-enable tripal_chado`
   - `drush pm-enable tripal_core, tripal_views, tripal_db, tripal_cv, tripal_analysis, tripal_organism, tripal_feature, tripal_pub, tripal_stock, tripal_ds, tripal_ws`
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

### Additional Modules

#### Tripal Manage Analyses
1. Enable
   - This is already done in the `post_migration.sh` script. Don't worry about populating the *analysis_organism* materialized view, that is also done automatically in the above script.

2. Add fields for Organism, Analysis, Gene, mRNA, Project
   - Navigate to the T3 fields page, `admin/structure/bio_data`, and update the fields for Organism, Analysis, Gene, mRNA, Genome Assembly, and Genome Annotatin
     - Scroll to 'Organism', click 'Manage Fields', and click "Check for new fields" on the new page.
     - Return to the previous page, scroll to Analysis, click 'Manage Fields', and click "Check for new fields" on the new page.

3. Enable fields
   - For each of these content types, navigate to their respective 'Manage Display' tabs and organize the fields. This is purely a design choice at this point. An outline for this is provided in the [Tripal Content Type configuration](#Tripal-Content-Type-configuration) section.

4. Apply/reapply Tripal Default Layout
   - For certain content types, the Tripal default needs to be applied.  On the 'Manage Display' tab for those content types, click *Apply Tripal Default Layout*. 

#### Tripal HQ and Tripal EUtils
These two modules, as well as any related or supporting modules, were installed as part of the Post Installation script section.
##### Tripal Eutils
Configuration for this module is simply an optional API key from NCBI. **This will be supplied by Monica.** Providing an API key lets the module access the resource at a faster rate. The process for obtaining one if you do not have one can be obtained [here](https://ncbiinsights.ncbi.nlm.nih.gov/2017/11/02/new-api-keys-for-the-e-utilities/).

Once an API key is obtained, it may be saved to the site by browsing to `admin/tripal/tripal_eutils`.

##### Tripal HQ
Tripal HQ requires some configuration, mainly for permissions. See [this documentation](https://tripal-hq.readthedocs.io/en/latest/setup/permissions.html) for an in-depth guide.
This module also utilizes the Field Permissions module. Configuration details to come.

 ##### Tripal Alchemist
Many analyses need to be converted to their appropriate type: **Genome Assembly** or **Genome Annotation**. Use the Tripal Alchemist module to perform this task with some degree of automation:
 1. On command line, navigate to where contributed modules are (typically the same directory where the Tripal module is).
 2. `git clone git@github.com:statonlab/tripal_alchemist.git`
 3. `drush pm-enable tripal_alchemist -y`
 4. On the site, navigate to /admin/tripal/extension/tripal_alchemist
 5. Select the 'Manual' for *Transformation method* and 'Analysis' for the *Source Bundle*
 6. Select 'Genome Annotation' for the *Destination Bundle* when it appears.
 7. Check the box for all Analyses that seem to be Genome Annotations. When unsure, Annotation typically overrides Assembly:
    - `BCM annotation of the Oncopeltus fasciatus assembly using Maker and additional analyses` would be an Annotation, not an Assembly
 8. Click 'Transform (Submit)'
 9. Repeat steps 5-8 for 'Genome Assembly' type instead of 'Genome Annotation'

  The transformation step is set up to run the next time that the Drupal cron gets run, and not with a specific Tripal Job.

 ##### Gene JBrowse Fields
This module makes a field appear on Gene/Feature pages that consists of an iframe linking to the related jbrowse instance, configured by the module.
1. Install the module into your modules file
   - `git clone git@github.com:NAL-i5K/gene_jbrowse_field.git`
   - `drush pm-enable gene_jbrowse_field`
2. Navigate to the configuration page - Tripal -> Extensions -> Gene Jbrowse Field Configuration
   - In the Gene Page URL Template field, paste the following:
    `/apollo2_redirect/[Genus]_[species]/jbrowse/?loc=[child sequence name]`
   - In the mRNA Page URL Template field, paste the following:
    `/apollo2_redirect/[Genus]_[species]/jbrowse/?loc=[uniquename]`
   - Hit 'Save' 
3. Navigate to `admin/structure/bio_data` and click on "manage fields" for the Gene type.
   - Click on ```+ Check for new fields```.
   - `local__jbrowse_link` should be added, along with a few others.

Gene pages should now render with an iframe featuring the corresponding Apollo instance for that organism.

#### Tripal 3 Species Glossary
This module provides a glossary of all species on the site. This gets created as a view.
1. Navigate to the directory where you store custom/contributed modules
2. `git clone git@github.com:NAL-i5K/t3_species_glossary.git`
3. `drush pm-enable t3_species_glossary -y`
This creates a page at `/t3_species_glossary` (this can be changed later). Now we must point the main menu "Organisms" button to this page instead of `/species`.
4. Navigate to `/admin/structure/menu/manage/main-menu`
5. Click on the "edit" link for "Organisms" (second in list)
6. Change the Path to `t3_species_glossary` and click Save.

<hr>

### Tripal Content Type configuration
In order to make our pages look presentable, we need to modify the current way they are configured.

The settings for the page configuration can be found at `admin/structure/bio_data`.
Click on the **Manage Fields** and **Manage Display** links/tabs for each content type below. New groups may have to be added in order to satisfy the structure outlined below, which can be done on the corresponding Manage Display tab. 

*These two tabs operate similarly to Apache's `-available` and `-enabled` module and config system—all available fields are listed on the **Manage Fields** tab and a field is either enabled or disabled in the **Manage Display** tab. Do not try to hide a field by deleting it.\ 
  *

*Note: for all content types, we need to hide empty fields. This involves the following steps:
 - Navigate to `admin/structure/bio_data`
 - For each content type, click "Edit".
 - Check the box for 'Hide empty fields'.
 - click 'Save Content Type' at the bottom. 

** For each content type listed below **
 1. On the *Manage Display* page, the layout at the bottom may not be set correctly. Perform the following two steps:
    1. Choose 'Tripal Feature Layout' from the dropdown and Save
    2. At the top, after saving, click `+ Apply Default Tripal Layout (will reset current layout)`
 2. Manage Display tab: Use the following hierarchies to define what fields should be shown in what order and under what category (category is first level).
 3. Manage Display tab: Drag any fields not wanted to the area below the "Disabled" section (Manage Display) and click Save. *Fields that are crossed out are to be renamed.*
 4. Edit tab: check the box to `Hide empty fields` and click Save Content Type.


##### Organism
 - Summary
   - Resource Type (to be removed later, maybe)
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
   - GC Content
 - Other information
   - Community contact
   - External links

##### Analysis/Genome Assembly/Genome Annotation
 - Summary Table (not the Tripal Pane)
   - Resource Type
   - Name
   - Program, Pipeline, Workflow or Method name
   - Program Version
   - Data Source
   - Organism
   - Description
   - Algorithm

##### Gene
 - Summary
   - Resource Type
   - Name
   - Identifier
   - Organism
   - Analysis
   - Sequence Coordinates
   - Is Obsolete
   - Time Accessioned (Format as plain on Manage Fields tab)
   - Time Last Modified (Format as plain on Manage Fields tab)
 - Properties
   - Cross Reference, **rename to External Databases** (Post-migration problem due to missing db config: https://github.com/NAL-i5K/general_issues/issues/139).
   - Any fields where Format is Chado Property, *except Accession*
 - Transcripts (Format: Tripal pane)
   - Transcripts (Format: Transcript, #)
 - JBrowse (Tripal Pane)
   - JBrowse

##### mRNA
 - Summary
   - Resource Type
   - Name
   - Identifier
   - Organism
   - Analyses
   - Relationship (not the Tripal Pane)
   - Is Obsolete
   - Time Accessioned (Format as plain on Manage Fields tab)
   - Time Last Modified (Format as plain on Manage Fields tab)
 - Sequences
   - Sequence, rename to **mRNA Sequence**
   - Sequence Length, rename to **mRNA Sequence Length**
   - Sequence Coordinates, rename to **mRNA Sequence Coordinates**
   - Coding Sequence (CDS)
   - Protein Sequence
 - Properties
   - Cross References, rename to **External Databases**
   - Any fields where Format is Chado Property
 - JBrowse (Tripal Pane)
   - JBrowse

##### Biosamples (from EUtils)
 *This content type may need to have its layout reset. If this is the case, perform the following steps:*
  - On the 'Manage Display' tab for this content type, click `Apply Default Tripal Layout (will reset current layout)` followed by `Yes, apply layout`.*
  - If after clearing the cache and refreshing, the layout is still not correct (for instance only the Summary tab appears), disabling all fields in 'Manage Display' tab, saving, and re-enabling the fields may work. 

 Layout: 
  - Properties
    - ~~Full Ncbi Xml~~
    - External Databases ~~Cross references~~

 Work may be required to get Biosample pages to link to the associated Analysis page (they already link to the associated Organism).

<hr>

### Other Configurations

#### HTML
The HTML formats have gotten stale (id vs machine_name). Full description [here](https://github.com/NAL-i5K/general_issues/issues/28#issuecomment-469293011)
1. Navigate to `admin/config/content/formats`
2. Rename text formats:
   - 'Filtered HTML' -> 'Filtered HTML Old'
   - 'Full HTML' -> 'Full HTML Old'
3. Recreate 'Filtered HTML' and 'Full HTML' with same options as the 'Old' versions

#### Permissions
By default, Tripal does not automatically set content types to be viewable by anybody except for admin and tripal admin users. For content types that we want site visitors to see, we need to set these permissions. 
 1. Navigate to `admin/people/permissions` in the browser. 
 2. In the **Tripal** section, check the "ANONYMOUS USER" box for any content types you'd like users to be able to view (or otherwise access - be careful).

#### Tripal Registration
 - For reporting purposes, the Tripal team wants sites to report back that they are using Tripal and how they are using it. It is optional.
 For all development and testing purposes, we can Opt out, but for the final production version, we should register our use. To do this, follow the link that appears on every page (as admin). If this link does not appear for some reason, the site's registration status can be viewed and changed at `admin/tripal/register`.

 #### Tripal 2 Content Removal
 The final step, once the site is looking good and everything is configured, is to delete the old Tripal 2 content. 
 1. Navigate to `/admin/tripal/storage/chado/migrate`
 2. On the **Step 4** tab, check "Unpublish Tripal v2 Content" under *All*
 3. ```drush trp-run-jobs --username=admin --root=/var/www/i5k```
 4. On the **Step 4** tab, check "Delete Tripal v2 Content" under *All*
 5. ```drush trp-run-jobs --username=admin --root=/var/www/i5k``` This may take a while.