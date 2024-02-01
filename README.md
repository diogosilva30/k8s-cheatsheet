# k8s-cheatsheet
Kubernetes cheatsheet


### Find Service CIDR of cluster

```shell
SVCRANGE=$(echo '{"apiVersion":"v1","kind":"Service","metadata":{"name":"tst"},"spec":{"clusterIP":"1.1.1.1","ports":[{"port":443}]}}' | kubectl apply -f - 2>&1 | sed 's/.*valid IPs is //')
echo $SVCRANGE
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
