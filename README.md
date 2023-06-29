# hyperthought-upload

This is a tool written in hopefully portable Bourne shell that can be used
to upload files to HyperThought.  If you don't know what HyperThought is,
then this tool will likely not be useful to you.

## Overview

This script tries to utilitize only "classic" Unix utilities (`sed`, `awk`)
and uses `curl` to perform the actual HTTPS transport.  It also makes use
of [JSON.awk](https://github.com/step-/JSON.awk) for JSON parsing of
HyperThought responses.

## Usage

Basic usage of this script is as follows:

```
	ht-upload [-D directory] file1 [file2 ...]
```

The optional `-D` argument specifies a directory to create and upload
the specified files into.  If a directory name is omitted then the files
will be uploaded into the root of the workspace.

You can use the following environment variables to control various settings
for `ht-upload`.

##### `HYPERTHOUGHT_URL`

The URL of the HyperThought server.  The OIDC configuration endpoint is
expected to be at `${HYPERTHOUGHT_URL}/openid/.well-known/openid-configuration`
and the API endpoints are all expected to be located under
`${HYPERTHOUGHT_URL}/api

##### `CLIENT_ID`

The numeric client identifier of the account to use to authenticate to
HyperThought.

##### `CLIENT_SECRET`

The client secret for the account identified by `CLIENT_ID`.

##### `WORKSPACE`

The HyperThought workspace the file will be uploaded into.  This can either
be the Workspace ID (in UUID format) or the alias of the desired workspace.

#### `CURL_PROXY`

An optional entry specifying a proxy that `curl` should use.  This can
be in any format supported by the `--proxy` flag.

#### `VERBOSE`

If set to `1` then `ht-upload` will generate more verbose output; this
will include the exact `curl` commands being used.

---

Note that all of these variables can be set in the beginning of the
script and should use standard *assign default value* parameter
expansion syntax to allow command line environment variables to
override defaults in the script itself.
E.g., if you wish to set the default value of `HYPERTHOUGHT_URL` to
`https://foo.bar.com`, you would do:

```
: ${HYPERTHOUGHT_URL:=https://foo.bar.com}"
```

## Additional information

Due to a limitation in `curl` that I was not able to work around, you
will need the following configuration on the Apache server that runs
your HyperThought instance:

```
    RewriteEngine on
    RewriteRule "/api/files/v1/default-file-upload/([^/]*)/.+" "/api/files/v1/default-file-upload/$1/" [PT]
```

The limitation here is that when using the `-T` option to specify that
you are doing a file upload and the specified URL ends with a solidus
(`/`) `curl` will always append the filename to the URL, which ends up
with a URL pointing to a non-existent API endpoint.  Unfortunately there
are not good options; the API endpoint has to end in a solidus, and curl
will unilaterally append the filename in that situation.  You can use a
shell redirect to do an upload using standard input, but in that case
curl will be unable to determine the file size and will try to use chunked
encoding to upload the file and it seems that either WSGI or the Python
library we are using does not support that.  The above `RewriteRule` will
strip off anything from the URL after the file UUID.
