# sac-hxe
Getting Started with SAP HANA express edition and SAP Analytics Cloud

In this series of videos we will walk through the steps to install and configure your very own instance of SAP HANA express edition in order to make a live connection from the database model to an SAP Analytics Cloud visualization.  From installation of the software to loading and modeling the data, to creating a visualization story, this series helps you get started today using the free SAP HANA express edition and trial account with SAP Analytics Cloud. This course is for anyone just starting with SAP HANA and SAC who would like to have a sandbox environment for experimentation and learning.

Code Snippets

```
hdbsql -n localhost:39013 -u SYSTEM

ALTER SYSTEM ALTER CONFIGURATION ('webdispatcher.ini','system') 
SET ('profile','icm/HTTP/mod_0') = 'PREFIX=/, FILE=/hana/shared/HXE/profile/rewrite.txt' 
WITH RECONFIGURE;

{
	"role": {
		"name": "HA_MTA.HA_DB::analytics",
		"schema_privileges": [
			{
				"privileges": [
					"SELECT", "SELECT METADATA"
				]
			}
		]
	}
}

grant "HA_PROJECT_1"."HA_MTA.HA_DB::analytics" to HA_USER;

grant "HA_PROJECT_1::external_privileges_role" to HA_USER;

```

