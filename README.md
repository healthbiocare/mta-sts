# MTA-STS for HBC

## This repository hosts the MTA-STS policy file for healthbiocare.at, served via GitHub Pages

### File layout

- `.nojekyll` - disables Jekyll processing
- `CNAME` - custom domain (`mta-sts.healthbiocare.at`)
- `.well-known/mta-sts.txt` - the policy file

### Updating the policy

1. Edit `.well-known/mta-sts.txt`.

   - Ensure the policy version is set to `STSv1`.
   - The `max_age` value should be set to a minimum of 604800 seconds (7 days).
   - The `mode` can be set to `enforce`, `testing`, or `none` depending on your testing phase.
   - Example content:

     ```txt
     version: STSv1
     mode: enforce
     max_age: 604800
     id: 20231001T120000Z
     ```

2. Commit and push changes.
3. Bump the `_mta-sts` TXT record's `id=` timestamp in DNS.

### Monitoring MTA-STS Reports

The MTA-STS policy includes reporting functionality via `reports@healthbiocare.at`, which receives aggregate reports from email servers about policy performance. Currently, we receive reports primarily from Microsoft and Google.

#### Semi-regular Monitoring Process

Follow these steps to analyze the MTA-STS reports:

##### 1. Download and Extract Reports

- Download all relevant monitoring emails locally
- Unzip any compressed email files if necessary

##### 2. Extract JSON Reports from Email Attachments

Use the following Python script to extract report attachments from `.eml` files:

```bash
# Download the extraction script
wget https://gist.githubusercontent.com/urschrei/5258588/raw/e79789621b340234e1a9d906cfefa8237f52b649/parseml.py
# Review the contents of parseml.py before running it to ensure it is safe.

# Run the script to extract all attachments to "output" folder
python3 parseml.py *.eml
```

This will output all attachments as `.json.gz` files to the "output" folder.

##### 3. Decompress Report Files

Navigate to the output folder and decompress all JSON files:

```bash
cd output
gunzip *.gz
```

##### 4. Analyze Report Summary

Create and use the following script to get a summary of successful and failed sessions:

```bash
#!/bin/bash
# save as: analyze_mta_sts_reports.sh

# Initialize counters
success_total=0
failure_total=0

# Loop through all JSON files in the current directory
for file in *.json; do
  # Extract successful session count and add to total
  success_count=$(jq '.policies[].summary."total-successful-session-count"' "$file")
  failure_count=$(jq '.policies[].summary."total-failure-session-count"' "$file")

  success_total=$((success_total + success_count))
  failure_total=$((failure_total + failure_count))
done

# Print the totals
echo "Total successful sessions: $success_total"
echo "Total failure sessions: $failure_total"
```

Make the script executable and run it:

```bash
chmod +x analyze_mta_sts_reports.sh
./analyze_mta_sts_reports.sh
```

#### Prerequisites

- `jq` - JSON processor (install with `sudo apt install jq` on Ubuntu/Debian)
- `python3` - Python 3 is recommended, as Python 2 is deprecated (required for running the attachment extraction script)
- `wget` or `curl` - For downloading the extraction script

#### Understanding the Reports

- **Successful sessions**: Email deliveries that complied with the MTA-STS policy
- **Failed sessions**: Email deliveries that failed due to policy violations or connectivity issues
- Monitor the failure rate to identify potential security issues or misconfigurations
