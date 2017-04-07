# api documentation for  [mysql-activerecord (v0.8.6)](https://github.com/martintajur/node-mysql-activerecord)  [![npm package](https://img.shields.io/npm/v/npmdoc-mysql-activerecord.svg?style=flat-square)](https://www.npmjs.org/package/npmdoc-mysql-activerecord) [![travis-ci.org build-status](https://api.travis-ci.org/npmdoc/node-npmdoc-mysql-activerecord.svg)](https://travis-ci.org/npmdoc/node-npmdoc-mysql-activerecord)
#### A lightweight MySQL query builder on top of the node-mysql module.

[![NPM](https://nodei.co/npm/mysql-activerecord.png?downloads=true)](https://www.npmjs.com/package/mysql-activerecord)

[![apidoc](https://npmdoc.github.io/node-npmdoc-mysql-activerecord/build/screenCapture.buildNpmdoc.browser._2Fhome_2Ftravis_2Fbuild_2Fnpmdoc_2Fnode-npmdoc-mysql-activerecord_2Ftmp_2Fbuild_2Fapidoc.html.png)](https://npmdoc.github.io/node-npmdoc-mysql-activerecord/build/apidoc.html)

![npmPackageListing](https://npmdoc.github.io/node-npmdoc-mysql-activerecord/build/screenCapture.npmPackageListing.svg)

![npmPackageDependencyTree](https://npmdoc.github.io/node-npmdoc-mysql-activerecord/build/screenCapture.npmPackageDependencyTree.svg)



# package.json

```json

{
    "author": {
        "name": "Martin Tajur",
        "email": "martin@tajur.ee"
    },
    "bugs": {
        "url": "https://github.com/martintajur/node-mysql-activerecord/issues"
    },
    "contributors": [
        {
            "name": "Daniel Bretoi",
            "email": "daniel@bretoi.com"
        },
        {
            "name": "Kyle Farris",
            "email": "kyle@chomponllc.com"
        },
        {
            "name": "Daehyub Kim",
            "email": "lateau@gmail.com"
        }
    ],
    "dependencies": {
        "mysql": "2.5.2"
    },
    "description": "A lightweight MySQL query builder on top of the node-mysql module.",
    "devDependencies": {},
    "directories": {},
    "dist": {
        "shasum": "47f012c30bdd052a87711ca1e99e0fe8ec71e4ed",
        "tarball": "https://registry.npmjs.org/mysql-activerecord/-/mysql-activerecord-0.8.6.tgz"
    },
    "engines": {
        "node": "*"
    },
    "gitHead": "22d61ae244f7436747ecd700bd262fb705a3235e",
    "homepage": "https://github.com/martintajur/node-mysql-activerecord",
    "main": "./",
    "maintainers": [
        {
            "name": "martin.tajur",
            "email": "martin@tajur.ee"
        }
    ],
    "name": "mysql-activerecord",
    "optionalDependencies": {},
    "readme": "ERROR: No README data found!",
    "repository": {
        "type": "git",
        "url": "git://github.com/martintajur/node-mysql-activerecord.git"
    },
    "scripts": {},
    "version": "0.8.6"
}
```



# <a name="apidoc.tableOfContents"></a>[table of contents](#apidoc.tableOfContents)

#### [module mysql-activerecord](#apidoc.module.mysql-activerecord)
1.  [function <span class="apidocSignatureSpan">mysql-activerecord.</span>Adapter (settings)](#apidoc.element.mysql-activerecord.Adapter)
1.  [function <span class="apidocSignatureSpan">mysql-activerecord.</span>Pool (settings)](#apidoc.element.mysql-activerecord.Pool)



# <a name="apidoc.module.mysql-activerecord"></a>[module mysql-activerecord](#apidoc.module.mysql-activerecord)

#### <a name="apidoc.element.mysql-activerecord.Adapter"></a>[function <span class="apidocSignatureSpan">mysql-activerecord.</span>Adapter (settings)](#apidoc.element.mysql-activerecord.Adapter)
- description and source-code
```javascript
Adapter = function (settings) {

	var mysql = require('mysql');

	var initializeConnectionSettings = function () {
		if(settings.server) {
			settings.host = settings.server;
		}
		if(settings.username) {
			settings.user = settings.username;
		}

		if (!settings.host) {
			throw new Error('Unable to start mysql-activerecord - no server given.');
		}
		if (!settings.port) {
			settings.port = 3306;
		}
		if (!settings.user) {
			settings.user = '';
		}
		if (!settings.password) {
			settings.password = '';
		}
		if (!settings.database) {
			throw new Error('Unable to start mysql-activerecord - no database given.');
		}

		return settings;
	};

	var connection;
	var connectionSettings;
	var pool;

	if (settings && settings.pool) {
		pool = settings.pool.pool;
		connection = settings.pool.connection;
	} else {
		connectionSettings = initializeConnectionSettings();
		connection = new mysql.createConnection(connectionSettings);
	}

	if (settings.charset) {
		connection.query('SET NAMES ' + settings.charset);
	}

	var whereClause = {},
		selectClause = [],
		orderByClause = '',
		groupByClause = '',
		havingClause = '',
		limitClause = -1,
		offsetClause = -1,
		joinClause = [],
		lastQuery = '';
	
	var resetQuery = function(newLastQuery) {
		whereClause = {};
		selectClause = [];
		orderByClause = '';
		groupByClause = '';
		havingClause = '',
		limitClause = -1;
		offsetClause = -1;
		joinClause = [];
		lastQuery = (typeof newLastQuery === 'string' ? newLastQuery : '');
		rawWhereClause = {};
		rawWhereString = {};
	};
	
	var rawWhereClause = {};
	var rawWhereString = {};
	
	var escapeFieldName = function(str) {
		return (typeof rawWhereString[str] === 'undefined' && typeof rawWhereClause[str] === 'undefined' ? ''' + str.replace('.',''.'') + ''' :
str);
	};
	
	var buildDataString = function(dataSet, separator, clause) {
		if (!clause) {
			clause = 'WHERE';
		}
		var queryString = '', y = 1;
		if (!separator) {
			separator = ', ';
		}
		var useSeparator = true;
		
		var datasetSize = getObjectSize(dataSet);
		
		for (var key in dataSet) {
			useSeparator = true;
			
			if (dataSet.hasOwnProperty(key)) {
				if (clause == 'WHERE' && rawWhereString[key] == true) {
					queryString += key;
				}
				else if (dataSet[key] === null) {
					queryString += escapeFieldName(key) + (clause == 'WHERE' ? " is NULL" : "=NULL");
				}
				else if (typeof dataSet[key] !== 'object') {
					queryString += escapeFieldName(key) + "=" + connection.escape(dataSet[key]);
				}
				else if (typeof dataSet[key] === 'object' && Object.prototype.toString.call(dataSet[key]) === '[object Array]' && dataSet[key
].length > 0) {
					queryString += escapeFieldName(key) + ' in ("' + dataSet[key].join('", "') + '")';
				}
				else {
					useSeparator = false;
					datasetSize = datasetSize - 1;
				}
				
				if (y < datasetSize && useSeparator) {
					queryString += separator;
					y++;
				}
			}
		}
		if (getObjectSize(dataSet) > 0) {
			queryString = ' ' + clause + ' ' + queryString;
		}
		return queryString;
	};

	var buildJoinString = function() {
		var joinString = '';

		for (var i = 0; i < joinClause.length; i++) {
			joinString += (joinClause[i].direction !== '' ? ' ' + joinClause[i].direction : '') + ' JOIN ' + escapeFieldName(joinClause[i
].table) + ' ON ' + joinClause[i].relation;
		}

		return joinString;
	};

	var mergeObjects = function() {
		for (var i = 1; i < arguments.length; i++) {
			for (var key in arguments[i]) {
				if (arguments[i].hasOwnProperty(key)) {
					arguments[0][key] = arguments[i][key];
				}
			}
		}
		return arguments[0];
	};

	var getObjectSize = function(object) {
		var size = 0;
		for (var key in object) {
			if (object.hasOwnProperty(key)) {
				size++;
			}
		}
		return size;
	};

	var trim = function (s) {
		var l = 0, r = s.length - 1;
		while (l < s.length && s[l] == ' ') {
			l++;
		}
		while (r > l && s[r] == ' ') {
			r-=1;
		}
		return s.substring(l, r + 1);
	};

	this.connectionSettings = function() { return connectionSettings; };
	this.connection = function() { return connection; };

	this.where = function(wher ...
```
- example usage
```shell
...
	npm install mysql-activerecord


Get started
-----------

	var Db = require('mysql-activerecord');
	var db = new Db.Adapter({
		server: 'localhost',
		username: 'root',
		password: '12345',
		database: 'test',
		reconnectTimeout: 2000
	});
...
```

#### <a name="apidoc.element.mysql-activerecord.Pool"></a>[function <span class="apidocSignatureSpan">mysql-activerecord.</span>Pool (settings)](#apidoc.element.mysql-activerecord.Pool)
- description and source-code
```javascript
Pool = function (settings) {
	if (!mysqlPool) {
		var mysql = require('mysql');

		var poolOption = {
			createConnection: settings.createConnection,
			waitForConnections: settings.waitForConnections,
			connectionLimit: settings.connectionLimit,
			queueLimit: settings.queueLimit
		};
		Object.keys(poolOption).forEach(function (element) {
			// Avoid pool option being used by mysql connection.
			delete settings[element];
			// Also remove undefined elements from poolOption
			if (!poolOption[element]) {
				delete poolOption[element];
			}
		});

		// Confirm settings with Adapter.
		var db = new Adapter(settings);
		var connectionSettings = db.connectionSettings();

		Object.keys(connectionSettings).forEach(function (element) {
			poolOption[element] = connectionSettings[element];
		});

		mysqlPool = mysql.createPool(poolOption);
		mysqlCharset = settings.charset;
	}

	this.pool = function () {
		return mysqlPool;
	};

	this.getNewAdapter = function (responseCallback) {
		mysqlPool.getConnection(function (err, connection) {
			if (err) {
				throw err;
			}
			var adapter = new Adapter({
				pool: {
					pool: mysqlPool,
					enabled: true,
					connection: connection
				},
				charset: mysqlCharset
			});
			responseCallback(adapter);
		});
	};

	this.disconnect = function (responseCallback) {
		this.pool().end(responseCallback);
	};

	return this;
}
```
- example usage
```shell
...
Pooling connections
===================

Single or multiple connections can be pooled with the Pool object.

	var Db = require('mysql-activerecord');

	var pool = new Db.Pool({
		server: 'localhost',
		username: 'root',
		password: '12345',
		database: 'test'
	});
	
	pool.getNewAdapter(function(db) {
...
```



# misc
- this document was created with [utility2](https://github.com/kaizhu256/node-utility2)
