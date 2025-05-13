## This repository hosts the MTA-STS policy file for healthbiocare.at, served via GitHub Pages.


## File layout
- `.nojekyll` - disables Jekyll processing
- `CNAME` - custom domain (`mta-sts.healthbiocare.at`)
- `.well-known/mta-sts.txt` - the policy file

## Updating the policy
1. Edit `.well-known/mta-sts.txt`.
2. Commit and push changes.
3. Bump the `_mta-sts` TXT record’s `id=` timestamp in DNS.
