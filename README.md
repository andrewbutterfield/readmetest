# Installation

1. Clone the repository
2. `cd` into the repository folder
3. `sudo bash ./install.sh`
4. `source ~/.bashrc`
5. `source env/bin/activate`
6. `pip install -r requirements.txt`

The `source` command above activates a python virtual environment.
When the environment is activated, an `(env)` will appear in your command prompt.
This environment must be activated for the running of our code.
To deactivate the environment simply use the `deactivate` command.

If upgrading from a previous release, perform all the installation steps again. The installation script will not reinstall any dependencies unnecessarily.

You may come across an issue where PEOS is trying to run a TCL script which it can't find. There is an environment variable in your `.bashrc` file called `$COMPILE_DIR` which points to `~/peos/models`. This is where that file is located. The problem is PEOS does not have permissions on that file.
This is fixed in Release 1, but there is a chance you may come across it whilst upgrading from Iteration 0.3.
To fix simply

```
cd ~/peos/models
chmod 777 peos_init.tcl
cd ~
```

#Dependencies

The application and features were developed to run on Ubuntu 16.04. No other operating systems or versions are currently supported.

A machine with at least 1GB of Memory is required to be able to load the entire DINTO ontology.

There are a few important Python libraries which are added when running the install script.
```
rdflib - A library for working with the Resource Description Framework language.
argparse - Allows for nice argument and passing of flags in a CLI.
pathlib - Allows for managing consistent filesystem paths for different OSs.
stemming - Used for nicely parsing drugs with a Stemming algorithm.
```

#Application

The complete application is capable of parsing medical pathways written in PML to extract any drug names and time restraints specified and it then uses this information to find any Drug Drug Interactions in the DINTO ontology which it presents to the user.

To run the application, execute the `python main.py` command from the home directory of the repo (the-pain-train-group-3). The program will immediately prompt you to enter the filepath to a PML medical pathway which it will attempt to extract information from. An example, `test/ddiExample.pml`, is included in the repo to test it. If an invalid file is entered, the application will tell the user `File does not exist` and exit.

After inputing the pathway the application will load the complete DINTO ontology, this will take a few minutes due to the large size of the dataset (if you encounter a `Killed` error after this command it means the machine you are attempting to run this application on does not have enough memory. It will load successfully on a machine with at least 1GB of memory).

When the application has finished loading in the DINTO ontology, it will extract all drug names from the input pathway file and then output them. Using the `test/ddiExample.pml` example, you should expect `['aluminium', 'dapsone']` to be output. These are all the drugs identified in the example pathway.

Next the application attempts to find any Drug Drug Interactions between all the drugs identified in the pathway. Using the entire DINTO ontology the application queries each pair of drugs which involves first finding the ids and descriptions for the individual drugs and then using these ids to search for any DDIs. Using the `test/ddiExample.pml` example, the program will first output

```
Querying drug name: aluminium
Successful query for drug: aluminium
Querying drug name: dapsone
Successful query for drug: dapsone
```

which indicates it's found the information for all the individual drugs found. After querying this data the application will compare each drug to find any DDIs, it will output

```
aluminium and dapsone comparison
DDI Found for aluminium / dapsone
```

which indicates a DDI has been found between the two drugs mentioned. After all combinations of drug pairs have been searched the application will output the complete data which is a JSON encoded list of DDIs which include the type, description and name of each DDIs and the information of the drugs involved in it. Using the `test/ddiExample.pml` the final expected output of the program will be

