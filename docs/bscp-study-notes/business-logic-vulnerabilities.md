# Business Logic Vulnerabilities

## Testing

- Try changing product IDs on intercepted requests etc
- Try submitting unconventional values, negative transfer value for example, or exceptionally high values, etc
    - Are there any limits that are imposed on the data?
    - What happens when you reach those limits?
    - Is any transformation or normalization being performed on your input?
- Use intruder/repeater to repeatedly add the maximum allowed of an input
- Try creating accounts with extremely long values and see if they get truncated
- Does changing email to an admin priviledge grant priviledges without email verification
- Try removing one parameter at a time from requests and observe the results
- Try navigating the site in an unexpected fashion with forced browsing (example: send to cart > order confirmed)
- Try dropping random requests (skip role selecter requests for example)
- Look for anywhere that user input is encoded/decoded, can it be used to decode anything else like session cookies or stay logged in tokens
- [Email parser discrepencies](https://portswigger.net/research/splitting-the-email-atom), try various encodings:
    - `=?iso-8859-1?q?=61=62=63?=foo@ginandjuice.shop`
    - `=?utf-8?q?=61=62=63?=foo@ginandjuice.shop`
    - `=?utf-7?q?&AGEAYgBj-?=foo@ginandjuice.shop`