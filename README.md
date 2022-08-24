# folder_exclusion
Shows how to exclude multiple specific folders from a LDM migration. Example: Exclude 700 out of 1700

### Compile a list of excluded folders.
This is a simple example of folders named folder1001 to folder1700

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


