# Depicted—dpxdt

[![Build Status](https://travis-ci.org/koddsson/dpxdt-server.svg?branch=master-travis)](https://travis-ci.org/koddsson/dpxdt-server)
[![Coverage Status](https://coveralls.io/repos/koddsson/dpxdt-server/badge.svg?branch=master)](https://coveralls.io/r/koddsson/dpxdt-server?branch=master)

Make continuous deployment safe by comparing before and after webpage 
screenshots for each release. Depicted shows when any visual, perceptual 
differences are found. This is the ultimate, automated end-to-end test.

### API Reference

All of these requests are POSTs with URL-encoded or multipart/form-data bodies 
and require HTTP Basic Authentication using your API key as the username and 
secret as the password. All responses are JSON. The 'success' key will be 
present in all responses and true if the request was successful. If 'success'
isn't present, a human-readable error message may be present in the response 
under the key 'error'.

Endpoints:

- [/api/create_release](#apicreate_release)
- [/api/find_run](#apifind_run)
- [/api/request_run](#apirequest_run)
- [/api/upload](#apiupload)
- [/api/report_run](#apireport_run)
- [/api/runs_done](#apiruns_done)

#### /api/create_release

Creates a new release candidate for a build.

##### Parameters
<dl>
    <dt>build_id</dt>
    <dd>ID of the build.</dd>
    <dt>release_name</dt>
    <dd>Name of the new release.</dd>
    <dt>url</dt>
    <dd>URL of the homepage of the new release. Only present for humans who need to understand what a release is for.</dd>
</dl>

##### Returns
<dl>
    <dt>build_id</dt>
    <dd>ID of the build.</dd>
    <dt>release_name</dt>
    <dd>Name of the release that was just created.</dd>
    <dt>release_number</dt>
    <dd>Number assigned to the new release by the system.</dd>
    <dt>url</dt>
    <dd>URL of the release's homepage.</dd>
</dl>

#### /api/find_run

Finds the last good run of the given name for a release. Returns an error if no run previous good release exists.

##### Parameters
<dl>
    <dt>build_id</dt>
    <dd>ID of the build.</dd>
    <dt>run_name</dt>
    <dd>Name of the run to find the last known-good version of.</dd>
</dl>

##### Returns
<dl>
    <dt>build_id</dt>
    <dd>ID of the build.</dd>
    <dt>release_name</dt>
    <dd>Name of the last known-good release for the run.</dd>
    <dt>release_number</dt>
    <dd>Number of the last known-good release for the run.</dd>
    <dt>run_name</dt>
    <dd>Name of the run that was found. May be null if a run could not be found.</dd>
    <dt>url</dt>
    <dd>URL of the last known-good release for the run. May be null if a run could not be found.</dd>
    <dt>image</dt>
    <dd>Artifact ID (SHA1 hash) of the screenshot image associated with the run. May be null if a run could not be found.</dd>
    <dt>log</dt>
    <dd>Artifact ID (SHA1 hash) of the log file from the screenshot process associated with the run. May be null if a run could not be found.</dd>
    <dt>config</dt>
    <dd>Artifact ID (SHA1 hash) of the config file used for the screenshot process associated with the run. May be null if a run could not be found.</dd>
</dl>

#### /api/request_run

Requests a new run for a release candidate. Causes the API system to take screenshots and do pdiffs. When `ref_url` and `ref_config` are supplied, the system will run two sets of captures (one for the baseline, one for the new release) and then compare them. When `rel_url` and `ref_config` are not specified, the last good run for this build is found and used for comparison.

##### Parameters
<dl>
    <dt>build_id</dt>
    <dd>ID of the build.</dd>
    <dt>release_name</dt>
    <dd>Name of the release.</dd>
    <dt>release_number</dt>
    <dd>Number of the release.</dd>
    <dt>url</dt>
    <dd>URL to request as a run.</dd>
    <dt>config</dt>
    <dd>JSON data that is the config for the new run.</dd>
    <dt>ref_url</dt>
    <dd>URL of the baseline to request as a run.</dd>
    <dt>ref_config</dt>
    <dd>JSON data that is the config for the baseline of the new run.</dd>
</dl>

###### Format of `config`

The config passed to the `request_run` function may have any or all of these fields. All fields are optional and have reasonably sane defaults.

```json
{
    "viewportSize": {
        "width": 1024,
        "height": 768
    },
    "injectCss": ".my-css-rules-here { display: none; }",
    "injectJs": "document.getElementById('foobar').innerText = 'foo';",
    "resourceTimeoutMs": 60000
}
```

##### Returns
<dl>
    <dt>build_id</dt>
    <dd>ID of the build.</dd>
    <dt>release_name</dt>
    <dd>Name of the release.</dd>
    <dt>release_number</dt>
    <dd>Number of the release.</dd>
    <dt>run_name</dt>
    <dd>Name of the run that was created.</dd>
    <dt>url</dt>
    <dd>URL that was requested for the run.</dd>
    <dt>config</dt>
    <dd>Artifact ID (SHA1 hash) of the config file that will be used for the screenshot process associated with the run.</dd>
    <dt>ref_url</dt>
    <dd>URL that was requested for the baseline reference for the run.</dd>
    <dt>ref_config</dt>
    <dd>Artifact ID (SHA1 hash) of the config file used for the baseline screenshot process of the run.</dd>
</dl>

#### /api/upload

Uploads an artifact referenced by a run.

##### Parameters
<dl>
    <dt>build_id</dt>
    <dd>ID of the build.</dd>
    <dt>(a single file in the multipart/form-data)</dt>
    <dd>Data of the file being uploaded. Should have a filename in the mime headers so the system can infer the content type of the uploaded asset.</dd>
</dl>

##### Returns
<dl>
    <dt>build_id</dt>
    <dd>ID of the build.</dd>
    <dt>sha1sum</dt>
    <dd>Artifact ID (SHA1 hash) of the file that was uploaded.</dd>
    <dt>content_type</dt>
    <dd>Content type of the artifact that was uploaded.</dd>
</dl>

#### /api/report_run

Reports data for a run for a release candidate. May be called multiple times as progress is made for a run. Should not be called once the screenshot image for the run has been assigned.

##### Parameters
<dl>
    <dt>build_id</dt>
    <dd>ID of the build.</dd>
    <dt>release_name</dt>
    <dd>Name of the release.</dd>
    <dt>release_number</dt>
    <dd>Number of the release.</dd>
    <dt>run_name</dt>
    <dd>Name of the run.</dd>
    <dt>url</dt>
    <dd>URL associated with the run.</dd>
    <dt>image</dt>
    <dd>Artifact ID (SHA1 hash) of the screenshot image associated with the run.</dd>
    <dt>log</dt>
    <dd>Artifact ID (SHA1 hash) of the log file from the screenshot process associated with the run.</dd>
    <dt>config</dt>
    <dd>Artifact ID (SHA1 hash) of the config file used for the screenshot process associated with the run.</dd>
    <dt>ref_url</dt>
    <dd>URL associated with the run's baseline release.</dd>
    <dt>ref_image</dt>
    <dd>Artifact ID (SHA1 hash) of the screenshot image associated with the run's baseline release.</dd>
    <dt>ref_log</dt>
    <dd>Artifact ID (SHA1 hash) of the log file from the screenshot process associated with the run's baseline release.</dd>
    <dt>ref_config</dt>
    <dd>Artifact ID (SHA1 hash) of the config file used for the screenshot process associated with the run's baseline release.</dd>
    <dt>diff_image</dt>
    <dd>Artifact ID (SHA1 hash) of the perceptual diff image associated with the run.</dd>
    <dt>diff_log</dt>
    <dd>Artifact ID (SHA1 hash) of the log file from the perceptual diff process associated with the run.</dd>
    <dt>diff_failed</dt>
    <dd>Present and non-empty string when the diff process failed for some reason. May be missing when diff ran and reported a log but may need to retry for this run.</dd>
    <dt>run_failed</dt>
    <dd>Present and non-empty string when the run failed for some reason. May be missing when capture ran and reported a log but may need to retry for this run.</dd>
    <dt>distortion</dt>
    <dd>Float amount of difference found in the diff that was uploaded, as a float between 0 and 1</dd>
</dl>

##### Returns
Nothing but success on success.

#### /api/runs_done

Marks a release candidate as having all runs reported.

##### Parameters
<dl>
    <dt>build_id</dt>
    <dd>ID of the build.</dd>
    <dt>release_name</dt>
    <dd>Name of the release.</dd>
    <dt>release_number</dt>
    <dd>Number of the release.</dd>
</dl>

##### Returns
<dl>
    <dt>results_url</dt>
    <dd>URL where a release candidates run status can be viewed in a web browser by a build admin.</dd>
</dl>
