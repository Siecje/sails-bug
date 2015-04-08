# testProject

I created a postgreSQL database because I am not able to look at the sqlite3 database.

```
psql (9.3.6)
Type "help" for help.

postgres=# CREATE DATABASE sails;
CREATE DATABASE
postgres=# CREATE USER sails_user WITH PASSWORD 'sails_password';
CREATE ROLE
postgres=# GRANT ALL PRIVILEGES ON DATABASE "sails" to sails_user;
GRANT
postgres=# \q

```

Steps taken

```bash
~/D/sail $ sails new testProject
info: Created a new Sails app `testProject`!
~/D/sail $ cd testProject/
~/D/s/testProject $ sails generate api user
info: Created a new api!
~/D/s/testProject $ sails generate api company
info: Created a new api!
~/D/s/testProject $ npm install sails-postgresql --save
```

##Models

api/models/Company.js
```JavaScript
module.exports = {

  attributes: {
    name: {
      type: 'string',
      required: true
    },
    // Relationships
    users: {
      collection: 'user',
      via: 'company'
    }
  }
};
```

api/models/User.js
```JavaScript
module.exports = {

  attributes: {
    name: {
      type: 'string',
      required: true
    },
    // Relationship
    company: {
      model: 'company',
      required: true
    }
  }
};
```

Create user without company and it gives an error message (Good)

```
(venv)~/D/shwhy $ http --json POST http://localhost:1337/user name=Test
HTTP/1.1 400 Bad Request
Access-Control-Allow-Credentials:
Access-Control-Allow-Headers:
Access-Control-Allow-Methods:
Access-Control-Allow-Origin:
Connection: keep-alive
Content-Length: 274
Content-Type: application/json; charset=utf-8
Date: Wed, 08 Apr 2015 15:24:52 GMT
Vary: X-HTTP-Method-Override
X-Powered-By: Sails <sailsjs.org>
set-cookie: sails.sid=s%3AQnkxK8OYECo_EId4-BmQ_GBpJxKOBCLL.7tTQqiJ2XXSWxBJFtugz8fdrAEwC3kejEVDn6y8Zegg; Path=/; HttpOnly

{
    "error": "E_VALIDATION",
    "invalidAttributes": {
        "company": [
            {
                "message": "\"required\" validation rule failed for input: null",
                "rule": "required"
            }
        ]
    },
    "model": "User",
    "status": 400,
    "summary": "1 attribute is invalid"
}
```

Create a user with a company id that does not exist. (I have not created a company)
```bash
(venv)~/D/shwhy $
http --json POST http://localhost:1337/user name=Test company=1
HTTP/1.1 201 Created
Access-Control-Allow-Credentials:
Access-Control-Allow-Headers:
Access-Control-Allow-Methods:
Access-Control-Allow-Origin:
Connection: keep-alive
Content-Length: 133
Content-Type: application/json; charset=utf-8
Date: Wed, 08 Apr 2015 15:25:02 GMT
Vary: X-HTTP-Method-Override
X-Powered-By: Sails <sailsjs.org>
set-cookie: sails.sid=s%3ASMBZhuUuov0tnENka8zVRDJ_SoPfcw_y.M7FE5h4tIPVQAJWOzyDR9pL3giCtCIT4Gv2wXv2T7Hw; Path=/; HttpOnly

{
    "company": 1,
    "createdAt": "2015-04-08T15:25:02.000Z",
    "id": 1,
    "name": "Test",
    "updatedAt": "2015-04-08T15:25:02.000Z"
}
```

The database table now contains the company id
```
sails=# select * from public.user;
 name | company | id |       createdAt        |       updatedAt
------+---------+----+------------------------+------------------------
 Test |       1 |  1 | 2015-04-08 11:25:02-04 | 2015-04-08 11:25:02-04
(1 row)
```

The database does not enforce the relationship because it is just an integer field.
```
Table "public.user"
Column   |           Type           |                     Modifiers                     | Storage  | Stats target | Description
-----------+--------------------------+---------------------------------------------------+----------+--------------+-------------
name      | text                     |                                                   | extended |              |
company   | integer                  |                                                   | plain    |              |
```

Visiting http://localhost:1337/user shows
```
[
  {
    "name": "Test",
    "id": 1,
    "createdAt": "2015-04-08T15:25:02.000Z",
    "updatedAt": "2015-04-08T15:25:02.000Z"
  }
]
```

Create a company
```
(venv)~/D/shwhy $ http --json POST http://localhost:1337/company name=Company
HTTP/1.1 201 Created
Access-Control-Allow-Credentials:
Access-Control-Allow-Headers:
Access-Control-Allow-Methods:
Access-Control-Allow-Origin:
Connection: keep-alive
Content-Length: 120
Content-Type: application/json; charset=utf-8
Date: Wed, 08 Apr 2015 15:26:06 GMT
Vary: X-HTTP-Method-Override
X-Powered-By: Sails <sailsjs.org>
set-cookie: sails.sid=s%3A8yHbCb5NBAKK9CvPDx1zbHhEJsmTy4O9.vnwqpmQ%2BLKS4icvzXSnUYxkdOFdAaNnhiEZwwGqTn2A; Path=/; HttpOnly

{
    "createdAt": "2015-04-08T15:26:06.000Z",
    "id": 1,
    "name": "Company",
    "updatedAt": "2015-04-08T15:26:06.000Z"
}

```

Now visiting http://localhost:1337/user shows the company association.
```
[
  {
    "company": {
      "name": "Company",
      "id": 1,
      "createdAt": "2015-04-08T15:26:06.000Z",
      "updatedAt": "2015-04-08T15:26:06.000Z"
    },
    "name": "Test",
    "id": 1,
    "createdAt": "2015-04-08T15:25:02.000Z",
    "updatedAt": "2015-04-08T15:25:02.000Z"
  }
]
```
