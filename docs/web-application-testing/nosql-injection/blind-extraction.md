# NoSQL Injection - Blind Data Extraction

**Title:** Blind NoSQL Injection via Regex for Character Extraction

**Severity:** High

**Impact:** High

**Exploitability:** Medium

**CVSS Base Score:** 7.5

**CVSS v3 Vector:** CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:N/A:N

**Vulnerability Type:** NoSQL Injection

**Target:** Search or filter parameters

## Impact

Full database exfiltration even without direct data return, by observing response differences.

## Description

Use $regex operator to perform character-by-character extraction based on application behavior.

## Steps to Reproduce

Use $regex for character-by-character extraction by observing response differences:

```json
{
    "trackingNum": {
        "$regex": "^3.*"
    }
}
```

Continue refining the regex to extract data one character at a time.

## Recommendation

Use parameterized/safe queries. Do not allow user-controlled operators. Normalize responses.

## References

NoSQL injection resources.