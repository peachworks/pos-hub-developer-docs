# Overview

The POS Hub API allows for the simple integration of a POS system with the Beyond One Platform. 

POS Hub creates a 2-way connection between one or more POS systems and Beyond One. The app gives users an easy way to standardize menu item names and connect shift or sales data to use with other solutions. 

## Table of Contents
* [Setting Up a Test Account](#setting-up-a-test-account)
* [Connecting](#connecting)
* [Schemas and Example Data](#viewing-schemas-and-example-data)
* [Configuration Data](#configuration-data)
   * [Categories](#categories)
   * [Items](#items)
   * [Revenue Centers](#revenue-centers)
   * [Order Types](#order-types)
   * [Voids](#voids)
   * [Discounts](#discounts)
   * [Customer (House) Accounts](#customer-house-accounts)
   * [Paid Ins/Outs](#paid-insouts)
   * [Payment Types](#payment-types)
   * [Employees](#employees)
   * [Jobs](#jobs)
   * [Adding Configuration Data](#adding-configuration-data)
   * [Deleting Configuration Data](#deleting-configuration-data)
* [Check Data](#check-data)
   * [Required Fields](#required-check-fields)
   * [Optional Fields](#optional-check-fields)
   * [Check Discounts](#check-discounts)
   * [Check Items](#check-items)
   * [Check Payments](#check-payments)
   * [Adding Check Data](#adding-check-data)
   * [Updating Check Data](#updating-check-data)
   * [Deleting Check Data](#deleting-check-data)
   * [Reloading Check Data](#reloading-check-data)
* [Shift Data](#shift-data)
   * [Required Fields](#required-shift-fields)
   * [Optional Fields](#optional-shift-fields)
   * [Breaks](#breaks)
   * [Adding Shift Data](#adding-shift-data)
   * [Updating Shift Data](#updating-shift-data)
   * [Deleting Shift Data](#deleting-shift-data)
* [Paid In/Out Payments](#)
   * [Required Fields](#required-fields)
   * [Optional Fields](#optional-fields)
   * [Sample XML](#sample-xml-2)
* [Deposits](#deposits)
   * [Required Fields](#required-fields-1)
   * [Optional Fields](#optional-fields-1)
   * [Sample XML](#sample-xml-3)
* [Scheduled Shift Exports](#scheduled-shift-exports)
   * [Request](#request)
   * [Response](#response)


----

# Setting Up a Test Account

In order to send your POS data to Peach, you'll need an account with a subscription to the POS Hub App. 

Once you have account credentials,

* Go to https://my.getbeyond.com and enter your username and password
* click on the POS Hub application icon
** Enter the System Setup wizard
** There is an option in the first step of the wizard to choose a POS system.  Select "Direct".
** In the next step, select the locations (which individual restaurants) you want to set up on that system
* On the final step, we generate a pos_token for each location selected, and display it for you.
** The pos_token is hashed and contains the source, location, account IDs and the account_token
* **The POS customer must use that pos_token when sending us data**

----

# Connecting

We use a RESTful API.

These are resources used to add/remove data from POS Hub

| URL                                                                                           | Methods     | Description                                                                                                                                                            |
| --------------------------------------------------------------------------------------------- | ----------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| https://hub.peachworks.com/v1/config               | POST        | Used to add all types of configuration data (employees, jobs, items, categories, etc).                                                                                                          |
| https://hub.peachworks.com/v1/transactions         | POST        | Used to add all types of transaction data (shifts, checks, deposits, etc).                                                                                                                                      |
| https://hub.peachworks.com/v1/pos_token           | POST,DELETE | Used to add or remove a pos\_token                                                                                                                                     |
| https://hub.peachworks.com/v1/checks               | DELETE      | Used to remove check data.                                                                                                                                             |
| https://hub.peachworks.com/v1/paid_inouts         | DELETE      | Used to remove payin/payout data.                                                                                                                                      |
| https://hub.peachworks.com/v1/deposits             | DELETE      | Used to remove bank deposit data.                                                                                                                                      |
| https://hub.peachworks.com/v1/shifts                | DELETE      | Used to remove actual shift data.                                                                                                                                      |
| https://hub.peachworks.com/v1/validate/config       | POST        | Post an XML file to validate it                                                                                                                                        |
| https://hub.peachworks.com/v1/validate/transactions | POST        | Post an XML file to validate it                                                                                                                                        |
| https://hub.peachworks.com/v1/scheduled_shifts     | GET         | Used to retrieve scheduling data (only with labor app).                                                                                                                |
| https://hub.peachworks.com/v1/instructions     | GET         | Let the server indicate to the client which days need reloaded


## Authentication/authorization

The type of authentication is using a _pos_token_, as it is the simplest method with the least amount of data to copy and know about.

The method for POS Token authorization is to include an "Authorization:" header in the HTTP headers.  The content of this header is the pos_token:

```
     curl -X DELETE "https://hub.peachworks.com/v1/things" -H "Authorization: ELO5afhGRMfRq05Zenk4ZlwFdDJttHuZ5dT12Zd6ASn…"
```

----

# Viewing Schemas and Example Data

## Schemas

The schemas defining what you see below are avaliable at the following links:

* https://hub.peachworks.com/v1/schemas/config (relaxNG)
* https://hub.peachworks.com/v1/schemas/config?type=xsd (XSD)
* https://hub.peachworks.com/v1/schemas/transactions (relaxNG)
* https://hub.peachworks.com/v1/schemas/transactions?type=xsd (XSD)

## Examples

Examples files showing the XML are available at the following links:

* https://hub.peachworks.com/v1/examples/config
* https://hub.peachworks.com/v1/examples/transactions

----

# Configuration Data

Config data is consists of definitions for the following things

* Categories - list of sales categories (ex. Apps, Entrees, Desserts)
* Items - list of sales items (ex. Pizza, Burger, Wings)
* Order Types - list of order types (ex. Table, Tab, Delivery) 
* Revenue Centers - list of revenue centers (ex. Dining Room, Bar, Drive Thru)
* Voids - list of void reasons (ex. 86'd, Server Error)
* Discounts - list of discounts (ex. Guest Satisfaction, 10% Off, $5 Off)
* House Accounts - list of house accounts (ex. )
* Paid In/Outs - list of paid in/out reasons (ex. )
* Payment Types - list of payment/tender types (ex. Master Card, Cash, Amex)
* Employees - list of employees 
* Jobs - list of jobs (ex. Server, Cashier, Bartender)


> The "id" field for these can be a number or a string.
> A string ID is helpful in cases where a numeric ID is not available. For example, if a category named "Apps" did not have an ID, you can also use "Apps" as the ID.



## Master Lists & Mapping

### How Mapping Works

#### Config Data

We treat each _config data set_ as _unique to the selected location_ and source, and then we map these data sets to the _master list_ for each config type.

EXAMPLE:  If a POS sends the item "Wings" as part of the config data set, and it has not been added _from the source and location sending it_, Peach will:

* check for an existing master item called "Wings" 
   * if found, this "Wings" would map to the master "Wings";  
   * if not found, Peach would create a new _master item_ called "Wings".

**So the same item in multiple locations and/or sources can map to a single master item.**


> In POS Hub, a user can always remap items to a different master.

#### Other Data

For all other data, the mapping procedure is similar, in that the relevant fields will first be looked for in the individual location and then the master list.  Here is the process step-by-step for (as an example) a piece of data of type "discount":

1.  Upon upload, the POS Hub system will look at the combination of source + location + pos_discount_ id to determine where to look
1. If all match an existing POS discount in the mapping table, it will then look at the name (the mapping field, as defined in this document, for /discount)
  1.  If the name is the same, no changes to the list will occur - a discount with that name already exists, and will simply be recorded
  1. If the name of the discount has changed from what is in pos_discount_id, the name in the mapping table will get updated _and_ a new wtm master item will be created.  If there are no other pos discounts in the original master, the master would be empty and would get auto-deleted.
1. If the source, location, or pos_discount_ id do not match (or do not exist), Hub will then look to see if a _master_ item exists with the same name  
   1. If there is a master with the same name, the new pos_discount_ id is added to the mapping table and assigned to the matching master item
   1. if there is not a master with the same name, we create a new master item and map the POS item to it

### Further notes on mapping:

* if the app admin changes a wtm master item name in Hub, those changes will stay and the pos item will remain mapped to this master on subsequent syncs
* if the app admin changes a wtm master item name in Hub and then the name changes on the POS, a new name is recognized and a new master item is created.  The original wtm master item will be empty and thus auto-deleted


## Mapping Objects

### Categories

Categories are sales categories for items and can be hierarchical.

* ID and NAME are required. 
* Maps on **"name" + "parent_id"**

```xml
// XML

<category>
	<id>10</id>
	<name>Apps</name>
	<parent_id>2</parent_id>
</category>
```

### Items

Items for sale.

* ID and NAME are REQUIRED. 
* Maps on **"name" + "category_id"**

```xml
// XML
<item>
    <id>1</id>
    <name>12 Wings</name>
    <category_id>1000</category_id>
    <price>8.99</price>
    <sku>7A12345-WHT-X</sku>
</item>
```

### Revenue Centers

Revenue centers.

* ID and NAME are REQUIRED. 
* Maps on **"name"**.

```xml
// XML
<revenue_center>
    <id>8</id>
    <name>Takeout</name>
</revenue_center>
```

### Order Types

Order types.

* ID and NAME are REQUIRED. 
* Maps on **"name"**

```xml
// XML
<order_type>
    <id>8</id>
    <name>Drive Thru</name>
</order_type>
```

### Voids

Voids and their reasons.

* ID and NAME are REQUIRED. 
* Maps on **"name"**.

```xml
// XML
<void>
    <id>11</id>
   	<name>Spilled/Dropped</name>
</void>
```

### Discounts

Discount types.

* ID and NAME are REQUIRED. 
* Maps on **"name"**.

```xml
// XML
<discount>
    <id>9</id>
   	<name>Birthday Dessert</name>
</discount>
```

### Customer (House) Accounts

House accounts (i.e. customer accounts for in-house tabs and similar):  These use "customer_accounts" as their API object, but are referred to on the front-end as "House Accounts", for consistency reasons.

* ID and NAME are REQUIRED. 
* No auto-mapping:  all new house accounts are added to the master list.

```xml
// XML
<customer_account>
    <id>22</id>
    <name>Antonio Smith</name>
</customer_account>
```

### Paid In/Outs

Paid In/Out 

* ID and NAME are REQUIRED. 
* Maps on **"name"**.

```xml
// XML
<paid_inout>
    <id>103</id>
    <name>Delivery Fee</name>
</paid_inout>
```

### Payment Types

Payment types.

* ID, NAME and PAYMENT_GROUP are REQUIRED. 
* Maps on*"name" + "payment_group"*

```xml
// XML
<payment_type>
    <id>44</id>
    <name>Master Card</name>
    <payment_group>CREDIT_CARD</payment_group>
</payment_type>
```

### Employees

Employees.

* ID and NAME are REQUIRED. 
* No auto-mapping: all new employees are added to the _master_ list.
* For employee jobs, only the ID is required.

```xml
// XML
<employee>
    <id>20</id>
    <employee_number>3953</employee_number>
    <first_name>Harold</first_name>
    <last_name>Carlson</last_name>
    <email>HaroldCCarlson@example.com</email>
    <mobile_phone>212-444-3421</mobile_phone>
    <home_phone>212-989-3499</home_phone>
    <hire_date>2006-06-05</hire_date>
    <inactive_date>2006-06-05</inactive_date>
    <terminated_date>2006-06-05</terminated_date>
    <address>2827 Gore Street</address>
    <address2>Suite D.</address2>
    <city>Sugar Land</city>
    <postal_code>77478</postal_code>
    <state_province>TX</state_province>
    <country>US</country> 
    <jobs>
        <job>
	        <id>5</id>
	        <wage>10.50</wage>
	        <is_primary>false</is_primary>
        </job>
        <job>
	        <id>4</id>
	        <wage>15.00</wage>
	        <is_primary>true</is_primary>
        </job>
    </jobs>
</employee>
```

### Jobs

Jobs.

* ID and NAME are REQUIRED. 
* Maps on **"name"**.

```xml
// XML
<job>
   	<id>108</id>
    <name>Busser</name>
</job>
```


## Adding Configuration Data

You can either:

* send an XML file attached to the request or send the XML inline

```xml
// Send as a file

POST https://hub.peachworks.com/v1/config
{
    "pos_token":"ELO5afhGRMfRq05Zenk4ZlwFdDJttHuZ5dT12Zd6ASn"
}
+attachment - XML File (name of file's form field does not matter, just send on attachment)

// Send inline XML
Authorization: "ELO5afhGRMfRq05Zenk4ZlwFdDJttHuZ5dT12Zd6ASn"
POST https://hub.peachworks.com/v1/config
+body is XML data
```

## Deleting Configuration Data
Configuration data can be deleted from the Beyond One UI or via the Beyond One API https://github.com/getbeyond/app-developer-docs/wiki/App-API-Objects:-POS-Hub

----

# Check Data

## Required Check Fields

A "check" refers to a single customer bill of sale (not to a personal check used for payment).

The required primary data fields for every check are...
| Field              | Type       | Description                                                                                                                                                                                                                                                                                                               |
| ------------------ | ---------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| check\_id          | string     | unique ID for a check for a given POS in a location. If the POS System does not provide a unique ID, you can try combining the check number with the check open datetime.                                                                                                                                                 |
| check\_number      | string     | the check number, if no check number exists, you can use the check\_id. This does not need to be unique.                                                                                                                                                                                                                  |
| business\_date     | date       | date used for reporting the sales                                                                                                                                                                                                                                                                                         |
| business\_time     | time       | time used for reporting the sales. This is a local time (not UTC) and is used so that we can easily report on sales from 3-4pm at multiple locations across different time zones. In our own integrations, we would need to calculate this field if we are provided UTC time data by looking up the locations time zone. |
| opened\_at         | datetime   | date & time the check was opened                                                                                                                                                                                                                                                                                          |
| last\_modified\_at | datetime   | last time the check was updated                                                                                                                                                                                                                                                                                           |
| total              | decimal(2) | the check total                                                                                                                                                                                                                                                                                                           |


## Optional Check Fields

The following check fields are optional.

| Field               | Type       | Description                                                                                 |
| ------------------- | ---------- | ------------------------------------------------------------------------------------------- |
| employee\_id        | string     | employee assigned to the check                                                              |
| closed\_at          | datetime   | date & time the check was closed, if this is NULL, we will report the check as being "open" |
| reopened\_at        | datetime   | date & time the check was reopened                                                          |
| reopened\_by\_id    | string     | employee id for the person that reopened the check                                          |
| order\_number       | string     | order number                                                                                |
| order\_type\_id     | string     | order type ID                                                                               |
| revenue\_center\_id | string     | revenue center ID                                                                           |
| guest\_count        | integer    | total number of guests                                                                      |
| discount\_total     | decimal(2) | total amount of discounts                                                                   |
| void\_total         | decimal(2) | total amount of voids                                                                       |
| inclusive\_tax      | decimal(2) | total amount of inclusive tax                                                               |
| exclusive\_tax      | decimal(2) | total amount of exlusive tax                                                                |
| table\_number       | string     | table number                                                                                |
| auto\_grat          | decimal(2) | total amount of auto gratuity applied to the check                                          |
| service\_charge     | decimal(2) | total amount of service charges applied to the check                                        |



## Check Discounts
A check can have multiple discounts applied to it.  Here are the fields for a discount

| Field            | Type       | Description                                                  |
| ---------------- | ---------- | ------------------------------------------------------------ |
| discount\_id     | string     | discount ID (REQUIRED)                                       |
| quantity         | integer    | total number of items affected by this discount (REQUIRED)   |
| amount           | decimal(2) | total amount of discounts for this discount (REQUIRED)       |
| category\_id     | string     | optional category id that this discount should be applied to |
| approver\_id     | string     | employee ID for the manager that approved this discount      |
| reopened\_at     | datetime   | date & time the check/discount was reopened                  |
| reopened\_by\_id | string     | employee id for the person that reopened the check/discount  |
| applied\_at      | datetime   | date & time this discount was applied                        |


## Check Items
A check can have multiple items applied to it.  Here are the fields for an item

| Field          | Type        | Description                                                                                                       |
| -------------- | ----------- | ----------------------------------------------------------------------------------------------------------------- |
| item\_id       | string      | item ID (REQUIRED)                                                                                                |
| quantity       | integer     | total number of items affected by this item (REQUIRED)                                                            |
| amount         | decimal(2)  | total sales amount for this item (REQUIRED) - this amount should INCLUDE discount/comp BUT DOES NOT INCLUDE taxes |
| category\_id   | string      | optional category id that this item should be applied to                                                          |
| inclusive\_tax | decimal(2)  | optional inclusive tax amount                                                                                     |
| exclusive\_tax | decimal(2)  | optional exclusive tax amount                                                                                     |
| discount       | xml element | optional , see check discounts (can only have one)                                                                |
| void           | xml element | optional , see check voids (can only have one)                                                                    |
| modifier       | xml element | optional , this, a check item modifier (can have many) things like bacon, cheese etc..|


## Check Payments

| Field                | Type       | Description                                                  |
| -------------------- | ---------- | ------------------------------------------------------------ |
| payment\_type\_id    | string     | ID for the payment type (REQUIRED)                           |
| total                | decimal(2) | total amount of the payment (REQUIRED)                       |
| received             | integer    | total number of items affected by this discount (REQUIRED)   |
| change               | decimal(2) | total amount of discounts for this discount (REQUIRED)       |
| tip                  | string     | optional category id that this discount should be applied to |
| tip\_processing\_fee | string     | employee ID for the manager that approved this discount      |
| auto\_grat           | datetime   | date & time this discount was applied                        |
| service\_charges     | decimal    | any service charges applied to the payment                   |
| memo                 | text       | notes about the payment                                      |
| last\_4\_digits      | text       | last four digits of CC number                                |
| customer\_name       | text       | first and last name of customer                              |
| reopened\_at         | datetime   | date & time the check/discount was reopened                  |
| reopened\_by\_id     | string     | employee id for the person that reopened the check/discount  |
| applied\_at          | datetime   | date & time this discount was applied                        |


## Adding Check Data

```xml
// Send as a file

POST https://hub.peachworks.com/v1/transactions
{
    "pos_token":"ELO5afhGRMfRq05Zenk4ZlwFdDJttHuZ5dT12Zd6ASn"
}
+attachment - XML File (name of file's form field does not matter, just send on attachment)

// Send inline XML
Authorization: "ELO5afhGRMfRq05Zenk4ZlwFdDJttHuZ5dT12Zd6ASn"
POST https://hub.peachworks.com/v1/transactions
+body is XML data

// Check XML
<check>
    <check_id>3001-2011-02-03T09:19:45</check_id>
    <check_number>3001</check_number> 
    <order_number>A203</order_number>
    <business_date>2011-02-03</business_date>
    <business_time>09:19:45</business_time>
    <opened_at>2011-02-03T09:19:45</opened_at>
    <closed_at>2011-02-03T11:02:01</closed_at>
    <last_modified_at>2011-02-03T11:02:01</last_modified_at>
    <reopened_at>2011-02-03T11:02:01</reopened_at> 
    <reopened_by_id>38</reopened_by_id>
    <order_number></order_number> 
    <revenue_center_id>2</revenue_center_id> 
    <order_type_id>2</order_type_id>   
    <guest_count>1</guest_count>   
    <total>34.08</total>
    <discount_total>5.00</discount_total>
    <void_total>5.00</void_total>  
    <inclusive_tax>0</inclusive_tax>   
    <exclusive_tax>2.67</exclusive_tax>
    <discount>
        <discount_id>5</discount_id>       
        <category_id></category_id>    
        <quantity>1</quantity>   
        <amount></amount>      
        <approver_id></approver_id>    
        <applied_at>2011-02-03T12:24:39</applied_at>   
    </discount>  
    <employee_id>102</employee_id> 
    <table_number></table_number>  
    <room_number></room_number>
    <reservation_number></reservation_number>  
    <loyalty_number></loyalty_number>  
    <auto_grat></auto_grat>    
    <service_charge></service_charge>  
    <item>
        <item_id>228</item_id>     
        <category_id></category_id>    
        <quantity>3</quantity>     
        <price>3.99</price>    
        <amount>11.97</amount>     
        <void>
            <void_id></void_id>        
            <approver_id></approver_id>    
            <applied_at>2011-02-03T12:24:39</applied_at>       
        </void>
        <discount>
            <discount_id></discount_id>    
            <category_id></category_id>    
            <quantity>1</quantity>   
            <amount></amount>      
            <approver_id></approver_id>    
            <applied_at>2011-02-03T12:24:39</applied_at>   
        </discount>  
        <inclusive_tax>0</inclusive_tax>   
        <exclusive_tax>.23</exclusive_tax>     
        <modifier>
            <item_id>99</item_id>          
            <category_id></category_id>    
            <quantity>2</quantity>         
            <price>0.00</price>        
            <amount>11.97</amount>     
            <discount>
                <discount_id></discount_id>        
                <category_id></category_id>
                <amount></amount>          
                <approver_id></approver_id>        
                <applied_at>2011-02-03T12:24:39</applied_at>       
            </discount>      
            <inclusive_tax>0</inclusive_tax>       
            <exclusive_tax>.23</exclusive_tax>     
        </modifier>      
    </item>
    <payment>
        <applied_at>2011-02-03T12:24:39</applied_at>   
        <total>34.08</total>       
        <received>50</received>    
        <change>15.92</change>     
        <tip></tip>    
        <tip_processing_fee></tip_processing_fee>      
        <payment_type_id>4</payment_type_id>
        <auto_grat></auto_grat>    
        <service_charge></service_charge>  
        <memo></memo>  
        <last_4_digits></last_4_digits>    
        <customer_name></customer_name>    
    </payment>
</check>
```

## Updating Check Data
Send the updated check data exactly as you would if adding it for the first time. If the check ID already exists, the data will be updated.

> If you're unsure of changes within individual checks, it is often simpler (and less error prone) to delete the checks for a period of time and then add them again.

## Deleting Check Data
If deleting  check data in order to reload it, see "Reloading Check Data" below to eliminate the need for a separate API request for deletion.

### Deleting Checks By Day

* start_date and end_date are required and it can be either an ISO date or an array of ISO dates
* If any checks on any date provided can't be deleted for any reason this will return an error and nothing deleted
   * 400 {"status":"error", "message":"Could not delete checks: <comma separated list of check ids>. Nothing was deleted."}
* Returns 200 {"status":"success", "delete_count":integer} on success
* When deciding the date to delete, use check.business_date
	
#### Example Request
```
DELETE /v1/checks
Authorization: eukj3pos_token…
{
  "start_date":"2013-08-28",
  "end_date":"2013-08-28",
  "is_developer":true
}
  
Returns
200
{
  "status":"success",
  "delete_count":153
}
```

#### Example Curl Request
`curl -X DELETE --form start_date=2013-08-28 --form end_date=2013-08-28 -H "Authorization: $TOKEN" https://hub.peachworks.com/v1/checks`

### Deleting Checks By ID
* "ids" is used and can be either one check id or an array of check ids
* If any checks can't be deleted for any reason this will return an error and nothing deleted
   * 400 {"status":"error", "message":"Could not delete checks: <comma separated list of check ids>. Nothing was deleted."}
	
#### Example Request
```
DELETE /v1/checks
Authorization: eukj3pos_token…
{
  "ids":["304773739739","30477373974","3047737392"],
//or
 "ids":"304773739739",  <-- note this also should work for one check_id
}
  
Returns
200
{
  "status":"success",
  "delete_count":3
}
```

#### Example Curl Request
`curl -X DELETE --form ids=10198702 -H "Authorization: $TOKEN" https://hub.peachworks.com/v1/checks`


## Reloading Check Data
In the case of just missing or bad data, the merchant can request specific days to be reloaded in Beyond One, these requested business days are available to integrators from our server using `/v1/instructions` .  If the array is empty, no missing days have been reported.

* /v1/instructions (GET)
   * authorize with the given pos_token(s), which the integration should have saved per source+location, just like normal uploads
   * response body is similar to:
      * `{ "reload": ["2016-03-01", "2016-03-02"], "is_active": true, "other": "things" }`
      * The only keys that apply universally are "reload" and “is_active”, but ignore everything except reload for now.
 
To reload the check data, a full day’s data should be generated and then uploaded to the `/v1/checks` end point with an additional query parameter indicating that this is a reload for a specific business date `?reload_date=2016-04-01`

This way the server knows that the integrator is uploading a replacement day and will DELETE the existing transactions on that business date before adding the new transactions. This will also update the reload requests (if there is one) as completed after a successful load.

for example:
```
POST /v1/transactions?reload_date=2016-04-01
[multipart-form with XML file] 
```

### When Data Isn't Available for a Day
If the client no longer has any data for a requested business date, due to the data having been deleted or maybe the store was actually closed that day, then the integrator can send an unavailable status so the server won’t ask for this day again.

* `/v1/client_status (POST)`
   * authorize with the given pos_token(s), which the integration should have saved per source+location, just like normal uploads
   * Parameters
      * unavailable_days - is an array of dates for which the client cannot upload the data because it is missing. Time part will be ignored.

for example:

```
POST /v1/client_status
{“unavailable_days”: [“2018-12-25”, “2017-12-25”, “2018-01-01”] }
```

### Testing during development
One thing you will need to do for testing is to set the reload_days on your own to trigger your systems’s reaction to the instructions.  This is the endpoint where that happens

* `/v1/reload_days (POST)`
   * The endpoint will not let there be more than 31 entries per source/location at a time, to prevent a large burden on the system from reloading.
   * an example of the request body:
      * {"account_id":1000,"access_token":"Ac~A3kW", location_ids: [1,2,3], source_id: 5, start_date: '2016-01-01', end_date: '2016-02-01'}

With the example values replaced by the values for your account, locations, source and dates.

Note: this uses an access token which you can get in a browser after logged in to https://my.getbeyond.com and looking in the web developer console network debugger.  It does not use a pos_token because it allows more than one location id to be passed in.


# Shift Data

## Required Shift Fields

The required primary data fields for every shift are:

| Field                 | Type                                                        | Description                                                                                                                                             |
| --------------------- | ----------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------- |
| id                    | string                                                      | unique ID for a shift in a location. If the POS System does not provide a unique ID, you can try combining the give shift ID with the shift start time. |
| employee\_id          | string                                                      | employee ID for the shift                                                                                                                               |
| job\_id               | string                                                      | job ID for the shift                                                                                                                                    |
| business\_date        | date                                                        | date used for reporting the labor                                                                                                                       |
| business\_start\_time | time                                                        | time used for reporting the labor                                                                                                                       |
| started\_at           | datetime                                                    | date & time the shift was started                                                                                                                       |
| total\_minutes        | integer - note: future updates may change this to a decimal | total number of minutes for the shift (including overtime)                                                                                              |
| total\_pay            | decimal(2)                                                  | total cost for the shift (including overtime)                                                                                                           |
| shift\_updated\_at    | datetime                                                    | last time the shift was updated                                                                                                                         |


## Optional Shift Fields

The following shift fields are optional.

| Field                  | Type       | Description                          |
| ---------------------- | ---------- | ------------------------------------ |
| business\_end\_time    | time       | business time the shift ends         |
| ended\_at              | datetime   | date & time the shift ended          |
| pay\_rate              | decimal(2) | pay rate                             |
| overtime\_minutes      | integer    | total number of overtime minutes     |
| overtime\_pay\_rate    | decimal(2) | pay rate for overtime                |
| overtime\_pay          | decimal(2) | total cost for overtime              |
| cc\_tips               | decimal(2) | total credit card tips               |
| cash\_tips             | decimal(2) | total cash tips                      |
| paid\_break\_minutes   | integer    | total number of paid break minutes   |
| paid\_break\_pay       | decimal(2) | total cost for paid breaks           |
| unpaid\_break\_minutes | integer    | total number of unpaid break minutes |
| start\_bank            | decimal(2) | bank cash total at start of shift    |
| end\_bank              | decimal(2) | bank cash total at end of shift      |



## Breaks
A shift can have multiple breaks within it.  Here are the fields for a break:

| Field          | Type       | Description                                       |
| -------------- | ---------- | ------------------------------------------------- |
| started\_at    | datetime   | date & time for the start of the break (REQUIRED) |
| ended\_at      | datetime   | date & time for the end of the break (REQUIRED)   |
| break\_minutes | integer    | total number of break minutes (REQUIRED)          |
| is\_paid       | boolean    | true if paid break, false if unpaid break         |
| break\_pay     | decimal(2) | total cost for this break ($0 if unpaid break)    |


## Adding Shift Data
```xml
// Send as a file
POST https://hub.peachworks.com/v1/transactions
{
    "pos_token":"ELO5afhGRMfRq05Zenk4ZlwFdDJttHuZ5dT12Zd6ASn"
}
+attachment - XML File (name of file's form field does not matter, just send on attachment)

// Send inline XML
Authorization: "ELO5afhGRMfRq05Zenk4ZlwFdDJttHuZ5dT12Zd6ASn"
POST https://hub.peachworks.com/v1/transactions
+body is XML data

// Shift XML
<shift>
    <id>50751</id>
    <employee_id>2089</employee_id>
    <job_id>8</job_id>
    <business_date>2014-10-01</business_date>
    <business_start_time>14:50:04</business_start_time>
    <business_end_time>21:14:11</business_end_time>
    <last_modified_at>2011-02-03T11:02:01</last_modified_at>
    <started_at>2011-02-03T09:50:04</started_at>
    <ended_at>2011-02-03T16:14:11</ended_at>
    <total_minutes>834</total_minutes>
    <pay_rate></pay_rate>
    <total_pay>65.65</total_pay>
    <overtime_minutes>0</overtime_minutes>
    <overtime_pay_rate></overtime_pay_rate>
    <overtime_pay>0</overtime_pay>
    <cc_tips></cc_tips>
    <cash_tips>0</cash_tips>
    <start_bank>50.00</start_bank>
    <end_bank>25.00</end_bank>
    <paid_break_minutes></paid_break_minutes>
    <paid_break_pay></paid_break_pay>
    <unpaid_break_minutes></unpaid_break_minutes>
    <break>
	    <started_at></started_at>
        <ended_at></ended_at>
        <break_minutes></break_minutes>
        <is_paid></is_paid>
        <break_pay></break_pay>
    </break>
</shift>
```

## Updating Shift Data
Send the updated shift data exactly as you would if adding it for the first time. If the shift ID already exists, the data will be updated.

> If you're unsure of changes within an individual shift, it is often simpler (and less error prone) to delete the shifts for a period of time and then add them again.

## Deleting Shift Data
If deleting shift data in order to reload it, see "Reloading Check Data" below to eliminate the need for a separate API request for deletion.

### Deleting Shifts By Day

* start_date and end_date are required and it can be either an ISO date or an array of ISO dates
* If any shifts on any date provided can't be deleted for any reason this will return an error and nothing deleted
   * Returns 400 `{"status":"error", "message":"Could not delete shifts: <comma separated list of shift ids>. Nothing was deleted."}`
* Returns 200 `{"status":"success", "delete_count":_integer_}` on success
* When deciding the date to delete, use shift.business_date

#### Example Request
```
DELETE /v1/shifts
Authorization: eukj3pos_token…
{
  "start_date":"2013-08-28",
  "end_date":"2013-08-28",
}
  
Returns
200
{
  "status":"success",
  "delete_count":153
}
```

#### Example Curl Request
`curl -X DELETE --form start_date=2013-08-28 --form end_date=2013-08-28 -H "Authorization: $TOKEN" https://hub.peachworks.com/v1/shifts`

### Deleting Shifts By ID
* "ids" is used and can be either one shift id or an array of shift ids
* If any shifts can't be deleted for any reason this will return an error and nothing deleted
   * 400 {"status":"error", "message":"Could not delete shifts: <comma separated list of shift ids>. Nothing was deleted."}
	
#### Example Request
```
DELETE /v1/shifts
Authorization: eukj3pos_token…
{
  "ids":["304773739739","30477373974","3047737392"],
//or
 "ids":"304773739739",  <-- note this also should work for one check_id
}
  
Returns
200
{
  "status":"success",
  "delete_count":3
}
```

#### Example Curl Request
`curl -X DELETE --form ids=10198702 -H "Authorization: $TOKEN" https://hub.peachworks.com/v1/shifts`




# Paid In/Out Payments

* negative amounts indicate paid out

## Required Fields

| Field              | Type       | Description                                             |
| ------------------ | ---------- | ------------------------------------------------------- |
| id                 | text       | unique identifier of the paid in/out payment            |
| amount             | decimal(2) | Total amount of payment made (does not include any tip) |
| paid\_at           | datetime   | time the paid in/out was entered                        |
| payment\_type\_id  | object     | reference to payments table                             |
| business\_date     | date       | reporting date for paid in/out                          |
| last\_modified\_at | datetime   | time of last payment edit                       |

## Optional Fields

| Field                 | Type       | Description                         |
| --------------------- | ---------- | ----------------------------------- |
| paid\_inout\_id       | object     | reason for paid in/out              |
| employee\_Id          | object     | employee entering the paid in/out   |
| customer\_account\_id | object     | Customer Account paid in/out is for |
| tip                   | decimal(2) | any additional amount paid as tip   |

## Sample XML
```xml
<paid_inout_payment>
    <id>10</id>
    <amount>102.00</amount>
    <paid_at>2015-01-05T19:07:36</paid_at>
    <last_modified_at>2015-01-06T01:37:09</last_modified_at>
    <payment_type_id>10</payment_type_id>
    <business_date>2015-01-05</business_date>
    <paid_inout_id>9</paid_inout_id>
    <employee_id>43</employee_id>
    <customer_account_id>6</customer_account_id>
    <tip>1.00</tip>
</paid_inout_payment>
```

# Deposits

## Required Fields

| Field              | Type       | Description                                            |
| ------------------ | ---------- | ------------------------------------------------------ |
| id                 | text       | unique identifier of the deposit                       |
| amount             | decimal(2) | total of the deposit (negative signifies a withdrawal) |
| deposited\_at      | datetime   | time the deposit was entered                           |
| business\_date     | date       | reporting date of deposit                              |
| last\_modified\_at | datetime   | time deposit was edited                                |

## Optional Fields

| Field        | Type   | Description                      |
| ------------ | ------ | -------------------------------- |
| employee\_id | object | employee who entered the deposit |

## Sample XML

```xml
<deposit>
    <id>10</id>
    <amount>700.54</amount>
    <deposited_at>2015-01-06T01:37:09</deposited_at>
    <last_modified_at>2015-01-06T01:37:09</last_modified_at>
    <business_date>2015-01-05</business_date>
    <employee_id>43</employee_id>
</deposit>
```

# Scheduled Shift Exports

## Request

* Accessible only with pos_token authorization.  The token should be put in the "Authorization" header.
* If the account is not subscribed to Schedule, the only result returned will be a 404 status.
* When no requested date range is given, all upcoming scheduled/published shifts are returned from the next four weeks and including those from today (the whole day, even if some are already completed).
* The range of shifts sent, in any case, is limited to four weeks of future shifts from “today” and then the past seven days, so one may fetch all of “this week”.
* Shifts are only returned for employees+jobs which have synced over from the POS.  If an employee were created in Peach and had no associated POS ID, their shifts would not be included.

Additional parameters maybe be supplied if a specific date range is needed.

* All timestamps may include a timezone, e.g. "2017-03-01T07:30:00-04:00" for specificity.  If no timezone is present on the timestamp then "local store time" is assumed.
* "start" and "end" may be bare dates "2017-03-20", which will return the whole business day's shifts.

| Optional Parameters (names case sensitive) | Description                                                                                                                                                                                                                                                             |
| ------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| start                                      | beginning timestamp of range for which to fetch scheduled shifts. Inclusive. YYYY-MM-DDTHH:mm:ss format. When "start" is omitted "end" will be ignored                                                                                                                                  |
| end                                        | ending timestamp of range for which to fetch scheduled shifts.  Inclusive YYYY-MM-DDTHH:mm:ss format. When "end" is omitted "start" will return all shifts after start time. If included, "end" should be no later than four weeks from today.  Otherwise an error status is returned. |
| modified                                   | timestamp - fetch all shifts modified since the given time. shifts earlier than "today" are not included. YYYY-MM-DDTHH:mm:ss format. Supplying a "modified" parameter will cause the "start" and "end" to be ignored.                                                                 |
| include\_names                             | Boolean.  Include the POS names for employees, **<first + " " + last\>** in one field, and jobs. (Most of the time you don't want this, unless it's for a system like Aloha that uses names in the export.)
                                                                      |

## Response
Returned data will include the followinge

* a version
* an array of count of shifts per date returned (based on start date of shift)
* an array of shifts which each include
   * employee_id - ID is from POS
   * job_id - ID is from POS
   * time_in - scheduled beginning date+time of shift in UTC
   * time_out - scheduled ending date+time of shift in UTC

```json
{
  "version": "1.0",
  "date_counts": {
     "2017-02-17": 20,
     "2017-02-18": 30
  },
  "shifts": [
    {
     "id": 123,
     "employee_id": "50",
     "job_id": "118",
     "time_in": "2017-02-17T14:15:00Z",
     "time_out": "2017-02-17T22:15:00Z"
    },
    {
     "id": 321,
     "employee_id": "53",
     "job_id": "10",
     "time_in": "2017-02-17T14:15:00Z",
     "time_out": "2017-02-17T22:15:00Z"
    },
    {
     "id": 456,
     "employee_id": "52",
     "job_id": "118",
     "time_in": "2017-02-17T16:15:00Z",
     "time_out": "2017-02-17T23:45:00Z"
    }
  ]
}
```


```json
{
     "id": 123,
     "employee_id": "50",
     "employee_name": "Ian Smythe",
     "job_id": "118",
     "job_name": "Host",
     "time_in": "2017-02-17T14:15:00Z",
     "time_out": "2017-02-17T22:15:00Z"
    }
```
