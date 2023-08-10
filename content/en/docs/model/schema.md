---
title: "Schema"
description: ""
lead: "There json on setup > model Folder"
date: 2023-08-09T16:57:25+05:30
lastmod: 2023-08-09T16:57:25+05:30
draft: false
images: []
menu:
  docs:
    parent: ""
    identifier: "schema-346d7db95c547a1f9c75dcf2afb05474"
weight: 999
toc: true
---
## Model Config/ Table

### Model/Table Name
```json
"name":"table_name", // model name 
"table":"table_names" // table name
```
We have singular for name and plural for table name. You can set up anything but this is what is recommended

### Column Define

Right now We call it data

```json
"data": [{
    "name": "name", // column name
    "mysql_data": "varchar(255)", // mysql datatype
    "datatype": "string", // For interface
    "default": "" //any deafault value for mysql
}],
```

### Enable Column

Some time we need to enable colum which is define is value row is enable or now for that there we just need to 


```json
"enable": 0
```
It can be 0 and 1 for default above is 0 for default

### Additional

We have 2 additonal thing that we can add first

#### Slug
```json
"addtional": ["slug"],
```

#### SEO

```json
"addtional": ["seo"],
```
We can use both 
```json
"addtional": ["seo", "slug"],
```

## CRUD

We use curd in schema it is somehow restricted but still very usefull

Let see

```json
"crud": {
    "isuper": ["c", "r", "u", "d", "a", "p"],
    "islogin": ["c", "r", "a"],
    "ipublic": ["r"]
},
```
We have more then crud we have Create Read Update Delete with 2 more 

[**a**] **GetAll Row** and 

[**p**] for **Insert Mutiple** on **PUT** request we know PHP don't support that but we can use laravel trick for that.
### Roles
We have also roles

```json
"crud": {
    "roles": {
      "exectuive": ["c", "r", "u", "d", "a", "p"],
      "customrole": ["c", "r", "a"]
    }
},
```
So we can create custom roles.

## Relations

We have relation so we just need to 

```json
"relation":["user"] 
```
It just pick up from user model primary key which is id so in above case is user_id

### Default Value
We can make it nullable by setting default value

```json
"relation":{
  "name": "photo",
  "default": "NULL"
}
```

### Alias

```json
"relation":{
  "name": "photo",
  "alias": "photo_id"
}
```

## Full Example 

```json

{
  "name": "brand", //MODEL NAME
  "table": "brands", //TABLE NAME
  "crud": { //DIFFERENT ROLES
    "isuper": ["c", "r", "u", "a", "p", "d"], 
    //CREATE(c) READ(r) UPDATE(u) ALLREAD(a) INSERTMUTIPLE(p) DELETE(d)
    "ipublic": ["a", "r" ],
    "roles":{//some roles
      "some_roles":["c","r"]
    }
  },
  "enable": 1, //ENABLE TABLE
  "addtional": ["seo", "slug"], //SOME ADDTIONAL
  "data": [
    {
      "name": "name", //NAME OF COLUMN
      "mysql_data": "varchar(255)", //MYSQL DATATYPES
      "datatype": "string" //TYPESCRIPT DATATYPE
    }
  ],
  "relation": [ //
    "check"//RELATION TO MODEL NAME
    {
      "name": "photo", //RELATION TO MODEL NMAE
      "default": "NULL" //DEFAULT VALUE MYSQL
    }
  ]
}
```