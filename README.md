# cdp

````md
# 🔐 TruffleHog Installation & Options

## 📌 About
[TruffleHog](https://github.com/trufflesecurity/trufflehog) is a tool that searches through Git repositories for secrets — even deep inside commit history and across branches.  
It helps developers and security teams detect accidentally committed secrets (API keys, tokens, passwords, etc.).

---

## ⚙️ Installation (Linux)

```bash
wget https://github.com/trufflesecurity/trufflehog/releases/download/v3.79.0/trufflehog_3.79.0_linux_amd64.tar.gz
tar -xvf trufflehog_3.79.0_linux_amd64.tar.gz
chmod +x trufflehog
mv trufflehog /usr/local/bin/
````

---

## 🛠️ Usage Help

```bash
trufflehog --help
```

### Key Flags

* `-j, --json` → Output in JSON format
* `--only-verified` → Show only verified secrets
* `--filter-entropy=3.0` → Reduce noise using entropy filtering
* `--concurrency=4` → Set number of concurrent workers
* `--fail` → Exit with code 183 if results are found (useful for CI/CD pipelines)
* `--include-detectors="all"` / `--exclude-detectors="..."` → Fine-tune detection types

---

## 📂 Available Commands

* **Git-based Sources**

  * `git <uri>` → Scan local or remote Git repos
  * `github` → Scan GitHub repos
  * `gitlab --token=TOKEN` → Scan GitLab repos

* **File & Storage**

  * `filesystem [path]` → Scan local filesystem
  * `s3` → Scan AWS S3 buckets
  * `gcs` → Scan Google Cloud Storage buckets
  * `docker --image=IMAGE` → Scan Docker images

* **CI/CD & Platforms**

  * `circleci --token=TOKEN` → Scan CircleCI
  * `travisci --token=TOKEN` → Scan TravisCI
  * `jenkins --url=URL` → Scan Jenkins
  * `postman` → Scan Postman collections

* **Other**

  * `syslog` → Scan system logs
  * `elasticsearch` → Scan Elasticsearch data

---

✅ Next step: Would you like me to show you **how to run TruffleHog against your local repo** (e.g., `trufflehog git file://.`) or set up a **CI/CD scan pipeline**?

````md
# 🔍 Running TruffleHog Scanner & Analyzing Secrets

## 🚀 Direct Repository Scan (HTTP)
TruffleHog v3 can scan repositories directly without needing to clone them.  
Example scan against **django-nv project**:

```bash
trufflehog git http://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/django-nv.git --json
````

### Key Option

* `--json` → outputs results in machine-readable **JSON** format

### Sample Findings

* **AWS Access Key** (✅ Verified)
* **Slack Token** (❌ Unverified)

---

## 🔑 SSH-based Scan

To authenticate securely, configure **SSH agent** and run scan:

```bash
# Start ssh-agent
eval "$(ssh-agent -s)"

# Secure private key
chmod 600 ~/.ssh/id_rsa

# Add key to ssh-agent
ssh-add ~/.ssh/id_rsa
```

### Configure SSH in `~/.ssh/config`

```bash
cat << EOF > ~/.ssh/config
Host gitlab-ce-kr6k1mdm
    HostName gitlab-ce-kr6k1mdm
    User git
    IdentityFile ~/.ssh/id_rsa
    IdentitiesOnly yes
EOF
```

### Run the Scan with SSH

```bash
trufflehog git git@gitlab-ce-kr6k1mdm:root/django-nv.git --json | tee secret.json
```

* `tee secret.json` → saves results to a JSON file
* If prompted about authenticity → type **yes** and press Enter

---

## 📂 Saving & Parsing Results

Always save output in **machine-readable formats** (CSV, JSON, XML).

Here we used JSON → `secret.json`.

### Pretty Print with `jq`

```bash
cat secret.json | jq
```

This formats the JSON output, making it easier to inspect.

---

## 📊 Secrets Detected (from JSON output)

1. **AWS Access Key**

   * **File:** `taskManager/settings.py`
   * **Commit:** `e2f15d93323ac871f4a6a0e8dd326df82b7a99cc`
   * **Verified:** ✅ Yes
   * **ExtraData:** Canary token from `canarytokens.org` (fake key for detection)

2. **Slack User Token**

   * **File:** `taskManager/settings.py`
   * **Commit:** same as above
   * **Verified:** ❌ No
   * **Rotation Guide:** [Slack token rotation guide](https://howtorotate.com/docs/tutorials/slack/)

---

✅ Next step: Would you like me to show you **how to filter only verified secrets from `secret.json` using jq**, or **how to export them into CSV for reporting**?


```md
How To Embed TruffleHog Into GitLab
A Simple CI/CD Pipeline
Considering your DevOps team created a simple CI pipeline with the following contents.

Click anywhere to copy

image: docker:20.10  # To run all jobs in this pipeline, use the latest docker image

services:
  - docker:dind       # To run all jobs in this pipeline, use a docker image that contains a docker daemon running inside (dind - docker in docker). Reference: https://forum.gitlab.com/t/why-services-docker-dind-is-needed-while-already-having-image-docker/43534

stages:
  - build
  - test
  - release
  - preprod
  - integration
  - prod

build:
  stage: build
  image: python:3.6
  before_script:
   - pip3 install --upgrade virtualenv
  script:
   - virtualenv env                       # Create a virtual environment for the python application
   - source env/bin/activate              # Activate the virtual environment
   - pip install -r requirements.txt      # Install the required third party packages as defined in requirements.txt
   - python manage.py check               # Run checks to ensure the application is working fine

test:
  stage: test
  image: python:3.6
  before_script:
   - pip3 install --upgrade virtualenv
  script:
   - virtualenv env
   - source env/bin/activate
   - pip install -r requirements.txt
   - python manage.py test taskManager

We have two jobs in this pipeline, a build job and a test job. As a security engineer, I do not care what they are doing as part of these jobs. Why? Imagine having to learn every build/testing tool used by your DevOps team, it will be a nightmare. Instead, rely on the DevOps team for help.

Let’s log in to Gitlab using the following details.

GitLab CI/CD Machine
Name	Value
Link	https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/django-nv/-/blob/main/.gitlab-ci.yml
Username	root
Password	pdso-training
Next, we need to create a CI/CD pipeline by adding the above content to the .gitlab-ci.yml file. Click on the Edit button to start adding the content.

Save changes to the file using Commit changes button.

Verify the pipeline run
As soon as a change is made to the repository, the pipeline starts executing the jobs.

We can see the results of this pipeline by visiting https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/django-nv/pipelines.

Click on the appropriate job name to see the output.

In the next step, we will embed Trufflehog in the CI/CD pipeline and follow all the best practices.
```

```md
Embed TruffleHog in CI/CD pipeline
As discussed in the Secrets Scanning exercise, we can embed TruffleHog in our CI/CD pipeline. However, remember that it’s important to locally test a tool before integrating it into the pipeline. Troubleshooting a tool manually in a local environment is much easier compared to troubleshooting it in a CI/CD system. Additionally, manually exploring the tool in a local environment helps you become familiar with the tool’s options and features.

Click anywhere to copy

image: docker:20.10  # To run all jobs in this pipeline, use the latest docker image

services:
  - docker:dind       # To run all jobs in this pipeline, use a docker image which contains a docker daemon running inside (dind - docker in docker). Reference: https://forum.gitlab.com/t/why-services-docker-dind-is-needed-while-already-having-image-docker/43534

stages:
  - build
  - test
  - release
  - preprod
  - integration
  - prod

build:
  stage: build
  image: python:3.6
  before_script:
   - pip3 install --upgrade virtualenv
  script:
   - virtualenv env                       # Create a virtual environment for the python application
   - source env/bin/activate              # Activate the virtual environment
   - pip install -r requirements.txt      # Install the required third party packages as defined in requirements.txt
   - python manage.py check               # Run checks to ensure the application is working fine

test:
  stage: test
  image: python:3.6
  before_script:
   - pip3 install --upgrade virtualenv
  script:
   - virtualenv env
   - source env/bin/activate
   - pip install -r requirements.txt
   - python manage.py test taskManager

git-secrets:
  stage: build
  script:
    - docker pull hysnsec/trufflehog
    - docker run --user $(id -u):$(id -g) -v $(pwd):/src --rm hysnsec/trufflehog git http://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/django-nv.git

As discussed, any change to the repo kick starts the pipeline.

Note

We’ve discussed different methods of using the tool:

Native Installation:
Directly installing the tool on the system.
Package Manager or Binary:
Installing via a package manager or using the binary file.
Docker:
Running the tool within a Docker container.
In summary:

All methods are suitable for CI/CD integration.
Docker is recommended for CI/CD as it operates smoothly without dependencies.
Using the binary file is efficient, avoiding additional dependencies.
Ultimately, you can choose either method based on your specific situation.

We can see the results of this pipeline by visiting https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/django-nv/pipelines.

Click on the appropriate job name to see the output.

Let’s try running the git status command after the Trufflehog command and see the job’s output.

Click anywhere to copy

git-secrets:
  stage: build
  script:
    - apk add git
    - docker pull hysnsec/trufflehog
    - docker run --user $(id -u):$(id -g) -v $(pwd):/src --rm hysnsec/trufflehog git http://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/django-nv.git
    - git status

Notice the last line of the above job, we have added git status command.

HEAD detached at c4560d9
nothing to commit, working tree clean
The output tells us that HEAD is detached; what does it mean? It means we’re not on a branch but checked out a specific commit in the history.

Our current job configuration is:

git-secrets:
  stage: build
  script:
    - docker run --user $(id -u):$(id -g) -v $(pwd):/src --rm hysnsec/trufflehog git http://gitlab-ce-7w3lmtto.lab.practical-devsecops.training/root/django-nv.git
The above configuration uses Trufflehog to scan the entire git repository for secrets.

Let’s break down the command:

--user $(id -u):$(id -g): runs the container with the current user’s UID and GID, avoiding permission issues.
-v $(pwd):/src: mounts the current directory to /src in the container.
--rm: removes the container after it finishes running.
hysnsec/trufflehog: the Docker image we’re using.
git http://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/django-nv.git: tells Trufflehog to use the git mode and scan the specified repository URL.
The git option in Trufflehog allows it to scan the entire git history of the repository, which is more thorough than just scanning the current state of the files. This is particularly useful for finding secrets that might have been committed in the past and then removed.

Notes
When using the git option with Trufflehog, we provide the repository URL as an argument. This allows Trufflehog to clone the repository and scan its entire history, regardless of the current state of the working directory in the CI/CD environment. This approach ensures a comprehensive scan of the entire git history for potential secrets.

Let’s move to the next step.
```
```md
Allow the job failure
Remember!

Except for DevSecOps-Box, every other machine closes after two hours, even if you are in the middle of the exercise
After two hours, in case of a 404, you need to refresh the exercise page and click on Start the Exercise button to continue working
We do not want to fail the builds/jobs/scan in DevSecOps Maturity Levels 1 and 2, as security tools spit a significant amount of false positives.

You can use the allow_failure tag to not fail the build even though the tool found issues.

git-secrets:
  stage: build
  script:
    - docker run -v $(pwd):/src --rm hysnsec/trufflehog git http://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/django-nv.git --fail --json | tee trufflehog-output.json
  artifacts:
    paths: [trufflehog-output.json]
    when: always  # What is this for?
    expire_in: one week
  allow_failure: true   #<--- allow the build to fail but don't mark it as such
After adding the allow_failure tag, the pipeline would look like the following.

Click anywhere to copy

image: docker:20.10  # To run all jobs in this pipeline, use the latest docker image

services:
  - docker:dind       # To run all jobs in this pipeline, use a docker image that contains a docker daemon running inside (dind - docker in docker). Reference: https://forum.gitlab.com/t/why-services-docker-dind-is-needed-while-already-having-image-docker/43534

stages:
  - build
  - test
  - release
  - preprod
  - integration
  - prod

build:
  stage: build
  image: python:3.6
  before_script:
   - pip3 install --upgrade virtualenv
  script:
   - virtualenv env                       # Create a virtual environment for the python application
   - source env/bin/activate              # Activate the virtual environment
   - pip install -r requirements.txt      # Install the required third party packages as defined in requirements.txt
   - python manage.py check               # Run checks to ensure the application is working fine

test:
  stage: test
  image: python:3.6
  before_script:
   - pip3 install --upgrade virtualenv
  script:
   - virtualenv env
   - source env/bin/activate
   - pip install -r requirements.txt
   - python manage.py test taskManager

git-secrets:
  stage: build
  script:
    - docker run -v $(pwd):/src --rm hysnsec/trufflehog git http://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/django-nv.git --fail --json | tee trufflehog-output.json
  artifacts:
    paths: [trufflehog-output.json]
    when: always  # What is this for?
    expire_in: one week
  allow_failure: true

integration:
  stage: integration
  script:
    - echo "This is an integration step"
    - exit 1
  allow_failure: true # Even if the job fails, continue to the next stages

prod:
  stage: prod
  script:
    - echo "This is a deploy step."
  when: manual # Continuous Delivery

You will notice that the git-secrets job failed however it didn’t block others from continuing further.

As discussed, any change to the repo kick starts the pipeline.

We can see the results of this pipeline by visiting https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/django-nv/pipelines.

Click on the appropriate job name to see the output.
```
```md
Static Analysis Using Bandit
Download the source code
We will do all the exercises locally first in DevSecOps-Box, so let’s start the exercise.

First, we need to download the source code of the project from our git repository.

git clone https://gitlab.practical-devsecops.training/pdso/django.nv webapp

Command Output
Cloning into 'webapp'...
warning: redirecting to https://gitlab.practical-devsecops.training/pdso/django.nv.git/
remote: Enumerating objects: 228, done.
remote: Total 228 (delta 0), reused 0 (delta 0), pack-reused 228
Receiving objects: 100% (228/228), 1.03 MiB | 1.04 MiB/s, done.
Resolving deltas: 100% (86/86), done.
Let’s cd into the application so we can scan the app.

cd webapp

We are now in the webapp directory.

Let’s move to the next step.
```
```md
Install Bandit
The Bandit is a tool designed to find common security issues in Python code.

To do this, Bandit processes each file, builds an AST, and runs appropriate plugins against the AST nodes. Once Bandit has finished scanning all the files, it generates a report.

Bandit was originally developed within the OpenStack Security Project and later rehomed to PyCQA.

You can find more details about the project at https://github.com/PyCQA/bandit.

Let’s install the Bandit scanner on the system to perform static analysis.

pip3 install bandit==1.7.4

Command Output
Collecting bandit==1.7.4
  Downloading bandit-1.7.4-py3-none-any.whl (118 kB)
     |████████████████████████████████| 118 kB 28 kB/s 
Collecting GitPython>=1.0.1
  Downloading GitPython-3.1.27-py3-none-any.whl (181 kB)
     |████████████████████████████████| 181 kB 45.3 MB/s 
Collecting stevedore>=1.20.0
  Downloading stevedore-4.0.0-py3-none-any.whl (49 kB)
     |████████████████████████████████| 49 kB 4.7 MB/s 
Collecting PyYAML>=5.3.1
  Downloading PyYAML-6.0-cp38-cp38-manylinux_2_5_x86_64.manylinux1_x86_64.manylinux_2_12_x86_64.manylinux2010_x86_64.whl (701 kB)
     |████████████████████████████████| 701 kB 39.7 MB/s 
Collecting gitdb<5,>=4.0.1
  Downloading gitdb-4.0.9-py3-none-any.whl (63 kB)
     |████████████████████████████████| 63 kB 928 kB/s 
Collecting pbr!=2.1.0,>=2.0.0
  Downloading pbr-5.9.0-py2.py3-none-any.whl (112 kB)
     |████████████████████████████████| 112 kB 38.9 MB/s 
Collecting smmap<6,>=3.0.1
  Downloading smmap-5.0.0-py3-none-any.whl (24 kB)
Installing collected packages: smmap, gitdb, GitPython, pbr, stevedore, PyYAML, bandit
Successfully installed GitPython-3.1.27 PyYAML-6.0 bandit-1.7.4 gitdb-4.0.9 pbr-5.9.0 smmap-5.0.0 stevedore-4.0.0
We have successfully installed the Bandit scanner. Let’s explore the functionality it provides us.

bandit --help

Command Output
usage: bandit [-h] [-r] [-a {file,vuln}] [-n CONTEXT_LINES] [-c CONFIG_FILE] [-p PROFILE] [-t TESTS] [-s SKIPS]
              [-l | --severity-level {all,low,medium,high}] [-i | --confidence-level {all,low,medium,high}]
              [-f {csv,custom,html,json,screen,txt,xml,yaml}] [--msg-template MSG_TEMPLATE] [-o [OUTPUT_FILE]] [-v]
              [-d] [-q] [--ignore-nosec] [-x EXCLUDED_PATHS] [-b BASELINE] [--ini INI_PATH] [--exit-zero] [--version]
              [targets [targets ...]]

Bandit - a Python source code security analyzer

positional arguments:
  targets               source file(s) or directory(s) to be tested

optional arguments:
  -h, --help            show this help message and exit
  -r, --recursive       find and process files in subdirectories
  -a {file,vuln}, --aggregate {file,vuln}
                        aggregate output by vulnerability (default) or by filename
  -n CONTEXT_LINES, --number CONTEXT_LINES
                        maximum number of code lines to output for each issue
  -c CONFIG_FILE, --configfile CONFIG_FILE
                        optional config file to use for selecting plugins and overriding defaults
  -p PROFILE, --profile PROFILE
                        profile to use (defaults to executing all tests)
  -t TESTS, --tests TESTS
                        comma-separated list of test IDs to run
  -s SKIPS, --skip SKIPS
                        comma-separated list of test IDs to skip
  -l, --level           report only issues of a given severity level or higher (-l for LOW, -ll for MEDIUM, -lll for
                        HIGH)
  --severity-level {all,low,medium,high}
                        report only issues of a given severity level or higher. "all" and "low" are likely to produce
                        the same results, but it is possible for rules to be undefined which will not be listed in
                        "low".
  -i, --confidence      report only issues of a given confidence level or higher (-i for LOW, -ii for MEDIUM, -iii for
                        HIGH)
  --confidence-level {all,low,medium,high}
                        report only issues of a given confidence level or higher. "all" and "low" are likely to
                        produce the same results, but it is possible for rules to be undefined which will not be
                        listed in "low".
  -f {csv,custom,html,json,screen,txt,xml,yaml}, --format {csv,custom,html,json,screen,txt,xml,yaml}
                        specify output format
  --msg-template MSG_TEMPLATE
                        specify output message template (only usable with --format custom), see CUSTOM FORMAT section
                        for list of available values
  -o [OUTPUT_FILE], --output [OUTPUT_FILE]
                        write report to filename
  -v, --verbose         output extra information like excluded and included files
  -d, --debug           turn on debug mode
  -q, --quiet, --silent
                        only show output in the case of an error
  --ignore-nosec        do not skip lines with # nosec comments
  -x EXCLUDED_PATHS, --exclude EXCLUDED_PATHS
                        comma-separated list of paths (glob patterns supported) to exclude from scan (note that these
                        are in addition to the excluded paths provided in the config file) (default:
                        .svn,CVS,.bzr,.hg,.git,__pycache__,.tox,.eggs,*.egg)
  -b BASELINE, --baseline BASELINE
                        path of a baseline report to compare against (only JSON-formatted files are accepted)
  --ini INI_PATH        path to a .bandit file that supplies command line arguments
  --exit-zero           exit with 0, even with results found
  --version             show program's version number and exit

...[SNIP]...
Let’s move to the next step.
```
```md
Run the scanner
As we have learned in the DevSecOps Gospel, we would like to store the tool results in a JSON file. We are using the tee command here to show the output and store it in a file simultaneously.

bandit -r . -f json | tee bandit-output.json

Bandit ran successfully, and it found seven security issues.
1. One high severity issue
2. Five medium severity issue
3. One low severity issue

Variable Scan Results

Your results might slightly vary because of the dynamic landscape of changing vulnerabilities, and security updates.

Filtering Vulnerabilities by Severity Level
In the previous step, we learned how to run Bandit and, based on the information provided in the bandit –help command, we can filter vulnerabilities by their severity levels. Bandit categorizes vulnerabilities into multiple severity levels: LOW, MEDIUM, and HIGH. You can customize your scan to target specific severity levels. Here are some examples:

To filter, you can use this command:

Command Output
bandit -r <path_to_code> -l
However, since we are already in the directory, you can use a dot (“.”) instead.

To filter for low-level vulnerabilities:

bandit -r . -l

This command filters vulnerabilities with a severity level from low to high.

Filtering Severity Low

To filter for medium-level vulnerabilities:

bandit -r . -ll

This command filters vulnerabilities with a severity level from medium to high.

Filtering Severity Medium

To filter for high-level vulnerabilities:

bandit -r . -lll

This command filters vulnerabilities with a high severity level.

Filtering Severity High

By specifying the number of l, you can filter vulnerabilities of the desired severity level and higher. This is useful for customizing your security analysis based on your project’s requirements.

With these additional steps, you’ll have a better understanding of Bandit’s exit codes and how to filter vulnerabilities by severity level when running the tool. This knowledge will help you make informed decisions and automate security checks in your CI/CD pipeline.

Understanding Bandit Exit Codes
It’s crucial to grasp how Bandit’s exit codes function and what they signify. These exit codes play a significant role in automating security checks within your CI/CD pipeline and making informed decisions about the results. You can refer to this understanding in the linked exercise, Introduction to Exit Codes, to enhance your knowledge further.

Exit Code	Description
0	This code is returned when Bandit finds no security issues in the scanned code.
1	Bandit issues this code when it detects security issues of the specified severity level or higher.
2	If an error occurs during the Bandit scan, such as an invalid command-line argument or an issue with the code being scanned.
Understanding these exit codes is crucial for the exercise mentioned in Introduction to Exit Codes. It will help you effectively integrate Bandit into your CI/CD pipeline and make informed decisions based on the scan results.
```
```md
How To Embed Bandit Into GitLab
A Simple CI/CD Pipeline
Considering your DevOps team created a simple CI pipeline with the following contents.

Click anywhere to copy

image: docker:20.10  # To run all jobs in this pipeline, use the latest docker image

services:
  - docker:dind       # To run all jobs in this pipeline, use a docker image that contains a docker daemon running inside (dind - docker in docker). Reference: https://forum.gitlab.com/t/why-services-docker-dind-is-needed-while-already-having-image-docker/43534

stages:
  - build
  - test
  - release
  - preprod
  - integration
  - prod

build:
  stage: build
  image: python:3.6
  before_script:
   - pip3 install --upgrade virtualenv
  script:
   - virtualenv env                       # Create a virtual environment for the python application
   - source env/bin/activate              # Activate the virtual environment
   - pip install -r requirements.txt      # Install the required third party packages as defined in requirements.txt
   - python manage.py check               # Run checks to ensure the application is working fine

test:
  stage: test
  image: python:3.6
  before_script:
   - pip3 install --upgrade virtualenv
  script:
   - virtualenv env
   - source env/bin/activate
   - pip install -r requirements.txt
   - python manage.py test taskManager

integration:
  stage: integration
  script:
    - echo "This is an integration step"
    - exit 1
  allow_failure: true # Even if the job fails, continue to the next stages

prod:
  stage: prod
  script:
    - echo "This is a deploy step."
  when: manual # Continuous Delivery

We have four jobs in this pipeline, a build job, a test job, a integration job and a prod job. We did integrate SCA/OAST beforehand, we can carry forward the same tactics in this exercise as well.

As a security engineer, I don’t need to care much about what the DevOps team is doing as part of these jobs. Why? imagine having to learn every build/testing tool used by your DevOps team, it will be a nightmare. Instead, rely on the DevOps team for help.

Let’s login into the GitLab using the following details and execute this pipeline.

GitLab CI/CD Machine
Name	Value
Link	https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/django-nv/-/blob/main/.gitlab-ci.yml
Username	root
Password	pdso-training
Next, we need to create a CI/CD pipeline by replacing the .gitlab-ci.yml file content with the above CI script. Click on the Edit button to replace the content (use Control+A and Control+V).

Save changes to the file using the Commit changes button.

Verify the pipeline run
As soon as a change is made to the repository, the pipeline starts executing the jobs.

We can see the results of this pipeline by visiting https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/django-nv/pipelines.

Click on the appropriate job name to see the output.

In the next step, we will embed Bandit in the CI/CD pipeline and follow all the best practices.

Let’s move to the next step.
```
```md
Embed Bandit in CI/CD pipeline
As discussed in the Static Analysis using Bandit exercise, we can embed Bandit in our CI/CD pipeline. However, remember that it’s important to locally test a tool before integrating it into the pipeline. Troubleshooting a tool manually in a local environment is much easier compared to troubleshooting it in a CI/CD system. Additionally, manually exploring the tool in a local environment helps you become familiar with the tool’s options and features.

Click anywhere to copy

image: docker:20.10  # To run all jobs in this pipeline, use the latest docker image

services:
  - docker:dind       # To run all jobs in this pipeline, use a docker image that contains a docker daemon running inside (dind - docker in docker). Reference: https://forum.gitlab.com/t/why-services-docker-dind-is-needed-while-already-having-image-docker/43534

stages:
  - build
  - test
  - release
  - preprod
  - integration
  - prod

build:
  stage: build
  image: python:3.6
  before_script:
   - pip3 install --upgrade virtualenv
  script:
   - virtualenv env                       # Create a virtual environment for the python application
   - source env/bin/activate              # Activate the virtual environment
   - pip install -r requirements.txt      # Install the required third party packages as defined in requirements.txt
   - python manage.py check               # Run checks to ensure the application is working fine

test:
  stage: test
  image: python:3.6
  before_script:
   - pip3 install --upgrade virtualenv
  script:
   - virtualenv env
   - source env/bin/activate
   - pip install -r requirements.txt
   - python manage.py test taskManager

sast:
  stage: build  # we moved this job from test stage to build stage, by replacing the text test to build
  script:
    # Download bandit docker container
    - docker pull hysnsec/bandit
    # Run docker container, please refer docker security course, if this doesn't make sense to you.
    - docker run --user $(id -u):$(id -g) -v $(pwd):/src --rm hysnsec/bandit -r /src -f json -o /src/bandit-output.json
  artifacts:
    paths: [bandit-output.json]
    when: always

integration:
  stage: integration
  script:
    - echo "This is an integration step"
    - exit 1
  allow_failure: true # Even if the job fails, continue to the next stages

prod:
  stage: prod
  script:
    - echo "This is a deploy step."
  when: manual # Continuous Delivery

As discussed, any change to the repo kick starts the pipeline.

Note

We’ve discussed different methods of using the tool:

Native Installation:
Directly installing the tool on the system.
Package Manager or Binary:
Installing via a package manager or using the binary file.
Docker:
Running the tool within a Docker container.
In summary:

All methods are suitable for CI/CD integration.
Docker is recommended for CI/CD as it operates smoothly without dependencies.
Using the binary file is efficient, avoiding additional dependencies.
Ultimately, you can choose either method based on your specific situation.

We can see the results of this pipeline by visiting https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/django-nv/pipelines.

Click on the appropriate job name to see the output.

You will notice that the sast job’s output is saved in bandit-output.json file.

Let’s move to the next step.
```
```md
Allow the job failure
Remember!

Except for DevSecOps-Box, every other machine closes after two hours, even if you are in the middle of the exercise
After two hours, in case of a 404, you need to refresh the exercise page and click on Start the Exercise button to continue working
We do not want to fail the builds/jobs/scan in DevSecOps Maturity Levels 1 and 2, as security tools spit a significant amount of false positives.

You can use the allow_failure tag to not fail the build even though the tool found issues.

sast:
  stage: build
  script:
    - docker pull hysnsec/bandit  # Download bandit docker container
    # Run docker container, please refer docker security course, if this doesn't make sense to you.
    - docker run --user $(id -u):$(id -g) -v $(pwd):/src --rm hysnsec/bandit -r /src -f json -o /src/bandit-output.json
  artifacts:
    paths: [bandit-output.json]
    when: always
  allow_failure: true   #<--- allow the build to fail but don't mark it as such
After adding the allow_failure tag, the pipeline would look like the following.

Click anywhere to copy

image: docker:20.10  # To run all jobs in this pipeline, use the latest docker image

services:
  - docker:dind       # To run all jobs in this pipeline, use a docker image that contains a docker daemon running inside (dind - docker in docker). Reference: https://forum.gitlab.com/t/why-services-docker-dind-is-needed-while-already-having-image-docker/43534

stages:
  - build
  - test
  - release
  - preprod
  - integration
  - prod

build:
  stage: build
  image: python:3.6
  before_script:
   - pip3 install --upgrade virtualenv
  script:
   - virtualenv env                       # Create a virtual environment for the python application
   - source env/bin/activate              # Activate the virtual environment
   - pip install -r requirements.txt      # Install the required third party packages as defined in requirements.txt
   - python manage.py check               # Run checks to ensure the application is working fine

test:
  stage: test
  image: python:3.6
  before_script:
   - pip3 install --upgrade virtualenv
  script:
   - virtualenv env
   - source env/bin/activate
   - pip install -r requirements.txt
   - python manage.py test taskManager

sast:
  stage: build
  script:
    - docker pull hysnsec/bandit  # Download bandit docker container
    # Run docker container, please refer docker security course, if this doesn't make sense to you.
    - docker run --user $(id -u):$(id -g) -v $(pwd):/src --rm hysnsec/bandit -r /src -f json -o /src/bandit-output.json
  artifacts:
    paths: [bandit-output.json]
    when: always
  allow_failure: true   #<--- allow the build to fail but don't mark it as such

integration:
  stage: integration
  script:
    - echo "This is an integration step"
    - exit 1
  allow_failure: true # Even if the job fails, continue to the next stages

prod:
  stage: prod
  script:
    - echo "This is a deploy step."
  when: manual # Continuous Delivery

You will notice that the sast job failed however it didn’t block others from continuing further.

As discussed, any change to the repo kick starts the pipeline.

We can see the results of this pipeline by visiting https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/django-nv/pipelines.

Click on the appropriate job name to see the output.

Notes
The above option mounts the current directory in the host (runner) to /src inside the container. This could also be -v /home/ubuntu/code:/src or c:\users\code:/src if you were using windows.

Instead of manually removing the container after the scan, we can use --rm option so docker can clean up after itself.

What is this --user option here, and why are we using it? Can you please try removing this option --user $(id -u):$(id -g) and see what would happen?

For more details please refer to https://medium.com/redbubble/running-a-docker-container-as-a-non-root-user-7d2e00f8ee15

Let’s move to the next step.
```
```md
Extra Mile: Write a custom wrapper for the bandit tool to fail the build only when you found 3 high, 1 low, and 1 medium issue
Who should do this challenges?
This exercise is beyond the scope of the CDP course and is added to help folks who already know these concepts in and out
You know how to write small scripts in Python
You consider yourself an expert or advanced user in SAST
Challenges
Recall techniques you have learned in the previous exercises.

Read the bandit documentation
Write a wrapper to parse the bandit output
Make the output pretty (Red = High Severity, Blue = Medium Severity, Orange = Low Severity)
Once done, create a docker image with the help of a Dockerfile
Run the docker image in CI/CD pipeline to pretty-print the output
Note

We will not provide solutions for this extra mile exercise.
```
```md
False Positive Analysis (FPA)
Download the Source Code
We will do all the exercises locally first in DevSecOps-Box, so let’s start the exercise.

First, we need to download the source code of the project from our git repository.

git clone https://gitlab.practical-devsecops.training/pdso/dvpa-api

Command Output
Cloning into 'dvpa-api'...
warning: redirecting to https://gitlab.practical-devsecops.training/pdso/dvpa-api.git/
remote: Enumerating objects: 77, done.
remote: Total 77 (delta 0), reused 0 (delta 0), pack-reused 77
Unpacking objects: 100% (77/77), 234.47 KiB | 333.00 KiB/s, done.
Let’s cd into the application so we can scan the app.

cd dvpa-api

We are now in the dvpa-api directory.

Let’s move to the next step.
```
```md
Install Bandit
The Bandit is a tool designed to find common security issues in Python code.

To do this Bandit, processes each file, builds an AST, and runs appropriate plugins against the AST nodes. Once Bandit has finished scanning all the files it generates a report.

Bandit was originally developed within the OpenStack Security Project and later rehomed to PyCQA.

You can find more details about the project at https://github.com/PyCQA/bandit.

Let’s install the bandit scanner on the system to perform static analysis.

pip3 install bandit==1.7.4

Command Output
Collecting bandit==1.7.4
  Downloading bandit-1.7.4-py3-none-any.whl (118 kB)
     |████████████████████████████████| 118 kB 28 kB/s 
Collecting GitPython>=1.0.1
  Downloading GitPython-3.1.27-py3-none-any.whl (181 kB)
     |████████████████████████████████| 181 kB 45.3 MB/s 
Collecting stevedore>=1.20.0
  Downloading stevedore-4.0.0-py3-none-any.whl (49 kB)
     |████████████████████████████████| 49 kB 4.7 MB/s 
Collecting PyYAML>=5.3.1
  Downloading PyYAML-6.0-cp38-cp38-manylinux_2_5_x86_64.manylinux1_x86_64.manylinux_2_12_x86_64.manylinux2010_x86_64.whl (701 kB)
     |████████████████████████████████| 701 kB 39.7 MB/s 
Collecting gitdb<5,>=4.0.1
  Downloading gitdb-4.0.9-py3-none-any.whl (63 kB)
     |████████████████████████████████| 63 kB 928 kB/s 
Collecting pbr!=2.1.0,>=2.0.0
  Downloading pbr-5.9.0-py2.py3-none-any.whl (112 kB)
     |████████████████████████████████| 112 kB 38.9 MB/s 
Collecting smmap<6,>=3.0.1
  Downloading smmap-5.0.0-py3-none-any.whl (24 kB)
Installing collected packages: smmap, gitdb, GitPython, pbr, stevedore, PyYAML, bandit
Successfully installed GitPython-3.1.27 PyYAML-6.0 bandit-1.7.4 gitdb-4.0.9 pbr-5.9.0 smmap-5.0.0 stevedore-4.0.0
We have successfully installed Bandit scanner. Let’s explore the functionality it provides us.

bandit --help

Command Output
usage: bandit [-h] [-r] [-a {file,vuln}] [-n CONTEXT_LINES] [-c CONFIG_FILE] [-p PROFILE] [-t TESTS] [-s SKIPS]
              [-l | --severity-level {all,low,medium,high}] [-i | --confidence-level {all,low,medium,high}]
              [-f {csv,custom,html,json,screen,txt,xml,yaml}] [--msg-template MSG_TEMPLATE] [-o [OUTPUT_FILE]] [-v]
              [-d] [-q] [--ignore-nosec] [-x EXCLUDED_PATHS] [-b BASELINE] [--ini INI_PATH] [--exit-zero] [--version]
              [targets [targets ...]]

Bandit - a Python source code security analyzer

positional arguments:
  targets               source file(s) or directory(s) to be tested

optional arguments:
  -h, --help            show this help message and exit
  -r, --recursive       find and process files in subdirectories
  -a {file,vuln}, --aggregate {file,vuln}
                        aggregate output by vulnerability (default) or by filename
  -n CONTEXT_LINES, --number CONTEXT_LINES
                        maximum number of code lines to output for each issue
  -c CONFIG_FILE, --configfile CONFIG_FILE
                        optional config file to use for selecting plugins and overriding defaults
  -p PROFILE, --profile PROFILE
                        profile to use (defaults to executing all tests)
  -t TESTS, --tests TESTS
                        comma-separated list of test IDs to run
  -s SKIPS, --skip SKIPS
                        comma-separated list of test IDs to skip
  -l, --level           report only issues of a given severity level or higher (-l for LOW, -ll for MEDIUM, -lll for
                        HIGH)
  --severity-level {all,low,medium,high}
                        report only issues of a given severity level or higher. "all" and "low" are likely to produce
                        the same results, but it is possible for rules to be undefined which will not be listed in
                        "low".
  -i, --confidence      report only issues of a given confidence level or higher (-i for LOW, -ii for MEDIUM, -iii for
                        HIGH)
  --confidence-level {all,low,medium,high}
                        report only issues of a given confidence level or higher. "all" and "low" are likely to
                        produce the same results, but it is possible for rules to be undefined which will not be
                        listed in "low".
  -f {csv,custom,html,json,screen,txt,xml,yaml}, --format {csv,custom,html,json,screen,txt,xml,yaml}
                        specify output format
  --msg-template MSG_TEMPLATE
                        specify output message template (only usable with --format custom), see CUSTOM FORMAT section
                        for list of available values
  -o [OUTPUT_FILE], --output [OUTPUT_FILE]
                        write report to filename
  -v, --verbose         output extra information like excluded and included files
  -d, --debug           turn on debug mode
  -q, --quiet, --silent
                        only show output in the case of an error
  --ignore-nosec        do not skip lines with # nosec comments
  -x EXCLUDED_PATHS, --exclude EXCLUDED_PATHS
                        comma-separated list of paths (glob patterns supported) to exclude from scan (note that these
                        are in addition to the excluded paths provided in the config file) (default:
                        .svn,CVS,.bzr,.hg,.git,__pycache__,.tox,.eggs,*.egg)
  -b BASELINE, --baseline BASELINE
                        path of a baseline report to compare against (only JSON-formatted files are accepted)
  --ini INI_PATH        path to a .bandit file that supplies command line arguments
  --exit-zero           exit with 0, even with results found
  --version             show program's version number and exit

...[SNIP]...
Let’s move to the next step.
```
```md
Run the scanner
Let’s scan our source code by executing the following command:

bandit -r .

Command Output
[main]  INFO    profile include tests: None
[main]  INFO    profile exclude tests: None
[main]  INFO    cli include tests: None
[main]  INFO    cli exclude tests: None
[main]  INFO    running on Python 3.8.10
Run started:2022-08-10 07:07:58.982038

Test results:
>> Issue: [B303:blacklist] Use of insecure MD2, MD4, MD5, or SHA1 hash function.
   Severity: Medium   Confidence: High
   CWE: CWE-327 (https://cwe.mitre.org/data/definitions/327.html)
   Location: ./flaskblog/auth.py:13:23
   More Info: https://bandit.readthedocs.io/en/1.7.4/blacklists/blacklist_calls.html#b303-md5
12          cur = db.connection.cursor()
13          hashsed_password = hashlib.md5(password.encode()).hexdigest()
14          cur.execute(f"SELECT * FROM users WHERE email='{username}' AND password='{hashsed_password}'")

...[SNIP]...

--------------------------------------------------
>> Issue: [B105:hardcoded_password_string] Possible hardcoded password: 'secret'
   Severity: Low   Confidence: Medium
   CWE: CWE-259 (https://cwe.mitre.org/data/definitions/259.html)
   Location: ./flaskblog/config.py:13:11
   More Info: https://bandit.readthedocs.io/en/1.7.4/plugins/b105_hardcoded_password_string.html
12      username = 'admin'
13      password = 'secret'
14
15      # Disqus Configuration
16      disqus_shortname = 'blogpythonlearning'  # please change this.

--------------------------------------------------

Code scanned:
        Total lines of code: 603
        Total lines skipped (#nosec): 0

Run metrics:
        Total issues (by severity):
                Undefined: 0
                Low: 2
                Medium: 11
                High: 0
        Total issues (by confidence):
                Undefined: 0
                Low: 0
                Medium: 9
                High: 4
Files skipped (0):
We found 13 issues in total (by severity), but we need to exclude the False Positives.

The output number may vary as it is dynamic.

Let’s move to the next step.
```
```md
False Positive Analysis (FPA)
There are two ways to perform False Positive Analysis: reading the source code or exploiting the vulnerability. In this exercise, we will only cover the first method.

According to Bandit’s scan results, we have identified the following security issues: hardcoded password strings, an insecure hash function issue, an insecure deserialization issue, and a SQL injection vulnerability.

Now, let’s analyze whether these findings are real issues or false positives. We will focus on investigating three specific issues from the list.

>> Issue: [B303:blacklist] Use of insecure MD2, MD4, MD5, or SHA1 hash function.
   Severity: Medium   Confidence: High
   CWE: CWE-327 (https://cwe.mitre.org/data/definitions/327.html)
   Location: ./flaskblog/auth.py:13:23
   More Info: https://bandit.readthedocs.io/en/1.7.4/blacklists/blacklist_calls.html#b303-md5
12          cur = db.connection.cursor()
13          hashsed_password = hashlib.md5(password.encode()).hexdigest()
14          cur.execute(f"SELECT * FROM users WHERE email='{username}' AND password='{hashsed_password}'")

--------------------------------------------------
>> Issue: [B608:hardcoded_sql_expressions] Possible SQL injection vector through string-based query construction.
   Severity: Medium   Confidence: Medium
   CWE: CWE-89 (https://cwe.mitre.org/data/definitions/89.html)
   Location: ./flaskblog/auth.py:14:16
   More Info: https://bandit.readthedocs.io/en/1.7.4/plugins/b608_hardcoded_sql_expressions.html
13          hashsed_password = hashlib.md5(password.encode()).hexdigest()
14          cur.execute(f"SELECT * FROM users WHERE email='{username}' AND password='{hashsed_password}'")
15          user = cur.fetchone()

...[SNIP]...
Analysis of the issues
Let’s explore the first issue now.

99              cur = db.connection.cursor()
100             cur.execute(f"UPDATE `users` SET `email` = '{email}', `full_name` = '{full_name}', `phone_number` = '{phone_number}', `dob` = '{dob}' WHERE `users`.`id` = {request.args.get('uid')}")
101             db.connection.commit()
It looks like the above code is vulnerable to SQL Injection because uid is used in the query directly, so it’s not a False Positive.

Next, lets check out the second result.

132             cur.execute(
133                 f"INSERT INTO posts (`body`, `slug`, `author`, `title`) VALUES (%s, %s, %s, %s)",
134                 [body, slug, claim.get("id"), title])
The above code is definitely False Positive as we are using Parameter Binding to create the query.

And the last one is a known vulnerability in Python’s YAML library called YAML deserialization. We can search for this known security issue on the CVE website.

247                 elif export_format == "yaml":
248                     import_post_data = yaml.load(import_data)
249
For more details about this issue, please visit CVE-2020-1747.

As mentioned before, our code is vulnerable to Deserialization Attacks and we can mark this as not a False Positive.

Let’s move to the next step.
```
```md
Excluding Issues Using the Baseline Feature
In the Bandit tool, there is an argument available for ignoring known vulnerabilities. This can be used during False Positive analysis to prevent the same issues from reappearing in subsequent scans. Alternatively, you can use #nosec within the source code to ignore an issue without needing to provide a baseline file. However, this approach is not recommended for marking issues as False Positives for three main reasons:

It requires parsing through the entire codebase to list all False Positives (FP).
It doesn’t allow for programmatic marking of these issues as FP in Vulnerability Management systems such as DefectDojo.
It doesn’t support creating custom criteria to fail a build in maturity levels 3 and 4.
Using #nosec may not be considered a DevSecOps-friendly way to handle False Positives because it lacks the advantages of automated management and integration with existing systems, which can lead to inefficiencies and potential oversight of security vulnerabilities.

Baseline Feature
The baseline file contains issues you would want to mark as False Positives.

This is useful for ignoring known vulnerabilities that you believe are non-issues (e.g. a cleartext password in a unit test).

Note

It’s similar to .retireignore.json, brakeman.ignore etc., files we have seen in other labs.

In layman terms, the issues you want to mark as False Positives, you will add it to this file. The real issues should not be present in this file.

In order to work with baseline feature, you need to generate the baseline.json file using the below command.

bandit -r . -f json | tee baseline.json

To update the baseline.json file in your preferred text editor, follow these steps:

Open the baseline.json file using any text editor of your choice.
Locate the results field within the file. This field contains the list of issues identified by Bandit.
Review each issue present in the results field.
Remove the issues that are NOT false positives from the file, keeping only the issues that are indeed false positives.
Save the changes made to the baseline.json file.
By removing the non-false positive issues from the baseline.json file, you can ensure that only the desired false positives are retained within the file for subsequent analysis or scanning processes.

Or the final baseline file will look like the following content if we marked Possible hardcoded password: ‘secret’ issue as False Positive

cat > baseline.json<<EOF
{
  "results": [
    {
      "code": "12 username = 'admin'\n13 password = 'secret'\n14 \n15 # Disqus Configuration\n16 disqus_shortname = 'blogpythonlearning'  # please change this.\n",
      "col_offset": 11,
      "filename": "./flaskblog/config.py",
      "issue_confidence": "MEDIUM",
      "issue_cwe": {
        "id": 259,
        "link": "https://cwe.mitre.org/data/definitions/259.html"
      },
      "issue_severity": "LOW",
      "issue_text": "Possible hardcoded password: 'secret'",
      "line_number": 13,
      "line_range": [
        13,
        14,
        15
      ],
      "more_info": "https://bandit.readthedocs.io/en/1.7.4/plugins/b105_hardcoded_password_string.html",
      "test_id": "B105",
      "test_name": "hardcoded_password_string"
    }
  ]
}
EOF

Note

Remember we saved the output of the bandit as baseline.json hence all the issues found by Bandit in the previous scan are a part of the baseline.json file. If we do not modify the baseline.json file, and use the baseline.json file as is, then all the issues reported by bandit would be marked as False Positives.

When supplying the baseline.json file, bandit will not report the issues present in the baseline.json file in the subsequent scans.

Before using the baseline feature, let’s check the number of vulnerabilities we found without ignoring any false positives.

bandit -r . -f json | jq '.results | length'

Command Output
[main]  INFO    profile include tests: None
[main]  INFO    profile exclude tests: None
[main]  INFO    cli include tests: None
[main]  INFO    cli exclude tests: None
13
The aforementioned result represents the total number of vulnerabilities we discovered prior to implementing the baseline.

Let’s use the baseline feature to observe the results.

bandit -r . -f json -b baseline.json

Check the result once again.

bandit -r . -f json -b baseline.json | jq '.results | length'

Command Output
[main]  INFO    profile include tests: None
[main]  INFO    profile exclude tests: None
[main]  INFO    cli include tests: None
[main]  INFO    cli exclude tests: None
12
Wow, the total count has decreased by one after ignoring a specific issue.

Recap
Ignoring the issue using this feature is straightforward. You only need to obtain the output from the initial scan and use it as a baseline file. Simply remove any True Positives from the baseline file, leaving only false positive issues on the list, and use it in the subsequent scan. This allows you to exclude False Positive issues from being reported in the results.

In short baseline.json will be the file, where you will store/mark issues as false positives.

You can read more about the baseline feature at this link: https://bandit.readthedocs.io/en/latest/start.html?#baseline

Let’s move to the next step.
```
```md
Challenges
In this exericse, you will mark an issue as a False Positive using the bandit’s baseline feature (https://bandit.readthedocs.io/en/latest/start.html?#baseline)

# 1
Create a baseline file for the dvpa-api source code. Include any issues marked as False Positive. Save the baseline file at /dvpa-api/baseline.json. Specifically, mark the issue in line 132 of the file ./flaskblog/blogapi/dashboard.py, related to potential SQL injection, as a false positive


Answer

How would you mark the following issue as a false positive now that you have learned to use the baseline feature?

Note

Please remove any issues from the baseline.json file that you believe are not false positives. This file should only contain false positive issues.

Here is the final baseline content.


cat > baseline.json <<'EOF'
{
  "results": [
    {
      "code": "132         cur.execute(\n133             f\"INSERT INTO posts (`body`, `slug`, `author`, `title`) VALUES (%s, %s, %s, %s)\",\n134             [body, slug, claim.get(\"id\"), title])\n",
      "col_offset": 12,
      "filename": "./flaskblog/blogapi/dashboard.py",
      "issue_confidence": "MEDIUM",
      "issue_cwe": {
        "id": 89,
        "link": "https://cwe.mitre.org/data/definitions/89.html"
      },
      "issue_severity": "MEDIUM",
      "issue_text": "Possible SQL injection vector through string-based query construction.",
      "line_number": 133,
      "line_range": [
        133
      ],
      "more_info": "https://bandit.readthedocs.io/en/1.7.4/plugins/b608_hardcoded_sql_expressions.html",
      "test_id": "B608",
      "test_name": "hardcoded_sql_expressions"
    }
  ]
}
EOF

Please save the content above as a file named baseline.json using any text editor you prefer.

Why the chosen issue is a false positive
The issue chosen here is in line # 132 in the file ./flaskblog/blogapi/dashboard.py.

The issue is a false positive because of the usage of %s in the SQL statement.

Using %s in the SQL statement, enforces python to use parameterized queries. Hence we will use the baseline file to mask this particular issue as false positive.

Command Output
"code": "132 cur.execute(\n133 f\"INSERT INTO posts (`body`, `slug`, `author`, `title`) VALUES (%s, %s, %s, %s)\",\n134 [body, slug, claim.get(\"id\"), title])\n",
# 2
Please run the scan and check using grep to ensure that the 132 issues marked as False Positives do not appear in the scan results.
```

```md
Static Analysis Using Brakeman
Download the source code
We will do all the exercises locally first in DevSecOps-Box, so let’s start the exercise.

First, we need to download the source code of the project from our git repository.

git clone https://gitlab.practical-devsecops.training/pdso/rails.git webapp

Let’s cd into the application so we can scan the app.

cd webapp

We are now in the webapp directory.

Let’s move to the next step.
```
```md
Install Brakeman
Brakeman is a Static Analysis tool for Rails applications to find vulnerabilities. It is fast, flexible, and comes with very good reporting, making it fit to embed in a CI/CD pipeline.

You can find more details about the project at https://brakemanscanner.org/.

Basically, our system doesn’t have Ruby installed, so let’s update apt first.

apt update

Then, let’s install Ruby with the following command.

apt install ruby-full -y

Let’s install the Brakeman tool to perform static analysis.

gem install brakeman -v 5.2.1

Command Output
Fetching: brakeman-5.2.1.gem (100%)
Successfully installed brakeman-5.2.1
Parsing documentation for brakeman-5.2.1
Installing ri documentation for brakeman-5.2.1
Done installing documentation for brakeman after 3 seconds
1 gem installed
We have successfully installed Brakeman. Let’s explore the functionality it provides us.

brakeman -h

Command Output
Usage: brakeman [options] rails/root/path
    -n, --no-threads                 Run checks and file parsing sequentially
        --[no-]progress              Show progress reports
    -p, --path PATH                  Specify path to Rails application
    -q, --[no-]quiet                 Suppress informational messages
    -z, --[no-]exit-on-warn          Exit code is non-zero if warnings found (Default)
        --[no-]exit-on-error         Exit code is non-zero if errors raised (Default)
        --ensure-latest              Fail when Brakeman is outdated
        --ensure-ignore-notes        Fail when an ignored warning does not include a note
    -3, --rails3                     Force Rails 3 mode
    -4, --rails4                     Force Rails 4 mode
    -5, --rails5                     Force Rails 5 mode
    -6, --rails6                     Force Rails 6 mode
    -7, --rails7                     Force Rails 7 mode

Scanning options:
    -A, --run-all-checks             Run all default and optional checks
    -a, --[no-]assume-routes         Assume all controller methods are actions (Default)
    -e, --escape-html                Escape HTML by default
        --faster                     Faster, but less accurate scan
        --ignore-model-output        Consider model attributes XSS-safe
        --ignore-protected           Consider models with attr_protected safe
        --[no-]index-libs            Add libraries to call index (Default)
        --interprocedural            Process method calls to known methods
        --no-branching               Disable flow sensitivity on conditionals
        --branch-limit LIMIT         Limit depth of values in branches (-1 for no limit)
        --parser-timeout SECONDS     Set parse timeout (Default: 10)
    -r, --report-direct              Only report direct use of untrusted data
    -s meth1,meth2,etc,              Set methods as safe for unescaped output in views
        --safe-methods
        --sql-safe-methods meth1,meth2,etc
                                     Do not warn of SQL if the input is wrapped in a safe method
        --url-safe-methods method1,method2,etc
                                     Do not warn of XSS if the link_to href parameter is wrapped in a safe method
        --skip-files file1,path2,etc Skip processing of these files/directories. Directories are application relative and must end in "/"
        --only-files file1,path2,etc Process only these files/directories. Directories are application relative and must end in "/"
        --[no-]skip-vendor           Skip processing vendor directory (Default)
        --skip-libs                  Skip processing lib directory
        --add-libs-path path1,path2,etc
                                     An application relative lib directory (ex. app/mailers) to process
        --add-engines-path path1,path2,etc
                                     Include these engines in the scan
    -E, --enable Check1,Check2,etc   Enable the specified checks
    -t, --test Check1,Check2,etc     Only run the specified checks
    -x, --except Check1,Check2,etc   Skip the specified checks
        --add-checks-path path1,path2,etc
                                     A directory containing additional out-of-tree checks to run

Output options:
    -d, --debug                      Lots of output
    -f, --format TYPE                Specify output formats. Default is text
        --css-file CSSFile           Specify CSS to use for HTML output
    -i, --ignore-config IGNOREFILE   Use configuration to ignore warnings
    -I, --interactive-ignore         Interactively ignore warnings
    -l, --[no-]combine-locations     Combine warning locations (Default)
        --[no-]highlights            Highlight user input in report
        --[no-]color                 Use ANSI colors in report (Default)
    -m, --routes                     Report controller information
        --message-limit LENGTH       Limit message length in HTML report
        --[no-]pager                 Use pager for output to terminal (Default)
        --table-width WIDTH          Limit table width in text report
    -o, --output FILE                Specify files for output. Defaults to stdout. Multiple '-o's allowed
        --[no-]separate-models       Warn on each model without attr_accessible (Default)
        --[no-]summary               Only output summary of warnings
        --absolute-paths             Output absolute file paths in reports
        --github-repo USER/REPO[/PATH][@REF]
                                     Output links to GitHub in markdown and HTML reports using specified repo
        --text-fields field1,field2,etc.
                                     Specify fields for text report format
    -w, --confidence-level LEVEL     Set minimal confidence level (1 - 3)
        --compare FILE               Compare the results of a previous Brakeman scan (only JSON is supported)

Configuration files:
    -c, --config-file FILE           Use specified configuration file
    -C, --create-config [FILE]       Output configuration file based on options
        --allow-check-paths-in-config
                                     Allow loading checks from configuration file (Unsafe)

    -k, --checks                     List all available vulnerability checks
        --optional-checks            List optional checks
    -v, --version                    Show Brakeman version
        --force-scan                 Scan application even if Rails is not detected
    -h, --help                       Display this message
Let’s move to the next step.
```
```md
Run the Scanner
As we have learned in DevSecOps Gospel, we would like to store the content in a JSON file. We are using the tee command here to show the output and store it in a file simultaneously.

brakeman -f json | tee result.json

Brakeman ran successfully, and it found multiple security issues.

Six issues with confidence level Medium
Eleven issues with confidence level High
It seems we have found quite a few issues; you can ignore issues using the brakeman.ignore file.

You will find fingerprints in the scan output.

Please create this brakeman.ignore file.

cat > brakeman.ignore <<EOF
{
    "ignored_warnings": [
        {
          "fingerprint": "febb21e45b226bb6bcdc23031091394a3ed80c76357f66b1f348844a7626f4df",
          "note": "ignore XSS"
        }
    ]
}
EOF

Let’s re-run the scanner.

brakeman -f json -i brakeman.ignore | tee result.json

You will see the issues are reduced because we ignored the Cross-Site Scripting (XSS) vulnerability.
```
```md
Secrets Scanning Using Detect-Secrets
Download the source code
We will do all the exercises locally first in DevSecOps-Box, so let’s start the exercise.

First, we need to download the source code of the project from our git repository.

git clone https://gitlab.practical-devsecops.training/pdso/django.nv webapp

Command Output
Cloning into 'webapp'...
warning: redirecting to https://gitlab.practical-devsecops.training/pdso/django.nv.git/
remote: Enumerating objects: 254, done.
remote: Counting objects: 100% (26/26), done.
remote: Compressing objects: 100% (26/26), done.
remote: Total 254 (delta 14), reused 0 (delta 0), pack-reused 228
Receiving objects: 100% (254/254), 1.04 MiB | 21.22 MiB/s, done.
Resolving deltas: 100% (100/100), done.
Let’s cd into the application so we can scan the app.

cd webapp

We are now in the webapp directory

Let’s move to the next step.
```
```md
Install detect-secrets
detect-secrets is a tool that designed with the enterprise client in mind: providing a backwards compatible to preventing new secrets from entering the code base and etc.

You can find more details about the project on this link.

Let’s install the detect-secrets tool on the system to scan for the secrets in our code.

pip3 install detect-secrets==1.4.0

Command Output
Collecting detect-secrets==1.4.0
  Downloading detect_secrets-1.4.0-py3-none-any.whl (116 kB)
     |████████████████████████████████| 116 kB 40.0 MB/s 
Requirement already satisfied: pyyaml in /usr/local/lib/python3.8/dist-packages (from detect-secrets==1.4.0) (6.0)
Requirement already satisfied: requests in /usr/local/lib/python3.8/dist-packages (from detect-secrets==1.4.0) (2.28.2)
Requirement already satisfied: charset-normalizer<4,>=2 in /usr/local/lib/python3.8/dist-packages (from requests->detect-secrets==1.4.0) (3.0.1)
Requirement already satisfied: idna<4,>=2.5 in /usr/lib/python3/dist-packages (from requests->detect-secrets==1.4.0) (2.8)
Requirement already satisfied: urllib3<1.27,>=1.21.1 in /usr/local/lib/python3.8/dist-packages (from requests->detect-secrets==1.4.0) (1.26.14)
Requirement already satisfied: certifi>=2017.4.17 in /usr/lib/python3/dist-packages (from requests->detect-secrets==1.4.0) (2019.11.28)
Installing collected packages: detect-secrets
Successfully installed detect-secrets-1.4.0
Let’s explore what options detect-secrets provides us.

detect-secrets --help

Command Output
usage: detect-secrets [-h] [-v] [--version] [-C <path>] [-c NUM_CORES] {scan,audit} ...

positional arguments:
  {scan,audit}
    scan                Creates a baseline by scanning a repository for secrets.
    audit               Manually assesses a baseline to determine validity of secrets found.

optional arguments:
  -h, --help            show this help message and exit
  -v, --verbose         Verbose mode.
  --version             Display version information.
  -C <path>             Run as if detect-secrets was started in <path>, rather than in the current working directory.
  -c NUM_CORES, --cores NUM_CORES
                        Specify the number of cores to use for parallel processing. Defaults to using the max cores on the current host.
Let’s move to the next step.
```
```md
Run the Scanner
As we have learned in the DevSecOps Gospel, we should save the output in the machine-readable format. JSON, CSV, XML formats can be parsed by the machines easily.

Let’s run the detect-secrets command with the scan argument to find any secrets in the current directory.

detect-secrets scan .

Command Output
{
{
  "version": "1.4.0",
  "plugins_used": [
    {
      "name": "ArtifactoryDetector"
    },

...[SNIP]...

      {
        "type": "Secret Keyword",
        "filename": "taskManager/settings.py",
        "hashed_secret": "877e30c29747a7ef645f02be73bd691e8a386e82",
        "is_verified": false,
        "line_number": 28
      }
    ]
  },
  "generated_at": "2023-02-25T03:20:55Z"
}
Besides, we can try to check if we are scanning the correct directory.

Let’s back to the previous path.

cd ..
ls

We are in the / directory. Now, our target, based on step1, is scanning webapp directory.

Let’s scan by specifying the webapp directory.

detect-secrets scan /webapp

Command Output
{
{
  "version": "1.4.0",
  "plugins_used": [
    {
      "name": "ArtifactoryDetector"
    },

...[SNIP]...

      {
        "type": "Secret Keyword",
        "filename": "taskManager/settings.py",
        "hashed_secret": "877e30c29747a7ef645f02be73bd691e8a386e82",
        "is_verified": false,
        "line_number": 28
      }
    ]
  },
  "generated_at": "2023-02-25T03:20:55Z"
}
As you can see, the output is still the same.

The process outlined above is consistent. We aim to guide you through the step-by-step process of how to scan a specific directory, whether you use . when you’re in the current directory or / when you’re targeting a specific directory.

It seems detect-secrets’s default output format is JSON format. Let’s store the output/results in a file.

detect-secrets scan . > secrets-output.json

We can also mark the issues/findings as false-positives in the output file using the audit command. You can accept an issue as a real issue with the letter y for yes and n for no.

Go ahead and mark the issues appropriately.

echo "q\n" | detect-secrets audit secrets-output.json

The echo “q\n” above is just STDIN to automatically quit or exit from the detect-secrets itself.

Command Output
Secret:      1 of 10
Filename:    fixtures/users.json
Secret Type: Secret Keyword
----------
4:    "pk": 1,
5:    "fields": {
6:      "first_name" : "",
7:      "last_name" : "",
8:      "username" : "admin",
9:      "password": "md5$oAKvI66ce0Xq$a5c1836db3d6dedff5deca630a358d8b",
10:      "is_superuser" : true,
11:      "email" : "admin@tm.com",
12:      "is_staff" : true,
13:      "is_active" : true
14:    }
----------
Is this a secret that should be committed to this repository? (y)es, (n)o, (s)kip, (q)uit: Quitting...
Did you notice it’s much more efficient than TruffleHog?
```
```md
Secrets Scanning with Talisman
Install Talisman
Talisman is a tool that installs a hook to your repository to ensure that potential secrets or sensitive information do not leave the developer’s workstation.
It validates the outgoing changeset for things that look suspicious - such as potential SSH keys, authorization tokens, private keys, etc.

Source: Talisman Repository.

This tool looks great, however it is far from perfect.

Here’s why.

Pre-commit/Pre-push hooks are configured only on a developer’s workstation.
If a developer has administrator access, they can easily disable these checks.
There’s no way to enforce it across the organization.
Please prefer to use secrets scanning tools as part of the CI/CD pipeline because of the above reasons. Embedding secret scanning tools provide the best of both worlds, i.e., enforcement and visibility.

Name	Value
Gitlab URL	https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/django-nv/
Username	root
Password	pdso-training
We will do all the exercises locally in DevSecOps-Box, and before installing the tool, we need to understand what is a git hook.

Note

Git hooks are scripts that run before or after git events like commit, push, rebase, etc., and are run locally. You can see a list of git hooks here.

There are two types of git hooks supported by Talisman.

pre-commit
pre-push
In this exercise, we will use pre-commit hook to prevent our developers from adding a sensitive file to our git repository.

You can install Talisman with the pre-commit hook, using the following command.

curl --silent https://raw.githubusercontent.com/thoughtworks/talisman/v1.32.0/global_install_scripts/install.bash > /tmp/install_talisman.bash && /bin/bash /tmp/install_talisman.bash pre-commit

Command Output
PLEASE CHOOSE WHERE YOU WISH TO SET TALISMAN_HOME VARIABLE AND talisman binary PATH (Enter option number): 
1) Set TALISMAN_HOME in ~/.bashrc        3) Set TALISMAN_HOME in ~/.profile
2) Set TALISMAN_HOME in ~/.bash_profile  4) I will set it later
You can choose option 1 to set TALISMAN_HOME in the bashrc. Why in bashrc.

Command Output
DO YOU WANT TO BE PROMPTED WHEN ADDING EXCEPTIONS TO .talismanrc FILE? 
Enter y to install in interactive mode. Press any key to continue without interactive mode (y/n):
You can click enter to continue.

Command Output
After the installation is complete, you will need to manually restart the terminal or source /root/.bashrc file
Press any key to continue ...
You can click press any key to continue.

Command Output
Setting up pre-commit hook in git template directory
No git template directory is configured. Let's add one.
(this will override any system git templates and modify your git config file)

Git template directory: (/root/.git-template) 
You can click enter to continue.

Command Output
Setting up template pre-commit hook
'/root/.git-template/hooks/pre-commit' -> '/root/.talisman/bin/talisman_hook_script'
Talisman template hook successfully installed.

Setting up talisman hook recursively in git repos
Please enter root directory to search for git repos (Default: /root): 
You can click enter to continue and the installation will be completed.

We have successfully installed the Talisman tool globally in our DevSecOps Box, so if there is any new repository that we clone or init, it will use Talisman as git hooks.

Let’s move to the next step.
```
```md
Download the source code
After installing the Talisman tool, let’s clone our code from Gitlab.

git clone http://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/django-nv.git webapp

After cloning the repository, you will be prompted for Username and Password. You can use the following credentials.

Name	Value
Username	root
Password	pdso-training
The cloning process will be completed successfully.

Let’s cd into the application so we can scan the app.

cd webapp

To see how Talisman works, we will try to add a sensitive file to this repository so that Talisman will prevent us from committing it.

Let’s copy SSH Private Key into the current directory.

cp ~/.ssh/id_rsa .

Let’s check the files in the current directory.

ls -l

Command Output
total 40
-rw-r--r-- 1 root root  537 Jan 17 03:50 Dockerfile
-rw-r--r-- 1 root root  127 Jan 17 03:50 README.md
-rw-r--r-- 1 root root  154 Jan 17 03:50 bandit.ignore
drwxr-xr-x 2 root root 4096 Jan 17 03:50 fixtures
-rw------- 1 root root 1704 Jan 17 03:50 id_rsa
-rw-r--r-- 1 root root  689 Jan 17 03:50 manage.py
-rw-r--r-- 1 root root 1579 Jan 17 03:50 package.json
-rw-r--r-- 1 root root   12 Jan 17 03:50 requirements.txt
-rw-r--r-- 1 root root  114 Jan 17 03:50 run.sh
drwxr-xr-x 5 root root 4096 Jan 17 03:50 taskManager
You will see that id_rsa file is present in the directory.

Now, let’s add the file to the repository.

git add id_rsa

git commit -m "Add private key"

Command Output
*** Please tell me who you are.

Run

  git config --global user.email "you@example.com"
  git config --global user.name "Your Name"

to set your account's default identity.
Omit --global to set the identity only in this repository.

fatal: unable to auto-detect email address (got 'root@devsecops-box-krqItici.(none)')
The above error appears because we haven’t added any git identity on the system. Let’s follow the provided suggestions to configure the git.

git config --global user.email "student@practical-devsecops.com"

git config --global user.name "DevSecOps Student"

Once done, you can re-execute the git commit command to see Talisman preventing us from committing the Private SSH key.

git commit -m "Add private key"

Command Output
Talisman Scan: 3 / 3 <--------------------------------------------------------------------------------------------------------> 100.00%  

Talisman Report:
+--------+----------------------------------------------------+----------+
|  FILE  |                       ERRORS                       | SEVERITY |
+--------+----------------------------------------------------+----------+
| id_rsa | The file name "id_rsa" failed                      | high     |
|        | checks against the pattern                         |          |
|        | ^.+_rsa$                                           |          |
+--------+----------------------------------------------------+----------+
| id_rsa | Expected file to not contain                       | high     |
|        | base64 encoded texts such as:                      |          |
|        | MIIEpAIBAAKCAQEAwNpZcFqfOVT0Q486JHr7kQO7HGD2zsG... |          |
+--------+----------------------------------------------------+----------+
| id_rsa | Expected file to not contain                       | high     |
|        | base64 encoded texts such as:                      |          |
|        | OUM633CIM0UYSqRTaOIRAi+HowZ5W0/8ZKW3okMQTQNjjY7... |          |
+--------+----------------------------------------------------+----------+
| id_rsa | Expected file to not contain                       | high     |
|        | base64 encoded texts such as:                      |          |
|        | dAimRGlgm1ycE/ucKXShiH9iSSeIrtkUiHWxvRssxSjdT0l... |          |

... [SNIP] ...

| id_rsa | Expected file to not contain                       | high     |
|        | base64 encoded texts such as:                      |          |
|        | KaPWM6APqjLRpeb+EGgCQQTtQpqFwnZa2lXqlospGdGu1dy... |          |
+--------+----------------------------------------------------+----------+
| id_rsa | Expected file to not contain                       | high     |
|        | base64 encoded texts such as:                      |          |
|        | 8uGfLRjRWwnS+q+erFI1lVp0HT1Mpu+Fj8dK2p1SBKDw7c1... |          |
+--------+----------------------------------------------------+----------+
| id_rsa | Expected file to not contain                       | high     |
|        | base64 encoded texts such as:                      |          |
|        | ySD5bQKBgQDJt2QT2oxFP/UYU1OQO1Vu38U82Yw5F7XCFE5... |          |
+--------+----------------------------------------------------+----------+
| id_rsa | Expected file to not contain                       | high     |
|        | base64 encoded texts such as:                      |          |
|        | sCXOqlg8u/Hat/tsAI4WNjyjXSLLdCvF85d6Sx2/pbPMfNB... |          |
+--------+----------------------------------------------------+----------+
| id_rsa | Expected file to not contain                       | high     |
|        | base64 encoded texts such as:                      |          |
|        | 1PSx8Bw50vWJVZFlHSC0thG7psyrflbOoLBKdwfvBkrfSrp... |          |
+--------+----------------------------------------------------+----------+
| id_rsa | Potential secret pattern : BEGIN RSA               | high     |
 MIIEpAIBAAKCAQEAwNpZ              |          |
       |          |Q486JHr7kQO7HGD2zsGC4iodBxc6bdU/S1zn
|        | OUM633CIM0UYSqRTaOIRAi+How...                      |          |
+--------+----------------------------------------------------+----------+

If you are absolutely sure that you want to ignore the above files from talisman detectors, consider pasting the following format in .talismanrc file in the project root

fileignoreconfig:
- filename: id_rsa
  checksum: 416122aefc170aef929dbbd1e01626e9066d36f7cfd83c00e38d38b5b03a7c69
version: ""

Talisman done in 877.602834ms
As expected, Talisman found a sensitive file (SSH private key) and stopped the file from being committed in to the repo. You can review supported rules and other configurations here.

Let’s move to the next step.
```
```md
Run the Scanner
You can also use Talisman as a command-line utility instead of using it as a git hook.

Let’s explore what options Talisman provides us so we can use it as a command-line utility.

source ~/.bashrc
talisman -h

Command Output
Usage of /root/.talisman/bin/talisman_linux_amd64:
  -c, --checksum string          checksum calculator calculates checksum and suggests .talismanrc entry
  -d, --debug                    enable debug mode (warning: very verbose)
  -g, --githook string           either pre-push or pre-commit (default "pre-push")
  -^, --ignoreHistory            scanner scans all files on current head, will not scan through git commit history
  -i, --interactive              interactively update talismanrc (only makes sense with -g/--githook)
  -l, --loglevel string          set log level for talisman (allowed values: error|info|warn|debug, default: error) (default "error")
  -p, --pattern string           pattern (glob-like) of files to scan (ignores githooks)
  -f, --profile                  profile cpu and memory usage of talisman
  -r, --reportDirectory string   directory where the scan report will be stored (default "talisman_report")
  -s, --scan                     scanner scans the git commit history for potential secrets
  -w, --scanWithHtml             generate html report (**Make sure you have installed talisman_html_report to use this, as mentioned in talisman Readme**)
  -v, --version                  show current version of talisman
pflag: help requested
Now it’s time to run the scanner to find any secrets in our code.

talisman --scan

Command Output
Talisman Fetch Blobs: 1 / 1 <-------------------------------------------------------------------------------------------------> 100.00%  


d8888b. db    db d8b   db d8b   db d888888b d8b   db  d888b    .d8888.  .o88b.  .d8b.  d8b   db
88  `8D 88    88 888o  88 888o  88   `88'   888o  88 88' Y8b   88'  YP d8P  Y8 d8' `8b 888o  88
88oobY' 88    88 88V8o 88 88V8o 88    88    88V8o 88 88        `8bo.   8P      88ooo88 88V8o 88
88`8b   88    88 88 V8o88 88 V8o88    88    88 V8o88 88  ooo     `Y8b. 8b      88~~~88 88 V8o88
88 `88. 88b  d88 88  V888 88  V888   .88.   88  V888 88. ~8~   db   8D Y8b  d8 88   88 88  V888 db db
88   YD ~Y8888P' VP   V8P VP   V8P Y888888P VP   V8P  Y888P    `8888Y'  `Y88P' YP   YP VP   V8P VP VP


Talisman Scan: 444 / 444 <----------------------------------------------------------------------------------------------------> 100.00%  

Please check 'talisman_report/talisman_reports/data' folder for the talisman scan report

Talisman done in 818.318823ms
The expected output number might change since it’s dynamic.

By default, Talisman outputs the results in the JSON format. You can check out the result file at talisman_reports/data/report.json,

cat talisman_report/talisman_reports/data/report.json | jq

Let’s move to the conclusion.
```
```md
Conclusion
In this exercise, we learned about Talisman, a powerful secrets scanning tool that helps prevent sensitive information from being accidentally committed to source code repositories. Here are the key takeaways:

Purpose and Function
- Talisman is designed to detect potential secrets and sensitive information before they leave a developer’s workstation  
- It works by scanning for suspicious patterns like SSH keys, authorization tokens, private keys, etc.  

Implementation Methods
- Can be used as a git hook (pre-commit or pre-push)  
- Can also be used as a standalone command-line utility  
- Supports both interactive and automated scanning modes  

Limitations and Best Practices
- Git hooks are only configured on individual workstations  
- Developers with admin access can bypass the checks  
- Therefore, it’s recommended to:  
  - Use Talisman as part of a broader security strategy  
  - Implement secrets scanning in CI/CD pipelines for better enforcement  
  - Combine both local hooks and pipeline scanning for maximum coverage  

Key Features
- Prevents accidental commits of sensitive files  
- Provides detailed reports of findings  
- Supports custom configurations via `.talismanrc`  
- Offers both JSON and HTML reporting options  

Integration Benefits
- Early detection of security issues  
- Prevents sensitive data leaks before they happen  
- Helps maintain security best practices across development teams  
- Provides immediate feedback to developers  

Remember that while Talisman is an excellent tool for local security checks, it should be part of a comprehensive security approach that includes CI/CD pipeline scanning and other security measures.
```
```md
Secrets Scanning Using Semgrep
Download the Source Code
We will do all the exercises locally first in DevSecOps-Box, so let’s start the exercise.

First, we need to download the source code of the project from our git repository.

git clone https://gitlab.practical-devsecops.training/pdso/django.nv webapp

Command Output
Cloning into 'webapp'...
warning: redirecting to https://gitlab.practical-devsecops.training/pdso/django.nv.git/
remote: Enumerating objects: 302, done.
remote: Total 302 (delta 0), reused 0 (delta 0), pack-reused 302
Receiving objects: 100% (302/302), 1.05 MiB | 37.21 MiB/s, done.
Resolving deltas: 100% (122/122), done.

Let’s cd into the application so we can scan the app.

cd webapp

We are now in the webapp directory.

Let’s move to the next step.
```
```md
Install Semgrep
Semgrep is a fast, open-source, static analysis engine for finding bugs, detecting vulnerabilities in third-party dependencies, and enforcing code standards. Semgrep analyzes code locally on your computer or in your build environment: code is never uploaded.
You can find more details about the project at https://github.com/returntocorp/semgrep.

Let’s install the Semgrep tool on the system to perform static analysis.

pip3 install semgrep==1.124.0

Command Output
Collecting semgrep==1.124.0
  Downloading semgrep-1.124.0-cp39.cp310.cp311.py39.py310.py311-none-musllinux_1_0_x86_64.manylinux2014_x86_64.whl.metadata (1.8 kB)
Collecting attrs>=21.3 (from semgrep==1.124.0)
  Downloading attrs-25.3.0-py3-none-any.whl.metadata (10 kB)
Collecting boltons~=21.0 (from semgrep==1.124.0)
  Downloading boltons-21.0.0-py2.py3-none-any.whl.metadata (1.5 kB)
Collecting click-option-group~=0.5 (from semgrep==1.124.0)
  Downloading click_option_group-0.5.7-py3-none-any.whl.metadata (5.8 kB)
Collecting click~=8.1.8 (from semgrep==1.124.0)
  Downloading click-8.1.8-py3-none-any.whl.metadata (2.3 kB)
[[....SNIPPED....]]
    Uninstalling importlib-metadata-4.6.4:
      Successfully uninstalled importlib-metadata-4.6.4
Successfully installed attrs-25.3.0 boltons-21.0.0 bracex-2.5.post1 click-8.1.8 click-option-group-0.5.7 colorama-0.4.6 defusedxml-0.7.1 deprecated-1.2.18 exceptiongroup-1.2.2 face-24.0.0 glom-22.1.0 googleapis-common-protos-1.70.0 importlib-metadata-7.1.0 jsonschema-4.24.0 jsonschema-specifications-2025.4.1 markdown-it-py-3.0.0 mdurl-0.1.2 opentelemetry-api-1.25.0 opentelemetry-exporter-otlp-proto-common-1.25.0 opentelemetry-exporter-otlp-proto-http-1.25.0 opentelemetry-instrumentation-0.46b0 opentelemetry-instrumentation-requests-0.46b0 opentelemetry-proto-1.25.0 opentelemetry-sdk-1.25.0 opentelemetry-semantic-conventions-0.46b0 opentelemetry-util-http-0.46b0 peewee-3.18.1 protobuf-4.25.8 pygments-2.19.1 referencing-0.36.2 rich-13.5.3 rpds-py-0.25.1 ruamel.yaml-0.18.14 ruamel.yaml.clib-0.2.12 semgrep-1.124.0 tomli-2.0.2 typing-extensions-4.14.0 wcmatch-8.5.2 wrapt-1.17.2
We have successfully installed semgrep, let’s explore its functionality now.

semgrep --help

Command Output
Usage: pysemgrep [OPTIONS] COMMAND [ARGS]...

  To get started quickly, run `semgrep scan --config auto`

  Run `semgrep SUBCOMMAND --help` for more information on each subcommand

  If no subcommand is passed, will run `scan` subcommand by default

Options:
  -h, --help  Show this message and exit.

Commands:
  ci                   The recommended way to run semgrep in CI
  install-semgrep-pro  Install the Semgrep Pro Engine
  login                Obtain and save credentials for semgrep.dev
  logout               Remove locally stored credentials to semgrep.dev
  lsp                  Start the Semgrep LSP server (useful for IDEs)
  publish              Upload rule to semgrep.dev
  scan                 Run semgrep rules on files
  show                 Show various information about Semgrep

For more information about the Semgrep scan feature, you can visit this link: Semgrep Scan Command Options or try inputting “semgrep scan –help”.

Notes
If you’re encountering the same error with every command inputted, like the following:

Command Output
/usr/lib/python3/dist-packages/requests/__init__.py:89: RequestsDependencyWarning: urllib3 (1.26.18) or chardet (3.0.4) doesn't match a supported version! warnings.warn("urllib3 ({}) or chardet ({}) doesn't match a supported
It’s likely due to an outdated version of the Python requests library. You can update it using the following command:

pip3 install --upgrade requests

Let’s move to the next step.
```
```md
Secret Scanning Using Semgrep
Let’s try and run secret scanning using Semgrep.

Passwords, API keys, GitHub personal access tokens (PATs), etc are some examples of secrets.

Semgrep has a custom config which has a bunch of secrets detection rules.

You can explore the secret scanning rules from semgrep by visiting this link

Note
Right now, there are 269 rules in the config. It’s important to know that this number will likely increase in the future as more rules are added to the semgrep.

The output number may vary as it is dynamic.

Let’s run semgrep using secrets detection config against our project.

semgrep --config "p/secrets"

Command Output

┌──── ○○○ ────┐
│ Semgrep CLI │
└─────────────┘

METRICS: Using configs from the Registry (like --config=p/ci) reports pseudonymous rule metrics to semgrep.dev.
To disable Registry rule metrics, use "--metrics=off".
When using configs only from local files (like --config=xyz.yml) metrics are sent only when the user is logged in.

More information: https://semgrep.dev/docs/metrics


Scanning 140 files (only git-tracked) with 52 Code rules:

  CODE RULES

  Language      Rules   Files          Origin      Rules
 ─────────────────────────────        ───────────────────
  <multilang>      36     140          Community      52
  js                5      19
  python            1      14
  yaml              1       1


  SUPPLY CHAIN RULES

  💎 Sign in with `semgrep login` and run
     `semgrep ci` to find dependency vulnerabilities and
     advanced cross-file findings.


  PROGRESS

  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 100% 0:00:00


┌─────────────────┐
│ 2 Code Findings │
└─────────────────┘

    taskManager/settings.py
   ❯❯❱ generic.secrets.security.detected-aws-access-key-id-value.detected-aws-access-key-id-value
          AWS Access Key ID Value detected. This is a sensitive credential and should not be hardcoded
          here. Instead, read this value from an environment variable or keep it in a separate, private
          file.
          Details: https://sg.run/GeD1

           19┆ AWS_ACCESS_KEY_ID = 'AKIAYVP4CIPPERUVIFXG'

   ❯❯❱ generic.secrets.security.detected-aws-secret-access-key.detected-aws-secret-access-key
          AWS Secret Access Key detected
          Details: https://sg.run/Bk39

           20┆ AWS_SECRET_ACCESS_KEY = 'Zt2U1h267eViPnuSA+JO5ABhiu4T7XUMSZ+Y2Oth'



┌──────────────┐
│ Scan Summary │
└──────────────┘
✅ Scan completed successfully.
 • Findings: 2 (2 blocking)
 • Rules run: 43
 • Targets scanned: 140
 • Parsed lines: ~100.0%
 • Scan skipped:
   ◦ Files matching .semgrepignore patterns: 8
 • Scan was limited to files tracked by git
 • For a detailed list of skipped files and lines, run semgrep with the --verbose flag
Ran 43 rules on 140 files: 2 findings.
💎 Missed out on 217 pro rules since you aren't logged in!
⚡ Supercharge Semgrep OSS when you create a free account at https://sg.run/rules.

You can see that 43 rules were run against 140 files and 2 secrets were enumerated.

Why does it say 43 rules were run, but the config has 269 rules?

If you had looked closely at the config page you would have noticed that only a small subsection of rules is run when you are not logged in.

illustration

Let’s login and rerun the scan to see if semgrep enumerate more secrets.

semgrep login

You will be presented with a URL in the output that you will need to copy and paste in a new tab. This facilitates the Semgrep login verification. You can choose to sign in with GitHub or GitLab to complete the login. You will be asked to validate your client token post login. Kindly go ahead and do so.

Note
Do not use your corporate GitHub or GitLab account to login to Semgrep.

Command Output
Login enables additional proprietary Semgrep Registry rules and running custom policies from Semgrep Cloud Platform.
Opening login at: https://semgrep.dev/login?cli-token=273fdcd0-382e-4734-8d3c-bcf715ea99ae&docker=False&gha=False

Once you've logged in, return here and you'll be ready to start using new Semgrep rules.
Saved login token

        76eb4af5827c0d3ad352148e0afd398cbb6302a0a4c1bf6d0d83945a3079f46b

in /root/.semgrep/settings.yml.
Note: You can always generate more tokens at https://semgrep.dev/orgs/-/settings/tokens
If everything goes correctly then you should see something like the above output.

Note
The token you see in the above output will be different for you.

From the output we can deduce that our token is saved at this /root/.semgrep/settings.yml location. We can validate it using cat /root/.semgrep/settings.yml command.

Finally, let’s run secret scan again.

semgrep --config "p/secrets"

Command Output

┌──── ○○○ ────┐
│ Semgrep CLI │
└─────────────┘


Scanning 140 files (only git-tracked) with 269 Code rules:

  CODE RULES

  Language      Rules   Files          Origin      Rules
 ─────────────────────────────        ───────────────────
  <multilang>      36     140          Pro rules     217
  js               37      19          Community      52
  python           58      14
  yaml              1       1


  SUPPLY CHAIN RULES

  💎 Run `semgrep ci` to find dependency
     vulnerabilities and advanced cross-file findings.


  PROGRESS

  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━╸ 100% 0:00:05
Warning: 3 timeout error(s) in taskManager/static/taskManager/js/jquery.js when running the following rules:
[javascript.lang.hardcoded.strings.detected-private-key.detected-private-key, javascript.superagent.hardcoded-basic-
token.hardcoded-basic-token, javascript.superagent.hardcoded-bearer-token.hardcoded-bearer-token]
Semgrep stopped running rules on taskManager/static/taskManager/js/jquery.js after 3 timeout error(s). See
`--timeout-threshold` for more info.


┌─────────────────┐
│ 2 Code Findings │
└─────────────────┘

    taskManager/settings.py
   ❯❯❱ generic.secrets.security.detected-aws-access-key-id-value.detected-aws-access-key-id-value
          AWS Access Key ID Value detected. This is a sensitive credential and should not be hardcoded
          here. Instead, read this value from an environment variable or keep it in a separate, private
          file.
          Details: https://sg.run/GeD1

           19┆ AWS_ACCESS_KEY_ID = 'AKIAYVP4CIPPERUVIFXG'

   ❯❯❱ generic.secrets.security.detected-aws-secret-access-key.detected-aws-secret-access-key
          AWS Secret Access Key detected
          Details: https://sg.run/Bk39

           20┆ AWS_SECRET_ACCESS_KEY = 'Zt2U1h267eViPnuSA+JO5ABhiu4T7XUMSZ+Y2Oth'



┌──────────────┐
│ Scan Summary │
└──────────────┘
✅ Scan completed successfully.
 • Findings: 2 (2 blocking)
 • Rules run: 132
 • Targets scanned: 140
 • Parsed lines: ~100.0%
 • Scan skipped:
   ◦ Files matching .semgrepignore patterns: 8
 • Scan was limited to files tracked by git
 • For a detailed list of skipped files and lines, run semgrep with the --verbose flag
Ran 132 rules on 140 files: 2 findings.

If you observe the output closely you will see that more rules are run but in our case the secrets found are still 2.

So keep in mind to never store API keys, access keys and passwords in source code.

In this case AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY are declared and set in the source code. Secrets need to be managed efficiently, and securely using a secret managed system.
```
```md
Secrets Scanning Using Gitleaks
Installation of the Tools
GitLeaks is a powerful tool designed to audit Git repositories for secrets and confidential information. Let’s go through the steps required to install it on your system.

Follow the command below:

wget https://github.com/gitleaks/gitleaks/releases/download/v8.18.1/gitleaks_8.18.1_linux_x64.tar.gz && tar -xvzf gitleaks_8.18.1_linux_x64.tar.gz && mv gitleaks /usr/local/bin

Here’s what each part of the command does:

Download GitLeaks Binary:  
Use wget to download the GitLeaks binary from the specified URL: https://github.com/gitleaks/gitleaks/releases/download/v8.18.1/gitleaks_8.18.1_linux_x64.tar.gz.

Extract the Downloaded Tar File:  
Extract the contents of the downloaded tar.gz file using the command tar -xvzf gitleaks_8.18.1_linux_x64.tar.gz.

Move the GitLeaks Binary:  
Move the extracted GitLeaks binary to the /usr/local/bin directory using the command mv gitleaks /usr/local/bin. This step makes GitLeaks accessible as a system-wide command from the command line.

You have now successfully set up GitLeaks on your system, and it is ready to be used for auditing your Git repositories for sensitive or confidential information.

Let’s move to the next step.
```
```md
Download the source code
We will do all the exercises locally first in DevSecOps-Box, so let’s start the exercise.

First, we need to download the source code of the project from our git repository.

git clone https://gitlab.practical-devsecops.training/pdso/django.nv webapp

Command Output
Cloning into 'webapp'...
warning: redirecting to https://gitlab.practical-devsecops.training/pdso/django.nv.git/
remote: Enumerating objects: 254, done.
remote: Counting objects: 100% (26/26), done.
remote: Compressing objects: 100% (26/26), done.
remote: Total 254 (delta 14), reused 0 (delta 0), pack-reused 228
Receiving objects: 100% (254/254), 1.04 MiB | 21.22 MiB/s, done.
Resolving deltas: 100% (100/100), done.

This command fetches the Django repository and stores it in a directory named webapp.

After cloning, navigate to the webapp directory with the following command:

cd webapp

We are now in the webapp directory.

Let’s move to the next step.
```
```md
Run the Scanner
Before running GitLeaks, let’s explore the available options and commands by executing the following:

gitleaks --help

Command Output
Gitleaks scans code, past or present, for secrets

Usage:
  gitleaks [command]

Available Commands:
  completion  generate the autocompletion script for the specified shell
  detect      detect secrets in code
  help        Help about any command
  protect     protect secrets in code
  version     display gitleaks version

Flags:
  -b, --baseline-path string       path to baseline with issues that can be ignored
  -c, --config string              config file path
                                   order of precedence:
                                   1. --config/-c
                                   2. env var GITLEAKS_CONFIG
                                   3. (--source/-s)/.gitleaks.toml
                                   If none of the three options are used, then gitleaks will use the default config
      --exit-code int              exit code when leaks have been encountered (default 1)
  -h, --help                       help for gitleaks
      --ignore-gitleaks-allow      ignore gitleaks:allow comments
  -l, --log-level string           log level (trace, debug, info, warn, error, fatal) (default "info")
      --max-target-megabytes int   files larger than this will be skipped
      --no-banner                  suppress banner
      --no-color                   turn off color for verbose output
      --redact uint[=100]          redact secrets from logs and stdout. To redact only parts of the secret just apply a percent value from 0..100. For example --redact=20 (default 100%)
  -f, --report-format string       output format (json, csv, junit, sarif) (default "json")
  -r, --report-path string         report file
  -s, --source string              path to source (default ".")
  -v, --verbose                    show verbose output from scan

This command provides a comprehensive list of available options, helping you understand how to utilize GitLeaks effectively.

To perform the GitLeaks scan, execute the following command:

gitleaks detect .

Command Output
    ○
    │╲
    │ ○
    ○ ░
    ░    gitleaks

5:19AM INF 22 commits scanned.
5:19AM INF scan completed in 309ms
5:19AM WRN leaks found: 4

Upon running this command, GitLeaks will diligently analyze the Django project’s Git history and codebase, searching for potential secrets or sensitive information. As the scan progresses, you’ll see informative output, including the number of commits scanned and any detected leaks.

Step 3: Run GitLeaks with Output File
To save the scan results in an output file, execute the following command:

gitleaks detect . --report-path gitleaks-output.txt

This command scans the Django project for sensitive information and saves the results in a file named “gitleaks-output.txt.”

To access and review the scan results, you can use the following commands:
- Use the ls command to list the files in your current directory. This allows you to verify the existence of the “gitleaks-output.txt” file:

ls

Command Output
Dockerfile  bandit.ignore  gitleaks-output.txt  package.json      run.sh
README.md   fixtures       manage.py            requirements.txt  taskManager

To delve deeper into the findings and understand the potential sensitive information detected, you can utilize the cat command:

cat gitleaks-output.txt

Command Output
[
 {
  "Description": "AWS",
  "StartLine": 19,
  "EndLine": 19,
  "StartColumn": 22,
  "EndColumn": 41,
  "Match": "AKIAYVP4CIPPERUVIFXG",
  "Secret": "AKIAYVP4CIPPERUVIFXG",
  "File": "taskManager/settings.py",
  "SymlinkFile": "",
  "Commit": "749ce09a83b1f6f25d9d2b7e6aad61cbfe16d2cf",
  "Entropy": 3.6464393,
  "Author": "Muhammad Yuga Nugraha",
  "Email": "yuga@Muhammads-MacBook-Pro-2.local",
  "Date": "2023-03-05T05:39:14Z",
  "Message": "feat: upgrade to Django 3",
  "Tags": [],
  "RuleID": "aws-access-token",
  "Fingerprint": "749ce09a83b1f6f25d9d2b7e6aad61cbfe16d2cf:taskManager/settings.py:aws-access-token:19"
 },

These commands enable you to gain insights into the scan’s findings, making it easier to review and address any identified issues.

Let’s move to the next step.
```
```md
Advanced Scanning
Sometimes, you may want to share or store the scan results while redacting sensitive information. You can use GitLeaks to redact secrets in the output by specifying the --redact option. For example, to redact 50% of the secret information, use the following command:

gitleaks detect . --report-path gitleaks-redact50.txt --redact=50

This command scans the current directory, saves the results to a file named gitleaks-redact50.txt, and redacts 50% of the secret information, making it safer to share.

Command Output
  "Match": "API_TOKEN = 'xoxp-4797898847-479939...'",
  "Secret": "xoxp-4797898847-479939...",

With these steps, you’ve learned about GitLeaks available options, initiated a scan of the Django repository, executed a scan with the results saved in gitleaks-output.txt, and explored advanced options for more tailored scanning. This exercise provides you with a hands-on experience of using GitLeaks to scan a Git repository for sensitive information, an essential step in software component analysis.
```
```md
How To Fix the Issues Reported by Bandit
In this scenario, you will learn how to fix issues reported by the Bandit tool in python source code.

You will do the following in this activity.

1. **Download the source code from the git repository.**

2. **Install the Bandit tool.**

3. **Run the SAST scan on the code.**

4. **Fix the issues found by Bandit.**
```
```md
Download the source code
We will do all the exercises locally first in DevSecOps-Box, so let’s start the exercise.

First, we need to download the source code of the project from our git repository.

git clone https://gitlab.practical-devsecops.training/pdso/dvpa-api webapp

Command Output
Cloning into 'webapp'...
warning: redirecting to https://gitlab.practical-devsecops.training/pdso/dvpa-api.git/
remote: Enumerating objects: 77, done.
remote: Total 77 (delta 0), reused 0 (delta 0), pack-reused 77
Unpacking objects: 100% (77/77), done.

Let’s cd into the application so we can scan the app.

cd webapp

We are now in the webapp directory.

Let’s move to the next step.
```
```md
Install Bandit
The Bandit is a tool designed to find common security issues in Python code.

To do this Bandit, processes each file, builds an AST, and runs appropriate plugins against the AST nodes. Once Bandit has finished scanning all the files it generates a report.

Bandit was originally developed within the OpenStack Security Project and later rehomed to PyCQA.

You can find more details about the project at https://github.com/PyCQA/bandit.

Let’s install the bandit scanner on the system to perform static analysis.

pip3 install bandit==1.7.4

Command Output
Collecting bandit==1.7.4
  Downloading bandit-1.7.4-py3-none-any.whl (118 kB)
     |████████████████████████████████| 118 kB 28 kB/s 
Collecting GitPython>=1.0.1
  Downloading GitPython-3.1.27-py3-none-any.whl (181 kB)
     |████████████████████████████████| 181 kB 45.3 MB/s 
Collecting stevedore>=1.20.0
  Downloading stevedore-4.0.0-py3-none-any.whl (49 kB)
     |████████████████████████████████| 49 kB 4.7 MB/s 
Collecting PyYAML>=5.3.1
  Downloading PyYAML-6.0-cp38-cp38-manylinux_2_5_x86_64.manylinux1_x86_64.manylinux_2_12_x86_64.manylinux2010_x86_64.whl (701 kB)
     |████████████████████████████████| 701 kB 39.7 MB/s 
Collecting gitdb<5,>=4.0.1
  Downloading gitdb-4.0.9-py3-none-any.whl (63 kB)
     |████████████████████████████████| 63 kB 928 kB/s 
Collecting pbr!=2.1.0,>=2.0.0
  Downloading pbr-5.9.0-py2.py3-none-any.whl (112 kB)
     |████████████████████████████████| 112 kB 38.9 MB/s 
Collecting smmap<6,>=3.0.1
  Downloading smmap-5.0.0-py3-none-any.whl (24 kB)
Installing collected packages: smmap, gitdb, GitPython, pbr, stevedore, PyYAML, bandit
Successfully installed GitPython-3.1.27 PyYAML-6.0 bandit-1.7.4 gitdb-4.0.9 pbr-5.9.0 smmap-5.0.0 stevedore-4.0.0

We have successfully installed Bandit scanner. Let’s explore the functionality it provides us.

bandit --help

Command Output
usage: bandit [-h] [-r] [-a {file,vuln}] [-n CONTEXT_LINES] [-c CONFIG_FILE] [-p PROFILE] [-t TESTS] [-s SKIPS]
              [-l | --severity-level {all,low,medium,high}] [-i | --confidence-level {all,low,medium,high}]
              [-f {csv,custom,html,json,screen,txt,xml,yaml}] [--msg-template MSG_TEMPLATE] [-o [OUTPUT_FILE]] [-v]
              [-d] [-q] [--ignore-nosec] [-x EXCLUDED_PATHS] [-b BASELINE] [--ini INI_PATH] [--exit-zero] [--version]
              [targets [targets ...]]

Bandit - a Python source code security analyzer

positional arguments:
  targets               source file(s) or directory(s) to be tested

optional arguments:
  -h, --help            show this help message and exit
  -r, --recursive       find and process files in subdirectories
  -a {file,vuln}, --aggregate {file,vuln}
                        aggregate output by vulnerability (default) or by filename
  -n CONTEXT_LINES, --number CONTEXT_LINES
                        maximum number of code lines to output for each issue
  -c CONFIG_FILE, --configfile CONFIG_FILE
                        optional config file to use for selecting plugins and overriding defaults
  -p PROFILE, --profile PROFILE
                        profile to use (defaults to executing all tests)
  -t TESTS, --tests TESTS
                        comma-separated list of test IDs to run
  -s SKIPS, --skip SKIPS
                        comma-separated list of test IDs to skip
  -l, --level           report only issues of a given severity level or higher (-l for LOW, -ll for MEDIUM, -lll for
                        HIGH)
  --severity-level {all,low,medium,high}
                        report only issues of a given severity level or higher. "all" and "low" are likely to produce
                        the same results, but it is possible for rules to be undefined which will not be listed in
                        "low".
  -i, --confidence      report only issues of a given confidence level or higher (-i for LOW, -ii for MEDIUM, -iii for
                        HIGH)
  --confidence-level {all,low,medium,high}
                        report only issues of a given confidence level or higher. "all" and "low" are likely to
                        produce the same results, but it is possible for rules to be undefined which will not be
                        listed in "low".
  -f {csv,custom,html,json,screen,txt,xml,yaml}, --format {csv,custom,html,json,screen,txt,xml,yaml}
                        specify output format
  --msg-template MSG_TEMPLATE
                        specify output message template (only usable with --format custom), see CUSTOM FORMAT section
                        for list of available values
  -o [OUTPUT_FILE], --output [OUTPUT_FILE]
                        write report to filename
  -v, --verbose         output extra information like excluded and included files
  -d, --debug           turn on debug mode
  -q, --quiet, --silent
                        only show output in the case of an error
  --ignore-nosec        do not skip lines with # nosec comments
  -x EXCLUDED_PATHS, --exclude EXCLUDED_PATHS
                        comma-separated list of paths (glob patterns supported) to exclude from scan (note that these
                        are in addition to the excluded paths provided in the config file) (default:
                        .svn,CVS,.bzr,.hg,.git,__pycache__,.tox,.eggs,*.egg)
  -b BASELINE, --baseline BASELINE
                        path of a baseline report to compare against (only JSON-formatted files are accepted)
  --ini INI_PATH        path to a .bandit file that supplies command line arguments
  --exit-zero           exit with 0, even with results found
  --version             show program's version number and exit

...[SNIP]...

Let’s move to the next step.
```
```md
Run the scanner
Let’s scan our source code by executing the following command:

bandit -r .

Command Output
[main]  INFO    profile include tests: None
[main]  INFO    profile exclude tests: None
[main]  INFO    cli include tests: None
[main]  INFO    cli exclude tests: None
[main]  INFO    running on Python 3.10.12
Run started:2025-04-12 04:49:16.524122

Test results:
>> Issue: [B324:hashlib] Use of weak MD4, MD5, or SHA1 hash for security. Consider usedforsecurity=False
   Severity: High   Confidence: High
   CWE: CWE-327 (https://cwe.mitre.org/data/definitions/327.html)
   Location: ./flaskblog/auth.py:13:23
   More Info: https://bandit.readthedocs.io/en/1.7.4/plugins/b324_hashlib.html
12          cur = db.connection.cursor()
13          hashsed_password = hashlib.md5(password.encode()).hexdigest()
14          cur.execute(f"SELECT * FROM users WHERE email='{username}' AND password='{hashsed_password}'")

--------------------------------------------------
>> Issue: [B608:hardcoded_sql_expressions] Possible SQL injection vector through string-based query construction.
   Severity: Medium   Confidence: Medium
   CWE: CWE-89 (https://cwe.mitre.org/data/definitions/89.html)
   Location: ./flaskblog/auth.py:14:16
   More Info: https://bandit.readthedocs.io/en/1.7.4/plugins/b608_hardcoded_sql_expressions.html
13          hashsed_password = hashlib.md5(password.encode()).hexdigest()
14          cur.execute(f"SELECT * FROM users WHERE email='{username}' AND password='{hashsed_password}'")
15          user = cur.fetchone()

...[SNIP]...

>> Issue: [B105:hardcoded_password_string] Possible hardcoded password: 'secret'
   Severity: Low   Confidence: Medium
   CWE: CWE-259 (https://cwe.mitre.org/data/definitions/259.html)
   Location: ./flaskblog/config.py:13:11
   More Info: https://bandit.readthedocs.io/en/1.7.4/plugins/b105_hardcoded_password_string.html
12      username = 'admin'
13      password = 'secret'
14
15      # Disqus Configuration
16      disqus_shortname = 'blogpythonlearning'  # please change this.

--------------------------------------------------

Code scanned:
        Total lines of code: 603
        Total lines skipped (#nosec): 0

Run metrics:
        Total issues (by severity):
                Undefined: 0
                Low: 2
                Medium: 8
                High: 3
        Total issues (by confidence):
                Undefined: 0
                Low: 0
                Medium: 9
                High: 4
Files skipped (0):

We got 13 issues in total (by confidence). Among these, there are two(2) hardcoded password strings, an insecure hash function issue, insecure deserialization, and 8 SQL Injections. We must fix all these issues in order to avoid security breaches in our organization.

Let’s move to the next step.
```
```md
Fixing Insecure Deserialization
Python yaml library has a known vulnerability around YAML deserialization. We can search for this known security issue on the CVE website.

For more details, please visit CVE-2020-1747. As mentioned before, our code is vulnerable to Deserialization Attacks.

If you recall, one of the findings in the previous step was unsafe YAML load, and the following was the code snippet for the same.

>> Issue: [B506:yaml_load] Use of unsafe yaml load. Allows instantiation of arbitrary objects. Consider yaml.safe_load().
   Severity: Medium   Confidence: High
   CWE: CWE-20 (https://cwe.mitre.org/data/definitions/20.html)
   Location: ./flaskblog/blogapi/dashboard.py:248:35
   More Info: https://bandit.readthedocs.io/en/1.7.4/plugins/b506_yaml_load.html
247                 elif export_format == "yaml":
248                     import_post_data = yaml.load(import_data)
249

Let’s verify this issue exists by opening up the flaskblog/blogapi/dashboard.py file using any text editor like vim or nano. Ensure the security issue exists at line 248 and the program uses the insecure yaml.load function. To fix this issue, we need to replace yaml.load with a safe alternative **yaml.safe_load**.

Let’s run the scanner once again.

bandit -r .

Command Output
[main]  INFO    profile include tests: None
[main]  INFO    profile exclude tests: None
[main]  INFO    cli include tests: None
[main]  INFO    cli exclude tests: None
[main]  INFO    running on Python 3.10.12
Run started:2025-04-12 04:50:38.338594

Test results:
>> Issue: [B324:hashlib] Use of weak MD4, MD5, or SHA1 hash for security. Consider usedforsecurity=False
   Severity: High   Confidence: High
   CWE: CWE-327 (https://cwe.mitre.org/data/definitions/327.html)
   Location: ./flaskblog/auth.py:13:23
   More Info: https://bandit.readthedocs.io/en/1.7.4/plugins/b324_hashlib.html
12          cur = db.connection.cursor()
13          hashsed_password = hashlib.md5(password.encode()).hexdigest()
14          cur.execute(f"SELECT * FROM users WHERE email='{username}' AND password='{hashsed_password}'")

...[SNIP]...

Code scanned:
        Total lines of code: 603
        Total lines skipped (#nosec): 0

Run metrics:
        Total issues (by severity):
                Undefined: 0
                Low: 2
                Medium: 7
                High: 3
        Total issues (by confidence):
                Undefined: 0
                Low: 0
                Medium: 9
                High: 3
Files skipped (0):

As you can see, there is no yaml.load issue in the output. The total High issue count (by confidence) has decreased from 4 to 3.

Let’s move to the next step.
```
````md
Fixing SQL Injection
Bandit also informed us that there are 7 possible SQL Injection issues in this application. We can fix these issues in various ways, but the best way to fix SQL Injection issues is **Parameterized queries**, also known as Parameter Binding.

Command Output
[main]  INFO    profile include tests: None
[main]  INFO    profile exclude tests: None
[main]  INFO    cli include tests: None
[main]  INFO    cli exclude tests: None
[main]  INFO    running on Python 3.10.12
Run started:2025-04-12 04:50:38.338594

Test results:
>> Issue: [B324:hashlib] Use of weak MD4, MD5, or SHA1 hash for security. Consider usedforsecurity=False
   Severity: High   Confidence: High
   CWE: CWE-327 (https://cwe.mitre.org/data/definitions/327.html)
   Location: ./flaskblog/auth.py:13:23
   More Info: https://bandit.readthedocs.io/en/1.7.4/plugins/b324_hashlib.html
12          cur = db.connection.cursor()
13          hashsed_password = hashlib.md5(password.encode()).hexdigest()
14          cur.execute(f"SELECT * FROM users WHERE email='{username}' AND password='{hashsed_password}'")

--------------------------------------------------
>> Issue: [B608:hardcoded_sql_expressions] Possible SQL injection vector through string-based query construction.
   Severity: Medium   Confidence: Medium
   CWE: CWE-89 (https://cwe.mitre.org/data/definitions/89.html)
   Location: ./flaskblog/auth.py:14:16
   More Info: https://bandit.readthedocs.io/en/1.7.4/plugins/b608_hardcoded_sql_expressions.html
13          hashsed_password = hashlib.md5(password.encode()).hexdigest()
14          cur.execute(f"SELECT * FROM users WHERE email='{username}' AND password='{hashsed_password}'")
15          user = cur.fetchone()

...[SNIP]...

---

### Fix Example
First, we will try to fix the SQL Injection issue present in the `flaskblog/auth.py` file at line number 14.  
As you can see in the following code snippet, there is a SQL injection issue in the `check_auth` function.

```python
def check_auth(username, password):
   """ This function is called to check if a username / password
       combination is valid.
   """
   cur = db.connection.cursor()
   hashsed_password = hashlib.md5(password.encode()).hexdigest()
   cur.execute(f"SELECT * FROM users WHERE email='{username}' AND password='{hashsed_password}'")
   user = cur.fetchone()

   if user is None:
      return False

   session["is_logged_in"] = True
   session["id"] = user.get("id")
   session["email"] = user.get("email")
   session["full_name"] = user.get("full_name")

   return user
````

The `cur.execute` function call will execute the SQL query on the database.
It takes the username and password as possible inputs to the SQL query.

You can also verify this behavior dynamically by reproducing SQL query errors.

You can learn more about SQL Injection [here](https://owasp.org/www-community/attacks/SQL_Injection).

---

### Fixed Code

You can replace line number 14 with the following code.
This code will fix the SQL Injection issue:

```python
cur.execute("SELECT * FROM users WHERE email=%s AND password=%s", [username, hashsed_password])
```

---

### Re-run Bandit

bandit -r .

Command Output (after fix)
\[main]  INFO    running on Python 3.10.12
Run started:2025-04-12 04:57:12.810803

Test results:

> > Issue: \[B324\:hashlib] Use of weak MD4, MD5, or SHA1 hash for security. Consider usedforsecurity=False
> > Severity: High   Confidence: High
> > CWE: CWE-327
> > Location: ./flaskblog/auth.py:13:23

...\[SNIP]...

> > Issue: \[B608\:hardcoded\_sql\_expressions] Possible SQL injection vector through string-based query construction.
> > Severity: Medium   Confidence: Medium
> > CWE: CWE-89
> > Location: ./flaskblog/blogapi/dashboard.py:85:20

...\[SNIP]...

Code scanned:
Total lines of code: 603
Total lines skipped (#nosec): 0

Run metrics:
Total issues (by severity):
Low: 2
Medium: 6
High: 3
Total issues (by confidence):
Medium: 8
High: 3

---

✅ Looking at the Bandit scan output after applying the fix, the issue in `auth.py` is gone.
The **total Medium issue count (by confidence) has decreased from 9 to 8**.

Let’s move to the next step.

```
```
````md
# Static Analysis Using Gosec

## Download the source code
We will do all the exercises locally first in DevSecOps-Box, so let’s start the exercise.

First, we need to download the source code of the project from our git repository.

```bash
git clone https://gitlab.practical-devsecops.training/pdso/golang.git webapp
````

Let’s cd into the application so we can scan the app.

```bash
cd webapp
```

We are now in the webapp directory.

Let’s move to the next step.

```
```

````md
# Install gosec

GoSec inspects source code for security problems by scanning the Go AST, also can be configured to only run a subset of rules, to exclude certain file paths, and produce reports in different formats.

You can find more details about the project at https://github.com/securego/gosec

---

Our system doesn’t have Golang installed, so we need to install it with the following command:

```bash
curl -s https://dl.google.com/go/go1.17.4.linux-amd64.tar.gz | tar xvz -C /usr/local
````

We also need to configure **GOROOT**, **GOPATH** and **PATH** variables so golang can find binaries and third party libraries.

```bash
export GOROOT=/usr/local/go
export GOPATH=$HOME/go
export PATH=$GOPATH/bin:$GOROOT/bin:$PATH
```

---

Let’s install gosec on the system to perform static analysis.

```bash
curl -sfL https://raw.githubusercontent.com/securego/gosec/master/install.sh | sh -s -- -b $(go env GOPATH)/bin v2.4.0
```

### Command Output

```
securego/gosec info checking GitHub for tag 'v2.4.0'
securego/gosec info found version: 2.4.0 for v2.4.0/linux/amd64
securego/gosec info installed /root/go/bin/gosec
```

Running curl and piping it to a shell is a security issue. How could we improve it?

You guessed it right. Use native tools to install it. For e.g., **go get**.

```bash
go get -u github.com/securego/gosec/v2/cmd/gosec
```

### Command Output

```
go: downloading github.com/securego/gosec/v2 v2.9.4
go: downloading github.com/securego/gosec v0.0.0-20200401082031-e946c8c39989
go: downloading gopkg.in/yaml.v2 v2.4.0
go: downloading github.com/nbutton23/zxcvbn-go v0.0.0-20210217022336-fa2cb2858354
go: downloading github.com/google/uuid v1.3.0
go: downloading golang.org/x/tools v0.1.8
go: downloading github.com/gookit/color v1.5.0
go: downloading golang.org/x/sys v0.0.0-20211019181941-9d821ace8654
go: downloading github.com/xo/terminfo v0.0.0-20210125001918-ca9a967f8778
go: downloading golang.org/x/sys v0.0.0-20211205182925-97ca703d548d
go: downloading golang.org/x/xerrors v0.0.0-20200804184101-5ec99f83aff1
go: downloading golang.org/x/mod v0.5.1
go get: installing executables with 'go get' in module mode is deprecated.
        Use 'go install pkg@version' instead.
        For more information, see https://golang.org/doc/go-get-install-deprecation
        or run 'go help get' or 'go help install'.
```

---

We have successfully installed gosec, let’s explore the functionality it provides us.

```bash
gosec --help
```

### Command Output

```
gosec - Golang security checker

gosec analyzes Go source code to look for common programming mistakes that
can lead to security problems.

VERSION: dev
GIT TAG: 
BUILD DATE: 

USAGE:

        # Check a single package
        $ gosec $GOPATH/src/github.com/example/project

        # Check all packages under the current directory and save results in
        # json format.
        $ gosec -fmt=json -out=results.json ./...

        # Run a specific set of rules (by default all rules will be run):
        $ gosec -include=G101,G203,G401  ./...

        # Run all rules except the provided
        $ gosec -exclude=G101 $GOPATH/src/github.com/example/project/...


OPTIONS:

  -color
        Prints the text format report with colorization when it goes in the stdout (default true)
  -conf string
        Path to optional config file
  -confidence string
        Filter out the issues with a lower confidence than the given value. Valid options are: low, medium, high (default "low")
  -exclude value
        Comma separated list of rules IDs to exclude. (see rule list)
  -exclude-dir value
        Exclude folder from scan (can be specified multiple times)
  -exclude-generated
        Exclude generated files
  -fmt string
        Set output format. Valid options are: json, yaml, csv, junit-xml, html, sonarqube, golint, sarif or text (default "text")
  -include string
        Comma separated list of rules IDs to include. (see rule list)
  -log string
        Log messages to file rather than stderr
  -no-fail
        Do not fail the scanning, even if issues were found
  -nosec
        Ignores #nosec comments when set
  -nosec-tag string
        Set an alternative string for #nosec. Some examples: #dontanalyze, #falsepositive
  -out string
        Set output file for results
  -quiet
        Only show output when errors are found
  -severity string
        Filter out the issues with a lower severity than the given value. Valid options are: low, medium, high (default "low")
  -show-ignored
        If enabled, ignored issues are printed
  -sort
        Sort issues by severity (default true)
  -stdout
        Stdout the results as well as write it in the output file
  -tags string
        Comma separated list of build tags
  -tests
        Scan tests files
  -track-suppressions
        Output suppression information, including its kind and justification
  -verbose string
        Overrides the output format when stdout the results while saving them in the output file.
        Valid options are: json, yaml, csv, junit-xml, html, sonarqube, golint, sarif or text
  -version
        Print version and quit with exit code 0


RULES:

        G101: Look for hardcoded credentials
        G102: Bind to all interfaces
        G103: Audit the use of unsafe block
        G104: Audit errors not checked
        G106: Audit the use of ssh.InsecureIgnoreHostKey function
        G107: Url provided to HTTP request as taint input
        G108: Profiling endpoint is automatically exposed
        G109: Converting strconv.Atoi result to int32/int16
        G110: Detect io.Copy instead of io.CopyN when decompression
        G201: SQL query construction using format string
        G202: SQL query construction using string concatenation
        G203: Use of unescaped data in HTML templates
        G204: Audit use of command execution
        G301: Poor file permissions used when creating a directory
        G302: Poor file permissions used when creation file or using chmod
        G303: Creating tempfile using a predictable path
        G304: File path provided as taint input
        G305: File path traversal when extracting zip archive
        G306: Poor file permissions used when writing to a file
        G307: Unsafe defer call of a method returning an error
        G401: Detect the usage of DES, RC4, MD5 or SHA1
        G402: Look for bad TLS connection settings
        G403: Ensure minimum RSA key length of 2048 bits
        G404: Insecure random number source (rand)
        G501: Import blocklist: crypto/md5
        G502: Import blocklist: crypto/des
        G503: Import blocklist: crypto/rc4
        G504: Import blocklist: net/http/cgi
        G505: Import blocklist: crypto/sha1
        G601: Implicit memory aliasing in RangeStmt
```

---

Let’s move to the next step.

```
```
````md
# Run the Scanner

We can perform static analysis on golang code using the following command.

```bash
gosec ./...
````

### Command Output

```
[gosec] 2021/12/09 13:47:33 Including rules: default
[gosec] 2021/12/09 13:47:33 Excluding rules: default
[gosec] 2021/12/09 13:47:33 Import directory: /webapp/util/config
[gosec] 2021/12/09 13:47:34 Checking package: config
[gosec] 2021/12/09 13:47:34 Checking file: /webapp/util/config/config.go
[gosec] 2021/12/09 13:47:34 Import directory: /webapp/user/session
[gosec] 2021/12/09 13:47:34 Checking package: session
[gosec] 2021/12/09 13:47:34 Checking file: /webapp/user/session/session.go
[gosec] 2021/12/09 13:47:34 Import directory: /webapp/vulnerability/idor
[gosec] 2021/12/09 13:47:34 Checking package: idor
[gosec] 2021/12/09 13:47:34 Checking file: /webapp/vulnerability/idor/function.go
[gosec] 2021/12/09 13:47:34 Checking file: /webapp/vulnerability/idor/idor.go
[gosec] 2021/12/09 13:47:34 Import directory: /webapp/vulnerability/sqli
[gosec] 2021/12/09 13:47:35 Checking package: sqli
[gosec] 2021/12/09 13:47:35 Checking file: /webapp/vulnerability/sqli/function.go
[gosec] 2021/12/09 13:47:35 Checking file: /webapp/vulnerability/sqli/sqli.go
[gosec] 2021/12/09 13:47:35 Import directory: /webapp/vulnerability/xss
[gosec] 2021/12/09 13:47:35 Checking package: xss
[gosec] 2021/12/09 13:47:35 Checking file: /webapp/vulnerability/xss/function.go
[gosec] 2021/12/09 13:47:35 Checking file: /webapp/vulnerability/xss/xss.go
[gosec] 2021/12/09 13:47:35 Import directory: /webapp/setup

...[SNIP]...

[/webapp/vulnerability/sqli/function.go:37-40] - G201 (CWE-89): SQL string formatting (Confidence: HIGH, Severity: MEDIUM)
    36:
  > 37:         getProfileSql := fmt.Sprintf(`SELECT p.user_id, p.full_name, p.city, p.phone_number
  > 38:                                                                 FROM Profile as p,Users as u
  > 39:                                                                 where p.user_id = u.id
  > 40:                                                                 and u.id=%s`, uid) //here is the vulnerable query
    41:         rows, err := DB.Query(getProfileSql)



[/webapp/vulnerability/idor/idor.go:8] - G501 (CWE-327): Blocklisted import crypto/md5: weak cryptographic primitive (Confidence: HIGH, Severity: MEDIUM)
    7:  "net/http"
  > 8:  "crypto/md5"
    9:  "encoding/hex"



[/webapp/vulnerability/csa/csa.go:7] - G501 (CWE-327): Blocklisted import crypto/md5: weak cryptographic primitive (Confidence: HIGH, Severity: MEDIUM)
    6:  "net/http"
  > 7:  "crypto/md5"
    8:  "encoding/hex"



[/webapp/user/user.go:8] - G501 (CWE-327): Blocklisted import crypto/md5: weak cryptographic primitive (Confidence: HIGH, Severity: MEDIUM)
    7:  "strconv"
  > 8:  "crypto/md5"
    9:  "database/sql"



[/webapp/vulnerability/idor/idor.go:124] - G104 (CWE-703): Errors unhandled. (Confidence: HIGH, Severity: LOW)
    123:        p := NewProfile()
  > 124:        p.GetData(sid)
    125:



[/webapp/vulnerability/idor/idor.go:82] - G104 (CWE-703): Errors unhandled. (Confidence: HIGH, Severity: LOW)
    81:         p := NewProfile()
  > 82:         p.GetData(sid)
    83:



[/webapp/vulnerability/idor/idor.go:61] - G104 (CWE-703): Errors unhandled. (Confidence: HIGH, Severity: LOW)
    60:         p := NewProfile()
  > 61:         p.GetData(sid)
    62:



[/webapp/vulnerability/idor/idor.go:42] - G104 (CWE-703): Errors unhandled. (Confidence: HIGH, Severity: LOW)
    41:         p := NewProfile()
  > 42:         p.GetData(sid)
    43:



[/webapp/util/template.go:41] - G104 (CWE-703): Errors unhandled. (Confidence: HIGH, Severity: LOW)
    40:         template := template.Must(template.ParseGlob("templates/*"))
  > 41:         template.ExecuteTemplate(w, name, data)
    42: }



[/webapp/util/template.go:35] - G104 (CWE-703): Errors unhandled. (Confidence: HIGH, Severity: LOW)
    34:         }
  > 35:         w.Write(b)
    36: }



[/webapp/util/middleware/middleware.go:73] - G104 (CWE-703): Errors unhandled. (Confidence: HIGH, Severity: LOW)
    72:                         w.WriteHeader(http.StatusForbidden)
  > 73:                         w.Write([]byte("Forbidden"))
    74:                         log.Printf("sqlmap detect ")



Summary:
  Gosec  : dev
  Files  : 20
  Lines  : 1573
  Nosec  : 0
  Issues : 20
```

---

### Variable Scan Results

Your results might slightly vary because of the dynamic landscape of changing vulnerabilities, and security updates.

As you can see, we found 23 issues and there were many false positives. We can reduce the bloat and false positives by using custom rules, proper configurations and by using arguments such as `--severity high`.

```bash
gosec -exclude=G104 ./...
```

The above command will reduce the bloat but does it also removes real issues?

```
```
````md
# How To Embed Gosec Into GitLab

## A Simple CI/CD Pipeline

Let’s download the code using `git clone` in **DevSecOps Box**.

```bash
git clone https://gitlab.practical-devsecops.training/pdso/golang.git golang
cd golang
````

Rename git url to the new one.

```bash
git remote rename origin old-origin
git remote add origin git@gitlab-ce-kr6k1mdm:root/golang.git
```

Then, push the code into the repository and it will automatically create a new project if not exist.

```bash
git push -u origin --all
```

And enter the GitLab credentials.

| Name     | Value         |
| -------- | ------------- |
| Username | root          |
| Password | pdso-training |

---

## Pipeline Definition

Next, considering your DevOps team created a simple **CI pipeline** with the following contents:

```yaml
image: docker:20.10  # To run all jobs in this pipeline, use the latest docker image

services:
  - docker:dind       # To run all jobs in this pipeline, use a docker image that contains a docker daemon running inside (dind - docker in docker). Reference: https://forum.gitlab.com/t/why-services-docker-dind-is-needed-while-already-having-image-docker/43534

stages:
  - build
  - test
  - release
  - preprod
  - integration
  - prod

build:
  stage: build
  script:
    - echo "This is a build step"

test:
  stage: test
  script:
    - echo "This is a test step"

integration:
  stage: integration
  script:
    - echo "This is an integration step"
    - exit 1
  allow_failure: true # Even if the job fails, continue to the next stages

prod:
  stage: prod
  script:
    - echo "This is a deploy step."
  when: manual # Continuous Delivery
```

We have **four jobs** in this pipeline:

* `build` job
* `test` job
* `integration` job
* `prod` job

---

## Security Engineer Perspective

As a security engineer, I do not care much about what the DevOps team is doing as part of these jobs.

**Why?**
Imagine having to learn every build/testing tool used by your DevOps team — it will be a nightmare. Instead, **rely on the DevOps team for help.**

---

## Run the Pipeline

Let’s login into GitLab using the following details and execute this pipeline.

**GitLab CI/CD Machine**

| Name     | Value                                                                                                                                                                                                    |
| -------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| URL      | [https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/golang/-/blob/main/.gitlab-ci.yml](https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/golang/-/blob/main/.gitlab-ci.yml) |
| Username | root                                                                                                                                                                                                     |
| Password | pdso-training                                                                                                                                                                                            |

Next:

1. Create a CI/CD pipeline by **replacing the `.gitlab-ci.yml` file** content with the above CI script.

   * Click on the **Edit** button.
   * Replace the content (use **Control+A** and **Control+V**).

2. Save changes to the file using the **Commit changes** button.

---

## Verify the Pipeline Run

As soon as a change is made to the repository, the pipeline starts executing the jobs.

* You can see the results of this pipeline by visiting:
  👉 [https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/golang/pipelines](https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/golang/pipelines)

* Click on the appropriate job name to see the **output logs**.

---

✅ In the next step, we will **embed gosec in the CI/CD pipeline** and follow all the **best practices**.

```
```
````md
# Embed Gosec in CI/CD Pipeline

As discussed in the **Static Analysis using Gosec** exercise, we can embed Gosec in our CI/CD pipeline.  

⚠️ **Important Reminder:**  
It’s always best to **locally test a tool** before integrating it into the pipeline.  

- Troubleshooting locally is much easier than inside CI/CD.  
- Running the tool manually helps you become familiar with its **options and features**.  

---

## GitLab CI/CD Configuration

```yaml
image: docker:20.10  # To run all jobs in this pipeline, use the latest docker image

services:
  - docker:dind       # To run all jobs in this pipeline, use a docker image that contains a docker daemon running inside (dind - docker in docker). Reference: https://forum.gitlab.com/t/why-services-docker-dind-is-needed-while-already-having-image-docker/43534

stages:
  - build
  - test
  - release
  - preprod
  - integration
  - prod

build:
  stage: build
  script:
    - echo "This is a build step"

sast:
  stage: build
  script:
    - docker run --rm -v $(pwd):/src -w /src securego/gosec -fmt json -out gosec-output.json ./...
  artifacts:
    paths: [gosec-output.json]
    when: always

test:
  stage: test
  script:
    - echo "This is a test step"

integration:
  stage: integration
  script:
    - echo "This is an integration step"
    - exit 1
  allow_failure: true # Even if the job fails, continue to the next stages

prod:
  stage: prod
  script:
    - echo "This is a deploy step."
  when: manual # Continuous Delivery
````

---

## How It Works

* Any **change** to the repository will **kickstart the pipeline**.
* The **`sast` job** runs Gosec inside Docker and outputs results into a JSON file:

  ```
  gosec-output.json
  ```
* This artifact can be downloaded or analyzed by other jobs.

---

## Methods of Using Gosec

We discussed three main ways to use Gosec:

1. **Native Installation**

   * Directly install on the system.

2. **Package Manager or Binary**

   * Install via a package manager or use a prebuilt binary.

3. **Docker**

   * Run inside a container without worrying about dependencies.

✅ **Recommendation for CI/CD:**

* **Docker** is the most reliable and dependency-free approach.
* **Binary** is efficient and avoids extra dependencies.
* Ultimately, choose the method best suited to your environment.

---

## Verify the Results

You can check the pipeline at:
👉 [GitLab Pipeline Results](https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/golang/pipelines)

* Click on the appropriate **job name** to see the output.
* Notice that the **`sast` job’s output** is saved in `gosec-output.json`.

---

➡️ Let’s move to the next step.

```
```
````md
# Allow the Job Failure

⚠️ **Remember!**  
- Except for **DevSecOps-Box**, every other machine closes after **two hours**, even if you are mid-exercise.  
- After two hours, in case of a **404 error**, refresh the exercise page and click on **Start the Exercise** to continue working.  

---

## Why Allow Failure?
- In **DevSecOps Maturity Levels 1 and 2**, we **do not want to fail builds/jobs/scans**.  
- Reason: Security tools (like Gosec) often produce **many false positives**.  
- To prevent the pipeline from being blocked, use the **`allow_failure`** tag.  

This tag ensures that **even if the job fails**, the pipeline continues.

---

## Updated `sast` Job

```yaml
sast:
  stage: build
  script:
    - docker run --rm -v $(pwd):/src -w /src securego/gosec -fmt json -out gosec-output.json ./...
  artifacts:
    paths: [gosec-output.json]
    when: always
  allow_failure: true
````

---

## Full Pipeline with `allow_failure`

```yaml
image: docker:20.10  # To run all jobs in this pipeline, use the latest docker image

services:
  - docker:dind       # To run all jobs in this pipeline, use a docker image that contains a docker daemon running inside (dind - docker in docker). Reference: https://forum.gitlab.com/t/why-services-docker-dind-is-needed-while-already-having-image-docker/43534

stages:
  - build
  - test
  - release
  - preprod
  - integration
  - prod

sast:
  stage: build
  script:
    - docker run --rm -v $(pwd):/src -w /src securego/gosec -fmt json -out gosec-output.json ./...
  artifacts:
    paths: [gosec-output.json]
    when: always
  allow_failure: true

integration:
  stage: integration
  script:
    - echo "This is an integration step"
    - exit 1
  allow_failure: true # Even if the job fails, continue to the next stages

prod:
  stage: prod
  script:
    - echo "This is a deploy step."
  when: manual # Continuous Delivery
```

---

## Key Observation

* The **`sast` job fails** due to findings.
* But thanks to **`allow_failure: true`**, it **does not block** the rest of the pipeline.

---

## Verify Results

As discussed, any **change to the repo** triggers the pipeline.

You can view pipeline results at:
👉 [GitLab Pipeline Results](https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/golang/pipelines)

Click on the appropriate **job name** to see detailed output.

```
```
````md
# Static Analysis Using Semgrep

## Step 1: Download the Source Code
We will perform all the exercises locally in **DevSecOps-Box**. Let’s start by downloading the source code from the Git repository.

```bash
git clone https://gitlab.practical-devsecops.training/pdso/django.nv webapp
````

### Command Output

```
Cloning into 'webapp'...
warning: redirecting to https://gitlab.practical-devsecops.training/pdso/django.nv.git/
remote: Enumerating objects: 228, done.
remote: Total 228 (delta 0), reused 0 (delta 0), pack-reused 228
Receiving objects: 100% (228/228), 1.03 MiB | 1.04 MiB/s, done.
Resolving deltas: 100% (86/86), done.
```

---

## Step 2: Change Directory

Navigate into the newly cloned project directory:

```bash
cd webapp
```

Now we are inside the **webapp** directory.

---

✅ Next, we’ll proceed with installing and running **Semgrep** for static analysis.

```
```
````md
# Install Semgrep

## Step 1: Introduction
**Semgrep** is a fast, open-source, static analysis tool that helps enforce code standards and detect vulnerabilities early. Unlike regexes or abstract syntax tree traversals, Semgrep rules look like the code you’re scanning for, making them simpler to write and understand.

More details about the project: [Semgrep GitHub](https://github.com/returntocorp/semgrep).

---

## Step 2: Install Semgrep
Install a specific version of **Semgrep (1.30.0)** with `pip3`:

```bash
pip3 install semgrep==1.30.0
````

### Command Output

```
Collecting semgrep==1.30.0
  Downloading semgrep-0.108.0-cp37.cp38.cp39.py37.py38.py39-none-any.whl (25.2 MB)
     |████████████████████████████████| 25.2 MB 5.0 MB/s 
Collecting colorama~=0.4.0
  Downloading colorama-0.4.5-py2.py3-none-any.whl (16 kB)
Collecting python-lsp-jsonrpc~=1.0.0
  Downloading python_lsp_jsonrpc-1.0.0-py3-none-any.whl (8.5 kB)

...[SNIP]...

Successfully installed attrs-21.4.0 boltons-21.0.0 bracex-2.3.post1 click-8.1.3 click-option-group-0.5.3 colorama-0.4.5 defusedxml-0.7.1 face-20.1.1 glom-22.1.0 importlib-resources-5.9.0 jsonschema-4.9.1 packaging-21.3 peewee-3.15.1 pkgutil-resolve-name-1.3.10 pyparsing-3.0.9 pyrsistent-0.18.1 python-lsp-jsonrpc-1.0.0 ruamel.yaml-0.17.21 ruamel.yaml.clib-0.2.6 semgrep-0.108.0 tqdm-4.64.0 typing-extensions-4.3.0 ujson-5.4.0 urllib3-1.26.11 wcmatch-8.4 zipp-3.8.1
```

✅ Semgrep has been installed successfully.

---

## Step 3: Explore Semgrep Functionality

Check available options:

```bash
semgrep --help
```

### Command Output

```
Usage: pysemgrep [OPTIONS] COMMAND [ARGS]...

  To get started quickly, run `semgrep scan --config auto`

  Run `semgrep SUBCOMMAND --help` for more information on each subcommand

  If no subcommand is passed, will run `scan` subcommand by default

Options:
  -h, --help  Show this message and exit.

Commands:
  ci                   The recommended way to run semgrep in CI
  install-semgrep-pro  Install the Semgrep Pro Engine
  login                Obtain and save credentials for semgrep.dev
  logout               Remove locally stored credentials to semgrep.dev
  lsp                  [EXPERIMENTAL] Start the Semgrep LSP server
  publish              Upload rule to semgrep.dev
  scan                 Run semgrep rules on files
  shouldafound         Report a false negative in this project.
```

---

## Notes

If you encounter warnings such as:

```
/usr/lib/python3/dist-packages/requests/__init__.py:89: RequestsDependencyWarning: urllib3 (1.26.18) or chardet (3.0.4) doesn't match a supported version!
```

This means the **requests** library is outdated. Fix it with:

```bash
pip3 install --upgrade requests
```

---

➡️ Next, we’ll run **Semgrep** on the project source code.

```
```
````md
# Basic Usage of Semgrep

Semgrep parameters can be grouped into **four categories**:

| Category              | Description                                                                 |
|-----------------------|-----------------------------------------------------------------------------|
| **positional arguments** | The target file or directory to scan                                       |
| **optional arguments**   | Options like include/exclude files or directories                         |
| **config**               | Configuration/rule set to scan the code                                   |
| **output**               | Format and destination of scan results                                    |

⚠️ *The number of findings may vary since results are dynamic.*

---

## Example 1: Scan for `os.system` Calls

We’ll scan the `webapp` source code for `os.system(...)` usage.

```bash
semgrep --lang python -e "os.system(...)" .
````

### Command Output

```
Scanning 50 files.
  100%|█████████████████████████████████████████████████████████████████████████████████████████████████████|50/50 tasks

Findings:

  taskManager/misc.py 
         33┆ os.system(
         34┆     "mv " +
         35┆     uploaded_file.temporary_file_path() +
         36┆     " " +
         37┆     "%s/%s" %
         38┆     (upload_dir_path,
         39┆      title))

Some files were skipped or only partially analyzed.
  Scan was limited to files tracked by git.

Ran 1 rule on 50 files: 1 finding.
```

* `--lang python` → specifies the programming language
* `-e "os.system(...)"` → defines the code pattern to search
* `.` → sets the **target directory**

---

## Example 2: Export Output to JSON

We can save the results in JSON format for further processing:

```bash
semgrep --lang python -e "os.system(...)" . --json | jq
```

### Command Output (trimmed)

```json
{
  "errors": [],
  "paths": {
    "scanned": [
      "manage.py",
      "taskManager/__init__.py",
      "taskManager/forms.py",
      "...",
      "taskManager/misc.py",
      "taskManager/models.py",
      "taskManager/settings.py"
    ]
  },
  "results": [
    {
      "check_id": "-",
      "path": "taskManager/misc.py",
      "start": { "line": 33, "col": 5 },
      "end": { "line": 39, "col": 17 },
      "extra": {
        "message": "os.system(...)",
        "severity": "ERROR",
        "lines": "    os.system(\n        \"mv \" +\n        uploaded_file.temporary_file_path() +\n        \" \" +\n        \"%s/%s\" %\n        (upload_dir_path,\n         title))"
      }
    }
  ],
  "version": "0.108.0"
}
```

---

## Example 3: Scan a Specific File with `--include`

We can restrict the scan to only the `settings.py` file and check if `DEBUG=True` exists.

```bash
semgrep --lang python -e "DEBUG =True" --include settings.py .
```

### Command Output

```
Scanning 1 file.

Findings:

  taskManager/settings.py 
         28┆ DEBUG = True

Some files were skipped or only partially analyzed.
  Scan was limited to files tracked by git.
  Scan skipped: 49 files not matching --include patterns
  For a full list of skipped files, run semgrep with the --verbose flag.

Ran 1 rule on 1 file: 1 finding.
```

---

✅ With these commands, you now know how to:

* Scan an entire project
* Export results in JSON format
* Limit scans to specific files

```
```
````md
# Challenges

## Tasks

### #1  
**Task:**  
Scan all declarations of variables in `webapp` source code.  
There is no specific variable name to scan for, so you can scan for **all variables**.

---

### ✅ Answer
```bash
semgrep --lang python -e '$X = $Y' .
````

```
```
````md
# How To Write Custom Rule in Semgrep

## Step 1: Download the source code

We will do all the exercises locally first in **DevSecOps-Box**, so let’s start the exercise.

Run the following command to download the source code of the project from our git repository:

```bash
git clone https://gitlab.practical-devsecops.training/pdso/django.nv webapp
````

---

## Step 2: Change into the application directory

Navigate into the `webapp` directory so we can scan the app:

```bash
cd webapp
```

---

✅ We are now in the `webapp` directory.

Let’s move to the next step.

```
```
````md
# Install Semgrep

Semgrep is a fast, open-source, static analysis tool that excels at expressing code standards — without complicated queries — and surfacing bugs early in the development flow. Precise rules look like the code you’re searching; no more traversing abstract syntax trees or wrestling with regexes.

📖 More details about the project: [Semgrep GitHub](https://github.com/returntocorp/semgrep)

---

## Step 1: Install Semgrep

Run the following command:

```bash
pip3 install semgrep==1.30.0
````

### Command Output

```
Collecting semgrep==1.30.0
  Downloading semgrep-0.81.0-cp36.cp37.cp38.cp39.py36.py37.py38.py39-none-any.whl (21.9 MB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 21.9/21.9 MB 2.7 MB/s eta 0:00:00
Collecting click>=8.0.1
  Downloading click-8.0.3-py3-none-any.whl (97 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 97.5/97.5 KB 24.7 MB/s eta 0:00:00
Collecting ruamel.yaml<0.18,>=0.16.0
  Downloading ruamel.yaml-0.17.20-py3-none-any.whl (109 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 109.5/109.5 KB 25.1 MB/s eta 0:00:00

...[SNIP]...

Collecting ruamel.yaml.clib>=0.2.6
  Downloading ruamel.yaml.clib-0.2.6-cp38-cp38-manylinux1_x86_64.whl (570 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 570.4/570.4 KB 65.7 MB/s eta 0:00:00
Collecting zipp>=3.1.0
  Downloading zipp-3.7.0-py3-none-any.whl (5.3 kB)
Building wheels for collected packages: peewee
  Building wheel for peewee (setup.py) ... done
  Created wheel for peewee: filename=peewee-3.14.8-py3-none-any.whl size=131197 sha256=46bfa9e7ce1c6bd8e6d6a16439533daf61e094a0ba149ecc1e96934b6c1f7c69
  Stored in directory: /root/.cache/pip/wheels/b2/c6/33/b4a2a2a0c867ba928b7abd0b8d16f0096dc30322aa5829b39f
Successfully built peewee
Installing collected packages: peewee, zipp, tqdm, ruamel.yaml.clib, pyrsistent, colorama, click, bracex, attrs, wcmatch, ruamel.yaml, importlib-resources, click-option-group, jsonschema, semgrep
Successfully installed attrs-21.4.0 bracex-2.2.1 click-8.0.3 click-option-group-0.5.3 colorama-0.4.4 importlib-resources-5.4.0 jsonschema-4.4.0 peewee-3.14.8 pyrsistent-0.18.1 ruamel.yaml-0.17.20 ruamel.yaml.clib-0.2.6 semgrep-0.81.0 tqdm-4.62.3 wcmatch-8.3 zipp-3.7.0
WARNING: Running pip as the 'root' user can result in broken permissions and conflicting behaviour with the system package manager. It is recommended to use a virtual environment instead: https://pip.pypa.io/warnings/venv
```

---

## Step 2: Explore Semgrep Options

Run the following command:

```bash
semgrep --help
```

### Command Output

```
Usage: pysemgrep [OPTIONS] COMMAND [ARGS]...

  To get started quickly, run `semgrep scan --config auto`

  Run `semgrep SUBCOMMAND --help` for more information on each subcommand

  If no subcommand is passed, will run `scan` subcommand by default

Options:
  -h, --help  Show this message and exit.

Commands:
  ci                   The recommended way to run semgrep in CI
  install-semgrep-pro  Install the Semgrep Pro Engine
  login                Obtain and save credentials for semgrep.dev
  logout               Remove locally stored credentials to semgrep.dev
  lsp                  [EXPERIMENTAL] Start the Semgrep LSP server
  publish              Upload rule to semgrep.dev
  scan                 Run semgrep rules on files
  shouldafound         Report a false negative in this project.
```

---

## Notes

If you see an error like:

```
/usr/lib/python3/dist-packages/requests/__init__.py:89: RequestsDependencyWarning: urllib3 (1.26.18) or chardet (3.0.4) doesn't match a supported version!
```

👉 Update the requests library:

```bash
pip3 install --upgrade requests
```

---

✅ We have successfully installed Semgrep and explored its functionality.

```
```
````md
# Pattern Syntax in Semgrep

Semgrep provides powerful pattern syntax options for matching code structures.  
Two important constructs are:

---

## 1. Ellipsis Operator (`...`)

The `...` operator matches **zero or more arguments, statements, or characters**.

### Example Pattern:
```python
os.system(...)
````

### Matches:

```
os.system("command")
os.system(command)
os.system("really_weird_command")
os.system(really_weird_command)
```

---

## 2. Metavariables (`$X`, `$Y`, etc.)

Metavariables are placeholders used to track values across code. They can represent variables, arguments, functions, classes, etc.

### Example Pattern:

```python
$X = $Y
```

### Command:

```bash
semgrep --lang python -e '$X = $Y' .
```

### Matches:

```
taskManager/forms.py
27:    user_list = User.objects.order_by('date_joined')
28:    user_tuple = []
29:    counter = 1
32:        counter = counter + 1
41:    task_list = []
42:    tasks = Task.objects.all()
47:    task_tuple = []
48:    counter = 1
51:        counter = counter + 1
60:    proj_list = Project.objects.all()
61:    proj_tuple = []
...[SNIP]...
```

---

## 3. Object Attribute Matching

We can also search for **specific method calls**.

### Example Pattern:

```python
$X.objects.get
```

### Command:

```bash
semgrep --lang python -e '$X.objects.get' .
```

### Matches:

```
taskManager/views.py
42:    proj = Project.objects.get(pk=project_id)
53:                user = User.objects.get(pk=userid)
54:                task = Task.objects.get(pk=taskid)
88:                user = User.objects.get(pk=userid)
89:                project = Project.objects.get(pk=projectid)
130:                    grp = Group.objects.get(name=accesslevel)
133:                specified_user = User.objects.get(pk=post_data["userid"])
174:        proj = Project.objects.get(pk=project_id)
208:    file = File.objects.get(pk=file_id)
222:    user = User.objects.get(pk=user_id)
244:        proj = Project.objects.get(pk=project_id)
```

---

✅ As shown, **Semgrep’s pattern syntax is flexible and powerful**.
It allows you to search for exact functions, variable assignments, or generalized patterns across your codebase.

```
```
````md
# Writing a Semgrep Rule to Detect SQL Injection

We can write a **custom Semgrep rule** to catch SQL Injection vulnerabilities caused by unsafe string formatting inside `cursor.execute()` calls.  

The provided code snippet shows:

```python
curs = connection.cursor()
curs.execute(
    "insert into taskManager_file ('name','path','project_id') values ('%s','%s',%s)" %
    (name, upload_path, project_id))
````

This is a classic **SQL Injection** vulnerability because string interpolation (`%`) is used instead of parameterized queries.

---

## Semgrep Rule: `sql_injection.yaml`

```yaml
rules:
- id: possible-sql-injection
  patterns:
    - pattern: $CURSOR.execute("%" % ...)
  message: >
    Possible SQL Injection vulnerability.
    Avoid string formatting in SQL queries and use parameterized queries instead.
  languages:
    - python
  severity: ERROR
```

---

## Running the Rule

```bash
semgrep --lang python -f sql_injection.yaml .
```

---

## Example Output

```
running 1 rules...
taskManager/views.py
severity:error rule:possible-sql-injection: Possible SQL Injection vulnerability.
                curs.execute(
                    "insert into taskManager_file ('name','path','project_id') values ('%s','%s',%s)" %
                    (name, upload_path, project_id))
ran 1 rules on 50 files: 1 findings
```

---

✅ This custom rule matches **any SQL query using string interpolation (`%`) inside `.execute()` calls**, flagging them as **SQL Injection risks**.

Would you like me to also show you how to **extend this rule** to catch **f-strings** and **`.format()`** usage in SQL queries (not just `%` formatting)?

````md
# Solution: Detecting SQL Injection with Semgrep

Here’s the **Semgrep rule** that detects SQL Injection (SQLi) vulnerabilities using multiple string concatenation/formatting techniques inside Python code.

## Rule File: `challenge.yaml`

```yaml
cat > challenge.yaml << "EOF"
rules:
 - id: find-sqli
   patterns:
     - pattern-inside: |
         def $FUNC(...):
           ...
     - pattern-either:
       - pattern: $C.execute(..., $FOO % $BAR, ...)
       - pattern: $C.execute(..., $FOO + $BAR, ...)
       - pattern: $C.execute(..., f"...{...}...", ...)
       - pattern: $C.execute(..., $FOO.format(...), ...)
   message: SQLi
   severity: WARNING
   languages:
   - python
EOF
````

---

## Running the Rule

```bash
semgrep --lang python -f challenge.yaml .
```

---

## Example Output

```
┌────────────────┐
│ 1 Code Finding │
└────────────────┘

    taskManager/views.py 
       find-sqli
          SQLi            

          175┆ curs.execute(
          176┆     "insert into taskManager_file ('name','path','project_id') values ('%s','%s',%s)" %
          177┆     (name, upload_path, project_id))
```

---

### ✅ Explanation

* **`pattern-inside`** ensures we only match SQL queries inside function definitions.
* **`pattern-either`** covers multiple unsafe query-building techniques:

  * `%` string interpolation
  * String concatenation (`+`)
  * f-strings (`f"..."`)
  * `.format()` usage

This ensures that any SQL queries constructed dynamically with untrusted input are flagged as **SQL Injection risks**.

---

````md
# Hunting Vulnerability Using Semgrep

## Download the source code
We will do all the exercises locally first in DevSecOps-Box, so let’s start the exercise.

First, we need to download the source code of the project from our git repository.

```bash
git clone https://gitlab.practical-devsecops.training/pdso/django.nv webapp
````

### Command Output

```
Cloning into 'webapp'...
warning: redirecting to https://gitlab.practical-devsecops.training/pdso/django.nv.git/
remote: Enumerating objects: 228, done.
remote: Total 228 (delta 0), reused 0 (delta 0), pack-reused 228
Receiving objects: 100% (228/228), 1.03 MiB | 1.04 MiB/s, done.
Resolving deltas: 100% (86/86), done.
```

Let’s `cd` into the application so we can scan the app.

```bash
cd webapp
```

We are now in the `webapp` directory.

Let’s move to the next step.

```
```
````md
# Install Semgrep

Semgrep is a fast, open-source, static analysis tool that excels at expressing code standards — without complicated queries — and surfacing bugs early in the development flow. Precise rules look like the code you’re searching; no more traversing abstract syntax trees or wrestling with regexes.

You can find more details about the project at [https://github.com/returntocorp/semgrep](https://github.com/returntocorp/semgrep).

Let’s install the Semgrep tool on the system to perform static analysis.

```bash
pip3 install semgrep==1.31.0
````

### Command Output

```
Collecting semgrep==1.30.0
  Downloading semgrep-0.108.0-cp37.cp38.cp39.py37.py38.py39-none-any.whl (25.2 MB)
     |████████████████████████████████| 25.2 MB 5.0 MB/s 
Collecting colorama~=0.4.0
  Downloading colorama-0.4.5-py2.py3-none-any.whl (16 kB)
Collecting python-lsp-jsonrpc~=1.0.0
  Downloading python_lsp_jsonrpc-1.0.0-py3-none-any.whl (8.5 kB)

...[SNIP]...

Successfully installed attrs-21.4.0 boltons-21.0.0 bracex-2.3.post1 click-8.1.3 click-option-group-0.5.3 colorama-0.4.5 defusedxml-0.7.1 face-20.1.1 glom-22.1.0 importlib-resources-5.9.0 jsonschema-4.9.1 packaging-21.3 peewee-3.15.1 pkgutil-resolve-name-1.3.10 pyparsing-3.0.9 pyrsistent-0.18.1 python-lsp-jsonrpc-1.0.0 ruamel.yaml-0.17.21 ruamel.yaml.clib-0.2.6 semgrep-0.108.0 tqdm-4.64.0 typing-extensions-4.3.0 ujson-5.4.0 urllib3-1.26.11 wcmatch-8.4 zipp-3.8.1
```

We have successfully installed semgrep, let’s explore its functionality now.

```bash
semgrep --help
```

### Command Output

```
Usage: pysemgrep [OPTIONS] COMMAND [ARGS]...

  To get started quickly, run `semgrep scan --config auto`

  Run `semgrep SUBCOMMAND --help` for more information on each subcommand

  If no subcommand is passed, will run `scan` subcommand by default

Options:
  -h, --help  Show this message and exit.

Commands:
  ci                   The recommended way to run semgrep in CI
  install-semgrep-pro  Install the Semgrep Pro Engine
  login                Obtain and save credentials for semgrep.dev
  logout               Remove locally stored credentials to semgrep.dev
  lsp                  [EXPERIMENTAL] Start the Semgrep LSP server
  publish              Upload rule to semgrep.dev
  scan                 Run semgrep rules on files
  shouldafound         Report a false negative in this project.
```

For more information about the Semgrep scan feature, you can visit this link: **Semgrep Scan Command Options** or try inputting:

```bash
semgrep scan --help
```

---

### Notes

If you’re encountering the same error with every command inputted, like the following:

#### Command Output

```
/usr/lib/python3/dist-packages/requests/__init__.py:89: RequestsDependencyWarning: urllib3 (1.26.18) or chardet (3.0.4) doesn't match a supported version! warnings.warn("urllib3 ({}) or chardet ({}) doesn't match a supported
```

It’s likely due to an outdated version of the Python `requests` library. You can update it using the following command:

```bash
pip3 install --upgrade requests
```

---

Let’s move to the next step.

```
```
````md
# Cross-Site Request Forgery (CSRF)

Django applications would be vulnerable to **CSRF attacks** if we’re using `csrf_exempt` decorator or `django.middleware.csrf.CsrfViewMiddleware` middleware is not set in `MIDDLEWARE_CLASSES`.  
We can use **Semgrep** to find those cases and mitigate CSRF vulnerabilities.

```bash
cat > csrf_hunting.yaml <<EOF
rules:
- id: possible-csrf
  patterns:
  - pattern-inside: | 
      @csrf_exempt
      def \$FUNC(\$X):
          ...
  message: |
    Possible CSRF
  languages:
  - python
  severity: WARNING

- id: no-csrf-middleware
  patterns:
  - pattern: MIDDLEWARE_CLASSES=(...)
  - pattern-not: MIDDLEWARE_CLASSES=(...,'django.middleware.csrf.CsrfViewMiddleware',...)
  message: |
    No CSRF middleware
  languages:
  - python
  severity: WARNING
EOF
````

There are **2 rules** in the YAML file:

1. **possible-csrf**: Find all function definitions that use `@csrf_exempt` as decorator.
2. **no-csrf-middleware**: Search the string `MIDDLEWARE_CLASSES` and check if it doesn’t have `'django.middleware.csrf.CsrfViewMiddleware'` in it.

⚠️ The output number may vary as it is dynamic.

---

### Run Semgrep with our rules against the source code:

```bash
semgrep -f csrf_hunting.yaml .
```

### Command Output

```
Scanning 50 files with 2 python rules.
  100%|█████████████████████████████████████████████████████████████████████████████████████████████████████|50/50 tasks

Findings:

  taskManager/views.py 
     possible-csrf
        Possible CSRF

        739┆ @csrf_exempt
        740┆ def reset_password(request):
        741┆ 
        742┆     if request.method == 'POST':
        743┆ 
        744┆         reset_token = request.POST.get('reset_token')
        745┆ 
        746┆         try:
        747┆             userprofile = UserProfile.objects.get(reset_token = reset_token)
        748┆             if timezone.now() > userprofile.reset_token_expiration:
           [hid 27 additional lines, adjust with --max-lines-per-finding] 
        779┆ @csrf_exempt
        780┆ def forgot_password(request):
        781┆ 
        782┆     if request.method == 'POST':
        783┆         t_email = request.POST.get('email')
        784┆ 
        785┆         try:
        786┆             reset_user = User.objects.get(email=t_email)
        787┆ 
        788┆             # Generate secure random 6 digit number
           [hid 21 additional lines, adjust with --max-lines-per-finding] 
        813┆ @csrf_exempt
        814┆ def change_password(request):
        815┆ 
        816┆     if request.method == 'POST':
        817┆         user = request.user
        818┆         old_password = request.POST.get('old_password')
        819┆         new_password = request.POST.get('new_password')
        820┆         confirm_password = request.POST.get('confirm_password')
        821┆ 
        822┆         if authenticate(username=user.username, password=old_password):
           [hid 12 additional lines, adjust with --max-lines-per-finding] 

Some files were skipped or only partially analyzed.
  Scan was limited to files tracked by git.

Ran 2 rules on 50 files: 3 findings.
```

---

✅ As you can see, we found **three functions** where CSRF vulnerability can be exploited.

---

Let’s move to the next step.

```
```
````md
# Misconfigurations

**Security misconfiguration** is a class of vulnerability that occurs when the software is set up incorrectly, left insecure, and can happen at any level of an application stack including the platform, web server, application server, database, framework, or any custom code.  

**Source:** OWASP Top 10

---

### Example: Detecting `DEBUG=True` in Django
Let’s create a rule to hunt misconfigurations in our application — e.g., availability of `DEBUG=True` flag in `settings.py` file.

```bash
cat > debug_enable.yaml <<EOF 
rules:
- id: debug-enabled
  patterns:
  - pattern: DEBUG=True
  message: |
    Detected Django app with DEBUG=True. Do not deploy to production with this flag enabled
    as it will leak sensitive information.
  metadata:
    cwe: 'CWE-489: Active Debug Code'
    owasp: 'A6: Security Misconfiguration'
    references:
    - https://blog.scrt.ch/2018/08/24/remote-code-execution-on-a-facebook-server/
  severity: WARNING
  languages:
  - python
EOF
````

---

### Run Semgrep with the rule:

```bash
semgrep -f debug_enable.yaml .
```

### Command Output

```
Scanning 50 files.
  100%|█████████████████████████████████████████████████████████████████████████████████████████████████████|50/50 tasks

Findings:

  taskManager/settings.py 
     debug-enabled
        Detected Django app with DEBUG=True. Do not deploy to production with this flag enabled as
        it will leak sensitive information.

         28┆ DEBUG = True

Some files were skipped or only partially analyzed.
  Scan was limited to files tracked by git.

Ran 1 rule on 50 files: 1 finding.
```

---

✅ We scanned about **51 files** and found **1 security issue**:
The `DEBUG=True` flag enabled in production, which is a dangerous **misconfiguration**.

---

Let’s move to the next step.

```
```
````md
# Ported Security Tools Ruleset

Semgrep’s team converted the security rulesets from different tools (like **Bandit**, **Gosec**, **NodeJsScan**, **FindSecBugs**, or **eslint-plugin-security**) into Semgrep rules.  
We will use the **Bandit ruleset** for Semgrep in this section.

---

### Run Semgrep with Bandit ruleset

```bash
semgrep --config "p/bandit" .
````

---

### Command Output

```
Scanning 50 files with 61 python rules.
  100%|█████████████████████████████████████████████████████████████████████████████████████████████████████|50/50 tasks

Findings:

  taskManager/misc.py 
     python.lang.security.audit.dangerous-system-call.dangerous-system-call
        Found dynamic content used in a system call. This is dangerous if external data can reach
        this function call because it allows a malicious actor to execute commands. Use the
        'subprocess' module instead, which is easier to use without accidentally exposing a command
        injection vulnerability.
        Details: https://sg.run/vzKA

         33┆ os.system(
         34┆     "mv " +
         35┆     uploaded_file.temporary_file_path() +
         36┆     " " +
         37┆     "%s/%s" %
         38┆     (upload_dir_path,
         39┆      title))


  taskManager/views.py 
     python.lang.security.audit.formatted-sql-query.formatted-sql-query
        Detected possible formatted SQL query. Use parameterized queries instead.
        Details: https://sg.run/EkWw

        183┆ curs.execute(
        184┆     "insert into taskManager_file ('name','path','project_id') values ('%s','%s',%s)" %
        185┆     (name, upload_path, project_id))

Some files were skipped or only partially analyzed.
  Scan was limited to files tracked by git.

Ran 61 rules on 50 files: 2 findings.
```

---

### Analysis

* ✅ Semgrep ran **61 rules** from the Bandit ruleset.
* ✅ It found **2 vulnerabilities**:

  * **Dangerous system call** (`os.system(...)`)
  * **Formatted SQL query** (possible SQLi)

---

### Question

> Did it miss any vulnerabilities which Bandit found? If so, why?

Yes.

* Bandit may detect **more issues** (e.g., weak hashing functions, hardcoded passwords, YAML unsafe load).
* Semgrep **ported rules** do not cover *every single Bandit plugin*. Some rules are simplified or not yet implemented.
* Semgrep matches **patterns in source code**, while Bandit sometimes goes deeper into **AST and context-specific analysis**, leading to differences in findings.

---

🔑 **Takeaway:**
Predefined rulesets (like Bandit-in-Semgrep) are useful but **not comprehensive**. Always combine:

* Native tool (**Bandit**)
* Semgrep’s flexible custom rules

for better coverage.

---

Let’s move to the next step.

```
```
```md
Challenge
Write a semgrep rule to detect Insecure Redirect in the following code snippet
Command Output
Scanning 50 files.
  100%|█████████████████████████████████████████████████████████████████████████████████████████████████████████████|50/50 tasks

Findings:

  taskManager/views.py 
     CWE-601
        Insecure Redirect

        369┆ return redirect(request.GET.get('redirect', '/taskManager/'))

Some files were skipped or only partially analyzed.
  Scan was limited to files tracked by git.

Ran 1 rule on 50 files: 1 finding.
Hint
Here is the final rule to detect Insecure Redirect issue:


cat > insecure_redirect.yaml <<EOF
rules:
- id: CWE-601
  pattern: |
    return redirect(request.$M.get(...))
  message: Insecure Redirect
  severity: WARNING
  languages:
  - python
EOF

semgrep -f insecure_redirect.yaml
```
```md
How to Fix The Issues Reported by Semgrep
In this scenario, you will learn how to fix issues reported by the semgrep tool in Django source code.

You will do the following in this activity.
1. Download the source code from the git repository.
2. Install the Semgrep tool.
3. Run the SAST scan on the code.
4. Fix the issues found by Semgrep.
```
```md
How to Fix The Issues Reported by Semgrep
Download the source code
We will do all the exercises locally first in DevSecOps-Box, so lets start the activity.

First, we need to download the source code of the project from our git repository.


git clone https://gitlab.practical-devsecops.training/pdso/dvpa webapp

Command Output
Cloning into 'webapp'...
warning: redirecting to https://gitlab.practical-devsecops.training/pdso/dvpa.git/
remote: Enumerating objects: 47, done.
remote: Counting objects: 100% (47/47), done.
remote: Compressing objects: 100% (47/47), done.
remote: Total 117 (delta 18), reused 0 (delta 0), pack-reused 70
Receiving objects: 100% (117/117), 241.67 KiB | 388.00 KiB/s, done.
Resolving deltas: 100% (24/24), done.
Lets cd into the application so we can scan the app.


cd webapp

We are now in the webapp directory.

Let’s move to the next step.
```
```md
Install Semgrep
Semgrep is a fast, open-source, static analysis tool that excels at expressing code standards — without complicated queries — and surfacing bugs early in the development flow. Precise rules look like the code you’re searching; no more traversing abstract syntax trees or wrestling with regexes.

You can find more details about the project at https://github.com/returntocorp/semgrep.

Let’s install the Semgrep tool on the system to perform static analysis.


pip3 install semgrep==1.31.0

Command Output
Collecting semgrep==1.30.0
  Downloading semgrep-0.108.0-cp37.cp38.cp39.py37.py38.py39-none-any.whl (25.2 MB)
     |████████████████████████████████| 25.2 MB 5.0 MB/s 
Collecting colorama~=0.4.0
  Downloading colorama-0.4.5-py2.py3-none-any.whl (16 kB)
Collecting python-lsp-jsonrpc~=1.0.0
  Downloading python_lsp_jsonrpc-1.0.0-py3-none-any.whl (8.5 kB)

...[SNIP]...

Successfully installed attrs-21.4.0 boltons-21.0.0 bracex-2.3.post1 click-8.1.3 click-option-group-0.5.3 colorama-0.4.5 defusedxml-0.7.1 face-20.1.1 glom-22.1.0 importlib-resources-5.9.0 jsonschema-4.9.1 packaging-21.3 peewee-3.15.1 pkgutil-resolve-name-1.3.10 pyparsing-3.0.9 pyrsistent-0.18.1 python-lsp-jsonrpc-1.0.0 ruamel.yaml-0.17.21 ruamel.yaml.clib-0.2.6 semgrep-0.108.0 tqdm-4.64.0 typing-extensions-4.3.0 ujson-5.4.0 urllib3-1.26.11 wcmatch-8.4 zipp-3.8.1
We have successfully installed semgrep, let’s explore its functionality now.


semgrep --help

Command Output
Usage: pysemgrep [OPTIONS] COMMAND [ARGS]...

  To get started quickly, run `semgrep scan --config auto`

  Run `semgrep SUBCOMMAND --help` for more information on each subcommand

  If no subcommand is passed, will run `scan` subcommand by default

Options:
  -h, --help  Show this message and exit.

Commands:
  ci                   The recommended way to run semgrep in CI
  install-semgrep-pro  Install the Semgrep Pro Engine
  login                Obtain and save credentials for semgrep.dev
  logout               Remove locally stored credentials to semgrep.dev
  lsp                  [EXPERIMENTAL] Start the Semgrep LSP server
  publish              Upload rule to semgrep.dev
  scan                 Run semgrep rules on files
  shouldafound         Report a false negative in this project.
For more information about the Semgrep scan feature, you can visit this link: Semgrep Scan Command Options or try inputting “semgrep scan –help”.

Notes

If you’re encountering the same error with every command inputted, like the following:

Command Output
/usr/lib/python3/dist-packages/requests/__init__.py:89: RequestsDependencyWarning: urllib3 (1.26.18) or chardet (3.0.4) doesn't match a supported version! warnings.warn("urllib3 ({}) or chardet ({}) doesn't match a supported
It’s likely due to an outdated version of the Python requests library. You can update it using the following command:


pip3 install --upgrade requests

Let’s move to the next step.
```
```md
Run the scanner
Before we run semgrep, we need to create the rules file, semgrep_rules.yaml. These rules will help us find vulnerabilities in the code. You can use any text editor to create this file with the following content.

Click anywhere to copy

rules:
- id: avoid-pyyaml-load
  metadata:
    owasp: 'A8: Insecure Deserialization'
    cwe: 'CWE-502: Deserialization of Untrusted Data'
    references:
    - https://github.com/yaml/pyyaml/wiki/PyYAML-yaml.load(input)-Deprecation
    - https://nvd.nist.gov/vuln/detail/CVE-2017-18342
  languages:
  - python
  message: |
    Avoid using `load()`. `PyYAML.load` can create arbitrary Python
    objects. A malicious actor could exploit this to run arbitrary
    code. Use `safe_load()` instead.
  fix: yaml.safe_load($FOO)
  severity: ERROR
  patterns:
  - pattern-inside: |
      import yaml
      ...
      yaml.load($FOO)
  - pattern: yaml.load($FOO)

- id: possible-sqli
  metadata:
    owasp: 'A1: Injection'
    references:
    - https://owasp.org/www-community/attacks/SQL_Injection
  languages:
  - python
  message: |
    Possible SQL Injection
  severity: WARNING
  patterns:
  - pattern: $X.execute(...)
  - pattern-not: $X.execute(..., [...])
  - pattern-not: $X.execute("...")

In the above file, we have created rules to find Insecure Deserialization and SQL Injection attacks. Let’s run the Semgrep tool against the current working directory.


semgrep -f semgrep_rules.yaml .

Command Output
canning 15 files with 2 python rules.
  100%|█████████████████████████████████████████████████████████████████████████████████████████████████████|15/15 tasks

Findings:

  blog/auth.py 
     possible-sqli
        Possible SQL Injection

         14┆ cur.execute(f"SELECT * FROM users WHERE email='{username}' AND password='{hashsed_password}'")


  blog/dashboard/post.py 
     avoid-pyyaml-load
        Avoid using `load()`. `PyYAML.load` can create arbitrary Python objects. A malicious actor
        could exploit this to run arbitrary code. Use `safe_load()` instead.

         ▶▶┆ Autofix ▶ yaml.safe_load(import_data)
        204┆ import_post_data = yaml.load(import_data)
          ⋮┆----------------------------------------
     possible-sqli
        Possible SQL Injection

         60┆ cur.execute(f"SELECT * FROM posts WHERE slug='{slug}'")


  blog/user.py 
     possible-sqli
        Possible SQL Injection

         57┆ cur.execute(
         58┆     f"INSERT INTO users (`email`, `full_name`, `password`) VALUES ('{email}', '{full_name}', '{hashed_password}')")


  blog/views.py 
     possible-sqli
        Possible SQL Injection

         37┆ cur.execute(f"SELECT * FROM posts WHERE slug='{slug}'")
          ⋮┆----------------------------------------
         68┆ cur.execute(f"SELECT * FROM posts WHERE title LIKE '%{query}%'")

Some files were skipped or only partially analyzed.
  Scan was limited to files tracked by git.

Ran 2 rules on 15 files: 6 findings.
The output number may vary as it is dynamic.

We got 6 findings, one insecure deserialization, and five SQL Injections. We must fix all of these issues to keep this app secure.

Let’s move to the next step.
```
```md
Fixing Insecure Deserialization
Python yaml library has a known vulnerability around YAML deserialization. We can search for this known security issue on the CVE website.

For more details, please visit CVE-2020-1747. As mentioned before, our code is vulnerable to Deserialization Attacks.

If you recall, one of the findings in the previous step was unsafe YAML load, and the following was the code snippet for the same.

Command Output
  blog/dashboard/post.py 
     avoid-pyyaml-load
        Avoid using `load()`. `PyYAML.load` can create arbitrary Python objects. A malicious actor
        could exploit this to run arbitrary code. Use `safe_load()` instead.

         ▶▶┆ Autofix ▶ yaml.safe_load(import_data)
        204┆ import_post_data = yaml.load(import_data)
          ⋮┆----------------------------------------
     possible-sqli
        Possible SQL Injection

         60┆ cur.execute(f"SELECT * FROM posts WHERE slug='{slug}'")
Lets verify this issue exists by opening up the blog/dashboard/post.py file using any text editor like vim or nano. Ensure the security issue exists at line 204 and the program uses the insecure yaml.load function. To fix this issue, we need to replace yaml.load with a safe alternative yaml.safe_load.

Let’s run the semgrep scanner once again


semgrep -f semgrep_rules.yaml .

Command Output
Scanning 15 files with 2 python rules.
  100%|█████████████████████████████████████████████████████████████████████████████████████████████████████|15/15 tasks

Findings:

  blog/auth.py 
     possible-sqli
        Possible SQL Injection

         14┆ cur.execute(f"SELECT * FROM users WHERE email='{username}' AND password='{hashsed_password}'")


  blog/dashboard/post.py 
     possible-sqli
        Possible SQL Injection

         60┆ cur.execute(f"SELECT * FROM posts WHERE slug='{slug}'")


  blog/user.py 
     possible-sqli
        Possible SQL Injection

         57┆ cur.execute(
         58┆     f"INSERT INTO users (`email`, `full_name`, `password`) VALUES ('{email}', '{full_name}', '{hashed_password}')")


  blog/views.py 
     possible-sqli
        Possible SQL Injection

         37┆ cur.execute(f"SELECT * FROM posts WHERE slug='{slug}'")
          ⋮┆----------------------------------------
         68┆ cur.execute(f"SELECT * FROM posts WHERE title LIKE '%{query}%'")

Some files were skipped or only partially analyzed.
  Scan was limited to files tracked by git.

Ran 2 rules on 15 files: 5 findings.
As you can see, there is no yaml.load issue in the output. The total issue count has decreased from 6 to 5.

Let’s move to the next step.
```
```md
Fixing SQL Injection Issue
Apart from Insecure Deserialization issue, there were five possible SQL Injection issues as well. We can fix these issues in various ways, but the best way to fix SQL Injection issues is Parameterized queries, also known as Parameter Binding.

Command Output
Findings:

  blog/auth.py 
     possible-sqli
        Possible SQL Injection

         14┆ cur.execute(f"SELECT * FROM users WHERE email='{username}' AND password='{hashsed_password}'")


  blog/dashboard/post.py 
     possible-sqli
        Possible SQL Injection

         60┆ cur.execute(f"SELECT * FROM posts WHERE slug='{slug}'")


  blog/user.py 
     possible-sqli
        Possible SQL Injection

         57┆ cur.execute(
         58┆     f"INSERT INTO users (`email`, `full_name`, `password`) VALUES ('{email}', '{full_name}', '{hashed_password}')")


  blog/views.py 
     possible-sqli
        Possible SQL Injection

         37┆ cur.execute(f"SELECT * FROM posts WHERE slug='{slug}'")
          ⋮┆----------------------------------------
         68┆ cur.execute(f"SELECT * FROM posts WHERE title LIKE '%{query}%'")

First, we will try to fix the SQL Injection issue present in the blog/auth.py file at line number 14. As you can see in the following code snippet, there is a SQL injection issue in the check_auth function.

Command Output
def check_auth(username, password):
   """ This function is called to check if a username / password
       combination is valid.
   """
   cur = db.connection.cursor()
   hashsed_password = hashlib.md5(password.encode()).hexdigest()
   cur.execute(f"SELECT * FROM users WHERE email='{username}' AND password='{hashsed_password}'")
   user = cur.fetchone()

   if user is None:
      return False

   session["is_logged_in"] = True
   session["id"] = user.get("id")
   session["email"] = user.get("email")
   session["full_name"] = user.get("full_name")

   return user

The cur.execute function call will execute the SQL query on the database. It takes the username and password as possible inputs to the SQL query.

You can also verify this behavior dynamically by reproducing SQL query errors.

Note

Learn more about SQL Injection here.

You can replace line number 14 with the following code. This code will fix the SQL Injection issue.

cur.execute(f"SELECT * FROM users WHERE email=%s AND password=%s", [username, hashsed_password ])

Then, rerun the scanner

semgrep -f semgrep_rules.yaml .

Command Output
Scanning 15 files with 2 python rules.
  100%|█████████████████████████████████████████████████████████████████████████████████████████████████████|15/15 tasks

Findings:

  blog/dashboard/post.py 
     possible-sqli
        Possible SQL Injection

         60┆ cur.execute(f"SELECT * FROM posts WHERE slug='{slug}'")


  blog/user.py 
     possible-sqli
        Possible SQL Injection

         57┆ cur.execute(
         58┆     f"INSERT INTO users (`email`, `full_name`, `password`) VALUES ('{email}', '{full_name}', '{hashed_password}')")


  blog/views.py 
     possible-sqli
        Possible SQL Injection

         37┆ cur.execute(f"SELECT * FROM posts WHERE slug='{slug}'")
          ⋮┆----------------------------------------
         68┆ cur.execute(f"SELECT * FROM posts WHERE title LIKE '%{query}%'")

Some files were skipped or only partially analyzed.
  Scan was limited to files tracked by git.

Ran 2 rules on 15 files: 4 findings.
As you can see(the last line), semgrep found only 4 issues. Great!

Let’s move to the next step.
```
```md
How To Embed Semgrep Into GitLab
A Simple CI/CD Pipeline
Considering your DevOps team created a simple CI pipeline with the following contents.

image: docker:20.10  # To run all jobs in this pipeline, use the latest docker image

services:
  - docker:dind       # To run all jobs in this pipeline, use a docker image that contains a docker daemon running inside (dind - docker in docker). Reference: https://forum.gitlab.com/t/why-services-docker-dind-is-needed-while-already-having-image-docker/43534

stages:
  - build
  - test
  - release
  - preprod
  - integration
  - prod

build:
  stage: build
  image: python:3.6
  before_script:
   - pip3 install --upgrade virtualenv
  script:
   - virtualenv env                       # Create a virtual environment for the python application
   - source env/bin/activate              # Activate the virtual environment
   - pip install -r requirements.txt      # Install the required third party packages as defined in requirements.txt
   - python manage.py check               # Run checks to ensure the application is working fine

test:
  stage: test
  image: python:3.6
  before_script:
   - pip3 install --upgrade virtualenv
  script:
   - virtualenv env
   - source env/bin/activate
   - pip install -r requirements.txt
   - python manage.py test taskManager
We have two jobs in this pipeline, a build job and a test job. As a security engineer, I do not care what they are doing as part of these jobs. Why? Imagine having to learn every build/testing tool used by your DevOps team, it will be a nightmare. Instead, rely on the DevOps team for help.

Let’s log in to GitLab using the following details.

GitLab CI/CD Machine
Name	Value
URL	https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/django-nv/-/blob/main/.gitlab-ci.yml
Username	root
Password	pdso-training
Next, we need to create a CI/CD pipeline by adding the above content to the .gitlab-ci.yml file. Click on the Edit button to start adding the content.

Save changes to the file using Commit changes button.

Verify the pipeline run
As soon as a change is made to the repository, the pipeline starts executing the jobs.

We can see the results of this pipeline by visiting https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/django-nv/pipelines.

Click on the appropriate job name to see the output.

In the next step, we will embed Semgrep in the CI/CD pipeline and follow all the best practices.

Let’s move to the next step.
```
```md
Embed Semgrep in CI/CD Pipeline
As discussed in the Static Analysis using Semgrep exercise, we can embed Semgrep in our CI/CD pipeline. However, do remember you need to run the command manually before you embed this SAST tool in the pipeline.

Click anywhere to copy

image: docker:20.10  # To run all jobs in this pipeline, use the latest docker image

services:
  - docker:dind       # To run all jobs in this pipeline, use a docker image that contains a docker daemon running inside (dind - docker in docker). Reference: https://forum.gitlab.com/t/why-services-docker-dind-is-needed-while-already-having-image-docker/43534

stages:
  - build
  - test
  - release
  - preprod
  - integration
  - prod

build:
  stage: build
  image: python:3.6
  before_script:
   - pip3 install --upgrade virtualenv
  script:
   - virtualenv env                       # Create a virtual environment for the python application
   - source env/bin/activate              # Activate the virtual environment
   - pip install -r requirements.txt      # Install the required third party packages as defined in requirements.txt
   - python manage.py check               # Run checks to ensure the application is working fine

test:
  stage: test
  image: python:3.6
  before_script:
   - pip3 install --upgrade virtualenv
  script:
   - virtualenv env
   - source env/bin/activate
   - pip install -r requirements.txt
   - python manage.py test taskManager

semgrep:
  stage: build
  script:
   - docker run --rm -v ${PWD}:/src returntocorp/semgrep semgrep --config auto --output semgrep-output.json --json

As discussed, any change to the repo kick starts the pipeline.

Note

We’ve discussed different methods of using the tool:

Native Installation:
Directly installing the tool on the system.
Package Manager or Binary:
Installing via a package manager or using the binary file.
Docker:
Running the tool within a Docker container.
In summary:

All methods are suitable for CI/CD integration.
Docker is recommended for CI/CD as it operates smoothly without dependencies.
Using the binary file is efficient, avoiding additional dependencies.
Ultimately, you can choose either method based on your specific situation.

We can see the results of this pipeline by visiting https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/django-nv/pipelines.

Click on the appropriate job name to see the output.

Let’s move to the next step.
```
````md
Allow the Job Failure
Remember!

Except for DevSecOps-Box, every other machine closes after two hours, even if you are in the middle of the exercise  
After two hours, in case of a 404, you need to refresh the exercise page and click on Start the Exercise button to continue working  

We do not want to fail the builds/jobs/scan in DevSecOps Maturity Levels 1 and 2, as security tools spit a significant amount of false positives.

You can use the allow_failure tag to not fail the build even though the tool found issues.

```yaml
semgrep:
  stage: build
  script:
   - docker run --rm -v ${PWD}:/src returntocorp/semgrep semgrep --config auto --output semgrep-output.json --json
  artifacts:
    paths: [semgrep-output.json]
    when: always  # What is this for?
    expire_in: one week
  allow_failure: true    #<--- allow the build to fail but don't mark it as such
````

After adding the allow\_failure tag, the pipeline would look like the following.

Click anywhere to copy

```yaml
image: docker:20.10  # To run all jobs in this pipeline, use the latest docker image

services:
  - docker:dind       # To run all jobs in this pipeline, use a docker image that contains a docker daemon running inside (dind - docker in docker). Reference: https://forum.gitlab.com/t/why-services-docker-dind-is-needed-while-already-having-image-docker/43534

stages:
  - build
  - test
  - release
  - preprod
  - integration
  - prod

build:
  stage: build
  image: python:3.6
  before_script:
   - pip3 install --upgrade virtualenv
  script:
   - virtualenv env                       # Create a virtual environment for the python application
   - source env/bin/activate              # Activate the virtual environment
   - pip install -r requirements.txt      # Install the required third party packages as defined in requirements.txt
   - python manage.py check               # Run checks to ensure the application is working fine

test:
  stage: test
  image: python:3.6
  before_script:
   - pip3 install --upgrade virtualenv
  script:
   - virtualenv env
   - source env/bin/activate
   - pip install -r requirements.txt
   - python manage.py test taskManager

semgrep:
  stage: build
  script:
   - docker run --rm -v ${PWD}:/src returntocorp/semgrep semgrep --config auto --output semgrep-output.json --json
  artifacts:
    paths: [semgrep-output.json]
    when: always  # What is this for?
    expire_in: one week
  allow_failure: true    #<--- allow the build to fail but don't mark it as such

integration:
  stage: integration
  script:
    - echo "This is an integration step"
    - exit 1
  allow_failure: true # Even if the job fails, continue to the next stages

prod:
  stage: prod
  script:
    - echo "This is a deploy step."
  when: manual # Continuous Delivery
```

You will notice that the semgrep job failed however it didn’t block others from continuing further.

As discussed, any change to the repo kick starts the pipeline.

We can see the results of this pipeline by visiting
👉 [https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/django-nv/pipelines](https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/django-nv/pipelines)

Click on the appropriate job name to see the output.

```
```
````md
# How To Embed Gitleaks Into GitLab

## A Simple CI/CD Pipeline

In this exercise, we will integrate GitLeaks into a CI/CD pipeline to enhance security practices. To begin, let’s understand the existing CI/CD pipeline and then proceed with the integration of GitLeaks.

---

```yaml
image: docker:20.10  # To run all jobs in this pipeline, use the latest docker image

services:
  - docker:dind       # To run all jobs in this pipeline, use a docker image that contains a docker daemon running inside (dind - docker in docker). Reference: https://forum.gitlab.com/t/why-services-docker-dind-is-needed-while-already-having-image-docker/43534

stages:
  - build
  - test
  - release
  - preprod
  - integration
  - prod

build:
  stage: build
  image: python:3.6
  before_script:
   - pip3 install --upgrade virtualenv
  script:
   - virtualenv env                       # Create a virtual environment for the python application
   - source env/bin/activate              # Activate the virtual environment
   - pip install -r requirements.txt      # Install the required third party packages as defined in requirements.txt
   - python manage.py check               # Run checks to ensure the application is working fine

test:
  stage: test
  image: python:3.6
  before_script:
   - pip3 install --upgrade virtualenv
  script:
   - virtualenv env
   - source env/bin/activate
   - pip install -r requirements.txt
   - python manage.py test taskManager
````

---

We have two jobs in this pipeline, a **build job** and a **test job**.

As a security engineer, I do not care what they are doing as part of these jobs. Why?
Imagine having to learn every build/testing tool used by your DevOps team, it will be a nightmare. Instead, rely on the DevOps team for help.

---

## GitLab CI/CD Machine

| Name     | Value                                                                                                                |
| -------- | -------------------------------------------------------------------------------------------------------------------- |
| Link     | [GitLab Repo](https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/django-nv/-/blob/main/.gitlab-ci.yml) |
| Username | root                                                                                                                 |
| Password | pdso-training                                                                                                        |

---

## Next Steps

1. Create a **CI/CD pipeline** by adding the above content to the `.gitlab-ci.yml` file.

   * Click on the **Edit** button to start adding the content.

2. Save changes to the file using the **Commit changes** button.

---

## Verify the pipeline run

As soon as a change is made to the repository, the pipeline starts executing the jobs.

We can see the results of this pipeline by visiting:
👉 [https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/django-nv/pipelines](https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/django-nv/pipelines)

Click on the appropriate job name to see the output.

---

In the next step, we will **embed Gitleaks** in the CI/CD pipeline and follow all the best practices.

👉 Let’s move to the next step.

```
```
````md
# How To Embed Gitleaks Into GitLab

## A Simple CI/CD Pipeline
In this exercise, we will integrate GitLeaks into a CI/CD pipeline to enhance security practices. To begin, let’s understand the existing CI/CD pipeline and then proceed with the integration of GitLeaks.

---

**Click anywhere to copy**

```yaml
image: docker:20.10  # To run all jobs in this pipeline, use the latest docker image

services:
  - docker:dind       # To run all jobs in this pipeline, use a docker image that contains a docker daemon running inside (dind - docker in docker). Reference: https://forum.gitlab.com/t/why-services-docker-dind-is-needed-while-already-having-image-docker/43534

stages:
  - build
  - test
  - release
  - preprod
  - integration
  - prod

build:
  stage: build
  image: python:3.6
  before_script:
   - pip3 install --upgrade virtualenv
  script:
   - virtualenv env                       # Create a virtual environment for the python application
   - source env/bin/activate              # Activate the virtual environment
   - pip install -r requirements.txt      # Install the required third party packages as defined in requirements.txt
   - python manage.py check               # Run checks to ensure the application is working fine

test:
  stage: test
  image: python:3.6
  before_script:
   - pip3 install --upgrade virtualenv
  script:
   - virtualenv env
   - source env/bin/activate
   - pip install -r requirements.txt
   - python manage.py test taskManager
````

---

We have two jobs in this pipeline, a build job and a test job. As a security engineer, I do not care what they are doing as part of these jobs. Why? Imagine having to learn every build/testing tool used by your DevOps team, it will be a nightmare. Instead, rely on the DevOps team for help.

---

### Let’s log in to Gitlab using the following details.

**GitLab CI/CD Machine**

| Name     | Value                                                                                                                                                                                                          |
| -------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Link     | [https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/django-nv/-/blob/main/.gitlab-ci.yml](https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/django-nv/-/blob/main/.gitlab-ci.yml) |
| Username | root                                                                                                                                                                                                           |
| Password | pdso-training                                                                                                                                                                                                  |

---

Next, we need to create a CI/CD pipeline by adding the above content to the `.gitlab-ci.yml` file. Click on the **Edit** button to start adding the content.

Save changes to the file using **Commit changes** button.

---

### Verify the pipeline run

As soon as a change is made to the repository, the pipeline starts executing the jobs.

We can see the results of this pipeline by visiting:
🔗 [https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/django-nv/pipelines](https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/django-nv/pipelines)

Click on the appropriate job name to see the output.

---

In the next step, we will embed **Gitleaks** in the CI/CD pipeline and follow all the best practices.

Let’s move to the next step.

```
```
````md
# Embed GitLeaks in CI/CD pipeline

As discussed in the **Secrets Scanning** exercise, we can embed GitLeaks in our CI/CD pipeline. However, remember that it’s important to **locally test a tool before integrating it into the pipeline**. Troubleshooting a tool manually in a local environment is much easier compared to troubleshooting it in a CI/CD system. Additionally, manually exploring the tool in a local environment helps you become familiar with the tool’s options and features.

---

**Click anywhere to copy**

```yaml
image: docker:20.10  # To run all jobs in this pipeline, use the latest docker image

services:
  - docker:dind       # To run all jobs in this pipeline, use a docker image that contains a docker daemon running inside (dind - docker in docker). Reference: https://forum.gitlab.com/t/why-services-docker-dind-is-needed-while-already-having-image-docker/43534

stages:
  - build
  - test
  - release
  - preprod
  - integration
  - prod

build:
  stage: build
  image: python:3.6
  before_script:
   - pip3 install --upgrade virtualenv
  script:
   - virtualenv env                       # Create a virtual environment for the python application
   - source env/bin/activate              # Activate the virtual environment
   - pip install -r requirements.txt      # Install the required third party packages as defined in requirements.txt
   - python manage.py check               # Run checks to ensure the application is working fine

test:
  stage: test
  image: python:3.6
  before_script:
   - pip3 install --upgrade virtualenv
  script:
   - virtualenv env
   - source env/bin/activate
   - pip install -r requirements.txt
   - python manage.py test taskManager

gitleaks:
  stage: build
  script:
    - docker pull zricethezav/gitleaks
    - docker run --user $(id -u):$(id -g) -v $(pwd):/path -w /path zricethezav/gitleaks detect .
````

---

### The Gitleaks job

The **Gitleaks** job is responsible for running GitLeaks. It pulls the GitLeaks Docker image and scans the repository for sensitive information.

---

### Exit Code!

If gitleaks detects any leaks in your repository, it will **exit with code 1 by default**.
This means that your pipeline will fail and stop executing the subsequent jobs.

You can change this behavior by using the `--exit-code` flag in the gitleaks command. For example:

* `--exit-code 2` will make gitleaks exit with code 2 instead of 1.
* You can then use the `when` or `allow_failure` keywords in your pipeline to control the execution of the other jobs based on the gitleaks exit code.

As soon as a change is made to the repository, the pipeline will automatically start executing the jobs.

---

### Note

We’ve discussed different methods of using the tool:

* **Native Installation:** Directly installing the tool on the system.
* **Package Manager or Binary:** Installing via a package manager or using the binary file.
* **Docker:** Running the tool within a Docker container.

✅ In summary:

* All methods are suitable for CI/CD integration.
* **Docker is recommended** for CI/CD as it operates smoothly without dependencies.
* Using the binary file is efficient, avoiding additional dependencies.
* Ultimately, you can choose either method based on your specific situation.

---

We can see the results of this pipeline by visiting:
🔗 [https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/django-nv/pipelines](https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/django-nv/pipelines)

Click on the appropriate job name to see the output.

---

### Adding Output

To add the output of the Gitleaks scan to the GitLab CI/CD pipeline, you can use the **artifacts feature** in GitLab CI/CD. This feature allows you to save files generated during the job and make them accessible for later stages or for review.

Here’s how you can modify the **Gitleaks job** to capture the output:

---

**Click anywhere to copy**

```yaml
gitleaks:
  stage: build
  script:
    - docker pull zricethezav/gitleaks
    - docker run --user $(id -u):$(id -g) -v $(pwd):/path -w /path zricethezav/gitleaks detect . --report-path gitleaks-output.txt
  artifacts:
    paths: [gitleaks-output.txt]
    when: always
    expire_in: one week
```

---

With this configuration, GitLab will save the `gitleaks-output.json` file as an **artifact**, making it available for review or for use in later stages of the CI/CD pipeline.

You can access the saved artifacts in the GitLab pipeline interface for the job.

---

Let’s move to the next step.

```
```
````md
# Allow the job failure

⚠️ **Remember!**

- Except for **DevSecOps-Box**, every other machine closes after **two hours**, even if you are in the middle of the exercise.  
- After two hours, in case of a **404**, you need to refresh the exercise page and click on **Start the Exercise** button to continue working.  

---

In a CI/CD pipeline, especially in the **DevSecOps Maturity Levels 1 and 2**, it’s common to encounter **false positives** from security tools.  
In these cases, failing the build or job may not be the desired outcome.  

To handle such scenarios, we can use the **`allow_failure`** tag to prevent the build or job from failing, even if the tool identifies issues.

```yaml
gitleaks:
  stage: build
  script:
    - docker pull zricethezav/gitleaks
    - docker run --user $(id -u):$(id -g) -v $(pwd):/path -w /path zricethezav/gitleaks detect . --report-path gitleaks-output.json
  artifacts:
    paths: [gitleaks-output.json]
    when: always
    expire_in: one week
  allow_failure: true   #<--- allow the build to fail
````

---

After adding the **`allow_failure`** tag, the pipeline would look like the following:

---

**Click anywhere to copy**

```yaml
image: docker:20.10  # To run all jobs in this pipeline, use the latest docker image

services:
  - docker:dind       # To run all jobs in this pipeline, use a docker image that contains a docker daemon running inside (dind - docker in docker). Reference: https://forum.gitlab.com/t/why-services-docker-dind-is-needed-while-already-having-image-docker/43534

stages:
  - build
  - test
  - release
  - preprod
  - integration
  - prod

build:
  stage: build
  image: python:3.6
  before_script:
   - pip3 install --upgrade virtualenv
  script:
   - virtualenv env                       # Create a virtual environment for the python application
   - source env/bin/activate              # Activate the virtual environment
   - pip install -r requirements.txt      # Install the required third party packages as defined in requirements.txt
   - python manage.py check               # Run checks to ensure the application is working fine

test:
  stage: test
  image: python:3.6
  before_script:
   - pip3 install --upgrade virtualenv
  script:
   - virtualenv env
   - source env/bin/activate
   - pip install -r requirements.txt
   - python manage.py test taskManager

gitleaks:
  stage: build
  script:
    - docker pull zricethezav/gitleaks
    - docker run --user $(id -u):$(id -g) -v $(pwd):/path -w /path zricethezav/gitleaks detect . --report-path gitleaks-output.json
  artifacts:
    paths: [gitleaks-output.json]
    when: always
    expire_in: one week
  allow_failure: true   #<--- allow the build to fail

integration:
  stage: integration
  script:
    - echo "This is an integration step"
    - exit 1
  allow_failure: true # Even if the job fails, continue to the next stages

prod:
  stage: prod
  script:
    - echo "This is a deploy step."
  when: manual # Continuous Delivery
```

---

✅ You will notice that the **gitleaks job failed**, however it didn’t block others from continuing further.

As discussed, any change to the repo **kick starts the pipeline**.

We can see the results of this pipeline by visiting:
🔗 [https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/django-nv/pipelines](https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/django-nv/pipelines)

Click on the appropriate job name to see the output.

```
```
````md
# Static Analysis Using Hadolint

## Download the source code

We will do all the exercises locally first in **DevSecOps-Box**, so let’s start the exercise.

First, we need to download the source code of the project from our git repository.

```bash
git clone https://gitlab.practical-devsecops.training/pdso/django.nv webapp
````

---

Let’s cd into the application so we can scan the app.

```bash
cd webapp
```

We are now in the **webapp** directory.

---

✅ Let’s move to the next step.

```
```
````md
# Installing Hadolint

Hadolint checks a Dockerfile for Docker image best practices. The linter is parsing the Dockerfile into an AST and performs rules on top of the AST.

You can find more details about the project at [https://github.com/hadolint/hadolint](https://github.com/hadolint/hadolint).

---

## Install Hadolint

Let’s install the Hadolint tool on the system to perform static analysis of Dockerfiles.

```bash
wget https://github.com/hadolint/hadolint/releases/download/v1.18.0/hadolint-Linux-x86_64
````

### Command Output

```
--2020-09-01 07:34:54--  https://github.com/hadolint/hadolint/releases/download/v1.18.0/hadolint-Linux-x86_64
Resolving github.com (github.com)... 140.82.113.4
Connecting to github.com (github.com)|140.82.113.4|:443... connected.
HTTP request sent, awaiting response... 302 Found
Location: https://github-production-release-asset-2e65be.s3.amazonaws.com/46234189/65143480-a71b-11ea-8792-d5ccbcc24f7a?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAIWNJYAX4CSVEH53A%2F20200901%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20200901T073447Z&X-Amz-Expires=300&X-Amz-Signature=7f5bf74b9644482dc3271999064cdaebc87f74ebeab6e351b43feba60c7926ba&X-Amz-SignedHeaders=host&actor_id=0&key_id=0&repo_id=46234189&response-content-disposition=attachment%3B%20filename%3Dhadolint-Linux-x86_64&response-content-type=application%2Foctet-stream [following]
--2020-09-01 07:34:54--  https://github-production-release-asset-2e65be.s3.amazonaws.com/46234189/65143480-a71b-11ea-8792-d5ccbcc24f7a?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAIWNJYAX4CSVEH53A%2F20200901%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20200901T073447Z&X-Amz-Expires=300&X-Amz-Signature=7f5bf74b9644482dc3271999064cdaebc87f74ebeab6e351b43feba60c7926ba&X-Amz-SignedHeaders=host&actor_id=0&key_id=0&repo_id=46234189&response-content-disposition=attachment%3B%20filename%3Dhadolint-Linux-x86_64&response-content-type=application%2Foctet-stream
Resolving github-production-release-asset-2e65be.s3.amazonaws.com (github-production-release-asset-2e65be.s3.amazonaws.com)... 52.217.12.220
Connecting to github-production-release-asset-2e65be.s3.amazonaws.com (github-production-release-asset-2e65be.s3.amazonaws.com)|52.217.12.220|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 3232076 (3.1M) [application/octet-stream]
Saving to: ‘hadolint-Linux-x86_64’

hadolint-Linux-x86_ 100%[===================>]   3.08M  --.-KB/s    in 0.05s   

2020-09-01 07:34:54 (59.7 MB/s) - ‘hadolint-Linux-x86_64’ saved [3232076/3232076]
```

---

## Rename and move the file

```bash
mv hadolint-Linux-x86_64 /usr/local/bin/hadolint
```

---

## Make it executable

```bash
chmod +x /usr/local/bin/hadolint
```

---

We have successfully installed **Hadolint** scanner, let’s explore the functionality it provides us.

```bash
hadolint --help
```

### Command Output

```
hadolint - Dockerfile Linter written in Haskell

Usage: hadolint [-v|--version] [-c|--config FILENAME] [-f|--format ARG] 
                [DOCKERFILE...] [--ignore RULECODE] 
                [--trusted-registry REGISTRY (e.g. docker.io)]
  Lint Dockerfile for errors and best practices

Available options:
  -h,--help                Show this help text
  -v,--version             Show version
  -c,--config FILENAME     Path to the configuration file
  -f,--format ARG          The output format for the results [tty | json |
                           checkstyle | codeclimate | codacy] (default: tty)
  --ignore RULECODE        A rule to ignore. If present, the ignore list in the
                           config file is ignored
  --trusted-registry REGISTRY (e.g. docker.io)
                           A docker registry to allow to appear in FROM
                           instructions
```

---

✅ Let’s move to the next step.

```
```
````md
# Run the Hadolint on Dockerfile

As we have learned in DevSecOps best practices, always run and test a tool locally first, before putting it into a CI/CD pipeline.

```bash
hadolint Dockerfile
````

### Command Output

```
Dockerfile:9 DL3018 Pin versions in apk add. Instead of `apk add <package>` use `apk add <package>=<version>`
```

Hadolint ran successfully, and it found one issue i.e., we are not following the best practices for Dockerfile.

---

## Check the contents of the Dockerfile

To fix this, you need to edit Dockerfile and add a specific version of the package.
Let’s check the contents of the Dockerfile using the `cat` command.

```bash
cat -n Dockerfile
```

⚡ **ProTip**
The `-n` option allows you to show line numbers.

### Command Output

```
     1  # FROM specify the underlying operating system you will use to build the image
     2  FROM python:3-alpine3.15
     3
     4  # WORKDIR set the default working directory
     5  WORKDIR /app
     6
     7  # COPY startup script
     8  COPY . .
     9
    10  # RUN any required dependencies by running the necessary commands
    11  RUN apk add --no-cache gawk sed bash grep bc coreutils \
    12      && pip install -r requirements.txt \
    13      && chmod +x run.sh
    14
    15  # EXPOSE port 8000 for communication to/from server
    16  EXPOSE 8000
    17
    18  # CMD specifies the command to be executed when the container starts
    19  CMD ["/app/run.sh"]
```

---

## Fixing the Dockerfile

As per the Hadolint finding `Dockerfile:9 DL3018 Pin versions in apk add.`, the Dockerfile line number 9 has a problem.

So let’s fix it.

**From:**

```
RUN apk add --no-cache gawk sed bash grep bc coreutils
```

**To:**

```
RUN apk add --no-cache gawk=5.0.1-r0 sed=4.7-r0 bash=5.0.11-r1 grep=3.3-r0 bc=1.07.1-r1 coreutils=8.31-r0
```

---

## Apply the fix

Let’s edit the Dockerfile to add these changes.

```bash
cat > Dockerfile <<EOL
# FROM python base image
FROM python:3-alpine3.15

# COPY startup script
COPY . /app

WORKDIR /app

RUN apk add --no-cache gawk=5.0.1-r0 sed=4.7-r0 bash=5.0.11-r1 grep=3.3-r0 bc=1.07.1-r1 coreutils=8.31-r0
RUN pip install -r requirements.txt
RUN chmod +x reset_db.sh && bash reset_db.sh

# EXPOSE port 8000 for communication to/from server
EXPOSE 8000

# CMD specifies the command to execute container starts running.
CMD ["/app/run_app_docker.sh"]
EOL
```

---

## Run Hadolint again

```bash
hadolint Dockerfile
```

### Command Output

```
/webapp# hadolint Dockerfile
/webapp#
```

✅ **Hurray! No issues found.**

```
```
````md
# Static Analysis Using FindSecBugs

## Download the source code

We will do all the exercises locally first in **DevSecOps-Box**, so let’s start the exercise.

First, we need to download the source code of the project from the git repository.

```bash
wget https://github.com/WebGoat/WebGoat/releases/download/v8.1.0/webgoat-server-8.1.0.jar
````

---

In the next step, we will install the **OWASP Find Sec Bugs** tool.

```
```
````md
# Static Analysis Using FindSecBugs

## Install OWASP FindSecBugs

The SpotBugs plugin for security audits of Java web applications and Android applications.

You can find more details about the project at  
👉 https://github.com/find-sec-bugs/find-sec-bugs

---

### Step 1: Verify Java installation

```bash
java -h
````

**Command Output**

```
bash: java: command not found
```

It seems Java is not installed, so we need to install Java.

---

### Step 2: Install Java

```bash
apt update && apt install openjdk-8-jre -y
```

**Command Output**

```
Reading package lists... Done
Building dependency tree       
Reading state information... Done
The following additional packages will be installed:
  adwaita-icon-theme at-spi2-core ca-certificates-java fontconfig fontconfig-config fonts-dejavu-core fonts-dejavu-extra gtk-update-icon-cache hicolor-icon-theme humanity-icon-theme java-common libapparmor1 libasound2 libasound2-data libasyncns0 libatk-bridge2.0-0
  libatk-wrapper-java libatk-wrapper-java-jni libatk1.0-0 libatk1.0-data libatspi2.0-0 libavahi-client3 libavahi-common-data libavahi-common3 libcairo2 libcroco3 libcups2 libdatrie1 libdrm-amdgpu1 libdrm-common libdrm-intel1 libdrm-nouveau2 libdrm-radeon1 libdrm2 libelf1 libflac8
  libfontconfig1 libfontenc1 libfreetype6 libgail-common libgail18 libgdk-pixbuf2.0-0 libgdk-pixbuf2.0-bin libgdk-pixbuf2.0-common libgif7 libgl1 libgl1-mesa-dri libgl1-mesa-glx libglapi-mesa libglvnd0 libglx-mesa0 libglx0 libgraphite2-3 libgtk2.0-0 libgtk2.0-bin libgtk2.0-common
  libharfbuzz0b libice6 libicu60 libjbig0 libjpeg-turbo8 libjpeg8 liblcms2-2 libllvm10 libnspr4 libnss3 libogg0 libpango-1.0-0 libpangocairo-1.0-0 libpangoft2-1.0-0 libpciaccess0 libpcsclite1 libpixman-1-0 libpng16-16 libpulse0 librsvg2-2 librsvg2-common libsensors4 libsm6
  libsndfile1 libthai-data libthai0 libtiff5 libvorbis0a libvorbisenc2 libx11-6 libx11-data libx11-xcb1 libxau6 libxaw7 libxcb-dri2-0 libxcb-dri3-0 libxcb-glx0 libxcb-present0 libxcb-render0 libxcb-shape0 libxcb-shm0 libxcb-sync1 libxcb1 libxcomposite1 libxcursor1 libxdamage1
  libxdmcp6 libxext6 libxfixes3 libxft2 libxi6 libxinerama1 libxml2 libxmu6 libxmuu1 libxpm4 libxrandr2 libxrender1 libxshmfence1 libxt6 libxtst6 libxv1 libxxf86dga1 libxxf86vm1 multiarch-support openjdk-8-jre-headless shared-mime-info ubuntu-mono x11-common x11-utils
Suggested packages:
...
[SNIP]
```

---

### Step 3: Download FindSecBugs

```bash
wget https://github.com/find-sec-bugs/find-sec-bugs/releases/download/version-1.9.0/findsecbugs-cli-1.9.0-fix2.zip && unzip findsecbugs-cli-1.9.0-fix2.zip -d /opt/
```

---

### Step 4: Fix line endings in the shell script

```bash
sed -i -e 's/\r$//' /opt/findsecbugs.sh
```

---

### Step 5: Make it executable

```bash
chmod +x /opt/findsecbugs.sh
```

---

### Step 6: Add FindSecBugs to PATH

```bash
export PATH=/opt/:$PATH
```

---

✅ We are now ready.
Let’s move to the next step.

```
```
```md
Run the SAST Scan
Let’s explore what options this tool provides us.



findsecbugs.sh -h

Command Output
SLF4J: No SLF4J providers were found.
SLF4J: Defaulting to no-operation (NOP) logger implementation
SLF4J: See http://www.slf4j.org/codes.html#noProviders for further details.
Usage: findbugs [general options] -textui [command line options...] [jar/zip/class files, directories...]
General options:
  -jvmArgs args    Pass args to JVM
  -maxHeap size    Maximum Java heap size in megabytes (default=768)
  -javahome <dir>  Specify location of JRE
  General FindBugs options:
    -project <project>                       analyze given project
    -home <home directory>                   specify SpotBugs home directory
    -pluginList <jar1[:jar2...]>             specify list of plugin Jar files to load
    -effort[:min|less|default|more|max]      set analysis effort level
    -adjustExperimental                      lower priority of experimental Bug Patterns
    -workHard                                ensure analysis effort is at least 'default'
    -conserveSpace                           same as -effort:min (for backward compatibility)
    -showPlugins                             show list of available detector plugins
    -userPrefs <filename>                    user preferences file, e.g /path/to/project/.settings/edu.umd.cs.findbugs.core.prefs for Eclipse projects
  Output options:
    -timestampNow                            set timestamp of results to be current time
    -quiet                                   suppress error messages
    -longBugCodes                            report long bug codes
    -progress                                display progress in terminal window
    -release <release name>                  set the release name of the analyzed application
    -experimental                            report of any confidence level including experimental bug patterns
    -low                                     report warnings of any confidence level
    -medium                                  report only medium and high confidence warnings [default]
    -high                                    report only high confidence warnings
    -maxRank <rank>                          only report issues with a bug rank at least as scary as that provided
    -dontCombineWarnings                     Don't combine warnings that differ only in line number
    -sortByClass                             sort warnings by class
    -xml[:withMessages]                      XML output (optionally with messages)
    -xdocs                                   xdoc XML output to use with Apache Maven
    -html[:stylesheet]                       Generate HTML output (default stylesheet is default.xsl)
    -emacs                                   Use emacs reporting format
    -relaxed                                 Relaxed reporting mode (more false positives!)
    -train[:outputDir]                       Save training data (experimental); output dir defaults to '.'
    -useTraining[:inputDir]                  Use training data (experimental); input dir defaults to '.'
    -redoAnalysis <filename>                 Redo analysis using configuration from previous analysis
    -sourceInfo <filename>                   Specify source info file (line numbers for fields/classes)
    -projectName <project name>              Descriptive name of project
    -reanalyze <filename>                    redo analysis in provided file
    -output <filename>                       Save output in named file
    -nested[:true|false]                     analyze nested jar/zip archives (default=true)
  Output filtering options:
    -bugCategories <cat1[,cat2...]>          only report bugs in given categories
    -onlyAnalyze <classes/packages>          only analyze given classes and packages; end with .* to indicate classes in a package, .- to indicate a package prefix
    -excludeBugs <baseline bugs>             exclude bugs that are also reported in the baseline xml output
    -exclude <filter file>                   exclude bugs matching given filter
    -include <filter file>                   include only bugs matching given filter
    -applySuppression                        Exclude any bugs that match suppression filter loaded from fbp file
  Detector (visitor) configuration options:
    -visitors <v1[,v2...]>                   run only named visitors
    -omitVisitors <v1[,v2...]>               omit named visitors
    -chooseVisitors <+v1,-v2,...>            selectively enable/disable detectors
    -choosePlugins <+p1,-p2,...>             selectively enable/disable plugins
    -adjustPriority <v1=(raise|lower)[,...]> raise/lower priority of warnings for given visitor(s)
  Project configuration options:
    -auxclasspath <classpath>                set aux classpath for analysis
    -auxclasspathFromInput                   read aux classpath from standard input
    -auxclasspathFromFile <filepath>         read aux classpaths from a designated file
    -sourcepath <source path>                set source path for analyzed classes
    -exitcode                                set exit code of process
    -noClassOk                               output empty warning file if no classes are specified
    -xargs                                   get list of classfiles/jarfiles from standard input rather than command line
    -analyzeFromFile <filepath>              get the list of class/jar files from a designated file
    -bugReporters <name,name2,-name3>        bug reporter decorators to explicitly enable/disable
    -printConfiguration                      print configuration and exit, without running analysis
    -version                                 print version, check for updates and exit, without running analysis
Perform the scan using the following command.



findsecbugs.sh -progress -html -output findsecbugs-report.html webgoat-server-8.1.0.jar

Note: This will take several minutes to complete.

Command Output
SLF4J: No SLF4J providers were found.
SLF4J: Defaulting to no-operation (NOP) logger implementation
SLF4J: See http://www.slf4j.org/codes.html#noProviders for further details.
Scanning archives (1 / 1)
2 analysis passes to perform
Pass 1: Analyzing classes (46404 / 46404) - 100% complete
Pass 2: Analyzing classes (20863 / 44219) - 47% complete
                           ...SNIP...
 [-1]==> +============================
| /!\ Warning : The taint debugging is not fully activated.
| [[ Stack ]]
Oups Accessing TOP or BOTTOM frame! @ 643

Dataflow (block 1): start fact is +============================
| /!\ Warning : The taint debugging is not fully activated.
| [[ Stack ]]
Oups Accessing TOP or BOTTOM frame!
Dataflow (block 1): orig result is +============================
| /!\ Warning : The taint debugging is not fully activated.
| [[ Stack ]]
Oups Accessing TOP or BOTTOM frame!
Dataflow (block 1): result is +============================
| /!\ Warning : The taint debugging is not fully activated.
| [[ Stack ]]
Oups Accessing TOP or BOTTOM frame! @ timestamp 19
----------------------------------------------------------------------
com.h3xstream.findsecbugs.taintanalysis.TaintDataflow iteration: 106, timestamp: 643
org.aspectj.weaver.AjcMemberMaker.interConstructor(Lorg/aspectj/weaver/ResolvedType;Lorg/aspectj/weaver/ResolvedMember;Lorg/aspectj/weaver/UnresolvedType;)Lorg/aspectj/weaver/ResolvedMember;
----------------------------------------------------------------------
Pass 2: Analyzing classes (23841 / 44219) - 53% complete
Pass 2: Analyzing classes (27720 / 44219) - 62% complete
Pass 2: Analyzing classes (27722 / 44219) - 62% complete
Pass 2: Analyzing classes (38412 / 44219) - 86% complete
As we have learned in the DevSecOps Gospel, we should save the output in a machine-readable format so the output can be parsed by the machines.

Do you think the above command is following the DevSecOps best practices?

What can we do to improve it further?
```
```md
Static Analysis Using SpotBugs
Installing SpotBugs
SpotBugs is an application that employs static analysis to detect bugs in Java code. SpotBugs is a free software that falls under the confines of the Lesser GNU Public License.

More details are available at https://spotbugs.github.io/

Initially, we will fetch the SpotBugs zip file from the GitHub resource for the purpose of installing SpotBugs.

wget https://github.com/spotbugs/spotbugs/releases/download/4.7.3/spotbugs-4.7.3.zip
unzip spotbugs-4.7.3.zip

Subsequently, we will integrate a variable into the .bashrc file with the intention of running spotbugs directly.

echo -e "\nexport PATH=\$PATH:/spotbugs-4.7.3/bin" >> ~/.bashrc
source ~/.bashrc
chmod +x /spotbugs-4.7.3/bin/spotbugs

We can now go ahead and execute the spotbugs command to determine if SpotBugs has been successfully installed.

spotbugs -h

Command Output
/spotbugs-4.7.3/bin/spotbugs: 209: exec: java: not found
Installation of Java

The above output signifies the fact that java is not detected. Consequently, we need to install java as per the following instructions.

apt update && apt install default-jdk -y

Excellent! The installation of java on our system has been accomplished.

We can now reinitiate the primary command for SpotBugs.

spotbugs -h

Command Output
Oct 08, 2023 6:34:11 AM edu.umd.cs.findbugs.FindBugs showSynopsis
WARNING: Usage: findbugs [general options] -textui [command line options...] [jar/zip/class files, directories...]
General options:
  -jvmArgs args    Pass args to JVM
  -maxHeap size    Maximum Java heap size in megabytes (default=768)
  -javahome <dir>  Specify location of JRE
  General FindBugs options:
    -project <project>                       analyze given project
    -home <home directory>                   specify SpotBugs home directory
    -pluginList <jar1[:jar2...]>             specify list of plugin Jar files to load
    -effort[:min|less|default|more|max]      set analysis effort level
    -adjustExperimental                      lower priority of experimental Bug Patterns
    -workHard                                ensure analysis effort is at least 'default'
    -conserveSpace                           same as -effort:min (for backward compatibility)
    -showPlugins                             show list of available detector plugins
    -userPrefs <filename>                    user preferences file, e.g /path/to/project/.settings/edu.umd.cs.findbugs.core.prefs for Eclipse projects
  Output options:
    -timestampNow                            set timestamp of results to be current time
    -quiet                                   suppress error messages
    -longBugCodes                            report long bug codes
    -progress                                display progress in terminal window
    -release <release name>                  set the release name of the analyzed application
    -experimental                            report of any confidence level including experimental bug patterns
    -low                                     report warnings of any confidence level
    -medium                                  report only medium and high confidence warnings [default]
    -high                                    report only high confidence warnings
    -maxRank <rank>                          only report issues with a bug rank at least as scary as that provided
    -dontCombineWarnings                     Don't combine warnings that differ only in line number
    -sortByClass                             sort warnings by class
    -xml[:withMessages]                      XML output (optionally with messages)
    -xdocs                                   xdoc XML output to use with Apache Maven
    -sarif                                   SARIF 2.1.0 output
    -html[:stylesheet]                       Generate HTML output (default stylesheet is default.xsl)
    -emacs                                   Use emacs reporting format
    -relaxed                                 Relaxed reporting mode (more false positives!)
    -train[:outputDir]                       Save training data (experimental); output dir defaults to '.'
    -useTraining[:inputDir]                  Use training data (experimental); input dir defaults to '.'
    -redoAnalysis <filename>                 Redo analysis using configuration from previous analysis
    -sourceInfo <filename>                   Specify source info file (line numbers for fields/classes)
    -projectName <project name>              Descriptive name of project
    -reanalyze <filename>                    redo analysis in provided file
    -output <filename>                       Save output in named file
    -nested[:true|false]                     analyze nested jar/zip archives (default=true)
  Output filtering options:
    -bugCategories <cat1[,cat2...]>          only report bugs in given categories
    -onlyAnalyze <classes/packages>          only analyze given classes and packages; end with .* to indicate classes in a package, .- to indicate a package prefix
    -excludeBugs <baseline bugs>             exclude bugs that are also reported in the baseline xml output
    -exclude <filter file>                   exclude bugs matching given filter
    -include <filter file>                   include only bugs matching given filter
    -applySuppression                        Exclude any bugs that match suppression filter loaded from fbp file
  Detector (visitor) configuration options:
    -visitors <v1[,v2...]>                   run only named visitors
    -omitVisitors <v1[,v2...]>               omit named visitors
    -chooseVisitors <+v1,-v2,...>            selectively enable/disable detectors
    -choosePlugins <+p1,-p2,...>             selectively enable/disable plugins
    -adjustPriority <v1=(raise|lower)[,...]> raise/lower priority of warnings for given visitor(s)
  Project configuration options:
    -auxclasspath <classpath>                set aux classpath for analysis
    -auxclasspathFromInput                   read aux classpath from standard input
    -auxclasspathFromFile <filepath>         read aux classpaths from a designated file
    -sourcepath <source path>                set source path for analyzed classes
    -exitcode                                set exit code of process
    -noClassOk                               output empty warning file if no classes are specified
    -xargs                                   get list of classfiles/jarfiles from standard input rather than command line
    -analyzeFromFile <filepath>              get the list of class/jar files from a designated file
    -bugReporters <name,name2,-name3>        bug reporter decorators to explicitly enable/disable
    -printConfiguration                      print configuration and exit, without running analysis
    -version                                 print version, check for updates and exit, without running analysis
Subsequently, we need to focus on the next step to further complete the workings of SpotBugs.
```
```md
Downloading The Source Code
We will do all the exercises locally first in DevSecOps-Box, so let’s start the exercise.

First, we need to download the source code of the project from the git repository.

git clone https://gitlab.practical-devsecops.training/pdso/android-java.git javaapp

Command Output
Cloning into 'javaapp'...
remote: Enumerating objects: 176, done.
remote: Total 176 (delta 0), reused 0 (delta 0), pack-reused 176
Receiving objects: 100% (176/176), 6.34 MiB | 36.87 MiB/s, done.
Resolving deltas: 100% (44/44), done.

In the next step, we will run the SpotBugs tool.
```
```md
Running Spotbugs
The utility of SpotBugs scans lies in the general review they can perform on your Java code.

For instance, during a standard scan, SpotBugs seeks out potential bugs and issues that might impact the correctness, performance, and the overall structural quality of your program.

Let’s perform a scan on the javaapp directory.

Initially, we will navigate to the directory.

cd javaapp

Subsequently, we can conduct the standard scan using SpotBugs and the -textui option to display the output.

spotbugs -textui .

Reminder

SpotBugs scrutinizes the code related to java. These include:

.class
.jar
Repository possessing a Java package
Command Output
M B Eq: com.google.common.collect.Interners$WeakInterner$1.equals(Object) checks for operand being a Interners$WeakInterner$InternReference  At Interners.java:[line 98]
M D NP: Method com.google.common.collect.Lists$CharSequenceAsList.equals(Object) overrides the nullness annotation of parameter o in an incompatible way  At Lists.java:[lines 688-704]
......[SNIP]......

M M IS: Inconsistent synchronization of com.google.common.collect.ComputingConcurrentHashMap$ComputingValueReference.computedReference; locked 60% of time  Unsynchronized access at ComputingConcurrentHashMap.java:[line 291]
The following classes needed for analysis were missing:
  android.app.Activity
  android.database.ContentObserver
... 
The aforementioned output originates from a static code analysis of a java program and it discloses numerous potential issues:

Issue	Description
Precise Type Comparison in equals() Method	The code contains a method that performs a specific type comparison within an equals() method. This can lead to issues when comparing the object with other similar but not identical types.
Incompatible Nullness Parameter Override	A method appears to override the nullness parameter in an incompatible manner compared to its original design. This can result in exceptions when encountering null values during the function execution.
Inconsistent Synchronization	Synchronization of an object in a certain class appears to be inconsistent, as it is locked only part of the time. This inconsistency could lead to data handling issues when multiple threads attempt to access or modify the object simultaneously.
Missing Classes in Classpath	Some of the classes required for the analysis were not found in the classpath. These classes are associated with Android, suggesting that an Android project analysis is being attempted outside the Android environment, causing failure in Android library dependencies.
How can we interpret the earlier output from Spotbugs?
Let’s generate the HTML output file from the SpotBugs scan.

spotbugs -html -output spotbugs-output.html .

Next, if you execute the ls command, you may see the spotbugs-output.html file.

How can we view the results clearly?

Preparation

Open a second terminal by clicking the + button.

2nd-terminal

Once the second terminal is open, execute the following command to run the python web server.

cd javaapp
python -m http.server 80

Command Output
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
After the python web server has started, try loading the webpage in the browser.

Access the webpage by visiting this URL - https://devsecops-box-kr6k1mdm.lab.practical-devsecops.training

The content may appear as follows; click on the spotbugs-output.html file on the page.

page

Next, you will see the following view for the HTML output page.

html-page

Disclaimer

We will not delve into the details of the HTML output. You can explore it on your own as the output also provides detailed information based on SpotBugs documentation. You can view these details at https://devsecops-box-kr6k1mdm.lab.practical-devsecops.training/spotbugs-output.html#Details

Do you think the above command is following the DevSecOps best practices?

What can we do to improve it further?
```
```md
SpotBugs Advanced Scanning Options
In this scenario, we will delve deeper into the options provided by SpotBugs. We will explore how to reduce error, display progress, report high confidence, and perform sorting by class in Java scanning.

Minimizing Error Message Information
First, let’s understand how we can eliminate the error message information about the code object from SpotBugs scanning.

We incorporate the -quiet option to decrease the likelihood of error output.

As a starting point, we will execute a basic scan using the previous command.



Upon completing the scan, it will display the following error message from the javaapp project.

Command Output
The following classes needed for analysis were missing:
  android.app.Activity
  android.database.ContentObserver
  android.database.CrossProcessCursor
  android.os.Binder
  android.database.Cursor
  android.os.Parcelable$Creator
  android.database.CursorWindow

.........[SNIP].........

  android.content.res.AssetManager
  android.os.Debug
  android.text.TextUtils
  android.database.DataSetObserver
  android.content.ContentResolver
  android.net.Uri
  android.database.DataSetObservable
  android.database.ContentObservable
  android.os.Message
  android.widget.Toast
  android.content.OperationApplicationException
We want to eliminate the above output so that the SpotBugs result only shows the vulnerability outcome rather than displaying error results.

spotbugs -textui -quiet .

Perfect! The output from the command above will appear as follows:

quiet-output

Displaying The Scan Progress
Now, let’s try another option that displays the percentage of progress and provides some details for each result in the SpotBugs scan.

To showcase the progress, we utilize the -progress option

We will scan the javaapp project while displaying the scanning progress.

spotbugs -textui -progress .

The progress will be exhibited in the following output:

Command Output
Scanning archives (1 / 1)
2 analysis passes to perform
Pass 1: Analyzing classes (880 / 1337) - 65% completeM B Eq: com.google.common.collect.Interners$WeakInterner$1.equals(Object) checks for operand being a Interners$WeakInterner$InternReference  At Interners.java:[line 98]
Pass 1: Analyzing classes (1337 / 1337) - 100% complete
Pass 2: Analyzing classes (16 / 1009) - 01% completeM D NP: Method com.google.common.collect.Lists$CharSequenceAsList.equals(Object) overrides the nullness annotation of parameter o in an incompatible way  At Lists.java:[lines 688-704]
Pass 2: Analyzing classes (30 / 1009) - 02% completeM B It: com.google.common.collect.LinkedListMultimap$ValueForKeyIterator.next() cannot throw NoSuchElementException  At LinkedListMultimap.java:[lines 376-380]

......[SNIP]......

Pass 2: Analyzing classes (995 / 1009) - 98% completeM D NP: object must be non-null but is marked as nullable  At ForwardingMap.java:[line 276]
M D NP: Method com.google.common.collect.ForwardingMap.equals(Object) overrides the nullness annotation of parameter object in an incompatible way  At ForwardingMap.java:[line 130]
Pass 2: Analyzing classes (1009 / 1009) - 100% completeM C UwF: Unwritten field: com.google.common.collect.MapMaker.cleanupExecutor  At MapMaker.java:[line 489]

......[SNIP]......
As seen from the above result, SpotBugs provides detailed information.

Note

The -progress option provides the following additional information:

Analyzing classes (xxx / xxxx) informs us that SpotBugs is analyzing each class in every Java code.
Percentage information will provide information on the percentage of progress until the scanning process is completed.
Reporting Only High Confidence Warnings
Next, we will locate the high severity level for the javaapp project.

The process will display only those warnings identified as having a high severity level using the -high option.

Let’s begin the scan on the javapp project.

spotbugs -textui -high .

Command Output
H I Dm: Found reliance on default encoding in org.apache.commons.codec.binary.Hex.decode(byte[]): new String(byte[])  At Hex.java:[line 131]
H I Dm: Found reliance on default encoding in org.apache.commons.codec.binary.Hex.encode(byte[]): String.getBytes()  At Hex.java:[line 168]
H I Dm: Found reliance on default encoding in org.apache.commons.codec.binary.Hex.encode(Object): String.getBytes()  At Hex.java:[line 184]
H I Dm: Found reliance on default encoding in org.gradle.wrapper.PathAssembler.getMd5Hash(String): String.getBytes()  At PathAssembler.java:[line 59]
H I Dm: Found reliance on default encoding in org.apache.commons.codec.digest.DigestUtils.md5(String): String.getBytes()  At DigestUtils.java:[line 86]
H I Dm: Found reliance on default encoding in org.apache.commons.codec.digest.DigestUtils.sha(String): String.getBytes()  At DigestUtils.java:[line 130]
H I Dm: Found reliance on default encoding in org.gradle.wrapper.Install.setExecutablePermissions(File): new java.io.InputStreamReader(InputStream)  At Install.java:[line 116]

......[SNIP]......
Excellent! The output only displays the high severity level as shown above.

Note

Depending on the severity level desired, we can use the -low, -medium, or -high options.

Pay Attention

Please use the -h option to see further details regarding the functionality of other options.

spotbugs -h

Sorting By Class
Lastly, we will sort the output according to each Java class to make the output more readable.

To sort the output by class, we can use the -sortByClass option.

Let’s proceed to scan the javaapp project.

spotbugs -textui -sortByClass .

Command Output
......[SNIP]......

M D NP: Method com.google.common.base.Functions$FunctionComposition.equals(Object) overrides the nullness annotation of parameter obj in an incompatible way  At Functions.java:[lines 208-212]
M D NP: Method com.google.common.base.Functions$FunctionForMapNoDefault.equals(Object) overrides the nullness annotation of parameter o in an incompatible way  At Functions.java:[lines 114-118]
M D NP: Method com.google.common.base.Functions$PredicateFunction.equals(Object) overrides the nullness annotation of parameter obj in an incompatible way  At Functions.java:[lines 250-254]
M D NP: o must be non-null but is marked as nullable  At Functions.java:[lines 60-61]
H D NP: first must be non-null but is marked as nullable  At Joiner.java:[line 117]
H D NP: second must be non-null but is marked as nullable  At Joiner.java:[line 117]
H D NP: first must be non-null but is marked as nullable  At Joiner.java:[line 150]

......[SNIP]......
As seen above, SpotBugs organizes the output nicely by classifying each function.

Explanation

The class output is first classified, and SpotBugs informs us about everything in the Functions.java class.

Next, SpotBugs reports on the vulnerabilities found in the Joiner.java class. The scanning will continue to classify similar classes accordingly.

Combining the Scanning
We now understand how to perform a more detailed scan in SpotBugs, which aids in the easier analysis of Java codes.

Let’s combine these methods so that the output is more easily analyzed, focusing on the high level in our analysis.

spotbugs -textui -quiet -progress -high -sortByClass .

Command Output
Scanning archives (1 / 1)
2 analysis passes to perform
Pass 1: Analyzing classes (1337 / 1337) - 100% complete
Pass 2: Analyzing classes (1009 / 1009) - 100% complete
Done with analysis
H D NP: first must be non-null but is marked as nullable  At Joiner.java:[line 117]
H D NP: second must be non-null but is marked as nullable  At Joiner.java:[line 117]
H D NP: first must be non-null but is marked as nullable  At Joiner.java:[line 150]
H D NP: second must be non-null but is marked as nullable  At Joiner.java:[line 150]
H D NP: first must be non-null but is marked as nullable  At Joiner.java:[line 174]

......[SNIP]......

H I Dm: Found reliance on default encoding in org.gradle.wrapper.PathAssembler.getMd5Hash(String): String.getBytes()  At PathAssembler.java:[line 59]
Perfect! The combination runs smoothly and now you can analyze the SpotBugs output with a comprehensible display.

Let’s continue to the next step.
```

```md
# Conclusion

By learning to use **SpotBugs**, you’ve now added another tool in your arsenal to write bug-free, efficient, and secure Java code.  
Keep scanning regularly to maintain your code’s health!

---

## 🔄 Scan Regularly
Static analysis tools like **SpotBugs** are most effective when used **regularly**.  
Don’t wait for the project end to start scanning your code — make it a part of your **development cycle**!

---

## 📚 Next Steps
If you want to dive in deeper, there are plenty of resources online that help you understand SpotBugs from **basic to advanced** levels.

---

## 🏋️ Exercise
👉 Do you want to try **multiple scans on different classes**?
```
````md
# Static Analysis Using njsscan

## Download the source code
We will do all the exercises locally first in DevSecOps-Box, so let’s start the exercise.

First, we need to download the source code of the project from our git repository.

```bash
git clone https://gitlab.practical-devsecops.training/pdso/nodejs.git webapp
````

Let’s cd into the application so we can scan the app.

```bash
cd webapp
```

We are now in the webapp directory.

Let’s move to the next step.

```
```
````md
# Installing njsscan

njsscan is a semantic aware SAST tool that can find insecure code patterns in your Node.js applications using a simple pattern matcher from libsast and pattern search tool like semgrep (lightweight static analysis for many languages).

You can find more details about the project at https://github.com/ajinabraham/njsscan.

Let’s install njsscan on the system to perform static analysis along with libsast for required module package in njsscan.

```bash
pip3 install njsscan libsast
````

**Note**

Please ignore when you notify the error with this output:

**Command Output**

```
ERROR: semgrep 0.104.0 has requirement attrs~=21.3, but you'll have attrs 23.1.0 which is incompatible.
ERROR: semgrep 0.104.0 has requirement urllib3~=1.26, but you'll have urllib3 2.0.3 which is incompatibl
```

We have successfully installed njsscan, lets explore the functionality it provides us.

```bash
njsscan --help
```

**Command Output**

```
usage: njsscan [-h] [--json] [--sarif] [--sonarqube] [--html] [-o OUTPUT]
               [-c CONFIG] [--missing-controls] [-w] [-v]
               [path [path ...]]

positional arguments:
  path                  Path can be file(s) or directories with source code

optional arguments:
  -h, --help            show this help message and exit
  --json                set output format as JSON
  --sarif               set output format as SARIF 2.1.0
  --sonarqube           set output format compatible with SonarQube
  --html                set output format as HTML
  -o OUTPUT, --output OUTPUT
                        output filename to save the result
  -c CONFIG, --config CONFIG
                        Location to .njsscan config file
  --missing-controls    enable missing security controls check
  -w, --exit-warning    non zero exit code on warning
  -v, --version         show njsscan version
```

Let’s move to the next step.

```
```
````md
# Run the scanner

As we have learned in DevSecOps Gospel, we would like to store the content in a JSON file.

```bash
njsscan --json -o output.json .
````

**Note**

If you encounter this error:

**Command Output**

```
/usr/lib/python3/dist-packages/requests/__init__.py:89: RequestsDependencyWarning: urllib3 (2.2.3) or chardet (3.0.4) doesn't match a supported version!
warnings.warn("urllib3 ({}) or chardet ({}) doesn't match a supported "
```

You can resolve it by upgrading the required packages:

```bash
pip install --upgrade urllib3 chardet requests
```

njsscan ran successfully and it found some security issues and false positives.

```bash
cat output.json
```

**Command Output**

```
  ...
  ...
    "cookie_session_no_secure": {
      "files": [
        {
          "file_path": "server.js",
          "match_lines": [
            78,
            102
          ],
          "match_position": [
            13,
            7
          ],
          "match_string": "    app.use(session({\n        // genid: (req) => {\n        //    return genuuid() // use UUIDs for session IDs\n        //},\n        secret: cookieSecret,\n        // Both mandatory in Express v4\n        saveUninitialized: true,\n        resave: true\n        /*\n        // Fix for A5 - Security MisConfig\n        // Use generic cookie name\n        key: \"sessionId\",\n        */\n\n        /*\n        // Fix for A3 - XSS\n        // TODO: Add \"maxAge\"\n        cookie: {\n            httpOnly: true\n            // Remember to start an HTTPS server to get this working\n            // secure: true\n        }\n        */\n\n    }));"
        }
      ],
      "metadata": {
        "cwe": "CWE-614: Sensitive Cookie in HTTPS Session Without 'Secure' Attribute",
        "description": "Default session middleware settings: `secure` not set. It ensures the browser only sends the cookie over HTTPS.",
        "owasp": "A2: Broken Authentication",
        "severity": "WARNING"
      }
    },
  ...
  ...
```

You can explore an appropriate njsscan options to reduce False Positives (FP) here

```
```
````md
# Code Quality Analysis With pylint

## Download the source code

We will do all the exercises locally first in DevSecOps-Box, so let’s start the exercise.

First, we need to download the source code of the project from our git repository.

```bash
git clone https://gitlab.practical-devsecops.training/pdso/django.nv webapp
````

**Command Output**

```
Cloning into 'webapp'...
warning: redirecting to https://gitlab.practical-devsecops.training/pdso/django.nv.git/
remote: Enumerating objects: 228, done.
remote: Total 228 (delta 0), reused 0 (delta 0), pack-reused 228
Receiving objects: 100% (228/228), 1.03 MiB | 1.04 MiB/s, done.
Resolving deltas: 100% (86/86), done.
```

Let’s cd into the application so we can scan the app.

```bash
cd webapp
```

We are now in the webapp directory

Let’s move to the next step.

```
```
````md
# Install Pylint Tool

Pylint is a static code analysis tool for Python which looks for programming errors, helps enforce a coding standard, sniffs out code smells and offers simple refactoring suggestions.

You can find more details about the project at [https://github.com/PyCQA/pylint](https://github.com/PyCQA/pylint).

Let’s install pylint on the system to perform static analysis.

```bash
pip3 install pylint
````

**Command Output**

```
Collecting pylint
  Downloading pylint-2.12.2-py3-none-any.whl (414 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 414.8/414.8 KB 22.3 MB/s eta 0:00:00
Collecting platformdirs>=2.2.0
  Downloading platformdirs-2.4.1-py3-none-any.whl (14 kB)
Collecting isort<6,>=4.2.5
  Downloading isort-5.10.1-py3-none-any.whl (103 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 103.4/103.4 KB 26.5 MB/s eta 0:00:00
Collecting mccabe<0.7,>=0.6
  Downloading mccabe-0.6.1-py2.py3-none-any.whl (8.6 kB)
Collecting typing-extensions>=3.10.0
  Using cached typing_extensions-4.0.1-py3-none-any.whl (22 kB)
Collecting toml>=0.9.2
  Downloading toml-0.10.2-py2.py3-none-any.whl (16 kB)
Collecting astroid<2.10,>=2.9.0
  Downloading astroid-2.9.3-py3-none-any.whl (254 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 254.5/254.5 KB 6.0 MB/s eta 0:00:00
Requirement already satisfied: setuptools>=20.0 in /usr/local/lib/python3.8/dist-packages (from astroid<2.10,>=2.9.0->pylint) (60.6.0)
Collecting lazy-object-proxy>=1.4.0
  Downloading lazy_object_proxy-1.7.1-cp38-cp38-manylinux_2_5_x86_64.manylinux1_x86_64.manylinux_2_17_x86_64.manylinux2014_x86_64.whl (60 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 60.5/60.5 KB 10.6 MB/s eta 0:00:00
Collecting wrapt<1.14,>=1.11
  Downloading wrapt-1.13.3-cp38-cp38-manylinux_2_5_x86_64.manylinux1_x86_64.manylinux_2_12_x86_64.manylinux2010_x86_64.whl (84 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 84.6/84.6 KB 2.9 MB/s eta 0:00:00
Installing collected packages: mccabe, wrapt, typing-extensions, toml, platformdirs, lazy-object-proxy, isort, astroid, pylint
Successfully installed astroid-2.9.3 isort-5.10.1 lazy-object-proxy-1.7.1 mccabe-0.6.1 platformdirs-2.4.1 pylint-2.12.2 toml-0.10.2 typing-extensions-4.0.1 wrapt-1.13.3
WARNING: Running pip as the 'root' user can result in broken permissions and conflicting behaviour with the system package manager. It is recommended to use a virtual environment instead: https://pip.pypa.io/warnings/venv
```

We have successfully installed pylint, let’s explore its functionality now.

```bash
pylint --help
```

**Command Output**

```
Usage: pylint [options]

Options:
  -h, --help            show this help message and exit
  --long-help           more verbose help.

  Master:
    --init-hook=<code>  Python code to execute, usually for sys.path
                        manipulation such as pygtk.require().
    -E, --errors-only   In error mode, checkers without error messages are
                        disabled and for others, only the ERROR messages are
                        displayed, and no reports are done by default.
    -v, --verbose       In verbose mode, extra non-checker-related info will
                        be displayed.
    --enable-all-extensions
                        Load and enable all available extensions. Use --list-
                        extensions to see a list all available extensions.
    --ignore=<file>[,<file>...]
                        Files or directories to be skipped. They should be
                        base names, not paths. [current: CVS]
    --ignore-patterns=<pattern>[,<pattern>...]
                        Files or directories matching the regex patterns are
                        skipped. The regex matches against base names, not
                        paths. [current: none]
    --ignore-paths=<pattern>[,<pattern>...]
                        Add files or directories matching the regex patterns
                        to the ignore-list. The regex matches against paths
                        and can be in Posix or Windows format. [current: none]
    --persistent=<y or n>
                        Pickle collected data for later comparisons. [current:
                        yes]
    --load-plugins=<modules>
                        List of plugins (as comma separated values of python
                        module names) to load, usually to register additional
                        checkers. [current: none]
    --fail-under=<score>
                        Specify a score threshold to be exceeded before
                        program exits with error. [current: 10.0]
    --fail-on=<msg ids>
                        Return non-zero exit code if any of these
                        messages/categories are detected, even if score is
                        above --fail-under value. Syntax same as enable.
                        Messages specified are enabled, while categories only
                        check already-enabled messages. [current: none]
    -j <n-processes>, --jobs=<n-processes>
                        Use multiple processes to speed up Pylint. Specifying
                        0 will auto-detect the number of processors available
                        to use. [current: 1]
    --limit-inference-results=<number-of-results>
                        Control the amount of potential inferred values when
                        inferring a single object. This can help the
                        performance when dealing with large functions or
                        complex, nested conditions.  [current: 100]
    --extension-pkg-allow-list=<pkg[,pkg]>
                        A comma-separated list of package or module names from
                        where C extensions may be loaded. Extensions are
                        loading into the active Python interpreter and may run
                        arbitrary code. [current: none]
    --extension-pkg-whitelist=<pkg[,pkg]>
                        A comma-separated list of package or module names from
                        where C extensions may be loaded. Extensions are
                        loading into the active Python interpreter and may run
                        arbitrary code. (This is an alternative name to
                        extension-pkg-allow-list for backward compatibility.)
                        [current: none]
    --suggestion-mode=<y or n>
                        When enabled, pylint would attempt to guess common
                        misconfiguration and emit user-friendly hints instead
                        of false-positive error messages. [current: yes]
    --exit-zero         Always return a 0 (non-error) status code, even if
                        lint errors are found. This is primarily useful in
                        continuous integration scripts.
    --from-stdin        Interpret the stdin as a python script, whose filename
                        needs to be passed as the module_or_package argument.
    --py-version=<py_version>
                        Minimum Python version to use for version dependent
                        checks. Will default to the version used to run
                        pylint. [current: 3.8]

  Commands:
    --rcfile=<file>     Specify a configuration file to load.
    --output=<file>     Specify an output file.
    --help-msg=<msg-id>
                        Display a help message for the given message id and
                        exit. The value may be a comma separated list of
                        message ids.
    --list-msgs         Display a list of all pylint's messages divided by
                        whether they are emittable with the given interpreter.
    --list-msgs-enabled
                        Display a list of what messages are enabled, disabled
                        and non-emittable with the given configuration.
    --list-groups       List pylint's message groups.
    --list-conf-levels  Generate pylint's confidence levels.
    --list-extensions   List available extensions.
    --full-documentation
                        Generate pylint's full documentation.
    --generate-rcfile   Generate a sample configuration file according to the
                        current configuration. You can put other options
                        before this one to get them in the generated
                        configuration.

  Messages control:
    --confidence=<levels>
                        Only show warnings with the listed confidence levels.
                        Leave empty to show all. Valid levels: HIGH,
                        INFERENCE, INFERENCE_FAILURE, UNDEFINED. [current:
                        none]
    -e <msg ids>, --enable=<msg ids>
                        Enable the message, report, category or checker with
                        the given id(s). You can either give multiple
                        identifier separated by comma (,) or put this option
                        multiple time (only on the command line, not in the
                        configuration file where it should appear only once).
                        See also the "--disable" option for examples.
    -d <msg ids>, --disable=<msg ids>
                        Disable the message, report, category or checker with
                        the given id(s). You can either give multiple
                        identifiers separated by comma (,) or put this option
                        multiple times (only on the command line, not in the
                        configuration file where it should appear only once).
                        You can also use "--disable=all" to disable everything
                        first and then re-enable specific checks. For example,
                        if you want to run only the similarities checker, you
                        can use "--disable=all --enable=similarities". If you
                        want to run only the classes checker, but have no
                        Warning level messages displayed, use "--disable=all
                        --enable=classes --disable=W".

  Reports:
    -f <format>, --output-format=<format>
                        Set the output format. Available formats are text,
                        parseable, colorized, json and msvs (visual studio).
                        You can also give a reporter class, e.g.
                        mypackage.mymodule.MyReporterClass. [current: text]
    -r <y or n>, --reports=<y or n>
                        Tells whether to display a full report or only the
                        messages. [current: no]
    --evaluation=<python_expression>
                        Python expression which should return a score less
                        than or equal to 10. You have access to the variables
                        'error', 'warning', 'refactor', and 'convention' which
                        contain the number of messages in each category, as
                        well as 'statement' which is the total number of
                        statements analyzed. This score is used by the global
                        evaluation report (RP0004). [current: 10.0 - ((float(5
                        * error + warning + refactor + convention) /
                        statement) * 10)]
    -s <y or n>, --score=<y or n>
                        Activate the evaluation score. [current: yes]
    --msg-template=<template>
                        Template used to display messages. This is a python
                        new-style format string used to format the message
                        information. See doc for all details.
```

Let’s move to the next step.

```
```
````md
# Run Pylint

We will run a simple pylint scan on our project:

```bash
pylint taskManager/*.py
````

**Command Output**

```
************* Module taskManager
taskManager/__init__.py:1:0: C0103: Module name "taskManager" doesn't conform to snake_case naming style (invalid-name)
************* Module taskManager.forms
taskManager/forms.py:18:0: E0401: Unable to import 'django' (import-error)
taskManager/forms.py:19:0: E0401: Unable to import 'django.contrib.auth.models' (import-error)
taskManager/forms.py:73:4: C0115: Missing class docstring (missing-class-docstring)
taskManager/forms.py:73:4: R0903: Too few public methods (0/2) (too-few-public-methods)
taskManager/forms.py:71:0: R0903: Too few public methods (0/2) (too-few-public-methods)
taskManager/forms.py:78:0: R0903: Too few public methods (0/2) (too-few-public-methods)
taskManager/forms.py:84:0: R0903: Too few public methods (0/2) (too-few-public-methods)
taskManager/forms.py:18:0: C0411: third party import "from django import forms" should be placed before "from taskManager.models import Project, Task" (wrong-import-order)
taskManager/forms.py:19:0: C0411: third party import "from django.contrib.auth.models import User" should be placed before "from taskManager.models import Project, Task" (wrong-import-order)
************* Module taskManager.models
taskManager/models.py:1:0: C0114: Missing module docstring (missing-module-docstring)
taskManager/models.py:17:0: E0401: Unable to import 'django.contrib.auth.models' (import-error)
taskManager/models.py:19:0: E0401: Unable to import 'django.utils' (import-error)
taskManager/models.py:20:0: E0401: Unable to import 'django.db' (import-error)
taskManager/models.py:23:0: C0115: Missing class docstring (missing-class-docstring)
taskManager/models.py:23:0: R0903: Too few public methods (0/2) (too-few-public-methods)
taskManager/models.py:29:0: C0115: Missing class docstring (missing-class-docstring)
taskManager/models.py:45:4: C0116: Missing function or method docstring (missing-function-docstring)
taskManager/models.py:48:4: C0116: Missing function or method docstring (missing-function-docstring)
taskManager/models.py:51:4: C0116: Missing function or method docstring (missing-function-docstring)
taskManager/models.py:61:0: C0115: Missing class docstring (missing-class-docstring)
taskManager/models.py:78:4: C0116: Missing function or method docstring (missing-function-docstring)
taskManager/models.py:81:4: C0116: Missing function or method docstring (missing-function-docstring)
taskManager/models.py:84:4: C0116: Missing function or method docstring (missing-function-docstring)
taskManager/models.py:88:0: C0115: Missing class docstring (missing-class-docstring)
taskManager/models.py:88:0: R0903: Too few public methods (1/2) (too-few-public-methods)
taskManager/models.py:99:0: C0115: Missing class docstring (missing-class-docstring)
taskManager/models.py:99:0: R0903: Too few public methods (1/2) (too-few-public-methods)
************* Module taskManager.settings
taskManager/settings.py:1:0: C0114: Missing module docstring (missing-module-docstring)
************* Module taskManager.taskManager_urls
taskManager/taskManager_urls.py:1:0: C0103: Module name "taskManager_urls" doesn't conform to snake_case naming style (invalid-name)
taskManager/taskManager_urls.py:1:0: C0114: Missing module docstring (missing-module-docstring)
taskManager/taskManager_urls.py:15:0: E0401: Unable to import 'django.conf.urls' (import-error)
************* Module taskManager.tests
taskManager/tests.py:1:0: C0114: Missing module docstring (missing-module-docstring)
taskManager/tests.py:3:0: E0401: Unable to import 'django.utils' (import-error)
taskManager/tests.py:4:0: E0401: Unable to import 'django.test' (import-error)
taskManager/tests.py:8:0: C0115: Missing class docstring (missing-class-docstring)
taskManager/tests.py:6:0: W0611: Unused User imported from models (unused-import)
************* Module taskManager.urls
taskManager/urls.py:1:0: C0114: Missing module docstring (missing-module-docstring)
taskManager/urls.py:15:0: E0401: Unable to import 'django.conf.urls' (import-error)
taskManager/urls.py:16:0: E0401: Unable to import 'django.contrib' (import-error)
************* Module taskManager.views
taskManager/views.py:796:0: C0301: Line too long (107/100) (line-too-long)
taskManager/views.py:802:0: C0301: Line too long (166/100) (line-too-long)
taskManager/views.py:1:0: C0114: Missing module docstring (missing-module-docstring)
taskManager/views.py:20:0: E0401: Unable to import 'django.shortcuts' (import-error)
taskManager/views.py:21:0: E0401: Unable to import 'django.http' (import-error)
taskManager/views.py:22:0: E0401: Unable to import 'django.utils' (import-error)
taskManager/views.py:23:0: E0401: Unable to import 'django.template' (import-error)
taskManager/views.py:24:0: E0401: Unable to import 'django.db' (import-error)
taskManager/views.py:26:0: E0401: Unable to import 'django.views.decorators.csrf' (import-error)
taskManager/views.py:28:0: E0401: Unable to import 'django.contrib' (import-error)
taskManager/views.py:29:0: E0401: Unable to import 'django.contrib.auth' (import-error)
taskManager/views.py:30:0: E0401: Unable to import 'django.contrib.auth' (import-error)
taskManager/views.py:31:0: E0401: Unable to import 'django.contrib.auth.models' (import-error)
taskManager/views.py:32:0: E0401: Unable to import 'django.contrib.auth' (import-error)
taskManager/views.py:39:0: C0116: Missing function or method docstring (missing-function-docstring)
taskManager/views.py:48:12: R1705: Unnecessary "else" after "return" (no-else-return)
taskManager/views.py:74:0: C0116: Missing function or method docstring (missing-function-docstring)
taskManager/views.py:83:12: R1705: Unnecessary "else" after "return" (no-else-return)
taskManager/views.py:112:0: C0116: Missing function or method docstring (missing-function-docstring)
taskManager/views.py:126:12: R1705: Unnecessary "else" after "return" (no-else-return)
taskManager/views.py:155:12: R1705: Unnecessary "else" after "return" (no-else-return)
taskManager/views.py:170:0: C0116: Missing function or method docstring (missing-function-docstring)
taskManager/views.py:177:8: R1705: Unnecessary "else" after "return" (no-else-return)
taskManager/views.py:174:8: W0612: Unused variable 'proj' (unused-variable)
taskManager/views.py:206:0: C0116: Missing function or method docstring (missing-function-docstring)
taskManager/views.py:206:13: W0613: Unused argument 'request' (unused-argument)
taskManager/views.py:220:0: C0116: Missing function or method docstring (missing-function-docstring)
taskManager/views.py:224:4: R1705: Unnecessary "else" after "return" (no-else-return)
taskManager/views.py:220:25: W0613: Unused argument 'request' (unused-argument)
taskManager/views.py:240:0: C0116: Missing function or method docstring (missing-function-docstring)
taskManager/views.py:242:4: R1705: Unnecessary "else" after "return" (no-else-return)
taskManager/views.py:273:0: C0116: Missing function or method docstring (missing-function-docstring)
taskManager/views.py:278:4: R1705: Unnecessary "else" after "return" (no-else-return)
taskManager/views.py:288:29: R1719: The if expression can be replaced with 'test' (simplifiable-if-expression)
taskManager/views.py:299:0: C0116: Missing function or method docstring (missing-function-docstring)
taskManager/views.py:299:16: W0613: Unused argument 'request' (unused-argument)
taskManager/views.py:311:0: C0116: Missing function or method docstring (missing-function-docstring)
taskManager/views.py:311:18: W0613: Unused argument 'request' (unused-argument)
taskManager/views.py:322:0: C0116: Missing function or method docstring (missing-function-docstring)
taskManager/views.py:324:4: R1705: Unnecessary "else" after "return" (no-else-return)
taskManager/views.py:350:0: C0116: Missing function or method docstring (missing-function-docstring)
taskManager/views.py:354:4: R1705: Unnecessary "else" after "return" (no-else-return)
taskManager/views.py:376:0: C0116: Missing function or method docstring (missing-function-docstring)
taskManager/views.py:376:19: W0613: Unused argument 'request' (unused-argument)
taskManager/views.py:385:0: C0116: Missing function or method docstring (missing-function-docstring)
taskManager/views.py:390:0: C0116: Missing function or method docstring (missing-function-docstring)
taskManager/views.py:398:16: R1705: Unnecessary "else" after "return" (no-else-return)
taskManager/views.py:421:0: C0116: Missing function or method docstring (missing-function-docstring)
taskManager/views.py:466:0: C0116: Missing function or method docstring (missing-function-docstring)
taskManager/views.py:479:4: R1705: Unnecessary "else" after "return" (no-else-return)
taskManager/views.py:491:0: C0116: Missing function or method docstring (missing-function-docstring)
taskManager/views.py:511:0: C0116: Missing function or method docstring (missing-function-docstring)
taskManager/views.py:515:4: R1705: Unnecessary "else" after "return" (no-else-return)
taskManager/views.py:530:0: C0116: Missing function or method docstring (missing-function-docstring)
taskManager/views.py:531:4: R1705: Unnecessary "else" after "return" (no-else-return)
taskManager/views.py:554:0: C0116: Missing function or method docstring (missing-function-docstring)
taskManager/views.py:560:4: R1705: Unnecessary "else" after "return" (no-else-return)
taskManager/views.py:581:0: C0116: Missing function or method docstring (missing-function-docstring)
taskManager/views.py:581:16: W0613: Unused argument 'request' (unused-argument)
taskManager/views.py:593:0: C0116: Missing function or method docstring (missing-function-docstring)
taskManager/views.py:593:26: W0613: Unused argument 'project_id' (unused-argument)
taskManager/views.py:628:0: C0116: Missing function or method docstring (missing-function-docstring)
taskManager/views.py:640:0: C0116: Missing function or method docstring (missing-function-docstring)
taskManager/views.py:655:0: C0116: Missing function or method docstring (missing-function-docstring)
taskManager/views.py:661:0: C0116: Missing function or method docstring (missing-function-docstring)
taskManager/views.py:678:0: C0116: Missing function or method docstring (missing-function-docstring)
taskManager/views.py:684:0: C0116: Missing function or method docstring (missing-function-docstring)
taskManager/views.py:685:4: R1705: Unnecessary "else" after "return" (no-else-return)
taskManager/views.py:703:0: C0116: Missing function or method docstring (missing-function-docstring)
taskManager/views.py:711:0: C0116: Missing function or method docstring (missing-function-docstring)
taskManager/views.py:740:0: C0116: Missing function or method docstring (missing-function-docstring)
taskManager/views.py:780:0: C0116: Missing function or method docstring (missing-function-docstring)
taskManager/views.py:790:0: R1721: Unnecessary use of a comprehension (unnecessary-comprehension)
taskManager/views.py:791:16: C0103: Variable name "x" doesn't conform to snake_case naming style (invalid-name)
taskManager/views.py:814:0: C0116: Missing function or method docstring (missing-function-docstring)
taskManager/views.py:837:0: C0116: Missing function or method docstring (missing-function-docstring)
taskManager/views.py:18:0: W0611: Unused import codecs (unused-import)
************* Module taskManager.wsgi
taskManager/wsgi.py:13:0: E0401: Unable to import 'django.core.wsgi' (import-error)
taskManager/wsgi.py:13:0: C0413: Import "from django.core.wsgi import get_wsgi_application" should be placed at the top of the module (wrong-import-position)

------------------------------------------------------------------
Your code has been rated at 6.37/10 (previous run: 6.37/10, +0.00)
```

Seems there’s a lot which can be improved in this code.

If you want to save the output as a JSON file, you can use `-f json`.

```bash
pylint taskManager/*.py -f json | tee output.json
```

The output of the pylint tool will be saved in JSON formatted file `output.json`.

If you open the file, you can see that the pylint has found several errors such as Missing function/method, unused argument, and so on.

However, there seems to be quite a bit of noise (false positives) in the results.

Let’s move to the next step.

```
```
````md
# Reduce False Positives

As we have seen in **bandit** exercise, we can reduce false positives using `.pylintrc` file and direct pylint to skip specific checks during the scan.

Let’s create `.pylintrc` to skip or disable some errors:

```bash
cat > .pylintrc <<EOF
[MASTER]
disable=missing-module-docstring,import-error
EOF
````

Let’s run the scan once again to see what’s changed.

```bash
pylint taskManager/*.py
```

**Command Output**

```
...[SNIP]...
taskManager/views.py:740:0: C0116: Missing function or method docstring (missing-function-docstring)
taskManager/views.py:780:0: C0116: Missing function or method docstring (missing-function-docstring)
taskManager/views.py:790:0: R1721: Unnecessary use of a comprehension (unnecessary-comprehension)
taskManager/views.py:791:16: C0103: Variable name "x" doesn't conform to snake_case naming style (invalid-name)
taskManager/views.py:814:0: C0116: Missing function or method docstring (missing-function-docstring)
taskManager/views.py:837:0: C0116: Missing function or method docstring (missing-function-docstring)
taskManager/views.py:18:0: W0611: Unused import codecs (unused-import)
************* Module taskManager.wsgi
taskManager/wsgi.py:13:0: C0413: Import "from django.core.wsgi import get_wsgi_application" should be placed at the top of the module (wrong-import-position)

------------------------------------------------------------------
Your code has been rated at 8.41/10 (previous run: 6.37/10, +2.03)
```

The output number may vary as it is dynamic.

Right now, our code is rated at **8.41/10**, earlier it was **6.37/10**.

✅ That’s a good jump from the quality perspective with `.pylintrc`.

```
```
````md
# Code Quality Analysis With SonarQube

## Downloading the Source Code

It’s important that we set up and try out the commands locally first in **DevSecOps-Box** to avoid potential issues or errors before deploying them to production environments.

First, we need to download the source code of the project from our git repository.

```bash
git clone https://gitlab.practical-devsecops.training/pdso/django.nv webapp
````

**Command Output**

```
Cloning into 'webapp'...
warning: redirecting to https://gitlab.practical-devsecops.training/pdso/django.nv.git/
remote: Enumerating objects: 228, done.
remote: Total 228 (delta 0), reused 0 (delta 0), pack-reused 228
Receiving objects: 100% (228/228), 1.03 MiB | 1.04 MiB/s, done.
Resolving deltas: 100% (86/86), done.
```

Let’s get into the **webapp** directory, so we can scan the application.

```bash
cd webapp
```

**Command Output**

```
/webapp#
```

We are now in the **webapp** directory.

---

✅ Let’s move to the next step.

```
```
````md
# Installing Sonar Scanner CLI

Sonar Scanner is the scanner command line for **SonarQube** and **SonarCloud**.  
It helps to run code analysis on the source code and integrates the scan results with the **SonarQube Server** to present them in a dashboard.

👉 More details: [Sonar Scanner CLI GitHub](https://github.com/SonarSource/sonar-scanner-cli)

---

## Step 1: Set Version

```bash
export SONAR_VERSION="6.0.0.4432"
````

---

## Step 2: Download the Sonar Scanner CLI package

```bash
wget https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-${SONAR_VERSION}-linux.zip -O /opt/sonar-scanner.zip
```

**Command Output**

```
--2024-07-12 05:52:52--  https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-4.7.0.2747-linux.zip
Resolving binaries.sonarsource.com (binaries.sonarsource.com)... 99.84.66.124, 99.84.66.95, 99.84.66.2, ...
Connecting to binaries.sonarsource.com (binaries.sonarsource.com)|99.84.66.124|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 43162003 (41M) [application/zip]
Saving to: ‘/opt/sonar-scanner.zip’

/opt/sonar-scanner.zip                 100%[============================================================================>]  41.16M  --.-KB/s    in 0.1s    

2024-07-12 05:52:52 (289 MB/s) - ‘/opt/sonar-scanner.zip’ saved [43162003/43162003]
```

---

## Step 3: Extract and Add to PATH

```bash
unzip /opt/sonar-scanner.zip -d /opt/
chmod +x /opt/sonar-scanner-${SONAR_VERSION}-linux/bin/sonar-scanner
export PATH=/opt/sonar-scanner-${SONAR_VERSION}-linux/bin/:$PATH
```

**Command Output (partial)**

```
Archive:  /opt/sonar-scanner.zip
   creating: /opt/sonar-scanner-4.7.0.2747-linux/
   creating: /opt/sonar-scanner-4.7.0.2747-linux/jre/
   creating: /opt/sonar-scanner-4.7.0.2747-linux/jre/legal/
   creating: /opt/sonar-scanner-4.7.0.2747-linux/jre/legal/java.sql/
   creating: /opt/sonar-scanner-4.7.0.2747-linux/jre/legal/java.compiler/
   creating: /opt/sonar-scanner-4.7.0.2747-linux/jre/legal/java.datatransfer/
   creating: /opt/sonar-scanner-4.7.0.2747-linux/jre/legal/jdk.management.jfr/
   creating: /opt/sonar-scanner-4.7.0.2747-linux/jre/legal/jdk.xml.dom/

...[SNIP]...

finishing deferred symbolic links:
  /opt/sonar-scanner-4.7.0.2747-linux/jre/legal/java.sql/LICENSE -> ../java.base/LICENSE
  /opt/sonar-scanner-4.7.0.2747-linux/jre/legal/java.sql/ADDITIONAL_LICENSE_INFO -> ../java.base/ADDITIONAL_LICENSE_INFO
  /opt/sonar-scanner-4.7.0.2747-linux/jre/legal/java.sql/ASSEMBLY_EXCEPTION -> ../java.base/ASSEMBLY_EXCEPTION
  /opt/sonar-scanner-4.7.0.2747-linux/jre/legal/java.compiler/LICENSE -> ../java.base/LICENSE
  /opt/sonar-scanner-4.7.0.2747-linux/jre/legal/java.compiler/ADDITIONAL_LICENSE_INFO -> ../java.base/ADDITIONAL_LICENSE_INFO
  /opt/sonar-scanner-4.7.0.2747-linux/jre/legal/java.compiler/ASSEMBLY_EXCEPTION -> ../java.base/ASSEMBLY_EXCEPTION

...[SNIP]...

 /opt/sonar-scanner-4.7.0.2747-linux/jre/legal/jdk.internal.vm.compiler.management/LICENSE -> ../java.base/LICENSE
  /opt/sonar-scanner-4.7.0.2747-linux/jre/legal/jdk.internal.vm.compiler.management/ADDITIONAL_LICENSE_INFO -> ../java.base/ADDITIONAL_LICENSE_INFO
  /opt/sonar-scanner-4.7.0.2747-linux/jre/legal/jdk.internal.vm.compiler.management/ASSEMBLY_EXCEPTION -> ../java.base/ASSEMBLY_EXCEPTION
```

---

✅ There you go! We have successfully installed **sonar-scanner**.

👉 Let’s move on to the next step.

```
```
````md
# Running the Scanner

---

## 1. Login into the machine

Before we can use SonarQube, we are required to log in to the machine using the following details:

| Name          | Value                                                                 |
|---------------|----------------------------------------------------------------------|
| Sonarqube URL | https://sonarqube-kr6k1mdm.lab.practical-devsecops.training/sessions/new |
| Login         | admin                                                                 |
| Password      | pdso-training                                                         |

---

## 2. Create a new project

Once logged in, we will be given options about our project location.

- Select **Create a local project** option.
- Next, fill out the project display name form with the following:

| Name                | Value   |
|---------------------|---------|
| Project display name | Django  |
| Project key          | Django  |
| Branch               | main    |

- Select **Use the global settings** option.
- Click **Create Project**.

Next, we will be redirected to the **Analysis Method** page.  
- Select **Locally**.  
- Click **Create Project**.

> **Note:** The project key is a unique identifier for our project and must be different for each project.

---

## 3. Create a new token

To execute the scan, we must create and use a **SonarQube token** for user identification.

```bash
SONARQUBE_TOKEN=$(curl -u admin:pdso-training -X POST 'https://sonarqube-kr6k1mdm.lab.practical-devsecops.training/api/user_tokens/generate' -d 'name=Django' | jq -r '.token')
````

Verify the token:

```bash
echo $SONARQUBE_TOKEN
```

**Command Output**

```
sqp_385615fb184ae524aa13f1a3df0f8845751d33e5
```

> Each project receives a unique token. Tokens can be revoked anytime for security reasons.

---

## 4. Running the SonarQube Scanner

```bash
sonar-scanner \
-Dsonar.projectKey=Django \
-Dsonar.sources=. \
-Dsonar.host.url=https://sonarqube-kr6k1mdm.lab.practical-devsecops.training \
-Dsonar.login=$SONARQUBE_TOKEN
```

**Command Output (partial)**

```
22:33:57.130 INFO  Scanner configuration file: /opt/sonar-scanner-6.0.0.4432-linux/conf/sonar-scanner.properties
22:33:57.153 INFO  SonarScanner CLI 6.0.0.4432
22:33:57.156 INFO  Java 17.0.11 Eclipse Adoptium (64-bit)
22:34:35.850 INFO  Analysis total time: 32.807 s
22:34:36.285 INFO  EXECUTION SUCCESS
22:34:36.287 INFO  Total time: 39.159s
```

---

## Installing NodeJS (if missing)

```bash
mkdir -p /etc/apt/keyrings
curl -fsSL https://deb.nodesource.com/gpgkey/nodesource-repo.gpg.key | gpg --dearmor -o /etc/apt/keyrings/nodesource.gpg
NODE_MAJOR=20
echo "deb [signed-by=/etc/apt/keyrings/nodesource.gpg] https://deb.nodesource.com/node_$NODE_MAJOR.x nodistro main" | tee /etc/apt/sources.list.d/nodesource.list
apt update
apt install nodejs -y
```

---

## Re-run the scanner

```bash
sonar-scanner \
-Dsonar.projectKey=Django \
-Dsonar.sources=. \
-Dsonar.host.url=https://sonarqube-kr6k1mdm.lab.practical-devsecops.training \
-Dsonar.login=$SONARQUBE_TOKEN
```

**Command Output (partial)**

```
22:35:42.539 INFO  Analysis report generated in 173ms, dir size=2.3 MB
22:35:43.122 INFO  ANALYSIS SUCCESSFUL, you can find the results at: https://sonarqube-kr6k1mdm.lab.practical-devsecops.training/dashboard?id=Django
22:35:44.437 INFO  SonarScanner Engine completed successfully
22:35:44.839 INFO  EXECUTION SUCCESS
22:35:44.841 INFO  Total time: 31.870s
```

---

## SonarQube Dashboard

After the scanning process is complete, we can check **SonarQube dashboard** for the results.

### Issues Summary

| Issue Type          | Count | Description                                                     |
| ------------------- | ----- | --------------------------------------------------------------- |
| **Security**        | 80    | Vulnerabilities and security weaknesses that could be exploited |
| **Reliability**     | 2     | Bugs and code issues that may cause unexpected behavior         |
| **Maintainability** | 429   | Code smells and technical debt affecting long-term quality      |
| **Duplication**     | 1.0%  | Copy-pasted code that may indicate design flaws                 |

---

### Ratings

| Rating | Reliability rating        | Security rating                     | Maintainability rating |
| ------ | ------------------------- | ----------------------------------- | ---------------------- |
| **A**  | No bugs                   | No vulnerabilities                  | Less than 5%           |
| **B**  | At least one minor bug    | At least one minor vulnerability    | Less than 10%          |
| **C**  | At least one major bug    | At least one major vulnerability    | Less than 20%          |
| **D**  | At least one critical bug | At least one critical vulnerability | Less than 50%          |
| **E**  | At least one blocker bug  | At least one blocker vulnerability  | Higher than 50%        |

---

✅ By running a static scan with **SonarQube**, we’ve uncovered **security issues, reliability bugs, maintainability problems, and code duplication risks**.
This is a crucial step toward improving **code quality and security posture**.

```
```
````md
# Download the Source Code

We will do all the exercises locally first in **DevSecOps-Box**, so let’s start the exercise.

First, we need to download the source code of the project from our git repository.

```bash
git clone https://gitlab.practical-devsecops.training/pdso/php.git
````

---

Now, let’s `cd` into the application so we can scan the app.

```bash
cd php
```

---

✅ We are now in the **php** directory.

Let’s move to the next step.

```
```
````md
# Installing PHPStan

**PHPStan** is a SAST tool that can find insecure code patterns in your PHP applications by identifying potential bugs, including security issues.  
It uses static analysis to check the code for errors, undefined variables, and other issues that could lead to vulnerabilities.

🔗 More details: [https://github.com/phpstan/phpstan](https://github.com/phpstan/phpstan)

---

## Step 1: Check PHP Version

```bash
php -v
````

**Command Output**

```
bash: php: command not found
```

The error above shows that **PHP** is not installed.

---

## Step 2: Install Composer (also installs PHP if missing)

```bash
apt update && apt install composer -y
```

Re-check PHP installation:

```bash
php -v
```

**Command Output**

```
PHP 7.4.3-4ubuntu2.23 (cli) (built: Jun 17 2024 13:22:20) ( NTS )
Copyright (c) The PHP Group
Zend Engine v3.4.0, Copyright (c) Zend Technologies
    with Zend OPcache v7.4.3-4ubuntu2.23, Copyright (c), by Zend Technologies
```

---

## Step 3: Install PHP 7.4

Add the correct repository:

```bash
apt install software-properties-common -y
add-apt-repository ppa:ondrej/php -y
apt update
```

Install PHP 7.4 and XML extension:

```bash
apt install php7.4 php7.4-xml -y
update-alternatives --set php /usr/bin/php7.4
```

Verify:

```bash
php -v
```

**Command Output**

```
PHP 7.4.3-4ubuntu2.23 (cli) (built: Jun 17 2024 13:22:20) ( NTS )
Copyright (c) The PHP Group
Zend Engine v3.4.0, Copyright (c) Zend Technologies
    with Zend OPcache v7.4.3-4ubuntu2.23, Copyright (c), by Zend Technologies
```

---

## Step 4: Install Composer (latest version)

```bash
apt install php-xml -y
curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer --version=2.2.22
```

Update PATH:

```bash
export PATH="/usr/local/bin:$PATH"
hash -r
composer --version
```

**Command Output**

```
Composer version 2.8.4 2024-12-11 11:57:47
PHP version 8.1.2-1ubuntu2.20 (/usr/bin/php8.1)
Run the "diagnose" command to get more detailed diagnostics output.
```

✅ Composer 2.x is required (Composer 1 will be discontinued on **August 1st, 2025**).

---

## Step 5: Install PHPStan

```bash
composer require --dev phpstan/phpstan:1.10.7
```

**Command Output**

```
70 package suggestions were added by new dependencies, use `composer suggest` to see details.
Package swiftmailer/swiftmailer is abandoned, you should avoid using it. Use symfony/mailer instead.
Package symfony/debug is abandoned, you should avoid using it. Use symfony/error-handler instead.
Package fzaninotto/faker is abandoned, you should avoid using it. No replacement was suggested.
Package phpunit/php-token-stream is abandoned, you should avoid using it. No replacement was suggested.
Generating optimized autoload files
...
Package manifest generated successfully.
...
Using version ^1.12 for phpstan/phpstan
```

Check if installed:

```bash
ls vendor/phpstan/phpstan
```

**Command Output**

```
LICENSE  README.md  bootstrap.php  composer.json  conf  phpstan  phpstan.phar  phpstan.phar.asc
```

Move binary:

```bash
mv vendor/phpstan/phpstan/phpstan.phar /usr/local/bin/phpstan
```

---

## Step 6: Verify Installation

```bash
phpstan --help
```

**Command Output**

```
Description:
  Analyses source code

Usage:
  analyse [options] [--] [<paths>...]
  analyze

Arguments:
  paths                                        Paths with source code to run analysis on

Options:
  -c, --configuration=CONFIGURATION            Path to project configuration file
  -l, --level=LEVEL                            Level of rule options - the higher the stricter
      --no-progress                            Do not show progress bar, only results
      --debug                                  Show debug information - which file is analysed, do not catch internal errors
  -a, --autoload-file=AUTOLOAD-FILE            Project's additional autoload file path
      --error-format=ERROR-FORMAT              Format in which to print the result of the analysis
  -b, --generate-baseline[=GENERATE-BASELINE]  Path to a file where the baseline should be saved [default: false]
      --allow-empty-baseline                   Do not error out when the generated baseline is empty
      --memory-limit=MEMORY-LIMIT              Memory limit for analysis
      --xdebug                                 Allow running with XDebug for debugging purposes
      --fix                                    Launch PHPStan Pro
      --watch                                  Launch PHPStan Pro
      --pro                                    Launch PHPStan Pro
  -h, --help                                   Display help for the given command...
```

---

✅ We have successfully installed **PHPStan**.
Let’s move to the next step.

```
```
````md
# Run the Scanner with PHPStan

First, we can test how **PHPStan** stores the output:

```bash
phpstan analyse .
````

---

### Command Output

```
...[SNIP]...

 ------ --------------------------------------------------------------------- 
  Line   vendor/symfony/var-dumper/Caster/ResourceCaster.php                  
 ------ --------------------------------------------------------------------- 
  73     Function mysql_get_host_info not found.                              
         💡 Learn more at https://phpstan.org/user-guide/discovering-symbols   
  74     Function mysql_get_proto_info not found.                             
         💡 Learn more at https://phpstan.org/user-guide/discovering-symbols   
  75     Function mysql_get_server_info not found.                            
         💡 Learn more at https://phpstan.org/user-guide/discovering-symbols   
 ------ --------------------------------------------------------------------- 

 ------ ---------------------------------------------------------------------------- 
  Line   vendor/symfony/var-dumper/Dumper/ContextProvider/SourceContextProvider.php  
 ------ ---------------------------------------------------------------------------- 
  64     Class Twig\Template not found.                                              
         💡 Learn more at https://phpstan.org/user-guide/discovering-symbols          
 ------ ---------------------------------------------------------------------------- 

 ------ ------------------------------------------------------------------------------------------------ 
  Line   vendor/tijsverkoyen/css-to-inline-styles/src/CssToInlineStyles.php                              
 ------ ------------------------------------------------------------------------------------------------ 
  173    Call to static method toXPath() on an unknown class Symfony\Component\CssSelector\CssSelector.  
         💡 Learn more at https://phpstan.org/user-guide/discovering-symbols                              
 ------ ------------------------------------------------------------------------------------------------ 

 ------ ------------------------------------------------------------------------------------------------------ 
  Line   vendor/webmozart/assert/src/Assert.php                                                                
 ------ ------------------------------------------------------------------------------------------------------ 
  1871   Template type T of method Webmozart\Assert\Assert::isMap() is not referenced in a parameter.          
  1895   Template type T of method Webmozart\Assert\Assert::isNonEmptyMap() is not referenced in a parameter.  
 ------ ------------------------------------------------------------------------------------------------------ 

 ------ --------------------------------------------------------------------------------------------------------- 
  Line   vendor/webmozart/assert/src/Mixin.php (in context of class Webmozart\Assert\Assert)                      
 ------ --------------------------------------------------------------------------------------------------------- 
  4871   Template type T of method Webmozart\Assert\Assert::nullOrIsMap() is not referenced in a parameter.       
  4889   Template type T of method Webmozart\Assert\Assert::allIsMap() is not referenced in a parameter.          
  4911   Template type T of method Webmozart\Assert\Assert::allNullOrIsMap() is not referenced in a parameter.    
  4932   Template type T of method Webmozart\Assert\Assert::nullOrIsNonEmptyMap() is not referenced in a          
         parameter.                                                                                               
  4949   Template type T of method Webmozart\Assert\Assert::allIsNonEmptyMap() is not referenced in a parameter.  
  4972   Template type T of method Webmozart\Assert\Assert::allNullOrIsNonEmptyMap() is not referenced in a       
         parameter.                                                                                               
 ------ --------------------------------------------------------------------------------------------------------- 


 [ERROR] Found 1851 errors
```

⚠️ You need to wait until the tool finishes scanning.
PHPStan ran successfully and it **found many issues/bugs**.

---

## Store Results in JSON

As we have learned in the **DevSecOps Gospel**, we should store results in a machine-readable format (JSON).

```bash
phpstan analyse --error-format=json . | tee phpstan-output.json
```

Check the JSON file:

```bash
cat phpstan-output.json | jq .
```

---

### Command Output (snippet)

```json
    "/php/vendor/webmozart/assert/src/Mixin.php (in context of class Webmozart\\Assert\\Assert)": {
      "errors": 6,
      "messages": [
        {
          "message": "Template type T of method Webmozart\\Assert\\Assert::nullOrIsMap() is not referenced in a parameter.",
          "line": 4871,
          "ignorable": true
        },
        {
          "message": "Template type T of method Webmozart\\Assert\\Assert::allIsMap() is not referenced in a parameter.",
          "line": 4889,
          "ignorable": true
        }
        ...
      ]
    }
  },
  "errors": []
```

✅ Now the results are stored in **`phpstan-output.json`**.

---

## PHPStan Levels (0 → 8+)

PHPStan uses **levels** to define how strict the analysis should be.

* **Level 0** → Loosest (only catches the most obvious issues).
* **Level 1** → Adds checks for undefined variables, unknown classes.
* **Level 2–3** → Detects missing typehints in functions/methods, improper type usage.
* **Level 4–5** → Enforces strict typing across parameters and return types.
* **Level 6** → Flags possible type errors across multiple function calls.
* **Level 7** → Requires full type coverage (arguments, return values, properties).
* **Level 8 (default in newer versions)** → The strictest, aims for 100% type safety.

---

### Example

Run with **level 0** (basic checks only):

```bash
phpstan analyse -l 0 .
```

Run with **level 7** (very strict):

```bash
phpstan analyse -l 7 .
```

---

### 📌 Summary

* **Low level (0–1)** → Good for beginners or legacy projects (few false positives).
* **Medium level (2–5)** → Recommended for most projects, balances findings and noise.
* **High level (6–8)** → Enforces maximum type safety, ideal for well-typed modern projects.

---

👉 Reference: [PHPStan Documentation – Getting Started](https://phpstan.org/user-guide/getting-started)

```
```
````md
# How to Embed PHPStan into GitLab

## A Simple CI/CD Pipeline

Let’s download the code using git clone in DevSecOps Box.

```bash
git clone https://gitlab.practical-devsecops.training/pdso/php.git
````

```bash
cd php
```

Rename git url to the new one.

```bash
git remote rename origin old-origin
```

```bash
git remote add origin git@gitlab-ce-kr6k1mdm:root/php.git
```

Then, push the code into the repository and it will automatically create a new project if not exist.

```bash
git push -u origin --all
```

And enter the GitLab credentials.

| Name     | Value         |
| -------- | ------------- |
| Username | root          |
| Password | pdso-training |

Considering your DevOps team created a simple CI pipeline with the following contents.

Hence, you should create .gitlab-ci.yml file in php repository by visiting [https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/php](https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/php)

```
create-new-file
```

```
new-file-format
```

**Click anywhere to copy**

```yaml
image: docker:20.10  # To run all jobs in this pipeline, use the latest docker image

services:
  - docker:dind       # To run all jobs in this pipeline, use a docker image that contains a docker daemon running inside (dind - docker in docker). Reference: https://forum.gitlab.com/t/why-services-docker-dind-is-needed-while-already-having-image-docker/43534

stages:
  - integration
  - prod

integration:
  stage: integration
  script:
    - echo "This is an integration step"
    - exit 1
  allow_failure: true # Even if the job fails, continue to the next stages

prod:
  stage: prod
  script:
    - echo "This is a deploy step."
  when: manual # Continuous Delivery
```

We have 2 jobs in this pipeline, a integration job and a prod job. We did integrate SCA/OAST beforehand, we can carry forward the same tactics in this exercise as well.

As a security engineer, I don’t need to care much about what the DevOps team is doing as part of these jobs. Why? imagine having to learn every build/testing tool used by your DevOps team, it will be a nightmare. Instead, rely on the DevOps team for help.

Let’s login into the GitLab using the following details and execute this pipeline.

### GitLab CI/CD Machine

| Name     | Value                                                                                                                                                                                              |
| -------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Link     | [https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/php/-/blob/main/.gitlab-ci.yml](https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/php/-/blob/main/.gitlab-ci.yml) |
| Username | root                                                                                                                                                                                               |
| Password | pdso-training                                                                                                                                                                                      |

Next, we need to create a CI/CD pipeline by replacing the .gitlab-ci.yml file content with the above CI script. Click on the Edit button to replace the content (use Control+A and Control+V).

Save changes to the file using the Commit changes button.

## Verify the pipeline run

As soon as a change is made to the repository, the pipeline starts executing the jobs.

We can see the results of this pipeline by visiting [https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/php/pipelines](https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/php/pipelines).

Click on the appropriate job name to see the output.

In the next step, we will embed PHPstan in the CI/CD pipeline and follow all the best practices.

Let’s move to the next step.

```
```
````md
# Embed PHPStan in CI/CD pipeline

As discussed in the **Static Analysis using PHPStan** exercise, we can embed PHPStan in our CI/CD pipeline. However, remember that it’s important to **locally test a tool before integrating it into the pipeline**. Troubleshooting a tool manually in a local environment is much easier compared to troubleshooting it in a CI/CD system. Additionally, manually exploring the tool in a local environment helps you become familiar with the tool’s options and features.

---

**Click anywhere to copy**
```yaml
image: docker:20.10  # To run all jobs in this pipeline, use the latest docker image

services:
  - docker:dind       # To run all jobs in this pipeline, use a docker image that contains a docker daemon running inside (dind - docker in docker). Reference: https://forum.gitlab.com/t/why-services-docker-dind-is-needed-while-already-having-image-docker/43534

stages:
  - build
  - integration
  - prod

sast:
  stage: build  # we moved this job from test stage to build stage, by replacing the text test to build
  script:
    # Download PHPStan docker container
    - docker pull phpstan/phpstan
    # Run docker container, please refer docker security course, if this doesn't make sense to you.
    - docker run -v $(pwd):/src --rm phpstan/phpstan analyse --error-format=json /src | tee phpstan-output.json
  artifacts:
    paths: [phpstan-output.json]
    when: always

integration:
  stage: integration
  script:
    - echo "This is an integration step"
    - exit 1
  allow_failure: true # Even if the job fails, continue to the next stages

prod:
  stage: prod
  script:
    - echo "This is a deploy step."
  when: manual # Continuous Delivery
````

---

As discussed, **any change to the repo kick starts the pipeline**.

### Note

We’ve discussed different methods of using the tool:

* **Native Installation**: Directly installing the tool on the system.
* **Package Manager or Binary**: Installing via a package manager or using the binary file.
* **Docker**: Running the tool within a Docker container.

**In summary**:

* All methods are suitable for CI/CD integration.
* **Docker** is recommended for CI/CD as it operates smoothly without dependencies.
* Using the **binary file** is efficient, avoiding additional dependencies.

Ultimately, you can choose either method based on your specific situation.

---

We can see the results of this pipeline by visiting:
👉 [https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/php/pipelines](https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/php/pipelines)

Click on the appropriate job name to see the output.

You will notice that the **sast job’s output** is saved in **phpstan-output.json** file.

---

Let’s move to the next step.

```
```
````md
# Allow the job failure

⚠️ **Remember!**
- Except for **DevSecOps-Box**, every other machine closes after two hours, even if you are in the middle of the exercise.  
- After two hours, in case of a **404**, you need to refresh the exercise page and click on **Start the Exercise** button to continue working.  

We do not want to fail the builds/jobs/scan in **DevSecOps Maturity Levels 1 and 2**, as security tools spit a significant amount of false positives.  

You can use the **allow_failure** tag to not fail the build even though the tool found issues.

---

### Example:

```yaml
sast:
  stage: build
  script:
    # Download PHPStan docker container
    - docker pull phpstan/phpstan
    # Run docker container, please refer docker security course, if this doesn't make sense to you.
    - docker run -v $(pwd):/src --rm phpstan/phpstan analyse --error-format=json /src | tee phpstan-output.json
  artifacts:
    paths: [phpstan-output.json]
    when: always
  allow_failure: true   #<--- allow the build to fail but don't mark it as such
````

---

### After adding the `allow_failure` tag, the pipeline would look like this:

```yaml
image: docker:20.10  # To run all jobs in this pipeline, use the latest docker image

services:
  - docker:dind       # To run all jobs in this pipeline, use a docker image that contains a docker daemon running inside (dind - docker in docker). Reference: https://forum.gitlab.com/t/why-services-docker-dind-is-needed-while-already-having-image-docker/43534

stages:
  - build
  - integration
  - prod

sast:
  stage: build
  script:
    # Download PHPStan docker container
    - docker pull phpstan/phpstan
    # Run docker container, please refer docker security course, if this doesn't make sense to you.
    - docker run -v $(pwd):/src --rm phpstan/phpstan analyse --error-format=json /src | tee phpstan-output.json
  artifacts:
    paths: [phpstan-output.json]
    when: always
  allow_failure: true   #<--- allow the build to fail but don't mark it as such

integration:
  stage: integration
  script:
    - echo "This is an integration step"
    - exit 1
  allow_failure: true # Even if the job fails, continue to the next stages

prod:
  stage: prod
  script:
    - echo "This is a deploy step."
  when: manual # Continuous Delivery
```

---

✅ You will notice that the **sast job failed** however it didn’t block others from continuing further.

As discussed, **any change to the repo kick starts the pipeline**.

We can see the results of this pipeline by visiting:
👉 [https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/php/pipelines](https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/php/pipelines)

Click on the appropriate job name to see the output.

---

### Notes

* The above option mounts the current directory in the host (runner) to `/src` inside the container.

  * On Linux: `-v /home/ubuntu/code:/src`
  * On Windows: `-v c:\users\code:/src`

* Instead of manually removing the container after the scan, we can use the `--rm` option so docker can clean up after itself.

```
```

````md
# Software Component Analysis Using Safety

## Download the source code
We will do all the exercises locally first in **DevSecOps-Box**, so let’s start the exercise.  

First, we need to download the source code of the project from our git repository.

```bash
git clone https://gitlab.practical-devsecops.training/pdso/django.nv webapp
````

### Command Output

```
Cloning into 'webapp'...
warning: redirecting to https://gitlab.practical-devsecops.training/pdso/django.nv.git/
remote: Enumerating objects: 299, done.
remote: Counting objects: 100% (71/71), done.
remote: Compressing objects: 100% (68/68), done.
remote: Total 299 (delta 34), reused 0 (delta 0), pack-reused 228
Receiving objects: 100% (299/299), 1.05 MiB | 29.16 MiB/s, done.
Resolving deltas: 100% (120/120), done.
```

Let’s **cd into the application code**, so we can scan the app.

```bash
cd webapp
```

We are now in the `webapp` directory. ✅

---

➡️ Let’s move to the next step.

```
```
````md
# Install Safety

Safety checks your installed dependencies for known security vulnerabilities.  
By default, it uses the open Python vulnerability database **Safety DB**, but it can be upgraded to use **pyup.io’s Safety API** using the `--key` option.

🔗 Source: [https://pypi.org/project/safety](https://pypi.org/project/safety)  

You can find more details about the project at **Safety**.

---

## Install the safety tool
We will install the **safety** tool on the system to scan Python dependencies.

```bash
pip3 install safety==2.3.5
````

### Command Output

```
Collecting safety==2.3.5
  Downloading safety-2.3.5-py3-none-any.whl (57 kB)
     |████████████████████████████████| 57 kB 8.0 MB/s 
Requirement already satisfied: setuptools>=19.3 in /usr/local/lib/python3.8/dist-packages (from safety==2.3.5) (67.4.0)
Collecting Click>=8.0.2
  Downloading click-8.1.3-py3-none-any.whl (96 kB)
     |████████████████████████████████| 96 kB 9.9 MB/s 
Requirement already satisfied: requests in /usr/local/lib/python3.8/dist-packages (from safety==2.3.5) (2.28.2)

...[SNIP]...

Successfully installed Click-8.1.3 dparse-0.6.2 packaging-21.3 pyparsing-3.0.9 ruamel.yaml-0.17.21 ruamel.yaml.clib-0.2.7 safety-2.3.5 toml-0.10.2
```

---

## Explore Safety options

Now let’s check what options Safety provides us:

```bash
safety check --help
```

### Command Output

```
Usage: safety check [OPTIONS]

  Find vulnerabilities in Python dependencies at the target provided.

Options:
  --key TEXT                      API Key for pyup.io's vulnerability
                                  database. Can be set as SAFETY_API_KEY
                                  environment variable. Default: empty
  --db TEXT                       Path to a local or remote vulnerability
                                  database. Default: empty
  --full-report / --short-report  Full reports include a security advisory (if
                                  available). Default: --short-report NOTE:
                                  This argument is mutually exclusive with
                                  arguments: [bare with values [True, False],
                                  json with values [True, False], output with
                                  values ['json', 'bare']].
  --stdin                         Read input from stdin. NOTE: This argument
                                  is mutually exclusive with  arguments:
                                  [files].
  -r, --file FILENAME             Read input from one (or multiple)
                                  requirement files. Default: empty NOTE: This
                                  argument is mutually exclusive with
                                  arguments: [stdin].
  -i, --ignore TEXT               Ignore one (or multiple) vulnerabilities by
                                  ID. Default: empty
  -o, --output [screen|text|json|bare]
  -pr, --proxy-protocol [http|https]
                                  Proxy protocol (https or http) --proxy-
                                  protocol NOTE: This argument requires the
                                  following flags  [proxy_host].
  -ph, --proxy-host TEXT          Proxy host IP or DNS --proxy-host
  -pp, --proxy-port INTEGER       Proxy port number --proxy-port NOTE: This
                                  argument requires the following flags
                                  [proxy_host].
  --exit-code / --continue-on-error
                                  Output standard exit codes. Default: --exit-
                                  code
  --policy-file FILENAME          Define the policy file to be used
  --audit-and-monitor / --disable-audit-and-monitor
                                  Send results back to pyup.io for viewing on
                                  your dashboard. Requires an API key.
  --project TEXT                  Project to associate this scan with on
                                  pyup.io. Defaults to a canonicalized github
                                  style name if available, otherwise unknown
  --save-json TEXT                Path to where output file will be placed, if
                                  the path is a directory, Safety will use
                                  safety-report.json as filename. Default:
                                  empty
  --help                          Show this message and exit.
```

---

✅ We have installed Safety successfully and explored its options.
➡️ Let’s move to the next step.

```
```
````md
# Run the Scanner

As we have learned in the **DevSecOps Gospel**, we should save the output in a **machine-readable format**.  

We are using the **tee** command to show the output and store it in a file simultaneously.

```bash
safety check -r requirements.txt --json | tee safety-output.json
````

* `-r` flag → specifies the input file (`requirements.txt`).
* `--json` flag → tells Safety to output results in **JSON format**.

---

### Command Output

```json
{
    "report_meta": {
        "scan_target": "files",
        "scanned": [
            "requirements.txt"
        ],
        "policy_file": null,
        "policy_file_source": "local",
        "api_key": false,
        "local_database_path": null,
        "safety_version": "2.3.5",
        "timestamp": "2023-03-05 05:49:22",
        "packages_found": 1,
        "vulnerabilities_found": 26,
        "vulnerabilities_ignored": 0,
        "remediations_recommended": 0,
        ...
    },
    "vulnerabilities": [
        {
            "package_name": "django",
            "installed_version": "3.0",
            "analyzed_version": "3.0",
            "advisory": "Django 3.2.14 and 4.0.6 include a fix for CVE-2022-34265: Potential SQL injection via Trunc(kind) and Extract(lookup_name) arguments.\r\nhttps://www.djangoproject.com/weblog/2022/jul/04/security-releases",
            "is_transitive": false,
            "published_date": null,
            "fixed_versions": [],
            "closest_versions_without_known_vulnerabilities": [],
            "resources": [],
            "CVE": "CVE-2022-34265",
            "severity": null,
            "affected_versions": [],
            "more_info_url": "https://pyup.io/v/49733/f17"
        }
    ],
    "ignored_vulnerabilities": [],
    "remediations": {
        "django": {
            "current_version": "3.0",
            "vulnerabilities_found": 26,
            "recommended_version": null,
            "other_recommended_versions": [],
            "more_info_url": "https://pyup.io/?from=3.0"
        }
    }
}
```

---

### Variable Scan Results

⚠️ Your results might slightly vary because of the dynamic landscape of changing vulnerabilities, and security updates.
````md
# Software Component Analysis Using RetireJS

## Download the Source Code
We will do all the exercises locally first in DevSecOps-Box, so let’s start the exercise.

First, We need to download the source code of the project from our git repository.

```bash
git clone https://gitlab.practical-devsecops.training/pdso/django.nv webapp
````

**Command Output**

```
Cloning into 'webapp'...
warning: redirecting to https://gitlab.practical-devsecops.training/pdso/django.nv.git/
remote: Enumerating objects: 299, done.
remote: Counting objects: 100% (71/71), done.
remote: Compressing objects: 100% (68/68), done.
remote: Total 299 (delta 34), reused 0 (delta 0), pack-reused 228
Receiving objects: 100% (299/299), 1.05 MiB | 29.16 MiB/s, done.
Resolving deltas: 100% (120/120), done.
```

Let’s cd into the application code so we can scan the app.

```bash
cd webapp
```

We are now in the webapp directory.

Let’s move to the next step.

```
```
````md
# Installing RetireJS

There are lots of JavaScript libraries for use on the Web and in Node.JS apps. The availability of these libraries greatly simplifies development, but also increases the security risks. Because of rampant abuse of these libraries, attackers are concentrating on these libraries. Realizing the need for awareness, OWASP has included **“Using Components with Known Vulnerabilities”** as part of the OWASP Top 10 list.

**RetireJS** helps you in finding the insecure Javascript libraries in your code.

**Source:** [https://github.com/retirejs/retire.js](https://github.com/retirejs/retire.js)

---

## Step 1: Install NodeJS and NPM
```bash
mkdir -p /etc/apt/keyrings
curl -fsSL https://deb.nodesource.com/gpgkey/nodesource-repo.gpg.key | gpg --dearmor -o /etc/apt/keyrings/nodesource.gpg
NODE_MAJOR=20
echo "deb [signed-by=/etc/apt/keyrings/nodesource.gpg] https://deb.nodesource.com/node_$NODE_MAJOR.x nodistro main" | tee /etc/apt/sources.list.d/nodesource.list
apt update

apt install nodejs -y
````

---

## Step 2: Install RetireJS

Then we can install the RetireJS tool on the system to find the insecure Javascript libraries.

```bash
npm install -g retire@5.0.0
```

**Command Output**

```
added 41 packages in 2s

2 packages are looking for funding
  run `npm fund` for details
npm notice
npm notice New minor version of npm available! 10.7.0 -> 10.8.1
npm notice Changelog: https://github.com/npm/cli/releases/tag/v10.8.1
npm notice To update run: npm install -g npm@10.8.1
npm notice
```

---

## Step 3: Explore RetireJS Options

```bash
retire --help
```

**Command Output**

```
Usage: retire [options]

Options:
  -V, --version            output the version number
  -v, --verbose            Show identified files (by default only vulnerable files are shown)
  -c, --nocache            Don't use local cache
  --jspath <path>          Folder to scan for javascript files (deprecated)
  --path <path>            Folder to scan for javascript files
  --jsrepo <path|url>      Local or internal version of repo. Can be multiple comma separated. Default: 'central')
  --cachedir <path>        Path to use for local cache instead of /tmp/.retire-cache
  --proxy <url>            Proxy url (http://some.host:8080)
  --outputformat <format>  Valid formats: text, json, jsonsimple, depcheck (experimental), cyclonedx and cyclonedxJSON
  --outputpath <path>      File to which output should be written
  --ignore <paths>         Comma delimited list of paths to ignore
  --ignorefile <path>      Custom ignore file, defaults to .retireignore / .retireignore.json
  --severity <level>       Specify the bug severity level from which the process fails. Allowed levels none, low, medium, high, critical. Default:
                           none
  --exitwith <code>        Custom exit code (default: 13) when vulnerabilities are found
  --colors                 Enable color output (console output only)
  --insecure               Enable fetching remote jsrepo/noderepo files from hosts using an insecure or self-signed SSL (TLS) certificate
  --ext <extensions>       Comma separated list of file extensions for JavaScript files. The default is "js"
  --cacert <path>          Use the specified certificate file to verify the peer used for fetching remote jsrepo/noderepo files
  --includeOsv             Include OSV advisories in the output
  --deep                   Deep scan (slower and experimental)
  -h, --help               display help for command
```

---

We have successfully installed **RetireJS**.
Let’s move to the next step.

```
```
````md
# Run the Scanner

Front-end dependencies managed via npm are stored in the package.json file. Let’s explore the contents of the package.json file.

```bash
cat package.json
````

As we can see, we are using various JavaScript libraries in this project. Let’s find out if we are using any vulnerable libraries.

```bash
retire --outputformat json --outputpath no-npm-install-retire-output.json
```

As we have learned in the DevSecOps Gospel, we should always try to save the tool’s output in a machine-readable format. This output will enable us to parse it via script(s).

* `--outputpath` : flag specifies the output file path.
* `--outputformat` : flag specifies that output format. Here it’s the JSON format.

If you are using an older version of RetireJS, the tool will prompt an error asking you to do npm install before scanning.

However, in the current version, the error is not displayed. It will scan available JS components in our source code, but we still need to run npm install to perform a thorough scan of npm components.

Let’s take a look at the output of the scan before installing the npm package.

```bash
cat no-npm-install-retire-output.json | jq .
```

**Command Output**

```
...[SNIP]...,

            {
              "info": [
                "https://blog.jquery.com/2020/04/10/jquery-3-5-0-released/"
              ],
              "below": "3.5.0",
              "atOrAbove": "1.2.0",
              "severity": "medium",
              "identifiers": {
                "summary": "Regex in its jQuery.htmlPrefilter sometimes may introduce XSS",
                "CVE": [
                  "CVE-2020-11022"
                ],
                "issue": "4642",
                "githubID": "GHSA-gxr4-xjj5-5px2"
              }
            }
          ]
        }
      ]
    }
  ],
  "messages": [],
  "errors": [],
  "time": 0.162
}
```

The number may vary, as it is dynamic.

Next, let’s install the npm component using the command below.

```bash
npm install
```

**Command Output**

```
...[SNIP]...

added 921 packages, and audited 1371 packages in 22s

69 packages are looking for funding
  run `npm fund` for details

104 vulnerabilities (3 low, 31 moderate, 54 high, 16 critical)

To address issues that do not require attention, run:
  npm audit fix

To address all issues possible (including breaking changes), run:
  npm audit fix --force

Some issues need review, and may require choosing
a different dependency.

Run `npm audit` for details.
```

Then, re-run the previous command to scan the dependencies.

```bash
retire --outputformat json --outputpath retire_output.json
```

Once done, you can use the cat command to see the RetireJS output with the help of the jq command.

```bash
cat retire_output.json | jq .
```

**Command Output**

```
...[SNIP]...

            {
              "info": [
                "https://github.com/advisories/GHSA-5359-pvf2-pw78",
                "https://github.com/tinymce/tinymce/security/advisories/GHSA-5359-pvf2-pw78",
                "https://nvd.nist.gov/vuln/detail/CVE-2024-29881",
                "https://github.com/tinymce/tinymce/commit/bcdea2ad14e3c2cea40743fb48c63bba067ae6d1",
                "https://github.com/tinymce/tinymce",
                "https://www.tiny.cloud/docs/tinymce/6/6.8.1-release-notes/#new-convert_unsafe_embeds-option-that-controls-whether-object-and-embed-elements-will-be-converted-to-more-restrictive-alternatives-namely-img-for-image-mime-types-video-for-video-mime-types-audio-audio-mime-types-or-iframe-for-other-or-unspecified-mime-types",
                "https://www.tiny.cloud/docs/tinymce/7/7.0-release-notes/#convert_unsafe_embeds-editor-option-is-now-defaulted-to-true"
              ],
              "below": "7.0.0",
              "atOrAbove": "0",
              "severity": "medium",
              "identifiers": {
                "summary": "TinyMCE Cross-Site Scripting (XSS) vulnerability in handling external SVG files through Object or Embed elements",
                "CVE": [
                  "CVE-2024-29881"
                ],
                "githubID": "GHSA-5359-pvf2-pw78"
              }
            }
          ]
        }
      ]
    }
  ],
  "messages": [],
  "errors": [],
  "time": 7.772
}
```

Did you notice the difference in the scanner output between running it without `npm install` and with `npm install`?

Let’s move to the next step.

```
```
````md
# Understanding the Behavior of Severity Flags

Each security tool may have a specific flag to filter the severity of findings. In this step, we will explore into how this works using RetireJS.

As shown in the second step, we can use the --severity argument to specify the severity level we want to filter. Let’s give it a try by using this command:

```bash
retire --severity critical --outputformat json --outputpath retire_output.json
````

We have successfully run the scan. Let’s check if the output contains only the severity level we are interested in, which is critical.

```bash
cat retire_output.json | jq .
```

You might be wondering why retire\_output.json still contains low, medium and high severity issues.

This behavior is not a bug but intentional.

**Note**

Each security tool behaves differently, and you might need to explore it further if something doesn’t work as expected.

So, what is the function of the severity flag here?

Let’s experiment with the exit codes that RetireJS provides when we filter the severity for critical and medium issues. You will notice the difference in exit codes when issues are found.

First, filter for critical issues using the command below:

```bash
retire --severity critical --outputformat json --outputpath retire_output.json
```

Then, check the exit code.

```bash
echo $?
```

**Command Output**

```
0
```

This shows 0, indicating no issues found with critical severity. Now, how about filtering for medium issues?

```bash
retire --severity medium --outputformat json --outputpath retire_output.json
```

Let’s check the exit code one more time.

```bash
echo $?
```

**Command Output**

```
13
```

Interesting! The exit code often returns 13 instead of 1 because 13 is the default exit code set by the RetireJS tool.

It’s important to note that an exit code isn’t always 1 when the tool finds security issues, it could be any number chosen by the tool’s developer.

Since each developer or organization may use different exit codes, we primarily focus on whether it’s zero or non-zero. For instance, the argument we used to filter severity returned an exit code of 13.

Let’s move to the next step.

```
```
````md
# Challenges
In this challenge, you will explore and use the advanced features of RetireJS.

## Tasks

### # 1
Configure RetireJS such that it only throws non zero exit code when high severity issues are present in the results

**Answer**

When running RetireJS, use the severity argument.

```bash
retire --severity high --outputformat json --outputpath retire_output.json
````

Confused why retire\_output.json has medium and low severity issues? Then you need to re-read what the --severity flag does in retirejs. Its not a bug but a functionality.

Please note that an exit code is not always 1 even though the tool found security issues. It can be any number pre-defined by the developer of the tool.

Since every developer/organization is different so are the exit codes. We only care if its zero or non-zero. For example, if you noticed, the argument that we used to filter severity would return exit code 13.

---

### # 2

Mark all high severity issues as False Positive and save the output in JSON format at location `/webapp/retire_output.json`

**Answer**

Below marks every **high**-severity finding in RetireJS output as a false positive by adding `"false_positive": true` to those entries, and writes the final JSON to `/webapp/retire_output.json`.

> Prereqs: ensure `jq` is installed (`apt-get update && apt-get install -y jq`) and you’ve run `npm install` in the repo so RetireJS sees all dependencies.

```bash
# 1) Run RetireJS and capture raw JSON first
retire --outputformat json --outputpath /webapp/retire_raw.json

# 2) Add a "false_positive": true flag to all HIGH-severity findings
jq '(.data[]?.results[]?.vulnerabilities[]? | select(.severity=="high")) += {false_positive: true}' \
  /webapp/retire_raw.json > /webapp/retire_output.json

# (Optional) sanity-check just the high items now marked
jq '.data[]?.results[]?.vulnerabilities[]? | select(.severity=="high") | {severity, false_positive, identifiers}' \
  /webapp/retire_output.json
```

Notes:

* We do not remove or hide the findings; we explicitly annotate them as **false positives** in the JSON so the context is preserved.
* If your RetireJS JSON schema differs, adjust the `jq` path `.data[]?.results[]?.vulnerabilities[]?` accordingly.

```
```
````md
# How To Embed Safety Into GitLab

## A Simple CI/CD Pipeline
Considering your DevOps team created a simple CI pipeline with the following contents. Please add the Safety scan to the below Gitlab CI Script.

**Click anywhere to copy**

```yaml
image: docker:20.10  # To run all jobs in this pipeline, use the latest docker image

services:
  - docker:dind       # To run all jobs in this pipeline, use a docker image that contains a docker daemon running inside (dind - docker in docker). Reference: https://forum.gitlab.com/t/why-services-docker-dind-is-needed-while-already-having-image-docker/43534

stages:
  - build
  - test
  - release
  - preprod
  - integration
  - prod

build:
  stage: build
  image: python:3.6
  before_script:
   - pip3 install --upgrade virtualenv
  script:
   - virtualenv env                       # Create a virtual environment for the python application
   - source env/bin/activate              # Activate the virtual environment
   - pip install -r requirements.txt      # Install the required third party packages as defined in requirements.txt
   - python manage.py check               # Run checks to ensure the application is working fine

test:
  stage: test
  image: python:3.6
  before_script:
   - pip3 install --upgrade virtualenv
  script:
   - virtualenv env
   - source env/bin/activate
   - pip install -r requirements.txt
   - python manage.py test taskManager
````

There are some jobs in the pipeline, and we need to embed the OAST tool in this pipeline.

Let’s login into GitLab using the following details and execute this pipeline once again.

**GitLab CI/CD Machine**

| Name     | Value                                                                                                                                                                                                          |
| -------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| URL      | [https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/django-nv/-/blob/main/.gitlab-ci.yml](https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/django-nv/-/blob/main/.gitlab-ci.yml) |
| Username | root                                                                                                                                                                                                           |
| Password | pdso-training                                                                                                                                                                                                  |

Next, we need to create a CI/CD pipeline by replacing the .gitlab-ci.yml file content with the above CI script. Click on the Edit button to replace the content (use Control+A and Control+V).

Save changes to the file using the Commit changes button.

### Verify the pipeline run

As soon as a change is made to the repository, the pipeline starts executing the jobs.

We can see the results of this pipeline by visiting [https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/django-nv/pipelines](https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/django-nv/pipelines).

Click on the appropriate job name to see the output.

### Notes

We need to share the project’s source code inside the container for it to access these files. We can do that using the -v option of the docker command.

```
-v $(pwd):/src
```

The above option mounts the current directory in the host (runner) to /src inside the container. This could also be -v /home/ubuntu/code:/src or c:\Users\student\code:/src if you were using windows.

---

## Challenge

Recall techniques you have learned in the previous module (Secure SDLC and CI/CD).

**Note**

Please try to do this exercise without looking at the solution on the next page.

### Tasks

#### # 1

Create a job named oast in the test stage using the safety tool. Make sure you are using hysnsec/safety docker image for this task. Understand the use of Docker’s -v (volume mount) flag/option. Ensure you follow the DevSecOps Gospel and best practices while embedding the safety tool.

---

### Answer

Add this job to .gitlab-ci.yml

**Click anywhere to copy**

```yaml
oast:
  stage: test
  script:
    # We are going to pull the hysnsec/safety image to run the safety scanner
    - docker pull hysnsec/safety
    # third party components are stored in requirements.txt for python, so we will scan this particular file with safety.
    - docker run --rm -v $(pwd):/src hysnsec/safety check -r requirements.txt --json > oast-results.json
  artifacts:
    paths: [oast-results.json]
    when: always # What does this do?
  allow_failure: true
```

```
```

✅ As you can see, we found **multiple security issues** in third-party components.

➡️ Let’s move to the next step.

```
```
```md
Embed Safety in CI/CD pipeline
As discussed in the SCA using the Safety exercise, we can embed the Safety tool in our CI/CD pipeline.

However, do remember that you need run commands manually in a local system like devsecops box before embedding commands for automated execution in CI/CD pipelines.

Manually testing commands in a local environment like devsecops box helps in easy troubleshooting.

Do you wonder which stage this job should go into?

Most of the code (up to 95%) in any software project is open-source/third-party components. It makes sense to perform SCA scans before static analysis.

Click anywhere to copy

image: docker:20.10  # To run all jobs in this pipeline, use the latest docker image

services:
  - docker:dind       # To run all jobs in this pipeline, use a docker image that contains a docker daemon running inside (dind - docker in docker). Reference: https://forum.gitlab.com/t/why-services-docker-dind-is-needed-while-already-having-image-docker/43534

stages:
  - build
  - test
  - release
  - preprod
  - integration
  - prod

build:
  stage: build
  image: python:3.6
  before_script:
   - pip3 install --upgrade virtualenv
  script:
   - virtualenv env                       # Create a virtual environment for the python application
   - source env/bin/activate              # Activate the virtual environment
   - pip install -r requirements.txt      # Install the required third party packages as defined in requirements.txt
   - python manage.py check               # Run checks to ensure the application is working fine

test:
  stage: test
  image: python:3.6
  before_script:
   - pip3 install --upgrade virtualenv
  script:
   - virtualenv env
   - source env/bin/activate
   - pip install -r requirements.txt
   - python manage.py test taskManager

oast:
  stage: test
  script:
    - docker pull hysnsec/safety  # We are going to pull the hysnsec/safety image to run the safety scanner
    # third party components are stored in requirements.txt for python, so we will scan this particular file with safety.
    - docker run --rm -v $(pwd):/src hysnsec/safety check -r requirements.txt --json > oast-results.json
  artifacts:
    paths: [oast-results.json]
    when: always # What does this do?

integration:
  stage: integration
  script:
    - echo "This is an integration step."
    - exit 1
  allow_failure: true # Even if the job fails, continue to the next stages

prod:
  stage: prod
  script:
    - echo "This is a deploy step."
  when: manual # Continuous Delivery

Instead of manually removing the container after the scan, we can use the --rm option so the container can clean up after itself.

Copy the above CI script and add it to the .gitlab.ci.yml file on Gitlab repo at https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/django-nv/-/blob/main/.gitlab-ci.yml

Do not forget to click on the “Commit Changes” button to save the file.

As discussed, any change to the repo kick starts the pipeline.

Note

We’ve discussed different methods of using the tool:

Native Installation:
Directly installing the tool on the system.
Package Manager or Binary:
Installing via a package manager or using the binary file.
Docker:
Running the tool within a Docker container.
In summary:

All methods are suitable for CI/CD integration.
Docker is recommended for CI/CD as it operates smoothly without dependencies.
Using the binary file is efficient, avoiding additional dependencies.
Ultimately, you can choose either method based on your specific situation.

We can see the results of this pipeline by visiting https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/django-nv/pipelines.

Click on the appropriate job name to see the output.

Remember!

Did you notice the job failed because of non zero exit code(here, exit code 64)?

Why did the job fail? Because the tool found security issues. This is the expected behavior of any DevSecOps friendly tool.

You need to either fix the security issues or add allow_failure: true to let the job finish without blocking other jobs.

If you have troubles understanding exit code, please go through the exercise titled Working with Exit Code.

You will notice that the oast job stores the output to a file oast-results.json. We need the output file to process the results further either via APIs or vulnerability management systems like DefectDojo.

In the next step, you will learn the need to not fail the builds.
```
```md
Allow the job failure
Remember! Except for DevSecOps-Box, every other machine closes after two hours, even if you are in the middle of the exercise. You need to refresh the exercise page and click on Start the Exercise button to continue working on it.

You do not want to fail the builds in DevSecOps Maturity Levels 1 and 2. If a security tool fails a build upon security findings, you would want to allow it to fail and not block the pipeline as there would be false positives in the results.

You can use the allow_failure tag to “not fail the build” even though the tool found issues.

oast:
  stage: test
  script:
    - docker pull hysnsec/safety
    - docker run --rm -v $(pwd):/src hysnsec/safety check -r requirements.txt --json > oast-results.json
  artifacts:
    paths: [oast-results.json]
    when: always        # What does this do?
  allow_failure: true   # <--- allow the build to fail but don't mark it as such
The pipeline would look like the following.

Click anywhere to copy

image: docker:20.10  # To run all jobs in this pipeline, use the latest docker image

services:
  - docker:dind       # To run all jobs in this pipeline, use a docker image that contains a docker daemon running inside (dind - docker in docker). Reference: https://forum.gitlab.com/t/why-services-docker-dind-is-needed-while-already-having-image-docker/43534

stages:
  - build
  - test
  - release
  - preprod
  - integration
  - prod

build:
  stage: build
  image: python:3.6
  before_script:
   - pip3 install --upgrade virtualenv
  script:
   - virtualenv env                       # Create a virtual environment for the python application
   - source env/bin/activate              # Activate the virtual environment
   - pip install -r requirements.txt      # Install the required third party packages as defined in requirements.txt
   - python manage.py check               # Run checks to ensure the application is working fine

test:
  stage: test
  image: python:3.6
  before_script:
   - pip3 install --upgrade virtualenv
  script:
   - virtualenv env
   - source env/bin/activate
   - pip install -r requirements.txt
   - python manage.py test taskManager

oast:
  stage: test
  script:
    # We are going to pull the hysnsec/safety image to run the safety scanner
    - docker pull hysnsec/safety
    # third party components are stored in requirements.txt for python, so we will scan this particular file with safety.
    - docker run --rm -v $(pwd):/src hysnsec/safety check -r requirements.txt --json > oast-results.json
  artifacts:
    paths: [oast-results.json]
    when: always # What does this do?
  allow_failure: true

integration:
  stage: integration
  script:
    - echo "This is an integration step"
    - exit 1
  allow_failure: true # Even if the job fails, continue to the next stages

prod:
  stage: prod
  script:
    - echo "This is a deploy step."
  when: manual # Continuous Delivery

Go ahead and add it to the .gitlab-ci.yml file to run the pipeline.

You will notice that the oast job has failed but didn’t block the other jobs from running.

As discussed, any change to the repo kick starts the pipeline.

We can see the results of this pipeline by visiting https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/django-nv/pipelines.

Click on the appropriate job name to see the output.

Understanding -v option in docker command
In docker command above, you can see -v tag to bind the volume with the following:

Command Output
-v $(pwd):/src
-v: This option is used to mount a volume from the host machine to the Docker container.
$(pwd): A shell command that outputs the current working directory on your local machine.
/src: Specifies the path inside the container where the current working directory will be mounted.
When running a specific Docker command with the -v option, Docker will mount the current directory into a specific directory inside the container. In this case, the local directory will be mounted to /src. In practice, if we open the container, we will be redirected to the / directory.

For some tools, we need to define and specify the file path, for example, /src/filename.json, so that the output file can be reflected in our current local directory.

For further explanation, you can review our “Manage Data in Docker” exercise.
```
```md
How To Embed RetireJS Into GitLab
A Simple CI/CD Pipeline
Considering your DevOps team created a simple CI pipeline with the following contents. Please add the Retire.js scan to this pipeline.

Click anywhere to copy

image: docker:20.10  # To run all jobs in this pipeline, use the latest docker image

services:
  - docker:dind       # To run all jobs in this pipeline, use a docker image that contains a docker daemon running inside (dind - docker in docker). Reference: https://forum.gitlab.com/t/why-services-docker-dind-is-needed-while-already-having-image-docker/43534

stages:
  - build
  - test
  - release
  - preprod
  - integration
  - prod

build:
  stage: build
  image: python:3.6
  before_script:
   - pip3 install --upgrade virtualenv
  script:
   - virtualenv env                       # Create a virtual environment for the python application
   - source env/bin/activate              # Activate the virtual environment
   - pip install -r requirements.txt      # Install the required third party packages as defined in requirements.txt
   - python manage.py check               # Run checks to ensure the application is working fine

test:
  stage: test
  image: python:3.6
  before_script:
   - pip3 install --upgrade virtualenv
  script:
   - virtualenv env
   - source env/bin/activate
   - pip install -r requirements.txt
   - python manage.py test taskManager

Let’s log into GitLab using the following details and run the above pipeline.

GitLab CI/CD Machine
Name	Value
Link	https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/django-nv/-/blob/main/.gitlab-ci.yml
Username	root
Password	pdso-training
Next, we need to create a CI/CD pipeline by replacing the .gitlab-ci.yml file content with the above CI script. Click on the Edit button to replace the content (use Control+A and Control+V).

Save changes to the file using the Commit changes button.

Verify the pipeline runs
As soon as a change is made to the repository, the pipeline starts executing the jobs.

We can see the results of this pipeline by visiting https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/django-nv/pipelines.

Click on the appropriate job name to see the output.

Challenge
Recall techniques you have learned in the previous module (Secure SDLC and CI/CD).

Tasks
# 1
Embed Retire.js in the test stage with job named oast-frontend. You can make use of any container image of your choice, including the node image.


Answer

Add this job to .gitlab-ci.yml

Click anywhere to copy

oast-frontend:
  stage: test
  image: node:alpine3.10
  script:
    - npm install             # Install the dependencies before scans
    - npm install -g retire@5.0.0   # Install retirejs tool
    - retire --outputformat json --outputpath retirejs-report.json --severity high

# 2
Save the output as JSON format with name retirejs-report.json and upload it using artifacts attribute


Answer

oast-frontend:
  stage: test
  image: node:alpine3.10
  script:
    - npm install
    - npm install -g retire@5.0.0
    - retire --outputformat json --outputpath retirejs-report.json --severity high
  artifacts:
    paths: [retirejs-report.json]
    when: always
    expire_in: one week # Optional
```
```md
Embed RetireJS in the CI/CD pipeline
As discussed in the SCA using the RetireJS exercise, we can embed the RetireJS tool in our CI/CD pipeline.

However, do remember that you need to run commands manually in a local system like a DevSecOps-Box before embedding commands for automated execution in CI/CD pipelines.

Manually testing commands in a local environment like DevSecOps-Box helps in easy troubleshooting.

Do you wonder which stage this job should go into?

Maybe you want to scan the components before performing SAST scans.

Click anywhere to copy

image: docker:20.10  # To run all jobs in this pipeline, use the latest docker image

services:
  - docker:dind       # To run all jobs in this pipeline, use a docker image that contains a docker daemon running inside (dind - docker in docker). Reference: https://forum.gitlab.com/t/why-services-docker-dind-is-needed-while-already-having-image-docker/43534

stages:
  - build
  - test
  - release
  - preprod
  - integration
  - prod

build:
  stage: build
  image: python:3.6
  before_script:
   - pip3 install --upgrade virtualenv
  script:
   - virtualenv env                       # Create a virtual environment for the python application
   - source env/bin/activate              # Activate the virtual environment
   - pip install -r requirements.txt      # Install the required third party packages as defined in requirements.txt
   - python manage.py check               # Run checks to ensure the application is working fine

test:
  stage: test
  image: python:3.6
  before_script:
   - pip3 install --upgrade virtualenv
  script:
   - virtualenv env
   - source env/bin/activate
   - pip install -r requirements.txt
   - python manage.py test taskManager

oast-frontend:
  stage: test
  image: node:alpine3.10
  script:
    - npm install
    - npm install -g retire@5.0.0 # Install retirejs npm package.
    - retire --outputformat json --outputpath retirejs-report.json --severity high
  artifacts:
    paths: [retirejs-report.json]
    when: always # What is this for?
    expire_in: one week

integration:
  stage: integration
  script:
    - echo "This is an integration step."
    - exit 1
  allow_failure: true # Even if the job fails, continue to the next stages

prod:
  stage: prod
  script:
    - echo "This is a deploy step."
  when: manual # Continuous Delivery

Copy the above CI script and add it to the .gitlab.ci.yml file on Gitlab repo at https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/django-nv/-/blob/main/.gitlab-ci.yml.

Do not forget to click on the Commit Changes button to save the file.

As discussed, any change to the repo kick starts the pipeline.

Note

We’ve discussed different methods of using the tool:

Native Installation:
Directly installing the tool on the system.
Package Manager or Binary:
Installing via a package manager or using the binary file.
Docker:
Running the tool within a Docker container.
In summary:

All methods are suitable for CI/CD integration.
Docker is recommended for CI/CD as it operates smoothly without dependencies.
Using the binary file is efficient, avoiding additional dependencies.
Ultimately, you can choose either method based on your specific situation.

We can see the results of this pipeline by visiting https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/django-nv/pipelines.

Click on the appropriate job name to see the output.

Remember!

Did you notice the job failed because of non-zero exit code(exit 13)?

Why did the job fail? Because the tool found security issues. This is the expected behavior of any DevSecOps-friendly tool.

You need to either fix the security issues or add allow_failure: true to let the job finish without blocking other jobs.

If you have trouble understanding the exit code, please go through the exercise titled Working with Exit Code.

You will notice that the oast-frontend job stores the output to a file retirejs-report.json. This is done to ensure we can process the results further via APIs or vulnerability management systems like DefectDojo.

In the next step, you will learn the need to not fail the builds.
```
```md
Allow the job failure
Remember!

Except for DevSecOps-Box, every other machine closes after two hours, even if you are in the middle of the exercise
After two hours, in case of a 404, you need to refresh the exercise page and click on Start the Exercise button to continue working
You do not want to fail the builds in DevSecOps Maturity Levels 1 and 2. If a security tool fails a build upon security findings, you would want to allow it to fail and not block the pipeline as there would be false positives in the results.

You can use the allow_failure tag to not fail the build even though the tool found security issues.

oast-frontend:
  stage: test
  image: node:alpine3.10
  script:
    - npm install
    - npm install -g retire@5.0.0 # Install retirejs npm package.
    - retire --outputformat json --outputpath retirejs-report.json --severity high
  artifacts:
    paths: [retirejs-report.json]
    when: always # What is this for?
    expire_in: one week
  allow_failure: true  #<--- allow the build to fail but don't mark it as such
Notice allow_failure: true at the end of the YAML file.

The final pipeline would look like the following.

Click anywhere to copy

image: docker:20.10  # To run all jobs in this pipeline, use the latest docker image

services:
  - docker:dind       # To run all jobs in this pipeline, use a docker image that contains a docker daemon running inside (dind - docker in docker). Reference: https://forum.gitlab.com/t/why-services-docker-dind-is-needed-while-already-having-image-docker/43534

stages:
  - build
  - test
  - release
  - preprod
  - integration
  - prod

build:
  stage: build
  image: python:3.6
  before_script:
   - pip3 install --upgrade virtualenv
  script:
   - virtualenv env                       # Create a virtual environment for the python application
   - source env/bin/activate              # Activate the virtual environment
   - pip install -r requirements.txt      # Install the required third party packages as defined in requirements.txt
   - python manage.py check               # Run checks to ensure the application is working fine

test:
  stage: test
  image: python:3.6
  before_script:
   - pip3 install --upgrade virtualenv
  script:
   - virtualenv env
   - source env/bin/activate
   - pip install -r requirements.txt
   - python manage.py test taskManager

oast-frontend:
  stage: test
  image: node:alpine3.10
  script:
    - npm install
    - npm install -g retire@5.0.0 # Install retirejs npm package.
    - retire --outputformat json --outputpath retirejs-report.json --severity high
  artifacts:
    paths: [retirejs-report.json]
    when: always # What is this for?
    expire_in: one week
  allow_failure: true

integration:
  stage: integration
  script:
    - echo "This is an integration step"
    - exit 1
  allow_failure: true # Even if the job fails, continue to the next stages

prod:
  stage: prod
  script:
    - echo "This is a deploy step."
  when: manual # Continuous Delivery

Go ahead and add it to the .gitlab-ci.yml file to run the pipeline.

You will notice that the oast-frontend job has failed but didn’t block the other jobs from running.

As discussed, any change to the repo kick starts the pipeline.

We can see the results of this pipeline by visiting https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/django-nv/pipelines.

Click on the appropriate job name to see the output.
```
````md
# Fix the Issues Reported by Safety Tool

## Download the source code
We will do all the exercises locally first in DevSecOps-Box, so let’s start the exercise.

First, we need to download the source code of the project from our git repository.

```bash
git clone https://gitlab.practical-devsecops.training/pdso/django.nv webapp
````

### Command Output

```
Cloning into 'webapp'...
warning: redirecting to https://gitlab.practical-devsecops.training/pdso/django.nv.git/
remote: Enumerating objects: 299, done.
remote: Counting objects: 100% (71/71), done.
remote: Compressing objects: 100% (68/68), done.
remote: Total 299 (delta 34), reused 0 (delta 0), pack-reused 228
Receiving objects: 100% (299/299), 1.05 MiB | 29.16 MiB/s, done.
Resolving deltas: 100% (120/120), done.
```

Let’s cd into the application code, so we can scan the app.

```bash
cd webapp
```

We are now in the webapp directory.

---

Let’s move to the next step.

```
```
````md
# Install Safety

Safety checks your installed dependencies for known security vulnerabilities.  
By default, it uses the open Python vulnerability database Safety DB but can be upgraded to use pyup.io’s Safety API using the –key option.

Source: [https://pypi.org/project/safety](https://pypi.org/project/safety)

You can find more details about the project at Safety.

---

## Install the Safety tool

Let’s install the safety tool on the system to scan the Python dependencies.

```bash
pip3 install safety==2.3.5
````

### Command Output

```
Collecting safety==2.3.5
  Downloading safety-2.3.5-py3-none-any.whl (57 kB)
     |████████████████████████████████| 57 kB 8.0 MB/s 
Requirement already satisfied: setuptools>=19.3 in /usr/local/lib/python3.8/dist-packages (from safety==2.3.5) (67.4.0)
Collecting Click>=8.0.2
  Downloading click-8.1.3-py3-none-any.whl (96 kB)
     |████████████████████████████████| 96 kB 9.9 MB/s 
Requirement already satisfied: requests in /usr/local/lib/python3.8/dist-packages (from safety==2.3.5) (2.28.2)

...[SNIP]...

Successfully installed Click-8.1.3 dparse-0.6.2 packaging-21.3 pyparsing-3.0.9 ruamel.yaml-0.17.21 ruamel.yaml.clib-0.2.7 safety-2.3.5 toml-0.10.2
```

---

## Explore Safety options

```bash
safety check --help
```

### Command Output

```
Usage: safety check [OPTIONS]

  Find vulnerabilities in Python dependencies at the target provided.

Options:
  --key TEXT                      API Key for pyup.io's vulnerability
                                  database. Can be set as SAFETY_API_KEY
                                  environment variable. Default: empty
  --db TEXT                       Path to a local or remote vulnerability
                                  database. Default: empty
  --full-report / --short-report  Full reports include a security advisory (if
                                  available). Default: --short-report NOTE:
                                  This argument is mutually exclusive with
                                  arguments: [bare with values [True, False],
                                  json with values [True, False], output with
                                  values ['json', 'bare']].
  --stdin                         Read input from stdin. NOTE: This argument
                                  is mutually exclusive with  arguments:
                                  [files].
  -r, --file FILENAME             Read input from one (or multiple)
                                  requirement files. Default: empty NOTE: This
                                  argument is mutually exclusive with
                                  arguments: [stdin].
  -i, --ignore TEXT               Ignore one (or multiple) vulnerabilities by
                                  ID. Default: empty
  -o, --output [screen|text|json|bare]
  -pr, --proxy-protocol [http|https]
                                  Proxy protocol (https or http) --proxy-
                                  protocol NOTE: This argument requires the
                                  following flags  [proxy_host].
  -ph, --proxy-host TEXT          Proxy host IP or DNS --proxy-host
  -pp, --proxy-port INTEGER       Proxy port number --proxy-port NOTE: This
                                  argument requires the following flags
                                  [proxy_host].
  --exit-code / --continue-on-error
                                  Output standard exit codes. Default: --exit-
                                  code
  --policy-file FILENAME          Define the policy file to be used
  --audit-and-monitor / --disable-audit-and-monitor
                                  Send results back to pyup.io for viewing on
                                  your dashboard. Requires an API key.
  --project TEXT                  Project to associate this scan with on
                                  pyup.io. Defaults to a canonicalized github
                                  style name if available, otherwise unknown
  --save-json TEXT                Path to where output file will be placed, if
                                  the path is a directory, Safety will use
                                  safety-report.json as filename. Default:
                                  empty
  --help                          Show this message and exit.
```

---

Let’s move to the next step.

```
```
````md
# Run the Scanner

As we have learned in the **DevSecOps Gospel**, we should save the output in a machine-readable format.

We are using the `tee` command to show the output and store it in a file simultaneously.

```bash
safety check -r requirements.txt --json | tee safety-output.json
````

* **-r flag** → used to specify the input file.
* **--json flag** → tells that output should be in the JSON format.

---

## Command Output

```json
{
    "report_meta": {
        "scan_target": "files",
        "scanned": [
            "requirements.txt"
        ],
        "policy_file": null,
        "policy_file_source": "local",
        "api_key": false,
        "local_database_path": null,
        "safety_version": "2.3.5",
        "timestamp": "2023-03-05 05:49:22",
        "packages_found": 1,
        "vulnerabilities_found": 26,
        "vulnerabilities_ignored": 0,
        "remediations_recommended": 0,

...[SNIP]...

            "analyzed_version": "3.0",
            "advisory": "Django 3.2.14 and 4.0.6 include a fix for CVE-2022-34265: Potential SQL injection via Trunc(kind) and Extract(lookup_name) arguments.\r\nhttps://www.djangoproject.com/weblog/2022/jul/04/security-releases",
            "is_transitive": false,
            "published_date": null,
            "fixed_versions": [],
            "closest_versions_without_known_vulnerabilities": [],
            "resources": [],
            "CVE": "CVE-2022-34265",
            "severity": null,
            "affected_versions": [],
            "more_info_url": "https://pyup.io/v/49733/f17"
        }
    ],
    "ignored_vulnerabilities": [],
    "remediations": {
        "django": {
            "current_version": "3.0",
            "vulnerabilities_found": 26,
            "recommended_version": null,
            "other_recommended_versions": [],
            "more_info_url": "https://pyup.io/?from=3.0"
        }
    }
}
```

---

✅ As you can see, we found **multiple security issues** in third-party components.

Let’s move to the next step.

```
```
````md
# Fixing the Issues

The tool found many **vulnerable components** in the source code.  
To fix these issues, we first need to see what components are installed in the system by checking the `requirements.txt` file.

```bash
cat requirements.txt
````

### Command Output

```
Django==3.0
```

---

✅ Only **one component** needs to be upgraded to fix the issues.
You can find out the safe versions of **Django** by visiting:
🔗 [https://pypi.org/project/Django/#history](https://pypi.org/project/Django/#history)

At the time of writing this exercise, the latest updated version is **Django 4.2.21**.

Let’s edit the `requirements.txt` file to use version **4.2.21**:

```bash
cat >requirements.txt<<EOF
Django==4.2.21
EOF
```

---

> ⚠️ **Note**
> You need to talk to the development team to see if they can upgrade the package to a specific version **without breaking the application**.
> The release notes explain why the new version is safer and how to determine what the developer needs in the relevant version.

---

Now, let’s run the scanner once again:

```bash
safety check -r requirements.txt --json | tee safety-output.json
```

### Command Output

```json
{
    "report_meta": {
        "scan_target": "files",
        "scanned": [
            "requirements.txt"
        ],
        "policy_file": null,
        "policy_file_source": "local",
        "api_key": false,
        "local_database_path": null,
        "safety_version": "2.3.5",
        "timestamp": "2023-03-05 05:50:47",
        "packages_found": 0,
        "vulnerabilities_found": 0,
        "vulnerabilities_ignored": 0,
        "remediations_recommended": 0,
        "telemetry": {
            "os_type": "Linux",
            "os_release": "5.15.0-1026-aws",
            "os_description": "Linux-5.15.0-1026-aws-x86_64-with-glibc2.29",
            "python_version": "3.8.10",
            "safety_command": "check",
            "safety_options": {
                "files": {
                    "-r": 1
                },
                "json": {
                    "--json": 1
                }
            },
            "safety_version": "2.3.5",
            "safety_source": "cli"
        },
        "git": {
            "branch": "main",
            "tag": "",
            "commit": "1cb7ab820ee83f93de1eea4f05b923c4bd49caf6",
            "dirty": true,
            "origin": "https://gitlab.practical-devsecops.training/pdso/django.nv"
        },
        "project": null,
        "json_version": 1
    },
    "scanned_packages": {},
    "affected_packages": {},
    "announcements": [],
    "vulnerabilities": [],
    "ignored_vulnerabilities": [],
    "remediations": {}
}
```

---

🎉 **Cool! We fixed the issues** found in our application’s components.

---

## Final Note

* After upgrading, always **check if the application works properly**.
* These fixes may break your application, so please **consult with your development team**.
* As a best practice, ensure the components aren’t vulnerable to **any critical or high issues**.

```
```
````md
# Software Component Analysis Using pip-audit

## Download the Source Code

We will do all the exercises **locally first in DevSecOps-Box**, so let’s start the exercise.

First, we need to download the source code of the project from our git repository:

```bash
git clone https://gitlab.practical-devsecops.training/pdso/django.nv webapp
````

Now, let’s `cd` into the application so we can scan the app:

```bash
cd webapp
```

✅ We are now in the **webapp** directory.

---

Let’s move to the **next step**.

```
```
````md
# Install pip-audit

The **pip-audit** is a tool for scanning Python environments for packages with known vulnerabilities.  
It uses the **Python Packaging Advisory Database** (https://github.com/pypa/advisory-db) via the **PyPI JSON API** as a source of vulnerability reports.

📖 Source: [https://github.com/trailofbits/pip-audit](https://github.com/trailofbits/pip-audit)

---

## Step 1: Install pip-audit

We will install the **pip-audit scanner** to perform static analysis:

```bash
pip3 install pip-audit==1.1.2
````

### Command Output

```
Collecting pip-audit==1.1.2
  Downloading pip_audit-1.1.2-py3-none-any.whl (54 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 54.5/54.5 KB 2.0 MB/s eta 0:00:00
Collecting pip-api>=0.0.26
  Downloading pip_api-0.0.27-py3-none-any.whl (111 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 111.3/111.3 KB 26.0 MB/s eta 0:00:00

...[SNIP]...

Successfully installed CacheControl-0.12.10 certifi-2021.10.8 charset-normalizer-2.0.11 cyclonedx-python-lib-0.12.3 html5lib-1.1 idna-3.3 lockfile-0.12.2 msgpack-1.0.3 packageurl-python-0.9.6 pip-api-0.0.27 pip-audit-1.1.2 progress-1.6 requests-2.27.1 resolvelib-0.8.1 six-1.16.0 toml-0.10.2 types-setuptools-57.4.8 types-toml-0.10.3 urllib3-1.26.8 webencodings-0.5.1
WARNING: Running pip as the 'root' user can result in broken permissions and conflicting behaviour with the system package manager. 
It is recommended to use a virtual environment instead: https://pip.pypa.io/warnings/venv
```

✅ We have successfully installed the **pip-audit scanner**.

---

## Step 2: Explore pip-audit Options

Run the help command to explore available options:

```bash
pip-audit --help
```

### Command Output

```
usage: pip-audit [-h] [-V] [-l] [-r REQUIREMENTS] [-f FORMAT] [-s SERVICE] [-d] [-S] [--desc [{on,off,auto}]] [--cache-dir CACHE_DIR] [--progress-spinner {on,off}] [--timeout TIMEOUT] [--path PATHS]
                 [-v]

audit the Python environment for dependencies with known vulnerabilities

optional arguments:
  -h, --help            show this help message and exit
  -V, --version         show program's version number and exit
  -l, --local           show only results for dependencies in the local environment (default: False)
  -r REQUIREMENTS, --requirement REQUIREMENTS
                        audit the given requirements file; this option can be used multiple times (default: None)
  -f FORMAT, --format FORMAT
                        the format to emit audit results in (choices: columns, json, cyclonedx-json, cyclonedx-xml) (default: columns)
  -s SERVICE, --vulnerability-service SERVICE
                        the vulnerability service to audit dependencies against (choices: osv, pypi) (default: pypi)
  -d, --dry-run         collect all dependencies but do not perform the auditing step (default: False)
  -S, --strict          fail the entire audit if dependency collection fails on any dependency (default: False)
  --desc [{on,off,auto}]
                        include a description for each vulnerability; `auto` defaults to `on` for the `json` format. This flag has no effect on the `cyclonedx-json` or `cyclonedx-xml` formats.
                        (default: auto)
  --cache-dir CACHE_DIR
                        the directory to use as an HTTP cache for PyPI; uses the `pip` HTTP cache by default (default: None)
  --progress-spinner {on,off}
                        display a progress spinner (default: on)
  --timeout TIMEOUT     set the socket timeout (default: 15)
  --path PATHS          restrict to the specified installation path for auditing packages; this option can be used multiple times (default: [])
  -v, --verbose         give more output; this setting overrides the `PIP_AUDIT_LOGLEVEL` variable and is equivalent to setting it to `debug` (default: False)
```

---

➡️ Let’s move to the **next step**.

```
```
```md
Run the scanner
As we have learned in the DevSecOps Gospel, we should save the output in a machine-readable format.

We are using the tee command to show the output, and store it in a file simultaneously.

```

pip-audit -r ./requirements.txt -f json | tee pip-audit-output.json

```

Command Output
```

\[{"name": "django", "version": "1.8.3", "vulns": \[{"id": "PYSEC-2019-16", "fix\_versions": \["1.11.27", "2.2.9"]}, {"id": "PYSEC-2017-9", "fix\_versions": \["1.8.18", "1.9.13", "1.10.7"]}, {"id": "PYSEC-2015-11", "fix\_versions": \["1.7.11", "1.8.7", "1.9rc2"]}, {"id": "PYSEC-2016-3", "fix\_versions": \["1.8.15", "1.9.10"]}, {"id": "PYSEC-2021-98", "fix\_versions": \["2.2.24", "3.1.12", "3.2.4"]}, {"id": "PYSEC-2018-6", "fix\_versions": \["1.8.19", "1.11.11", "2.0.3"]}, {"id": "PYSEC-2016-18", "fix\_versions": \["1.8.16", "1.9.11", "1.10.3"]}, {"id": "PYSEC-2016-15", "fix\_versions": \["1.8.10", "1.9.3"]}, {"id": "PYSEC-2016-16", "fix\_versions": \["1.8.10", "1.9.3"]}, {"id": "PYSEC-2016-2", "fix\_versions": \["1.8.14", "1.9.8", "1.10rc1"]}, {"id": "PYSEC-2016-17", "fix\_versions": \["1.8.16", "1.9.11", "1.10.3"]}, {"id": "PYSEC-2018-5", "fix\_versions": \["1.8.19", "1.11.11", "2.0.3"]}, {"id": "PYSEC-2017-10", "fix\_versions": \["1.8.18", "1.9.13", "1.10.7"]}, {"id": "PYSEC-2015-22", "fix\_versions": \["1.4.22", "1.7.10", "1.8.4"]}]}]

```

As you can see, we found multiple security issues in third-party components.
```
```md
Software Component Analysis Using Dependency-Check
Download the Source Code
We will do all the exercises locally first in DevSecOps-Box, so let’s start the exercise.

First, we need to download the source code of the project from the git repository.

```

git clone [https://github.com/WebGoat/WebGoat.git](https://github.com/WebGoat/WebGoat.git) webapp

```

Let’s move to the next step.
```
```md
# Software Component Analysis Using Dependency-Check

## Install OWASP Dependency Check
Dependency-Check is SCA tool that attempts to detect publicly disclosed vulnerabilities contained within a project’s dependencies.  

You can find more details about the project at [https://github.com/jeremylong/DependencyCheck](https://github.com/jeremylong/DependencyCheck).

---

Before installing Dependency-Check, we need to ensure if java is installed in our system.

```

java -h

```

### Command Output
```

bash: java: command not found

```

It appears that Java is not installed, so we need to proceed with its installation.  
Before that, let’s update apt first.

```

apt update

```

Now, let’s install Java.

```

apt install openjdk-17-jre -y

```

Next, we need to download Dependency Check source code.

```

wget -O /opt/v8.3.1.zip [https://github.com/jeremylong/DependencyCheck/releases/download/v8.3.1/dependency-check-8.3.1-release.zip](https://github.com/jeremylong/DependencyCheck/releases/download/v8.3.1/dependency-check-8.3.1-release.zip)

```

Extract the source code.

```

unzip /opt/v8.3.1.zip -d /opt/

```

We will add the tool to the PATH so that we can reference it on the command line.

```

export PATH=/opt/dependency-check/bin:\$PATH

```

Let’s explore the options that this tool provides us.

```

dependency-check.sh -h

```

### Command Output
```

usage: Dependency-Check Core \[--advancedHelp] \[--enableExperimental]
\[--exclude <pattern>] \[-f <format>] \[--failOnCVSS <score>] \[-h]
\[--junitFailOnCVSS <score>] \[-l <file>] \[-n] \[-o <path>]
\[--prettyPrint] \[--project <name>] \[-s <path>] \[--suppression <file>] \[-v]

Dependency-Check Core can be used to identify if there are any known CVE
vulnerabilities in libraries utilized by an application. Dependency-Check
Core will automatically update required data from the Internet, such as
the CVE and CPE data files from nvd.nist.gov.

```
--advancedHelp              Print the advanced help message.
--enableExperimental        Enables the experimental analyzers.
--exclude <pattern>         Specify an exclusion pattern. This option
                            can be specified multiple times and it
                            accepts Ant style exclusions.
```

-f,--format <format>           The report format (HTML, XML, CSV, JSON,
JUNIT, SARIF, JENKINS, or ALL). The
default is HTML. Multiple format
parameters can be specified.
\--failOnCVSS <score>        Specifies if the build should be failed if
a CVSS score above a specified level is
identified. The default is 11; since the
CVSS scores are 0-10, by default the build
will never fail.
-h,--help                      Print this message.
\--junitFailOnCVSS <score>   Specifies the CVSS score that is
considered a failure when generating the
junit report. The default is 0.
-l,--log <file>                The file path to write verbose logging
information.
-n,--noupdate                  Disables the automatic updating of the
NVD-CVE, hosted-suppressions and RetireJS
data.
-o,--out <path>                The folder to write reports to. This
defaults to the current directory. It is
possible to set this to a specific file
name if the format argument is not set to
ALL.
\--prettyPrint               When specified the JSON and XML report
formats will be pretty printed.
\--project <name>            The name of the project being scanned.
-s,--scan <path>               The path to scan - this option can be
specified multiple times. Ant style paths
are supported (e.g. 'path/\*\*/\*.jar'); if
using Ant style paths it is highly
recommended to quote the argument value.
\--suppression <file>        The file path to the suppression XML file.
This can be specified more then once to
utilize multiple suppression files
-v,--version                   Print the version information.

```

---

Let’s move to the next step.
```
````md
# Run the Scanner

Perform the scan on the **webapp** source code directory using the following command:

```bash
dependency-check.sh --scan webapp --format "JSON" --project "Webgoat" --out /opt
````

Notice, the **format**, **project name** and **output directory**.

---

## Scanning Time

If you’re trying using the latest version of the tool, the scanning process may take **20-30 minutes** depending on the number of files and the complexity of the project.

You can make it faster by using the **NVD API Key** from [https://nvd.nist.gov/developers/request-an-api-key](https://nvd.nist.gov/developers/request-an-api-key).
Without an NVD API Key dependency-check’s updates will be extremely slow.

To begin with, the Dependency-Check tool will update its database before scanning the code, which may take a few minutes.

---

### Command Output

```
[INFO] Checking for updates
[INFO] NVD CVE requires several updates; this could take a couple of minutes.
[INFO] Download Started for NVD CVE - 2002
[INFO] Download Complete for NVD CVE - 2002  (133 ms)

...[SNIP]...

[INFO] Analysis Started
[INFO] Finished Archive Analyzer (0 seconds)
[INFO] Finished File Name Analyzer (0 seconds)
[INFO] Finished Dependency Merging Analyzer (0 seconds)
[INFO] Finished Version Filter Analyzer (0 seconds)
[INFO] Finished Hint Analyzer (0 seconds)
[INFO] Created CPE Index (1 seconds)
[INFO] Finished CPE Analyzer (1 seconds)
[INFO] Finished False Positive Analyzer (0 seconds)
[INFO] Finished NVD CVE Analyzer (0 seconds)
[INFO] Finished RetireJS Analyzer (1 seconds)
[INFO] Finished Sonatype OSS Index Analyzer (0 seconds)
[INFO] Finished Vulnerability Suppression Analyzer (0 seconds)
[INFO] Finished Known Exploited Vulnerability Analyzer (0 seconds)
[INFO] Finished Dependency Bundling Analyzer (0 seconds)
[INFO] Finished Unused Suppression Rule Analyzer (0 seconds)
[INFO] Analysis Complete (3 seconds)
[INFO] Writing report to: /opt/dependency-check-report.json
```

---

As we have learned in the **DevSecOps Gospel**, we should save the output in a **machine-readable format** so the output can be parsed by the machines.

You can see several vulnerabilities found in **jquery** from the output.
Also the report was stored in the `/opt/dependency-check-report.json` file.

---

## Fail on CVSS

In the previous step, if you look carefully, there is an interesting argument named `--failOnCVSS`.
What does it mean?

By including this parameter in the CI/CD pipeline, we can specify a **CVSS score threshold** that, if exceeded, will cause the build to fail.

Normally, the tool does not automatically fail the build despite the presence of vulnerabilities. However, with this parameter, we can enforce a failure condition based on the specified CVSS score.

---

### Example: Using `--failOnCVSS`

```bash
dependency-check.sh --scan webapp --format "JSON" --project "Webgoat" --failOnCVSS 4 --out /opt
```

#### Command Output

```
...[SNIP]...
One or more dependencies were identified with vulnerabilities that have a CVSS score greater than or equal to '4.0': 

bootstrap.min.js: CVE-2018-14042(6.1), CVE-2019-8331(6.1), CVE-2018-14041(6.1), CVE-2016-10735(6.1), CVE-2018-20676(6.1), CVE-2018-20677(6.1)
bootstrap.min.js: CVE-2018-14042(6.1), CVE-2019-8331(6.1), CVE-2018-14041(6.1), CVE-2016-10735(6.1), CVE-2018-20676(6.1), CVE-2018-20677(6.1)
jquery-1.10.2.min.js: CVE-2015-9251(6.1), CVE-2019-11358(6.1), CVE-2020-23064(6.1), CVE-2020-11023(6.1), CVE-2020-11022(6.1)
jquery-2.1.4.min.js: CVE-2015-9251(6.1), CVE-2019-11358(6.1), CVE-2020-23064(6.1), CVE-2020-11023(6.1), CVE-2020-11022(6.1)
jquery-ui-1.10.4.custom.min.js: CVE-2016-7103(6.1), CVE-2022-31160(6.1), CVE-2021-41184(6.1), CVE-2021-41183(6.1), CVE-2021-41182(6.1)
jquery-ui-1.10.4.js: CVE-2016-7103(6.1), CVE-2022-31160(6.1), CVE-2021-41184(6.1), CVE-2021-41183(6.1), CVE-2021-41182(6.1)
jquery-ui.min.js: CVE-2022-31160(6.1), CVE-2021-41184(6.1), CVE-2021-41183(6.1), CVE-2021-41182(6.1)
jquery.min.js: CVE-2020-23064(6.1), CVE-2020-11023(6.1), CVE-2020-11022(6.1)
underscore-min.js: CVE-2021-23358(7.2)

See the dependency-check report for more details.
```

Now, check the exit code:

```bash
echo $?
```

#### Command Output

```
15
```

---

### Example: Without `--failOnCVSS`

```bash
dependency-check.sh --scan webapp --format "JSON" --project "Webgoat" --out /opt
```

```bash
echo $?
```

#### Command Output

```
0
```

---

✅ Great! We have learned something new about the behavior of this tool.

If you are not familiar with the exit code, please refer to the chapter titled **Introduction to the tools of the trade** and the exercise about **Linux Exit Code**.

---

Let’s move to the next step.

```
```
````md
# Challenges

Recall techniques you have learned in the previous modules.

- Explore the different options offered by the Dependency Check tool.  
- At which stage should you include the Dependency Check tool in the CI/CD pipeline?  
- Use the available parameter in the tool to mark the **CVE-2021-23358** issue as a false positive.  
- Try solving the challenges without looking at the solution in the next step.  

---

# Solutions

### At which stage should you include the Dependency Check tool in the CI/CD pipeline?

You can include this tool in the **initial stage** of your CI/CD pipeline, such as the **build stage** or any stage before packaging the application into a single package like a JAR or containerized image.  

By doing so, the organization can ensure that no malicious issues are present in the application’s dependencies or libraries.

---

### Use the available parameter in the tool to mark the CVE-2021-23358 issue as a false positive

We have the option to use the `--suppression` parameter in Dependency Check to mark an issue as a **false positive**.  

This feature allows us to specify that the tool should ignore the marked issue in subsequent scans.  

It functions similarly to the **Baseline feature in the Bandit tool**, which also allows ignoring false positive issues.  

Implementing the `--suppression` parameter helps streamline the scanning process by excluding known false positives from future reports.

---

### Step 1: Create a suppression file

```bash
cat >suppression.xml<<EOF
<?xml version="1.0" encoding="UTF-8"?>
<suppressions xmlns="https://jeremylong.github.io/DependencyCheck/dependency-suppression.1.3.xsd">
    <suppress>
        <notes><![CDATA[
        Ignore underscore-min.js issue as FP
        ]]></notes>
        <vulnerabilityName>CVE-2021-23358</vulnerabilityName>
    </suppress>
</suppressions>
EOF
````

---

### Step 2: Run the scanner using suppression option

```bash
dependency-check.sh --scan webapp --format "JSON" --project "Webgoat" --failOnCVSS 7 --suppression suppression.xml --out /opt
```

The output **no longer mentions the issue**, but if you run the following command **without ignoring** the issue, you will see the below output:

```bash
dependency-check.sh --scan webapp --format "JSON" --project "Webgoat" --failOnCVSS 7 --out /opt
```

#### Command Output

```
...[SNIP]...
One or more dependencies were identified with vulnerabilities that have a CVSS score greater than or equal to '7.0': 

underscore-min.js: CVE-2021-23358(7.2)

See the dependency-check report for more details.
```

```
```
````md
# How To Embed Dependency-Check Into GitLab

## A Simple CI/CD Pipeline

Let’s download the code using **git clone** in DevSecOps Box.  
Once done, we need to push our source code into GitLab.  

> **Note**  
> Before proceeding with the steps below, please wait until the GitLab machine is fully provisioned as the initialization process takes time to automatically add the projects to your GitLab.

```bash
git clone https://github.com/WebGoat/WebGoat.git webgoat
````

```bash
cd webgoat
```

Rename git url to the new one:

```bash
git remote rename origin old-origin
```

```bash
git remote add origin http://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/webgoat.git
```

Then, push the code into the repository:

```bash
git push -u origin --all
```

And enter the GitLab credentials:

| Name     | Value         |
| -------- | ------------- |
| Username | root          |
| Password | pdso-training |

The above command will create a new project for **webgoat** if not exist.

---

## Base CI/CD Pipeline

Considering your DevOps team created a simple CI pipeline with the following contents. Please add the **Dependency-Check** scan to this pipeline.

```yaml
image: docker:20.10  # To run all jobs in this pipeline, use the latest docker image

services:
  - docker:dind       # To run all jobs in this pipeline, use a docker image that contains a docker daemon running inside (dind - docker in docker). Reference: https://forum.gitlab.com/t/why-services-docker-dind-is-needed-while-already-having-image-docker/43534

stages:
  - build
  - test
  - release
  - preprod
  - integration
  - prod

build:
  stage: build
  script:
    - echo "This is a build step"

test:
  stage: test
  script:
    - echo "This is a test step"
```

---

## GitLab Access

Let’s log into Gitlab using the following details and run the above pipeline.

| Name       | Value                                                                                                                                                |
| ---------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| GitLab URL | [https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/webgoat](https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/webgoat) |
| Username   | root                                                                                                                                                 |
| Password   | pdso-training                                                                                                                                        |

---

## Create `.gitlab-ci.yml`

Next, we need to create a CI/CD pipeline by adding the `.gitlab-ci.yml` file.

* Click on the **+ (plus)** button, then click **New File**
* Copy the above CI script (use **Control+A** and **Control+V**)

Save changes to the file using the **Commit changes** button.

---

## Verify the Pipeline Runs

As soon as a change is made to the repository, the pipeline starts executing the jobs.

We can see the results of this pipeline by visiting:
👉 [WebGoat GitLab Pipelines](https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/webgoat/pipelines)

Click on the appropriate job name to see the output.

---

Let’s move to the next step.

```
```
````md
# Embed Dependency Check in CI/CD Pipeline

As discussed in the **SCA using Dependency Check** exercise, we can put **Dependency Check** in our CI/CD pipeline.  

However, do remember that you need to run commands **manually in a local system like DevSecOps-Box before embedding** commands for automated execution in CI/CD pipelines.  
Manually testing commands in a local environment like DevSecOps-Box helps in easy troubleshooting.

---

## Step 1: Create `run-depcheck.sh`

Visit 👉 [GitLab WebGoat Project](https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/webgoat) to create a new file called **`run-depcheck.sh`** with the following contents:

```sh
#!/bin/sh

DATA_DIRECTORY="$PWD/data"
REPORT_DIRECTORY="$PWD/reports"

if [ ! -d "$DATA_DIRECTORY" ]; then
  echo "Initially creating persistent directories"
  mkdir -p "$DATA_DIRECTORY"
  chmod -R 777 "$DATA_DIRECTORY"

  mkdir -p "$REPORT_DIRECTORY"
  chmod -R 777 "$REPORT_DIRECTORY"
fi

docker run --rm \
  --volume $(pwd):/src \
  --volume "$DATA_DIRECTORY":/usr/share/dependency-check/data \
  --volume "$REPORT_DIRECTORY":/reports \
  hysnsec/dependency-check \
  --scan /src \
  --format "JSON" \
  --project "Webgoat" \
  --failOnCVSS 4 \
  --out /reports
````

📌 Don’t forget to use **File Name as `run-depcheck.sh`**

---

## Step 2: Add CI/CD Job to `.gitlab-ci.yml`

Now, add the following content to your `.gitlab-ci.yml` file:

```yaml
image: docker:20.10  # To run all jobs in this pipeline, use the latest docker image

services:
  - docker:dind       # To run all jobs in this pipeline, use a docker image that contains a docker daemon running inside (dind - docker in docker). Reference: https://forum.gitlab.com/t/why-services-docker-dind-is-needed-while-already-having-image-docker/43534

stages:
  - build
  - test
  - release
  - preprod
  - integration
  - prod

build:
  stage: build
  script:
    - echo "This is a build step"

test:
  stage: test
  script:
    - echo "This is a test step"

odc-backend:
  stage: test
  image: gitlab/dind:latest
  script:
    - chmod +x ./run-depcheck.sh
    - mkdir -p reports
    - ./run-depcheck.sh > reports/dependency-check-report.json
  artifacts:
    paths:
      - reports/dependency-check-report.json
    when: always
    expire_in: one week

integration:
  stage: integration
  script:
    - echo "This is an integration step"
    - exit 1
  allow_failure: true # Even if the job fails, continue to the next stages

prod:
  stage: prod
  script:
    - echo "This is a deploy step."
  when: manual # Continuous Delivery
```

---

## Step 3: Trigger the Pipeline

As discussed, **any change to the repo kickstarts the pipeline**.

You can see the results of this pipeline by visiting:
👉 [WebGoat Pipelines](https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/webgoat/pipelines)

Click on the appropriate job name to see the output.

---

## Notes

We’ve discussed different methods of using the tool:

* **Native Installation**: Directly installing the tool on the system.
* **Package Manager or Binary**: Installing via a package manager or using the binary file.
* **Docker**: Running the tool within a Docker container.

✅ All methods are suitable for CI/CD integration.
⭐ Docker is recommended for CI/CD as it operates smoothly without dependencies.
⚡ Using the binary file is efficient, avoiding additional dependencies.

Ultimately, you can choose either method based on your specific situation.

---

## Why Did the Job Fail?

Did you notice the job **failed because of non zero exit code**?
Why did the job fail? Because the tool found **security issues**.

👉 This is the expected behavior of any DevSecOps friendly tool.

You need to either:

* **Fix the security issues**, or
* **Add `allow_failure: true`** to let the job finish without blocking other jobs.

If you have troubles understanding exit code, please go through the exercise titled **Working with Exit Code**.

---

In the next step, you will learn **why you should not fail the build (pipeline).**

```
```
````md
# Allow the Job Failure

### Reminder
- Except for **DevSecOps-Box**, every other machine closes after two hours, even if you are in the middle of the exercise.  
- After two hours, in case of a **404**, you need to refresh the exercise page and click on **Start the Exercise** button to continue working.  

---

### Why Not Fail the Pipeline in Maturity Levels 1 and 2?

You do not want to fail the builds (pipeline) in **DevSecOps Maturity Levels 1 and 2** because:  

- If a security tool fails a job, it **won’t allow other DevOps jobs like release/deploy to run**, hence causing great distress to the DevOps Team.  
- Security tools often suffer from **false positives**.  
- Failing a build on inaccurate data is a **sure recipe for disaster**.  

---

### Use `allow_failure`

You can use the `allow_failure` tag to **not fail the build** even though the tool found security issues.  

---

### Example

```yaml
odc-backend:
  stage: test
  image: gitlab/dind:latest
  script:
    - chmod +x ./run-depcheck.sh
    - mkdir -p reports
    - ./run-depcheck.sh > reports/dependency-check-report.json
  artifacts:
    paths:
      - reports/dependency-check-report.json
    when: always
    expire_in: one week
  allow_failure: true #<--- allow the build to fail but don't mark it as such
````

---

### Final Pipeline

```yaml
image: docker:20.10  # To run all jobs in this pipeline, use the latest docker image

services:
  - docker:dind       # To run all jobs in this pipeline, use a docker image that contains a docker daemon running inside (dind - docker in docker). Reference: https://forum.gitlab.com/t/why-services-docker-dind-is-needed-while-already-having-image-docker/43534

stages:
  - build
  - test
  - release
  - preprod
  - integration
  - prod

build:
  stage: build
  script:
    - echo "This is a build step"

test:
  stage: test
  script:
    - echo "This is a test step"

odc-backend:
  stage: test
  image: gitlab/dind:latest
  script:
    - chmod +x ./run-depcheck.sh
    - mkdir -p reports
    - ./run-depcheck.sh > reports/dependency-check-report.json
  artifacts:
    paths:
      - reports/dependency-check-report.json
    when: always
    expire_in: one week
  allow_failure: true

integration:
  stage: integration
  script:
    - echo "This is an integration step"
    - exit 1
  allow_failure: true # Even if the job fails, continue to the next stages

prod:
  stage: prod
  script:
    - echo "This is a deploy step."
  when: manual # Continuous Delivery
```

---

### Outcome

* You will notice that the **odc-backend job has failed**, but other jobs and stages **ran fine**.
* As discussed, **any change to the repo kick starts the pipeline**.

You can see the results of this pipeline by visiting:
👉 [WebGoat Pipelines](https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/webgoat/pipelines)

Click on the appropriate job name to see the **output/results**.

```
```
````md
# Software Component Analysis Using Snyk

## Download the Source Code

We will do all the exercises locally first in **DevSecOps-Box**, so let’s start the exercise.

First, we need to download the source code of the project from our git repository.

```bash
git clone https://gitlab.practical-devsecops.training/pdso/django.nv webapp
````

### Command Output

```
Cloning into 'webapp'...
warning: redirecting to https://gitlab.practical-devsecops.training/pdso/django.nv.git/
remote: Enumerating objects: 228, done.
remote: Total 228 (delta 0), reused 0 (delta 0), pack-reused 228
Receiving objects: 100% (228/228), 1.03 MiB | 1.04 MiB/s, done.
Resolving deltas: 100% (86/86), done.
```

Let’s cd into the application code, so we can scan the app.

```bash
cd webapp
```

We are now in the **webapp** directory.

---

✅ Let’s move to the next step.

```
```
````md
# Install Snyk

Dependency analysis tool or **SCA** is used to find vulnerabilities in open source dependencies and it supports multiple programming languages such as Ruby, Python, NodeJS, Java, Go, .NET. It’s easy to use:  
- A developer can use it on his/her machine using the command line  
- DevOps team can embed it into CI/CD pipeline.  

(Source: https://snyk.io)

📌 More details about the project: https://github.com/snyk/snyk

---

## Step 1: Download Snyk

```bash
wget -O /usr/local/bin/snyk https://github.com/snyk/cli/releases/download/v1.984.0/snyk-linux
````

---

## Step 2: Give executable permission

```bash
chmod +x /usr/local/bin/snyk
```

---

## Step 3: Explore options provided by Snyk

```bash
snyk --help
```

### Command Output

```
CLI commands help
  Snyk CLI scans and monitors your projects for security vulnerabilities and license issues.

  For more information visit the Snyk website https://snyk.io

  For details see the CLI documentation https://docs.snyk.io/features/snyk-cli

How to get started
  1. Authenticate by running snyk auth
  2. Test your local project with snyk test
  3. Get alerted for new vulnerabilities with snyk monitor

Available commands
  To learn more about each Snyk CLI command, use the --help option, for example, snyk auth --help or 
  snyk container --help

  snyk auth
    Authenticate Snyk CLI with a Snyk account.

  snyk test
    Test a project for open source vulnerabilities and license issues.

    Note: Use snyk test --unmanaged to scan all files for known open source dependencies (C/C++ only).

  snyk monitor
    Snapshot and continuously monitor a project for open source vulnerabilities and license issues.

  snyk container
    Test container images for vulnerabilities.

  snyk iac
    Commands to find and manage security issues in Infrastructure as Code files.

  snyk code
    Find security issues using static code analysis.

  snyk log4shell
    Find Log4Shell vulnerability.

  snyk config
    Manage Snyk CLI configuration.

  snyk policy
    Display the .snyk policy for a package.

  snyk ignore
    Modify the .snyk policy to ignore stated issues.

Debug
  Use -d option to output the debug logs.

Configure the Snyk CLI
  You can use environment variables to configure the Snyk CLI and also set variables to configure the
  Snyk CLI to connect with the Snyk API. See Configure the Snyk CLI 
  https://docs.snyk.io/features/snyk-cli/configure-the-snyk-cli
```

---

✅ Let’s move to the next step.

```
```
````md
# Run the Scanner

As we have learned in the **DevSecOps Gospel**, we should save the output in a **machine-readable format**.

We are using `--json` argument to output the results in JSON format.

```bash
snyk test --json .
````

---

### Command Output

```
MissingApiTokenError: `snyk` requires an authenticated account. Please run `snyk auth` and try again.
    at Object.apiTokenExists (/snapshot/snyk/dist/cli/webpack:/snyk/src/lib/api-token.ts:22:11)
    at Object.validateCredentials (/snapshot/snyk/dist/cli/webpack:/snyk/src/cli/commands/test/validate-credentials.ts:10:5)
    at test (/snapshot/snyk/dist/cli/webpack:/snyk/src/cli/commands/test/index.ts:76:3)
    at callModule (/snapshot/snyk/dist/cli/webpack:/snyk/src/cli/commands/index.js:6:1)
    at process.runNextTicks [as _tickCallback] (node:internal/process/task_queues:61:5)
    at Function.runMain (pkg/prelude/bootstrap.js:1941:13)
    at node:internal/main/run_main_module:17:47
    at runCommand (/snapshot/snyk/dist/cli/webpack:/snyk/src/cli/main.ts:51:25)
    at main (/snapshot/snyk/dist/cli/webpack:/snyk/src/cli/main.ts:293:11)
    at /snapshot/snyk/dist/cli/index.ts:11:3
    at Object.callHandlingUnexpectedErrors (/snapshot/snyk/dist/cli/webpack:/snyk/src/lib/unexpected-error.ts:28:5)
```

---

We are greeted with an error.
Snyk is complaining about **missing API Token** as it’s a paid solution and wants you to register before it can run the scan.

---

## Authenticate with Snyk

If you haven’t registered before:

1. Go to [Snyk Website](https://snyk.io)
2. Click **SIGN UP FOR FREE**
3. Select Google account option
4. Complete sign-up
5. Go to account settings page: [https://app.snyk.io/account](https://app.snyk.io/account)
6. Copy the API token

⚠️ **Remember!**
Do not use your company’s Snyk credentials or token to practice these exercises. Please sign up for a **free account** instead.

⚠️ **Note**
Make sure you are copying the correct key, or you will receive:
`Authentication failed. Please check the API token on https://snyk.io.`

---

### Option 1: Authenticate using CLI

```bash
snyk auth YOUR_SNYK_API_TOKEN_HERE
```

#### Command Output

```
Your account has been authenticated. Snyk is now ready to be used.
```

---

### Option 2: Export the token as ENV variable

```bash
export SNYK_TOKEN=YOUR_SNYK_TOKEN_HERE
```

---

## Run the Scan Again

```bash
snyk test --json .
```

---

### Command Output

```
{
  "ok": false,
  "error": "Missing node_modules folder: we can't test without dependencies.\nPlease run 'npm install' first.",
  "path": "."
}
```

---

## Install Dependencies

It raises another error; we need to install the dependencies using `npm install`.

But first, install **npm**:

```bash
mkdir -p /etc/apt/keyrings
curl -fsSL https://deb.nodesource.com/gpgkey/nodesource-repo.gpg.key | gpg --dearmor -o /etc/apt/keyrings/nodesource.gpg
NODE_MAJOR=20
echo "deb [signed-by=/etc/apt/keyrings/nodesource.gpg] https://deb.nodesource.com/node_$NODE_MAJOR.x nodistro main" | tee /etc/apt/sources.list.d/nodesource.list
apt update
apt install nodejs -y
```

Ensure you are in the `webapp` directory:

```bash
cd /webapp
```

Now run:

```bash
npm install
```

---

## Re-run the Scan

```bash
snyk test --json . > output.json
```

---

### Check the Output

```bash
cat output.json
```

#### Command Output

```
{
  "ok": false,
  "error": "Dependency bcrypt-nodejs was not found in package-lock.json. Your package.json and package-lock.json are probably out of sync. Please run \"npm install\" and try again.",
  "path": "."
}
```

---

We are experiencing another error while scanning the dependencies.

👉 After exploration, the issue is related to out-of-sync **package.json** and **package-lock.json**.

---

## Fix: Ignore Out-of-Sync Errors

```bash
snyk test --strict-out-of-sync=false --json . > output.json
```

---

📌 **Note**
If you want to get the dashboard on the Snyk website, then monitor for new vulnerabilities, please use:

```bash
snyk monitor
```

```
```
````md
# How To Embed Snyk Into GitLab

## A Simple CI/CD Pipeline  
Considering your DevOps team created a simple CI pipeline with the following contents.  

Please add the Snyk scan to this pipeline.  

---

**Click anywhere to copy**

```yaml
image: docker:20.10  # To run all jobs in this pipeline, use the latest docker image

services:
  - docker:dind       # To run all jobs in this pipeline, use a docker image that contains a docker daemon running inside (dind - docker in docker). Reference: https://forum.gitlab.com/t/why-services-docker-dind-is-needed-while-already-having-image-docker/43534

stages:
  - build
  - test
  - release
  - preprod
  - integration
  - prod

build:
  stage: build
  image: python:3.6
  before_script:
   - pip3 install --upgrade virtualenv
  script:
   - virtualenv env                       # Create a virtual environment for the python application
   - source env/bin/activate              # Activate the virtual environment
   - pip install -r requirements.txt      # Install the required third party packages as defined in requirements.txt
   - python manage.py check               # Run checks to ensure the application is working fine

test:
  stage: test
  image: python:3.6
  before_script:
   - pip3 install --upgrade virtualenv
  script:
   - virtualenv env
   - source env/bin/activate
   - pip install -r requirements.txt
   - python manage.py test taskManager
````

---

## Let’s log into GitLab using the following details and run the above pipeline.

**GitLab CI/CD Machine**

| Name     | Value                                                                                                                                                                                                          |
| -------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Link     | [https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/django-nv/-/blob/main/.gitlab-ci.yml](https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/django-nv/-/blob/main/.gitlab-ci.yml) |
| Username | root                                                                                                                                                                                                           |
| Password | pdso-training                                                                                                                                                                                                  |

---

Next, we need to create a CI/CD pipeline by replacing the `.gitlab-ci.yml` file content with the above CI script.
Click on the **Edit** button to replace the content (use Control+A and Control+V).

Save changes to the file using the **Commit changes** button.

---

## Verify the pipeline runs

As soon as a change is made to the repository, the pipeline starts executing the jobs.

We can see the results of this pipeline by visiting:
👉 [https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/django-nv/pipelines](https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/django-nv/pipelines)

Click on the appropriate job name to see the output.

---

## Challenge

We will use a commercial offering **Snyk** to scan for vulnerable third-party components.

* Add another job name **oast-snyk** under the **build stage** with the Snyk tool
* Sign up for the Snyk’s Free service and generate/copy the token
* Store Snyk token in **Variables** via:
  Project → Settings → CI/CD → Variables
  [https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/django-nv/-/settings/ci\_cd](https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/django-nv/-/settings/ci_cd)
* Create a new variable **SNYK\_TOKEN** and put the token you got from the service, and ensure the **protected** flag is turned on
* Use Snyk’s Linux binary to scan the dependencies in the project

---

⚠️ Please try to do this exercise **without looking at the solution** on the next page.

---

Let’s move to the next step.

```
```
````md
# Embed Snyk in CI/CD pipeline

As discussed in the **SCA using the Snyk exercise**, we can put Snyk in our CI/CD pipeline.  

However, do remember that you need run commands manually in a local system like **DevSecOps Box** before embedding commands for automated execution in CI/CD pipelines.  

Manually testing commands in a local environment like DevSecOps Box helps in easy troubleshooting.  

Since **Snyk** is a paid tool, you would also need to add the **SNYK_TOKEN** to the CI/CD variables on GitLab CI.  

⚠️ **Do not use your company’s Snyk Credentials/Token** to practice these exercises. Please sign up for a **Free Account** instead.  

---

## Configure the Snyk Token

Let’s login into GitLab and configure this token:  
👉 [GitLab CI/CD Settings](https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/django-nv/-/settings/ci_cd)

Click on the **Expand** button under the Variables section, then click on the **Add Variable** button.  

Add the following key/value pair in the form:  

| Name | Value |
|------|-------|
| Key  | `SNYK_TOKEN` |
| Value | `INSERT_SNYK_TOKEN_HERE`, you can find the token in your Snyk account at [https://app.snyk.io/account](https://app.snyk.io/account) |

Finally, click on the **Add Variable** button.  

---

## Update the Pipeline

Next, please visit:  
👉 [Edit .gitlab-ci.yml](https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/django-nv/-/blob/main/.gitlab-ci.yml)  

Click on the **Edit** button and append the following code to the `.gitlab-ci.yml` file.  

---

**Click anywhere to copy**

```yaml
image: docker:20.10  # To run all jobs in this pipeline, use the latest docker image

services:
  - docker:dind       # To run all jobs in this pipeline, use a docker image that contains a docker daemon running inside (dind - docker in docker). Reference: https://forum.gitlab.com/t/why-services-docker-dind-is-needed-while-already-having-image-docker/43534

stages:
  - build
  - test
  - release
  - preprod
  - integration
  - prod

build:
  stage: build
  image: python:3.6
  before_script:
   - pip3 install --upgrade virtualenv
  script:
   - virtualenv env                       # Create a virtual environment for the python application
   - source env/bin/activate              # Activate the virtual environment
   - pip install -r requirements.txt      # Install the required third party packages as defined in requirements.txt
   - python manage.py check               # Run checks to ensure the application is working fine

test:
  stage: test
  image: python:3.6
  before_script:
   - pip3 install --upgrade virtualenv
  script:
   - virtualenv env
   - source env/bin/activate
   - pip install -r requirements.txt
   - python manage.py test taskManager

oast-snyk:
  stage: build
  image: node:alpine3.10
  before_script:
    - wget -O snyk https://github.com/snyk/cli/releases/download/v1.1156.0/snyk-alpine
    - chmod +x snyk
    - mv snyk /usr/local/bin/
  script:
    - npm install
    - snyk auth $SNYK_TOKEN
    - snyk test --json > snyk-results.json
    - cat snyk-results.json
  artifacts:
    paths:
      - snyk-results.json
    when: always
    expire_in: one week

integration:
  stage: integration
  script:
    - echo "This is an integration step"
    - exit 1
  allow_failure: true # Even if the job fails, continue to the next stages

prod:
  stage: prod
  script:
    - echo "This is a deploy step."
  when: manual # Continuous Delivery
````

---

## Verify the Pipeline

As discussed, **any change to the repo kickstarts the pipeline.**

We can see the results of this pipeline by visiting:
👉 [Pipeline Results](https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/django-nv/pipelines)

Click on the appropriate job name to see the output.

---

## Next Step

In the next step, you will learn **why you need to fail the builds**.

```
```
````md
# Allow the job failure

**Remember!**

Except for **DevSecOps-Box**, every other machine closes after two hours, even if you are in the middle of the exercise.  
You need to refresh the exercise page and click on **Start the Exercise** button to continue working on it.  

You do not want to fail the builds in **DevSecOps Maturity Levels 1 and 2**.  
If a security tool fails a job upon finding security issues, you would want to allow it to fail and not block the builds as there would be **false positives** in the results.  

You can use the **allow_failure** tag to *“not fail the build”* even though the tool found security issues.  

```yaml
oast-snyk:
  stage: build
  image: node:alpine3.10
  before_script:
    - wget -O snyk https://github.com/snyk/cli/releases/download/v1.1156.0/snyk-alpine
    - chmod +x snyk
    - mv snyk /usr/local/bin/
  script:
    - npm install
    - snyk auth $SNYK_TOKEN
    - snyk test --json > snyk-results.json
    - cat snyk-results.json
  artifacts:
    paths:
      - snyk-results.json
    when: always
  allow_failure: true #<--- allow the build to fail but don't mark it as such
````

---

## Final Pipeline

```yaml
image: docker:20.10  # To run all jobs in this pipeline, use the latest docker image

services:
  - docker:dind       # To run all jobs in this pipeline, use a docker image that contains a docker daemon running inside (dind - docker in docker). Reference: https://forum.gitlab.com/t/why-services-docker-dind-is-needed-while-already-having-image-docker/43534

stages:
  - build
  - test
  - release
  - preprod
  - integration
  - prod

build:
  stage: build
  image: python:3.6
  before_script:
   - pip3 install --upgrade virtualenv
  script:
   - virtualenv env                       # Create a virtual environment for the python application
   - source env/bin/activate              # Activate the virtual environment
   - pip install -r requirements.txt      # Install the required third party packages as defined in requirements.txt
   - python manage.py check               # Run checks to ensure the application is working fine

test:
  stage: test
  image: python:3.6
  before_script:
   - pip3 install --upgrade virtualenv
  script:
   - virtualenv env
   - source env/bin/activate
   - pip install -r requirements.txt
   - python manage.py test taskManager

oast-snyk:
  stage: build
  image: node:alpine3.10
  before_script:
    - wget -O snyk https://github.com/snyk/cli/releases/download/v1.1156.0/snyk-alpine
    - chmod +x snyk
    - mv snyk /usr/local/bin/
  script:
    - npm install
    - snyk auth $SNYK_TOKEN
    - snyk test --json > snyk-results.json
    - cat snyk-results.json
  artifacts:
    paths:
      - snyk-results.json
    when: always
    expire_in: one week
  allow_failure: true

integration:
  stage: integration
  script:
    - echo "This is an integration step"
    - exit 1
  allow_failure: true # Even if the job fails, continue to the next stages

prod:
  stage: prod
  script:
    - echo "This is a deploy step."
  when: manual # Continuous Delivery
```

---

## Key Notes

* You will notice that the **oast-snyk** job has failed, but other jobs and stages ran fine.
* As discussed, any change to the repo **kick starts the pipeline**.
* We can see the results of this pipeline by visiting:
  👉 [GitLab Pipeline Results](https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/django-nv/pipelines)

Click on the appropriate job name to see the output/results.

---

## Remember!

* Did you notice the job failed because of **non zero exit code** (here, exit code 1)?
* **Why did the job fail?** Because the tool found security issues.

  * This is the **expected behavior** of any DevSecOps friendly tool.
* You need to either **fix the security issues** or add `allow_failure: true` to let the job finish without blocking other jobs.

👉 If you have troubles understanding **exit code**, please go through the exercise titled **Working with Exit Code**.

---

✅ Let’s move to the next step.

```
```
```md
# Extra Mile Exercise: Analyze the results from Snyk and remove any false positives

## Who should do this exercise?
This exercise is beyond the CDP course’s scope. It’s added to help folks who already know these concepts in and out.
You can write small scripts in Python or Ruby or Golang.
You consider yourself an expert or advanced user in SAST.

## Challenges
Recall techniques you have learned in the previous exercises.

- Login into the Snyk portal and explore the findings
- Mark false positives as false positives
- Ensure each result is at the right severity level (HIGH, MEDIUM, LOW)
- Export the final report and suggest remediation steps to developers
- Ensure false positives are not shown again in CI scans

**Note**

We will not provide solutions for this extra mile exercise.
```
```markdown
Software Component Analysis Using NPM Audit
Download the source code
We will do all the exercises locally first in DevSecOps-Box, so let’s start the exercise.

First, We need to download the source code of the project from our git repository.


git clone https://gitlab.practical-devsecops.training/pdso/django.nv webapp

Command Output
Cloning into 'webapp'...
warning: redirecting to https://gitlab.practical-devsecops.training/pdso/django.nv.git/
remote: Enumerating objects: 299, done.
remote: Counting objects: 100% (71/71), done.
remote: Compressing objects: 100% (68/68), done.
remote: Total 299 (delta 34), reused 0 (delta 0), pack-reused 228
Receiving objects: 100% (299/299), 1.05 MiB | 29.16 MiB/s, done.
Resolving deltas: 100% (120/120), done.
Let’s cd into the application code so we can scan the app.


cd webapp

We are now in the webapp directory.

Let’s move to the next step.
```
```markdown
Install NPM
NPM has security built in since npm@6, this command allows you to analyze dependencies in your existing code to identify insecure dependencies, have a good security report, and also easy to implement it into CI/CD pipeline without install any packages except NPM itself.

We will install NodeJS and also NPM with the following command.


curl -sL https://deb.nodesource.com/setup_14.x | bash -


apt install nodejs -y

npm audit tool is part of npm and it gets installed when you install node.js. Let’s check the help menu of npm audit to understand its feature set.


npm audit -h

Command Output
npm audit [--json] [--production]
npm audit fix [--force|--package-lock-only|--dry-run|--production|--only=(dev|prod)]
Which options would you choose from this list?

Let’s move to the next step.
```
````markdown
Run the Scanner
We can run the basic command without any extra arguments, however the output is not suitable for a CI/CD pipeline because the output is not in a machine readable format.

First, we need to install the dependencies using npm install.


npm install

Let’s run the npm audit command.


npm audit

Command Output

...[SNIP]...

┌───────────────┬──────────────────────────────────────────────────────────────┐
│ High          │ ini before 1.3.6 vulnerable to Prototype Pollution via       │
│               │ ini.parse                                                    │
├───────────────┼──────────────────────────────────────────────────────────────┤
│ Package       │ ini                                                          │
├───────────────┼──────────────────────────────────────────────────────────────┤
│ Patched in    │ >=1.3.6                                                      │
├───────────────┼──────────────────────────────────────────────────────────────┤
│ Dependency of │ grunt-npm-install [dev]                                      │
├───────────────┼──────────────────────────────────────────────────────────────┤
│ Path          │ grunt-npm-install > npm > config-chain > ini                 │
├───────────────┼──────────────────────────────────────────────────────────────┤
│ More info     │ https://github.com/advisories/GHSA-qqgx-2p2h-9c37            │
└───────────────┴──────────────────────────────────────────────────────────────┘
found 250 vulnerabilities (22 low, 79 moderate, 119 high, 30 critical) in 1347 scanned packages
  38 vulnerabilities require semver-major dependency updates.
  212 vulnerabilities require manual review. See the full report for details.
As we can see, the basic command gives us the report in a table format

As we have learned in the DevSecOps Gospel, we should save the output in a machine-readable format so the output can be parsed by the machines.

We can save the output in JSON format using --json flag.


npm audit --json | tee results.json

Command Output

...[SNIP]...

    "1091252": {
      "findings": [
        {
          "version": "1.3.4",
          "paths": [
            "grunt-npm-install>npm>ini",
            "grunt-npm-install>npm>config-chain>ini"
          ]
        }
      ],
      "metadata": null,
      "vulnerable_versions": "<1.3.6",
      "module_name": "ini",
      "severity": "high",
      "github_advisory_id": "GHSA-qqgx-2p2h-9c37",
      "cves": [
        "CVE-2020-7788"
      ],
      "access": "public",
      "patched_versions": ">=1.3.6",
      "cvss": {
        "score": 7.3,
        "vectorString": "CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:L/I:L/A:L"
      },
      "updated": "2023-03-02T21:03:59.000Z",
      "recommendation": "Upgrade to version 1.3.6 or later",
      "cwe": [
        "CWE-1321"
      ],
      "found_by": null,
      "deleted": null,
      "id": 1091252,
      "references": "- https://github.com/npm/ini/commit/56d2805e07ccd94e2ba0984ac9240ff02d44b6f1\n- https://www.npmjs.com/advisories/1589\n- https://snyk.io/vuln/SNYK-JS-INI-1048974\n- https://nvd.nist.gov/vuln/detail/CVE-2020-7788\n- https://lists.debian.org/debian-lts-announce/2020/12/msg00032.html\n- https://github.com/advisories/GHSA-qqgx-2p2h-9c37",
      "created": "2020-12-10T16:53:45.000Z",
      "reported_by": null,
      "title": "ini before 1.3.6 vulnerable to Prototype Pollution via ini.parse",
      "npm_advisory_id": null,
      "overview": "### Overview\nThe `ini` npm package before version 1.3.6 has a Prototype Pollution vulnerability.\n\nIf an attacker submits a malicious INI file to an application that parses it with `ini.parse`, they will pollute the prototype on the application. This can be exploited further depending on the context.\n\n### Patches\n\nThis has been patched in 1.3.6.\n\n### Steps to reproduce\n\npayload.ini\n```\n[__proto__]\npolluted = \"polluted\"\n```\n\npoc.js:\n```\nvar fs = require('fs')\nvar ini = require('ini')\n\nvar parsed = ini.parse(fs.readFileSync('./payload.ini', 'utf-8'))\nconsole.log(parsed)\nconsole.log(parsed.__proto__)\nconsole.log(polluted)\n```\n\n```\n> node poc.js\n{}\n{ polluted: 'polluted' }\n{ polluted: 'polluted' }\npolluted\n```",
      "url": "https://github.com/advisories/GHSA-qqgx-2p2h-9c37"
    }
  },
  "muted": [],
  "metadata": {
    "vulnerabilities": {
      "info": 0,
      "low": 22,
      "moderate": 79,
      "high": 119,
      "critical": 30
    },
    "dependencies": 409,
    "devDependencies": 934,
    "optionalDependencies": 57,
    "totalDependencies": 1347
  },
  "runId": "096c6f2f-8d40-4af6-8796-da4e94c0bc43"
}
There you go. Why don’t you embed npm audit in CI/CD pipeline?
````
```markdown
Software Component Analysis Using AuditJS
Download the source code
Like always, we will do all the exercises locally first in DevSecOps-Box, so let’s start the exercise.

First, We need to download the source code of the project from our git repository.


git clone https://gitlab.practical-devsecops.training/pdso/django.nv webapp

Command Output
Cloning into 'webapp'...
warning: redirecting to https://gitlab.practical-devsecops.training/pdso/django.nv.git/
remote: Enumerating objects: 299, done.
remote: Counting objects: 100% (71/71), done.
remote: Compressing objects: 100% (68/68), done.
remote: Total 299 (delta 34), reused 0 (delta 0), pack-reused 228
Receiving objects: 100% (299/299), 1.05 MiB | 29.16 MiB/s, done.
Resolving deltas: 100% (120/120), done.
Let’s cd into the application code so we can scan the app.


cd webapp

We are now in the webapp directory.

Let’s move to the next step.
```
```markdown
Install AuditJS
AuditJS Audits an NPM package.json file to identify known vulnerabilities. It uses the OSS Index v3 REST API from Sonatype to identify known vulnerabilities and outdated package versions.

Since AuditJS needs npm, we will install NodeJS and NPM with the following command.


mkdir -p /etc/apt/keyrings
curl -fsSL https://deb.nodesource.com/gpgkey/nodesource-repo.gpg.key | gpg --dearmor -o /etc/apt/keyrings/nodesource.gpg
NODE_MAJOR=20
echo "deb [signed-by=/etc/apt/keyrings/nodesource.gpg] https://deb.nodesource.com/node_$NODE_MAJOR.x nodistro main" | tee /etc/apt/sources.list.d/nodesource.list
apt update


apt install nodejs -y

Let’s install auditjs with NPM.


npm install -g auditjs

Perfect, we have installed the tool. Let’s check the help menu to understand its feature set.


auditjs -h

Command Output
auditjs [command]

Commands:
  auditjs iq [options]    Audit this application using Nexus IQ Server
  auditjs config          Set config for OSS Index or Nexus IQ Server
  auditjs ossi [options]  Audit this application using Sonatype OSS Index

Options:
  --version  Show version number                                       [boolean]
  --help     Show help                                                 [boolean]
auditjs provides you with the ability to use two different indices i.e., either Nexus IQ Server or Sonatype OSS Index. This tool will automatically update the index based on your input and will poll the index to find if we are using any vulnerable components.

Let’s move to the next step.
```
```markdown
Run the Scanner
We will use Sonatype as an index to scan our project’s components. If you explore the help menu, you will see we can use ossi as index.


auditjs ossi --help

Command Output
auditjs ossi [options]

Audit this application using Sonatype OSS Index

Commands:
  auditjs ossi sbom  Output the purl only CycloneDx sbom to std_out

Options:
      --version    Show version number                                 [boolean]
      --help       Show help                                           [boolean]
  -u, --user       Specify OSS Index username                           [string]
  -p, --password   Specify OSS Index password or token                  [string]
  -c, --cache      Specify path to use as a cache location              [string]
  -q, --quiet      Only print out vulnerable dependencies              [boolean]
  -j, --json       Set output to JSON                                  [boolean]
  -x, --xml        Set output to JUnit XML format                      [boolean]
  -w, --whitelist  Set path to whitelist file                           [string]
      --clear      Clears cache location if it has been set in config  [boolean]
      --bower      Force the application to explicitly scan for Bower  [boolean]
Let’s perform the scan using the following command.


auditjs ossi

Command Output
 ________   ___  ___   ________   ___   _________       ___   ________
|\   __  \ |\  \|\  \ |\   ___ \ |\  \ |\___   ___\    |\  \ |\   ____\
\ \  \|\  \\ \  \\\  \\ \  \_|\ \\ \  \\|___ \  \_|    \ \  \\ \  \___|_
 \ \   __  \\ \  \\\  \\ \  \ \\ \\ \  \    \ \  \   __ \ \  \\ \_____  \
  \ \  \ \  \\ \  \\\  \\ \  \_\\ \\ \  \    \ \  \ |\  \\_\  \\|____|\  \
   \ \__\ \__\\ \_______\\ \_______\\ \__\    \ \__\\ \________\ ____\_\  \
    \|__|\|__| \|_______| \|_______| \|__|     \|__| \|________||\_________\
                                                                \|_________|


  _      _                       _   _
 /_)    /_`_  _  _ _/_   _  _   (/  /_`_._  _   _/ _
/_)/_/ ._//_// //_|/ /_//_//_' (_X /  ///_'/ //_/_\
   _/                _//

  AuditJS version: 4.0.39

✔ Starting application
[2023-03-05T05:58:10.320] [ERROR] auditjs - Failed project directory validation.  Are you in a (built) node, yarn, or bower project directory?
Error: Could not instantiate muncher
    at new Application (/usr/lib/node_modules/auditjs/bin/Application/Application.js:68:19)
    at Object.<anonymous> (/usr/lib/node_modules/auditjs/bin/index.js:206:23)
    at Module._compile (internal/modules/cjs/loader.js:1114:14)
    at Object.Module._extensions..js (internal/modules/cjs/loader.js:1143:10)
    at Module.load (internal/modules/cjs/loader.js:979:32)
    at Function.Module._load (internal/modules/cjs/loader.js:819:12)
    at Function.executeUserEntryPoint [as runMain] (internal/modules/run_main.js:75:12)
    at internal/main/run_main_module.js:17:47
As you can see, it’s complaining that we haven’t installed these dependencies locally, so lets install our components using npm install.


npm install

Are you wondering how we can figure these commands? It’s simple, you either read the project’s documentation or you run some trials and figure out what to do.

Now that’s done, re-run the auditjs command once again.


auditjs ossi

Command Output
  ________   ___  ___   ________   ___   _________       ___   ________      
|\   __  \ |\  \|\  \ |\   ___ \ |\  \ |\___   ___\    |\  \ |\   ____\     
\ \  \|\  \\ \  \\\  \\ \  \_|\ \\ \  \\|___ \  \_|    \ \  \\ \  \___|_    
 \ \   __  \\ \  \\\  \\ \  \ \\ \\ \  \    \ \  \   __ \ \  \\ \_____  \   
  \ \  \ \  \\ \  \\\  \\ \  \_\\ \\ \  \    \ \  \ |\  \\_\  \\|____|\  \  
   \ \__\ \__\\ \_______\\ \_______\\ \__\    \ \__\\ \________\ ____\_\  \ 
    \|__|\|__| \|_______| \|_______| \|__|     \|__| \|________||\_________\
                                                                \|_________|


  _      _                       _   _              
 /_)    /_`_  _  _ _/_   _  _   (/  /_`_._  _   _/ _
/_)/_/ ._//_// //_|/ /_//_//_' (_X /  ///_'/ //_/_\ 
   _/                _//                            

  AuditJS version: 4.0.39

✔ Starting application
✔ Getting coordinates for Sonatype OSS Index
✔ Auditing your application with Sonatype OSS Index
✔ Submitting coordinates to Sonatype OSS Index
✔ Reticulating splines
✔ Removing whitelisted vulnerabilities

  Sonabot here, beep boop beep boop, here are your Sonatype OSS Index results:
  Total dependencies audited: 350

...[SNIP]...

[335/350] - pkg:npm/utile@0.2.1 - 1 vulnerability found!

  Vulnerability Title:  1 vulnerability found
  ID:  sonatype-2018-0208
  Description:  1 non-CVE vulnerability found. To see more details, please create a free account at https://ossindex.sonatype.org/ and request for this information using your registered account
  CVSS Score:  9.1
  CVSS Vector:  CVSS:3.0/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:N/A:H
  Reference:  https://ossindex.sonatype.org/vulnerability/sonatype-2018-0208

[336/350] - pkg:npm/utile@0.3.0 - 1 vulnerability found!

  Vulnerability Title:  1 vulnerability found
  ID:  sonatype-2018-0208
  Description:  1 non-CVE vulnerability found. To see more details, please create a free account at https://ossindex.sonatype.org/ and request for this information using your registered account
  CVSS Score:  9.1
  CVSS Vector:  CVSS:3.0/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:N/A:H
  Reference:  https://ossindex.sonatype.org/vulnerability/sonatype-2018-0208

[337/350] - pkg:npm/utils-merge@1.0.0 - No vulnerabilities found!
[338/350] - pkg:npm/utils-merge@1.0.1 - No vulnerabilities found!
[339/350] - pkg:npm/vary@1.1.2 - No vulnerabilities found!
[340/350] - pkg:npm/which-boxed-primitive@1.0.2 - No vulnerabilities found!
[341/350] - pkg:npm/which-collection@1.0.1 - No vulnerabilities found!
[342/350] - pkg:npm/which-typed-array@1.1.9 - No vulnerabilities found!
[343/350] - pkg:npm/window-size@0.1.0 - No vulnerabilities found!
[344/350] - pkg:npm/winston@0.8.0 - No vulnerabilities found!
[345/350] - pkg:npm/winston@0.8.3 - No vulnerabilities found!
[346/350] - pkg:npm/wordwrap@0.0.2 - No vulnerabilities found!
[347/350] - pkg:npm/wordwrap@0.0.3 - No vulnerabilities found!
[348/350] - pkg:npm/wrappy@1.0.2 - No vulnerabilities found!
[349/350] - pkg:npm/x-xss-protection@1.0.0 - No vulnerabilities found!
[350/350] - pkg:npm/yargs@3.5.4 - No vulnerabilities found!
--------------------------------------------------------------------------
As we can see, the basic command gives us the report in a table format.

As we have learned in the DevSecOps Gospel, we should save the output in a machine-readable format so the output can be parsed by the machines.

We can save the output in JSON format using -j flag and make it print the vulnerable dependencies only using -q flag.


auditjs ossi -q -j | tee auditjs-output.json

Command Output
[

...[SNIP]...

  {
    "coordinates": "pkg:npm/uglify-js@2.4.24",
    "description": "JavaScript parser, mangler/compressor and beautifier toolkit",
    "reference": "https://ossindex.sonatype.org/component/pkg:npm/uglify-js@2.4.24?utm_source=auditjs&utm_medium=integration&utm_content=4.0.39",
    "vulnerabilities": [
      {
        "id": "sonatype-2015-0010",
        "title": "[sonatype-2015-0010] CWE-400: Uncontrolled Resource Consumption ('Resource Exhaustion')",
        "description": "uglify-js - [ CVE-2015-8858 ] Regular Expression Denial of Service \n\nThe software does not properly restrict the size or amount of resources that are requested or influenced by an actor, which can be used to consume more resources than intended.",
        "cvssScore": 7.5,
        "cvssVector": "CVSS:3.0/AV:N/AC:L/PR:N/UI:N/S:U/C:N/I:N/A:H",
        "cve": "CVE-2015-8858",
        "reference": "https://ossindex.sonatype.org/vulnerability/sonatype-2015-0010?component-type=npm&component-name=uglify-js&utm_source=auditjs&utm_medium=integration&utm_content=4.0.39"
      }
    ]
  }
]
Great! Why don’t you embed auditjs in CI/CD pipeline?
```
```markdown
How To Embed AuditJS Into GitLab
A Simple CI/CD Pipeline
Let’s download the code using git clone in DevSecOps Box.


git clone https://gitlab.practical-devsecops.training/pdso/nodejs.git nodejs


cd nodejs

Rename git url to the new one.


git remote rename origin old-origin


git remote add origin git@gitlab-ce-kr6k1mdm:root/nodejs.git

Then, push the code into the repository.


git push -u origin --all

And enter the GitLab credentials.

Name	Value
Username	root
Password	pdso-training
The above command will create a new project for nodejs if not exist.

Next, considering your DevOps team created a simple CI pipeline with the following contents.

Click anywhere to copy

image: node:alpine3.10

stages:
  - build
  - test
  - release
  - preprod
  - integration
  - prod

test:
  stage: test
  script:
    - echo "This is an test step"

integration:
  stage: integration
  script:
    - echo "This is an integration step"

prod:
  stage: prod
  script:
    - echo "This is a deploy step"

Let’s log into GitLab using the following details and run the above pipeline.

GitLab CI/CD Machine
Name	Value
URL	https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/nodejs/-/blob/main/.gitlab-ci.yml
Username	root
Password	pdso-training
Next, we need to create a CI/CD pipeline by replacing the .gitlab-ci.yml file content with the above CI script. Click on the Edit button to replace the content (use Control+A and Control+V).

Save changes to the file using the Commit changes button.

Verify the pipeline runs
As soon as a change is made to the repository, the pipeline starts executing the jobs.

We can see the results of this pipeline by visiting https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/nodejs/pipelines.

Click on the appropriate job name to see the output.

Let’s move to the next step.
```
```markdown
Embed AuditJS in CI/CD pipeline
As discussed in the SCA using AuditJS exercise, we can embed the AuditJS tool in our CI/CD pipeline.

However, do remember that you need run commands manually in a local system like devsecops box before embedding commands for automated execution in CI/CD pipelines.

Manually testing commands in a local environment like devsecops box helps in easy troubleshooting.

Do you wonder which stage this job should go into?

Maybe you want to scan the components before performing SAST scans.

Click anywhere to copy

image: node:alpine3.10

cache:
  paths:
  - node_modules/

stages:
  - build
  - test
  - release
  - preprod
  - integration
  - prod

build:
  stage: build
  script:
    - npm install

test:
  stage: test
  script:
    - echo "This is an test step"

auditjs:
  image: docker:dind
  stage: test
  script:
    - docker run --rm -v $(pwd):/src -w /src hysnsec/auditjs ossi -q -j | tee auditjs-output.json
  artifacts:
    paths: [auditjs-output.json]
    when: always # What is this for?
    expire_in: one week

integration:
  stage: integration
  script:
    - echo "This is an integration step"

prod:
  stage: prod
  script:
    - echo "This is a deploy step"

Copy the above CI script and add it to the .gitlab.ci.yml file on Gitlab repo at https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/nodejs/-/blob/main/.gitlab-ci.yml.

Do not forget to click on the “Commit Changes” button to save the file.

As discussed, any change to the repo kick starts the pipeline.

We can see the results of this pipeline by visiting https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/nodejs/pipelines.

Click on the appropriate job name to see the output.

Remember!

Did you notice the job failed because of non zero exit code?

Why did the job fail? Because the tool found security issues. This is the expected behavior of any DevSecOps friendly tool.

You need to either fix the security issues or add allow_failure: true to let the job finish without blocking other jobs.

If you have troubles understanding exit code, please go through the exercise titled Working with Exit Code.

You will notice that the auditjs job stores the output to a file auditjs-output.json.

In the next step, you will learn the need to not fail the builds (pipeline).
```
```markdown
Allow the job failure
Remember!

Except for DevSecOps-Box, every other machine closes after two hours, even if you are in the middle of the exercise
After two hours, in case of a 404, you need to refresh the exercise page and click on Start the Exercise button to continue working
You do not want to fail the builds (pipeline) in DevSecOps Maturity Levels 1 and 2. If a security tool fails a build upon security findings, you would want to allow it to fail and not block the pipeline as there would be false positives in the results.

You can use the allow_failure tag to not fail the builds even though the tool found security issues.

auditjs:
  image: docker:dind
  stage: test
  script:
    - docker run --rm -v $(pwd):/src -w /src hysnsec/auditjs ossi -q -j | tee auditjs-output.json
  artifacts:
    paths: [auditjs-output.json]
    when: always # What is this for?
    expire_in: one week
  allow_failure: true  #<--- allow the build to fail but don't mark it as such
Notice allow_failure: true at the end of the YAML file.

The final pipeline would look like the following.

Click anywhere to copy

image: node:alpine3.10

cache:
  paths:
  - node_modules/

stages:
  - build
  - test
  - release
  - preprod
  - integration
  - prod

build:
  stage: build
  script:
    - npm install

test:
  stage: test
  script:
    - echo "This is an test step"

auditjs:
  image: docker:dind
  stage: test
  script:
    - docker run --rm -v $(pwd):/src -w /src hysnsec/auditjs ossi -q -j | tee auditjs-output.json
  artifacts:
    paths: [auditjs-output.json]
    when: always # What is this for?
    expire_in: one week
  allow_failure: true  #<--- allow the build to fail but don't mark it as such

integration:
  stage: integration
  script:
    - echo "This is an integration step"

prod:
  stage: prod
  script:
    - echo "This is a deploy step"

Go ahead and add it to the .gitlab-ci.yml file to run the pipeline.

You will notice that the auditjs job has failed but didn’t block the other jobs from running.

As discussed, any change to the repo kick starts the pipeline.

Note

We’ve discussed different methods of using the tool:

Native Installation:
Directly installing the tool on the system.
Package Manager or Binary:
Installing via a package manager or using the binary file.
Docker:
Running the tool within a Docker container.
In summary:

All methods are suitable for CI/CD integration.
Docker is recommended for CI/CD as it operates smoothly without dependencies.
Using the binary file is efficient, avoiding additional dependencies.
Ultimately, you can choose either method based on your specific situation.

We can see the results of this pipeline by visiting https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/nodejs/pipelines.

Click on the appropriate job name to see the output.
```
```markdown
Software Component Analysis Using bundler-audit
Download the source code
We will do all the exercises locally first in DevSecOps-Box, so let’s start the exercise.

First, We need to download the source code of the project from our git repository.


git clone https://gitlab.practical-devsecops.training/pdso/rails.git webapp

Let’s cd into the application so we can scan the app.


cd webapp

We are now in the webapp directory.

Let’s move to the next step.
```
```markdown
Install bundler-audit
Basically, our system doesn’t have Ruby installed, so we need to install it using rbenv utility so lets install rbenv.


curl -fsSL https://github.com/rbenv/rbenv-installer/raw/HEAD/bin/rbenv-installer | bash

Command Output
Installing rbenv with git...
Initialized empty Git repository in /root/.rbenv/.git/
Updating origin
remote: Enumerating objects: 2889, done.
remote: Counting objects: 100% (46/46), done.
remote: Compressing objects: 100% (37/37), done.
remote: Total 2889 (delta 20), reused 19 (delta 9), pack-reused 2843
Receiving objects: 100% (2889/2889), 558.75 KiB | 18.63 MiB/s, done.
Resolving deltas: 100% (1808/1808), done.
From https://github.com/rbenv/rbenv
 * [new branch]      master     -> origin/master
 * [new tag]         v0.1.0     -> v0.1.0
 * [new tag]         v0.1.1     -> v0.1.1
 * [new tag]         v0.1.2     -> v0.1.2
 * [new tag]         v0.2.0     -> v0.2.0
 * [new tag]         v0.2.1     -> v0.2.1
 * [new tag]         v0.3.0     -> v0.3.0
 * [new tag]         v0.4.0     -> v0.4.0
 * [new tag]         v1.0.0     -> v1.0.0
 * [new tag]         v1.1.0     -> v1.1.0
 * [new tag]         v1.1.1     -> v1.1.1
 * [new tag]         v1.1.2     -> v1.1.2
Branch 'master' set up to track remote branch 'master' from 'origin'.
Already on 'master'
warning: gcc not found; using CC=cc
aborted: compiler not found: cc
Optional bash extension failed to build, but things will still work normally.

Installing ruby-build with git...
Cloning into '/root/.rbenv/plugins/ruby-build'...
remote: Enumerating objects: 11574, done.
remote: Counting objects: 100% (267/267), done.
remote: Compressing objects: 100% (138/138), done.
remote: Total 11574 (delta 197), reused 172 (delta 116), pack-reused 11307
Receiving objects: 100% (11574/11574), 2.44 MiB | 22.53 MiB/s, done.
Resolving deltas: 100% (7664/7664), done.

All done!
Note that this installer does NOT edit your shell configuration files:
1. You'll want to ensure that `~/.rbenv/bin' is added to PATH.
2. Run `rbenv init' to view instructions on how to configure rbenv for your shell.
3. Launch a new terminal window after editing shell configuration files.
4. (Optional) Run the doctor command to verify the installation:
   wget -q "https://github.com/rbenv/rbenv-installer/raw/HEAD/bin/rbenv-doctor" -O- | bash
The rbenv utility is complaining about not being on the path, so lets add it to the path using the following command.


export PATH="~/.rbenv/bin:$PATH"

Before we install ruby, lets check what version of ruby our project needs by peeking into Gemfile


cat Gemfile

Command Output
# frozen_string_literal: true
source "https://rubygems.org"

#don't upgrade
gem "rails", "6.0.0"

ruby "2.6.5"

gem "aruba"
gem "bcrypt"
gem "coffee-rails"
gem "execjs"
gem "foreman"
gem "jquery-fileupload-rails"
gem "jquery-rails"
gem "minitest"
gem "powder" # Pow related gem
gem "pry-rails" # not in dev group in case running via prod/staging @ a training
gem "puma"
gem "rails-perftest"
gem "rake"
gem "responders" #For Rails 4.2 # LOCKED DOWN
gem "ruby-prof"
gem "sassc-rails"
gem "simplecov", require: false, group: :test
gem "sqlite3"
gem "therubyracer"
gem "turbolinks"
gem "uglifier"
gem "unicorn"

# Add SMTP server support using MailCatcher
# NOTE: https://github.com/sj26/mailcatcher#bundler
# gem 'mailcatcher'

group :development, :mysql do
  gem "better_errors"
  gem "binding_of_caller"
  gem "bundler-audit"
  gem "guard-livereload"
  gem "guard-rspec"
  gem "guard-shell"
  gem "pry"
  gem "rack-livereload"
  gem "rb-fsevent"
  gem "rubocop-github"
  gem "travis-lint"
end

group :development, :test, :mysql do
  gem "capybara"
  gem "database_cleaner"
  gem "launchy"
  gem "poltergeist"
  gem "rspec-rails", '4.0.0.beta3' # 4/26/2019: LOCKED DOWN
  gem "test-unit"
end

group :openshift do
  gem "pg"
end

group :mysql do
  gem "mysql2"
end
Okay, it seems we need to use ruby “2.6.5”. Let’s go ahead and install it using rbenv


rbenv install 2.6.5

Command Output
Downloading ruby-2.6.5.tar.bz2...
-> https://cache.ruby-lang.org/pub/ruby/2.6/ruby-2.6.5.tar.bz2
Installing ruby-2.6.5...

BUILD FAILED (Ubuntu 18.04 using ruby-build 20210510)

Inspect or clean up the working tree at /tmp/ruby-build.20210513032818.200.7hPWvg
Results logged to /tmp/ruby-build.20210513032818.200.log

Last 10 log lines:
checking for ruby... false
checking build system type... x86_64-pc-linux-gnu
checking host system type... x86_64-pc-linux-gnu
checking target system type... x86_64-pc-linux-gnu
checking for gcc... no
checking for cc... no
checking for cl.exe... no
configure: error: in `/tmp/ruby-build.20210513032818.200.7hPWvg/ruby-2.6.5':
configure: error: no acceptable C compiler found in $PATH
See `config.log' for more details
Now, we need to install a compiler. Let’s update apt first.


apt update

Then install the essential compiler for Ruby.


apt-get install build-essential libreadline-dev -y

Command Output
...[SNIP]...

Setting up gcc-7 (7.5.0-3ubuntu1~18.04) ...
Setting up g++-7 (7.5.0-3ubuntu1~18.04) ...
Setting up gcc (4:7.4.0-1ubuntu2.3) ...
Setting up dpkg-dev (1.19.0.5ubuntu2.3) ...
Setting up g++ (4:7.4.0-1ubuntu2.3) ...
update-alternatives: using /usr/bin/g++ to provide /usr/bin/c++ (c++) in auto mode
update-alternatives: warning: skip creation of /usr/share/man/man1/c++.1.gz because associated file /usr/share/man/man1/g++.1.gz (of link group c++) doesn't exist
Setting up build-essential (12.4ubuntu1) ...
Processing triggers for libc-bin (2.27-3ubuntu1.4) ...
Let’s try installing ruby again.


rbenv install --verbose 2.6.5

We need to add ruby to the path.


export PATH="/root/.rbenv/versions/2.6.5/bin:$PATH"

Okay, its done. Now, we can verify the ruby version using the following command.


ruby --version

Command Output
ruby 2.6.5p114 (2019-10-01 revision 67812) [x86_64-linux]
Let’s install the bundler-audit tool to perform SCA.


gem install --user-install bundler-audit

Command Output
Fetching bundler-audit-0.8.0.gem
Fetching thor-1.1.0.gem
WARNING:  You don't have /root/.gem/ruby/2.6.0/bin in your PATH,
          gem executables will not run.
Successfully installed thor-1.1.0
Successfully installed bundler-audit-0.8.0
Parsing documentation for thor-1.1.0
Installing ri documentation for thor-1.1.0
Parsing documentation for bundler-audit-0.8.0
Installing ri documentation for bundler-audit-0.8.0
Done installing documentation for thor, bundler-audit after 1 seconds
2 gems installed
We will add the tool to the PATH, so we can refer it on the command line.


export PATH="~/.gem/ruby/2.6.0/bin/:$PATH"

We have successfully installed bundler-audit.

Let’s explore what options bundle-audit gives us.


bundle-audit -h

Command Output
Commands:
  bundle-audit check [DIR]     # Checks the Gemfile.lock for insecure dependencies
  bundle-audit download        # Downloads ruby-advisory-db
  bundle-audit help [COMMAND]  # Describe available commands or one specific command
  bundle-audit stats           # Prints ruby-advisory-db stats
  bundle-audit update          # Updates the ruby-advisory-db
  bundle-audit version         # Prints the bundler-audit version
Let’s move to the next step.
```
```markdown
Run the OAST/SCA Scan
The idea here is to show you that the tools are not perfect, installation steps might break and even simple things won’t work in a predictable manner.

Let’s perform the SCA scan on our code with the following command.


bundle-audit

Command Output
Download ruby-advisory-db ...
Cloning into '/root/.local/share/ruby-advisory-db'...
remote: Enumerating objects: 5127, done.
remote: Counting objects: 100% (130/130), done.
remote: Compressing objects: 100% (92/92), done.
remote: Total 5127 (delta 48), reused 86 (delta 26), pack-reused 4997
Receiving objects: 100% (5127/5127), 943.82 KiB | 18.51 MiB/s, done.
Resolving deltas: 100% (2505/2505), done.
ruby-advisory-db:
  advisories:   500 advisories
  last updated: 2021-05-10 10:23:43 -0700
Name: actionpack
Version: 6.0.0
CVE: CVE-2021-22904
Criticality: Unknown
URL: https://groups.google.com/g/rubyonrails-security/c/Pf1TjkOBdyQ
Title: Possible DoS Vulnerability in Action Controller Token Authentication
Solution: upgrade to ~> 5.2.4.6, ~> 5.2.6, ~> 6.0.3.7, >= 6.1.3.2

Name: actionpack
Version: 6.0.0
CVE: CVE-2020-8264
Criticality: Unknown
URL: https://groups.google.com/g/rubyonrails-security/c/yQzUVfv42jk
Title: Possible XSS Vulnerability in Action Pack in Development Mode
Solution: upgrade to >= 6.0.3.4

Name: actionpack
Version: 6.0.0
CVE: CVE-2021-22881
Criticality: Unknown
URL: https://groups.google.com/g/rubyonrails-security/c/zN_3qA26l6E
Title: Possible Open Redirect in Host Authorization Middleware
Solution: upgrade to ~> 6.0.3.5, >= 6.1.2.1


...[SNIP]...


Name: rack
Version: 2.0.7
CVE: CVE-2020-8161
Criticality: Unknown
URL: https://groups.google.com/forum/#!topic/ruby-security-ann/T4ZIsfRf2eA
Title: Directory traversal in Rack::Directory app bundled with Rack
Solution: upgrade to ~> 2.1.3, >= 2.2.0

Name: websocket-extensions
Version: 0.1.4
CVE: CVE-2020-7663
GHSA: GHSA-g6wq-qcwm-j5g2
Criticality: High
URL: https://github.com/faye/websocket-extensions-ruby/security/advisories/GHSA-g6wq-qcwm-j5g2
Title: Regular Expression Denial of Service in websocket-extensions (RubyGem)
Solution: upgrade to >= 0.1.5

Vulnerabilities found!
As you can see, there are several vulnerabilities in our Rails application and also bundle-audit allows us to mark an issue or issues as False Positive (FP) using .bundler-audit.yml file, please refer this link to learn more.
```
```markdown
How To Embed bundler-audit Into GitLab
A Simple CI/CD Pipeline
Let’s download the code using git clone in DevSecOps Box.


git clone https://gitlab.practical-devsecops.training/pdso/rails.git rails


cd rails

Rename git url to the new one.


git remote rename origin old-origin


git remote add origin git@gitlab-ce-kr6k1mdm:root/rails.git

Then, push the code into the repository.


git push -u origin --all

And enter the GitLab credentials.

Name	Value
Username	root
Password	pdso-training
The above command will create a new project for rails if not exist.

Next, considering your DevOps team created a simple CI pipeline with the following contents.

Click anywhere to copy

image: docker:20.10

services:
  - docker:dind

stages:
  - build
  - test
  - release
  - preprod
  - integration
  - prod

build:
  stage: build
  script:
    - echo "This is a build step"

test:
  stage: test
  script:
    - echo "This is a test step"

integration:
  stage: integration
  script:
    - echo "This is an integration step"
    - exit 1
  allow_failure: true # Even if the job fails, continue to the next stages

prod:
  stage: prod
  script:
    - echo "This is a deploy step"
  when: manual # Continuous Delivery

Let’s log into GitLab using the following details and run the above pipeline.

GitLab CI/CD Machine
Name	Value
URL	https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/rails/-/blob/main/.gitlab-ci.yml
Username	root
Password	pdso-training
Next, we need to create a CI/CD pipeline by replacing the .gitlab-ci.yml file content with the above CI script. Click on the Edit button to replace the content (use Control+A and Control+V).

Save changes to the file using the Commit changes button.

Verify the pipeline runs
As soon as a change is made to the repository, the pipeline starts executing the jobs.

We can see the results of this pipeline by visiting https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/rails/pipelines.

Click on the appropriate job name to see the output.

Let’s move to the next step.
```
```markdown
Embed bundler-audit in CI/CD pipeline
As discussed in the SCA using bundler-audit exercise, we can embed the bundler-audit tool in our CI/CD pipeline.

However, do remember that you need run commands manually in a local system like devsecops box before embedding commands for automated execution in CI/CD pipelines.

Manually testing commands in a local environment like devsecops box helps in easy troubleshooting.

Do you wonder which stage this job should go into?

Maybe you want to scan the components before performing SAST scans.

Click anywhere to copy

image: docker:20.10

services:
  - docker:dind

stages:
  - build
  - test
  - release
  - preprod
  - integration
  - prod

build:
  stage: build
  script:
    - echo "This is a build step"

test:
  stage: test
  script:
    - echo "This is a test step"

bundler-audit:
  stage: test
  script:
    - docker run --rm -v $(pwd):/src -w /src hysnsec/bundle-audit check --format json --output bundle-audit-output.json
  artifacts:
    paths: [bundle-audit-output.json]
    when: always # What is this for?
    expire_in: one week

integration:
  stage: integration
  script:
    - echo "This is an integration step"
    - exit 1
  allow_failure: true # Even if the job fails, continue to the next stages

prod:
  stage: prod
  script:
    - echo "This is a deploy step"
  when: manual # Continuous Delivery

Copy the above CI script and add it to the .gitlab.ci.yml file on Gitlab repo at https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/rails/-/blob/main/.gitlab-ci.yml.

Do not forget to click on the Commit Changes button to save the file.

As discussed, any change to the repo kick starts the pipeline.

We can see the results of this pipeline by visiting https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/rails/pipelines.

Click on the appropriate job name to see the output.

Remember!

Did you notice the job failed because of non zero exit code(exit code 1)?

Why did the job fail? Because the tool found security issues. This is the expected behavior of any DevSecOps friendly tool.

You need to either fix the security issues or add allow_failure: true to let the job finish without blocking other jobs.

If you have troubles understanding exit code, please go through the exercise titled Working with Exit Code.

You will notice that the bundler-audit job stores the output to a file bundle-audit-output.json.

In the next step, you will learn the need to not fail the builds.
```
```markdown
Allow the job failure
Remember!

Except for DevSecOps-Box, every other machine closes after two hours, even if you are in the middle of the exercise
After two hours, in case of a 404, you need to refresh the exercise page and click on Start the Exercise button to continue working
You do not want to fail the builds (pipeline) in DevSecOps Maturity Levels 1 and 2. If a security tool fails a build upon security findings, you would want to allow it to fail and not block the pipeline as there would be false positives in the results.

You can use the allow_failure tag to not fail the builds even though the tool found security issues.

bundler-audit:
  stage: test
  script:
    - docker run --rm -v $(pwd):/src -w /src hysnsec/bundle-audit check --format json --output bundle-audit-output.json
  artifacts:
    paths: [bundle-audit-output.json]
    when: always # What is this for?
    expire_in: one week
  allow_failure: true  #<--- allow the build to fail but don't mark it as such
Notice allow_failure: true at the end of the YAML file.

The final pipeline would look like the following.

Click anywhere to copy

image: docker:20.10

services:
  - docker:dind

stages:
  - build
  - test
  - release
  - preprod
  - integration
  - prod

build:
  stage: build
  script:
    - echo "This is a build step"

test:
  stage: test
  script:
    - echo "This is a test step"

bundler-audit:
  stage: test
  script:
    - docker run --rm -v $(pwd):/src -w /src hysnsec/bundle-audit check --format json --output bundle-audit-output.json
  artifacts:
    paths: [bundle-audit-output.json]
    when: always # What is this for?
    expire_in: one week
  allow_failure: true  #<--- allow the build to fail but don't mark it as such

integration:
  stage: integration
  script:
    - echo "This is an integration step"
    - exit 1
  allow_failure: true # Even if the job fails, continue to the next stages

prod:
  stage: prod
  script:
    - echo "This is a deploy step"
  when: manual # Continuous Delivery

Go ahead and add it to the .gitlab-ci.yml file to run the pipeline.

Understanding -v option in docker command
You will notice that the bundler-audit job has failed but didn’t block the other jobs from running.

As discussed, any change to the repo kick starts the pipeline.

Note

We’ve discussed different methods of using the tool:

Native Installation:
Directly installing the tool on the system.
Package Manager or Binary:
Installing via a package manager or using the binary file.
Docker:
Running the tool within a Docker container.
In summary:

All methods are suitable for CI/CD integration.
Docker is recommended for CI/CD as it operates smoothly without dependencies.
Using the binary file is efficient, avoiding additional dependencies.
Ultimately, you can choose either method based on your specific situation.

We can see the results of this pipeline by visiting https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/rails/pipelines.

Click on the appropriate job name to see the output.

Back
Mark as compl
```
```markdown
Software Component Analysis Using Composer Audit
Download the source code
We will do all the exercises locally first in DevSecOps-Box, so let’s start the exercise.

First, We need to download the source code of the project from our git repository.


git clone https://gitlab.practical-devsecops.training/pdso/php.git

Command Output
Cloning into 'php'...
warning: redirecting to https://gitlab.practical-devsecops.training/pdso/php.git/
remote: Enumerating objects: 312, done.
remote: Counting objects: 100% (50/50), done.
remote: Compressing objects: 100% (40/40), done.
remote: Total 312 (delta 29), reused 23 (delta 10), pack-reused 262
Receiving objects: 100% (312/312), 756.40 KiB | 19.39 MiB/s, done.
Resolving deltas: 100% (111/111), done.
Let’s cd into the application code so we can scan the app.


cd php

We are now in the php directory.

Let’s move to the next step.
```
```markdown
Install Composer version 2
Composer is a tool that manages dependencies for PHP. It simplifies the process of managing PHP software packages and libraries, allowing developers to easily declare the libraries that their projects depend on, and manage them. Composer version 2 comes with a security feature that analyzes dependencies defined in a repository using the composer audit command.

We will install PHP and also Composer with the following command.


apt update && apt install -y software-properties-common
add-apt-repository -y ppa:ondrej/php
apt update
apt install -y php7.4 php7.4-gd php7.4-intl php7.4-xsl php7.4-mbstring php7.4-curl

Then we will install composer version 2 which has audit feature.


php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
php composer-setup.php --install-dir=/usr/local/bin --filename=composer
php -r "unlink('composer-setup.php');"

composer audit tool is part of composer version 2. Let’s check the help menu of composer audit to understand its feature set.


composer audit -h

Command Output
Description:
  Checks for security vulnerability advisories for installed packages

Usage:
  audit [options]

Options:
      --no-dev                   Disables auditing of require-dev packages.
  -f, --format=FORMAT            Output format. Must be "table", "plain", "json", or "summary". [default: "table"]
      --locked                   Audit based on the lock file instead of the installed packages.
  -h, --help                     Display help for the given command. When no command is given display help for the list command
  -q, --quiet                    Do not output any message
  -V, --version                  Display this application version
      --ansi|--no-ansi           Force (or disable --no-ansi) ANSI output
  -n, --no-interaction           Do not ask any interactive question
      --profile                  Display timing and memory usage information
      --no-plugins               Whether to disable plugins.
      --no-scripts               Skips the execution of all scripts defined in composer.json file.
  -d, --working-dir=WORKING-DIR  If specified, use the given directory as working directory.
      --no-cache                 Prevent use of the cache
  -v|vv|vvv, --verbose           Increase the verbosity of messages: 1 for normal output, 2 for more verbose output and 3 for debug

Help:
  The audit command checks for security vulnerability advisories for installed packages.

  If you do not want to include dev dependencies in the audit you can omit them with --no-dev

  Read more at https://getcomposer.org/doc/03-cli.md#audit
Which options would you choose from this list?

Let’s move to the next step.
```
```markdown
Run the Scanner
We can run the basic command without any extra arguments, however the output is not suitable for a CI/CD pipeline because the output is not in a machine readable format.

First, we need to install the dependencies using composer install.


composer install

Let’s run the composer audit command.


composer audit

Command Output
Found 15 security vulnerability advisories affecting 7 packages:
+-------------------+----------------------------------------------------------------------------------+
| Package           | guzzlehttp/guzzle                                                                |
| CVE               | CVE-2022-31091                                                                   |
| Title             | Change in port should be considered a change in origin                           |
| URL               | https://github.com/guzzle/guzzle/security/advisories/GHSA-q559-8m2m-g699         |
| Affected versions | >=7,<7.4.5|>=4,<6.5.8                                                            |
| Reported at       | 2022-06-20T22:24:00+00:00                                                        |
+-------------------+----------------------------------------------------------------------------------+
+-------------------+----------------------------------------------------------------------------------+
| Package           | guzzlehttp/guzzle                                                                |
| CVE               | CVE-2022-31090                                                                   |
| Title             | CURLOPT_HTTPAUTH option not cleared on change of origin                          |
| URL               | https://github.com/guzzle/guzzle/security/advisories/GHSA-25mq-v84q-4j7r         |
| Affected versions | >=7,<7.4.5|>=4,<6.5.8                                                            |
| Reported at       | 2022-06-20T22:24:00+00:00                                                        |
+-------------------+----------------------------------------------------------------------------------+
+-------------------+----------------------------------------------------------------------------------+
| Package           | guzzlehttp/guzzle                                                                |
| CVE               | CVE-2022-31043                                                                   |
| Title             | Fix failure to strip Authorization header on HTTP downgrade                      |
| URL               | https://github.com/guzzle/guzzle/security/advisories/GHSA-w248-ffj2-4v5q         |
| Affected versions | >=7,<7.4.4|>=4,<6.5.7                                                            |
| Reported at       | 2022-06-09T23:36:00+00:00                                                        |
+-------------------+----------------------------------------------------------------------------------+
+-------------------+----------------------------------------------------------------------------------+
| Package           | guzzlehttp/guzzle                                                                |
| CVE               | CVE-2022-31042                                                                   |
| Title             | Failure to strip the Cookie header on change in host or HTTP downgrade           |
| URL               | https://github.com/guzzle/guzzle/security/advisories/GHSA-f2wf-25xc-69c9         |
| Affected versions | >=7,<7.4.4|>=4,<6.5.7                                                            |
| Reported at       | 2022-06-09T23:36:00+00:00                                                        |
+-------------------+----------------------------------------------------------------------------------+
+-------------------+----------------------------------------------------------------------------------+
| Package           | guzzlehttp/guzzle                                                                |
| CVE               | CVE-2022-29248                                                                   |
| Title             | Cross-domain cookie leakage                                                      |
| URL               | https://github.com/guzzle/guzzle/security/advisories/GHSA-cwmx-hcrq-mhc3         |
| Affected versions | >=7,<7.4.3|>=4,<6.5.6                                                            |
| Reported at       | 2022-05-25T13:21:00+00:00                                                        |
+-------------------+----------------------------------------------------------------------------------+

...[SNIP]...
As we can see, the basic command gives us the report in a table format

As we have learned in the DevSecOps Gospel, we should save the output in a machine-readable format so the output can be parsed by the machines.

We can save the output in JSON format using -f flag.


composer audit -f json | tee results.json

Command Output

...[SNIP]...

            "2": {
                "title": "CVE-2020-15094: Prevent RCE when calling untrusted remote with CachingHttpClient",
                "cve": "CVE-2020-15094",
                "link": "https://symfony.com/cve-2020-15094",
                "reportedAt": "2020-09-02T08:00:00+00:00",
                "sources": [
                    {
                        "name": "GitHub",
                        "remoteId": "GHSA-754h-5r27-7x3r"
                    },
                    {
                        "name": "FriendsOfPHP/security-advisories",
                        "remoteId": "symfony/http-kernel/CVE-2020-15094.yaml"
                    }
                ],
                "advisoryId": "PKSA-8knv-7jsn-12w8",
                "packageName": "symfony/http-kernel",
                "affectedVersions": ">=4.3.0,<4.4.0|>=4.4.0,<4.4.13|>=5.0.0,<5.1.0|>=5.1.0,<5.1.5"
            },
            "3": {
                "title": "CVE-2019-18887: Use constant time comparison in UriSigner",
                "cve": "CVE-2019-18887",
                "link": "https://symfony.com/cve-2019-18887",
                "reportedAt": "2019-11-13T08:00:00+00:00",
                "sources": [
                    {
                        "name": "GitHub",
                        "remoteId": "GHSA-q8hg-pf8v-cxrv"
                    },
                    {
                        "name": "FriendsOfPHP/security-advisories",
                        "remoteId": "symfony/http-kernel/CVE-2019-18887.yaml"
                    }
                ],
                "advisoryId": "PKSA-sn9k-4yr8-6s9c",
                "packageName": "symfony/http-kernel",
                "affectedVersions": ">=2.2.0,<2.3.0|>=2.3.0,<2.4.0|>=2.4.0,<2.5.0|>=2.5.0,<2.6.0|>=2.6.0,<2.7.0|>=2.7.0,<2.8.0|>=2.8.0,<2.8.52|>=3.0.0,<3.1.0|>=3.1.0,<3.2.0|>=3.2.0,<3.3.0|>=3.3.0,<3.4.0|>=3.4.0,<3.4.35|>=4.0.0,<4.1.0|>=4.1.0,<4.2.0|>=4.2.0,<4.2.12|>=4.3.0,<4.3.8"
            }
        },
        "symfony/mime": [
            {
                "title": "CVE-2019-18888: Prevent argument injection in a MimeTypeGuesser",
                "cve": "CVE-2019-18888",
                "link": "https://symfony.com/cve-2019-18888",
                "reportedAt": "2019-11-13T08:00:00+00:00",
                "sources": [
                    {
                        "name": "GitHub",
                        "remoteId": "GHSA-xhh6-956q-4q69"
                    },
                    {
                        "name": "FriendsOfPHP/security-advisories",
                        "remoteId": "symfony/mime/CVE-2019-18888.yaml"
                    }
                ],
                "advisoryId": "PKSA-7y15-t1fp-w94f",
                "packageName": "symfony/mime",
                "affectedVersions": ">=4.3.0,<4.3.8"
            }
        ]
    }
}
Variable Scan Results

Your results might slightly vary because of the dynamic landscape of changing vulnerabilities, and security updates.

There you go. Why don’t you embed composer audit in CI/CD pipeline?
```
```markdown
How To Embed Composer Into GitLab
A Simple CI/CD Pipeline
Let’s download the code using git clone in DevSecOps Box.


git clone https://gitlab.practical-devsecops.training/pdso/php.git


cd php

Rename git url to the new one.


git remote rename origin old-origin


git remote add origin git@gitlab-ce-kr6k1mdm:root/php.git

Then, push the code into the repository and it will automatically create a new project if not exist.


git push -u origin --all

And enter the GitLab credentials.

Name	Value
Username	root
Password	pdso-training
Considering your DevOps team created a simple CI pipeline with the following contents.

Hence, you should create .gitlab-ci.yml file in php repository by visiting https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/php

create-new-file

new-file-format

Click anywhere to copy

image: docker:20.10  # To run all jobs in this pipeline, use the latest docker image

services:
  - docker:dind       # To run all jobs in this pipeline, use a docker image that contains a docker daemon running inside (dind - docker in docker). Reference: https://forum.gitlab.com/t/why-services-docker-dind-is-needed-while-already-having-image-docker/43534

stages:
  - integration
  - prod

integration:
  stage: integration
  script:
    - echo "This is an integration step"
    - exit 1
  allow_failure: true # Even if the job fails, continue to the next stages

prod:
  stage: prod
  script:
    - echo "This is a deploy step."
  when: manual # Continuous Delivery

We have 2 jobs in this pipeline, a integration job and a prod job. We did integrate SCA/OAST beforehand, we can carry forward the same tactics in this exercise as well.

As a security engineer, I don’t need to care much about what the DevOps team is doing as part of these jobs. Why? imagine having to learn every build/testing tool used by your DevOps team, it will be a nightmare. Instead, rely on the DevOps team for help.

Let’s login into the GitLab using the following details and execute this pipeline.

GitLab CI/CD Machine
Name	Value
Link	https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/php/-/blob/main/.gitlab-ci.yml
Username	root
Password	pdso-training
Next, we need to create a CI/CD pipeline by replacing the .gitlab-ci.yml file content with the above CI script. Click on the Edit button to replace the content (use Control+A and Control+V).

Save changes to the file using the Commit changes button.

Verify the pipeline run
As soon as a change is made to the repository, the pipeline starts executing the jobs.

We can see the results of this pipeline by visiting https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/php/pipelines.

Click on the appropriate job name to see the output.

In the next step, we will embed PHPstan in the CI/CD pipeline and follow all the best practices.

Let’s move to the next step.
```
```markdown
Embed Composer Audit in CI/CD pipeline
As discussed in the SCA using Composer exercise, we can embed Composer Audit in our CI/CD pipeline. However, do remember you need to run the command manually before you embed this SAST tool in the pipeline.

Click anywhere to copy

image: docker:20.10  # To run all jobs in this pipeline, use the latest docker image

services:
  - docker:dind       # To run all jobs in this pipeline, use a docker image that contains a docker daemon running inside (dind - docker in docker). Reference: https://forum.gitlab.com/t/why-services-docker-dind-is-needed-while-already-having-image-docker/43534

stages:
  - build
  - integration
  - prod

oast-backend:
  stage: build  # we moved this job from test stage to build stage, by replacing the text test to build
  image: php:7.4
  before_script:
    - php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
    - php composer-setup.php --install-dir=/usr/local/bin --filename=composer
    - php -r "unlink('composer-setup.php');"
    - apt update
    - apt install unzip
  script:
    - composer install
    - composer audit -f json | tee composer-output.json
  artifacts:
    paths: [composer-output.json]
    when: always

integration:
  stage: integration
  script:
    - echo "This is an integration step"
    - exit 1
  allow_failure: true # Even if the job fails, continue to the next stages

prod:
  stage: prod
  script:
    - echo "This is a deploy step."
  when: manual # Continuous Delivery

As discussed, any change to the repo kick starts the pipeline.

Note

We’ve discussed different methods of using the tool:

Native Installation:
Directly installing the tool on the system.
Package Manager or Binary:
Installing via a package manager or using the binary file.
Docker:
Running the tool within a Docker container.
In summary:

All methods are suitable for CI/CD integration.
Docker is recommended for CI/CD as it operates smoothly without dependencies.
Using the binary file is efficient, avoiding additional dependencies.
Ultimately, you can choose either method based on your specific situation.

We can see the results of this pipeline by visiting https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/php/pipelines.

Click on the appropriate job name to see the output.

Remember!

Did you notice the job failed because of non zero exit code(exit 13)?

Why did the job fail? Because the tool found security issues. This is the expected behavior of any DevSecOps friendly tool.

You need to either fix the security issues or add allow_failure: true to let the job finish without blocking other jobs.

If you have troubles understanding exit code, please go through the exercise titled Working with Exit Code.

You will notice that the oast-backend job stores the output to a file composer-report.json.

Let’s move to the next step.
```
```markdown
Allow the job failure
Remember!

Except for DevSecOps-Box, every other machine closes after two hours, even if you are in the middle of the exercise
After two hours, in case of a 404, you need to refresh the exercise page and click on Start the Exercise button to continue working
You do not want to fail the builds (pipeline) in DevSecOps Maturity Levels 1 and 2. If a security tool fails a job, it won’t allow other DevOps jobs like release/deploy to run hence causing great distress to DevOps Team. Moreover, the security tools suffer from false positives. Failing a build on inaccurate data is a sure recipe for disaster.

You can use the allow_failure tag to not fail the build even though the tool found security issues.

Command Output
oast-backend:
  stage: build  # we moved this job from test stage to build stage, by replacing the text test to build
  image: php:7.4
  before_script:
    - php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
    - php composer-setup.php --install-dir=/usr/local/bin --filename=composer
    - php -r "unlink('composer-setup.php');"
    - apt update
    - apt install unzip
  script:
    - composer install
    - composer audit -f json | tee composer-output.json
  artifacts:
    paths: [composer-output.json]
    when: always
  allow_failure: true #<--- allow the build to fail but don't mark it as such
The pipeline would look like the following.

Click anywhere to copy

image: docker:20.10  # To run all jobs in this pipeline, use the latest docker image

services:
  - docker:dind       # To run all jobs in this pipeline, use a docker image that contains a docker daemon running inside (dind - docker in docker). Reference: https://forum.gitlab.com/t/why-services-docker-dind-is-needed-while-already-having-image-docker/43534

stages:
  - build
  - test
  - release
  - preprod
  - integration
  - prod

build:
  stage: build
  script:
    - echo "This is a build step"

test:
  stage: test
  script:
    - echo "This is a test step"

oast-backend:
  stage: build 
  image: php:7.4
  before_script:
    - php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
    - php composer-setup.php --install-dir=/usr/local/bin --filename=composer
    - php -r "unlink('composer-setup.php');"
    - apt update
    - apt install unzip
  script:
    - composer install
    - composer audit -f json | tee composer-output.json
  artifacts:
    paths: [composer-output.json]
    when: always
  allow_failure: true

integration:
  stage: integration
  script:
    - echo "This is an integration step"
    - exit 1
  allow_failure: true # Even if the job fails, continue to the next stages

prod:
  stage: prod
  script:
    - echo "This is a deploy step."
  when: manual # Continuous Delivery

You will notice that the oast-backend job has failed, but other jobs and stages ran fine.

As discussed, any change to the repo kick starts the pipeline.

We can see the results of this pipeline by visiting https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/php/pipelines.

Click on the appropriate job name to see the output/results.
```
```markdown
Addressing Issues via Renovate on GitLab
Downloading The Source Code
We will do all the exercises locally first in DevSecOps-Box, so let’s start the exercise.

First, we need to download the source code of the project from our git repository.


git clone git@gitlab-ce-kr6k1mdm:root/django-nv.git

Authentication Confirmation Required

If you encounter the following prompt while trying to clone GitLab, Please type yes and press Enter to proceed.

Command Output
The authenticity of host 'gitlab-ce-kr6k1mdm (10.x.x.x)' can't be established.
ECDSA key fingerprint is SHA256:U1W8wQm8tgHmuF/T0uf1jNVCzHyhqeJ4MmGG/K4XmcI.
Are you sure you want to continue connecting (yes/no/[fingerprint])?
Lets cd into the application code, so we can scan the app.


cd django-nv

We are now in the django-nv directory.

Let’s move to the next step.
```
```markdown
Installing Renovate
Renovate is an open-source tool that automates the process of updating dependencies in software projects. It runs continuously to detect outdated dependencies and opens pull requests with updates. Renovate is primarily designed to be used with version control systems like Git.

As Renovate requires npm, let’s proceed to install NodeJS and NPM using the following command.


mkdir -p /etc/apt/keyrings
curl -fsSL https://deb.nodesource.com/gpgkey/nodesource-repo.gpg.key | gpg --dearmor -o /etc/apt/keyrings/nodesource.gpg
NODE_MAJOR=20
echo "deb [signed-by=/etc/apt/keyrings/nodesource.gpg] https://deb.nodesource.com/node_$NODE_MAJOR.x nodistro main" | tee /etc/apt/sources.list.d/nodesource.list
apt update


apt install nodejs -y

Let’s install Renovate with NPM.


npm install -g renovate

Note

Kindly exercise patience as the installation of Renovate may take approximately 2-4 minutes.

If you notice an outdated version of npm post the installation of Renovate, please ignore them for now.

Excellent, we’ve successfully installed the tool. Now, let’s explore the help menu to better comprehend its array of features.


renovate -h

Command Output
Usage: renovate [options] [repositories...]

Options:
  --detect-global-manager-config [boolean]                     If `true`, Renovate tries to detect global manager configuration from the file system.
  --detect-host-rules-from-env [boolean]                       If `true`, Renovate tries to detect host rules from environment variables.
  --allow-post-upgrade-command-templating [boolean]            Set this to `false` to disable template compilation for post-upgrade commands.
  --allowed-post-upgrade-commands <array>                      A list of regular expressions that decide which post-upgrade tasks are allowed.
  --post-upgrade-tasks <object>                                Post-upgrade tasks that are executed before a commit is made by Renovate.
  --onboarding-no-deps [boolean]                               Onboard the repository even if no dependencies are found.
  --config-migration [boolean]                                 Enable this to get config migration PRs when needed.
  --product-links <object>                                     Links which are used in PRs, issues and comments.
  --secrets <object>                                           Object which holds secret name/value pairs.
  --migrate-presets <object>                                   Define presets here which have been removed or renamed and should be migrated automatically.
  --global-extends <array>                                     Configuration presets to use or extend for a self-hosted config.
  --enabled [boolean]                                          Enable or disable Renovate bot.
  --repository-cache <string>                                  This option decides if Renovate uses a JSON cache to speed up extractions.
  --repository-cache-type <string>                             Set the type of renovate repository cache if `repositoryCache` is enabled.
  --force-cli [boolean]                                        Decides if CLI configuration options are moved to the `force` config section.
  --draft-p-r [boolean]                                        If set to `true` then Renovate creates draft PRs, instead of normal status PRs.
  --dry-run <string>                                           If enabled, perform a dry run by logging messages instead of creating/updating/deleting
                                                               branches and PRs.
  --print-config [boolean]                                     If enabled, Renovate logs the fully resolved config for each repository, plus the fully
                                                               resolved presets.
  --binary-source <string>                                     Controls how third-party tools like npm or Gradle are called: directly, via Docker sidecar
                                                               containers, or via dynamic install.
  --redis-url <string>                                         If set, this Redis URL will be used for caching instead of the file system.
  --base-dir <string>                                          The base directory for Renovate to store local files, including repository files and cache. If
                                                               left empty, Renovate will create its own temporary directory to use.
  --cache-dir <string>                                         The directory where Renovate stores its cache. If left empty, Renovate creates a subdirectory
                                                               within the `baseDir`.
  --containerbase-dir <string>                                 The directory where Renovate stores its containerbase cache. If left empty, Renovate creates a
                                                               subdirectory within the `cacheDir`.
  --custom-env-variables <object>                              Custom environment variables for child processes and sidecar Docker containers.
  --custom-datasources <object>                                Defines custom datasources for usage by managers
  --docker-child-prefix <string>                               Change this value to add a prefix to the Renovate Docker sidecar container names and labels.
  --docker-cli-options <string>                                Pass CLI flags to `docker run` command when `binarySource=docker`.
  --docker-sidecar-image <string>                              Change this value to override the default Renovate sidecar image.
  --docker-user <string>                                       Set the `UID` and `GID` for Docker-based binaries if you use `binarySource=docker`.
  --composer-ignore-platform-reqs <array>                      Configure use of `--ignore-platform-reqs` or `--ignore-platform-req` for the Composer package
                                                               manager.
  --go-get-dirs <array>                                        Directory pattern to run `go get` on
  --log-file <string>                                          Log file path.
  --log-file-level <string>                                    Set the log file log level.
  --log-context <string>                                       Add a global or per-repo log context to each log entry.
  --onboarding [boolean]                                       Require a Configuration PR first.
  --onboarding-config <object>                                 Configuration to use for onboarding PRs.
  --onboarding-rebase-checkbox [boolean]                       Set to enable rebase/retry markdown checkbox for onboarding PRs.
  --fork-processing <string>                                   Whether to process forked repositories. By default, all forked repositories are skipped when in
                                                               `autodiscover` mode.
  --include-mirrors [boolean]                                  Whether to process repositories that are mirrors. By default, repositories that are mirrors are
                                                               skipped.
  --fork-token <string>                                        Set a personal access token here to enable "fork mode".
  --fork-org <string>                                          The preferred organization to create or find forked repositories, when in fork mode.
  --github-token-warn [boolean]                                Display warnings about GitHub token not being set.
  --require-config <string>                                    Controls Renovate's behavior regarding repository config files such as `renovate.json`.
  --optimize-for-disabled [boolean]                            Set to `true` to perform a check for disabled config prior to cloning.
  --dependency-dashboard [boolean]                             Whether to create a "Dependency Dashboard" issue in the repository.
  --dependency-dashboard-approval [boolean]                    Controls if updates need manual approval from the Dependency Dashboard issue before PRs are
                                                               created.
  --dependency-dashboard-autoclose [boolean]                   Set to `true` to let Renovate close the Dependency Dashboard issue if there are no more
                                                               updates.
  --dependency-dashboard-title <string>                        Title for the Dependency Dashboard issue.
  --dependency-dashboard-header <string>                       Any text added here will be placed first in the Dependency Dashboard issue body.
  --dependency-dashboard-footer <string>                       Any text added here will be placed last in the Dependency Dashboard issue body, with a divider
                                                               separator before it.
  --dependency-dashboard-labels <array>                        These labels will always be applied on the Dependency Dashboard issue, even when they have been
                                                               removed manually.
  --dependency-dashboard-o-s-v-vulnerability-summary <string>  Control if the Dependency Dashboard issue lists CVEs supplied by [osv.dev](https://osv.dev).
  --config-warning-reuse-issue [boolean]                       Set this to `false` to make Renovate create a new issue for each config warning, instead of
                                                               reopening or reusing an existing issue.
  --private-key <string>                                       Server-side private key.
  --private-key-old <string>                                   Secondary or old private key to try.
  --private-key-path <string>                                  Path to the Server-side private key.
  --private-key-path-old <string>                              Path to the Server-side old private key.
  --encrypted <object>                                         An object containing configuration encrypted with project key.
  --timezone <string>                                          [IANA Time Zone](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones)
  --schedule <array>                                           Limit branch creation to these times of day or week.
  --automerge-schedule <array>                                 Limit automerge to these times of day or week.
  --update-not-scheduled [boolean]                             Whether to update branches when not scheduled. Renovate will not create branches outside of the
                                                               schedule.
  --persist-repo-data [boolean]                                If set to `true`: keep repository data between runs instead of deleting the data.
  --expose-all-env [boolean]                                   Set this to `true` to allow passing of all environment variables to package managers.
  --allow-plugins [boolean]                                    Set this to `true` if repositories are allowed to run install plugins.
  --allow-scripts [boolean]                                    Set this to `true` if repositories are allowed to run install scripts.
  --allow-custom-crate-registries [boolean]                    Set this to `true` to allow custom crate registries.
  --ignore-plugins [boolean]                                   Set this to `true` if `allowPlugins=true` but you wish to skip running plugins when updating
                                                               lock files.
  --ignore-scripts [boolean]                                   Set this to `false` if `allowScripts=true` and you wish to run scripts when updating lock
                                                               files.
  --platform <string>                                          Platform type of repository.
  --endpoint <string>                                          Custom endpoint to use.
  --token <string>                                             Repository Auth Token.
  --username <string>                                          Username for authentication.
  --password <string>                                          Password for authentication.
  --npmrc <string>                                             String copy of `.npmrc` file. Use `\n` instead of line breaks.
  --npmrc-merge [boolean]                                      Whether to merge `config.npmrc` with repo `.npmrc` content if both are found.
  --npm-token <string>                                         npm token used to authenticate with the default registry.
  --update-lock-files [boolean]                                Set to `false` to disable lock file updating.
  --skip-installs [boolean]                                    Skip installing modules/dependencies if lock file updating is possible without a full install.
  --autodiscover [boolean]                                     Autodiscover all repositories.
  --autodiscover-filter <array>                                Filter the list of autodiscovered repositories.
  --autodiscover-topics <array>                                Filter the list of autodiscovered repositories by topics.
  --pr-commits-per-run-limit <integer>                         Set the maximum number of commits per Renovate run. By default there is no limit.
  --use-base-branch-config <string>                            Whether to read configuration from `baseBranches` instead of only the default branch.
  --git-author <string>                                        Author to use for Git commits. Must conform to
                                                               [RFC5322](https://datatracker.ietf.org/doc/html/rfc5322).
  --git-ignored-authors <array>                                Git authors which are ignored by Renovate. Must conform to
                                                               [RFC5322](https://datatracker.ietf.org/doc/html/rfc5322).
  --git-timeout <integer>                                      Configure the timeout with a number of milliseconds to wait for a Git task.

...[SNIP]...

  -v, --version                                                output the version number
  -h, --help                                                   display help for command
  Examples:

    $ renovate --token 123test singapore/lint-condo
    $ LOG_LEVEL=debug renovate --labels=renovate,dependency --ignore-unstable=false singapore/lint-condo
    $ renovate singapore/lint-condo singapore/package-test
Spend some time to review the number of options provided by the renovate tool.

Renovate provides these following features:

Feature	Description
Automation	Renovate automates the process of identifying and updating outdated dependencies in software projects.
Customization	Renovate offers extensive customization options to suit various project needs and workflows.
Scalability	Renovate can handle projects of any size, from small to large-scale applications.
Support	Renovate provides support for a broad spectrum of languages and package managers, namely JavaScript (npm & Yarn), Python (Pip), Java (Maven), PHP (Composer), Ruby (Bundler), Docker and more.
Efficiency	By automating dependency management, Renovate frees up developers’ time for writing code, leading to improved productivity.
Security & Stability	By keeping dependencies updated, Renovate helps integrate the latest security patches and bug fixes promptly, therefore contributing to safer and more stable software.
Let’s move to the next step.
```
```markdown
Running The Scanner
First, we have to check the git version before running the scanner.

The minimum Git version to run Renovate is 2.33.0

Let’s check the git version.


git --version

Command Output
git version 2.34.1
Splendid! We have the reached the minimum Git version to run Renovate, now let’s run the scanner.


renovate

Command Output
FATAL: You must configure a GitHub token
 INFO: Renovate is exiting with a non-zero code due to the following logged errors
       "loggerErrors": [
         {
           "name": "renovate",
           "level": 60,
           "logContext": "s07HAxdQDgUL0FHaV9FjW",
           "msg": "You must configure a GitHub token"
         }
       ]

We have an error that requires us to configure a GitHub token for Renovate. To run renovate we have to configure our GitHub Token, in the next step we will learn how to configure GitHub Token.
In this exercise, we will be utilizing GitLab. Therefore, we’ll set up GitLab Access in the subsequent step.

Let’s move to the next step to configure GitLab in Renovate.
```
```markdown
Initiating Renovate For Addressing Issue
In the preceding step, we configured GitLab to accept changes initiated by Renovate. Now, we’ll proceed with this process.

Renovate is programmed to automate alterations to our GitLab.

GitLab Setup
In this step, we will set up the gitlab access to integrate renovate to a gitlab repository.

Why We Need Use Git To Use Renovate?

Renovate uses GitHub (as well as other platforms like GitLab or Bitbucket) for several reasons:

Feature              Description
Version Control      Renovate is built to work with version control systems. It uses Git to create branches and commits for each dependency update it proposes. This maintains a record of all changes and allows for easier rollbacks if something goes wrong.
Collaboration and Review   By creating pull or merge requests in GitHub, changes are visible to all team members, open for discussion and can be reviewed and merged by others. Automated testing integrations in the repository can test the changes before merging.
Automated Workflow   Everything is integrated into the developer’s usual workflow. Developers don’t have to manually check for updates, the bot will asynchronously alert them when updates are needed.
Access to Dependency Files Renovate needs access to files like package.json, composer.json, pom.xml, etc., which are typically stored in the repository, to determine which dependencies are outdated.

For these reasons, Renovate needs access to repositories hosted on services like GitLab. However, a token for authentication is required to access private repositories or to make API requests beyond a certain limit. This token must be provided in environment variables when running Renovate.

Create A Personal Access Token (PAT) In GitLab Account
```
```markdown
Addressing Issues via Renovate on GitLab
Initiating Renovate For Addressing Issue
In the preceding step, we configured GitLab to accept changes initiated by Renovate. Now, we’ll proceed with this process.

Renovate is programmed to automate alterations to our GitLab.

GitLab Setup
In this step, we will set up the gitlab access to integrate renovate to a gitlab repository.

Why We Need Use Git To Use Renovate?

Renovate uses GitHub (as well as other platforms like GitLab or Bitbucket) for several reasons:

Feature	Description
Version Control	Renovate is built to work with version control systems. It uses Git to create branches and commits for each dependency update it proposes. This maintains a record of all changes and allows for easier rollbacks if something goes wrong.
Collaboration and Review	By creating pull or merge requests in GitHub, changes are visible to all team members, open for discussion and can be reviewed and merged by others. Automated testing integrations in the repository can test the changes before merging.
Automated Workflow	Everything is integrated into the developer’s usual workflow. Developers don’t have to manually check for updates, the bot will asynchronously alert them when updates are needed.
Access to Dependency Files	Renovate needs access to files like package.json, composer.json, pom.xml, etc., which are typically stored in the repository, to determine which dependencies are outdated.
For these reasons, Renovate needs access to repositories hosted on services like GitLab. However, a token for authentication is required to access private repositories or to make API requests beyond a certain limit. This token must be provided in environment variables when running Renovate.

Create A Personal Access Token (PAT) In GitLab Account
Let’s log into GitLab using the following details.

Name	Value
URL	https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/django-nv/-/blob/main/.gitlab-ci.yml
Username	root
Password	pdso-training
Next, we will create our Personal Access Token from GitLab at :

https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/-/profile/personal_access_tokens.
Provide the token name as renovatebot, select the following checkboxes:

api
read_api
read_repository
write_repository
After that click on Create personal access token button.

Note

For the intended operation, we need to assign the following functions to their respective categories:

Permission	Description
api	This grants complete read/write access to the API, which includes all groups and projects, the container registry, and the package registry.
read_api	This provides read access to the API, covering all groups and projects, the container registry, and the package registry.
read_repository	This allows read-only access to repositories on private projects using Git-over-HTTP or the Repository Files API.
write_repository	This permits read-write access to repositories on private projects via Git-over-HTTP (but not using the API).
Remember to safely store your Personal Access Token, as we will use it in the next step to connect Renovate to GitLab for automated merge requests.

Initiating Renovate
Next, we will establish the renovate-config.js file to configure the file within the django-nv repository. Let’s input the following configuration file.

Please use the nano/vim text editor to modify the configuration file and create the renovate-config.js file.

Click anywhere to copy

module.exports = {
  endpoint: 'https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/api/v4/',
  token: '<your_gitlab_token>',
  platform: 'gitlab',
  onboardingConfig: {
    extends: ['config:recommended'],
  },
  repositories: ['root/django-nv'],
};
Please replace with the GitLab token you previously generated.

Subsequently, we should export the RENOVATE_CONFIG_FILE variable.


export RENOVATE_CONFIG_FILE="/django-nv/renovate-config.js"

Next, we will run the Renovate command to automatically update dependencies. However, it’s necessary that we specify a particular repository to scan the dependencies.

Attention

As we are using https for our endpoint, we must ignore the https restriction. Node.js, which Renovate is based on, refuses such requests by default to prevent “man-in-the-middle” attacks.

This is common if you’re operating a local GitLab instance via HTTPS with a self-signed certificate.

To address this issue, you can set the NODE_TLS_REJECT_UNAUTHORIZED environment variable to “0”


export NODE_TLS_REJECT_UNAUTHORIZED=0

Proceed by running the Renovate command.


renovate

Command Output
(node:3069) Warning: Setting the NODE_TLS_REJECT_UNAUTHORIZED environment variable to '0' makes TLS connections and HTTPS requests insecure by disabling certificate verification.
(Use `node --trace-warnings ...` to show where the warning was created)
 INFO: Repository started (repository=root/django-nv)
       "renovateVersion": "36.93.0"
 INFO: Branch created (repository=root/django-nv, branch=renovate/configure)
       "commit": "dda5672f52e5a08389c42aa944c225b1ae6f21df",
       "onboarding": true
 INFO: Dependency extraction complete (repository=root/django-nv, baseBranch=main)
       "stats": {
         "managers": {
           "dockerfile": {"fileCount": 1, "depCount": 1},
           "gitlabci": {"fileCount": 1, "depCount": 2},
           "npm": {"fileCount": 1, "depCount": 34},
           "pip_requirements": {"fileCount": 1, "depCount": 1}
         },
         "total": {"fileCount": 4, "depCount": 38}
       }
 WARN: GitHub token is required for some dependencies (repository=root/django-nv)
       "githubDeps": ["node"]
 INFO: Onboarding PR created (repository=root/django-nv)
       "pr": "Pull Request #1"
 INFO: Repository finished (repository=root/django-nv)
       "cloned": true,
       "durationMs": 15274
Perfect! let’s understand the output.

Action	Description
Repository started	Renovate bot began working on the /webapp repository.
Branch created	Renovate created a new branch named renovate/configure for updating dependencies.
Dependency extraction complete	Renovate completed checking the code, identifying 4 files, and detecting a total of 38 dependencies.
Onboarding PR created	Renovate generated a Pull Request containing suggested updates.
Repository finished	Renovate concluded the scan and PR creation process in approximately 14 seconds.
How To Run The Command In Docker?
Please navigate to the GitLab Merge Request (MR) page at https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/django-nv/-/merge_requests.

Within your repository, you’ll discover a New Merge Request, created by Renovate for the first time. Renovate will add a renovate.json file to establish the necessary schema.

renovate-configure-MR

On the following page, an overview will be displayed as you scroll down.

renovate-MR-overview

Proceed by clicking on the Merge button.

Great! Your repository has been updated successfully.

Go back to your terminal and execute a pull command in the django-nv directory to update its contents.


git pull origin main

Command Output
remote: Enumerating objects: 5, done.
remote: Counting objects: 100% (5/5), done.
remote: Compressing objects: 100% (4/4), done.
remote: Total 4 (delta 2), reused 0 (delta 0), pack-reused 0
Unpacking objects: 100% (4/4), 518 bytes | 518.00 KiB/s, done.
From gitlab-ce-kr6k1mdm:root/django-nv
 * branch            main       -> FETCH_HEAD
   0345cd1..d16fc4e  main       -> origin/main
Updating 0345cd1..d16fc4e
Fast-forward
 renovate.json | 6 ++++++
 1 file changed, 6 insertions(+)
 create mode 100644 renovate.json
Next, we will run the renovate command to check for any further updates.


renovate

Command Output
(node:3174) Warning: Setting the NODE_TLS_REJECT_UNAUTHORIZED environment variable to '0' makes TLS connections and HTTPS requests insecure by disabling certificate verification.
(Use `node --trace-warnings ...` to show where the warning was created)
 INFO: Repository started (repository=root/django-nv)
       "renovateVersion": "36.93.0"
 INFO: Dependency extraction complete (repository=root/django-nv, baseBranch=main)
       "stats": {
         "managers": {
           "dockerfile": {"fileCount": 1, "depCount": 1},
           "gitlabci": {"fileCount": 1, "depCount": 2},
           "npm": {"fileCount": 1, "depCount": 34},
           "pip_requirements": {"fileCount": 1, "depCount": 1}
         },
         "total": {"fileCount": 4, "depCount": 38}
       }
 WARN: GitHub token is required for some dependencies (repository=root/django-nv)
       "githubDeps": ["node"]
 INFO: Branch created (repository=root/django-nv, branch=renovate/django-3.x)
       "commitSha": "a07389acccc4f47a5621f7c57bb84b59af8df9e7"
 WARN: No github.com token has been configured. Skipping release notes retrieval (repository=root/django-nv, branch=renovate/django-3.x)
       "manager": "pip_requirements",
       "packageName": "Django",
       "sourceUrl": "https://github.com/django/django"
 INFO: PR created (repository=root/django-nv, branch=renovate/django-3.x)
       "pr": 2,
       "prTitle": "Update dependency Django to v3.2.21"
 INFO: Branch created (repository=root/django-nv, branch=renovate/consolidate-0.x)
       "commitSha": "8b71ac4869fc6e07588d8ed22185151f5e319cb1"
 WARN: No github.com token has been configured. Skipping release notes retrieval (repository=root/django-nv, branch=renovate/consolidate-0.x)
       "manager": "npm",
       "packageName": "consolidate",
       "sourceUrl": "https://github.com/ladjs/consolidate"
 INFO: PR created (repository=root/django-nv, branch=renovate/consolidate-0.x)
       "pr": 3,
       "prTitle": "Update dependency consolidate to ^0.16.0"
 INFO: Issue created (repository=root/django-nv)
 INFO: Repository finished (repository=root/django-nv)
       "cloned": true,
       "durationMs": 14928
In the following execution, as demonstrated above, Renovate scanned the dependency and detected that both the Consolidate and Django versions were outdated, each indicated in separate Merge Request.

Please navigate to the GitLab Merge Request (MR) page at https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/django-nv/-/merge_requests, where you will observe the update from Renovate.

dependency-fix-renovate-gitlab

Renovate, acting automatically, updated the Django version to v3.2.21 and Consolidate version to ^0.16.0 and then initiated a Merge Request in GitLab.

Open each Merge Request and repeat the previous action to accept the Pull Request from Renovate.

Great! You have successfully updated the django-nv repository.

Let’s now proceed to pull the new changes.


git pull origin main

Command Output
remote: Enumerating objects: 12, done.
remote: Counting objects: 100% (12/12), done.
remote: Compressing objects: 100% (4/4), done.
remote: Total 9 (delta 6), reused 7 (delta 4), pack-reused 0
Unpacking objects: 100% (9/9), 998 bytes | 499.00 KiB/s, done.
From gitlab-ce-kr6k1mdm:root/django-nv
 * branch            main       -> FETCH_HEAD
   d16fc4e..c6604d1  main       -> origin/main
Updating d16fc4e..c6604d1
Fast-forward
 package.json     | 2 +-
 requirements.txt | 2 +-
 2 files changed, 2 insertions(+), 2 deletions(-)
Next, let’s try to re-scan the repository.


renovate

Command Output
(node:3436) Warning: Setting the NODE_TLS_REJECT_UNAUTHORIZED environment variable to '0' makes TLS connections and HTTPS requests insecure by disabling certificate verification.
(Use `node --trace-warnings ...` to show where the warning was created)
 INFO: Repository started (repository=root/django-nv)
       "renovateVersion": "36.93.0"
 INFO: Dependency extraction complete (repository=root/django-nv, baseBranch=main)
       "stats": {
         "managers": {
           "dockerfile": {"fileCount": 1, "depCount": 1},
           "gitlabci": {"fileCount": 1, "depCount": 2},
           "npm": {"fileCount": 1, "depCount": 34},
           "pip_requirements": {"fileCount": 1, "depCount": 1}
         },
         "total": {"fileCount": 4, "depCount": 38}
       }
 WARN: GitHub token is required for some dependencies (repository=root/django-nv)
       "githubDeps": ["node"]
 INFO: Repository finished (repository=root/django-nv)
       "cloned": true,
       "durationMs": 9120
Fantastic! An automatic update has been successfully carried out using Renovate.

Note

For further learning, you might want to visit this page. Renovate creates a list here for manual updates of dependencies, where you can select the necessary dependencies using checkboxes.

https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/django-nv/-/issues/1.
```
```markdown
Addressing Issues via Renovate on GitLab
Conclusion
Renovate is an incredibly potent tool that allows developers to automate the often tedious task of maintaining software dependencies. By identifying and updating these dependencies continuously, Renovate ensures software projects are less prone to bugs, are more secure, and feature the latest functionality. It supports a wide range of languages and package managers, making it a versatile tool for repositories hosted on GitHub, GitLab, and other platforms.

That said, to maximize its benefit, it’s crucial to understand the full capabilities of Renovate, particularly how to customize and control its behaviors using different configuration options. Its seamless integration within existing workflows, and its collaborative features like creating pull requests for updates, allows for transparency and shared control over dependency updates within teams.

This tool, however, does require specific configurations and access to function correctly – including setup of environment variables for access tokens, and some awareness of rate limits when dealing with larger or multiple repositories.

References
Renovate Documentation: https://docs.renovatebot.com/
Renovate’s GitHub repository: https://github.com/renovatebot/renovate
Renovate - Dependency Management: https://medium.com/javarevisited/renovate-dependency-management-c1f0a9072e47
Renovate - Self-Hosted Gitlab setup: https://docs.renovatebot.com/examples/self-hosting/
```
```markdown
Addressing Issues via Dependabot on GitLab
Downloading The Source Code
We will do all the exercises locally first in DevSecOps-Box, so let’s start the exercise.

First, we need to download the source code of the project from our GitLab repository.

git clone git@gitlab-ce-kr6k1mdm:root/django-nv.git

Authentication Confirmation Required

If you encounter the following prompt while trying to clone GitLab, Please type yes and press Enter to proceed.

Command Output
The authenticity of host 'gitlab-ce-kr6k1mdm (10.x.x.x)' can't be established.
ECDSA key fingerprint is SHA256:U1W8wQm8tgHmuF/T0uf1jNVCzHyhqeJ4MmGG/K4XmcI.
Are you sure you want to continue connecting (yes/no/[fingerprint])?
Lets cd into the application code, so we can scan the app.

cd django-nv

We are now in the django-nv directory.

Let’s move to the next step.
```
```markdown
GitLab Setup
In this step, we will set up the gitlab access to integrate dependabot-gitlab to a gitlab repository.

Create A Personal Access Token (PAT) In GitLab Account
Let’s log into GitLab using the following details.

Name	Value
URL	https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/django-nv/-/blob/main/.gitlab-ci.yml
Username	root
Password	pdso-training
Next, we will create our Personal Access Token from GitLab at :

https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/-/profile/personal_access_tokens.
Provide the token name as renovatebot, select the following checkboxes:

api  
read_api  
read_repository  
write_repository  

After that click on Create personal access token button.

Note

For the intended operation, we need to assign the following functions to their respective categories:

Permission	Description  
api	This grants complete read/write access to the API, which includes all groups and projects, the container registry, and the package registry.  
read_api	This provides read access to the API, covering all groups and projects, the container registry, and the package registry.  
read_repository	This allows read-only access to repositories on private projects using Git-over-HTTP or the Repository Files API.  
write_repository	This permits read-write access to repositories on private projects via Git-over-HTTP (but not using the API).  

Remember to safely store your Personal Access Token, as we will use it in the next step to connect dependabot-gitlab to GitLab for automated merge requests.
```
```markdown
Configuring Dependabot-GitLab
Why We Are Using Dependabot-GitLab?

Dependabot is an essential tool in modern software development due to its features that help automate dependency updates and provide security alerts. Here’s why you should consider using Dependabot in GitLab:

Feature	Description  
Security	Dependabot helps keep your dependencies up-to-date, reducing the risk of using outdated packages that may have security vulnerabilities.  
Automation	Dependabot automatically submits pull requests anytime it discovers updates. This saves you time by sparing you from manually tracking and applying updates.  
New Features and Bug Fixes	Keeping your dependencies updated means you also get to benefit from any new features, improvements, or bug fixes that have been introduced in the newer versions.  
Compatibility Checks	Dependabot runs your test suite on each update it puts together to ensure that it doesn’t break your software. This provides additional confidence in the updates.  
Efficient Workflow Integration	Dependabot directly integrates with Git workflow, meaning you can process its pull requests the same way you handle others. This includes conducting code reviews, running tests against changes, and more.  
Configurability	You can configure Dependabot to suit your needs – control the frequency of updates, limit the number of open pull requests, ignore certain dependencies, and more.  

By frequent and automated updating, Dependabot helps to keep codebases healthy, secure, less buggy, and ensures they are using the most efficient implementations available.

We need to add the GitLab Personal Access Token variable (Go to Project (django.nv) → Settings → CI/CD → Variables → Expand).

You can also follow this URL https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/django-nv/-/settings/ci_cd

In the Variables section, you need to insert your GitLab Personal Access Token. This allows the dependabot-gitlab to carry out an automated Merge Request in the upcoming steps using the GitLab Personal Access Token.

Don’t forget to have a backup of your original Personal Access Token.

Click on the Add Variable button and you’ll be directed to the variable input box.

The tool Dependabot-GitLab employs the SETTINGS__GITLAB_ACCESS_TOKEN to source the Personal Access Token.

Then populate the SETTINGS__GITLAB_ACCESS_TOKEN field with the previously generated Personal Access Token value.

Ensure the Protect Variable box is unchecked for a more adaptable pipeline, then click add variable.

set-variable

Next, click the Add Variable button.

Now that we’ve successfully set up the token, let’s proceed to the next stage to implement dependabot-gitlab.
```
```markdown
Initiating Dependabot-GitLab For Resolving Issues
Configuring Dependabot-GitLab
We will set up the configuration file to ensure Dependabot can function correctly and scan the necessary packages.

Return to your devsecops-box terminal. We will start by creating a dependabot.yml file within our django-nv directory. Initially, we will create a .gitlab folder which will serve as the location for the dependabot.yml file.

mkdir /django-nv/.gitlab

Next, we will insert content into the dependabot.yml file.

cat > /django-nv/.gitlab/dependabot.yml <<EOF
version: 2
updates:
  - package-ecosystem: npm
    directory: /
EOF

The dependabot.yml configuration instructs Dependabot to manage and update npm package dependencies for a project:

Configuration	Description  
version: 2	Specifies the use of version 2 of the Dependabot configuration file, which is the latest version available.  
updates:	Indicates a list of updating actions to be performed by Dependabot.  
package-ecosystem: npm	Determines the ecosystem to be updated, specifically npm package dependencies.  
directory: /	Points Dependabot to the root directory of the project, indicating where to search for dependencies to update.  

You simply need to specify the package ecosystems and their corresponding configurations. Generally, you’ll need only one npm (which also observes yarn). However, here is a complete list of all potential ecosystems.

For other package ecosystems, please refer to this link: https://docs.github.com/en/code-security/dependabot/dependabot-version-updates/configuration-options-for-the-dependabot.yml-file#package-ecosystem

Note

The directory: is mandatory as it defines the location of package manifests (e.g., package.json) for each package manager.

Next, we should commit the changes to the Django-nv repository.

Initial Git Setup

To work with git repositories, we need to first setup a username and email.

We can use git config commands to set it up.

git config --global user.email "student@practical-devsecops.com"

git config --global user.name "student"

Having completed the initial setup, let’s proceed to push the new changes.

git add .gitlab/

git commit -m "add dependabot configuration file"

Command Output
[main 2368c30] add dependabot configuration file
 1 file changed, 7 insertions(+)
 create mode 100644 .gitlab/dependabot.yml

git push origin main

Command Output
Enumerating objects: 5, done.
Counting objects: 100% (5/5), done.
Delta compression using up to 32 threads
Compressing objects: 100% (3/3), done.
Writing objects: 100% (4/4), 397 bytes | 397.00 KiB/s, done.
Total 4 (delta 1), reused 0 (delta 0)
To gitlab-ce-kr6k1mdm:root/django-nv.git
   8c86acd..2368c30  main -> main
Excellent, we have successfully added the Dependabot-GitLab configuration file to the django-nv repository.

Running Dependabot-GitLab
Let’s navigate to the django-nv repository and update the .gitlab-ci.yml file to enable the execution of the dependabot tool, which scans and automatically generates merge requests for any forthcoming package changes and updates.

Follow the link below to update the .gitlab-ci.yml file.

https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/django-nv/-/blob/main/.gitlab-ci.yml

Please replace the existing content of the .gitlab-ci.yml file with the following.

Click anywhere to copy

image: docker:20.10

services:
  - docker:dind

stages:
  - build

.npm:
  stage: build
  image:
    name: docker.io/andrcuns/dependabot-gitlab:0.11.0
    entrypoint: [""]
  variables:
    GIT_STRATEGY: none
    PACKAGE_MANAGER: npm
    RAILS_ENV: production
    SETTINGS__GITLAB_URL: $CI_SERVER_URL
    SETTINGS__STANDALONE: "true"
  before_script:
    - cd /home/dependabot/app
  script:
    - bundle exec rake "dependabot:update[$CI_PROJECT_NAMESPACE/$CI_PROJECT_NAME,$PACKAGE_MANAGER,/]"

dependabot:
  extends: .npm
  when: manual

Note

Here are explanations for the variable functions:

Variable	Explanation  
GIT_STRATEGY	Specifies how jobs interact with the Git repository. Values: fetch for efficient fetching, clone for full clone.  
PACKAGE_MANAGER	Indicates the package manager responsible for managing dependencies.  
RAILS_ENV	Specifies the application’s operating environment. Adjust based on the current stage: “development” or “test”.  
SETTINGS_GITLAB_URL	Refers to the URL of the GitLab instance. Adjust based on your CI/CD setup.  
SETTINGS_STANDALONE	Specifies whether Dependabot operates as a standalone service. Value: false for operation as a GitHub App in the cloud.  

Commit the changes and then check out the running pipeline at https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/django-nv/-/pipelines

We have set the dependabot job’s pipeline to when: manual because we don’t want to command dependabot to perform its duties every time we run the pipeline. This allows us, as developers, to decide when we want to utilize dependabot in running the pipeline manually.

Proceed by launching the most recent pipeline and clicking the play button to run the dependabot job.

play-button

Let’s allow the pipeline to run. You can monitor the process by clicking on the dependabot job.

Note

Dependabot will scan and analyze all packages. It could take roughly 10 to 15 minutes for a full package update scan to be completed, as Dependabot will subsequently create a Merge Request.

runner-duration

Excellent! The pipeline has finished its run. You will now find new Merge Requests created by Dependabot in relation to the package update within the NPM ecosystem.

pipeline-result

Your final task is to merge all of the Merge Requests. Here is an example of how to do this:

Open one of the Merge Requests in https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/django-nv/-/merge_requests.

dependabot-MRs

Choose one of the Merge Requests and proceed to Merge in order to update the packages.

MRs-package

Click the Merge button and the package will be updated.

Apply similar actions to the other Merge Requests.

To verify that the latest changes in the django-nv repository are reflected, open a terminal and perform a git pull on the django-nv directory.

git pull origin main

Command Output
remote: Enumerating objects: 42, done.
remote: Counting objects: 100% (42/42), done.
remote: Compressing objects: 100% (17/17), done.
remote: Total 39 (delta 33), reused 26 (delta 22), pack-reused 0
Unpacking objects: 100% (39/39), 4.33 KiB | 443.00 KiB/s, done.
From gitlab-ce-kr6k1mdm:root/django-nv
 * branch            main       -> FETCH_HEAD
   3f92423..9a7b4be  main       -> origin/main
Updating 3f92423..9a7b4be
Fast-forward
 .gitlab-ci.yml | 25 ++++++++++++++++++-------
 package.json   |  8 ++++----
 2 files changed, 22 insertions(+), 11 deletions(-)
Excellent! You have updated the NPM packages within the package.json file using dependabot-gitlab.

Similar steps can also be carried out for other package ecosystems.
```
```markdown
Conclusion
Dependabot is a critical tool for managing the dependencies in programming projects, specifically those hosted on platforms like GitLab. It helps keep project dependencies up-to-date by automatically checking for updates and generating merge requests for each one found.

Automation reduces the overall workload for developers, streamlining the update process and maintaining the currency and security of the project. It also frees developers to focus on coding rather than the laborious task of keeping track of numerous dependencies.

Despite having more limited functionality on GitLab compared to GitHub, it’s possible to still reap significant benefits from Dependabot through correct setup and regular usage. Overall, Dependabot is an invaluable resource for any team or individual developing software on GitLab.
```
```markdown
Additional Resources
Dependabot-GitLab Repository: https://gitlab.com/dependabot-gitlab/dependabot  
Running Dependabot on GitLab: https://dependabot-gitlab.gitlab.io/dependabot/  
Setting Dependabot-GitLab environment: https://dependabot-gitlab.gitlab.io/dependabot/config/environment.html  
```
```markdown
Software Component Analysis Using OSV Scanner
Preparation
We will initially complete all tasks at a local level in DevSecOps-Box, so let’s initiate this exercise.

To start, we need to pull the project’s source code from our Git repository.

git clone https://gitlab.practical-devsecops.training/pdso/django.nv webapp

Command Output
Cloning into 'webapp'...
warning: redirecting to https://gitlab.practical-devsecops.training/pdso/django.nv.git/
remote: Enumerating objects: 299, done.
remote: Counting objects: 100% (71/71), done.
remote: Compressing objects: 100% (68/68), done.
remote: Total 299 (delta 34), reused 0 (delta 0), pack-reused 228
Receiving objects: 100% (299/299), 1.05 MiB | 29.16 MiB/s, done.
Resolving deltas: 100% (120/120), done.

Let’s navigate into the application code using the cd command to allow us to scan the app.

cd webapp

We now find ourselves in the webapp directory.

The forthcoming step necessitates us using the lock file to initiate the OSV Scanner.

Configuration Of package-lock.json File

To ensure a smooth installation of npm, let’s first update the package list.

apt update

The generation of the package-lock.json file from npm is now required. Prior to this however, the installation of npm is necessary.

apt install npm -y

Following this installation, we can proceed to formulate the package-lock.json file utilizing npm.

npm install

Significant Remark: Disregard error vulnerabilities, should they arise as illustrated below, because our primary need is the package-lock.json.

npm-output

What we necessitate is that the npm forms a package-lock.json file. The existence of this can be established by applying the ls command.

ls

Command Output
Dockerfile  bandit.ignore  manage.py     __package-lock.json__  requirements.txt  taskManager
README.md   fixtures       node_modules  package.json       run.sh

We now proceed to the subsequent step to initiate the installation of OSV Scanner.
```
```markdown
Getting Started With OSV Scanner
OSV (Open Source Vulnerability) Scanner is a highly efficient security tool systematically developed to examine and identify vulnerability points within open-source software and projects. OSV Scanner is a non-invasive scanner specifically engineered to unveil critical security issues. The primary resource that OSV Scanner leverages is the Software Bill of Materials (SBOM), a comprehensive inventory file that encapsulates detailed information, such as names and version specifications, about every open-source component used in a software project.

OSV Scanner can also process lock files from several programming languages like package-lock.json for JavaScript or requirements.txt for Python, which detail the specific versions of each dependency used in the project.

Reference: https://google.github.io/osv-scanner/

Installing OSV Scanner
First, we aim to install the OSV Scanner. Please utilize the command below to carry out the installation of the OSV Scanner.

wget -O /usr/bin/osv-scanner https://github.com/google/osv-scanner/releases/download/v1.4.0/osv-scanner_1.4.0_linux_amd64 && sudo chmod +x /usr/bin/osv-scanner

Command Output
--2023-09-26 05:41:17--  https://github.com/google/osv-scanner/releases/download/v1.4.0/osv-scanner_1.4.0_linux_amd64
Resolving github.com (github.com)... 140.82.112.3
Connecting to github.com (github.com)|140.82.112.3|:443... connected.
HTTP request sent, awaiting response... 302 Found
Location: https://objects.githubusercontent.com/github-production-release-asset-2e65be/565629124/1d1fae28-4d4c-4530-a4a5-08ecd13e6d97?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAIWNJYAX4CSVEH53A%2F20230926%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20230926T054117Z&X-Amz-Expires=300&X-Amz-Signature=bef6cfa0d9021577837008ebafde4732dfff0304057f0058e402817b47325268&X-Amz-SignedHeaders=host&actor_id=0&key_id=0&repo_id=565629124&response-content-disposition=attachment%3B%20filename%3Dosv-scanner_1.4.0_linux_amd64&response-content-type=application%2Foctet-stream [following]
--2023-09-26 05:41:17--  https://objects.githubusercontent.com/github-production-release-asset-2e65be/565629124/1d1fae28-4d4c-4530-a4a5-08ecd13e6d97?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAIWNJYAX4CSVEH53A%2F20230926%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20230926T054117Z&X-Amz-Expires=300&X-Amz-Signature=bef6cfa0d9021577837008ebafde4732dfff0304057f0058e402817b47325268&X-Amz-SignedHeaders=host&actor_id=0&key_id=0&repo_id=565629124&response-content-disposition=attachment%3B%20filename%3Dosv-scanner_1.4.0_linux_amd64&response-content-type=application%2Foctet-stream
Resolving objects.githubusercontent.com (objects.githubusercontent.com)... 185.199.109.133, 185.199.110.133, 185.199.108.133, ...
Connecting to objects.githubusercontent.com (objects.githubusercontent.com)|185.199.109.133|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 16457728 (16M) [application/octet-stream]
Saving to: ‘/usr/bin/osv-scanner’

/usr/bin/osv-scanner                       100%[=====================================================================================>]  15.70M  --.-KB/s    in 0.1s    

2023-09-26 05:41:17 (157 MB/s) - ‘/usr/bin/osv-scanner’ saved [16457728/16457728]

Great! The installation of the OSV Scanner has been successfully completed.

At this point, we should execute a command to assess the functionality and availability of the scanning feature in the OSV Scanner.

osv-scanner --help

Command Output
NAME:
   osv-scanner - scans various mediums for dependencies and matches it against the OSV database

USAGE:
   osv-scanner [global options] command [command options] [directory1 directory2...]

VERSION:
   1.4.0

COMMANDS:
   help, h  Shows a list of commands or help for one command

GLOBAL OPTIONS:
   --docker value, -D value [ --docker value, -D value ]      scan docker image with this name
   --lockfile value, -L value [ --lockfile value, -L value ]  scan package lockfile on this path
   --sbom value, -S value [ --sbom value, -S value ]          scan sbom file on this path
   --config value                                             set/override config file
   --format value, -f value                                   sets the output format (default: "table")
   --json                                                     sets output to json (deprecated, use --format json instead) (default: false)
   --output value                                             saves the result to the given file path
   --skip-git                                                 skip scanning git repositories (default: false)
   --recursive, -r                                            check subdirectories (default: false)
   --experimental-call-analysis                               attempt call analysis on code to detect only active vulnerabilities (default: false)
   --no-ignore                                                also scan files that would be ignored by .gitignore (default: false)
   --experimental-local-db                                    checks for vulnerabilities using local databases (default: false)
   --experimental-offline                                     checks for vulnerabilities using local databases that are already cached (default: false)
   --help, -h                                                 show help
   --version, -v                                              print the version

Let’s move to the next step.
```
```markdown
Running The Scanner
In this section, we will initiate the OSV Scanner to conduct an analysis of vulnerabilities based on the OSV Database. This section encompasses various methods for scanning, including:

Method Scanning	Description  
Elementary Scanning	This method involves a straightforward scan of the entire operating system or software against a database of known vulnerabilities. It aims to identify vulnerabilities without focusing on specific components.  
Targeted Scanning	In contrast to elementary scanning, targeted scanning focuses on specific components or areas of the system or software. It is used when the user suspects vulnerabilities in certain components or wants to limit the scope of the scanning process.  
Output Generation from Scanning	After completing the scan, an output in the form of a report is generated. This report details each identified vulnerability, including potential risks and impacts. It may also provide recommendations for addressing detected vulnerabilities.  
Local Database Scanning	Local database scanning, or offline mode scanning, allows the OSV scan to be conducted without an internet connection. It refers to a local database (Local DB) containing information on known vulnerabilities. This is useful in disconnected or secure environments.  
False Positive Analysis Scanning	False positive analysis scanning aims to mitigate inaccurately reported vulnerabilities known as false positives. It involves cross-referencing findings, coordinating data, and employing sophisticated algorithms to reduce the occurrence of false positives.  

We will now proceed step-by-step through each scanning process.

Basic Scanning
Firstly, we’ll employ general scanning. Let’s perform a scan on our webapp directory to examine what can be accessed by the OSV Scanner.

osv-scanner .

Command Output
Scanning dir .  
Scanning /django.nv/ at commit 74aa195986bee858fd61ad1f94f20c94bdd29aff  
Scanned /django.nv/package-lock.json file and found 1020 packages  
Scanned /django.nv/requirements.txt file and found 1 package  

╭─────────────────────────────────────┬──────┬───────────┬───────────────────┬─────────┬───────────────────╮
│ OSV URL                             │ CVSS │ ECOSYSTEM │ PACKAGE           │ VERSION │ SOURCE            │
├─────────────────────────────────────┼──────┼───────────┼───────────────────┼─────────┼───────────────────┤
│ https://osv.dev/GHSA-3v6h-hqm4-2rg6 │ 5.5  │ npm       │ adm-zip           │ 0.4.4   │ package-lock.json │
│ https://osv.dev/GHSA-pp7h-53gx-mx7r │ 6.5  │ npm       │ bl                │ 1.0.3   │ package-lock.json │
│ https://osv.dev/GHSA-pp7h-53gx-mx7r │ 6.5  │ npm       │ bl                │ 1.1.2   │ package-lock.json │
│ https://osv.dev/GHSA-832h-xg76-4gv6 │ 7.5  │ npm       │ brace-expansion   │ 1.1.6   │ package-lock.json │
│ https://osv.dev/GHSA-cwfw-4gq5-mrqx │      │ npm       │ braces            │ 1.8.5   │ package-lock.json │
│ https://osv.dev/GHSA-g95f-p29q-9xw4 │ 3.7  │ npm       │ braces            │ 1.8.5   │ package-lock.json │

......[SNIP]......

│ https://osv.dev/GHSA-v6rh-hp5x-86rv │ 7.3  │ PyPI      │ django            │ 3.0     │ requirements.txt  │
│ https://osv.dev/GHSA-vfq6-hq5r-27r6 │ 9.8  │ PyPI      │ django            │ 3.0     │ requirements.txt  │
│ https://osv.dev/GHSA-wpjr-j57x-wxfw │ 5.9  │ PyPI      │ django            │ 3.0     │ requirements.txt  │
│ https://osv.dev/PYSEC-2020-31       │      │           │                   │         │                   │
│ https://osv.dev/GHSA-xgxc-v2qg-chmh │ 5.3  │ PyPI      │ django            │ 3.0     │ requirements.txt  │
│ https://osv.dev/PYSEC-2021-6        │      │           │                   │         │                   │
│ https://osv.dev/GHSA-xpfp-f569-q3p2 │ 9.8  │ PyPI      │ django            │ 3.0     │ requirements.txt  │
╰─────────────────────────────────────┴──────┴───────────┴───────────────────┴─────────┴───────────────────╯

Impressive! We have completed the scan using the basic scanning method, and have obtained results from the package-lock.json and requirements.txt files.

Interpreting the Output
Let’s briefly decipher the meaning of the aforementioned results.

The table shared essentially portrays the outcomes of a security vulnerability examination using the Open Source Vulnerabilities (OSV) scanner. This tool allows you to pinpoint any known vulnerabilities in your software, with an emphasis on open source usage. Each column represents:

Field	Description  
OSV URL	These links provide comprehensive descriptions of each detected vulnerability. Following these links imparts further understanding over the threat and any available fixes or mitigations.  
CVSS	An acronym for Common Vulnerability Scoring System. It’s an industry-standard method for rating the severity of security vulnerabilities within software, with scores ranging from 0 to 10. More elevated scores imply more critical vulnerabilities.  
ECOSYSTEM	This indicates the software or runtime environment in which the package finds typical use, for instance, npm or PyPI.  
PACKAGE	This signifies the distinctive software package within that ecosystem that has a known vulnerability. Packages such as adm-zip, bl, brace-expansion in the npm ecosystem, or django in the PyPI ecosystem, feature in the table.  
VERSION	This denotes the exact version of the package comprising the vulnerability. It’s possible that only certain versions of a package are affected.  
SOURCE	This column informs of the file location of the vulnerable package. In your case, vulnerabilities were detected in the package-lock.json file for npm packages and requirements.txt file for PyPI packages.  

The outcomes revealed numerous dependencies that are utilizing versions possessing known security vulnerabilities. The information can be harnessed to update your dependencies as needed— either manually, or by utilizing a tool that automatically updates your packages to versions deemed secure.

Nice! We will try to address some of the issues identified in the Fix the Issues Reported by Safety Tool exercises.
```
```markdown
Software Component Analysis Using OSV-Scalibr
In this scenario, you will explore how to utilize OSV-Scalibr (Software Composition Analysis Library) to extract software inventory data and detect vulnerabilities in your applications.

You will learn how to:  
- Install and configure OSV-Scalibr  
- Scan local files and container images  
- Generate SPDX reports  
- Understand and analyze scan results  

Note

We have already set up the necessary tools including Docker on the DevSecOps-Box machine, you just have to use them for the exercise.

Always remember that the DevSecOps Box is an immutable machine, and thus it does not save the previous state after you close or reload the browser.
```
```markdown
Introduction to OSV-Scalibr
OSV-Scalibr (Software Composition Analysis Library) is an extensible file system scanner developed by Google that focuses on extracting software inventory data and detecting vulnerabilities. While OSV-Scanner primarily focuses on vulnerability scanning using the OSV database, OSV-Scalibr provides additional capabilities:

Key features of OSV-Scalibr:
- Extracts installed language packages and software inventory data  
- Performs vulnerability scanning on local machines and container images  
- Supports custom plugins for inventory extraction and vulnerability detection  
- Can be used as both a standalone binary or as a library in custom applications  
- Provides special features like RPM extraction support and MacOS Application extraction  

The main differences between OSV-Scanner and OSV-Scalibr:

- **Focus**: OSV-Scanner focuses primarily on vulnerability scanning, while OSV-Scalibr focuses on both inventory extraction and vulnerability detection  
- **Extensibility**: OSV-Scalibr is designed to be highly extensible through custom plugins  
- **Integration Options**: OSV-Scalibr can be used as a library in other applications, making it more flexible for custom implementations  

Let’s move to the next step.
```
```markdown
Installing OSV-SCALIBR
After we know the difference between OSV-Scanner and OSV-Scalibr, let’s start by getting our environment ready. First, we’ll clone a sample application that we can scan with OSV-Scalibr in this case we will use django.nv repository.

git clone https://gitlab.practical-devsecops.training/pdso/django.nv webapp

Command Output
Cloning into 'webapp'...
warning: redirecting to https://gitlab.practical-devsecops.training/pdso/django.nv.git/
remote: Enumerating objects: 299, done.
remote: Counting objects: 100% (71/71), done.
remote: Compressing objects: 100% (68/68), done.
remote: Total 299 (delta 34), reused 0 (delta 0), pack-reused 228
Receiving objects: 100% (299/299), 1.05 MiB | 29.16 MiB/s, done.
Resolving deltas: 100% (120/120), done.

Let’s move into the application directory:

cd webapp

Now we need to install OSV-Scalibr. Since it’s written in Go, we’ll first ensure Go is installed:

curl -OL https://dl.google.com/go/go1.24.0.linux-amd64.tar.gz  
tar -C /usr/local -xvf go1.24.0.linux-amd64.tar.gz  
export PATH=$PATH:/usr/local/go/bin  

Let’s verify Go is installed correctly:

go version

Command Output
go version go1.24.0 linux/amd64

Now after all the dependencies are installed, we can install OSV-Scalibr:

export GOPROXY=https://goproxy.cn,direct  
go install github.com/google/osv-scalibr/binary/scalibr@v0.2.0  
export PATH=$PATH:$(go env GOPATH)/bin  

We have successfully installed the OSV-Scalibr. Let’s explore what osv scalibr can do by running the help command:

scalibr --help

Command Output
Usage of scalibr:  
  -cdx-authors string  
        The 'authors' field for the output CDX document. Format is --cdx-authors=author1,author2  
  -cdx-component-name string  
        The 'metadata.component.name' field for the output CDX document  
  -cdx-component-version string  
        The 'metadata.component.version' field for the output CDX document  
  -detectors value  
        Comma-separated list of detectors plugins to run  
  -explicit-extractors  
        If set, the program will exit with an error if not all extractors required by enabled detectors are explicitly enabled.  
  -extractors value  
        Comma-separated list of extractor plugins to run  
  -filter-by-capabilities  
        If set, plugins whose requirements (network access, OS, etc.) aren't satisfied by the scanning environment will be silently disabled instead of throwing a validation error. (default true)  
  -govulncheck-db string  
        Path to the offline DB for the govulncheck detectors to use. Leave empty to run the detectors in online mode.  
  -image-platform string  
        The platform of the remote image to scan. If not specified, the platform of the client is used. Format is os/arch (e.g. linux/arm64)  
  -o value  
        The path of the scanner outputs in various formats, e.g. -o textproto=result.textproto -o spdx23-json=result.spdx.json -o cdx-json=result.cyclonedx.json  
  -remote-image string  
        The remote image to scan. If specified, SCALIBR pulls and scans this image instead of the local filesystem.  
  -result string  
        The path of the output scan result file  
  -root string  
        The root dir used by detectors and by file walking during extraction (e.g.: "/", "c:\" or ".")  
  -skip-dir-glob string  
        If the glob matches a directory, it will be skipped. The glob is matched against the absolute file path.  
  -skip-dir-regex string  
        If the regex matches a directory, it will be skipped. The regex is matched against the absolute file path.  
  -skip-dirs value  
        Comma-separated list of file paths to avoid traversing  
  -spdx-creators string  
        The 'creators' field for the output SPDX document. Format is --spdx-creators=creatortype1:creator1,creatortype2:creator2  
  -spdx-document-name string  
        The 'name' field for the output SPDX document  
  -spdx-document-namespace string  
        The 'documentNamespace' field for the output SPDX document  
  -verbose  
        Enable this to print debug logs  
  -windows-all-drives  
        Scan all drives on Windows  

As you can see, there are a lot of options available. Let’s move on to learning how to use it.

Let’s move to the next step.
```
```markdown
Basic Scanning with OSV-Scalibr
OSV-Scalibr is an extensible file system scanner that can:

- Extract software inventory data (installed packages)  
- Detect vulnerabilities  
- Generate SPDX reports  
- Scan both local files and container images  

Let’s start with a basic scan of our webapp directory:

scalibr --result=result.textproto

Command Output
2025/02/04 05:20:32 End status: 18001 dirs visited, 134705 inodes visited, 1801 Extract calls, 4.618332981s elapsed  
2025/02/04 05:20:32 Scan status: SUCCEEDED  
2025/02/04 05:20:32 Found 4476 software inventories, 0 security findings  
2025/02/04 05:20:32 Writing scan results to result.textproto  
2025/02/04 05:20:32 Marshaled result proto has 1858954 bytes  

The command above will generate a scan result in Protocol Buffer text format (a human-readable serialization format used by Google’s Protocol Buffers for structured data).

Let’s examine the results:

cat result.textproto

Command Output
.... [SNIP] ....

inventories:  {
  name:  "zwitch"
  version:  "1.0.5"
  source_code:  {}
  purl:  {
    purl:  "pkg:npm/zwitch@1.0.5"
    type:  "npm"
    name:  "zwitch"
    version:  "1.0.5"
  }
  ecosystem:  "npm"
  locations:  "usr/local/lib/python3.10/dist-packages/ansible_collections/grafana/grafana/yarn.lock"
  extractor:  "javascript/yarnlock"
}

The output of the scanning will show detected packages and any associated vulnerabilities as the above result shows.

---

Scanning a Container Image
The tool is not only able to scan a local directory, but it can also scan a container image. Let’s give it a try by scanning the Alpine Linux image.

scalibr --result=alpine-scan.textproto --remote-image=alpine:latest

Command Output
2025/02/04 05:22:31 End status: 45 dirs visited, 464 inodes visited, 22 Extract calls, 4.162351ms elapsed  
2025/02/04 05:22:31 Scan status: SUCCEEDED  
2025/02/04 05:22:31 Found 15 software inventories, 0 security findings  
2025/02/04 05:22:31 Writing scan results to alpine-scan.textproto  

When you run the scanner, it automatically performs these steps:
- Downloads the container image from the registry  
- Extracts all the contents from the image layers  
- Identifies and catalogs all installed packages and dependencies  
- Checks each package against vulnerability databases for known security issues  

Let’s examine these results:

cat alpine-scan.textproto

Command Output
    qualifiers:  {
      key:  "origin"
      value:  "zlib"
    }
  }
  ecosystem:  "Alpine:v3.21"
  locations:  "lib/apk/db/installed"
  extractor:  "os/apk"
  apk_metadata:  {
    package_name:  "zlib"
    origin_name:  "zlib"
    os_id:  "alpine"
    os_version_id:  "3.21.2"
    maintainer:  "Natanael Copa <ncopa@alpinelinux.org>"
    architecture:  "x86_64"
    license:  "Zlib"
  }
}

Let’s move to the next step.
```
```markdown
Generating SPDX Reports
Software Package Data Exchange (SPDX) is an open standard for documenting software bill of materials (SBOM). It provides a standardized way to share information about software components, their licenses, and dependencies.

OSV-Scalibr can generate SPDX reports in various formats, helping you:

- Track software components and dependencies  
- Document license information  
- Share software inventory data with other tools  
- Meet compliance requirements  

Let’s generate an SPDX report in JSON format:

scalibr -o spdx23-json=webapp.spdx.json

Command Output
2025/02/04 05:26:01 End status: 18046 dirs visited, 135172 inodes visited, 1822 Extract calls, 1.841682496s elapsed  
2025/02/04 05:26:01 Scan status: SUCCEEDED  
2025/02/04 05:26:01 Found 4476 software inventories, 0 security findings  
2025/02/04 05:26:01 Writing scan results to webapp.spdx.json  

Not only we can generate SPDX reports in JSON format, but also we can customize the SPDX document metadata.

Let’s customize the SPDX document metadata:

scalibr \
  --spdx-document-name="Django.nv Scan" \
  --spdx-document-namespace="https://devsecops.training/django-nv" \
  --spdx-creators=Organization:DevSecOps-Training \
  -o spdx23-json=webapp-custom.spdx.json

Command Output
2025/02/04 05:29:01 End status: 18046 dirs visited, 135172 inodes visited, 1822 Extract calls, 1.723608258s elapsed  
2025/02/04 05:29:01 Scan status: SUCCEEDED  
2025/02/04 05:29:01 Found 4476 software inventories, 0 security findings  
2025/02/04 05:29:01 Writing scan results to webapp-custom.spdx.json  

After the scanning is completed, let’s examine our SPDX report:

cat webapp-custom.spdx.json

Command Output
{
  "spdxVersion": "SPDX-2.3",
  "dataLicense": "CC0-1.0",
  "SPDXID": "SPDXRef-DOCUMENT",
  "name": "Django.nv Scan",
  "documentNamespace": "https://devsecops.training/django-nv",
  "creationInfo": {
    "creators": [
      "Tool: SCALIBR",
      "Organization: DevSecOps-Training"
    ],
    "created": "2025-02-04T05:29:01Z"
  },
  "packages": [

... [SNIP] ...

    {
      "spdxElementId": "SPDXRef-Package-main-f9956188-bd71-4b43-b89a-3c93f7ff4c4b",
      "relatedSpdxElement": "SPDXRef-Package-zwitch-5a79f10e-f728-4b01-aaca-c07f35444a63",
      "relationshipType": "CONTAINS"
    },
    {
      "spdxElementId": "SPDXRef-Package-zwitch-5a79f10e-f728-4b01-aaca-c07f35444a63",
      "relatedSpdxElement": "NOASSERTION",
      "relationshipType": "CONTAINS"
    }
  ]
}

The SPDX report that we generated before will include:

- Package information  
- License information  
- File inventory  
- Relationships between components  

We can also generate YAML format if we want:

scalibr -o spdx23-yaml=webapp.spdx.yaml

Command Output
2025/02/04 05:31:20 End status: 18049 dirs visited, 135176 inodes visited, 1822 Extract calls, 1.634190416s elapsed  
2025/02/04 05:31:20 Scan status: SUCCEEDED  
2025/02/04 05:31:20 Found 4476 software inventories, 0 security findings  
2025/02/04 05:31:20 Writing scan results to webapp.spdx.yaml  

Or the traditional tag-value format:

scalibr -o spdx23-tag-value=webapp.spdx

Command Output
2025/02/04 05:31:50 End status: 18049 dirs visited, 135177 inodes visited, 1822 Extract calls, 1.67123317s elapsed  
2025/02/04 05:31:50 Scan status: SUCCEEDED  
2025/02/04 05:31:50 Found 4476 software inventories, 0 security findings  
2025/02/04 05:31:50 Writing scan results to webapp.spdx  

Let’s examine the SPDX report:

cat webapp.spdx

Command Output
Relationship: SPDXRef-SPDXRef-Package-main-76789389-d48c-4f3f-a9ee-ebdbd5c35067 CONTAINS SPDXRef-SPDXRef-Package-zlib1g-dev-6e4efd8a-119e-487f-a604-24e0ce2a6f45  
Relationship: SPDXRef-SPDXRef-Package-zlib1g-dev-6e4efd8a-119e-487f-a604-24e0ce2a6f45 CONTAINS NOASSERTION  
Relationship: SPDXRef-SPDXRef-Package-main-76789389-d48c-4f3f-a9ee-ebdbd5c35067 CONTAINS SPDXRef-SPDXRef-Package-zwitch-aadf5f2d-d10a-4f2a-b6cb-0c6015cbbb69  
Relationship: SPDXRef-SPDXRef-Package-zwitch-aadf5f2d-d10a-4f2a-b6cb-0c6015cbbb69 CONTAINS NOASSERTION  

These SPDX reports can be used to:

- Document software components  
- Track dependencies  
- Share software inventory data  
- Comply with license obligations  

Let’s move to the next step.
```
```markdown
Working with Plugins
OSV-Scalibr is designed to be extensible through plugins. It comes with built-in plugins for inventory extraction and vulnerability detection.

Let’s look at running specific extractors:

scalibr --extractors=python,os --result=specific.textproto

Command Output
2025/02/04 05:59:06 Found 743 software inventories, 0 security findings  
2025/02/04 05:59:06 Writing scan results to specific.textproto  
2025/02/04 05:59:06 Marshaled result proto has 450111 bytes  

The above command will run only the Python and OS package extractors. Common extractors include:

- python: For Python packages and dependencies  
- javascript: For JavaScript/Node.js packages  
- java: For Java packages and Maven dependencies  
- go: For Golang modules  
- os: For operating system packages  

---

Note  
Not only can we specify extractors, we can also specify detectors by using the –detectors flag:

scalibr --detectors=cis --result=cis-scan.textproto

Command Output
2025/02/04 06:02:04 End status: 18049 dirs visited, 135179 inodes visited, 1822 Extract calls, 1.874597188s elapsed  
2025/02/04 06:02:04 Scan status: SUCCEEDED  
2025/02/04 06:02:04 Found 4476 software inventories, 0 security findings  
2025/02/04 06:02:04 Writing scan results to cis-scan.textproto  
2025/02/04 06:02:04 Marshaled result proto has 1859064 bytes  

The above command will run the CIS benchmark detector.

---

Understanding the Results
Let’s examine our scan results in more detail for the specific.textproto file:

cat specific.textproto

Command Output
....[SNIP]....

  ecosystem:  "Ubuntu:22.04"
  locations:  "var/lib/dpkg/status"

  extractor:  "os/dpkg"
  dpkg_metadata:  {
    package_name:  "zlib1g-dev"
    source_name:  "zlib"
    package_version:  "1:1.2.11.dfsg-2ubuntu9.2"
    os_id:  "ubuntu"
    os_version_codename:  "jammy"
    os_version_id:  "22.04"
    maintainer:  "Ubuntu Developers <ubuntu-devel-discuss@lists.ubuntu.com>"
    architecture:  "amd64"
    status:  "install ok installed"
  }
}

---

Whats about the cis-scan.textproto file?

cat cis-scan.textproto

Command Output
....[SNIP]....
inventories:  {
  name:  "zwitch"
  version:  "1.0.5"
  source_code:  {}
  purl:  {
    purl:  "pkg:npm/zwitch@1.0.5"
    type:  "npm"
    name:  "zwitch"
    version:  "1.0.5"
  }
  ecosystem:  "npm"
  locations:  "usr/local/lib/python3.10/dist-packages/ansible_collections/grafana/grafana/yarn.lock"
  extractor:  "javascript/yarnlock"
}

The scan provides a comprehensive inventory of all installed packages across different ecosystems (Node.js, Python, Java, Go, OS packages) and their locations in the system.

---

Handling Multiple Scans
We can scan multiple directories if you have a large number of files to scan:

scalibr --result=multi.textproto dir1/ dir2/ dir3/

---

Note  
Let’s move on to the next step.
```
```markdown
Conclusion
From the experiments that we have done, we can conclude that OSV-Scalibr is a powerful and flexible scanning tool that offers several key advantages:

### Comprehensive Scanning Capabilities
- Can scan both local filesystems and container images  
- Detects packages across multiple ecosystems (Python, Node.js, Java, Go, etc.)  
- Provides detailed metadata about discovered packages  
- Supports various output formats including SPDX and Protocol Buffers  

### Extensible Architecture
- Supports plugin-based extractors for different package types  
- Allows custom detectors for specific security checks  
- Can be used as both a standalone tool and a library  
- Enables integration into larger security workflows  

### Standards Compliance
- Generates standardized SPDX reports  
- Supports multiple SPDX formats (JSON, YAML, tag-value)  
- Provides detailed package URLs (purls)  
- Maintains proper software bill of materials (SBOM)  

### Enterprise-Ready Features
- Handles large-scale scanning efficiently  
- Supports recursive directory scanning  
- Provides detailed inventory reports  
- Enables customization of scan scope and output  

---

These capabilities make OSV-Scalibr an excellent choice for organizations needing to:

- Track software dependencies  
- Generate compliance documentation  
- Perform security assessments  
- Maintain software inventories  
- Create standardized SBOMs  

The tool’s flexibility and extensibility ensure it can adapt to various use cases while maintaining compatibility with industry standards.
```
```markdown
Fix the Issues Reported by SCA Tools
Preparing the Environment for Security Analysis
To begin dependency scanning, we first need to download the source code of an application.

Let’s use git clone to download the django.nv app’s source code into a directory named webapp.

git clone https://gitlab.practical-devsecops.training/pdso/django.nv webapp

Command Output
Cloning into 'webapp'...
warning: redirecting to https://gitlab.practical-devsecops.training/pdso/django.nv.git/
remote: Enumerating objects: 299, done.
remote: Counting objects: 100% (71/71), done.
remote: Compressing objects: 100% (68/68), done.
remote: Total 299 (delta 34), reused 0 (delta 0), pack-reused 228
Receiving objects: 100% (299/299), 1.05 MiB | 29.16 MiB/s, done.
Resolving deltas: 100% (120/120), done.

After successfully cloning the project into our machine, we need to move into the project’s directory.

cd webapp

Command Output
/webapp# 

With the source code successfully cloned into the DevSecOps-Box and the webapp directory ready, the next step is to prepare the environment for a thorough security analysis. This involves setting up the package-lock.json file, which is essential for effectively using npm packages and tools.

In the next step, we will use the package-lock.json file to run the OSV Scanner.

---

Installing npm and Generating package-lock.json
To begin, we need to make sure that our package list is up-to-date and that npm, the package manager for JavaScript, is installed in our environment. Let’s enter the following commands to accomplish this:

Update the package list to ensure we have the latest versions of available packages and can install new ones without any issues.

apt update

Install npm, which is crucial for managing JavaScript packages and will be used to generate the package-lock.json file.

apt install npm -y

With npm installed, the next step was to generate the package-lock.json file. The file is critical as it locks down the versions of npm package dependencies for the project, ensuring consistent installations across environments.

Let’s input the command to generate the file:

npm install

During the process, you might see some warnings about vulnerabilities. These warnings could be overlooked for the moment because the main aim is to get the package-lock.json file ready. An example of what they might see is illustrated below:

npm-output

After running npm install, let’s input the ls command to list the files in the directory and confirm that the package-lock.json file has been successfully created:

ls

Command Output
Dockerfile  bandit.ignore  manage.py     __package-lock.json__  requirements.txt  taskManager
README.md   fixtures       node_modules  package.json       run.sh

After checking with ls, we can see the package-lock.json file listed among others, which means the process was successful. Now, we are ready for the next step: installing a tool called the OSV Scanner to find and fix any hidden security issues in the project.

With the package-lock.json file ready and a clear plan in place, we are one step closer to making the project secure.
```
```markdown
Getting Started With OSV Scanner
Imagine a team of developers eager to ensure their project was safe from any lurking digital threats. They learned about a tool that could help them, known as the OSV (Open Source Vulnerability) Scanner. The tool was like a digital detective that could sift through their project’s code, identifying any hidden vulnerabilities by examining a list of all the open-source components they were using, also known as the Software Bill of Materials (SBOM). For their JavaScript or Python projects, it could also scan specific files like package-lock.json or requirements.txt to check the versions of the dependencies they had employed.

Reference: https://google.github.io/osv-scanner/

---

Setting Up the OSV Scanner
First, we need to prepare the OSV scanner for use. Let us enter the command:

wget -O /usr/bin/osv-scanner https://github.com/google/osv-scanner/releases/download/v1.4.0/osv-scanner_1.4.0_linux_amd64 && sudo chmod +x /usr/bin/osv-scanner

Command Output

...[SNIP]...

/usr/bin/osv-sca 100%[=========>]  15.70M  62.9MB/s    in 0.2s    

2024-05-25 09:49:41 (62.9 MB/s) - ‘/usr/bin/osv-scanner’ saved [16457728/16457728]

As we executed the command, we received a confirmation message indicating that the OSV Scanner was being downloaded.

Now that the OSV Scanner is installed, let’s type the following command in the terminal to view the available options:

osv-scanner --help

Command Output
NAME:
   osv-scanner - scans various mediums for dependencies and matches it against the OSV database

USAGE:
   osv-scanner [global options] command [command options] [directory1 directory2...]

VERSION:
   1.4.0

COMMANDS:
   help, h  Shows a list of commands or help for one command

GLOBAL OPTIONS:
   --docker value, -D value [ --docker value, -D value ]      scan docker image with this name  
   --lockfile value, -L value [ --lockfile value, -L value ]  scan package lockfile on this path  
   --sbom value, -S value [ --sbom value, -S value ]          scan sbom file on this path  
   --config value                                             set/override config file  
   --format value, -f value                                   sets the output format (default: "table")  
   --json                                                     sets output to json (deprecated, use --format json instead) (default: false)  
   --output value                                             saves the result to the given file path  
   --skip-git                                                 skip scanning git repositories (default: false)  
   --recursive, -r                                            check subdirectories (default: false)  
   --experimental-call-analysis                               attempt call analysis on code to detect only active vulnerabilities (default: false)  
   --no-ignore                                                also scan files that would be ignored by .gitignore (default: false)  
   --experimental-local-db                                    checks for vulnerabilities using local databases (default: false)  
   --experimental-offline                                     checks for vulnerabilities using local databases that are already cached (default: false)  
   --help, -h                                                 show help  
   --version, -v                                              print the version  

The help command displayed various commands demonstrating how to utilize the OSV Scanner effectively. It provided instructions on scanning projects using Docker images, lock files for JavaScript or Python projects, and more.

---

Let’s move on to the next step.
```
```markdown
Scanning and Fixing requirements.txt
With the required tools installed, our goal is to discover what insights the tool can provide about our project.

---

### Specific Scanning requirements.txt
To ensure our project’s safety and address any potential issues, we will utilize a tool called the OSV Scanner. Our initial step involves examining the requirements.txt file for any issues. We can accomplish this by executing the following command:

osv-scanner -L requirements.txt

The scanner quickly got to work and highlighted security issues with the Django version we were using. The screen displayed several warnings along with links for more details.

Command Output
Scanned /webapp/requirements.txt file and found 1 package
╭─────────────────────────────────────┬──────┬───────────┬─────────┬─────────┬──────────────────╮
│ OSV URL                             │ CVSS │ ECOSYSTEM │ PACKAGE │ VERSION │ SOURCE           │
├─────────────────────────────────────┼──────┼───────────┼─────────┼─────────┼──────────────────┤
│ https://osv.dev/GHSA-2m34-jcjv-45xf │ 6.1  │ PyPI      │ django  │ 3.0     │ requirements.txt │
│ https://osv.dev/PYSEC-2020-32       │      │           │         │         │                  │
│ https://osv.dev/GHSA-3gh2-xw74-jmcw │ 8.8  │ PyPI      │ django  │ 3.0     │ requirements.txt │
│ https://osv.dev/PYSEC-2020-36       │      │           │         │         │                  │
│ https://osv.dev/GHSA-68w8-qjq3-2gfm │ 4.9  │ PyPI      │ django  │ 3.0     │ requirements.txt │
│ https://osv.dev/PYSEC-2021-98       │      │           │         │         │                  │
│ https://osv.dev/GHSA-fr28-569j-53c4 │ 7.5  │ PyPI      │ django  │ 3.0     │ requirements.txt │
│ https://osv.dev/PYSEC-2020-34       │      │           │         │         │                  │
│ https://osv.dev/GHSA-fvgf-6h6h-3322 │ 5.3  │ PyPI      │ django  │ 3.0     │ requirements.txt │
│ https://osv.dev/PYSEC-2021-9        │      │           │         │         │                  │
│ https://osv.dev/GHSA-hmr4-m2h5-33qx │ 9.8  │ PyPI      │ django  │ 3.0     │ requirements.txt │
│ https://osv.dev/PYSEC-2020-35       │      │           │         │         │                  │
│ https://osv.dev/GHSA-m6gj-h9gm-gw44 │ 7.5  │ PyPI      │ django  │ 3.0     │ requirements.txt │
│ https://osv.dev/PYSEC-2020-33       │      │           │         │         │                  │
│ https://osv.dev/GHSA-p99v-5w3c-jqq9 │ 7.5  │ PyPI      │ django  │ 3.0     │ requirements.txt │
│ https://osv.dev/PYSEC-2021-99       │      │           │         │         │                  │
│ https://osv.dev/GHSA-rxjp-mfm9-w4wr │ 7.5  │ PyPI      │ django  │ 3.0     │ requirements.txt │
│ https://osv.dev/GHSA-v6rh-hp5x-86rv │ 7.3  │ PyPI      │ django  │ 3.0     │ requirements.txt │
│ https://osv.dev/GHSA-vfq6-hq5r-27r6 │ 9.8  │ PyPI      │ django  │ 3.0     │ requirements.txt │
│ https://osv.dev/GHSA-wpjr-j57x-wxfw │ 5.9  │ PyPI      │ django  │ 3.0     │ requirements.txt │
│ https://osv.dev/PYSEC-2020-31       │      │           │         │         │                  │
│ https://osv.dev/GHSA-xgxc-v2qg-chmh │ 5.3  │ PyPI      │ django  │ 3.0     │ requirements.txt │
│ https://osv.dev/PYSEC-2021-6        │      │           │         │         │                  │
│ https://osv.dev/GHSA-xpfp-f569-q3p2 │ 9.8  │ PyPI      │ django  │ 3.0     │ requirements.txt │
│ https://osv.dev/GHSA-xxj9-f6rv-m3x4 │ 5.9  │ PyPI      │ django  │ 3.0     │ requirements.txt │
╰─────────────────────────────────────┴──────┴───────────┴─────────┴─────────┴──────────────────╯

---

### Resolve Issue in requirements.txt
After noticing the warnings, we realized we should upgrade Django to a newer, safer version. To do this, we decided to modify our requirements.txt file to use Django 5.0 instead. To update Django, we need to enter the following command:

cat > requirements.txt <<EOL
Django==5.0
EOL

To ensure that the requirements are already updated, we need to check the file:

cat requirements.txt

Command Output
Django==5.0

And it was! The new version of Django was now in their list.

---

### Re-Scanning to Verify Fix
With the update made, we need to re-scan again using OSV scanner to make sure our project was safe:

osv-scanner -L requirements.txt

Command Output
Scanned /webapp/requirements.txt file and found 1 package
╭─────────────────────────────────────┬──────┬───────────┬─────────┬─────────┬──────────────────╮
│ OSV URL                             │ CVSS │ ECOSYSTEM │ PACKAGE │ VERSION │ SOURCE           │
├─────────────────────────────────────┼──────┼───────────┼─────────┼─────────┼──────────────────┤
│ https://osv.dev/GHSA-vm8q-m57g-pff3 │      │ PyPI      │ django  │ 5.0     │ requirements.txt │
│ https://osv.dev/PYSEC-2024-47       │      │           │         │         │                  │
│ https://osv.dev/GHSA-xxj9-f6rv-m3x4 │ 5.9  │ PyPI      │ django  │ 5.0     │ requirements.txt │
│ https://osv.dev/PYSEC-2024-28       │      │           │         │         │                  │
╰─────────────────────────────────────┴──────┴───────────┴─────────┴─────────┴──────────────────╯

---

### Variable Scan Results
Your results might slightly vary because of the dynamic landscape of changing vulnerabilities and security updates.

✅ Voila, the result is much fewer than the scanning before — we successfully reduced the vulnerabilities here.

---

Let’s attempt to fix the npm issue on the next page.
```
```markdown
Scanning and Fixing NPM
After successfully addressing the issue in the requirements.txt file, our next task is to tackle npm vulnerabilities. We have opted to utilize the OSV Scanner tool to inspect our package-lock.json file for any security concerns.

---

### Specific Scanning NPM
To enhance the safety of our project further, we have decided to address not only the requirements.txt but also the NPM issues. Our initial step was to inspect the package-lock.json file for any potential problems.

To perform this action, please input the following command:

osv-scanner -L package-lock.json

Just like that, the scanner got to work and found some security problems with a few packages we were using. It showed us lots of warnings and links to more details.

Command Output
Scanned /webapp/requirements.txt file and found 1 package  
Scanned /webapp/package-lock.json file and found 1045 packages  
╭─────────────────────────────────────┬──────┬───────────┬───────────────────┬─────────┬───────────────────╮
│ OSV URL                             │ CVSS │ ECOSYSTEM │ PACKAGE           │ VERSION │ SOURCE            │
├─────────────────────────────────────┼──────┼───────────┼───────────────────┼─────────┼───────────────────┤
│ https://osv.dev/GHSA-3v6h-hqm4-2rg6 │ 5.5  │ npm       │ adm-zip           │ 0.4.4   │ package-lock.json │
│ https://osv.dev/GHSA-67hx-6x53-jw92 │ 9.3  │ npm       │ babel-traverse    │ 6.11.4  │ package-lock.json │
│ https://osv.dev/GHSA-pp7h-53gx-mx7r │ 6.5  │ npm       │ bl                │ 1.0.3   │ package-lock.json │
│ https://osv.dev/GHSA-pp7h-53gx-mx7r │ 6.5  │ npm       │ bl                │ 1.1.2   │ package-lock.json │
│ https://osv.dev/GHSA-832h-xg76-4gv6 │ 7.5  │ npm       │ brace-expansion   │ 1.1.6   │ package-lock.json │
│ https://osv.dev/GHSA-cwfw-4gq5-mrqx │      │ npm       │ braces            │ 1.8.5   │ package-lock.json │
│ https://osv.dev/GHSA-g95f-p29q-9xw4 │ 3.7  │ npm       │ braces            │ 1.8.5   │ package-lock.json │
│ https://osv.dev/GHSA-4jwp-vfvf-657p │ 5.4  │ npm       │ bson              │ 1.0.9   │ package-lock.json │
│ https://osv.dev/GHSA-v8w9-2789-6hhr │ 9.8  │ npm       │ bson              │ 1.0.9   │ package-lock.json │
│ https://osv.dev/GHSA-c6rq-rjc2-86v2 │ 2.5  │ npm       │ chownr            │ 1.0.1   │ package-lock.json │
...[SNIP]...

---

### Resolve Issue in NPM
The scan showed several vulnerabilities, including one in the **adm-zip** package. And we need to tackle this first.

Command Output
╭─────────────────────────────────────┬──────┬───────────┬───────────────────┬─────────┬───────────────────╮
│ OSV URL                             │ CVSS │ ECOSYSTEM │ PACKAGE           │ VERSION │ SOURCE            │
├─────────────────────────────────────┼──────┼───────────┼───────────────────┼─────────┼───────────────────┤
│ https://osv.dev/GHSA-3v6h-hqm4-2rg6 │ 5.5  │ npm       │ adm-zip           │ 0.4.4   │ package-lock.json │
╰─────────────────────────────────────┴──────┴───────────┴───────────────────┴─────────┴───────────────────╯

---

### Step 1: Identify dependency usage
npm list adm-zip

Command Output
owasp-nodejs-goat@1.3.0 /webapp  
└─┬ selenium-webdriver@2.53.3  
  └── adm-zip@0.4.4  

From the above result, we see that the **adm-zip** package is used in **selenium-webdriver**.  

---

### Step 2: Update dependency
npm install selenium-webdriver@latest --save

Command Output
+ selenium-webdriver@4.21.0  
added 5 packages from 17 contributors, removed 6 packages, updated 3 packages and audited 1379 packages in 5.942s  

Check the update:

npm list selenium-webdriver

Command Output
owasp-nodejs-goat@1.3.0 /webapp  
└── selenium-webdriver@4.21.0  

✅ Voila! It was updated from version 2.53.3 → 4.21.0.

---

### Step 3: Re-scan for issues
osv-scanner -L package-lock.json

Command Output
Scanned /webapp/package-lock.json file and found 1043 packages  
...[SNIP]...  
The adm-zip package is gone, and we have successfully reduced the vulnerability.  

---

### Step 4: Fixing another vulnerable package (yargs-parser)
Now let’s address the **yargs-parser** package.

Check where it’s used:

npm list yargs-parser

Command Output
owasp-nodejs-goat@1.3.0 /webapp  
└─┬ grunt-if@0.2.0  
  └─┬ grunt-contrib-nodeunit@1.0.0  
    └─┬ nodeunit@0.9.5  
      └─┬ tap@7.1.2  
        └─┬ nyc@7.1.0  
          ├─┬ yargs@4.8.1  
          │ └── yargs-parser@2.4.1  deduped  
          └── yargs-parser@2.4.1  

Update grunt-if:

npm install grunt-if@latest --save

Command Output
+ yargs@17.7.2  
added 5 packages from 7 contributors, updated 1 package, moved 1 package and audited 1638 packages in 6.896s  

Verify:

npm list yargs-parser

Command Output
owasp-nodejs-goat@1.3.0 /webapp  
└─┬ grunt-if@0.1.5  
  └─┬ grunt-contrib-nodeunit@0.3.3  
    └─┬ nodeunit@0.8.8  
      └─┬ tap@19.0.2  
        └─┬ @tapjs/run@2.0.2  
          └─┬ c8@9.1.0  
            ├─┬ yargs@17.7.2  
            │ └── yargs-parser@21.1.1  deduped  
            └── yargs-parser@21.1.1  

✅ It was updated from version 2.4.1 → 21.1.1.

---

### Step 5: Final Re-scan
osv-scanner -L package-lock.json

Command Output
Scanned /webapp/package-lock.json file and found 1043 packages  
...[SNIP]...  
The yargs-parser package is gone, and we have successfully reduced the vulnerability count again.  

---

### Important Point: Python vs Node.js Package Management
Comparison Aspect | Description
---|---
**Python’s requirements.txt** | Enumerates precise versions of each package needed. Like a shopping list with specific items.  
**Node.js’s package.json** | Lists required packages but allows varying versions. Like asking for any variety of apples.  
**Node.js’s package-lock.json** | Ensures everyone installs the exact same versions. Like a detailed shopping list that guarantees identical purchases.  

The OSV scanner uses `requirements.txt` in Python and `package-lock.json` in Node.js because both are explicit about versions — ensuring reliable vulnerability detection.

---
```
````markdown
Advance Scanning Methods
To ensure the security of our project’s dependencies, we learned about two advanced scanning methods offered by the OSV Scanner:

1. **Experimental Local DB Scanning**  
2. **False Positive Analysis Scanning**

---

### Implementation with Experimental Local DB
We started with the **Experimental Local DB Scanning** method.  
This method uses the `--experimental-local-db` option in the OSV Scanner, designed to download or update the local database and scan the package based on this database.

Command:
```bash
osv-scanner --experimental-local-db .
````

Command Output

```
Scanning dir .
Scanned /webapp/package-lock.json file and found 1020 packages
Scanned /webapp/requirements.txt file and found 1 package
Loaded npm local db from /root/.cache/osv-scanner/npm/all.zip
Loaded PyPI local db from /root/.cache/osv-scanner/PyPI/all.zip
╭─────────────────────────────────────┬──────┬───────────┬───────────────────┬─────────┬───────────────────╮
│ OSV URL                             │ CVSS │ ECOSYSTEM │ PACKAGE           │ VERSION │ SOURCE            │
├─────────────────────────────────────┼──────┼───────────┼───────────────────┼─────────┼───────────────────┤
│ https://osv.dev/GHSA-pp7h-53gx-mx7r │ 6.5  │ npm       │ bl                │ 1.0.3   │ package-lock.json │
│ https://osv.dev/GHSA-pp7h-53gx-mx7r │ 6.5  │ npm       │ bl                │ 1.1.2   │ package-lock.json │
│ https://osv.dev/GHSA-832h-xg76-4gv6 │ 7.5  │ npm       │ brace-expansion   │ 1.1.6   │ package-lock.json │
...[SNIP]...
╰─────────────────────────────────────┴──────┴───────────┴───────────────────┴─────────┴───────────────────╯
```

The databases are cached locally under `/root/.cache/osv-scanner/`.

Inspect database contents:

```bash
zipinfo /root/.cache/osv-scanner/npm/all.zip
```

Command Output (excerpt)

```
-rw-r--r--  2.0 unx     2199 b- defN 23-Sep-26 06:18 MAL-2023-982.json
-rw-r--r--  2.0 unx     2210 b- defN 23-Sep-26 06:18 MAL-2023-983.json
...
12410 files, 28171686 bytes uncompressed, 11624512 bytes compressed:  58.7%
```

---

### Experimental Offline Scan

Now, we try the **offline scan**:

```bash
osv-scanner --experimental-offline .
```

Command Output

```
Scanning dir .
Scanned /webapp/package-lock.json file and found 1020 packages
Scanned /webapp/requirements.txt file and found 1 package
Loaded npm local db from /root/.cache/osv-scanner/npm/all.zip
Loaded PyPI local db from /root/.cache/osv-scanner/PyPI/all.zip
...[SNIP]...
```

---

### Experimental Offline VS Local DB

| Option                    | Description                                                                      |
| ------------------------- | -------------------------------------------------------------------------------- |
| `--experimental-local-db` | Downloads or updates the database online, stores in `/root/.cache/osv-scanner/`. |
| `--experimental-offline`  | Uses existing cached database, requires no internet connection.                  |

---

### Ignoring Vulnerability For False Positive Analysis

Now, let’s use **False Positive Analysis** by excluding certain vulnerability IDs.

Run scan:

```bash
osv-scanner -L requirements.txt
```

Command Output

```
Scanned /webapp/requirements.txt file and found 1 package
╭─────────────────────────────────────┬──────┬───────────┬─────────┬─────────┬──────────────────╮
│ OSV URL                             │ CVSS │ ECOSYSTEM │ PACKAGE │ VERSION │ SOURCE           │
├─────────────────────────────────────┼──────┼───────────┼─────────┼─────────┼──────────────────┤
│ https://osv.dev/GHSA-vm8q-m57g-pff3 │      │ PyPI      │ django  │ 5.0     │ requirements.txt │
│ https://osv.dev/PYSEC-2024-47       │      │           │         │         │                  │
│ https://osv.dev/GHSA-xxj9-f6rv-m3x4 │ 5.9  │ PyPI      │ django  │ 5.0     │ requirements.txt │
│ https://osv.dev/PYSEC-2024-28       │      │           │         │         │                  │
╰─────────────────────────────────────┴──────┴───────────┴─────────┴─────────┴──────────────────╯
```

We decide to exclude **PYSEC-2024-47** and **PYSEC-2024-28**.

Create config file:

```bash
cat > /webapp/config.toml <<EOF
[[IgnoredVulns]]
id = "PYSEC-2024-47"
reason = "false positive"

[[IgnoredVulns]]
id = "PYSEC-2024-28"
reason = "false positive"
EOF
```

Run scan with config:

```bash
osv-scanner --config=/webapp/config.toml -L requirements.txt
```

Command Output

```
Scanned /webapp/requirements.txt file and found 1 package
PYSEC-2024-47 and 1 alias have been filtered out because: false positive
PYSEC-2024-28 and 1 alias have been filtered out because: false positive
Filtered 4 vulnerabilities from output
No vulnerabilities found
```

---

✅ With configuration, we successfully filtered out false positives.
You can extend this method by adding more IDs, expiry dates, or reasons in the `osv-scanner.toml`.

---

```
```
```markdown
Conclusion
The Open Source Vulnerabilities (OSV) scanner is a useful tool that helps in identifying known vulnerabilities in software packages, particularly within the context of open-source usage. OSV scanner does this by scanning specific files such as Python’s `requirements.txt`, Node.js’ `package-lock.json`, and similar lock files in other environments. Each line in the output presents a known vulnerability within a specific version of a package that has been detected within your software.

Note that while OSV scanner can inspect a variety of package systems, as of the current version, OSV scanner does not directly support Node.js’ `package.json` due to its lack of specific versioning. Instead, OSV scanner requires `package-lock.json` for Node.js projects.

As the digital landscape constantly evolves, so does the OSV scanner. It is always recommended to regularly check for updates and improvements in functionality.
```
```markdown
Additional Resources
For more detailed information, and for regular updates regarding the OSV scanner you can refer to:

- [Official OSV Scanner Repository](https://github.com/google/osv-scanner)  
- [OSV Website](https://osv.dev)  
- [OSV Scanner Announcement](https://security.googleblog.com/2022/12/announcing-osv-scanner-vulnerability.html)  
```
````markdown
How To Embed OSV Scanner Into GitLab
---

### A Simple CI/CD Pipeline with OSV Scanner
Considering your DevOps team created a simple CI pipeline with the following contents. We will **add the OSV Scanner step** into this pipeline to perform Software Composition Analysis (SCA) during the CI process.

---

### Modified `.gitlab-ci.yml`

```yaml
image: docker:20.10  # To run all jobs in this pipeline, use the latest docker image

services:
  - docker:dind       # To run all jobs in this pipeline, use a docker image that contains a docker daemon running inside (dind - docker in docker).
                      # Reference: https://forum.gitlab.com/t/why-services-docker-dind-is-needed-while-already-having-image-docker/43534

stages:
  - build
  - test
  - sca
  - release
  - preprod
  - integration
  - prod

build:
  stage: build
  image: python:3.6
  before_script:
   - pip3 install --upgrade virtualenv
  script:
   - virtualenv env                       # Create a virtual environment for the python application
   - source env/bin/activate              # Activate the virtual environment
   - pip install -r requirements.txt      # Install the required third party packages as defined in requirements.txt
   - python manage.py check               # Run checks to ensure the application is working fine

test:
  stage: test
  image: python:3.6
  before_script:
   - pip3 install --upgrade virtualenv
  script:
   - virtualenv env
   - source env/bin/activate
   - pip install -r requirements.txt
   - python manage.py test taskManager

osv-scanner:
  stage: sca
  image:
    name: golang:1.22  # OSV Scanner requires Go or binary installation
    entrypoint: [""]
  script:
    - apt-get update && apt-get install -y wget unzip
    - wget -O /usr/bin/osv-scanner https://github.com/google/osv-scanner/releases/download/v1.4.0/osv-scanner_1.4.0_linux_amd64
    - chmod +x /usr/bin/osv-scanner
    # Run scan on lock files
    - osv-scanner -L requirements.txt || true
    - osv-scanner -L package-lock.json || true
  allow_failure: true   # Let pipeline continue even if vulnerabilities are found
  artifacts:
    when: always
    paths:
      - osv-scanner-report.json
    reports:
      dotenv: osv-scanner-report.json
````

---

### Explanation of Changes

* **New Stage Added**: `sca` (software composition analysis).
* **OSV Scanner Job**:

  * Downloads and installs the `osv-scanner` binary.
  * Runs scans against `requirements.txt` and `package-lock.json`.
  * Uses `|| true` to ensure the pipeline continues even if vulnerabilities are found (can be removed if you want pipeline to fail on vulnerabilities).
  * Generates an **artifact report** for later review.
* **Artifacts**: Scan results are stored for visibility and compliance review.

---

### Verify the Pipeline

1. Log into GitLab with the provided credentials:

   * **Link**: [CI/CD Machine](https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/django-nv/-/blob/main/.gitlab-ci.yml)
   * **Username**: root
   * **Password**: pdso-training

2. Replace the content of `.gitlab-ci.yml` with the above pipeline.

   * Click **Edit** → Replace (Ctrl+A, Ctrl+V) → **Commit changes**.

3. Navigate to **Pipelines**:
   [Pipelines Page](https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/django-nv/pipelines)

4. The pipeline will start automatically.

   * You should now see **build → test → osv-scanner → release → ...** in the job sequence.
   * Click on the `osv-scanner` job to view vulnerability scan results.

---

✅ With this setup, your GitLab CI/CD pipeline now automatically performs dependency vulnerability scanning with **OSV Scanner** during every run.

```

Would you like me to also extend this pipeline to **fail the build if high/critical vulnerabilities are found**, instead of allowing it to pass?
```
```markdown
Embed OSV Scanner In CI/CD Pipeline
As discussed in the SCA using the OSV Scanner exercise, we can embed the OSV Scanner tool in our CI/CD pipeline.

However, do remember that you need run commands manually in a local system like devsecops box before embedding commands for automated execution in CI/CD pipelines.

Manually testing commands in a local environment like devsecops box helps in easy troubleshooting.

Do you wonder which stage this job should go into?

Maybe you want to scan the components before performing SAST scans.

Click anywhere to copy

image: docker:20.10  # To run all jobs in this pipeline, use the latest docker image

services:
  - docker:dind       # To run all jobs in this pipeline, use a docker image that contains a docker daemon running inside (dind - docker in docker). Reference: https://forum.gitlab.com/t/why-services-docker-dind-is-needed-while-already-having-image-docker/43534

stages:
  - build
  - test
  - release
  - preprod
  - integration
  - prod

build:
  stage: build
  image: python:3.6
  before_script:
   - pip3 install --upgrade virtualenv
  script:
   - virtualenv env                       # Create a virtual environment for the python application
   - source env/bin/activate              # Activate the virtual environment
   - pip install -r requirements.txt      # Install the required third party packages as defined in requirements.txt
   - python manage.py check               # Run checks to ensure the application is working fine

test:
  stage: test
  image: python:3.6
  before_script:
   - pip3 install --upgrade virtualenv
  script:
   - virtualenv env
   - source env/bin/activate
   - pip install -r requirements.txt
   - python manage.py test taskManager

osv-scanner:
  stage: test
  image: golang:1.22-alpine3.19
  before_script:
    - apk add npm
    - npm install
  script:
    - go install github.com/google/osv-scanner/cmd/osv-scanner@v1
    - osv-scanner .

integration:
  stage: integration
  script:
    - echo "This is an integration step."
    - exit 1
  allow_failure: true # Even if the job fails, continue to the next stages

prod:
  stage: prod
  script:
    - echo "This is a deploy step."
  when: manual # Continuous Delivery

Copy the above CI script and add it to the .gitlab.ci.yml file on Gitlab repo at https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/django-nv/-/blob/main/.gitlab-ci.yml.

Do not forget to click on the Commit Changes button to save the file.

As discussed, any change to the repo kick starts the pipeline.

We can see the results of this pipeline by visiting https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/django-nv/pipelines.

Click on the appropriate job name to see the output.

Perfect! In the result above, we have noticed for the OSV Scanner output in the CI/CD pipeline in the table. You can see the vulnerability list of the package above in every suggested package.

osv-scanner-output

In the next step, you will learn the need to create artifact for OSV Scanner output in the builds.
```
````markdown
Saving File As An Artifacts
---

In the previous step, we have understood the basic scanning process using OSV Scanner. Let’s now convert the scan output into a file format so that it can be easily analyzed. We will generate the OSV Scanner results in **JSON format** and save them as an artifact.

---

### Updated `.gitlab-ci.yml`

```yaml
image: docker:20.10  # To run all jobs in this pipeline, use the latest docker image

services:
  - docker:dind       # To run all jobs in this pipeline, use a docker image that contains a docker daemon running inside (dind - docker in docker).
                      # Reference: https://forum.gitlab.com/t/why-services-docker-dind-is-needed-while-already-having-image-docker/43534

stages:
  - build
  - test
  - release
  - preprod
  - integration
  - prod

build:
  stage: build
  image: python:3.6
  before_script:
   - pip3 install --upgrade virtualenv
  script:
   - virtualenv env                       # Create a virtual environment for the python application
   - source env/bin/activate              # Activate the virtual environment
   - pip install -r requirements.txt      # Install the required third party packages as defined in requirements.txt
   - python manage.py check               # Run checks to ensure the application is working fine

test:
  stage: test
  image: python:3.6
  before_script:
   - pip3 install --upgrade virtualenv
  script:
   - virtualenv env
   - source env/bin/activate
   - pip install -r requirements.txt
   - python manage.py test taskManager

osv-scanner:
  stage: test
  image: golang:1.22-alpine3.19
  before_script:
    - apk add npm
    - npm install
  script:
    - go install github.com/google/osv-scanner/cmd/osv-scanner@v1
    - osv-scanner --json . > osv-report.json
  artifacts:
    paths: [osv-report.json]
    when: always

integration:
  stage: integration
  script:
    - echo "This is an integration step."
    - exit 1
  allow_failure: true # Even if the job fails, continue to the next stages

prod:
  stage: prod
  script:
    - echo "This is a deploy step."
  when: manual # Continuous Delivery
````

---

### Explanation of Changes

* **`--json` output**: We added the `--json` flag so OSV Scanner generates results in JSON format.
* **Redirecting output**: The results are saved to `osv-report.json`.
* **Artifacts block**:

  * Ensures the JSON report is always saved.
  * Allows developers/security teams to download and review the OSV scan results.

---

### Key Notes

* We discussed different installation methods:

  * **Native installation** – directly on the system.
  * **Package manager or binary** – faster and dependency-free.
  * **Dockerized** – most recommended for CI/CD since it avoids dependency issues.

* If the job fails with a **non-zero exit code (exit 1)**:

  * It is because OSV Scanner found vulnerabilities.
  * This is **expected behavior** for security tools.
  * Options to handle:

    * Fix the vulnerabilities.
    * Or add `allow_failure: true` to let the pipeline continue without blocking other jobs.

---

### Verification

1. Copy the updated CI script and paste it into the `.gitlab-ci.yml` file at:
   [django-nv GitLab CI file](https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/django-nv/-/blob/main/.gitlab-ci.yml)

2. Commit changes using **Commit changes** button.

3. The pipeline will start automatically.

   * Navigate to: [Pipelines Page](https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/django-nv/pipelines)
   * Click the `osv-scanner` job to download and inspect `osv-report.json`.

---

✅ In the next step, you will learn how to **allow failure** for the OSV Scanner job in the builds.

```
```
````markdown
Allow The Job Failure
---

In DevSecOps **Maturity Levels 1 and 2**, you usually **do not want to block the entire pipeline** because of tool findings, since there may be false positives.  
Instead, you can let the security scan job fail, but **mark it as non-blocking** using the `allow_failure: true` option.

This way:
- The scanner still runs.  
- You still get the vulnerability report (`osv-report.json`).  
- The pipeline continues to the next stages, even if vulnerabilities are found.  

---

### Final `.gitlab-ci.yml`

```yaml
image: docker:20.10  # To run all jobs in this pipeline, use the latest docker image

services:
  - docker:dind       # To run all jobs in this pipeline, use a docker image that contains a docker daemon running inside (dind - docker in docker).
                      # Reference: https://forum.gitlab.com/t/why-services-docker-dind-is-needed-while-already-having-image-docker/43534

stages:
  - build
  - test
  - release
  - preprod
  - integration
  - prod

build:
  stage: build
  image: python:3.6
  before_script:
   - pip3 install --upgrade virtualenv
  script:
   - virtualenv env                       # Create a virtual environment for the python application
   - source env/bin/activate              # Activate the virtual environment
   - pip install -r requirements.txt      # Install the required third party packages as defined in requirements.txt
   - python manage.py check               # Run checks to ensure the application is working fine

test:
  stage: test
  image: python:3.6
  before_script:
   - pip3 install --upgrade virtualenv
  script:
   - virtualenv env
   - source env/bin/activate
   - pip install -r requirements.txt
   - python manage.py test taskManager

osv-scanner:
  stage: test
  image: golang:1.22-alpine3.19
  before_script:
    - apk add npm
    - npm install
  script:
    - go install github.com/google/osv-scanner/cmd/osv-scanner@v1
    - osv-scanner --json . > osv-report.json
  artifacts:
    paths: [osv-report.json]
    when: always
  allow_failure: true   # <--- allows scan failures without blocking pipeline

integration:
  stage: integration
  script:
    - echo "This is an integration step"
    - exit 1
  allow_failure: true   # Even if the job fails, continue to the next stages

prod:
  stage: prod
  script:
    - echo "This is a deploy step."
  when: manual           # Continuous Delivery
````

---

### Verification

1. Add the above `.gitlab-ci.yml` to your GitLab repo:
   [django-nv GitLab CI file](https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/django-nv/-/blob/main/.gitlab-ci.yml)

2. Commit the changes.

3. The pipeline starts automatically.

   * Navigate to: [Pipeline Results](https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/django-nv/pipelines)
   * Click on the `osv-scanner` job to see the results and download the `osv-report.json`.

---

✅ Now the `osv-scanner` job may fail (exit code `1` if vulnerabilities are found), but it **won’t block the pipeline**, letting other jobs continue.

Do you want me to also show you how to make the **failure conditional** (e.g., only fail if *critical* CVSS > 8 vulnerabilities are found)?

```markdown
Software Component Analysis Using Semgrep  
Download the Source Code  
We will do all the exercises locally first in DevSecOps-Box, so let’s start the exercise.

First, we need to download the source code of the project from our git repository.

```

git clone [https://gitlab.practical-devsecops.training/pdso/django.nv](https://gitlab.practical-devsecops.training/pdso/django.nv) webapp

```

Command Output  
```

Cloning into 'webapp'...
warning: redirecting to [https://gitlab.practical-devsecops.training/pdso/django.nv.git/](https://gitlab.practical-devsecops.training/pdso/django.nv.git/)
remote: Enumerating objects: 302, done.
remote: Total 302 (delta 0), reused 0 (delta 0), pack-reused 302
Receiving objects: 100% (302/302), 1.05 MiB | 37.21 MiB/s, done.
Resolving deltas: 100% (122/122), done.

```

Let’s cd into the application so we can scan the app.

```

cd webapp

```

We are now in the webapp directory.

Let’s move to the next step.
```
```markdown
Install Semgrep  
Semgrep is a fast, open-source, static analysis engine for finding bugs, detecting vulnerabilities in third-party dependencies, and enforcing code standards. Semgrep analyzes code locally on your computer or in your build environment: code is never uploaded.  
You can find more details about the project at semgrep.

Let’s install the Semgrep tool on the system to perform software component analysis.

```

pip3 install semgrep==1.81.0

```

Command Output  
```

Collecting semgrep==1.31.0
Downloading semgrep-1.31.0-cp37.cp38.cp39.cp310.cp311.py37.py38.py39.py310.py311-none-any.whl (31.4 MB)
|████████████████████████████████| 31.4 MB 32.4 MB/s
Collecting typing-extensions\~=4.2
Downloading typing\_extensions-4.11.0-py3-none-any.whl (34 kB)
Collecting urllib3\~=1.26
Downloading urllib3-1.26.18-py2.py3-none-any.whl (143 kB)
|████████████████████████████████| 143 kB 80.9 MB/s
Collecting click\~=8.1
Downloading click-8.1.7-py3-none-any.whl (97 kB)
|████████████████████████████████| 97 kB 19.2 MB/s
Collecting ruamel.yaml<0.18,>=0.16.0
Downloading ruamel.yaml-0.17.40-py3-none-any.whl (113 kB)
|████████████████████████████████| 113 kB 76.7 MB/s
Requirement already satisfied: requests\~=2.22 in /usr/lib/python3/dist-packages (from semgrep==1.31.0) (2.22.0)
\[\[\[....SNIPPED....]]]
Found existing installation: urllib3 1.25.8
Not uninstalling urllib3 at /usr/lib/python3/dist-packages, outside environment /usr
Can't uninstall 'urllib3'. No files were found to uninstall.
Successfully installed attrs-23.2.0 boltons-21.0.0 bracex-2.4 click-8.1.7 click-option-group-0.5.6 colorama-0.4.6 defusedxml-0.7.1 face-22.0.0 glom-22.1.0 jsonschema-4.21.1 jsonschema-specifications-2023.12.1 markdown-it-py-3.0.0 mdurl-0.1.2 peewee-3.17.2 pygments-2.17.2 python-lsp-jsonrpc-1.0.0 referencing-0.34.0 rich-13.7.1 rpds-py-0.18.0 ruamel.yaml-0.17.40 ruamel.yaml.clib-0.2.8 semgrep-1.31.0 tomli-2.0.1 typing-extensions-4.11.0 ujson-5.9.0 urllib3-1.26.18 wcmatch-8.5.1

```

We have successfully installed semgrep, let’s explore its functionality now.

```

semgrep --help

```

Command Output  
```

Usage: semgrep \[OPTIONS] COMMAND \[ARGS]...

To get started quickly, run `semgrep scan --config auto`

Run `semgrep SUBCOMMAND --help` for more information on each subcommand

If no subcommand is passed, will run `scan` subcommand by default

Options:
-h, --help  Show this message and exit.

Commands:
ci                   The recommended way to run semgrep in CI
install-semgrep-pro  Install the Semgrep Pro Engine
login                Obtain and save credentials for semgrep.dev
logout               Remove locally stored credentials to semgrep.dev
lsp                  Start the Semgrep LSP server (useful for IDEs)
publish              Upload rule to semgrep.dev
scan                 Run semgrep rules on files
show                 Show various information about Semgrep

```

For more information about the Semgrep scan feature, you can visit this link: **Semgrep Scan Command Options** or try inputting `semgrep scan --help`.

---

### Notes
If you’re encountering the same error with every command inputted, like the following:

Command Output  
```

/usr/lib/python3/dist-packages/requests/**init**.py:89: RequestsDependencyWarning: urllib3 (1.26.18) or chardet (3.0.4) doesn't match a supported version! warnings.warn("urllib3 ({}) or chardet ({}) doesn't match a supported

```

It’s likely due to an outdated version of the Python requests library. You can update it using the following command:

```

pip3 install --upgrade requests

```

Let’s move to the next step.
```
```markdown
SCA Using Semgrep  
Let’s try and run the SCA scan using Semgrep.  
SCA is referred as supply chain scan in the semgrep ecosystem.

`--supply-chain` argument is a sub command of the `ci` command.

```

semgrep ci --supply-chain

```

Command Output  
```

run `semgrep login` before using `semgrep ci` or set `--config`

```

When we try and run the above command we get the error stating we need to login before we run the scan.

For a successful SCA scan we need a bunch of rules against which our project will be assessed which are hosted on semgrep’s site.

Let’s follow the instruction and try and login to the site.

```

semgrep login

```

You will be presented with a URL in the output that you will need to copy and paste in a new tab.

This facilitates the Semgrep login verification. You can choose to sign in with Github or Gitlab to complete the login.

You will be asked to validate your client token post login. Kindly go ahead and do so.

---

### Note
Do not use your corporate Github or Gitlab account to login to Semgrep.

---

Command Output  
```

Login enables additional proprietary Semgrep Registry rules and running custom policies from Semgrep Cloud Platform.
Opening login at: [https://semgrep.dev/login?cli-token=273fdcd0-382e-4734-8d3c-bcf715ea99ae\&docker=False\&gha=False](https://semgrep.dev/login?cli-token=273fdcd0-382e-4734-8d3c-bcf715ea99ae&docker=False&gha=False)

Once you've logged in, return here and you'll be ready to start using new Semgrep rules.
Saved login token

```
    76eb4af5827c0d3ad352148e0afd398cbb6302a0a4c1bf6d0d83945a3079f46b
```

in /root/.semgrep/settings.yml.
Note: You can always generate more tokens at [https://semgrep.dev/orgs/-/settings/tokens](https://semgrep.dev/orgs/-/settings/tokens)

```

If everything goes correctly then you should see something like the above output.

---

### Note
The token you see in the above output will be different for you.

From the output we can deduce that our token is saved at this `/root/.semgrep/settings.yml` location.  
We can validate it using:

```

cat /root/.semgrep/settings.yml

```

---

Finally, let’s run the SCA scan.  

The output number may vary as it is dynamic.

```

semgrep ci --supply-chain

```

Command Output  
```

Warning: 3 timeout error(s) in taskManager/static/taskManager/js/jquery.js when running the following
rules: \[ssc-0c175d3d-9931-bc83-3b79-915b3bd8486d, ssc-20d1add6-68b6-7310-b587-7c694bc14de1,
ssc-5e9dab7c-3887-460a-b992-4b0715556b8c]
Semgrep stopped running rules on taskManager/static/taskManager/js/jquery.js after 3 timeout error(s). See
`--timeout-threshold` for more info.

```

Let’s try and run the scan again with the `--timeout` argument.

```

semgrep ci --supply-chain --timeout 45

```

---

### What the function of the above command?  

Command Output (truncated for readability)  
```

┌────────────────┐
│ Debugging Info │
└────────────────┘

SCAN ENVIRONMENT
versions    - semgrep 1.81.0 on python 3.9.5
environment - running in environment git, triggering event is unknown

CONNECTION
Initializing scan (deployment=practical-devsecops-com, scan\_id=37116279)
Enabled products: Supply Chain

ENGINE
Semgrep Pro Engine will be installed in /usr/local/lib/python3.9/dist-packages/semgrep/bin/semgrep-core-proprietary
Overwriting Semgrep Pro Engine already installed!
Downloading... ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 212.9/212.9 MB 156.7 MB/s 0:00:00

Successfully installed Semgrep Pro Engine (version 1.80.0)!

```

---

```

┌─────────────┐
│ Scan Status │
└─────────────┘
Scanning 148 files (only git-tracked) with 0 Code rules, 3111 Supply Chain rules:

CODE RULES
Nothing to scan.

SUPPLY CHAIN RULES

Ecosystem   Rules   Files   Lockfiles
──────────────────────────────────────────────
Pypi         3111      21   requirements.txt

Analysis       Rules
──────────────────────
Basic           2787
Reachability     324

```

---

### Supported Languages and Package Managers

| Language   | Supported package managers | Lockfile                     |
|------------|-----------------------------|------------------------------|
| Go         | Go modules (go mod)        | go.mod                       |
| JavaScript / TypeScript | npm (Node.js) | package-lock.json            |
| Yarn       | Yarn 2, Yarn 3             | yarn.lock                    |
| pnpm       | pnpm                       | pnpm-lock.yaml               |
| Python     | pip                        | requirements.txt             |
| Pipenv     | pip-tools / Pipenv         | requirements.txt / Pipfile.lock |
| Poetry     | Poetry                     | poetry.lock                  |
| Ruby       | RubyGems                   | Gemfile.lock                 |
| Java       | Gradle / Maven             | gradle.lockfile / Maven tree |
| Rust       | Cargo                      | cargo.lock                   |
| PHP        | Composer                   | composer.lock                |
| C#         | NuGet                      | packages.lock.json           |

---

Command Output  
```

┌───────────────────────────────────┐
│ 2 Reachable Supply Chain Findings │
└───────────────────────────────────┘

```
requirements.txt
```

❯❯❱ django - CVE-2019-19844
Severity: CRITICAL
django 3.x before 3.0.1, django 2.x before 2.2.9, and django 1.x before 1.11.27 are
vulnerable to weak password recovery mechanism for forgotten password. A malicious user
could provide a Unicode homoglyph email address to Django and obtain a password reset
token for another user due to unsafe normalization of Unicode characters when comparing
email addresses. Upgrade to django 3.0.1, django 2.2.9, or django 1.11.27.

```
       ▶▶┆ Fixed for django at versions: 1.11.27, 2.2.9, 3.0.1
        1┆ Django==3.0
```

❯❯❱ django - CVE-2021-33203
Severity: MODERATE
django versions before 2.2.24, >= 3.2.0 before 3.2.4, >= 3.0.0 before 3.1.12 are
vulnerable to Improper Limitation Of A Pathname To A Restricted Directory ('Path
Traversal'). Staff members can use the `admindocs` `TemplateDetailView` view to check
the existence of arbitrary files. Additionally, if (and only if) the default admindocs
templates have been customized by application developers to also show file contents,
then not only the *existence* but also the *file contents* would have been exposed. In
simpler terms, there is directory traversal outside of the template root directories.

```
       ▶▶┆ Fixed for django at versions: 2.2.24, 3.1.12, 3.2.4
        1┆ Django==3.0
```

```

---

### What does "Reachable" mean?

Unlike other SCA tools which would just scan the lockfile and flag vulnerabilities based on the pinned version of the package/dependency, Semgrep is slightly different.

Semgrep performs **reachability analysis** for the identified SCA vulnerabilities.  
Reachability pertains to whether a vulnerable code fragment from a dependency is actually used within the codebase.

For a vulnerability to be classified as **reachable**, two conditions must be met:
1. The code pattern in the dependency must match.  
2. The vulnerable version must also match.  

---

Command Output  
```

┌──────────────────────────────────────┐
│ 6 Undetermined Supply Chain Findings │
└──────────────────────────────────────┘

```
requirements.txt
❯❱ django - CVE-2024-24680
      Severity: MODERATE                                                     
      Affected versions of django are vulnerable to Reliance on Uncontrolled Component.

       ▶▶┆ Fixed for django at versions: 3.2.24, 4.2.10, 5.0.2
        1┆ Django==3.0
```

```

---

### What does "Undetermined" mean?

An **undetermined supply chain finding** indicates that:
- The version of the dependency includes a recognized vulnerability.  
- But the particular vulnerable code component remains **unused** within your codebase.

---

Let’s move to the next step.
```
```markdown
Save Semgrep SCA Output  
We would like to save the output of the Semgrep SCA scan for further analysis.

Let’s try and save the output in JSON format.

```

semgrep ci --supply-chain --timeout 45 --output semgrep-sca-output.json --json

```

Command Output  
```

┌────────────────┐
│ Debugging Info │
└────────────────┘

SCAN ENVIRONMENT
versions    - semgrep 1.81.0 on python 3.9.5
environment - running in environment git, triggering event is unknown

CONNECTION
Initializing scan (deployment=practical-devsecops-com, scan\_id=37116697)
Enabled products: Supply Chain

ENGINE
Using Semgrep Pro Version: 1.80.0
Installed at /usr/local/lib/python3.9/dist-packages/semgrep/bin/semgrep-core-proprietary

```
```

┌─────────────┐
│ Scan Status │
└─────────────┘
Scanning 148 files (only git-tracked) with 0 Code rules, 3111 Supply Chain rules:

CODE RULES
Nothing to scan.

SUPPLY CHAIN RULES

Ecosystem   Rules   Files   Lockfiles
──────────────────────────────────────────────
Pypi         3111      21   requirements.txt

Analysis       Rules
──────────────────────
Basic           2787
Reachability     324

```
```

PROGRESS

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━╸  99% 0:00:53

```
```

┌──────────────┐
│ Scan Summary │
└──────────────┘
Some files were skipped or only partially analyzed.
Scan was limited to files tracked by git.
Partially scanned: 41 files only partially analyzed due to parsing or internal Semgrep errors
Scan skipped: 8 files matching --exclude patterns
For a full list of skipped files, run semgrep with the --verbose flag.

CI scan completed successfully.
Found 10 findings (0 blocking) from 19639 rules.
Uploading scan results
Finalizing scan
View results in Semgrep Cloud Platform:
[https://semgrep.dev/orgs/practical-devsecops-com/findings?repo=local\_scan/webapp\&ref=main](https://semgrep.dev/orgs/practical-devsecops-com/findings?repo=local_scan/webapp&ref=main)
[https://semgrep.dev/orgs/practical-devsecops-com/supply-chain](https://semgrep.dev/orgs/practical-devsecops-com/supply-chain)
No blocking findings so exiting with code 0

⏫  A new version of Semgrep is available. See [https://semgrep.dev/docs/upgrading](https://semgrep.dev/docs/upgrading)

```

---

Let’s validate the existence of the output file using:

```

ls -l semgrep-sca-output.json

```

We can integrate **Semgrep SCA** into our **CI/CD pipeline** and also attach the generated report as an artifact for developer consumption.
```
```markdown
Software Component Analysis Using Trivy  
Installing Trivy  
Trivy is a comprehensive vulnerability scanner for containers and file systems, designed to detect security vulnerabilities in OS packages and application dependencies.

Source: Trivy Github Page.

Let’s install trivy tool in our DevSecOps-Box.

```

wget [https://github.com/aquasecurity/trivy/releases/download/v0.48.1/trivy\_0.48.1\_Linux-64bit.deb](https://github.com/aquasecurity/trivy/releases/download/v0.48.1/trivy_0.48.1_Linux-64bit.deb) && dpkg -i trivy\_\*.deb

```

After Trivy is installed, we can explore the functionality of trivy using the following command.

```

trivy -h

```

Command Output  
```

Scanner for vulnerabilities in container images, file systems, and Git repositories, as well as for configuration issues and hard-coded secrets

Usage:
trivy \[global flags] command \[flags] target
trivy \[command]

Examples:

# Scan a container image

\$ trivy image python:3.4-alpine

# Scan a container image from a tar archive

\$ trivy image --input ruby-3.1.tar

# Scan local filesystem

\$ trivy fs .

# Run in server mode

\$ trivy server

Scanning Commands
aws         \[EXPERIMENTAL] Scan AWS account
config      Scan config files for misconfigurations
filesystem  Scan local filesystem
image       Scan a container image
kubernetes  \[EXPERIMENTAL] Scan kubernetes cluster
repository  Scan a remote repository
rootfs      Scan rootfs
sbom        Scan SBOM for vulnerabilities
vm          \[EXPERIMENTAL] Scan a virtual machine image

Management Commands
module      Manage modules
plugin      Manage plugins

Utility Commands
completion  Generate the autocompletion script for the specified shell
convert     Convert Trivy JSON report into a different format
help        Help about any command
server      Server mode
version     Print the version

Flags:
\--cache-dir string          cache directory (default "/root/.cache/trivy")
-c, --config string             config path (default "trivy.yaml")
-d, --debug                     debug mode
-f, --format string             version format (json)
\--generate-default-config   write the default config to trivy-default.yaml
-h, --help                      help for trivy
\--insecure                  allow insecure server connections
-q, --quiet                     suppress progress bar and log output
\--timeout duration          timeout (default 5m0s)
-v, --version                   show version

Use "trivy \[command] --help" for more information about a command.

```

Let’s move to the next step.
```
```markdown
Downloading The Source Code  
First, we need to download the source code of the project from the git repository.

```

git clone [https://gitlab.practical-devsecops.training/pdso/django.nv.git](https://gitlab.practical-devsecops.training/pdso/django.nv.git) webapp

```

Command Output  
```

Cloning into 'webapp'...
remote: Enumerating objects: 176, done.
remote: Total 176 (delta 0), reused 0 (delta 0), pack-reused 176
Receiving objects: 100% (176/176), 6.34 MiB | 36.87 MiB/s, done.
Resolving deltas: 100% (44/44), done.

```

In the next step, we will run the Trivy tool.
```
```markdown
Running The Scanner  
In the previous step, we have downloaded the webapp directory.

As we have seen in the first step, Trivy has a command known as **filesystem** which scans the packages from a certain directory.

Let’s cd into the application code, so we can scan the repository.

```

cd webapp

```

We are now in the webapp directory.

Next, let’s execute the scanner to scan `webapp` directory using **filesystem** command from Trivy.

```

trivy filesystem .

```

---

### Database Rate Limit Error

If you encounter the following error:

Command Output  
```

init error: DB error: failed to download vulnerability DB: OCI artifact error: OCI artifact error: OCI repository error: GET [https://ghcr.io/v2/aquasecurity/trivy-db/manifests/2](https://ghcr.io/v2/aquasecurity/trivy-db/manifests/2): TOOMANYREQUESTS: retry-after: 101µs, allowed: 44000/minute

```

This occurs because the Trivy database download is hitting a rate limit. To resolve this, you can change the database repository by running:

```

export TRIVY\_DB\_REPOSITORY=public.ecr.aws/aquasecurity/trivy-db

```

Then try running the scanning command again.

---

Command Output (successful run)  
```

2024-02-07T07:02:37.069Z        INFO    Need to update DB
2024-02-07T07:02:37.069Z        INFO    DB Repository: ghcr.io/aquasecurity/trivy-db
2024-02-07T07:02:37.069Z        INFO    Downloading DB...
42.78 MiB / 42.78 MiB \[-------------------------------------------------------------------] 100.00% 25.67 MiB p/s 1.9s
2024-02-07T07:02:39.566Z        INFO    Vulnerability scanning is enabled
2024-02-07T07:02:39.566Z        INFO    Secret scanning is enabled
2024-02-07T07:02:39.566Z        INFO    If your scanning is slow, please try '--security-checks vuln' to disable secret scanning
2024-02-07T07:02:39.566Z        INFO    Please see also [https://aquasecurity.github.io/trivy/0.30.4/docs/secret/scanning/#recommendation](https://aquasecurity.github.io/trivy/0.30.4/docs/secret/scanning/#recommendation) for faster secret detection
2024-02-07T07:02:39.933Z        INFO    Number of language-specific files: 1
2024-02-07T07:02:39.933Z        INFO    Detecting pip vulnerabilities...

requirements.txt (pip)

Total: 14 (UNKNOWN: 0, LOW: 0, MEDIUM: 5, HIGH: 6, CRITICAL: 3)

```

......[SNIP]......

```

├─────────┼────────────────┼──────────┼───────────────────┼────────────────────────┼──────────────────────────────────────────────────────────────┤
│ Django  │ CVE-2021-31542 │ HIGH     │ 3.0               │ 2.2.21, 3.1.9, 3.2.1   │ Potential directory-traversal via uploaded files             │
│         │                │          │                   │                        │ [https://avd.aquasec.com/nvd/cve-2021-31542](https://avd.aquasec.com/nvd/cve-2021-31542)                   │
├─────────┼────────────────┼──────────┼───────────────────┼────────────────────────┼──────────────────────────────────────────────────────────────┤
│ Django  │ CVE-2021-33571 │ HIGH     │ 3.0               │ 2.2.24, 3.1.12, 3.2.4  │ Possible indeterminate SSRF, RFI, and LFI attacks since      │
│         │                │          │                   │                        │ validators accepted leading zeros...                         │
│         │                │          │                   │                        │ [https://avd.aquasec.com/nvd/cve-2021-33571](https://avd.aquasec.com/nvd/cve-2021-33571)                   │
│         ├────────────────┤          │                   ├────────────────────────┼──────────────────────────────────────────────────────────────┤
│         │ CVE-2021-44420 │          │                   │ 2.2.25, 3.1.14, 3.2.10 │ potential bypass of an upstream access control based on URL  │
│         │                │          │                   │                        │ paths                                                        │
│         │                │          │                   │                        │ [https://avd.aquasec.com/nvd/cve-2021-44420](https://avd.aquasec.com/nvd/cve-2021-44420)                   │
├─────────┼────────────────┼──────────┼───────────────────┼────────────────────────┼──────────────────────────────────────────────────────────────┤

```

......[SNIP]......

---

The output number may vary as it is dynamic.

As you can see, **Trivy found vulnerabilities** by scanning the contents of the `django.nv` source code present in the `django.nv` directory.

By scanning a specific repository with **Trivy filesystem**, you can:
- Proactively address security risks.  
- Ensure compliance with security standards.  
- Maintain overall code integrity within the development and deployment cycles of the repository’s applications.

---

Let’s move to the next step.
```
```markdown
Conclusion  
By scanning a Docker image at the filesystem level, **Trivy** highlights potential security risks identified within each layer of an image which helps DevOps engineers and developers to address vulnerabilities based on their severity and impact.

The critical advantage of Trivy is in its **simplicity of setup and execution**. Thus, Trivy requires very few steps for installation and operation compared to other scanners. Also, the ability to scan not only images in a Docker registry but also files saved in the local filesystem, provides flexibility and broadens the tool’s utility.

---

### Additional Resources
- https://sadilchamishka.medium.com/trivy-for-vulnerability-scanning-f9e967aea85f  
- https://www.youtube.com/watch?v=QXLpZruy6mM  
- https://medium.com/@maheshwar.ramkrushna/scanning-docker-images-for-vulnerabilities-using-trivy-for-effective-security-analysis-fa3e2844db22  
- https://www.devoteam.com/expert-view/shift-left-security-in-your-cloud-native-environment-with-trivy/  
```
```markdown
How To Embed Trivy Into GitLab
A Simple CI/CD Pipeline
Considering your DevOps team created a simple CI pipeline with the following contents. Please add the Trivy scan to this pipeline.

Click anywhere to copy

image: docker:20.10  # To run all jobs in this pipeline, use the latest docker image

services:
  - docker:dind       # To run all jobs in this pipeline, use a docker image that contains a docker daemon running inside (dind - docker in docker). Reference: https://forum.gitlab.com/t/why-services-docker-dind-is-needed-while-already-having-image-docker/43534

stages:
  - build
  - test
  - release
  - preprod
  - integration
  - prod

build:
  stage: build
  image: python:3.6
  before_script:
   - pip3 install --upgrade virtualenv
  script:
   - virtualenv env                       # Create a virtual environment for the python application
   - source env/bin/activate              # Activate the virtual environment
   - pip install -r requirements.txt      # Install the required third party packages as defined in requirements.txt
   - python manage.py check               # Run checks to ensure the application is working fine

test:
  stage: test
  image: python:3.6
  before_script:
   - pip3 install --upgrade virtualenv
  script:
   - virtualenv env
   - source env/bin/activate
   - pip install -r requirements.txt
   - python manage.py test taskManager

Let’s log into GitLab using the following details and run the above pipeline.

GitLab CI/CD Machine
Name	Value
Link	https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/django-nv/-/blob/main/.gitlab-ci.yml
Username	root
Password	pdso-training
Next, we need to create a CI/CD pipeline by replacing the .gitlab-ci.yml file content with the above CI script. Click on the Edit button to replace the content (use Control+A and Control+V).

Save changes to the file using the Commit changes button.

Verify the pipeline runs
As soon as a change is made to the repository, the pipeline starts executing the jobs.

We can see the results of this pipeline by visiting https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/django-nv/pipelines.

Click on the appropriate job name to see the output.

Let’s move to the next step.
```
```markdown
Embed Trivy in CI/CD pipeline
As discussed in the SCA using the Trivy exercise, we can embed the Trivy tool in our CI/CD pipeline.

However, do remember that you need run commands manually in a local system like devsecops box before embedding commands for automated execution in CI/CD pipelines.

Manually testing commands in a local environment like devsecops box helps in easy troubleshooting.

Do you wonder which stage this job should go into?

Maybe you want to scan the components before performing SAST scans.

Click anywhere to copy

image: docker:20.10  # To run all jobs in this pipeline, use the latest docker image

services:
  - docker:dind       # To run all jobs in this pipeline, use a docker image that contains a docker daemon running inside (dind - docker in docker). Reference: https://forum.gitlab.com/t/why-services-docker-dind-is-needed-while-already-having-image-docker/43534

stages:
  - build
  - test
  - release
  - preprod
  - integration
  - prod

build:
  stage: build
  image: python:3.6
  before_script:
   - pip3 install --upgrade virtualenv
  script:
   - virtualenv env                       # Create a virtual environment for the python application
   - source env/bin/activate              # Activate the virtual environment
   - pip install -r requirements.txt      # Install the required third party packages as defined in requirements.txt
   - python manage.py check               # Run checks to ensure the application is working fine

test:
  stage: test
  image: python:3.6
  before_script:
   - pip3 install --upgrade virtualenv
  script:
   - virtualenv env
   - source env/bin/activate
   - pip install -r requirements.txt
   - python manage.py test taskManager

trivy:
  stage: test
  script:
    - docker run --rm -v $(pwd):/src hysnsec/trivy fs . --exit-code 1 -f json -o trivy-report.json
  artifacts:
    paths: [trivy-report.json]
    when: always # What is this for?
    expire_in: one week

integration:
  stage: integration
  script:
    - echo "This is an integration step."
    - exit 1
  allow_failure: true # Even if the job fails, continue to the next stages

prod:
  stage: prod
  script:
    - echo "This is a deploy step."
  when: manual # Continuous Delivery

Copy the above CI script and add it to the .gitlab.ci.yml file on Gitlab repo at https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/django-nv/-/blob/main/.gitlab-ci.yml.

Do not forget to click on the Commit Changes button to save the file.

As discussed, any change to the repo kick starts the pipeline.

Note

We’ve discussed different methods of using the tool:

Native Installation:
Directly installing the tool on the system.
Package Manager or Binary:
Installing via a package manager or using the binary file.
Docker:
Running the tool within a Docker container.
In summary:

All methods are suitable for CI/CD integration.
Docker is recommended for CI/CD as it operates smoothly without dependencies.
Using the binary file is efficient, avoiding additional dependencies.
Ultimately, you can choose either method based on your specific situation.

We can see the results of this pipeline by visiting https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/django-nv/pipelines.

Click on the appropriate job name to see the output.

You will notice that the Trivy job stores the output to a file trivy-report.json. This is done to ensure we can process the results further via APIs or vulnerability management systems like DefectDojo.

Database Rate Limit Error

If you encounter the following error:

Command Output
init error: DB error: failed to download vulnerability DB: OCI artifact error: OCI artifact error: OCI repository error: GET https://ghcr.io/v2/aquasecurity/trivy-db/manifests/2: TOOMANYREQUESTS: retry-after: 101µs, allowed: 44000/minute
This occurs because the Trivy database download is hitting a rate limit. To resolve this, you can change the database repository by running:

export TRIVY_DB_REPOSITORY=public.ecr.aws/aquasecurity/trivy-db

Then try running the scanning command again.

If you notice, the Trivy job indicates exit code 1, which prevents other jobs from continuing the pipeline process.

Now, let’s move on to the next step, where we will learn how to manage this failure.
```
```markdown
Allow the Job Failure
Remember!

Except for DevSecOps-Box, every other machine closes after two hours, even if you are in the middle of the exercise
After two hours, in case of a 404, you need to refresh the exercise page and click on Start the Exercise button to continue working
We do not want to fail the builds/jobs/scan in DevSecOps Maturity Levels 1 and 2, as security tools spit a significant amount of false positives.

You can use the allow_failure tag to not fail the build even though the tool found issues.

trivy_scanning:
  stage: test
  script:
    - docker run --rm -v $(pwd):/src hysnsec/trivy fs . --exit-code 1 -f json -o trivy-report.json
  artifacts:
    paths: [trivy-report.json]
    when: always # What is this for?
    expire_in: one week
  allow_failure: true   #<--- allow the build to fail but don't mark it as such
After adding the allow_failure tag, the pipeline would look like the following.

Click anywhere to copy

image: docker:20.10

services:
  - docker:dind

stages:
  - build
  - test
  - release
  - preprod
  - integration
  - prod

trivy:
  stage: test
  script:
    - docker run --rm -v $(pwd):/src hysnsec/trivy fs . --exit-code 1 -f json -o trivy-report.json
  artifacts:
    paths: [trivy-report.json]
    when: always # What is this for?
    expire_in: one week
  allow_failure: true   #<--- allow the build to fail but don't mark it as such

test:
  stage: test
  image: python:3.6
  before_script:
   - pip3 install --upgrade virtualenv
  script:
   - virtualenv env
   - source env/bin/activate
   - pip install -r requirements.txt
   - python manage.py test taskManager

integration:
  stage: integration
  script:
    - echo "This is an integration step"
    - exit 1
  allow_failure: true # Even if the job fails, continue to the next stages

prod:
  stage: prod
  script:
    - echo "This is a deploy step."
  when: manual # Continuous Delivery

You will notice that the container_scanning job failed. However, it didn’t block other jobs from continuing further.

As discussed, any change to the repo kick starts the pipeline.

We can see the results of this pipeline by visiting https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/django-nv/pipelines.

Click on the appropriate job name to see the output.

Perfect! You’ll now see that the Trivy job processes the failure correctly and the rest of the tasks can proceed as expected.
```
```markdown
Software Component Analysis Using Sandworm
Download the Source Code
We will initially perform all exercises locally on DevSecOps-Box, so let’s begin the exercise.

First, we need to download the source code of the project from our Git repository.

```

git clone [https://gitlab.practical-devsecops.training/pdso/django.nv](https://gitlab.practical-devsecops.training/pdso/django.nv) webapp

```

Command Output
```

Cloning into 'webapp'...
warning: redirecting to [https://gitlab.practical-devsecops.training/pdso/django.nv.git/](https://gitlab.practical-devsecops.training/pdso/django.nv.git/)
remote: Enumerating objects: 299, done.
remote: Counting objects: 100% (71/71), done.
remote: Compressing objects: 100% (68/68), done.
remote: Total 299 (delta 34), reused 0 (delta 0), pack-reused 228
Receiving objects: 100% (299/299), 1.05 MiB | 29.16 MiB/s, done.
Resolving deltas: 100% (120/120), done.

```

Lets cd into the application code so we can scan the app.

```

cd webapp

```

We are now in the webapp directory.

Let’s move to the next step.
```
```markdown
Installing Sandworm
Sandworm is an audit tool designed to help assess security vulnerabilities and compliance levels within a system. It offers features to scan codebases for known vulnerabilities, analyze dependencies for issues, and ensure adherence to coding standards and best practices. Sandworm aids in identifying and addressing security flaws, thereby enhancing the overall security posture of a project or system.

Source: https://github.com/sandworm-hq/sandworm-audit

Sandworm is based on Node.js. Firstly, we need to install it.

```

curl -fsSL [https://deb.nodesource.com/gpgkey/nodesource-repo.gpg.key](https://deb.nodesource.com/gpgkey/nodesource-repo.gpg.key) | gpg --dearmor -o /etc/apt/keyrings/nodesource.gpg
NODE\_MAJOR=20
echo "deb \[signed-by=/etc/apt/keyrings/nodesource.gpg] [https://deb.nodesource.com/node\_\$NODE\_MAJOR.x](https://deb.nodesource.com/node_$NODE_MAJOR.x) nodistro main" | tee /etc/apt/sources.list.d/nodesource.list
apt update
apt install nodejs -y

```

Then we can install the Sandworm tool on the system to identify insecure libraries within our source code.

```

npm install -g @sandworm/audit

```

Let’s explore the options that sandworm provides.

```

sandworm --help

```

Command Output
```

Sandworm

Security & License Compliance For Your App's Dependencies 🪱

Commands:
Sandworm audit              Security & License Compliance For Your App's
Dependencies 🪱                           \[default]
Sandworm resolve \[issueId]  Security & License Compliance For Your App's
Dependencies 🪱

Options:
\--help                  Show help                                \[boolean]
-v, --version               Show version number                      \[boolean]
-o, --output-path           The path of the output directory, relative to the
application path    \[string] \[default: "sandworm"]
-d, --include-dev           Include dev dependencies\[boolean] \[default: false]
\--sv, --show-versions   Show package versions in chart names
\[boolean] \[default: false]
-p, --path                  The path to the application to audit      \[string]
\--pt, --package-type    The type of package to search for at the given
path                                      \[string]
\--md, --max-depth       Max depth to represent in charts          \[number]
\--ms, --min-severity    Min issue severity to represent in charts \[string]
\--lp, --license-policy  Custom license policy JSON string         \[string]
-f, --from                  Load data from "registry" or "disk"
\[string] \[default: "registry"]
\--fo, --fail-on         Fail policy JSON string   \[string] \[default: "\[]"]
-s, --summary               Print a summary of the audit results to the
console                  \[boolean] \[default: true]
\--root-vulnerabilites   Include vulnerabilities for the root project
\[boolean] \[default: false]
\--skip-license-issues   Skip scanning for license issues
\[boolean] \[default: false]
\--skip-meta-issues      Skip scanning for meta issues
\[boolean] \[default: false]
\--skip-tree             Don't output the dependency tree chart
\[boolean] \[default: false]
\--force-tree            Force build large dependency tree charts
\[boolean] \[default: false]
\--skip-treemap          Don't output the dependency treemap chart
\[boolean] \[default: false]
\--skip-csv              Don't output the dependency csv file
\[boolean] \[default: false]
\--skip-report           Don't output the report json file
\[boolean] \[default: false]
\--skip-all              Don't output any file   \[boolean] \[default: false]
\--show-tips             Show usage tips          \[boolean] \[default: true]

```

It provides two flags with different purposes, one is to scan the dependencies, and the other is to resolve them automatically based on the findings. We will explore this in the next step.

Let’s move to the next step.
```
```markdown
Run the Scanner
Let’s run the tool using the command below to identify vulnerabilities in third-party components.

```

sandworm audit

```

Command Output
```

Sandworm 🪱
Security and License Compliance Audit
⠋ Building dependency graph...
❌ Failed: No lockfile found

```

The output above indicates that sandworm couldn’t find any lockfile. A lock file describes the entire dependency tree, and most Software Component Analysis (SCA) tools that scan Node.js packages need to install the package first to obtain this kind of information.

So let’s install the dependencies and re-run the tool again.

```

npm install

```

Then, execute the same command for a basic scan.

```

sandworm audit

```

Command Output
```

Sandworm 🪱
Security and License Compliance Audit
✔ Built dependency graph
✔ Got vulnerabilities
✔ Scanned licenses
✔ Scanned issues
✔ Tree chart done
✔ Treemap chart done
✔ CSV done
✔ Report written to disk

⚠️ Identified 13 critical severity, 31 high severity, 7 moderate severity, 11 low severity issues
🔴 bson\@1.0.9 Deserialization of Untrusted Data in bson GHSA-v8w9-2789-6hhr

...\[SNIP]...

✨ Done

//
// 💡 Configure Sandworm with a config file
//    .sandworm.config.json
//    [https://docs.sandworm.dev/audit/configuration](https://docs.sandworm.dev/audit/configuration)
//

```

The output number may vary as it is dynamic.

The scanner’s result doesn’t provide JSON format output in the terminal, but it is stored in the sandworm directory within the current directory. You can verify this by using the ls command.

```

ls -l sandworm

```

Command Output
```

total 1824
-rw-r--r-- 1 root root 140055 Mar 11 22:53 [owasp-nodejs-goat@1.3.0-dependencies.csv](mailto:owasp-nodejs-goat@1.3.0-dependencies.csv)
-rw-r--r-- 1 root root 143559 Mar 11 22:53 [owasp-nodejs-goat@1.3.0-report.json](mailto:owasp-nodejs-goat@1.3.0-report.json)
-rw-r--r-- 1 root root 855904 Mar 11 22:53 [owasp-nodejs-goat@1.3.0-tree.svg](mailto:owasp-nodejs-goat@1.3.0-tree.svg)
-rw-r--r-- 1 root root 717029 Mar 11 22:53 [owasp-nodejs-goat@1.3.0-treemap.svg](mailto:owasp-nodejs-goat@1.3.0-treemap.svg)

```

The tool provides us with different output formats such as JSON and CSV for easier parsing when reading the report.

**Note**

As we have learned in the DevSecOps Gospel, we should always try to save the tool’s output in a machine-readable format. This output will enable us to parse it via script(s).

If you wish to ignore CSV, for example, please use the following command.

```

sandworm audit --skip-csv

```

Do not forget to remove the entire directory output using `rm -rf sandworm` before running the above command, as it contains the JSON output inside.

We have finished our first scan, but what’s next?

We will use the jq command to parse the output and filter the necessary information we need. This will enable us to see how many vulnerabilities were found and the severity of each finding, as sandworm does not provide us with a summary of these.

Let’s display all keys of the output file.

```

cat sandworm/owasp-[nodejs-goat@1.3.0-report.json](mailto:nodejs-goat@1.3.0-report.json) | jq 'keys'

```

**Note**

`keys` is used to retrieve the key names from a JSON document. It is simple and more efficient to extract information from within the JSON file instead of reading the entire file to identify the keys we need to filter.

Command Output
```

\[
"configuration",
"createdAt",
"dependencyVulnerabilities",
"errors",
"licenseIssues",
"licenseUsage",
"metaIssues",
"name",
"resolvedIssues",
"rootVulnerabilities",
"system",
"version"
]

```

There is an interesting key we can use, `dependencyVulnerabilities`.

```

cat sandworm/owasp-[nodejs-goat@1.3.0-report.json](mailto:nodejs-goat@1.3.0-report.json) | jq '.dependencyVulnerabilities'

```

Command Output
```

...\[SNIP]...

{
"findings": {
"sources": \[
{
"name": "underscore",
"version": "1.8.3",
"flags": {
"prod": true,
"dev": true
}
}
],
"affects": \[
{
"name": "underscore",
"version": "1.8.3",
"flags": {
"prod": true,
"dev": true
}
}
],
"rootDependencies": \[
{
"name": "underscore",
"version": "1.8.3",
"flags": {
"prod": true,
"dev": true
}
}
],
"paths": \[
"underscore"
]
},
"source": 1095097,
"githubAdvisoryId": "GHSA-cf4h-3jhx-xvhq",
"name": "underscore",
"title": "Arbitrary Code Execution in underscore",
"type": "vulnerability",
"url": "[https://github.com/advisories/GHSA-cf4h-3jhx-xvhq](https://github.com/advisories/GHSA-cf4h-3jhx-xvhq)",
"severity": "critical",
"range": ">=1.3.2 <1.12.1",
"recommendation": "Update grunt-retire to 1.0.9"
}
]

```

Now, the output is better, allowing us to extract more specific information, such as the number of each severity type in the findings.

Let’s check how many vulnerabilities are found by filtering the `severity` field.

```

cat sandworm/owasp-[nodejs-goat@1.3.0-report.json](mailto:nodejs-goat@1.3.0-report.json) | jq '.dependencyVulnerabilities | length'

```

Command Output
```

21

```

**Remember**

The total findings are not sorted out because the same component can have multiple vulnerabilities, resulting in duplicated component names. You will observe this in the next command.

How can we ensure the above statement is accurate?

We have provided the command you can use to check which components are vulnerable and their severity.

```

cat sandworm/owasp-[nodejs-goat@1.3.0-report.json](mailto:nodejs-goat@1.3.0-report.json) | jq '.dependencyVulnerabilities\[] | "(.name) (.severity)"'

```

Command Output
```

"braces low"
"braces low"
"bson moderate"
"bson critical"
"debug moderate"
"debug high"
"glob-parent high"
"helmet-csp moderate"
"marked high"
"marked high"
"minimist moderate"
"minimist moderate"
"minimist critical"
"minimist critical"
"mongodb high"
"ms moderate"
"nconf high"
"swig high"
"timespan high"
"uglify-js high"
"underscore critical"

```

You can explore more advanced uses of `jq`, but for now, we will use the above output.

Let’s move to the next step.
```
```markdown
Configure the Exit Codes
We have previously discussed the importance of exit codes in certain exercises, especially when integrating DevSecOps into your CI/CD pipeline. Now, we will explore the tool to ensure it can appropriately fail the pipeline or provide an exit code indicating the presence of vulnerabilities with specific severity levels.

Let’s check the audit flag to see if it provides the argument or not.

```

sandworm audit --help

```

Command Output
```

Sandworm audit

Security & License Compliance For Your App's Dependencies 🪱

Options:
\--help                  Show help                                \[boolean]
-v, --version               Show version number                      \[boolean]
-o, --output-path           The path of the output directory, relative to the
application path    \[string] \[default: "sandworm"]
-d, --include-dev           Include dev dependencies\[boolean] \[default: false]
\--sv, --show-versions   Show package versions in chart names
\[boolean] \[default: false]
-p, --path                  The path to the application to audit      \[string]
\--pt, --package-type    The type of package to search for at the given
path                                      \[string]
\--md, --max-depth       Max depth to represent in charts          \[number]
\--ms, --min-severity    Min issue severity to represent in charts \[string]
\--lp, --license-policy  Custom license policy JSON string         \[string]
-f, --from                  Load data from "registry" or "disk"
\[string] \[default: "registry"]
\--fo, --fail-on         Fail policy JSON string   \[string] \[default: "\[]"]
-s, --summary               Print a summary of the audit results to the
console                  \[boolean] \[default: true]
\--root-vulnerabilites   Include vulnerabilities for the root project
\[boolean] \[default: false]
\--skip-license-issues   Skip scanning for license issues
\[boolean] \[default: false]
\--skip-meta-issues      Skip scanning for meta issues
\[boolean] \[default: false]
\--skip-tree             Don't output the dependency tree chart
\[boolean] \[default: false]
\--force-tree            Force build large dependency tree charts
\[boolean] \[default: false]
\--skip-treemap          Don't output the dependency treemap chart
\[boolean] \[default: false]
\--skip-csv              Don't output the dependency csv file
\[boolean] \[default: false]
\--skip-report           Don't output the report json file
\[boolean] \[default: false]
\--skip-all              Don't output any file   \[boolean] \[default: false]
\--show-tips             Show usage tips          \[boolean] \[default: true]

```

The --fail-on flag seems interesting to explore. It describes the fail policy, indicating that we might need to create the configuration file or define it directly.

After some experimentation, we found the documentation at https://web.archive.org/web/20250419010940/https://docs.sandworm.dev/audit/fail-policies to configure the tool to fail on specific severity levels.

Before that, let’s rerun the tool again to check the default exit code.

```

sandworm audit

```

Then, use the command below to check the exit code after the tool has executed.

```

echo \$?

```

Command Output
```

0

```

In the Sandworm documentation, fail conditions can be defined in different types, such as dependencies, licenses, meta, or even all that meet specific criteria.

We will use dependencies as conditions and fail the build when high severity issues are found. The final command will be as follows:

```

sandworm-audit --fail-on='\["dependencies.high"]'

```

Check the exit code once again.

```

echo \$?

```

Command Output
```

1

```

Perfect, it fails the build when the output is non-zero, which is 1 or a higher value, as indicated below:

exit 0: Indicates that the command or program executed successfully without any errors.
exit 1 or non-zero : Catch-all exit code for a variety of general errors.

In this case, error refers to vulnerabilities found by the security tool. If you wish to understand more about exit codes, kindly refer to the Working With Linux Exit Code exercise.
```
```markdown
Addressing Issues With Sandworm
Download the Source Code
We will initially perform all exercises locally on DevSecOps-Box, so let’s begin the exercise.

First, we need to download the source code of the project from our Git repository.

```

git clone [https://gitlab.practical-devsecops.training/pdso/django.nv](https://gitlab.practical-devsecops.training/pdso/django.nv) webapp

```

Command Output
```

Cloning into 'webapp'...
warning: redirecting to [https://gitlab.practical-devsecops.training/pdso/django.nv.git/](https://gitlab.practical-devsecops.training/pdso/django.nv.git/)
remote: Enumerating objects: 299, done.
remote: Counting objects: 100% (71/71), done.
remote: Compressing objects: 100% (68/68), done.
remote: Total 299 (delta 34), reused 0 (delta 0), pack-reused 228
Receiving objects: 100% (299/299), 1.05 MiB | 29.16 MiB/s, done.
Resolving deltas: 100% (120/120), done.

```

Lets cd into the application code so we can scan the app.

```

cd webapp

```

We are now in the webapp directory.

Let’s move to the next step.
```
```markdown
Installing Sandworm
Sandworm is an audit tool designed to help assess security vulnerabilities and compliance levels within a system. It offers features to scan codebases for known vulnerabilities, analyze dependencies for issues, and ensure adherence to coding standards and best practices. Sandworm aids in identifying and addressing security flaws, thereby enhancing the overall security posture of a project or system.

Source: https://github.com/sandworm-hq/sandworm-audit

Sandworm is based on Node.js. Firstly, we need to install it.

```

curl -fsSL [https://deb.nodesource.com/gpgkey/nodesource-repo.gpg.key](https://deb.nodesource.com/gpgkey/nodesource-repo.gpg.key) | gpg --dearmor -o /etc/apt/keyrings/nodesource.gpg
NODE\_MAJOR=20
echo "deb \[signed-by=/etc/apt/keyrings/nodesource.gpg] [https://deb.nodesource.com/node\_\$NODE\_MAJOR.x](https://deb.nodesource.com/node_$NODE_MAJOR.x) nodistro main" | tee /etc/apt/sources.list.d/nodesource.list
apt update
apt install nodejs -y

```

Then we can install the Sandworm tool on the system to identify insecure libraries within our source code.

```

npm install -g @sandworm/audit

```

Let’s explore the options that sandworm provides.

```

sandworm resolve --help

```

Command Output
```

Sandworm resolve \[issueId]

Security & License Compliance For Your App's Dependencies 🪱

Options:
\--help                Show help                                  \[boolean]
-v, --version             Show version number                        \[boolean]
\--issueId             The unique id of the issue to resolve
\[string] \[required]
-p, --path                The path to the application to audit        \[string]
\--pt, --package-type  The type of package to search for at the given path
\[string]
-o, --output-path         The path of the audit output directory, relative to
the application path  \[string] \[default: "sandworm"]

```

In the previous exercise, we learned how to use the audit flag along with the exit code. This exercise will cover how to utilize the resolve flag to automatically patch the findings based on the specified issue we aim to fix.

Let’s move to the next step.
```
```markdown
Run the Scanner
Let’s install the dependencies before run the tool.

```

npm install

```

Next, run the tool using the command below to identify vulnerabilities in third-party components.

```

sandworm audit

```

Command Output
```

Sandworm 🪱
Security and License Compliance Audit
✔ Built dependency graph
✔ Got vulnerabilities
✔ Scanned licenses
✔ Scanned issues
✔ Tree chart done
✔ Treemap chart done
✔ CSV done
✔ Report written to disk

⚠️ Identified 13 critical severity, 31 high severity, 7 moderate severity, 11 low severity issues
🔴 bson\@1.0.9 Deserialization of Untrusted Data in bson GHSA-v8w9-2789-6hhr

...\[SNIP]...

```
In the above scan results, we not only identify vulnerabilities but also license issues. For this exercise, let’s focus on fixing the third-party library vulnerabilities and ignore the license issues.

```

sandworm audit --skip-license-issues

```

Command Output
```

Sandworm 🪱
Security and License Compliance Audit
✔ Built dependency graph
✔ Got vulnerabilities
✔ Scanned issues
✔ Tree chart done
✔ Treemap chart done
✔ CSV done
✔ Report written to disk

⚠️ Identified 4 critical severity, 22 high severity, 7 moderate severity, 2 low severity issues
🔴 bson\@1.0.9 Deserialization of Untrusted Data in bson GHSA-v8w9-2789-6hhr
🔴 minimist\@0.0.10 Prototype Pollution in minimist GHSA-xvch-5gv4-984h
🔴 minimist\@1.2.0 Prototype Pollution in minimist GHSA-xvch-5gv4-984h
🔴 underscore\@1.8.3 Arbitrary Code Execution in underscore GHSA-cf4h-3jhx-xvhq

...\[SNIP]...

```
The output number may vary as it is dynamic.

The output of the second scan is less compared to the first scan.

To address the issue using the sandworm resolve command, we require something called an issueId. How can we obtain this?

You can refer to the output in the terminal, or you can read the output file in JSON/CSV format. If you choose to read the output file, use jq as a tool to parse the JSON file effortlessly. We have provided you with the direct syntax.

```

cat sandworm/owasp-[nodejs-goat@1.3.0-report.json](mailto:nodejs-goat@1.3.0-report.json) | jq '.dependencyVulnerabilities\[] | select(.severity == "critical") | .githubAdvisoryId'

```

Command Output
```

"GHSA-v8w9-2789-6hhr"
"GHSA-xvch-5gv4-984h"
"GHSA-xvch-5gv4-984h"
"GHSA-cf4h-3jhx-xvhq"

```

Let’s move to the next step.
```
```markdown
Fixing the Issues
In the previous steps, we identified four findings with critical severity. Now, let’s address them using sandworm resolve command without having to determine which secure version to use from the npm registry, as it could be time-consuming.

Let’s begin fixing the issue using the command below:

```

sandworm resolve --issueId GHSA-v8w9-2789-6hhr

```

Command Output
```

Sandworm 🪱
Security and License Compliance Audit
Resolving issue GHSA-v8w9-2789-6hhr:
🔴 Deserialization of Untrusted Data in bson

✔ Select paths to resolve › mongodb>mongodb-core>bson
✔ Enter resolution notes: … fix the issue

✨ Done

```
The above command will prompt as follows:

Command Output
```

? Select paths to resolve

```
You can type a to select all, and the ordered list will change to green color. Then press Enter.

At the end, it will ask for resolution notes. You can write fix the issue and press Enter.

Note

The issueId flag does not support multiple values. Therefore, you need to execute the command one by one for each issue ID.

After all critical issues have been resolved, re-run the tool to verify the results.

```

sandworm audit --skip-license-issues

```

Command Output
```

Sandworm 🪱
Security and License Compliance Audit
✔ Built dependency graph
✔ Got vulnerabilities
✔ Scanned issues
✔ Tree chart done
✔ Treemap chart done
✔ CSV done
✔ Report written to disk

🙌 4 issues already resolved
⚠️ Identified 22 high severity, 7 moderate severity, 2 low severity issues

...\[SNIP]...

```
Remember

After upgrading the version, it is essential to verify that our application is functioning correctly. Keep in mind that these fixes might potentially break your application.

Cool! we fixed the issues found in our application’s components.

Note

You need to talk to the development team to see if they can upgrade the package to a specific version without breaking the application. The release note explains why the new version is safer than the previous one, and how to determine what the developer needs in the relevant version based on the release note.

For best practices, we must ensure that the components are free from any critical issues currently. If you wish to address high issues, you can proceed with the experiment.
```
```markdown
Software Component Analysis Using pip-licenses
In this scenario, you will learn how to install and run pip-licenses to check the licenses of Python packages in your project.

You will learn how to:

- Install pip-licenses  
- Run pip-licenses to scan the dependencies licenses  
- Generate a report in different formats like Markdown, HTML, JSON  

---

### Installing pip-licenses
pip-licenses is a CLI tool for checking the software licenses of Python packages installed with pip. It helps development teams ensure license compliance in their projects.

pip-licenses scans your Python environment and identifies the licenses of all installed packages. It was inspired by the composer licenses command in PHP’s Composer package manager.

First, we clone the sample application that we will use to scan the dependencies licenses.

```

git clone [https://gitlab.practical-devsecops.training/pdso/django.nv](https://gitlab.practical-devsecops.training/pdso/django.nv) webapp

```

Let’s change into the application directory so we can scan the app.

```

cd webapp

```

Next, let’s install the pip-licenses tool on the system to perform license analysis of Python dependencies.

```

pip install pip-licenses==5.0.0

```

Command Output
```

...\[SNIP]...
Installing collected packages: wcwidth, tomli, prettytable, pip-licenses
Successfully installed pip-licenses-5.0.0 prettytable-3.16.0 tomli-2.2.1 wcwidth-0.2.13

```

We have successfully installed pip-licenses scanner, let’s explore the functionality it provides us.

```

pip-licenses --help

```

Command Output
```

usage: pip-licenses \[-h] \[-v] \[--python PYTHON\_EXEC] \[--from SOURCE] \[-o COL] \[-f STYLE] \[--summary]
\[--output-file OUTPUT\_FILE] \[-i PKG \[PKG ...]] \[-p PKG \[PKG ...]] \[-s] \[-a]
\[--with-maintainers] \[-u] \[-d] \[-nv] \[-l] \[--no-license-path] \[--with-notice-file]
\[--filter-strings] \[--filter-code-page CODE] \[--fail-on FAIL\_ON] \[--allow-only ALLOW\_ONLY]
\[--partial-match]

Dump the software license list of Python packages installed with pip.

options:
-h, --help                  show this help message and exit
-v, --version               show program's version number and exit

```

**Common options:**
```

\--python PYTHON\_EXEC         path to python executable to search distributions from
\--from SOURCE                where to find license information ("meta", "classifier, "mixed", "all")
-o COL, --order COL          order by column ("name", "license", "author", "url")
-f STYLE, --format STYLE     dump as set format style ("plain", "markdown", "html", "json", etc.)
\--summary                    dump summary of each license
\--output-file OUTPUT\_FILE    save license list to file
-i, --ignore-packages        ignore package names
-p, --packages               only include selected packages in output

```

**Format options:**
```

-s, --with-system            include system packages
-a, --with-authors           include package authors
\--with-maintainers           include package maintainers
-u, --with-urls              include package urls
-d, --with-description       include package description
-nv, --no-version            dump without package version
-l, --with-license-file      include license file path and contents (JSON output)
\--with-notice-file           include notice file path and contents

```

**Verify options:**
```

\--fail-on FAIL\_ON            fail (exit with code 1) on first occurrence of listed licenses
\--allow-only ALLOW\_ONLY      fail (exit with code 1) if license is not in allowed list
\--partial-match              enables partial matching for --allow-only/--fail-on

```

---

### Common Challenges When Using pip-licenses
When scanning your Python dependencies with pip-licenses, you might encounter several issues:

- Some packages may not provide proper license metadata, resulting in **UNKNOWN** licenses in your report that require manual investigation.  
- You might discover **license conflicts** that could create legal issues, especially when combining GPL-licensed packages with proprietary code.  
- The package you directly depend on might pull in **transitive dependencies** with problematic licenses that aren’t immediately obvious.  
- The `--fail-on` and `--allow-only` flags might flag legitimate packages due to **license naming inconsistencies** (e.g., “MIT” vs. “MIT License”).  

---

Command Output
```

# Example of missing license information

\$ pip-licenses
Name          Version  License
some-package  1.2.3    UNKNOWN

# Example of license incompatibility error

\$ pip-licenses --fail-on GPL
Error: Package 'some-dependency' has a GPL license which is not allowed

```

---

Let’s move to the next step to see how we can use pip-licenses to scan the licenses of Python dependencies.
```
```markdown
Run pip-licenses on Python dependencies
Let’s run the pip-licenses tool to scan the dependencies licenses and generate a report.

First, let’s check the output of pip-licenses:

```

pip-licenses

```

Command Output
```

Jinja2              3.1.5          BSD License
MarkupSafe          3.0.2          BSD License
PyGObject           3.42.1         GNU Lesser General Public License v2 or later (LGPLv2+)
PyJWT               2.3.0          MIT License
PyYAML              6.0.2          MIT License
SecretStorage       3.3.1          BSD License
ansible             8.7.0          GNU General Public License v3 or later (GPLv3+)
ansible-core        2.15.13        GNU General Public License v3 or later (GPLv3+)
blinker             1.4            MIT License
certifi             2024.12.14     Mozilla Public License 2.0 (MPL 2.0)
charset-normalizer  3.4.1          MIT License
cryptography        3.4.8          Apache Software License; BSD License
dbus-python         1.2.18         MIT License
distro              1.7.0          Apache Software License
httplib2            0.20.2         MIT License
idna                3.10           BSD License
importlib-metadata  4.6.4          Apache Software License
jeepney             0.7.1          MIT License
keyring             23.5.0         MIT License; Python Software Foundation License
launchpadlib        1.10.16        GNU Library or Lesser General Public License (LGPL)
lazr.restfulclient  0.14.4         GNU Library or Lesser General Public License (LGPL)
lazr.uri            1.0.6          GNU Library or Lesser General Public License (LGPL)
more-itertools      8.10.0         MIT License
oauthlib            3.2.0          BSD License
packaging           24.2           Apache Software License; BSD License
pyparsing           2.4.7          MIT License
python-apt          2.4.0+ubuntu4  GNU GPL
requests            2.32.3         Apache Software License
resolvelib          1.0.1          ISC License (ISCL)
six                 1.16.0         MIT License
urllib3             2.3.0          MIT License
wadllib             1.3.6          GNU Library or Lesser General Public License (LGPL)
zipp                1.0.0          MIT License

```

Good, we have the report of pip-licenses. From the above output, we can see that pip-licenses is able to detect the license of the packages — for example, **Jinja2** is under **BSD License**.

---

In the next step, we will explore how to format this report and add more details.

Let’s move to the next step.
```
```markdown
Formatting Reports and Extracting License Details
In this step, we will explore multiple output formats for pip-licenses reports and learn how to include additional package information such as authors, URLs, and license summaries.

---

### Generate a report in Markdown format
Let’s generate a report in Markdown format using the following command:

```

pip-licenses --format=markdown

```

Command Output
```

| Name               | Version    | License                                                 |
| ------------------ | ---------- | ------------------------------------------------------- |
| Jinja2             | 3.1.5      | BSD License                                             |
| MarkupSafe         | 3.0.2      | BSD License                                             |
| PyGObject          | 3.42.1     | GNU Lesser General Public License v2 or later (LGPLv2+) |
| PyJWT              | 2.3.0      | MIT License                                             |
| PyYAML             | 6.0.2      | MIT License                                             |
| SecretStorage      | 3.3.1      | BSD License                                             |
| ansible            | 8.7.0      | GNU General Public License v3 or later (GPLv3+)         |
| ansible-core       | 2.15.13    | GNU General Public License v3 or later (GPLv3+)         |
| blinker            | 1.4        | MIT License                                             |
| certifi            | 2024.12.14 | Mozilla Public License 2.0 (MPL 2.0)                    |
| charset-normalizer | 3.4.1      | MIT License                                             |
| cryptography       | 3.4.8      | Apache Software License; BSD License                    |
| dbus-python        | 1.2.18     | MIT License                                             |
| distro             | 1.7.0      | Apache Software License                                 |
| httplib2           | 0.20.2     | MIT License                                             |
| idna               | 3.10       | BSD License                                             |
| importlib-metadata | 4.6.4      | Apache Software License                                 |
| jeepney            | 0.7.1      | MIT License                                             |
| keyring            | 23.5.0     | MIT License; Python Software Foundation License         |
| ...\[SNIP]...      |            |                                                         |

```

---

### Generate a report in HTML format
```

pip-licenses --format=html

````

Command Output
```html
<table>
    <thead>
        <tr>
            <th>Name</th>
            <th>Version</th>
            <th>License</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Jinja2</td>
            <td>3.1.5</td>
            <td>BSD License</td>
        </tr>
        <tr>
            <td>MarkupSafe</td>
            <td>3.0.2</td>
            <td>BSD License</td>
        </tr>
        <tr>
            <td>PyGObject</td>
            <td>3.42.1</td>
            <td>GNU Lesser General Public License v2 or later (LGPLv2+)</td>
        </tr>
        ...[SNIP]...
    </tbody>
</table>
````

---

### Generate a report in JSON format

```
pip-licenses --format=json
```

Command Output

```json
[
  {
    "License": "BSD License",
    "Name": "Jinja2",
    "Version": "3.1.5"
  },
  {
    "License": "BSD License",
    "Name": "MarkupSafe",
    "Version": "3.0.2"
  },
  {
    "License": "GNU Lesser General Public License v2 or later (LGPLv2+)",
    "Name": "PyGObject",
    "Version": "3.42.1"
  },
  {
    "License": "MIT License",
    "Name": "PyJWT",
    "Version": "2.3.0"
  },
  ...[SNIP]...
]
```

Great, we already generated reports in **3 different formats** (Markdown, HTML and JSON). You can generate reports in other formats supported by pip-licenses such as CSV, RST, or Confluence.

---

### Add more details with Author

We can include the author in the report by using the `--with-authors` option.

```
pip-licenses --with-authors
```

Command Output

```
 Name                Version        License                                                  Author
 Jinja2              3.1.5          BSD License                                              UNKNOWN
 MarkupSafe          3.0.2          BSD License                                              UNKNOWN
 PyGObject           3.42.1         GNU Lesser General Public License v2 or later (LGPLv2+)  James Henstridge
 PyJWT               2.3.0          MIT License                                              Jose Padilla
 PyYAML              6.0.2          MIT License                                              Kirill Simonov
 SecretStorage       3.3.1          BSD License                                              Dmitry Shachnev
 ansible             8.7.0          GNU General Public License v3 or later (GPLv3+)          Ansible, Inc.
 ...[SNIP]...
```

---

### Add more details with URL

We can include the URL in the report by using the `--with-urls` option.

```
pip-licenses --with-urls
```

Command Output

```
 Name       Version  License          URL
 Jinja2     3.1.5    BSD License      https://github.com/pallets/jinja/
 MarkupSafe 3.0.2    BSD License      https://github.com/pallets/markupsafe/
 PyJWT      2.3.0    MIT License      https://github.com/jpadilla/pyjwt
 PyYAML     6.0.2    MIT License      https://pyyaml.org/
 ...[SNIP]...
```

---

### Generate a summary of licenses

We can also generate a summary of the licenses used in our project. This helps us quickly see license distribution.

```
pip-licenses --summary
```

Command Output

```
 3      Apache Software License
 2      Apache Software License; BSD License
 5      BSD License
 1      GNU GPL
 2      GNU General Public License v3 or later (GPLv3+)
 1      GNU Lesser General Public License v2 or later (LGPLv2+)
 4      GNU Library or Lesser General Public License (LGPL)
 1      ISC License (ISCL)
 12     MIT License
 1      MIT License; Python Software Foundation License
 1      Mozilla Public License 2.0 (MPL 2.0)
```

---

✅ That’s all for now. We have learned how to:

* Generate reports in different formats
* Add more details to the pip-licenses report (authors, URLs)
* Generate a summary of licenses used in the project

```
```
```markdown
Managing Allowed Licenses
In this step, you’ll learn how to enforce license compliance by specifying a whitelist of allowed licenses using pip-licenses. This is a crucial feature for ensuring your project only uses dependencies with licenses that align with your organization’s policies.

---

### Why Use an Allowed License List?
Controlling the licenses of the software components you use is a key aspect of **software supply chain management** and **legal compliance**. Here’s why organizations implement allow-lists for licenses:

- **Legal and Policy Compliance**  
  Many organizations have strict policies about the types of open-source licenses they can accept.  
  For example, some licenses (like the **GNU General Public License - GPL**) have “viral” characteristics, meaning that if you use a GPL-licensed component in your software, your entire software might also need to be licensed under GPL, potentially forcing you to open-source proprietary code. An allow-list helps prevent the accidental inclusion of such licenses.

- **Risk Mitigation**  
  Certain licenses might come with **patent clauses** or other terms that could introduce legal risks or complexities for your project or organization.

- **Intellectual Property Protection**  
  By ensuring that all components use approved licenses, companies can better protect their own intellectual property and avoid unintended obligations.

- **Standardization**  
  Maintaining an allow-list promotes consistency in how open source software is consumed across different projects within an organization.

⚠️ Using **GPL** or **LGPL** licensed code can be problematic, especially for businesses, due to the viral nature of these licenses and the legal gray areas when distributing software that incorporates them. (Source: *Human Who Codes*)

By using an allow-list, you proactively manage these risks and ensure your project adheres to acceptable licensing standards.

---

### Using the `--allow-only` option
The `--allow-only` option is used to specify a whitelist of acceptable licenses.  
It ensures that only packages with licenses from this list are permitted.  

If any package is found with a license **not** in the allow-list, `pip-licenses` will fail and exit with a **non-zero status code**.

Example:  
Let’s say your project policy only allows packages with **MIT License** or **BSD License**.  
You can enforce this using the following command:

```

pip-licenses --allow-only "MIT License;BSD License"

```

Command Output
```

license Mozilla Public License 2.0 (MPL 2.0) not in allow-only licenses was found for package certifi:2024.12.14

```

If a package like **certifi** (which often has MPL 2.0) is present and its license is not in the allow-list, the command will fail and output an error message similar to the one above.

---

### Verify Exit Code
This behavior makes it a strong control for ensuring license compliance, as it will halt any process (like a CI/CD pipeline) that relies on its success if a non-approved license is detected.  

You can verify this non-zero exit code in your terminal. For example, after the command fails:

```

echo \$?

```

Command Output
```

1

```

---

### NOTE
- The **license names must exactly match** the names `pip-licenses` recognizes.  
- You might need to run `pip-licenses` without filters first to see the exact license names for your dependencies.

---

✅ This feature is invaluable for maintaining a compliant and secure software supply chain, especially when integrated into **CI/CD pipelines**.
```
```markdown
Conclusion
In this hands-on exercise, we have successfully learned how to use **pip-licenses** to analyze and manage Python package licenses in your projects.  
This important component analysis tool helps ensure your software remains **compliant with license requirements**.

---

### What We Achieved
- Discovered pip-licenses as a handy **command-line tool** for scanning Python dependencies and pulling up license information.  
- Installed and ran our **first report**.  
- Explored how to **present results** in various formats like **Markdown, HTML, or JSON** so different teams can use the data.  
- Dug deeper by adding useful details like **author names** and **project URLs**.  
- Learned to generate a **clean summary** for a quick snapshot of license compliance.  
- Understood how to enforce an **allow-list of licenses** to ensure only compliant dependencies are used.  

---

### Keep Exploring!
We’ve covered the basics, but there’s much more to discover with pip-licenses.  
We encourage you to:

- Experiment with the `--packages` and `--ignore-packages` options to **focus on specific dependencies**.  
- Explore more output formats like **CSV** or **RST** for different documentation needs.  
- Check the official documentation for advanced features:  
  👉 https://github.com/raimon49/pip-licenses  

---

✅ The best way to learn is by experimenting with your own projects.  
Try running pip-licenses on different Python applications to see what licenses they depend on!
```
```markdown
A Simple CI/CD Pipeline
Considering your DevOps team created a simple CI pipeline with the following contents. Please add the Safety scan to the below Gitlab CI Script.

Click anywhere to copy

image: docker:20.10  # To run all jobs in this pipeline, use the latest docker image

services:
  - docker:dind       # To run all jobs in this pipeline, use a docker image that contains a docker daemon running inside (dind - docker in docker). Reference: https://forum.gitlab.com/t/why-services-docker-dind-is-needed-while-already-having-image-docker/43534

stages:
  - build
  - test
  - release
  - preprod
  - integration
  - prod

build:
  stage: build
  image: python:3.6
  before_script:
   - pip3 install --upgrade virtualenv
  script:
   - virtualenv env                       # Create a virtual environment for the python application
   - source env/bin/activate              # Activate the virtual environment
   - pip install -r requirements.txt      # Install the required third party packages as defined in requirements.txt
   - python manage.py check               # Run checks to ensure the application is working fine

test:
  stage: test
  image: python:3.6
  before_script:
   - pip3 install --upgrade virtualenv
  script:
   - virtualenv env
   - source env/bin/activate
   - pip install -r requirements.txt
   - python manage.py test taskManager

# --- Added Safety (OAST/SCA) job ---
safety:
  stage: test
  image: python:3.6
  before_script:
    - pip3 install --upgrade pip
    - pip3 install safety
  script:
    - safety check -r requirements.txt --full-report --json --output safety-report.json --exit-code 1
  artifacts:
    paths: [safety-report.json]
    when: always # What is this for?
    expire_in: one week
# -----------------------------------

There are some jobs in the pipeline, and we need to embed the OAST tool in this pipeline.

Let’s login into GitLab using the following details and execute this pipeline once again.

GitLab CI/CD Machine
Name	Value
URL	https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/django-nv/-/blob/main/.gitlab-ci.yml
Username	root
Password	pdso-training
Next, we need to create a CI/CD pipeline by replacing the .gitlab-ci.yml file content with the above CI script. Click on the Edit button to replace the content (use Control+A and Control+V).

Save changes to the file using the Commit changes button.

Verify the pipeline run
As soon as a change is made to the repository, the pipeline starts executing the jobs.

We can see the results of this pipeline by visiting https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/django-nv/pipelines.

Click on the appropriate job name to see the output.

Let’s move to the next step.
```
```markdown
Embed pip-licenses in CI/CD pipeline
We can embed the pip-licenses tool in our CI/CD pipeline.

However, do remember that you need run commands manually in a local system like devsecops box before embedding commands for automated execution in CI/CD pipelines.

Manually testing commands in a local environment like devsecops box helps in easy troubleshooting.

Do you wonder which stage this job should go into?

Most of the code (up to 95%) in any software project is open-source/third-party components. It makes sense to perform SCA scans before static analysis.

Click anywhere to copy

image: docker:20.10  # To run all jobs in this pipeline, use the latest docker image

services:
  - docker:dind       # To run all jobs in this pipeline, use a docker image that contains a docker daemon running inside (dind - docker in docker). Reference: https://forum.gitlab.com/t/why-services-docker-dind-is-needed-while-already-having-image-docker/43534

stages:
  - build
  - test
  - release
  - preprod
  - integration
  - prod

build:
  stage: build
  image: python:3.10
  before_script:
   - pip3 install --upgrade virtualenv
  script:
   - virtualenv env                       # Create a virtual environment for the python application
   - source env/bin/activate              # Activate the virtual environment
   - pip install -r requirements.txt      # Install the required third party packages as defined in requirements.txt
   - python manage.py check               # Run checks to ensure the application is working fine

test:
  stage: test
  image: python:3.10
  before_script:
   - pip3 install --upgrade virtualenv
  script:
   - virtualenv env
   - source env/bin/activate
   - pip install -r requirements.txt
   - python manage.py test taskManager

license:
  stage: test
  image: python:3.10
  before_script:
   - pip3 install --upgrade virtualenv
  script:
    - virtualenv env
    - source env/bin/activate
    - pip install -r requirements.txt
    - pip install pip-licenses==5.0.0
    - pip-licenses --allow-only "MIT License;BSD License" --format=json --output-file=license-results.json
  artifacts:
    paths: [license-results.json]
    when: always # What does this do?

integration:
  stage: integration
  script:
    - echo "This is an integration step."
    - exit 1
  allow_failure: true # Even if the job fails, continue to the next stages

prod:
  stage: prod
  script:
    - echo "This is a deploy step."
  when: manual # Continuous Delivery

Instead of manually removing the container after the scan, we can use the --rm option so the container can clean up after itself.

Copy the above CI script and add it to the .gitlab.ci.yml file on Gitlab repo at https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/django-nv/-/blob/main/.gitlab-ci.yml

Do not forget to click on the “Commit Changes” button to save the file.

As discussed, any change to the repo kick starts the pipeline.

Note

We’ve discussed different methods of using the tool:

Native Installation:
Directly installing the tool on the system.
Package Manager or Binary:
Installing via a package manager or using the binary file.
Docker:
Running the tool within a Docker container.
In summary:

All methods are suitable for CI/CD integration.
Docker is recommended for CI/CD as it operates smoothly without dependencies.
Using the binary file is efficient, avoiding additional dependencies.
Ultimately, you can choose either method based on your specific situation.

We can see the results of this pipeline by visiting https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/django-nv/pipelines.

Click on the appropriate job name to see the output.

You will notice that the license job is failing. Why? you can see the output in the job logs. and you will see the following error:

Command Output
license UNKNOWN not in allow-only licenses was found for package typing_extensions:4.14.0
This means that the typing_extensions package is not in the allow-only licenses. Because the license is UNKNOWN.

Note

If none of the license in the allow list match, the check will fail and return an non-zero exit code. https://github.com/raimon49/pip-licenses?tab=readme-ov-file#option-allow-only

Did you remember we already discussed about non-zero exit code?
In the next step, you will learn the need to not fail the builds.
```
```markdown
Allow the Job Failure
Remember! Except for DevSecOps-Box, every other machine closes after two hours, even if you are in the middle of the exercise. You need to refresh the exercise page and click on Start the Exercise button to continue working on it.

You do not want to fail the builds in DevSecOps Maturity Levels 1 and 2. If a security tool fails a build upon security findings, you would want to allow it to fail and not block the pipeline as there would be false positives in the results.

You can use the allow_failure tag to “not fail the build” even though the tool found issues.

license:
  stage: test
  image: python:3.10
  before_script:
   - pip3 install --upgrade virtualenv
  script:
    - virtualenv env
    - source env/bin/activate
    - pip install -r requirements.txt
    - pip install pip-licenses==5.0.0
    - pip-licenses --allow-only "MIT License;BSD License" --format=json --output-file=license-results.json
  artifacts:
    paths: [license-results.json]
    when: always        # What does this do?
  allow_failure: true   # <--- allow the build to fail but don't mark it as such
The pipeline would look like the following.

Click anywhere to copy

image: docker:20.10  # To run all jobs in this pipeline, use the latest docker image

services:
  - docker:dind       # To run all jobs in this pipeline, use a docker image that contains a docker daemon running inside (dind - docker in docker). Reference: https://forum.gitlab.com/t/why-services-docker-dind-is-needed-while-already-having-image-docker/43534

stages:
  - build
  - test
  - release
  - preprod
  - integration
  - prod

build:
  stage: build
  image: python:3.10
  before_script:
   - pip3 install --upgrade virtualenv
  script:
   - virtualenv env                       # Create a virtual environment for the python application
   - source env/bin/activate              # Activate the virtual environment
   - pip install -r requirements.txt      # Install the required third party packages as defined in requirements.txt
   - python manage.py check               # Run checks to ensure the application is working fine

test:
  stage: test
  image: python:3.10
  before_script:
   - pip3 install --upgrade virtualenv
  script:
   - virtualenv env
   - source env/bin/activate
   - pip install -r requirements.txt
   - python manage.py test taskManager

license:
  stage: test
  image: python:3.10
  before_script:
   - pip3 install --upgrade virtualenv
  script:
    - virtualenv env
    - source env/bin/activate
    - pip install -r requirements.txt
    - pip install pip-licenses==5.0.0
    - pip-licenses --allow-only "MIT License;BSD License" --format=json --output-file=license-results.json
  artifacts:
    paths: [license-results.json]
    when: always        # What does this do?
  allow_failure: true   # <--- allow the build to fail but don't mark it as such

integration:
  stage: integration
  script:
    - echo "This is an integration step."
    - exit 1
  allow_failure: true # Even if the job fails, continue to the next stages

prod:
  stage: prod
  script:
    - echo "This is a deploy step."
  when: manual # Continuous Delivery

Go ahead and add it to the .gitlab-ci.yml file to run the pipeline.

You will notice that the license job has failed but didn’t block the other jobs from running.

As discussed, any change to the repo kick starts the pipeline.

We can see the results of this pipeline by visiting https://gitlab-ce-kr6k1mdm.lab.practical-devsecops.training/root/django-nv/pipelines.

Click on the appropriate job name to see the output.

Why the command is not giving the expected output?
The job is failing because the license is not in the allow list.

If the job fails, pip-licenses will not generate the output file. This is why the license job is not storing the artifacts.

Understanding -v option in docker command
In docker command above, you can see -v tag to bind the volume with the following:

Command Output
-v $(pwd):/src
-v: This option is used to mount a volume from the host machine to the Docker container.
$(pwd): A shell command that outputs the current working directory on your local machine.
/src: Specifies the path inside the container where the current working directory will be mounted.
When running a specific Docker command with the -v option, Docker will mount the current directory into a specific directory inside the container. In this case, the local directory will be mounted to /src. In practice, if we open the container, we will be redirected to the / directory.

For some tools, we need to define and specify the file path, for example, /src/filename.json, so that the output file can be reflected in our current local directory.

For further explanation, you can review our “Manage Data in Docker” exercise.
```
