# CrowdStrike Container Image Scan [![Flake8](https://github.com/CrowdStrike/container-image-scan/actions/workflows/linting.yml/badge.svg)](https://github.com/CrowdStrike/container-image-scan/actions/workflows/linting.yml)

This script will scan a container and return response codes indicating pass/fail status.

Specifically, this script:

1. Tags your image using ``docker tag``
2. Authenticates to CrowdStrike using your [OAuth2 API keys](https://falcon.crowdstrike.com/support/api-clients-and-keys)
3. Pushes your image to CrowdStrike for evaluation using ``docker push``, after which CrowdStrike performs an Image Scan
4. Parses returned scan report, generating return error codes as needed

All output is sent to stdout/stderr.

## Prerequisites

This sample/demo script requires the [Docker Engine API python library](https://pypi.org/project/docker/) and the [``requests`` HTTP library](https://pypi.org/project/requests/). These can be installed via ``pip``:

```shell
$ pip3 install docker requests docker-image-py
```

## Usage

```shell
$ python3 cs_scanimage.py --help
usage: cs_scanimage.py [-h] -u CLIENT_ID -r REPO
                       [-c {us-1,us-2,eu-1}] [--json-report REPORT]
                       [--log-level {DEBUG,INFO,WARNING,ERROR,CRITICAL}]

Crowdstrike - scan your container image.

optional arguments:
  -h, --help            show this help message and exit
  --json-report REPORT  Export JSON report to specified file
  --log-level {DEBUG,INFO,WARNING,ERROR,CRITICAL}
                        Set the logging level
  -s SCORE --score_threshold
                        Vulnerability score threshold default 500

required arguments:
  -u CLIENT_ID, --clientid CLIENT_ID
                        Falcon OAuth2 API ClientID
  -r REPO, --repo REPO  Container image repository
  -c {us-1,us-2,eu-1}, --cloud-region {us-1,us-2,eu-1}
                        CrowdStrike cloud region
```

Note that CrowdStrike Falcon OAuth2 credentials may be supplied also by the means of environment variables: FALCON_CLIENT_ID, FALCON_CLIENT_SECRET, and FALCON_CLOUD_REGION. Establishing and retrieving OAuth2 API credentials can be performed at https://falcon.crowdstrike.com/support/api-clients-and-keys.

FALCON_CLIENT_ID and FALCON_CLIENT_SECRET can be set via environment variables for automation.

## Example Scans

### Example 1

```shell
$ python cs_scanimage.py --clientid FALCON_CLIENT_ID --repo <repo> --cloud-region <cloud_region>

please enter password to login
Password:
```

The command above will return output similar to:

```shell
INFO    Downloading Image Scan Report
INFO    Searching for vulnerabilities in scan report...
INFO    Searching for leaked secrets in scan report...
INFO    Searching for malware in scan report...
INFO    Searching for misconfigurations in scan report...
WARNING Alert: Misconfiguration found
INFO    Vulnerability score threshold not met: '0' out of '500'
```

### Example 2

The script provided was built to score vulnerabilities on a scale show below.

```txt
critical_score = 2000
high_score = 500
medium_score = 100
low_score = 20
```

The default value to return a non-zero error code for vulnerabilties is one high vulnerabilty. This can be overridden by providing the `-s` parameters to the script.

The example below will accomodate vulnerabilities with a sum of 1500.

```shell
$ python cs_scanimage.py --clientid FALCON_CLIENT_ID --repo <repo> \
    --cloud-region <cloud_region> -s 1500

```

The ```echo $?``` command can be utilized to review the return code, e.g:

```shell 
echo $?
1
```

The ```echo $?``` above displays the returned code with the following mappings:

```shell
VulnerabilityScoreExceeded = 1
Malware = 2
Secrets = 3
Success = 0
Misconfig = 0
ScriptFailure = 10
```

### Example 3

Using ENV variables with default `us-1` Cloud scanner to inspect the open-source `_\hello-world` docker container.

```shell
$ FALCON_CLIENT_ID=AAAAAAAAAAAAA FALCON_CLIENT_SECRET=BBBBBBBBBBBB python cs_scanimage.py -r hello-world:latest
INFO    Tagging 'hello-world:latest' to 'container-upload.us-1.crowdstrike.com/hello-world:latest'
INFO    Performing docker login to CrowdStrike Image Assessment Service
INFO    Performing docker push to container-upload.us-1.crowdstrike.com/hello-world:latest
INFO    Docker: The push refers to repository [container-upload.us-1.crowdstrike.com/hello-world]
INFO    Docker: Preparing
INFO    Docker: Pushed=====================================>]  14.85kB/13.26kB
INFO    Docker: latest: digest: sha256:f54a58bc1aac5ea1a25d796ae155dc228b3f0e11d046ae276b39c4bf2f13d8c4 size: 525
INFO    Authenticating with CrowdStrike Falcon API
INFO    Downloading Image Scan Report
INFO    Searching for vulnerabilities in scan report...
INFO    Searching for leaked secrets in scan report...
INFO    Searching for malware in scan report...
INFO    Searching for misconfigurations in scan report...
INFO    Vulnerability score threshold not met: '0' out of '500'
```

## Running the Scan using CI/CD

* You can use the [container-image-scan](https://github.com/marketplace/actions/crowdstrike-container-image-scan) GitHub Action in your GitHub workflows. Checkout the action at [https://github.com/marketplace/actions/crowdstrike-container-image-scan](https://github.com/marketplace/actions/crowdstrike-container-image-scan)

* Pipeline examples, including the GitHub Action, can be found at the CrowdStrike [image-scan-example](https://github.com/CrowdStrike/image-scan-example) repositotry.
