<a name="d"></a>
<table width=100% border=0>
<tr ><td colspan=2><h1>SAP Analytics Cloud & HANA Express</h1></td></tr>
<tr><td><h3>Partner Video Series</h3></td><td width=75%></br>Getting Started with SAP HANA Express Edition and SAP Analytics Cloud</td>
</table>

### Description

This ReadMe file contains code and data for the "Getting Started with SAP HANA Express Edition and SAP Analytics Cloud" video series.  Please note that this file is subject to changes and additions.

[Click here to go to the playlist for the video series](https://www.youtube.com/playlist?list=PLkzo92owKnVxc5ywbnmPjmTPTUSZVlDIy) on the SAP HANA Academy YouTube channel.

### <a name="d"></a>ReadMe Contents

1) [Live Connection Commands](#lcc)
2) [Data Download](#dd)
3) [HANA Data SQL](#hds)
4) [Additional HANA Code Snippets](#ahcs)
<br>

### <a name="lcc"></a>Live Connection Commands

```
sudo openssl genrsa -rand /var/log/y2log:/var/log/messages \
        -out /etc/apache2/ssl.key/ca.key 2048


cat >~/.mkcert.cfg <<EOT
[ req ]
default_bits  = 2048
default_keyfile  = keyfile.pem
distinguished_name = req_distinguished_name
attributes  = req_attributes
prompt  = no
output_password = mypass

[ req_distinguished_name ]
C  = DE
ST = Baden-Wuerttemberg
L  = Walldorf
O  = SAP
OU  = Digital Partner Engineering
CN  = hxehost.localdomain
emailAddress  = digitalenablement@sap.com

[ req_attributes ]
challengePassword  = 1234
EOT


sudo openssl req -new -x509 -days 365 \
        -config ~/.mkcert.cfg \
        -key /etc/apache2/ssl.key/ca.key \
        -out /etc/apache2/ssl.crt/ca.crt 


sudo openssl genrsa -rand /etc/rc.config:/var/log/messages \
        -out /etc/apache2/ssl.key/server.key 2048


sudo openssl req -new \
        -config ~/.mkcert.cfg \
        -key /etc/apache2/ssl.key/server.key \
        -out /etc/apache2/ssl.csr/server.csr


cat >~/.mkcert.cfg <<EOT
extensions = x509v3
[ x509v3 ]
subjectAltName   = email:copy
nsComment        = SAP Digital Partner Engineering
nsCertType       = server

authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
subjectAltName = @alt_names
[alt_names]
DNS.1 = *.localdomain
DNS.2 = hxehost.localdomain
EOT


test -f ~/.mkcert.serial || echo 01 >~/.mkcert.serial


sudo openssl x509 -req -days 365 \
        -extfile ~/.mkcert.cfg \
        -CAserial ~/.mkcert.serial \
        -CA /etc/apache2/ssl.crt/ca.crt \
        -CAkey /etc/apache2/ssl.key/ca.key \
        -in /etc/apache2/ssl.csr/server.csr  \
        -out /etc/apache2/ssl.crt/server.crt 


sudo chmod 0666 /etc/apache2/ssl.crt/server.crt

sudo openssl dhparam 2048  >> /etc/apache2/ssl.crt/server.crt


cd /etc/apache2
cd vhosts.d
sudo vim hxehost-ssl.conf

<IfDefine SSL>
<IfDefine !NOSSL>
<VirtualHost _default_:443>
        ServerAdmin digitalenablement@sap.com
        ServerName hxehost.localdomain
        DocumentRoot /srv/www/htdocs
        ErrorLog /var/log/apache2/error_log
        TransferLog /var/log/apache2/access_log
        CustomLog /var/log/apache2/ssl_request_log   ssl_combined
        ProxyPreserveHost on 
        SSLEngine on
        SSLProxyEngine on
        SSLCertificateFile /etc/apache2/ssl.crt/server.crt
        SSLCertificateKeyFile /etc/apache2/ssl.key/server.key
        <Location />
           ProxyPass https://hxehost.localdomain:4390/ 
           ProxyPassReverse https://hxehost.localdomain:4390/ 
        </Location>
</VirtualHost>
</IfDefine>
</IfDefine>


sudo systemctl reload apache2
sudo systemctl status apache2


sudo cat /etc/apache2/ssl.crt/ca.crt

hdbsql -n localhost:39015 -u SYSTEM

\mu ON

ALTER SYSTEM ALTER CONFIGURATION ('xsengine.ini', 'database') 
SET ('httpserver', 'sessiontimeout') ='43200' 
WITH RECONFIGURE;

ALTER SYSTEM ALTER CONFIGURATION ('indexserver.ini', 'database') 
SET ('session', 'idle_connection_timeout') ='900' 
WITH RECONFIGURE;

CALL GRANT_ACTIVATED_ROLE('sap.hana.xs.admin.roles::RuntimeConfAdministrator','HA_USER');
CALL GRANT_ACTIVATED_ROLE('sap.hana.xs.admin.roles::SAMLViewer','HA_USER');
CALL GRANT_ACTIVATED_ROLE('sap.bc.ina.service.v2.userRole::INA_USER','HA_USER');

ALTER SYSTEM ALTER CONFIGURATION ('xsengine.ini', 'database') SET ('public_urls', 'http_url') = 'http://hxehost.localdomain:8090' WITH RECONFIGURE;
ALTER SYSTEM ALTER CONFIGURATION ('xsengine.ini', 'database') SET ('public_urls', 'https_url') = 'https://hxehost.localdomain:4390' WITH RECONFIGURE;

hdbsql -n localhost:39015 -u HA_USER

UPDATE "_SYS_XS"."RUNTIME_CONFIGURATION"
SET "CONFIGURATION" = ' {"cors":{
  "enabled":true,
  "allowOrigin":["https://your.sac.instance"],
  "exposeHeaders":["x-csrf-token"],
  "allowHeaders":["accept-language","x-sap-cid","x-request-with","x-csrf-token","content-type","authorization","accept"],
  "allowMethods":["GET","HEAD","POST","OPTIONS"],
  "maxAge":3600}
}'
WHERE "PACKAGE_ID" = 'sap.bc.ina.service.v2';



Define SAC your.sac.instance

<If "req_novary('ORIGIN') == 'https://${SAC}'">
    Header set Access-Control-Allow-Origin "https://${SAC}"
    Header set Access-Control-Allow-Credentials true
    Header set Access-Control-Allow-Methods "GET, POST, PUT"
    Header set Access-Control-Allow-Headers "X-Csrf-Token, x-csrf-token, x-sap-cid, Content-Type, Authorization"
    Header set Access-Control-Expose-Headers "x-csrf-token"
</If>
```
[Go back to the top of this ReadMe.](#d)
<br>
<br>

### <a name="dd"></a>Data Download

[Click here to download the Sample Data.](/SampleData.zip?raw=true)
<br>
<br>

### <a name="hds"></a>HANA Data SQL

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
[Go back to the top of this ReadMe.](#d)
<br>
<br>

#### <a name="ahcs"></a>Additional HANA Code Snippets

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
[Go back to the top of this ReadMe.](#d)

