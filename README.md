# CICD

## Github Actions

- GitHub provides Linux, Windows, and macOS virtual machines to run your workflows, or you can host your own self-hosted runners in your own data center or cloud infrastructure.

- Workflow triggeres when an event occurs
    - Workflow contains one or more jobs
        -  Job has one or more steps
            - Step will run a script or run an action

- Jobs present in a workflow can run in sequential order or in parallel. (TODO :  how to run Jobs in parallel.)

- Workflows are defined by a YAML file

- Workflows can be triggered manually, or at a defined schedule. (TODO : how to run jobs in a schedule.)

- You can reference a workflow within another workflow. (TODO : "Reusing workflows.")

## Events

Major events that we have are -

#### Trigger on push  
```
on: [push]
```
- Triggered every time someone pushes a change to the repository or merges a pull request.
- This is triggered by a push to every branch.


#### Tigger on pull request creation
```
on: [pull_request]
```
- Trigger on every pull request creation.


#### Trigger on pull request creation on specific target branch
```
on:
  pull_request:
    branches:    
      - main
      - 'mona/octocat'
      - 'releases/**'
```
pull_request event for a pull request targeting:
- A branch named main (refs/heads/main)
- A branch named mona/octocat (refs/heads/mona/octocat)
- A branch whose name starts with releases/, like releases/10 (refs/heads/releases/10)

#### Trigger on pull request creation on excluding some branch
```
on:
  pull_request:
    branches-ignore:    
      - 'mona/octocat'
      - 'releases/**-alpha'
```
pull_request event for a pull request excluding some branches.

#### Trigger on pull request creation on including and excluding some branches
```
on:
  pull_request:
    branches:    
      - 'releases/**'
      - '!releases/**-alpha'
```
We cannot have `branches` and `branches-ignore` to filter the same event. So we need to use `branches` with `!` character to indicate which branches should be excluded.

#### Trigger on push on branches and tags
```
on:
  push:
    branches:    
      - main
      - 'releases/**'
    tags:        
      - v2
      - v1.*
```

#### Trigger on push with Including and excluding tags.
```
on:
  push:
    tags:        
      - v2.**
      - !v2.2
```

#### Trigger on File paths are changed.
```
on:
  push:
    paths:
      - '**.js'
```
- When using the `push` and `pull_request` events, you can configure a workflow to run based on what file paths are changed. Path filters are not evaluated for pushes of tags.
- Use the `paths-ignore` filter when you only want to exclude file path patterns.

#### Trigger on filepath changes, include and exclude files
```
on:
  push:
    paths:
      - 'sub-project/**'
      - '!sub-project/docs/**'
```

#### Trigger on filepath changes, include and exclude files
```
on:
  push:
    paths:
      - 'sub-project/**'
      - '!sub-project/docs/**'
```


## Jobs
- A workflow run is made up of one or more `jobs`, which run in parallel by default.
- To run jobs sequentially, you can define dependencies on other jobs using the `jobs.<job_id>.needs` keyword.
- Each job runs in a runner environment specified by `runs-on`.    

#### Setting an ID for a job
```
jobs:
  my_first_job:
    name: My first job
  my_second_job:
    name: My second job
```
- In this example, two jobs have been created, and their `job_id` values are `my_first_job` and `my_second_job`.
- Use `jobs.<job_id>.name` to set a name for the job, which is displayed in the GitHub UI.    

#### running job sequentially
```
jobs:
  job1:
  job2:
    needs: job1
  job3:
    needs: [job1, job2]
```
- Use `jobs.<job_id>.needs` to identify any jobs that must complete successfully before this job will run. It can be a string or array of strings.

#### Not requiring successful dependent jobs
```
jobs:
  job1:
  job2:
    needs: job1
  job3:
    if: ${{ always() }}
    needs: [job1, job2]
```
- In this example, `job3` uses the `always()` conditional expression so that it always runs after `job1` and `job2` have completed, regardless of whether they were successful.

#### Choosing the runner for a job
```
jobs:
  job1:
    runs-on: macos-latest
```
- Use `jobs.<job_id>.runs-on` to define the type of machine to run the job on
- for mac we have following options
    - macos-12                      ->   macOS Monterey 12
    - macos-latest or macos-11      ->   macOS Big Sur 11 ( The macos-latest label currently uses the macOS 11 runner image. )
    - macos-10.15                   ->   macOS Catalina 10.15

#### Choosing self-hosted runners
- All self-hosted runners have the `self-hosted` label. Using only this label will select any `self-hosted` runner. To select runners that meet certain criteria, such as operating system or architecture, we recommend providing an array of labels that begins with `self-hosted` (this must be listed first) and then includes additional labels as needed. When you specify an array of labels, jobs will be queued on runners that have all the labels that you specify.
```
runs-on: [self-hosted, macos]
```

