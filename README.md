## Repair Utilities
Allows you to perform sugar repair, including repair roles and reports, directly from the CLI console (shell). This utility also include osy_repair.php but launches it ONLY in case where the sugarcrm CLI commands are not available.

### How to use?
From the Sugarcrm folder launch:
**./bin/sugarcrm osy:repair -h**
  
```
Usage:
  osy: repair [options]
 
Options:
  -e, --execute_query if set, executes the queries to database
  -d, --dont_execute_query if set, does not execute the queries to database
  -r --dont_execute_relationships if set, does not execute Rebuild Relationships
  -l --dont_execute_roles if set, does not execute Repair Roles
  -h, --help Display this help message
  -q, --quiet Do not output any message
  -V, --version Display this application version
      --ansi Force ANSI output
      --no-ansi Disable ANSI output
  -n, --no-interaction Do not ask any interactive question
      --profile Display timing and memory usage information
  -v | vv | vvv, --verbose Increase the verbosity of messages: 1 for normal output, 2 for more verbose output and 3 for debug
```

### Examples
**./bin/sugarcrm osy:repair**
1. executes "Quick Repair and Rebuild" and asks whether to execute the queries or not
1. performs "Rebuild Relationships"
1. performs "Repair Roles"

**./bin/sugarcrm osy:repair -e**
1. executes "Quick Repair and Rebuild" and executes the queries directly without waiting for confirmation
1. performs "Rebuild Relationships"
1. performs "Repair Roles"

**./bin/sugarcrm osy:repair -d**
1. executes "Quick Repair and Rebuild" and doest not executes the queries directly to database
1. performs "Rebuild Relationships"
1. performs "Repair Roles"


### Exit code
In case of error in the execution of the queries, an exit code returns > 0
- "0" = all ok
- "3" = error in query execution
- any other character = generic error

### In case of problems
Give attention to the file/folder permissions:
- The script is sometimes not executable do chmod +x bin/sugarm
- The user running the script must have access to the files (read vardef etc ..)
- The script modifies/regenerates cache, check that www-data can read the newly generated files