```
[
 {
  'DDI': 'aluminium/dapsone DDI', 
  'DDI_id': 'http://purl.obolibrary.org/obo/DINTO_05284', 
  'DDI_type_name': 'non-absorbable complex formation', 
  'DDI_type_description': 'A non-absorbable complex formation is a pharmacokinetic DDI mechanism which occurs when drug B binds to drug A leading to the formation of a higher molecular weight complex.'
  'DDI_type': 'http://purl.obolibrary.org/obo/DINTO_000422',  
  'first_drug_id': 'http://purl.obolibrary.org/obo/DINTO_DB01370', 
  'first_drug': {
   'definitions': ['A metallic element that has the atomic number 13, atomic symbol Al, and atomic weight 26.98. [PubChem]'],
   'name': 'aluminium', 
   'ids': ['http://purl.obolibrary.org/obo/DINTO_DB01370']
  },
  'second_drug_id': 'http://purl.obolibrary.org/obo/CHEBI_4325',
  'second_drug': {
   'definitions': ['Diphenylsulfone in which the hydrogen atom at the 4 position of each of the phenyl groups is substituted by an amino group. It is active against a wide  range of bacteria, but is mainly employed for its actions against Mycobacterium leprae, being used as part of multidrug regimens in the treatment of all forms of leprosy.'], 
   'name': 'dapsone', 
   'ids': ['http://purl.obolibrary.org/obo/CHEBI_4325']
  }
 }
]
```

#Feature List
Note: All commands assume you are executing them from the home directory of the repo (the-pain-train-group-3).

Three unit testing suites exist for ensuring all features work as intended, they can be run altogether or each unit test individually. 

To invoke the DINTO test suite, execute the `python test/dinto-tests.py` command. This will verbosely guide and desribe to the user everything that it is testing.

There are two test suites for PYOS. One for the PYOS object and another for the Time Parser. These can be run, from the home direcrory, with `python test/pyos_test.py` and `python test/pml_time_parser_test.py`. For acceptance test, we supply a default test pml file, `peos/build_test.pml`. 
###PML File Selection (4)
Allow for user input to select a pml file in the repository. An error occurs when a given file cannot be found in the current repository or is not a PML file. 
######Testing
Run `python pyos.py`.On startup, the program quiries the user for a file. The example file is `peos/build_test.pml`. Upon success, the program will render a list of options to choose. Upon failure, the program will show a corresponding error message and exit. 
###PML File Loading (4)
Loading the file into the PYOS object to allow for the analysis and extraction of drugs within the file. This is done on initialisation of a PYOS object and requires a file name as a string parameter.
######Testing
Testing PML File Loading, is the same as PML File Selection, as a file is immediately loaded after selection. Run `python pyos.py`. To show that a file is loaded, you can extract drugs from the file with option *4*.

###Running PML Analysis (15)
The PML analysis used is PMLcheck. The output of that is directly routed back to the user.

######Testing
Load in a PML file and select Option 1. The user will be outputted with content from PMLcheck. If an error occurs here, it will be an error within PML check itself. Loading `peos/built_test.pml` should return back
```
peos/build_test.pml:3: pain in action 'headache' is unprovided
peos/build_test.pml:4: infection in action 'headache' is unprovided
peos/build_test.pml:10: wellness in action 'earache' is unprovided
peos/build_test.pml:12: wellness in action 'earache' is not consumed
```

###On-Screen PML Reporting (6)
The program will continually update you on how it is progressing. An action should only report important information back to the user.

######Testing
Run `python pyos.py`. Enter the test file, `peos/build_test.pml`. You should be returned with an option menu
```
(1) Check if a PML file is valid using Peos' pmlcheck binary
(2) Bind an object to a pml resource
(3)Enter an action to be executed
(4)Extract script tags from given file.
(5)Inspect current state of PEOS
(q)Exit program
```
Each action will report back any relevant information to the user. 

###PML Log-file Generation (5)
All actions of the program are written to `logging/main.log`. If the log file does not previously exist, it will be generated. 

######Testing 
First run a few actions on a PML file to log. In this narrative, we bind all the neccessary values to actions and then run those actions. There is no need to run through this whole narrative to sufficiently test.
```
Load `peos/build_test.pml`
Select option *2*
Bind `pain throbbing`
Select option *2*
Bind `infection dangerous`
Select option *2*
Bind `wellness bliss`
Select option *3*
Run Action `headache start`
Select option *3*
Run Action `headache finish`
Select option *3*
Run Action `earache start`
Select option *3*
Run Action `earache finish`
Select option *q*
```
Use `tail logging/main.log` to view the end of the log file and what content was generated