#### Using conditions to control job execution
- You can use the `jobs.<job_id>.if` conditional to prevent a job from running unless a condition is met.
```
name: example-workflow
on: [push]
jobs:
  production-deploy:
    if: github.repository == 'octo-org/octo-repo-prod'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '14'
      - run: npm install -g bats
```
- The above example will only run if the repository is named octo-repo-prod and is within the octo-org organization.
- If the condition is false then the job will be marked as skipped.

### Using a matrix for your jobs
```
jobs:
  example_matrix:
    strategy:
      matrix:
        version: [10, 12, 14]
        os: [ubuntu-latest, windows-latest]
```
- A matrix strategy lets you use variables in a single job definition to automatically create multiple job runs that are based the combinations of the variables. 
- Use `jobs.<job_id>.strategy.matrix` to define a matrix of different job configurations.
- In the above example - A job will run for each possible combination of the variables. In this example, the workflow will run six jobs, one for each combination of the os and version variables.
- A matrix will generate a maximum of 256 jobs per workflow run.

#### Using a single-dimension matrix
```
jobs:
  example_matrix:
    strategy:
      matrix:
        version: [10, 12, 14]
    steps:
      - uses: actions/setup-node@v3
        with:
          node-version: ${{ matrix.version }}
```
- The above workflow defines the variable version with the values [10, 12, 14]. The workflow will run three jobs, one for each value in the variable.
- Each job will access the version value through the matrix.version context and pass the value as node-version to the actions/setup-node action.

#### Using contexts to create matrices

- You can use contexts to create matrices.
- For example, the following workflow triggers on the repository_dispatch event and uses information from the event payload to build the matrix. 
- When a repository dispatch event is created with a payload like the one below, the matrix version variable will have a value of [12, 14, 16].
```
{
  "event_type": "test",
  "client_payload": {
    "versions": [12, 14, 16]
  }
}
```
```
on:
  repository_dispatch:
    types:
      - test
 
jobs:
  example_matrix:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        version: ${{ github.event.client_payload.versions }}
    steps:
      - uses: actions/setup-node@v3
        with:
          node-version: ${{ matrix.version }}
```

#### Expanding or adding matrix configurations
- Use `jobs.<job_id>.strategy.matrix.include` to expand existing matrix configurations or to add new configurations. The value of include is a list of objects.
- For example, this matrix:
```
strategy:
  matrix:
    fruit: [apple, pear]
    animal: [cat, dog]
    include:
      - color: green
      - color: pink
        animal: cat
      - fruit: apple
        shape: circle
      - fruit: banana
      - fruit: banana
        animal: cat
```
- will result in six jobs with the following matrix combinations:
* {fruit: apple, animal: cat, color: pink, shape: circle}
* {fruit: apple, animal: dog, color: green, shape: circle}
* {fruit: pear, animal: cat, color: pink}
* {fruit: pear, animal: dog, color: green}
* {fruit: banana}
* {fruit: banana, animal: cat}
* following this logic:

- {color: green} is added to all of the original matrix combinations because it can be added without overwriting any part of the original combinations.
- {color: pink, animal: cat} adds color:pink only to the original matrix combinations that include animal: cat. This overwrites the color: green that was added by the previous include entry.
- {fruit: apple, shape: circle} adds shape: circle only to the original matrix combinations that include fruit: apple.
- {fruit: banana} cannot be added to any original matrix combination without overwriting a value, so it is added as an additional matrix combination.
- {fruit: banana, animal: cat} cannot be added to any original matrix combination without overwriting a value, so it is added as an additional matrix combination. It does not add to the {fruit: banana} matrix combination because that combination was not one of the original matrix combinations.

#### Expanding configurations
- The following workflow will run six jobs, one for each combination of os and node. When the job for the os value of windows-latest and node value of 16 runs, an additional variable called npm with the value of 6 will be included in the job.
```
jobs:
  example_matrix:
    strategy:
      matrix:
        os: [windows-latest, ubuntu-latest]
        node: [12, 14, 16]
        include:
          - os: windows-latest
            node: 16
            npm: 6
    runs-on: ${{ matrix.os }}
    steps:
      - uses: actions/setup-node@v3
        with:
          node-version: ${{ matrix.node }}
      - if: ${{ matrix.npm }}
        run: npm install -g npm@${{ matrix.npm }}
      - run: npm --version
```

