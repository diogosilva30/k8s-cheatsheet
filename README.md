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

## Cleanup DigitalOcean Unattached Volumes
```python
import requests
import time

# DigitalOcean API credentials
API_TOKEN = "TOKEN_HERE"


def get_block_storage_volumes():
    volumes = []
    page = 1
    while True:
        url = f"https://api.digitalocean.com/v2/volumes?page={page}&per_page=100"
        headers = {
            "Authorization": f"Bearer {API_TOKEN}",
            "Content-Type": "application/json",
        }
        response = requests.get(url, headers=headers, timeout=10)
        if response.status_code == 200:
            page_volumes = response.json()["volumes"]
            if not page_volumes:
                break
            volumes.extend(page_volumes)
            page += 1
            time.sleep(1)  # To prevent rate limiting
        else:
            print(f"Failed to retrieve block storage volumes: {response.text}")
            break
    return volumes if volumes else None


# Function to remove unattached block storage volumes
def remove_unattached_volumes() -> None:
    block_storage_volumes: list[dict] | None = get_block_storage_volumes()
    if block_storage_volumes is None:
        return

    # Remove unattached volumes
    for volume in block_storage_volumes:
        if volume["droplet_ids"] is None or len(volume["droplet_ids"]) == 0:
            print(
                f"Removing unattached volume: {volume['id']} with name: {volume['name']} and size: {volume['size_gigabytes']} GB"
            )
            remove_volume(volume["id"])


# Function to remove a volume by ID
def remove_volume(volume_id):
    url = f"https://api.digitalocean.com/v2/volumes/{volume_id}"
    headers = {
        "Authorization": f"Bearer {API_TOKEN}",
        "Content-Type": "application/json",
    }
    response = requests.delete(url, headers=headers, timeout=10)
    if response.status_code == 204:
        print(f"Volume {volume_id} removed successfully.")
    else:
        print(f"Failed to remove volume {volume_id}: {response.text}")


def main():
    remove_unattached_volumes()


if __name__ == "__main__":
    main()
```
