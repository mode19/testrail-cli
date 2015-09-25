# TestRail CLI

A simple command line tool to report test runs to TestRail from a build server.


## Installation and usage

To install testrail-cli, ensure you have node installed, then run
`npm install -g testrail-cli`

To run commands, change directories to the root of your working project (where
you will likely have a `.testrail-cli.yml` file--see details below), and run a
command like this:

```sh
testrail-cli init --runName="Name of my test run"
```

Full configuration and command details below.


## Configuration

This CLI provides configuration in two places: environment variables (for all
configs related to TestRail authentication), and a `.testrail-cli.yml` file,
assumed to be in the working directory from which the CLI is run. Details for
each below.

#### Environment Variables

You must provide TestRail authentication information via environment variables.
It's assumed that you are storing these securely.

__`TESTRAIL_URL`__ - The URL of your TestRail instance.
```sh
export TESTRAIL_URL=https://[yourinstance].testrail.net
```

__`TESTRAIL_UN`__ - The username/e-mail address to authenticate with
```sh
export TESTRAIL_UN=user@example.com
```

__`TESTRAIL_PW`__ - The password associated with the aforementioned username.
```sh
export TESTRAIL_PW=yourPasswordhere
```

Note that the API must be enabled for your TestRail instance.

#### `.testrail-cli.yml`

It's recommended that you also supply additional configurations in a dot-file
at the root of your project. This file (`.testrail-cli.yml`) must contain a map
of test names to TestRail case IDs, but can also be used to provide default
values for other useful properties. An example is provided below:

```yaml
# The ID of the default project against which to run commands.
projectId: 2

# The ID of the default test suite used for test run reporting.
suiteId: 2

# A mapping that relates testcase names to TestRail case IDs. The testcase name
# (on the left) represents the "name" attribute on the "testcase" element in
# your xUnit XML files.
caseNameToIdMap:
  "Name of Test Case 7": 7
  "Another Test Name": 8
  "Not to be confused with the class name": 14

# If you have complex test suites where a case name may be identical across
# multiple classes, an alternative mapping can be provided like so:
caseClassAndNameToIdMap:
  "Name of xUnit Class 1":
    "Name of TestCase 7": 7
    "Another Test Name": 8
  "Name of another xUnit class":
    "Another Test Name": 88
    "className::testName14": 14
```


## Commands

A list of currently supported commands and their arguments/flags. Note that
every command can take a `--debug` flag, which outputs verbose information to
stderr.

#### `testrail-cli init`

Used to initialize a test run in a given project. On success, the command will
return a runId.

__Arguments__
- `--projectId` or `-p`
  - Required if no ID is provided in `.testrail-cli.yml`. The value should
    be the ID of the project for which you are beginning a test run.
- `--runName` or `-n`
  - Required. A name representing this test run.
- `--suiteId` or `-s`
  - Optional. The ID of the test suite to use when initializing the run. If
    none is supplied, then a suite containing all cases is used. This can
    also be supplied via `.testrail-cli.yml`.
- `--description` or `-d`
  - Optional. If desired, you can pass a description of this test run.
- `--milestoneId` or `-m`
  - Optional. If desired, you can pass the ID of a milestone to associate
    with this test run.

__Examples__

Start a test run called "Automated Run" against project with ID 5.
```sh
testrail-cli init --projectId=5 --runName="Automated Run"
```

Start a test run called "Another Run" where the projectId is assumed from a
`.testrail-cli.yml` file. Stash the runId for later commands.
```sh
export TESTRAIL_RUNID=$(testrail-cli init -n "Another Run")
```

#### `testrail-cli report`

Used to report test results for a given test run. On success, the command will
report the number of case runs successfully reported.

Note that this tool makes the assumption that you have created a map in your
`.testrail-cli.yml` file that maps test case names (as reported in your xUnit
XML) to test case IDs in TestRail. An example is highlighted in the
"configuration" section above.

__Arguments__
- `--runId` or `-r`
  - Required. The ID of the test run for which you wish to report test results.
- `--file` or `-f`
  - Required. The path to a xUnit XML file (or a path to a directory containing
    such files).

__Examples__

Report a test run for a single XML file and known runId.
```sh
testrail-cli report --runId=5 --file=logs/junit.xml
```

Report a test run for a directory of xUnit XML files using a runId pulled from
environment variables.

testrail-cli report -r $TESTRAIL_RUNID -f /path/to/xml/

#### `testrail-cli finish`

Used to close a given test run.

__Arguments__
- `--runId` or `-r`
  - Required. The ID of the test run you wish to close.

__Examples__

Close a test run based on an ID in an environment variable.
```sh
testrail-cli finish --runId=$TESTRAIL_RUNID
```
