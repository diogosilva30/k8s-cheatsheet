# k8s-cheatsheet
Kubernetes cheatsheet


### Find Service CIDR of cluster

```shell
SVCRANGE=$(echo '{"apiVersion":"v1","kind":"Service","metadata":{"name":"tst"},"spec":{"clusterIP":"1.1.1.1","ports":[{"port":443}]}}' | kubectl apply -f - 2>&1 | sed 's/.*valid IPs is //')
echo $SVCRANGE
```

### Export GitLab CI/CD project variables to local env
```python
import os

import requests

try:
    ACCESS_TOKEN = os.environ["GITLAB_TOKEN"]
except KeyError as err:
    raise ValueError(
        "GITLAB_TOKEN environment variable is not set. "
        " Please define it with your GitLab access token."
    ) from err

# GitLab API endpoint
GITLAB_HOST = "https://gitlab.company.com"
PROJECT = "1234"
API_URL = f"{GITLAB_HOST}/api/v4/projects/{PROJECT}/variables?per_page=1000"


# Curl command to retrieve CI/CD variables
headers = {"PRIVATE-TOKEN": ACCESS_TOKEN}
response = requests.get(API_URL, headers=headers, timeout=10)
response.raise_for_status()
data = response.json()

# Loop through each key-value pair
for item in data:
    key = item["key"]
    value = item["value"]
    os.environ[key] = value


print(f"Loaded {len(data)} environment variables from GitLab Project {PROJECT}")

```


### Find size of S3 bucket

```shell
#!/bin/bash

# Don't forget to make this executable "chmod +x s3_size.sh"

# Replace these variables with your own values or set them as environment variables
bucket_name="your-bucket-name"
custom_endpoint="https://your-custom-endpoint-url"

# Check if environment variables for AWS access key and secret key are set
if [ -z "${AWS_ACCESS_KEY_ID}" ] || [ -z "${AWS_SECRET_ACCESS_KEY}" ]; then
    echo "Error: AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY environment variables must be set."
    exit 1
fi

aws s3 ls --summarize --human-readable --recursive s3://${bucket_name} --endpoint-url "${custom_endpoint} | tail -2

```
### Clear contents of S3 bucket

```shell
#!/bin/bash

# Don't forget to make this executable "chmod +x s3_clear.sh"

# Replace these variables with your own values or set them as environment variables
bucket_name="your-bucket-name"
custom_endpoint="https://your-custom-endpoint-url"

# Check if environment variables for AWS access key and secret key are set
if [ -z "${AWS_ACCESS_KEY_ID}" ] || [ -z "${AWS_SECRET_ACCESS_KEY}" ]; then
    echo "Error: AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY environment variables must be set."
    exit 1
fi

# List all objects in the bucket and delete them one by one
aws s3 ls "s3://${bucket_name}" --recursive --endpoint-url "${custom_endpoint}" | \
while IFS= read -r line; do
    file=$(echo "${line}" | awk '{print $4}')
    if [ -n "${file}" ]; then
        aws s3 rm "s3://${bucket_name}/${file}" --endpoint-url "${custom_endpoint}"
    fi
done

echo "Bucket ${bucket_name} contents cleared."
```

### Debug pod without distro

```shell
kubectl debug -it <pod_name> --image=busybox:1.28 -n <namespace> --target <container_name>

# Then you can find the pod's mounted filesystem in /proc/1/root
# In case of permission denied: https://stackoverflow.com/a/76772820/11122248
```
