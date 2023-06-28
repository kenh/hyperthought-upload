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
	ht-upload file1 [file2 ...]
```

You can use the following environment variables to control various settings
for `ht-upload`.

### `HYPERTHOUGHT_URL`

The URL of the HyperThought server.  The OIDC configuration endpoint is
 expected to be at `${HYPERTHOUGHT_URL}/openid/.well-known/openid-configuration`
  and the API endpoints are all expected to be located under
  `${HYPERTHOUGHT_URL}/api'

### `CLIENT_ID`

The numeric client identifier of the account to use to authenticate to
  HyperThought.

Note that all of these variables can be set in the beginning of the
script and should use standard *assign default value* parameter
expansion syntax to allow command line environment variable override.
E.g., you should
