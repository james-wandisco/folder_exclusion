
# LDM, adding more precise exclusion example.

Do these examples in a test directory or lab until fully accurate.

### 1. Pick a test directory.
```
/example
/example/dontneed1
/example/neededfolder1
/example/neededfolder1/folder
/example/neededfolder1/folder/dontneed
/example/neededfolder1/folder2
/example/neededfolder1/folder2/dontneedthis
/example/neededfolder2
```

We will exclude '/example/neededfolder1/folder/dontneed' but keep 'dontneed1' and 'dontneedthis'
We want the match only on the absolute folder.

### 2. Add a single exclusion for a folder you don’t want.

Get the correct regex, you can use https://regex101.com/.

![img_1.png](img_1.png)

For the curl command we can change the '/' characters to URL encoded character

See https://www.w3schools.com/tags/ref_urlencode.ASP for a list of all URL encoded characters.

```
curl -X 'PUT' "http://10.6.123.132:18080/exclusions/regex/example_exclusion?description=example_exclusion&patternType=JAVA_PCRE&regex=example%2Fneededfolder1%2Ffolder%2Fdontneed"
```

### 3. Create the migration rule for path = /example
```
curl -X 'PUT' 'http://10.6.123.132:18080/migrations/example?path=/example&source=SourceHDFS&target=TragetHDFS&actionPolicy=com.wandisco.livemigrator2.migration.OverwriteActionPolicy&autoStart=false'
```
### 4. Add your exclusions to the migration rule = example
```
curl -X 'PUT' "http://10.6.123.132:18080/migrations/example/exclusions/example_exclusion"
```
### 5. Run the migration.
```
curl -X 'POST' 'http://10.6.123.132:18080/migrations/example/start'
```

### 6. Check the target.
```
hdfs dfs -ls -R /example
```

### Reset
```
curl -X 'POST' 'http://10.6.123.132:18080/migrations/example/stop'
curl -X 'DELETE' 'http://10.6.123.132:18080/migrations/example'
curl -X 'DELETE' "http://10.6.123.132:18080/exclusions/example_exclusion"
curl -s -X 'GET' 'http://10.6.123.132:18080/exclusions/userDefined'
hdfs dfs -rm -r /example
```

### References
https://regex101.com/

https://www.w3schools.com/tags/ref_urlencode.ASP