###PML Error and Warning highlights (10)
This feature is implemented throughout all PML related features and ensures the program return informative information when errors occur either from the system of invalid through user input.

######Testing 
A good measure to test error handling is to attempt to load non-pml files or files that do not exist in the repo. Run `python pyos.py` and enter the filename `nothing` to load. The program will tell the user `File does not exist` and exit safely.

###Select specific OWL Ontology (4)
This feature allows the user to specify and select which OWL ontology is to be used for all queries throughout the program.

######Testing

To test this feature use the `-p` flag to give a specific owl file to load in. An example of this is `python dinto.py drug_id --drug aluminium -p ./DINTO/DINTO\ 1/DINTO_1.owl`. The first line output from this command should be `Loading Ontology: ./DINTO/DINTO 1/DINTO_1.owl` which indicates that the ontology was successfully selected.

As the contents of the specified ontology are not checked in this feature, it is not possible to occur any error except for failing to enter an argument after the `-p` flag which should output `dinto.py <command>: error: argument -p/--path: expected one argument`.

###Load Selected Ontology (4)
This feature allows the  program to load in a selected or default OWL ontology into a graph which can then be queried and utilised for identifying drugs and DDIs in subsequent features. The loading enforces that the ontology it's loading is correctly formed and of the right type.

######Testing

To test this feature simply run `python dinto.py drug_id --drug aluminium`. This will select the default ontology and attempt to load it in. The line `Loading Ontology: <default/path/owl>` indicates the selected ontoloy and that the program has begun loading it in. When successfully loaded it will output `Dinto Loaded` and continue with the input command.

By using the `-p` flag we can specify badly formatted ontologies to thoroughly test the system. The two cases to test for are when the owl ontology does not exist and when the ontology is incorrectly formatted. A test for a non existant ontology is `python dinto.py drug_id --drug aluminium -p this/should/not/work`, the program should output an error of `FileNotFoundError: [Errno 2] No such file or directory: '</path/to/repo>/this'` following it's initial attempt to load it in. This is the correct output and means the program will never insecurely attempt to work with a non existant ontology. 

A badly formatted version of DINTO is included in the repository to test against in `DINTO/DINTO 1/BROKEN_DINTO.owl`. To test this, run the command `python dinto.py drug_id --drug aluminium -p ./DINTO/DINTO\ 1/BROKEN_DINTO.owl`. The feature will attempt to load in this badly formatted ontology and will output `Error Loading Dinto.` followed by `Dinto not loaded.` which successfully shows the feature safely handles erroneous ontologies.

A unit test exists for this feature, the command `python -m unittest test.dinto-tests.TestDinto.test_load_ontology` will execute it and will output exactly what it's testing and the results of the test.

###Identify drugs in PML (20)
A given pml file must have drug names in script tags. This is the structure that is expected by PYOS to extract drugs. We have created a strict but understandable language for representing drugs in PML, `Every $time_duration take $drug_name for $time_duration`. For now, this structure must be striclty followed.  A time duration can be of the structure 2 days, 48 hours, etc. Upon entering this option, pyos will extract all values within script tags, parse out the data and relay it to the screen. It is important that only relevant information is placed within these tags, i.e Drug Names for querying DDIs.

######Testing
Run `python pyos.py`. Select `peos/build_test.pml` as the test file. Select Option *4*. Upon success, the user should be presented with a dictionary of data.
```
[
 {
  'every': 
  {
   'unitOfTime': 'day', 
   'duration': 2
  }, 
  'take': 'paracetamol',
  'for': {
          'unitOfTime': 'year',
          'duration': 29
         }
 },
 {
  'every': 
  {
   'unitOfTime': 'day',
   'duration': 3
  }, 
  'take': 'antibiotics',
  'for': 
  {
   'unitOfTime': 'week',
   'duration': 2
  }
 }
]
```

