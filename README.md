# Overview

The POS Hub API allows for the simple integration of a POS system with the Beyond One Platform. 

POS Hub creates a 2-way connection between one or more POS systems and Beyond One. The app gives users an easy way to standardize menu item names and connect shift or sales data to use with other solutions. 

The schemas against which the XML is validated are available in two formats for config, [rng|^config_1.0.grammar.rng] and [xsd|^config_1.0.grammar.xsd], and transactions, [rng|^transactions_1.0.grammar.rng] and [xsd|^transactions_1.0.grammar.xsd].


We also have sample files for [config|^sample_config.xml] and [transactions|^sample_transactions.xml] as reference.  Sometimes it's just easier to see "real" examples than read a definition.

All these files are also included at the bottom of this document in their full form.

----

# Setting Up a POS Hub Test Account

In order to send your POS data to Peach, you'll need an account with a subscription to the POS Hub App. 

Once you have account credentials,

* Go to [https://my.getbeyond.com|https://my.getbeyond.com] and enter your username and password
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
| [https://hub.peachworks.com/v1/config](https://hub.whentomanage.com/v1/config)                | POST        | Used to add configuration data for all types of transactions.                                                                                                          |
| [https://hub.peachworks.com/v1/transactions](https://hub.whentomanage.com/v1/transactions)    | POST        | Used to add all transaction data.                                                                                                                                      |
| [https://hub.peachworks.com/v1/pos\_token](https://hub.peachworks.com/v1/pos_token)           | POST,DELETE | Used to add or remove a pos\_token                                                                                                                                     |
| [https://hub.peachworks.com/v1/checks](https://hub.whentomanage.com/v1/checks)                | DELETE      | Used to remove check data.                                                                                                                                             |
| [https://hub.peachworks.com/v1/paid\_inouts](https://hub.whentomanage.com/v1/paid_inouts)     | DELETE      | Used to remove payin/payout data.                                                                                                                                      |
| [https://hub.peachworks.com/v1/deposits](https://hub.whentomanage.com/v1/deposits)            | DELETE      | Used to remove bank deposit data.                                                                                                                                      |
| [https://hub.peachworks.com/v1/shifts](https://hub.whentomanage.com/v1/shifts)                | DELETE      | Used to remove actual shift data.                                                                                                                                      |
| [https://hub.peachworks.com/v1/validate/config](https://hub.whentomanage.com/v1/shifts)       | POST        | Post an XML file to validate it                                                                                                                                        |
| [https://hub.peachworks.com/v1/validate/transactions](https://hub.whentomanage.com/v1/shifts) | POST        | Post an XML file to validate it                                                                                                                                        |
| [https://hub.peachworks.com/v1/scheduled\_shifts](https://hub.whentomanage.com/v1/shifts)     | GET         | Used to retrieve scheduling data (only with labor app).                                                                                                                |
| [https://hub.peachworks.com/v1/instructions](https://hub.peachworks.com/v1/instructions)      | GET         | Let the server indicate to the client which days need reloaded


## Authentication/authorization

The type of authentication is using a _pos_token_, as it is the simplest method with the least amount of data to copy and know about.

The method for POS Token authorization is to include an "Authorization:" header in the HTTP headers.  The content of this header is the pos_token:

```
     curl -X DELETE "[https://hub.peachworks.com/v1/things|https://api.peachworks.com/v1/app_objects/272]" -H "Authorization: ELO5afhGRMfRq05Zenk4ZlwFdDJttHuZ5dT12Zd6ASn…"
```

----

# Viewing Schemas and Example Data

## Schemas

The schemas defining what you see below are avaliable at the following links:

* [Configuration Schema|https://hub.peachworks.com/v1/schemas/config] (relaxNG)
* [Configuration Schema|https://hub.peachworks.com/v1/schemas/config?type=xsd] (XSD)
* [Transaction Schema|https://hub.peachworks.com/v1/schemas/transactions] (relaxNG)
* [Transaction Schema|https://hub.peachworks.com/v1/schemas/transactions?type=xsd] (XSD)

## Examples

Examples files showing the XML are available at the following links:

* [Configuration XML Example|https://hub.peachworks.com/v1/examples/config]
* [Transaction XML Example|https://hub.peachworks.com/v1/examples/transactions]

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


> The "id" field for these can be a number or a string.\\ \\
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

```
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

```
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

```
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

```
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

```
// XML
<void>
    <id>11</id>
   	<name>Spilled/Dropped</name>
</void>
```

### Discounts

Discount types.

* ID and NAME are REQUIRED. 
* Maps on *"name"*.

```
// XML
<discount>
    <id>9</id>
   	<name>Birthday Dessert</name>
</discount>
```

## Customer (House) Accounts

House accounts (i.e. customer accounts for in-house tabs and similar):  These use "customer_accounts" as their API object, but are referred to on the front-end as "House Accounts", for consistency reasons.

* ID and NAME are REQUIRED. 
* No auto-mapping:  all new house accounts are added to the master list.

\\


{code}
// XML
<customer_account>
    <id>22</id>
    <name>Antonio Smith</name>
</customer_account>
{code}

h2. Paid In/Outs

Paid In/Out 

* ID and NAME are REQUIRED. 
* Maps on *"name"*.

\\


{code}
// XML
<paid_inout>
    <id>103</id>
    <name>Delivery Fee</name>
</paid_inout>
{code}

h2. Payment Types

Payment types.

* ID, NAME and PAYMENT_GROUP are REQUIRED. 
* Maps on*"name" + "payment_group"*

\\


{code}
// XML
<payment_type>
    <id>44</id>
    <name>Master Card</name>
    <payment_group>CREDIT_CARD</payment_group>
</payment_type>
{code}

h2. Employees

Employees.

* ID and NAME are REQUIRED. 
* No auto-mapping: all new employees are added to the _master_ list.
* For employee jobs, only the ID is required.

\\


{code}
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
{code}

\\


h2. Jobs

Jobs.

* ID and NAME are REQUIRED. 
* Maps on *"name"*.

\\


{code}
// XML
<job>
   	<id>108</id>
    <name>Busser</name>
</job>

{code}

\\


h2. Sample Request

You can either:

* send a file attached to the request

\\


{code}
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
{code}

\\


----

\\


h1. Check Data

h2. Required Check Fields

A "check" refers to a single customer bill of sale (not to a personal check used for payment).

The required primary data fields for every check are...Optional Check Fields

The following check fields are optional.

h2. A check can have multiple discounts applied to it.  Here are the fields for a discount:Check Discounts

h2. \\
A check can have multiple items applied to it.  Here are the fields for an item:Check Items

h2. Check Payments

\\
\\


{code}
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
{code}

\\


h1. Shift Data

h2. Required Shift Fields

The required primary data fields for every shift are:

h2. Optional Shift Fields

The following shift fields are optional.

h2. \\
A shift can have multiple breaks within it.  Here are the fields for a break:Breaks

\\
\\


{code}
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
{code}

h1. Paid In/Out Payments

* negative amounts indicate paid out

h2. Required Fields

h2. \\
\\
Optional Fields

\\


\\


{code:language=xml|title=Paid In/Outs}
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
{code}

h1. Deposits

h2. Required Fields

h2. Optional Fields

\\


\\


{code:language=xml|title=Deposits}
<deposit>
    <id>10</id>
    <amount>700.54</amount>
    <deposited_at>2015-01-06T01:37:09</deposited_at>
    <last_modified_at>2015-01-06T01:37:09</last_modified_at>
    <business_date>2015-01-05</business_date>
    <employee_id>43</employee_id>
</deposit>
{code}

h1. Scheduled Shift Exports

h2. Request

* Accessible only with pos_token authorization.  The token should be put in the "Authorization" header.
* If the account is not subscribed to Schedule, the only result returned will be a 404 status.
* When no requested date range is given, all upcoming scheduled/published shifts are returned from the next four weeks and including those from today (the whole day, even if some are already completed).
* The range of shifts sent, in any case, is limited to four weeks of future shifts from “today” and then the past seven days, so one may fetch all of “this week”.
* Shifts are only returned for employees+jobs which have synced over from the POS.  If an employee were created in Peach and had no associated POS ID, their shifts would not be included.

Additional parameters maybe be supplied if a specific date range is needed.

* All timestamps may include a timezone, e.g. "2017-03-01T07:30:00-04:00" for specificity.  If no timezone is present on the timestamp then "local store time" is assumed.
* "start" and "end" may be bare dates "2017-03-20", which will return the whole business day's shifts.

h2. \\
Returned data will include the followingResponse

* a version
* an array of count of shifts per date returned (based on start date of shift)
* an array of shifts which each include
** employee_id - ID is from POS
** job_id - ID is from POS
** time_in - scheduled beginning date+time of shift in UTC
** time_out - scheduled ending date+time of shift in UTC

\\


{code:language=js|title=Exported Shifts JSON}
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
{code}

\\


{code:language=js|title=using include_names\=true}
{
     "id": 123,
     "employee_id": "50",
     "employee_name": "Ian Smythe",
     "job_id": "118",
     "job_name": "Host",
     "time_in": "2017-02-17T14:15:00Z",
     "time_out": "2017-02-17T22:15:00Z"
    }
{code}

\\