#### Adding configurations
- This matrix will run 10 jobs, one for each combination of os and version in the matrix, plus a job for the os value of windows-latest and version value of 17.
```
jobs:
  example_matrix:
    strategy:
      matrix:
        os: [macos-latest, windows-latest, ubuntu-latest]
        version: [12, 14, 16]
        include:
          - os: windows-latest
            version: 17
```

- If you don't specify any matrix variables, all configurations under include will run. For example, the following workflow would run two jobs, one for each include entry. This lets you take advantage of the matrix strategy without having a fully populated matrix.
```
jobs:
  includes_only:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        include:
          - site: "production"
            datacenter: "site-a"
          - site: "staging"
            datacenter: "site-b"
```

#### Excluding matrix configurations
- To remove specific configurations defined in the matrix, use jobs.<job_id>.strategy.matrix.exclude. 
- An excluded configuration only has to be a partial match for it to be excluded. 
- For example, the following workflow will run nine jobs: one job for each of the 12 configurations, minus the one excluded job that matches {os: macos-latest, version: 12, environment: production}, and the two excluded jobs that match {os: windows-latest, version: 16}.
```
strategy:
  matrix:
    os: [macos-latest, windows-latest]
    version: [12, 14, 16]
    environment: [staging, production]
    exclude:
      - os: macos-latest
        version: 12
        environment: production
      - os: windows-latest
        version: 16
runs-on: ${{ matrix.os }}
```

#### Handling failures
- You can control how job failures are handled with `jobs.<job_id>.strategy.fail-fast` and `jobs.<job_id>.continue-on-error`.
- `fail-fast` applies to the entire matrix. If `jobs.<job_id>.strategy.fail-fast` is set to true, GitHub will cancel all in-progress and queued jobs in the matrix if any job in the matrix fails. This property defaults to true.
- `jobs.<job_id>.continue-on-error` applies to a single job. If `jobs.<job_id>.continue-on-error` is true, other jobs in the matrix will continue running even if the job with jobs.<job_id>.continue-on-error: true fails.    
- You can use `jobs.<job_id>.strategy.fail-fast` and `jobs.<job_id>.continue-on-error` together. 
    - For example, the following workflow will start four jobs. For each job, `continue-on-error` is determined by the value of `matrix.experimental`.
```
jobs:
  test:
    runs-on: ubuntu-latest
    continue-on-error: ${{ matrix.experimental }}
    strategy:
      fail-fast: true
      matrix:
        version: [6, 7, 8]
        experimental: [false]
        include:
          - version: 9
            experimental: true
```

#### Defining the maximum number of concurrent jobs
- To set the maximum number of jobs that can run simultaneously when using a matrix job strategy, use jobs.<job_id>.strategy.max-parallel.
```
jobs:
  example_matrix:
    strategy:
      max-parallel: 2
      matrix:
        version: [10, 12, 14]
        os: [ubuntu-latest, windows-latest]
```


## iOS app repo usage

Following are the main usage of the CICD pipeline that is used for iOS application.
- For every push check if test cases are passed.
- For every push check if build is successfull.
- For every push on master check if build is successfull and test cases are passing.
- For every PR raised on master check if build is successfull and test cases are passing. Add this as comment in PR.
- For every PR raised on master if the launch time of app is effected. Add this as comment in PR.


### For every push check if test cases are passed.

