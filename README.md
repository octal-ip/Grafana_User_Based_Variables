# Grafana: User Based Variables
Sometimes there's a need to produce a variable in Grafana that's possible values are specific to the user who's logged in. This may be for convenience (so a user doesn't see what's not relevent to them on shared dashboards) or to prevent a user from seeing data that they shouldn't be able to access (e.g. so different customers can share a single dashboard or query without encountering each others data).
As there doesn't seem to be a way to assign variable values on a per-user basis natively in Grafana, this page details how to use a simple SQLite database to provide customised variable values for each user.

In the below example, usernames (as used in Grafana) are populated into the *permissions* table, referencing the site ID they should be able to access in the *site_id* column, one per row. Substitute the sites nomenclature and the "mac_address" column name with whatever best suits your purpose.

**Steps (tested on Ubuntu 24.04):**
1. Install SQLite: 
*sudo apt install sqlite3*

2. Create a database file in a suitable location:
*sqlite3 grafana_acls.db*

3. Make sure Grafana has access to this file:
*chown root:grafana grafana_acls.db*

4. Create 2 tables and an index:

    ```sql
    CREATE TABLE permissions  (
      site_id integer NOT NULL,
      "username" TEXT NOT NULL
    )

    CREATE TABLE sites  (
      id integer NOT NULL,
      "site_name" text NOT NULL,
      "mac_address" TEXT NOT NULL
    )

    CREATE UNIQUE INDEX "sites_id"
    ON "sites" ("id")
    ```

5. Populate the the *mac_address* column in the sites table with **all** the variable values you wish to have available.

6. Populate the permissions table with one row for each variable you would like to assign to each person. The following example would provide *john.smith* with the mac_addresses corresponding to id 0 (Site1 - c69773f4caf9) and id 1 (Site2 - a19074f3f5d1), and *jane.doe* with mac_address items corresponding to ids 1 (Site2 - a19074f3f5d1) and 2 (Site3 - 915d730928c5).

Permissions:

| site_id | username |
| ------------ | ------------ |
| 0 | john.smith  |
| 1 | john.smith |
| 2 | jane.doe |
| 1 | jane.doe |
| 3 | bill.citizen |

Sites:

| id | site_name | mac_address |
| ------------ | ------------ | ------------ |
| 0 | Site1 | c69773f4caf9 |
| 1 | Site2 | a19074f3f5d1 |
| 2 | Site3 | 915d730928c5 |
| 3 | Site4 | afe9156b536f |



7. Create a SQLite datasource in Grafana pointing to your SQLite file.

8. Create a variable for your dashboard with the following settings:
**Variable type:** Query
**Data source:** (your SQLite datasource)
**Query:**
```sql
SELECT sites.mac_address as "__value", sites.site_name as "__text" FROM sites INNER JOIN permissions on site_id = sites.id where permissions.username = "${__user.login}"
```

Update your queries to filter data based on the variable created above.
You will now have a list of variables customised for each user, as maintained in the permissions SQLite table.
