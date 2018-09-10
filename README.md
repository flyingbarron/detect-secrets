# river-detector

Scanning:

```SH
docker run -v $CODEBASE:/code --rm -it txo-whitewater-docker-local.artifactory.swg-devops.com/river-dector
```

The docker image just runs the `run-scan.sh` script in the repo, which this package install. The secrets are stored in the `.secrets.baseline`, in a json format.

After the repo is scanned and new secrets are found, run the audit to mark the secrets as valid or invalid. You can do this by running audit flag on the script.

Audit:
```SH
docker run -v $CODEBASE:/code --rm -it txo-whitewater-docker-local.artifactory.swg-devops.com/river-dector audit
```

#### Inline Whitelisting

Another way of whitelisting secrets is through the inline comment
`# pragma: whitelist secret`.

For example:

```
API_KEY = "blah-blah-but-actually-not-secret" # pragma: whitelist secret

def main():
    print('hello world')

if __name__ == '__main__'
    main()
```

This may be a convenient way for you to whitelist secrets, without having to
regenerate the entire baseline again. Furthermore, this makes the whitelisted
secrets easily searchable, auditable, and maintainable.

### Pre-commit Hook

**TODO: validate**

The code can be run as a pre commit hook using either docker, or running the pip package directly. An easy to manage pre commit hooks is with the pre commit hook framework, which can be found here: https://pre-commit.com/

```
$ cat .pre-commit-config.yaml
-   repo: git@github.ibm.com:river/river-detector
    rev: master
    hooks:
    -   id: detect-secrets
```

## Running via Travis CI

**TODO: add documentation**

## A Few Caveats

This is not meant to be a sure-fire solution to prevent secrets from entering
the codebase. Only proper developer education can truly do that. This pre-commit
hook merely implements several heuristics to try and prevent obvious cases of
committing secrets.

### Things that won't be prevented

* Multi-line secrets
* Default passwords that do not trigger the `KeywordDetector` (e.g. `paaassword = "paaassword"`)

### Plugin Configuration

One method that this package uses to find secrets is by searching for high
entropy strings in the codebase. This is calculated through the [Shannon entropy
formula](http://blog.dkbza.org/2007/05/scanning-data-for-entropy-anomalies.html).
If the entropy of a given string exceeds the preset amount, the string will be
rejected as a potential secret.

This preset amount can be adjusted in several ways:

* Specifying it within the config file, for server scanning.
* Specifying it with command line flags (eg. `--base64-limit`)

Lowering these limits will identify more potential secrets, but also create
more false positives. Adjust these limits to suit your needs.