```
name: For every push check if build is successfull.

on:
  push
jobs:
  build:
    name: Build default scheme using any available iPhone simulator
    runs-on: macos-latest
    
    steps:
      - name: Checkout
        # Check https://github.com/actions/checkout#usage for complete info on checkout workflow.
        uses: actions/checkout@v3
      - name: Set Default Scheme
          # Lets discuss the commands below
          # For one of my project running - `xcodebuild -list -json | tr -d "\n"` on terminal will return
          # {
          #   "project": {
          #     "configurations": [
          #       "Debug",
          #       "Release"
          #     ],
          #     "name": "Wordle",
          #     "schemes": [
          #       "Wordle"
          #     ],
          #     "targets": [
          #       "Wordle",
          #       "WordleTests",
          #       "WordleUITests"
          #     ]
          #   }
          # }
          #
          # And the command - `default=$(echo $scheme_list | ruby -e "require 'json'; puts JSON.parse(STDIN.gets)['project']['targets'][0]")` will set `default` to value - `Wordle`
          # command - `echo $default | cat >default` will create a file named default and store the text `Wordle` in it.
          # command - `echo Using default scheme: $default` will just print out - Using default scheme: Wordle
        run: |
          scheme_list=$(xcodebuild -list -json | tr -d "\n")
          default=$(echo $scheme_list | ruby -e "require 'json'; puts JSON.parse(STDIN.gets)['project']['targets'][0]")
          echo $default | cat >default
          echo Using default scheme: $default
      
      - name: Build
        # The below command will set the Environment variables 
        # The `scheme: ${{ 'default' }}` will set the string "default" to scheme
        # The `platform: ${{ 'iOS Simulator' }}` will set the string "iOS Simulator" to platform
        env:
          scheme: ${{ 'default' }}
          platform: ${{ 'iOS Simulator' }}
          #
          # Note: xcrun xctrace returns via stderr, not the expected stdout (see https://developer.apple.com/forums/thread/663959)
          # `xcrun xctrace list devices` will return :
          # 
          # == Devices ==
          # Syed’s Mac mini (9E6F2318-7726-5F7F-AD17-E1B86559533F)
          #
          # == Simulators ==
          # Apple TV Simulator (15.0) (800CC045-E64E-49CD-9612-04E56DF0E2AC)
          # Apple TV 4K (2nd generation) Simulator (15.0) (D0A3A614-BD32-40D0-A5A2-B2AA5F9ABD00)
          # iPad (9th generation) Simulator (15.0) (BD1BA1E1-0D1A-4289-ADB3-DF8652EAED34)
          # iPad Air (4th generation) Simulator (15.0) (06850C91-9436-4ACC-B055-267BAB1E5E0A)
          # iPad Pro (11-inch) (3rd generation) Simulator (15.0) (CF449DED-556F-4E07-9826-2BD0D57F6BF6)
          # iPad mini (6th generation) Simulator (15.0) (6F786809-27D2-44E8-B8D1-60F6D96EF0CB)
          # iPhone 11 Simulator (15.0) (6DEE4205-9FA0-4EF8-AE24-335618AF367B)
          # iPhone 11 Pro Simulator (15.0) (D316CBA0-9762-4119-83E8-F97F7FC1B1FE)
          # iPhone 11 Pro Max Simulator (15.0) (9E9B873C-4678-4919-9D31-6C73E3DC9A41)
          # iPhone 12 Simulator (15.0) (B98ED742-88DB-4F56-9BB9-CC79CABECCB8)
          # iPhone 12 Pro Simulator (15.0) (BAFCF138-11B9-4644-8217-02FB9CC1FFF3)
          # iPhone 13 Simulator (15.0) (C2C70772-8820-43D1-9ACC-DEB016AA1758)
          # iPhone 13 Simulator (15.0) + Apple Watch Series 7 - 45mm (8.0) (DCDB702F-F585-47ED-A444-F053474F50C3)
          # iPhone 8 Simulator (15.0) (2BC39A26-3F3E-44B4-8A1F-081E8FA9BF9B)
          # iPhone 8 Plus Simulator (15.0) (3976CC62-4565-48CB-B0E1-67B0D449F593)
          # iPhone SE (2nd generation) Simulator (15.0) (A731E958-1AC1-4F2C-AE92-F7CFFAF92657)
          # iPod touch (7th generation) Simulator (15.0) (28E189CB-76CF-43A9-88CC-665CD4CBD3D7)
          # 
          # The command - `xcrun xctrace list devices 2>&1` will convert stderr to stdout (see last comment of https://developer.apple.com/forums/thread/663959)
          # 
          # The command - `xcrun xctrace list devices 2>&1 | grep -oE 'iPhone.*?[^\(]+'` will return :
          # 
          # iPhone 11 Simulator
          # iPhone 11 Pro Simulator
          # iPhone 11 Pro Max Simulator
          # iPhone 12 Simulator
          # iPhone 12 Pro Simulator
          # iPhone 13 Simulator
          # iPhone 13 Simulator
          # iPhone 8 Simulator
          # iPhone 8 Plus Simulator
          # iPhone SE 
          #
          # `head -1` will select the first line : iPhone 11 Simulator
          # `awk '$1=$1'` will remove extra spaces
          # `awk '{$1=$1;print}' will print the iPhone 11 Simulator without any extra spaces.
          # 
          # `sed -e "s/ Simulator$//"` will return `iPhone 11`
          # so at the end `$device` will assigned the value - iPhone 11
          # 
          # `if [ $scheme = default ]; then scheme=$(cat default); fi` will check that if `$scheme = default` which is true as we have just assigned it in environment variables. then it will set the scheme to contet of `default` file, which is "Wordle"
          # `ls -A` will list all files at repo root :     Wordle            Wordle.xcodeproj    WordleTests        WordleUITests        default
          # `"`ls -A | grep -i \\.xcworkspace\$`"` will check if there is a workspace file.
          # `then filetype_parameter="workspace" && file_to_build="`ls -A | grep -i \\.xcworkspace\$`";` will set `filetype_parameter` variable to "workspace" and `file_to_build` to workspace file.
          # `else filetype_parameter="project" && file_to_build="`ls -A | grep -i \\.xcodeproj\$`";` will set `filetype_parameter` variable to "project" and `file_to_build` to xcode-project file.
          # `file_to_build=`echo $file_to_build | awk '{$1=$1;print}'` will just print the file to build.
          # `xcodebuild build-for-testing -scheme "$scheme" -"$filetype_parameter" "$file_to_build" -destination "platform=$platform,name=$device"` will build the project
          #
          
        run: |
          device=`xcrun xctrace list devices 2>&1 | grep -oE 'iPhone.*?[^\(]+' | head -1 | awk '{$1=$1;print}' | sed -e "s/ Simulator$//"`
          if [ $scheme = default ]; then scheme=$(cat default); fi
          if [ "`ls -A | grep -i \\.xcworkspace\$`" ]; then filetype_parameter="workspace" && file_to_build="`ls -A | grep -i \\.xcworkspace\$`"; else filetype_parameter="project" && file_to_build="`ls -A | grep -i \\.xcodeproj\$`"; fi
          file_to_build=`echo $file_to_build | awk '{$1=$1;print}'`
          xcodebuild build-for-testing -scheme "$scheme" -"$filetype_parameter" "$file_to_build" -destination "platform=$platform,name=$device"
          
      - name: Test
        env:
          scheme: ${{ 'default' }}
          platform: ${{ 'iOS Simulator' }}
          # The below commands are same as the above.
          # `xcodebuild test-without-building -scheme "$scheme" -"$filetype_parameter" "$file_to_build" -destination "platform=$platform,name=$device"` will just test the project

        run: |
          device=`xcrun xctrace list devices 2>&1 | grep -oE 'iPhone.*?[^\(]+' | head -1 | awk '{$1=$1;print}' | sed -e "s/ Simulator$//"`
          if [ $scheme = default ]; then scheme=$(cat default); fi
          if [ "`ls -A | grep -i \\.xcworkspace\$`" ]; then filetype_parameter="workspace" && file_to_build="`ls -A | grep -i \\.xcworkspace\$`"; else filetype_parameter="project" && file_to_build="`ls -A | grep -i \\.xcodeproj\$`"; fi
          file_to_build=`echo $file_to_build | awk '{$1=$1;print}'`
          xcodebuild test-without-building -scheme "$scheme" -"$filetype_parameter" "$file_to_build" -destination "platform=$platform,name=$device"
          