###Identify drugs in DINTO (20)

This feature allows the program to find all relevant details of a drug in the DINTO ontology by simply providing the drug name. The relevant details include the ids and definitions for the input drug. This feature can return multiple results for a single drug as the structure of DINTO has many duplicate entries.

A drug can be manually found by using the command `python dinto.py drug_id --drug "<drugname>"`.

######Testing

To test this feature simply run `python dinto.py drug_id --drug "aluminium"`. After it has loaded in the ontology the program will output `Querying drug name: aluminium` which indicates it's attempt to identify the drug in DINTO. When a result is found it will output `Successful query for drug: aluminium` and then output the relevant formatted data in JSON format:

```
{
  "drug_name": "aluminium", 
  "drug_ids": ["http://purl.obolibrary.org/obo/DINTO_DB01370"], 
  "drug_definitions": ["A metallic element that has the atomic number 13, atomic symbol Al, and atomic weight 26.98. [PubChem]"]
}
```

When the feature returns this information it completes successfully. The other alternative is when a drugname is entered that does not exist in DINTO. An example to test this is `python dinto.py drug_id -d nothing`. After attempting to query the DINTO file for this drug, it returns `No results for the drug: nothing` and a JSON with the data: 

```
{
  "drug_name": "nothing", 
  "drug_ids": [], 
  "drug_definitions": []
}
```

Which shows the test works as intended.

If a drugname is ommitted, the feature will warn the user with `dinto.py drug_id: error: argument -d/--drug is required`.

A unit test exists for this feature, the command `python -m unittest test.dinto-tests.TestDinto.test_drug_id_query` will execute it and will output exactly what it's testing and the results of the test.

###Identify DDIs (30)

This feature allows the program to find all Drug Drug Interactions in a given list of drugs. The relevant details include the id, name, definitionst and type of each detected DDI alongside the details of the drugs involved.

A DDI between drugs can be found by using the command `python dinto.py ddis --drugs "<drugname>" "<drugname>" ...` with no limit on the number of drug names that can be input.

######Testing

To test this feature, run `python dinto.py ddis --drugs "aluminium" "dapsone"`. After it has loaded in the ontology the program will query each of the input drugs, in this example `aluminium` and `dapsone`. After each drug has been queried the feature will then query for each possible interaction between all the drugs passed in. The feature will output `aluminium and dapsone comparison` when it begins querying for a DDI and `DDI Found for aluminium / dapsone` when it finds one otherwise it will not output anything. After querying all DDIs it will output it's results, for this example you should expect:

```
[
 {
  'DDI': 'aluminium/dapsone DDI', 
  'DDI_id': 'http://purl.obolibrary.org/obo/DINTO_05284', 
  'DDI_type_name': 'non-absorbable complex formation', 
  'DDI_type_description': 'A non-absorbable complex formation is a pharmacokinetic DDI mechanism which occurs when drug B binds to drug A leading to the formation of a higher molecular weight complex.'
  'DDI_type': 'http://purl.obolibrary.org/obo/DINTO_000422',  
  'first_drug_id': 'http://purl.obolibrary.org/obo/DINTO_DB01370', 
  'first_drug': {
   'definitions': ['A metallic element that has the atomic number 13, atomic symbol Al, and atomic weight 26.98. [PubChem]'],
   'name': 'aluminium', 
   'ids': ['http://purl.obolibrary.org/obo/DINTO_DB01370']
  },
  'second_drug_id': 'http://purl.obolibrary.org/obo/CHEBI_4325',
  'second_drug': {
   'definitions': ['Diphenylsulfone in which the hydrogen atom at the 4 position of each of the phenyl groups is substituted by an amino group. It is active against a wide  range of bacteria, but is mainly employed for its actions against Mycobacterium leprae, being used as part of multidrug regimens in the treatment of all forms of leprosy.'], 
   'name': 'dapsone', 
   'ids': ['http://purl.obolibrary.org/obo/CHEBI_4325']
  }
 }
]
```

