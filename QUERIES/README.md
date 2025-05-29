# Useful Queries (T-SQL)

> <i>by Alberto Cesar <gato.playero@proton.me></i>

<hr>

## Table of Contents
1. **[Retrieve Roles and Granted Permissions in AzSQL](#RetrieveRolesandGrantedPermissionsinAzSQL)**
2. **[Obfuscate Data](#ObfuscateData)**

## <font style="Color:blue;">Retrieve&nbsp;Roles&nbsp;and&nbsp;Granted&nbsp;Permissions&nbsp;in&nbsp;AzSQL&#160;</font>

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

## <font style="Color:blue;">Obfuscate&nbsp;Data</font>
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