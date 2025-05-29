# Useful Queries (T-SQL)

> <i>by Alberto Cesar <gato.playero@proton.me></i>

<hr>

## Table of Contents
1. **[Retrieve roles and granted permissions in AzSQL](#RetrieveRolesandGrantedPermissionsinAzSQL)**
2. **[Obfuscate Data](#ObfuscateData)**
3. **[Truncate date for grouping and comparing](#Truncatedateforgroupingandcomparing)**
<!--
4. **[](#)**
5. **[](#)**
6. **[](#)**
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