Which includes the DDI name `non-absorbable complex formation` and it's description `A non-absorbable complex formation is a pharmacokinetic DDI mechanism which occurs when drug B binds to drug A leading to the formation of a higher molecular weight complex.` which will indicate to the user whether it is a positive or negative DDI.

A list of drugs with no DDIs will return an empty array `[]` as it's result. Similarly, entering only one drug will return an empty list as no DDIs will be found. The examples `python dinto.py ddis --drugs "aluminium"` and `python dinto.py ddis --drugs "aluminium" "caffeine"` both show queries with no DDI results.

A unit test exists for this feature, the command `python -m unittest test.dinto-tests.TestDinto.test_ddi_query` will execute it and will output exactly what it's testing and the results of the test.

###On-Screen DINTO Reporting (7)

This feature ensures every action throughout the entire program involving selecting, loading and querying the DINTO ontology is reported on-screen to the user. As such this feature reports what action it is about to do e.g. `Loading Dinto` and the outcome of that action e.g. `Dinto Loaded`.

######Testing

Running the `dinto.py` as outlined in any of the other DINTO related features and observe the terminal output to see the on-screen DINTO Reporting.

###DINTO Logfile Generation (6)

This feature logs every action throughout the entire program involving selecting, loading and querying the DINTO ontology. It ensures that everything logged is in a consistent manner.

The logs are written to `the-pain-train-group-3/logging/main.log` where all logged actions can be found. An example of the content of the log file could be: 

```
2017-03-10 11:41:01,602 Loading Dinto from file:///Users/calvinnolan/Documents/Websites/the-pain-train-group-3/DINTO/DINTO%201/DINTO_1_WITH_INFERRED.owl
2017-03-10 11:42:24,302 Dinto loaded
2017-03-10 11:42:25,515 Successfull query for drug: aluminium
2017-03-10 11:42:25,526 Successfull query for drug: dapsone
```

The logging maintains consistency in it's logs with each row representing a `DATE TIME DESCRIPTION` format.

######Testing

To test this feature, run and command for another feature and then check the contents of `main.log` to see that it represents the actions performed. An example of this is running the `python dinto.py drug_id --drug "aluminium"` command. If it executes successfully, running `tail logging/main.log` from the root folder of the repository should output the most recent content of `main.log` to the console and the last lines will be:

```
2017-03-10 17:27:37,087 Converting file path, <your/user/path>/the-pain-train-group-3/DINTO/DINTO 1/DINTO_1_WITH_INFERRED.owl, to uri
2017-03-10 17:27:37,087 Loading Dinto from <your/user/path>/the-pain-train-group-3/DINTO/DINTO%201/DINTO_1_WITH_INFERRED.owl
2017-03-10 17:28:58,501 Dinto loaded
2017-03-10 17:28:59,804 Successfull query for drug: aluminium
```

Which are the logged actions from the drug query command previously executed. Seeing these actions logged in `main.log` means the logging is working successfully and will continue to log all DINTO related actions throughout the program.

A unit test exists for this feature, the command `python -m unittest test.dinto-tests.TestDinto.test_logging` will execute it and will output exactly what it's testing and the results of the test.

###DINTO Error and Warning highlights (10)

This feature ensures every action throughout the entire program involving selecting, loading and querying the DINTO ontology is has correct error and warning handling and highlights these to the user. Every DINTO related action accounts for error cases which can be seen in the tests explained for each DINTO related feature.

All actions and warnings are also logged in `logging/main.log` for inspection at any time.

######Testing

One example of error handling from this feature is to run the command `python dinto.py drug_id --drug` i.e. attempt to query a drug without passing in a drug name. The feature will respond with `dinto.py drug_id: error: argument -d/--drug: expected one argument` which tells the user the exact problem and how to fix it. Error and warning handling like this exists for all areas of user input and querying of the DINTO ontology.

###Adding Time to PML (40)
See Identifying drugs in PML as time is interwoven with Drugs and they both must exist in a valid PML file.