```

### For every push check if build is successfull.

```
name: build the project

on:
  push
jobs:
  build:
    name: Build default scheme using any available iPhone simulator
    runs-on: macos-latest
    
    steps:
      - name: Checkout
        uses: actions/checkout@v3
        
      - name: Set Default Scheme
        run: |
          scheme_list=$(xcodebuild -list -json | tr -d "\n")
          default=$(echo $scheme_list | ruby -e "require 'json'; puts JSON.parse(STDIN.gets)['project']['targets'][0]")
          echo $default | cat >default
          echo Using default scheme: $default
      
      - name: Build
        env:
          scheme: ${{ 'default' }}
          platform: ${{ 'iOS Simulator' }}
          
        run: |
          device=`xcrun xctrace list devices 2>&1 | grep -oE 'iPhone.*?[^\(]+' | head -1 | awk '{$1=$1;print}' | sed -e "s/ Simulator$//"`
          if [ $scheme = default ]; then scheme=$(cat default); fi
          if [ "`ls -A | grep -i \\.xcworkspace\$`" ]; then filetype_parameter="workspace" && file_to_build="`ls -A | grep -i \\.xcworkspace\$`"; else filetype_parameter="project" && file_to_build="`ls -A | grep -i \\.xcodeproj\$`"; fi
          file_to_build=`echo $file_to_build | awk '{$1=$1;print}'`
          xcodebuild -scheme "$scheme" -"$filetype_parameter" "$file_to_build" -destination "platform=$platform,name=$device" build
          
```

### For every push on master check if build is successfull and test cases are passing.

- This will be same as - "For every push check if test cases are passed."
- Note: sometime the normal iOS project's default scheme have the testing targets as unit test cases and UI testcases both. To run only unit testcases in CI pipelines we need to create a different scheme that has the test target as UITestCases. Also, we need to remove the UITest target from the default scheme.

### For every PR raised on master check if build is successfull and test cases are passing. Add this as comment in PR.






