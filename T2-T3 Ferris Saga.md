# Sean's attempt at T2 to T3 Migration
Following [Bradford's Guide](https://github.com/NAL-i5K/T2-to-T3-Upgrade-Guide)

## Table of Contents

1. [Replicate the i5k Environment](#Replicate_the_i5k_environment)
   - [Webserver & Database](#Webserver_&_Database)
   - [Appearance](#Appearance)
2. [Actual Chado Update](#Actual_Chado_Update)

<hr>

<a name="Replicate_the_i5k_environment"></a>

## Replicate i5k Environment

<a name="Webserver_&_Database"></a>

### Webserver & Database
- Install Drupal 7, Drush 8
- Ensure webserver has appropriate rewrite rules for Drupal and .htaccess works (if using Apache)
- Create database user and database in Postgres, give access **Do not use these values in production!**
    ```sql
        create user tripal with password 'tripal';
        create database tripal;
        alter database tripal owner to tripal;
        grant all on database "tripal" to tripal;
    ```

- Import database
  ```bash
  psql -U tripal -d tripal -f sanitized_for_cg.db.sql -h localhost
  ```

- Install missing modules (any drush command that you issue will complain about them)
   - diff
   - file_entity
   - flexslider
   - flexslider_fields
   - flexslider_views
   - flexslider_views_slideshow
   - media
   - menu_block
   - views_slideshow
   - views_slideshow_simple_pager
   ```bash
   > drush pm-enable <modulename> -y
   ```
- Change the password of user 1 (uid=1)
  ```bash
  drush user-password admin --password="very_secure_password"
  ```

<a name="Appearance"></a>

### Appearance

Many appearance issues are apparent after a fresh install. This is due to some remote content and also to some content that returns 404 without intervention.
 
 1. Remote Content
 2. 404 Content
    - These files were not included in the git repository (everything in `sites/default/files` was left out) 


<hr>

<a name="Actual_Chado_Update"></a>

## Actual Chado Update
- Run modified premigration scripts (which points at modified scripts/generate_xxx.sh script)
  - pre_migration.sh
  - pre_migration_step2.sh

Attempting to run job after:
```
Calling: tripal_core_install_chado(Upgrade Chado v1.2 to v1.3, 537)
Checking for existing v1.3 tables in v1.2 and fixing bigints...
Incorporating additional changes...
Loading sites/all/modules/contrib/tripal/tripal_core/chado_schema/default_schema-1.2-1.3-diff.sql...
WD tripal_core: FAILED. Line  1926, 0                                                                                                                                                     [error]
SQLSTATE[42804]: Datatype mismatch: 7 ERROR:  operator class "int4_ops" does not accept data type bigint:
ALTER TABLE featuregroup 
    ALTER featuregroup_id TYPE bigint,
    ALTER subject_id TYPE bigint,
    ALTER object_id TYPE bigint,
    ALTER group_id TYPE bigint,
    ALTER srcfeature_id TYPE bigint,
    ALTER fmin TYPE bigint,
    ALTER fmax TYPE bigint;

> [site http://test.i5k.local] [TRIPAL ERROR] [TRIPAL_CORE] FAILED. Line  1926, 0SQLSTATE[42804]: Datatype mismatch: 7 ERROR:  operator class "int4_ops" does not accept data type bigint:ALTER TABLE featuregroup     ALTER featuregroup_id TYPE bigint,    ALTER subject_id TYPE bigint,    ALTER object_id TYPE bigint,    ALTER group_id TYPE bigint,    ALTER srcfeature_id TYPE bigint,    ALTER fmin TYPE bigint,    ALTER fmax TYPE bigint;
```


Some other person had issue:
https://www.gitmemory.com/issue/tripal/tripal/970/509397593

Indexes in the database related to featuregroup:
```
tripal=# select indexname from pg_indexes where indexname like '%featuregroup%';
     indexname     
-------------------
 featuregroup_c1
 featuregroup_pkey
 featuregroup_idx1
 featuregroup_idx2
 featuregroup_idx3
 featuregroup_idx4
 featuregroup_idx5
 featuregroup_idx6
(8 rows)
```

Apparently this works: (from [here](https://github.com/tripal/tripal/issues/970))
Modified below: 
```
set search_path=public,so,frange,genetic_code,chado;

select 'drop index ' || schemaname || '.' || indexname || ';'
from pg_indexes where indexname like '%idx%' AND schemaname = 'chado';
```

This drops all the indexes that it can. This is useful and necessary because the `featuregroup` indexes weren't the only ones tripping up the process

**TODO: DETERMINE IF THIS CAUSES ISSUES** (for example, missing/inaccessible data)

¿Do we need to modify the Full and Filtered HTML filters, described [here](https://github.com/NAL-i5K/general_issues/issues/28)? **YES**

Delete Views Integrations (There are duplicates which messes up the Tripal upgrade)
 - Navigate to `/admin/tripal/views-integration`
 - Click **Delete ALL Integrations** and confirm
<hr>

## Tripal Upgrade
Following [the Official Guide](https://tripal.readthedocs.io/en/latest/user_guide/install_tripal/upgrade_from_tripal2.html)

Step 3 (`drush pm-disable tripal_core`) spit out a number of warnings about views it couldn't disable
```
Unable to find a view by the name of 'tripal_library_user_library'. Unable to disable this view.
Unable to find a view by the name of 'tripal_library_admin_libraries'. Unable to disable this view.
Unable to find a view by the name of 'tripal_contact_user_contacts'. Unable to disable this view.
Unable to find a view by the name of 'tripal_contact_admin_contacts'. Unable to disable this view.
Unable to find a view by the name of 'tripal_bulk_loading_jobs'. Unable to disable this view.
Unable to find a view by the name of 'tripal_bulk_loader_templates'. Unable to disable this view.
Unable to find a view by the name of 'tripal_feature_user_feature'. Unable to disable this view.
Unable to find a view by the name of 'tripal_feature_admin_features'. Unable to disable this view.
Unable to find a view by the name of 'tripal_analysis_user_analyses'. Unable to disable this view.
Unable to find a view by the name of 'tripal_analysis_admin_analyses'. Unable to disable this view.
Unable to find a view by the name of 'tripal_organism_user_organisms'. Unable to disable this view.
Unable to find a view by the name of 'tripal_organism_admin_organisms'. Unable to disable this view.
Unable to find a view by the name of 'tripal_cv_admin_cvs'. Unable to disable this view.
Unable to find a view by the name of 'tripal_cv_admin_cvterms'. Unable to disable this view.
Unable to find a view by the name of 'tripal_db_admin_dbs'. Unable to disable this view.
Unable to find a view by the name of 'tripal_db_admin_dbxrefs'. Unable to disable this view.
```

Next step is to delete the old tripal directory - but what subdirectories, precisely?
(i5k had the bulk of the Tripal modules in a subdirectory, but some leaked out)
```
tripal=# select filename, name from system where type='module' and name like '%tripal%' order by filename asc;
                                         filename                                          |           name           
-------------------------------------------------------------------------------------------+--------------------------
 sites/all/modules/contrib/tripal/tripal_analysis/tripal_analysis.module                   | tripal_analysis
 sites/all/modules/contrib/tripal/tripal_bulk_loader/tripal_bulk_loader.module             | tripal_bulk_loader
 sites/all/modules/contrib/tripal/tripal_contact/tripal_contact.module                     | tripal_contact
 sites/all/modules/contrib/tripal/tripal_core/tripal_core.module                           | tripal_core
 sites/all/modules/contrib/tripal/tripal_cv/tripal_cv.module                               | tripal_cv
 sites/all/modules/contrib/tripal/tripal_db/tripal_db.module                               | tripal_db
 sites/all/modules/contrib/tripal/tripal_example/tripal_example.module                     | tripal_example
 sites/all/modules/contrib/tripal/tripal_featuremap/tripal_featuremap.module               | tripal_featuremap
 sites/all/modules/contrib/tripal/tripal_feature/tripal_feature.module                     | tripal_feature
 sites/all/modules/contrib/tripal/tripal_genetic/tripal_genetic.module                     | tripal_genetic
 sites/all/modules/contrib/tripal/tripal_library/tripal_library.module                     | tripal_library
 sites/all/modules/contrib/tripal/tripal_natural_diversity/tripal_natural_diversity.module | tripal_natural_diversity
 sites/all/modules/contrib/tripal/tripal_organism/tripal_organism.module                   | tripal_organism
 sites/all/modules/contrib/tripal/tripal_phenotype/tripal_phenotype.module                 | tripal_phenotype
 sites/all/modules/contrib/tripal/tripal_phylogeny/tripal_phylogeny.module                 | tripal_phylogeny
 sites/all/modules/contrib/tripal/tripal_phylogeny/tripal_phylogeny.module                 | tripal_phylogeny
 sites/all/modules/contrib/tripal/tripal_project/tripal_project.module                     | tripal_project
 sites/all/modules/contrib/tripal/tripal_pub/tripal_pub.module                             | tripal_pub
 sites/all/modules/contrib/tripal/tripal_stock/tripal_stock.module                         | tripal_stock
 sites/all/modules/contrib/tripal/tripal_views/tripal_views.module                         | tripal_views
 sites/all/modules/tripal_analysis_blast/tripal_analysis_blast.module                      | tripal_analysis_blast
 sites/all/modules/tripal_analysis_interpro/tripal_analysis_interpro.module                | tripal_analysis_interpro
 sites/all/modules/tripal_analysis_kegg/tripal_analysis_kegg.module                        | tripal_analysis_kegg
(23 rows)
```

Issuing any drush command reveals that the `i5k_features` module depended on the `tripal_core/includes/tripal_core.toc.inc`

Also disable the i5k_features module when disabling tripal core:
drush pm-disable tripal_core
drush pm-disable i5k_features

~~Disable that module via the database: (don't worry about re-enabling it later, its functionality is deprecated in Tripal 3)~~
> ~~update system set status='0' where name='i5k_features';~~

~~Clear the database cache (drush still won't work at this point):~~
> ~~DELETE FROM cache_bootstrap WHERE cid='system_list';~~

Continue to follow along with the Official Guide
  Navigate to the directory where the tripal module is stored: `sites/all/modules/contrib`
> rm -rf tripal/<br>
> drush pm-download tripal-7.x-3.1<br>
> drush pm-enable tripal<br>
> drush pm-enable tripal_chado
  - requires `redirect` module
  - warnings:
    > The block Dashboard Notifications was assigned to the invalid region dashboard_main and has been disabled.
    > The block Published Tripal Content was assigned to the invalid region dashboard_main and has been disabled.

> drush pm-enable tripal_core, tripal_views, tripal_db, tripal_cv, tripal_analysis, tripal_organism, tripal_feature, tripal_pub, tripal_stock
<br>
> drush pm-enable tripal_ds tripal_ws

Now is as good of time as any to update the database:
> drush updatedb

The `tripal_daemon` module was not installed, no need to remove it before re-enabling it (if desired):
(it is not part of Tripal core)

In `sites/all/libraries`:
> git clone https://github.com/shaneharter/PHP-Daemon.git

> drush pm-enable tripal_daemon

Trigger the Prepare Drupal and Chado task via the GUI. 
"(This may take a while. Go get some runts.)"

Seems to have gone well

Site complains:
>Could not find details about the vocabulary: SOFP. Note: if this vocabulary does exist, try re-populating the db2cv_mview materialized view at Admin > Tripal > Data Storage > Chado > Materialized views.

<hr>

## Migrating Content

As per the instructions, use the GUI to migrate the content into Tripal 3 entities.

**Warning**: `sites/default/files/tripal_organism` must exist and be writable.

- (The following warnings are likely caused by importing the same type (organism) more than once, oops).

 **TODO: TEST THIS ON NEXT TRIAL**

```
WD tripal_chado: PDOException: SQLSTATE[23505]: Unique violation: 7 ERROR:  duplicate key value violates unique constraint [error]
"field_data_bio_data_1_resource_links_pkey"
DETAIL:  Key (entity_type, entity_id, deleted, delta, language)=(TripalEntity, 56232, 0, 4, und) already exists.: 
        INSERT INTO field_data_bio_data_1_resource_links (entity_type, bundle, entity_id, revision_id, language, delta,
bio_data_1_resource_links_url, bio_data_1_resource_links_title)
        VALUES (:entity_type, :bundle, :entity_id, :revision_id, :language, :delta, :url, :title)
        ; Array
(
    [:entity_type] => TripalEntity
    [:bundle] => bio_data_1
    [:entity_id] => 56232
    [revision_id] => 56232
    [:language] => und
    [:delta] => 4
    [:url] => /node/739391
    [:title] => Annotation1
)
 in tripal_chado_migrate_resource_links() (line 1216 of
/var/www/i5k/i5k-tripal/sites/all/modules/contrib/tripal/tripal_chado/includes/tripal_chado.migrate.inc).
```

The rest of the migration runs smoothly (steps 2 and each part of 4, skipping the not-recommended step 3). Some of the steps do take a while to run (10-30 minutes).
 - Occasional warnings about the `db2cv_mview` materialized view missing the SOFP vocabulary. Repopulating that mview makes the warning go away. **TODO**: investigate if there is an issue here.

Deleting tripal v2 content (part four of *Step 4*)

 - Issues with Solr:

    ``` 
    Verifying chado_analysis records...
    Deleted 0 record(s) from chado_analysis missing either a node or chado entry.
    Verifying nodes...
    WD Apache Solr: Environment solr; HTTP Status: 0; Message: Request failed: Connection refused; Response: ; Request:        [error]
    Unknown; Caller: module_invoke_all() (line 965 of /var/www/i5k/i5k-tripal/includes/module.inc)
    WD Apache Solr: Exception caught for function DrupalApacheSolrService->_sendRawPost() (line 542 of                         [error]
    /var/www/i5k/i5k-tripal/sites/all/modules/contrib/apachesolr/Drupal_Apache_Solr_Service.php), Environment solr: HTTP 0;
    Request failed: Connection refused
    WD Apache Solr: Environment localfiles_index; HTTP Status: 0; Message: Request failed: Connection refused; Response: ;     [error]
    Request: Unknown; Caller: module_invoke_all() (line 965 of /var/www/i5k/i5k-tripal/includes/module.inc)
    WD Apache Solr: Exception caught for function DrupalApacheSolrService->_sendRawPost() (line 542 of                         [error]
    /var/www/i5k/i5k-tripal/sites/all/modules/contrib/apachesolr/Drupal_Apache_Solr_Service.php), Environment localfiles_index:
    HTTP 0; Request failed: Connection refused
    ```

 - Not necessarily a showstopper, progress seems to continue via localfiles_index
   - Very slow, however (20 minutes = 20% progress checking the 50000+ nodes)


## Additional Modules

- Tripal Manage Analyses
  - **Unable to navigate to `admin/structure/bio_data/manage`**
- Tripal HQ
  - Also installs Field Permissions module (contrib, recommended)

## Post-migration Script
- Forgot to run this, perhaps it should be done sooner?

- Many errors were to be had:
  ```
  array_flip(): Can only flip STRING and INTEGER values! entity.inc:175 [warning]
  array_flip(): Can only flip STRING and INTEGER values! entity.inc:388 [warning]
  ```
- previous two: 162x (one for each organism), from convert_node_links_to_chado.php)
  ```
  Error looking up analysis for nid 739169.  Continuing... 
  ```
- (last one: 57 different nids, from convert_sourceuris_from_nodes.php)

<hr>

# Manual Steps
Certain parts of how Tripal 3 entities are displayed are not easily automated by scripts or modules. Here we apply those agreed-upon changes.

### General Steps

1) Follow instructions [here](https://github.com/statonlab/tripal_manage_analyses#installation  ) to add the fields to organism and analysis

### Organism display

1) Create new group - "Other Information"
   - "Tripal Pane" format to be hidden on pageload
   - "Fieldset" format to always show
2) Enable "Links"
   - put in *Other Information* pane

### HTML Content
Alter the Full HTML and Filtered HTML formats following the instructions [here](https://github.com/NAL-i5K/general_issues/issues/28)

### Style / Theme
This section exists while the i5k_bootstrap theme is still integrated into the overall codebase (and not its own repository)

A patch can be found in this repository to apply to the site's code. Leaving this as a manual step so as not to override any other changes to the i5k_bootstrap theme in the interim.
To apply this patch, run this command:
`TODO what command to run/how do I into patches`

### Permissions
Review the permissions at `admin/people/permissions`. Most importantly, "View Content" permissions must be checked for Anonymous users for every Tripal Content Type. By default, only "Tripal Administrators" are given this permission.