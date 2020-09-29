# sac-hxe
Getting Started with SAP HANA express edition and SAP Analytics Cloud

HANA Data SQL

```
create table HA_DATA.PRODUCT (
   PRODUCT_ID BIGINT not null,
   PRODUCT_NAME VARCHAR(128),
   COLOR VARCHAR(64),
   SIZE VARCHAR(32),
   DEMOGRAPHIC VARCHAR(32),
   PRICE DECIMAL(8,2),
   PRODUCT_TYPE_ID BIGINT,
   PRODUCT_CLASS VARCHAR(64),
   SUPPLIER_ID BIGINT
);

alter table HA_DATA.PRODUCT
   add constraint PK_PRODUCT primary key cpbtree (PRODUCT_ID);


create table HA_DATA.ORDER_DETAIL (
   ORDER_ID BIGINT not null,
   PRODUCT_ID BIGINT not null,
   QUANTITY	 INTEGER
);

alter table HA_DATA.ORDER_DETAIL
   add constraint PK_ORDER_DETAIL primary key cpbtree (ORDER_ID, PRODUCT_ID);

create index RELATIONSHIP_1_FK on HA_DATA.ORDER_DETAIL (PRODUCT_ID);


alter table HA_DATA.ORDER_DETAIL
   add constraint FK_ORDER_RELATIONS_PRODUCT foreign key (PRODUCT_ID)
      references HA_DATA.PRODUCT (PRODUCT_ID) on delete restrict on update restrict;


IMPORT FROM CSV FILE '/mnt/hgfs/SampleData/Product.csv' INTO HA_DATA.PRODUCT
WITH
   RECORD DELIMITED BY '\n'
   FIELD DELIMITED BY ','
   OPTIONALLY ENCLOSED BY '"'
   SKIP FIRST 1 ROW
   FAIL ON INVALID DATA
   ERROR LOG '/mnt/hgfs/SampleData/Product.csv.err'
;

IMPORT FROM CSV FILE '/mnt/hgfs/SampleData/Orders_Detail.csv' INTO HA_DATA.ORDER_DETAIL
WITH
   RECORD DELIMITED BY '\n'
   FIELD DELIMITED BY ','
   OPTIONALLY ENCLOSED BY '"'
   SKIP FIRST 1 ROW
   FAIL ON INVALID DATA
   ERROR LOG '/mnt/hgfs/SampleData/Orders_Detail.csv.err'
;



create table HA_DATA.CUSTOMER (
	CUSTOMER_ID BIGINT not null,
	NAME VARCHAR(64),
	CONTACT_FIRST VARCHAR(32),
	CONTACT_LAST VARCHAR(32),
	CONTACT_TITLE VARCHAR(16),
	CONTACT_POSITION VARCHAR(64),
	LAST_YEAR_SALES DECIMAL(11,2),
	ADDRESS_1 VARCHAR(128),
	ADDRESS_2 VARCHAR(64),
	CITY VARCHAR(32),
	REGION VARCHAR(64),
	COUNTRY VARCHAR(64),
	POSTAL_CODE VARCHAR(32),
	EMAIL VARCHAR(64),
	PHONE VARCHAR(32)
);

alter table HA_DATA.CUSTOMER
   add constraint PK_CUSTOMER primary key cpbtree (CUSTOMER_ID);


create table HA_DATA.SAMPLE_ORDER (
	ORDER_ID BIGINT not null,
	CUSTOMER_ID BIGINT not null,
	ORDER_DATE DATE,
	REQUIRED_DATE DATE,
	SHIP_DATE DATE,
	SHIP_VIA VARCHAR(32),
	SHIPPED BOOLEAN,
	PO INTEGER,
	PAYMENT_RECEIVED BOOLEAN
);

alter table HA_DATA.SAMPLE_ORDER
   add constraint PK_ORDER primary key cpbtree (ORDER_ID);

create index RELATIONSHIP_2_FK on HA_DATA.ORDER_DETAIL (ORDER_ID);

create index RELATIONSHIP_3_FK on HA_DATA.SAMPLE_ORDER (CUSTOMER_ID);


IMPORT FROM CSV FILE '/mnt/hgfs/SampleData/Customer.csv' INTO HA_DATA.CUSTOMER
WITH
   RECORD DELIMITED BY '\n'
   FIELD DELIMITED BY ','
   OPTIONALLY ENCLOSED BY '"'
   SKIP FIRST 1 ROW
   FAIL ON INVALID DATA
   ERROR LOG '/mnt/hgfs/SampleData/Customer.csv.err'
;

IMPORT FROM CSV FILE '/mnt/hgfs/SampleData/Orders.csv' INTO HA_DATA.SAMPLE_ORDER
WITH
   RECORD DELIMITED BY '\n'
   FIELD DELIMITED BY ','
   OPTIONALLY ENCLOSED BY '"'
   SKIP FIRST 1 ROW
   FAIL ON INVALID DATA
   ERROR LOG '/mnt/hgfs/SampleData/Orders.csv.err'
;
```

Additional HANA Code Snippets

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

