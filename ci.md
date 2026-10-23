# CI

The `ci` role holds what automated builds and tests found about a
revision: Jenkins builds, GitHub check runs, GitLab pipelines, test
result databases.

Terms:

    RUN         one build or test session on one COMMIT
    SUITE       tests a RUN ran together, e.g. a test script, a pipeline
                stage, a check
    TEST        one test in a SUITE
    RESULT      pass, fail, skip, timeout or error; error is a failure of
                the CI itself, not of the code
    LOG         a file a RUN, SUITE or TEST produced

## Commands

    runs COMMIT     print the run RECORDs that tested COMMIT
    run RUN         print the run RECORD
    suites RUN      print the suite RECORDs of RUN
    tests SUITE     print the test RECORDs of SUITE
    logs ID         print the log RECORDs of a RUN, SUITE or TEST
    retest RUN      ask for RUN again, with a reason read as JSON from stdin

`retest` runs only on the human's explicit action, as `publish` does.

## Capabilities

`capabilities` prints an object with these booleans:

    tests       the SOURCE has results per TEST, not only per SUITE
    logs        `logs` works
    retest      `retest` works
    bugs        the SOURCE links failures to issues

## Run

`kind` is `run`. `data`:

    commit      the COMMIT
    status      queued, running or done
    result      a RESULT once done, else null
    started     the time, RFC 3339, or null
    finished    the time, RFC 3339, or null
    url         where a browser shows it, or null
    config      what it ran on, as the SOURCE names it, e.g.
                {"arch": "x86_64", "distro": "el9"}

`links` include `commit` and, when the SOURCE knows them, `change` and
`revision` of the discussion it tested.

## Suite

`kind` is `suite`. `data`:

    run         id of the RUN
    name        e.g. sanity
    result      a RESULT
    counts      tests per RESULT, e.g. {"pass": 412, "fail": 2, "skip": 9}
    duration    seconds, or null
    url         or null

## Test

`kind` is `test`. `data`:

    suite       id of the SUITE
    name        e.g. test_51b
    result      a RESULT
    error       the first line of the failure, or null
    duration    seconds, or null
    url         or null

`links` include an `issue` for each bug the SOURCE has linked to the
failure.

```json
{"id": "9b3e", "source": "https://ci.example.org", "role": "ci",
 "kind": "test", "basis": "9b3e:done",
 "links": [["commit", "4a2f..."], ["issue", "BUG-123"]],
 "data": {"suite": "77c1", "name": "test_51b", "result": "fail",
          "error": "dd failed: No space left on device",
          "duration": 41, "url": "https://ci.example.org/tests/9b3e"},
 "notes": {}}
```

## Log

`kind` is `log`. `data`:

    of          id of the RUN, SUITE or TEST
    name        e.g. console.log
    url         where to download it
    size        bytes, or null

`logs` prints where logs are, not their content: logs are large, and a
STEP fetches only what it reads.

## Retest input

`retest` reads one object:

    reason      why, as the SOURCE wants it, e.g. a text or an issue id

## Not in this role

Whether a failure comes from the change under test or is a known flake is
a judgement. Issues the SOURCE itself links to a failure are data, in
`links`; anything else a STEP decides and keeps in `notes`.
