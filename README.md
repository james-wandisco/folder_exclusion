# folder_exclusion
Shows how to exclude multiple specific folders from a LDM migration. 

Example: Exclude 700 out of 1700.

My source has directory /warehouse/tablespace/external/hive/database_1700_tables.db/

This directory contains 1700 folders. I want to replicate only 1000 of these folders.

### Compile a list of excluded folders.
This is a simple **example** of folders named table1001 to table1700

You would have to compile the list with accurate names of the table folders you need to exclude.

Example file for testing...
```
for i in {1001..1700}; do echo table$i >> folder_list.txt; done
```

### Add individual exclusion template for each unwanted folder.
![](https://github.com/james-wandisco/folder_exclusion/blob/main/ezgif-5-eab58faebb.gif)
Add an exclusion template for all folders in the folders_list.txt file.
```
while read -r i; do curl -X 'PUT' "http://10.6.123.132:18080/exclusions/regex/db_fulldatabase_exclusion_table$i?description=Excludefolder_table$i&patternType=JAVA_PCRE&regex=$i"; done < folder_list.txt
```

#### Alternatively, (more risk of human error) Add a single exclusion with regex for ALL unwanted folders.

```
regex=""; while read -r i; do regex=$i%20%7C%20$regex; done < folder_list.txt; echo $regex
curl -X 'PUT' "http://10.6.123.132:18080/exclusions/regex/db_fulldatabase_exclusion_full?description=Excludefolder_table$i&patternType=JAVA_PCRE&regex=$regex"
```


### LDM rule. 
The rule can be created to include the full database folder.
Add the rule but don't autostart or start.
```
curl -X 'PUT' 'http://10.6.123.132:18080/migrations/fulldatabase?path=/warehouse/tablespace/external/hive/database_1700_tables.db&source=SourceHDFS&target=TragetHDFS&actionPolicy=com.wandisco.livemigrator2.migration.OverwriteActionPolicy&autoStart=false'
```

### Add exclusions to this rule.
Exclusions have been created but need to be added to the rule.
```
while read -r i; do curl -X 'PUT' "http://10.6.123.132:18080/migrations/fulldatabase/exclusions/db_fulldatabase_exclusion_table$i"; done < folder_list.txt
```

### Start the migration.
Start the migration, and check results. 
```
curl -X 'POST' 'http://10.6.123.132:18080/migrations/fulldatabase/start'
```

My target should not have folders 1001-1700, only 1-1000
```
hdfs dfs -ls /warehouse/tablespace/external/hive/database_1700_tables.db/
```

![](https://github.com/james-wandisco/folder_exclusion/blob/main/screen.gif)

### Extras
Check exclusion template list.
```
curl -X 'GET' 'http://10.6.123.132:18080/exclusions/userDefined'
```
```
curl -X 'GET' 'http://10.6.123.132:18080/exclusions/userDefined' | grep regex | sort
```
Add a single exclusion template. Name=exclusiontester, Description=JustATest, Type=JAVA Perl Compatible RegEx, Regex Pattern=SecretFolder1
```
curl -X 'PUT' "http://10.6.123.132:18080/exclusions/regex/exclusiontester?description=JustATest&patternType=JAVA_PCRE&regex=SecretFolder1"
```
Remove a single exclusion template with Name=exclusiontester
```
curl -X 'DELETE' "http://10.6.123.132:18080/exclusions/exclusiontester"
```
Check exclusions applied to a rule.
```
curl -s -X 'GET' "http://10.6.123.132:18080/migrations/fulldatabase"
```
```
curl -s -X 'GET' "http://10.6.123.132:18080/migrations/fulldatabase" | grep regex | awk '{print $3}' | sed 's/^"\([^"]*\).*/\1/' | sort
```
Add a single exclusion template to a rule called fulldatabase. 
```
curl -X 'PUT' "http://10.6.123.132:18080/migrations/fulldatabase/exclusions/exclusiontester"
```
Remove exclusions from a rule.
```
curl -X 'DELETE' "http://10.6.123.132:18080/migrations/fulldatabase/exclusions/db_fulldatabase_exclusion_table1001"
```
```
while read -r i; do curl -X 'DELETE' "http://10.6.123.132:18080/migrations/fulldatabase/exclusions/db_fulldatabase_exclusion_table$i"; done < folder_list.txt
```
Remove exclusion templates.
```
while read -r i; do curl -X 'DELETE' "http://10.6.123.132:18080/exclusions/db_fulldatabase_exclusion_table$i"; done < folder_list.txt
```
Stop migration.
```
curl -X 'POST' 'http://10.6.123.132:18080/migrations/fulldatabase/stop'
```
Remove migration.
```
curl -X 'DELETE' 'http://10.6.123.132:18080/migrations/fulldatabase' 

```

### Reset my test.
Remove exclusions from my rule.
```
while read -r i; do curl -X 'DELETE' "http://10.6.123.132:18080/migrations/fulldatabase/exclusions/db_fulldatabase_exclusion_table$i"; done < folder_list.txt;
```
Remove exclusion templates
```
while read -r i; do curl -X 'DELETE' "http://10.6.123.132:18080/exclusions/db_fulldatabase_exclusion_table$i"; done < folder_list.txt
```
Stop migration
```
curl -X 'POST' 'http://10.6.123.132:18080/migrations/fulldatabase/stop'
```
Delete migration
```
curl -X 'DELETE' 'http://10.6.123.132:18080/migrations/fulldatabase' 
```

delete target data
```
hdfs dfs -rm -r /warehouse/tablespace/external/hive/database_1700_tables.db
```




 




