# Solution

## Problem with the current implementation

- Need to run through a loop to get the individual model running
- shell script is required making it more complicated
- Both Dev abd Prod environment are diverging, DEV should almost always be a close replica of what prod looks like atleast in structure.
- Purpose of manifest.yml file is unclear.
- Current state of solution doesn't utilise any parallel processing, where we can declare threads and run multiple models at the same time.
- Cross Google Project models are considered bad practice from dbt labs.

## Simple Solution

- Make the structure of the dev/nonprod/prod the same. So it will have the same tables and dataset.
- Same naming convention.
- Replace the bash script with 14 sql files under models directory.
  - This will make the project scalable in the long run.
  - Provide more granular control over the models.
  - Potential to do some clean up in the dbt transformation stage.
- Repurpose `manifest.yml` to `profiles.yml`

### Local Run

```bash
dbt run --profiles-dir . --project-dir . --target dev-uk # for dev local run
dbt run --profiles-dir . --project-dir . --target preprod-ca # for preprod local run
dbt run --profiles-dir . --project-dir . --target prod-ca # for prod local run
```
Since all the projects are using the same Service account, we will need to change the service account in the profiles.yml file or provide access to service account to
all three environments


## Complicated Solution

- Don't change any of the structure in the dev/prod environment.
- Create a dbt macro that can run on dbt start up macro.
  - Create another yml file with a similar structure to `table_config.yml`.
  - This macro will have if then statements to loop over the different tables in different dataset
  - With a single select \* statement.

### Drawbacks

Complicated solution is quite hard to debug which is the same case with the bash script.
Its not really scalable and will slow things down.
Its process time is sequential.
Using dbt to just renaming tables and dataset is not best use or considered good practice.


### Each file description:

What are each of the files for?

- models/: all the files under this folder are sql models that are the transformation of the raw data.

- schema.yml: this file declares which tables to be use from which database and which dataset.
				- In this project the database and schema/dataset are based on the environment variables provided at the run time.
				- It also declares the identifiers for the tables in the data warehouse and the alias which will be used in this project

- dbt_project.yml: this is a standard yaml file that tells how the dbt core cli on how the project is structured.
					- In case of this project everything is almost identical to a standard dbt project other than the locations of macro folder location.
					- It also tells how the tables are suppose to materialise via the dbt models
					- In general This should be the main configuration file for a dbt project. It defines the project name, version, file paths, and various settings that control how dbt interacts with the project.

- manifest.yml: This is a nonstandard file in respect to dbt project, the structure of the file is exactly the same as dbt’s profile.yml file.
				- In general This is an automatically generated artifact by dbt after each run, containing metadata about the dbt project. 
					It includes details about the compiled SQL, dependencies, and configurations for each model, seed, and snapshot within the project. 

- This should be renamed to profiles.yml

-dbt_command_loop.sh: This shell script orchestrates running dbt commands based on configurations provided.
