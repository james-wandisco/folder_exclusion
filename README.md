# folder_exclusion
Shows how to exclude multiple specific folders from a LDM migration. Example: Exclude 700 out of 1700
My source has directory /warehouse/tablespaceexternal/hive/database_1700_tables.db/
This directory contains 1700 folders.
I want to replicate only 1000 of these folders.

### Compile a list of excluded folders.
This is a simple **example** of folders named folder1001 to folder1700
You would have to compile the list with accurate names of the table folders you need to exclude.

```
for i in {1001..1700}; do echo folder$i >> folder_list.txt; done
```

### Add exclusion templates.
Add an exclusion template for all folders in the folders_list.txt file.
```
while read -r i; do curl -X 'PUT' "http://10.6.123.132:18080/exclusions/regex/db_fulldatabase_exlusion_table$i?description=Excludefolder_table$i&patternType=JAVA_PCRE&regex=$i"; done < folder_list.txt
```

### LDM rule. 
The rule can be created to include the full database folder.
Add the rule but don't autostart or start.
```
curl -X 'PUT' 'http://10.6.123.132:18080/migrations/fulldatabase?path=/warehouse/tablespaceexternal/hive/database_1700_tables.db&source=source_jhugh02&target=target_jhugh01&actionPolicy=com.wandisco.livemigrator2.migration.OverwriteActionPolicy&autoStart=false'
```

### Add exclusions to this rule.
Exclusions have been created but need to be added to the rule.
```
while read -r i; do curl -X 'PUT' "http://10.6.123.132:18080/migrations/fulldatabase/exclusions/db_fulldatabase_exlusion_table$i"; done < folder_list.txt
```

### Start the migration.
Start the migration, and check results. 
```
curl -X 'POST' 'http://10.6.123.132:18080/migrations/fulldatabase/start'
```

My target should not have folders 1001-1700, only 1-1000
```
hdfs dfs -ls /warehouse/tablespaceexternal/hive/database_1700_tables.db/
```


### Extras
Check exclusion template list.
```
curl -X 'GET' 'http://10.6.123.132:18080/exclusions/userDefined'

```
```
curl -X 'GET' 'http://10.6.123.132:18080/exclusions/userDefined' | grep regex | sort
```
Check exclusions applied to a rule.
```
curl -s -X 'GET' "http://10.6.123.132:18080/migrations/fulldatabase"
```
```
curl -s -X 'GET' "http://10.6.123.132:18080/migrations/fulldatabase" | grep regex | awk '{print $3}' | sed 's/^"\([^"]*\).*/\1/' | sort
```
Remove exclusions from a rule.
```
curl -X 'DELETE' "http://10.6.123.132:18080/migrations/fulldatabase/exclusions/db_fulldatabase_exlusion_table1001"
```
```
while read -r i; do curl -X 'DELETE' "http://10.6.123.132:18080/migrations/fulldatabase/exclusions/db_fulldatabase_exlusion_table$i"; done < folder_list.txt
```
Remove exclusion templates.
```
while read -r i; do curl -X 'DELETE' "http://10.6.123.132:18080/exclusions/db_fulldatabase_exlusion_table$i"; done < folder_list.txt
```
Stop migration.
```
curl -X 'POST' 'http://10.6.123.132:18080/migrations/fulldatabase/stop'
```
Remove migration.
```
curl -X 'DELETE' 'http://10.6.123.132:18080/migrations/fulldatabase' 

```



 




