# Useful Queries (T-SQL)

> <i>by Alberto Cesar <gato.playero@proton.me></i>

<hr>

## Table of Contents
1. **[Retrieve roles and granted permissions in AzSQL](#RetrieveRolesandGrantedPermissionsinAzSQL)**
2. **[Obfuscate Data](#ObfuscateData)**
3. **[Truncate date for grouping and comparing](#Truncatedateforgroupingandcomparing)**
4. **[Parse/Deflate JASON](#ParseDeflateJASON)**
5. **[Flattening JSON](#FlatteningJSON)**
6. **[Parse/Deflate XML](#ParseDeflateXML)**
<!--
7. **[](#)**
8. **[](#)**
9. **[](#)**
10. **[](#)**
11. **[](#)**
-->




## <font style="Color:blue;">Retrieve&nbsp;roles&nbsp;and&nbsp;granted&nbsp;permissions&nbsp;in&nbsp;AzSQL&#160;</font>

```sql
/* Get db users and their roles for Azure SQL */
SELECT 
		[ServerName]		=	@@ServerName
	,	[DatabaseName]		=	DB_NAME()
	,	[PrincipalName]		=	[dbp].[name]
	,	[PrincipalType]		=	[dbp].[type_desc]
	,	[RoleName]			=	[r].[name]
	,	[RoleState]			=	[dp].[state_desc]
	,	[PermissionName]	=	NULL
	,	[PermissionState]	=	NULL
	,	[PermissionClass]	=	NULL
	,	[ObjectName]		=	NULL
FROM
	[sys].[database_role_members]	[dbrm]
JOIN
	[sys].[database_principals] [dbp]
ON
	[dbrm].[member_principal_id] = [dbp].[principal_id]
JOIN
	[sys].[database_principals] [r]
ON
	[dbrm].[role_principal_id] = [r].[principal_id]
LEFT JOIN
	[sys].[database_permissions] [dp]
ON
	[dbp].[principal_id] = [dp].[grantee_principal_id]
WHERE
	[dbp].[type_desc] IN
						(
								'SQL_USER'
							,	'WINDOWS_USER'
							,	'WINDOWS_GROUP'
							,	'EXTERNAL_USER'
							,	'EXTERNAL_GROUP'
							,	'APPLICATION_ROLE'
						) -- You can apply filter(s) for users and groups

UNION
/* Get db users and the granted permissions for Azure SQL */
SELECT
		[ServerName]		=	@@ServerName
	,	[DatabaseName]		=	DB_NAME()
	,	[PrincipalName]		=	[dbp].[name]
	,	[PrincipalType]		=	[dbp].[type_desc]
	,	[RoleName]			=	NULL
	,	[RoleState]			=	NULL
	,	[PermissionName]	=	[dbpr].[permission_name]
	,	[PermissionState]	=	[dbpr].[state_desc]
	,	[PermissionClass]	=	[dbpr].[class_desc]
	,	[ObjectName]		=	OBJECT_NAME([dbpr].[major_id])
FROM
	[sys].[database_permissions] [dbpr]
JOIN
	[sys].[database_principals] [dbp]
ON
	[dbpr].[grantee_principal_id] = [dbp].[principal_id]
WHERE
    [dbp].[type_desc] IN
						(
								'SQL_USER'
							,	'WINDOWS_USER'
							,	'WINDOWS_GROUP'
							,	'EXTERNAL_USER'
							,	'EXTERNAL_GROUP'
							,	'APPLICATION_ROLE'
						) -- You can apply filter(s) for users and groups
```

## <font style="Color:blue;">Obfuscate&nbsp;data</font>
```sql
DECLARE @TopCat AS TABLE
					(
						[Id] [int] IDENTITY(1,1) NOT NULL
					,	[Name] [varchar](50) NULL
					,	[Alias] [varchar](50) NULL
					);
 
INSERT INTO @TopCat ([Name], [Alias]) VALUES ('Don Gato',		'Top Cat');
INSERT INTO @TopCat ([Name], [Alias]) VALUES ('Demostenes',		'The Brain');
INSERT INTO @TopCat ([Name], [Alias]) VALUES ('Benito Bodoque',	'Benny the Ball');
INSERT INTO @TopCat ([Name], [Alias]) VALUES ('Panza',			'Fancy-Fancy');
INSERT INTO @TopCat ([Name], [Alias]) VALUES ('Espanto',		'Spook');
INSERT INTO @TopCat ([Name], [Alias]) VALUES ('Cucho',			'Choo-Choo');

/* Algorithms Available >> MD2 | MD4 | MD5 | SHA | SHA1 | SHA2_256 | SHA2_512  */
SELECT
		[Name]
	,	[Option1]	=	CONVERT(NVARCHAR(MAX), HASHBYTES('SHA2_512', CONVERT(NVARCHAR(MAX), [Alias])), 2)
	,	[Option2]	=	HASHBYTES('SHA2_512', CONVERT(NVARCHAR(MAX), [Alias]))
FROM
	@TopCat
ORDER BY
	[Id] ASC;
```

## <font style="Color:blue;">Truncate&nbsp;date&nbsp;for&nbsp;grouping&nbsp;and&nbsp;comparing</font>
```sql
/* Equivalent to bin() in KQL */
DECLARE	@d datetime2	=	'2021-12-08 11:30:15.1234567';
SELECT	@d				=	sysdatetimeoffset();
SELECT
		'_'				=	@d
	,	'Year'			=	DATETRUNC(year, @d)
	,	'Quarter'		=	DATETRUNC(quarter, @d)
	,	'Month'			=	DATETRUNC(month, @d)
	,	'Week'			=	DATETRUNC(week, @d) -- Using the default DATEFIRST setting value of 7 (U.S. English)
	,	'Iso_week'		=	DATETRUNC(iso_week, @d)
	,	'DayOfYear'		=	DATETRUNC(dayofyear, @d)
	,	'Day'			=	DATETRUNC(day, @d)
	,	'Hour'			=	DATETRUNC(hour, @d)
	,	'Minute'		=	DATETRUNC(minute, @d)
	,	'Second'		=	DATETRUNC(second, @d)
	,	'Millisecond'	=	DATETRUNC(millisecond, @d)
	,	'Microsecond'	=	DATETRUNC(microsecond, @d);
```

## <font style="Color:blue;">Parse/Deflate&nbsp;JASON</font>
```sql
DECLARE @tbl AS TABLE
					(
						[ID] [int] IDENTITY(1,1) NOT NULL
					,	[j] [nvarchar](MAX) NULL
					);
 
DECLARE @json NVARCHAR(MAX)
	= '{
			"firstName": "John",
			"lastName": "doe",
			"age": 26,
			"address": {
				"streetAddress": "naist street",
				"city": "Nara",
				"postalCode": "630-0192"
			},
			"phoneNumbers": [
				{
					"type": "iPhone",
					"number": "0123-4567-8888"
				},
				{
					"type": "home",
					"number": "0123-4567-8910"
				}
			]
		}';
 
INSERT INTO @tbl ([j]) VALUES (@json);
SELECT @json = NULL;
 
SELECT
		Core.*
	,	ARRAY.[Type]
	,	ARRAY.[Number]
FROM
	@tbl [tbl]
	OUTER APPLY
		OPENJSON([tbl].[j])
			WITH
			(
					FirstName NVARCHAR(25) '$.firstName'
				,	LastName NVARCHAR(25) '$.lastName'
				,	Age INT '$.age'
				,	streetAddress NVARCHAR(25) '$.address.streetAddress'
				,	city NVARCHAR(25) '$.address.city'
			) AS Core
		CROSS APPLY
		OPENJSON([tbl].[j], '$.phoneNumbers')
			WITH
			(
					[Type] NVARCHAR(25) '$.type'
				,	[Number] NVARCHAR(25) '$.number'
			) AS ARRAY;
```

## <font style="Color:blue;">Flattening&nbsp;JSON</font>
```
/* ••••••••••••••••••••••••••••••••••••••••••••••••••••••••••• */
DECLARE @TopCat AS TABLE
					(
						[Id] [int] IDENTITY(1,1) NOT NULL
					,	[Name] [varchar](50) NULL
					,	[Alias] [varchar](50) NULL
					,	[GroupColumn] [int] NULL
					);
 
INSERT INTO @TopCat ([Name], [Alias], [GroupColumn]) VALUES ('Don Gato',		'Top Cat',				0);
INSERT INTO @TopCat ([Name], [Alias], [GroupColumn]) VALUES ('Demostenes',		'The Brain',			1);
INSERT INTO @TopCat ([Name], [Alias], [GroupColumn]) VALUES ('Benito Bodoque',	'Benny the Ball',		2);
INSERT INTO @TopCat ([Name], [Alias], [GroupColumn]) VALUES ('Panza',			'Fancy-Fancy',			3);
INSERT INTO @TopCat ([Name], [Alias], [GroupColumn]) VALUES ('Espanto',			'Spook',				1);
INSERT INTO @TopCat ([Name], [Alias], [GroupColumn]) VALUES ('Cucho',			'Choo-Choo',			0);
/* ••••••••••••••••••••••••••••••••••••••••••••••••••••••••••• */
SELECT
		[Id]
	,	[Full_json]
	,	[NamesList_json]
FROM
	(
	SELECT
		[i].[Id]
		,[Full_json] = '[{' + STUFF((
										SELECT
											',{'
											+
												CASE WHEN
													1 = 1
												THEN
													'"Nombre":' + IIF([sj].[Name] IS NOT NULL, '"' + CONVERT([varchar](50), [sj].[Name]) + '"', 'null') + ','
												ELSE
													''
												END
											+ '"Apellido":' + IIF([sj].[Alias] IS NOT NULL, '"' + CONVERT([varchar](50), [sj].[Alias]) + '"', 'null') + ','
											+ '"ID":' + IIF([sj].[Id] IS NOT NULL, CONVERT([varchar], [sj].[Id]), 'null') + ''
											+ '}'
										FROM
											@TopCat [sj]
										WHERE
											[sj].[Id] = [i].[Id]
							GROUP BY
											[sj].[Id]
											,[sj].[Name]
											,[sj].[Alias]
										FOR XML
											PATH(''), TYPE
										).value('.[1]','NVARCHAR(MAX)'),1,2,'') + ']'
		,[NamesList_json] = '[' + STUFF((
										SELECT
											','
											+
												CASE WHEN
													[sj].[Name] IS NOT NULL
												THEN
													'"' + [sj].[Name] + '"'
												ELSE
													null
												END
										FROM
											@TopCat [sj]
										WHERE
											[sj].[GroupColumn] = [i].[GroupColumn]
										GROUP BY
											[sj].[Name]
										FOR XML
											PATH(''), TYPE
										).value('.[1]','NVARCHAR(MAX)'),1,1,'') + ']'
	FROM
		@TopCat [i]
	) [j]
GROUP BY
	[Id]
	,[Full_json]
	,[NamesList_json]
ORDER BY
	[Id] ASC
/* ••••••••••••••••••••••••••••••••••••••••••••••••••••••••••• */
DECLARE @VALUE AS [varchar](8000);
 
SELECT
	@VALUE = COALESCE(@VALUE + ',', '') + [Name]
FROM
	(
	SELECT 'Jehu' AS [Name]
	UNION SELECT 'Alberto' AS [Name]
	UNION SELECT 'Erilex' AS [Name]
	) AS [Temp]
 
SELECT [Flat] = @VALUE;
/* ••••••••••••••••••••••••••••••••••••••••••••••••••••••••••• */ 
SELECT
	SUBSTRING((
SELECT
	',' + [Name]
FROM
	(
	SELECT 'Jehu' AS [Name]
	UNION SELECT 'Alberto' AS [Name]
	UNION SELECT 'Erilex' AS [Name]
	) AS [Temp]
FOR XML PATH('')),2,200) AS [CSV]
/* ••••••••••••••••••••••••••••••••••••••••••••••••••••••••••• */
```

## <font style="Color:blue;">Parse/Deflate&nbsp;XML</font>
```sql
/* ~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~••~•~•~•~•~•~•~•~•~•~•~•~•~•~•~• */
DECLARE @xml XML;
/* ~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~••~•~•~•~•~•~•~•~•~•~•~•~•~•~•~• */
SELECT @xml = '
				<dimensions>
					<dimension name="height" value="0.14" /> 
					<dimension name="width"  value="12.77"/>
					<dimension name="width" value="12.77">fff</dimension>
				</dimensions>
				';
SELECT
	x.v.value('@name[1]', 'VARCHAR(100)') AS dimtype
	,x.v.value('@value[1]', 'VARCHAR(100)') AS dimvalue
	,x.v.value('/', 'VARCHAR(100)') as intvalue
FROM
	@xml.nodes('/dimensions/dimension') x(v);
/* ~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~••~•~•~•~•~•~•~•~•~•~•~•~•~•~•~• */
SELECT @xml = '
				<dimensions>
					<dimension name="height" value="0.14" /> 
					<dimension name="width" value="12.77"/> 
					<dimension name="depth" value="12.92"/>
				</dimensions>
				';
SELECT
	x.v.value('@name[1]', 'VARCHAR(100)') AS dimtype
	,x.v.value('@value[1]', 'VARCHAR(100)') AS dimvalue
FROM
	@xml.nodes('/dimensions/dimension[@name = "height" or @name = "width"]') x(v);
/* ~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~••~•~•~•~•~•~•~•~•~•~•~•~•~•~•~• */
DECLARE @demo TABLE  (columnXML xml);
insert into @demo (columnXML) values ('
									<dimensions>
										<dimension name="height" value="0.14" /> 
										<dimension name="width" value="12.77">valor interno 1</dimension>
										<dimension name="depth"	value="12.92"/>
									</dimensions>
									');
insert into @demo (columnXML) values ('
									<dimensions>
										<dimension name="height" value="0.15" /> 
										<dimension name="width" value="12.78">valor interno 2</dimension>
										<dimension name="depth"	value="12.93"/>
									</dimensions>
									');
SELECT
	x.v.value('@name[1]', 'VARCHAR(100)') AS dimtype
	,x.v.value('@value[1]', 'VARCHAR(100)') AS dimvalue
	,x.v.value('/', 'VARCHAR(100)') AS invalue
	,[columnXML].value('(/dimensions/dimension[@name = "width"])[1]', 'VARCHAR(100)') AS intvalue
FROM
	@demo
CROSS APPLY
	columnXML.nodes('/dimensions/dimension[@name = "height" or @name = "width"]') x(v);
/* ~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~••~•~•~•~•~•~•~•~•~•~•~•~•~•~•~• */
SELECT
	x.v.value('@name[1]', 'VARCHAR(100)') AS dimtype
	,x.v.value('@value[1]', 'VARCHAR(100)') AS dimvalue
FROM
	@demo
CROSS APPLY
	columnXML.nodes('/dimensions/dimension') x(v);
/* ~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~••~•~•~•~•~•~•~•~•~•~•~•~•~•~•~• */
SELECT 
	dimtype,
	dimvalue
FROM
	(
	SELECT
		x.v.value('@name[1]', 'VARCHAR(100)') AS dimtype
		,x.v.value('@value[1]', 'VARCHAR(100)') AS dimvalue
	FROM
		@demo
	CROSS APPLY
		columnXML.nodes('/dimensions/dimension') x(v)
	) [T]
WHERE
	T.dimtype in('height','width');
/* ~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~•~••~•~•~•~•~•~•~•~•~•~•~•~•~•~•~• */
```