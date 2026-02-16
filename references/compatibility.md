# Package Compatibility Map

Known compatibility requirements discovered from production CVE fix issues. **Check this before upgrading any package listed here.** Add new entries as they are discovered.

## urllib3

**Breaking version**: 2.0.0+

urllib3 2.x removed `DEFAULT_CIPHERS` which older boto3/botocore versions import.

| Companion Package | Minimum Version | Reason |
|-------------------|-----------------|--------|
| boto3 | >= 1.34.46 | urllib3 2.x removed DEFAULT_CIPHERS |
| botocore | >= 1.34.46 | urllib3 2.x removed DEFAULT_CIPHERS |

## aiohttp

**Breaking version**: 3.9.0+

aiohttp 3.9+ changed the `AIOHTTPTransport` API used by gql.

| Companion Package | Minimum Version | Reason |
|-------------------|-----------------|--------|
| gql | >= 3.5.0 | aiohttp 3.9+ changed AIOHTTPTransport API |

**Note**: aiohttp 3.13+ has additional breaking changes. Test carefully.

## gql

**Breaking version**: 3.5.0+

gql 3.5+ added new dependencies for retry logic and requests transport.

| Companion Package | Minimum Version | Reason |
|-------------------|-----------------|--------|
| backoff | >= 2.2.1 | gql 3.5+ requires backoff for retry logic |
| requests-toolbelt | >= 1.0.0 | gql[requests] requires requests-toolbelt |

**Note**: If using `gql[requests]` extra, `requests-toolbelt` is required.

## How to Use This Map

When the CVE fix requires upgrading a package listed above:

1. Check if the new version crosses the "breaking version" threshold.
2. If yes, also upgrade all companion packages to at least the minimum version listed.
3. Verify companion packages exist in the current `Pipfile.lock` before adding them.
4. Include companion package changes in both Pipfile and Pipfile.lock.
5. Document the companion upgrades in the MR description.

## Adding New Entries

After discovering a new compatibility issue in production:

1. Add the entry to this file following the format above.
2. Record it via `learn_tool_fix()` for the memory system.
3. Include the breaking version threshold and required companion versions.
