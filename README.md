# cf-python-buildpack-vendoring-issue

Pushing a cf app with Python buildpack 1.8.11 and later fail with the following error:

```
➜ cf push .                   
Pushing app . to org XXXX/ space XXXXX as XXXXXXX...
Applying manifest file XXXXXX/manifest.yml...

Updating with these attributes...
  ---
  applications:
  - name: .
    disk-quota: 2G
    health-check-type: process
    path: XXXXX/cf-python-buildpack-vendoring-issue
    memory: 1G
+   no-route: true
    stack: cflinuxfs4
+   buildpack: https://github.com/cloudfoundry/python-buildpack.git#v1.8.11
    command: sleep infinity
Manifest applied
Packaging files to upload...
Uploading files...
 8.52 MiB / 8.52 MiB [====================================================================================================================================================================================================================================================================================================================================================================================================] 100.00% 3s

Waiting for API to complete processing files...

Staging app and tracing logs...
   Cell 51dfa8f8-3575-4f95-ab1e-f05fda13bcee creating container for instance 981b8b11-1e34-4c60-83ed-9bd47ef6c69c
   Security group rules were updated
   Cell 51dfa8f8-3575-4f95-ab1e-f05fda13bcee successfully created container for instance 981b8b11-1e34-4c60-83ed-9bd47ef6c69c
   Downloading app package...
   Downloading build artifacts cache...
   Downloaded app package (25.6M)
   Downloaded build artifacts cache (98.6M)
   -----> Download go 1.19
   -----> Running go build supply
   /tmp/buildpackdownloads/43c9f301558104f9213efd44956d51a2 ~
   ~
   -----> Python Buildpack version 1.8.11
   **WARNING** buildpack version changed from 1.8.13 to 1.8.11
   -----> Supplying Python
   -----> Installing python 3.11.4
   Copy [/tmp/cache/final/dependencies/462b9f8a5b122efaaa271479eecd2bad9de9335c4857d86c87b1a76c4f37e501/python_3.11.4_linux_x64_cflinuxfs4_32c71612.tgz]
   Using python's pip module
   pip 23.1.2 from /tmp/contents3563999740/deps/0/python/lib/python3.11/site-packages/pip (python 3.11)
   -----> Running Pip Install (Vendored)
   Using the pip --no-build-isolation flag since it is available
   python -m pip install -r /tmp/app/requirements.txt --ignore-installed --exists-action=w --src=/tmp/contents3563999740/deps/0/src --no-index --find-links=file:///tmp/app/vendor --disable-pip-version-check --no-warn-script-location --no-build-isolation
   Looking in links: file:///tmp/app/vendor
   Processing ./vendor/oss2-2.18.1.tar.gz (from -r /tmp/app/requirements.txt (line 1))
   Preparing metadata (pyproject.toml): started
   Preparing metadata (pyproject.toml): finished with status 'error'
   error: subprocess-exited-with-error
   
   × Preparing metadata (pyproject.toml) did not run successfully.
   │ exit code: 1
   ╰─> [13 lines of output]
   running dist_info
   creating /tmp/pip-modern-metadata-r605vwxw/oss2.egg-info
   writing /tmp/pip-modern-metadata-r605vwxw/oss2.egg-info/PKG-INFO
   writing dependency_links to /tmp/pip-modern-metadata-r605vwxw/oss2.egg-info/dependency_links.txt
   writing requirements to /tmp/pip-modern-metadata-r605vwxw/oss2.egg-info/requires.txt
   writing top-level names to /tmp/pip-modern-metadata-r605vwxw/oss2.egg-info/top_level.txt
   writing manifest file '/tmp/pip-modern-metadata-r605vwxw/oss2.egg-info/SOURCES.txt'
   reading manifest file '/tmp/pip-modern-metadata-r605vwxw/oss2.egg-info/SOURCES.txt'
   reading manifest template 'MANIFEST.in'
   adding license file 'LICENSE'
   writing manifest file '/tmp/pip-modern-metadata-r605vwxw/oss2.egg-info/SOURCES.txt'
   creating '/tmp/pip-modern-metadata-r605vwxw/oss2-2.18.1.dist-info'
   error: invalid command 'bdist_wheel'
   [end of output]
   
   note: This error originates from a subprocess, and is likely not a problem with pip.
   error: metadata-generation-failed
   
   × Encountered error while generating package metadata.
   ╰─> See above for output.
   
   note: This is an issue with the package mentioned above, not pip.
   hint: See above for details.
   Running pip install failed. You need to include all dependencies in the vendor directory.
   **ERROR** Could not install vendored pip packages: could not run pip: exit status 1
```


This problem likely happens due to `--build-isolation` - see [issue #171](https://github.com/cloudfoundry/python-buildpack/issues/171).

## How to reproduce

First, download `vendor`:

```
docker run --platform linux/amd64 -v $(pwd):/mnt -ti docker.io/library/python:3.7-buster /bin/sh -c 'pip install pip==21.1.3 && pip download -d /mnt/vendor -r /mnt/requirements.txt'
```

Second, push the app:

```
cf push .
```

Note that buildpack 1.8.13 does not show the full error message. For this, I need 1.8.11.

## Non-vendored setup works


By moving the `vendor` directory, the app works normally when downloading the dependency directly from
PyPI.

