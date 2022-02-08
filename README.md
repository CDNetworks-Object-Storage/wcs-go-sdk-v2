# Version
`Current version: v1.0.1`

# Requirements
`Go 1.7 or above`

# Install
`go get -u github.com/CDNetworks-Object-Storage/wcs-go-sdk-v2/wos`

## Initialization

WosClient is a client which provides series of interfaces to manage buckets and objects on Object Storage.

You should create a `wosClient` instance and adjust parameters before sending a request to Object Storage.

### Create a wosClient
```
var ak = "*** Provide your Access Key ***"
var sk = "*** Provide your Secret Key ***"
var endpoint = "https://your-endpoint"
 
wosClient, err := wos.New(ak, sk, endpoint, wos.WithRegion("your-region"))
if err == nil {
    // Access OS with wosClient
    // ...
 
    // close wosClient
    wosClient.Close()
}
```

**Configurations**

You can do configurations by specifing paramters when create a wosclient.

For example, set timeout time of getting a response header at 30 seconds.
```
wosClient, err := wos.New(ak, sk, endpoint, wos.WithHeaderTimeout(30))
```

**Parameters**
| method | description | recommended |
| -- | -- | -- |
| WithSignature(signature SignatureType)	| Signature type, which is set at `wos.SignatureWos`(provided by CDNetworks) by default. You can also set the parameter at aws-v2(wos.SignatureV2) or aws-v4(wos.SignatureV4)| N/A
| WithSslVerifyAndPemCerts(sslVerify bool, pemCerts []byte)	| Set the parameters for verifying the server certificate.| N/A
| WithHeaderTimeout(headerTimeout int)	| Timeout time of getting a response header, 60s by default.	| 10, 60
| WithMaxConnections(maxIdleConns int)	| Maximum number of idle HTTP connections, 1000 by default.| N/A
| WithConnectTimeout(connectTimeout int)	| Timeout time of establishing HTTP/HTTPS connection, 60 by default(unit: second).| 10, 60
| WithSocketTimeout(socketTimeout int)	| Timeout time of reading or writing, 60 by default(unit: second).| 10, 60
| WithIdleConnTimeout(idleConnTimeout int)	|Timeout time of idle HTTP connections, 60 by default(unit: second). | 60
| WithMaxRetryCount(maxRetryCount int)	|Maximum retry times when connect failed, 3 by default.	| 1, 5
| WithProxyUrl(proxyUrl string)	| Set proxy url.| N/A
| WithHttpTransport(transport *http.Transport)	| Customized transport.| N/A
| WithRequestContext(ctx context.Context)	| Set HTTP context.| N/A
| WithMaxRedirectCount(maxRedirectCount int)	| Maximum redirect times, 3 by default.| 1, 5
| WithRegion(region string) | Set the region of S3| N/A
| WithPathStyle(pathStyle boolean)| Whether to use path sytle. When path style is disabled, URL with `bucketName.endpoint` format is used to access Object Storage, otherwise, `bucketName.endpoint` format is used. Path style is disabled by default.| N/A

# How to use
## List Bucket
```
var ak = "*** Provide your Access Key ***"
var sk = "*** Provide your Secret Key ***"
var endpoint = "https://your-endpoint"
 
wosClient, _ := wos.New(ak, sk, endpoint)
 
input := &wos.ListBucketsInput{}
input.QueryLocation = true
output, err := wosClient.ListBuckets(input)
if err == nil {
    fmt.Printf("StatusCode:%d, RequestId:%s\n", output.StatusCode, output.RequestId)
    fmt.Printf("Owner.DisplayName:%s, Owner.ID:%s\n", output.Owner.DisplayName, output.Owner.ID)
    for index, val := range output.Buckets {
        fmt.Printf("Bucket[%d]-Name:%s,CreationDate:%s,EndPoint:%s,Region:%s\n", index, val.Name, val.CreationDate, val.Endpoint, val.Region)
    }
}
```

## List Objects
```
var ak = "*** Provide your Access Key ***"
var sk = "*** Provide your Secret Key ***"
var endpoint = "https://your-endpoint"
 
wosClient, _ := wos.New(ak, sk, endpoint)
 
input := &wos.ListObjectsInput{}
input.Bucket = bucketName
//  input.Prefix = "src/"
output, err := wosClient.ListObjects(input)
if err == nil {
    fmt.Printf("StatusCode:%d, RequestId:%s,OwnerId:%s,OwnerName:%s,\n", output.StatusCode, output.RequestId, output.Owner.ID, output.Owner.DisplayName)
    for index, val := range output.Contents {
        fmt.Printf("Content[%d]-ETag:%s, Key:%s, LastModified:%s, Size:%d, StorageClass:%s\n",
            index, val.ETag, val.Key, val.LastModified, val.Size, val.StorageClass)
    }
}wosSdkVersion
```

## Put Object
```
var ak = "*** Provide your Access Key ***"
var sk = "*** Provide your Secret Key ***"
var endpoint = "https://your-endpoint"
 
wosClient, _ := wos.New(ak, sk, endpoint)
 
input := &wos.PutObjectInput{}
input.Bucket = bucketName
input.Key = objectKey
input.Metadata = map[string]string{"meta": "value"}
input.Body = strings.NewReader("Hello WOS")
output, err := wosClient.PutObject(input)
if err == nil {
    fmt.Printf("StatusCode:%d, RequestId:%s\n", output.StatusCode, output.RequestId)
    fmt.Printf("ETag:%s", output.ETag)
}
```

## Get Object
```
var ak = "*** Provide your Access Key ***"
var sk = "*** Provide your Secret Key ***"
var endpoint = "https://your-endpoint"
 
wosClient, _ := wos.New(ak, sk, endpoint)
 
input := &wos.GetObjectInput{}
input.Bucket = bucketName
input.Key = objectKey
output, err := wosClient.GetObject(input)
if err == nil {
    defer output.Body.Close()
    fmt.Printf("StatusCode:%d, RequestId:%s\n", output.StatusCode, output.RequestId)
    fmt.Printf("StorageClass:%s, ETag:%s, ContentType:%s, ContentLength:%d, LastModified:%s\n",
        output.StorageClass, output.ETag, output.ContentType, output.ContentLength, output.LastModified)
    p := make([]byte, 1024)
    var readErr error
    var readCount int
    for {
        readCount, readErr = output.Body.Read(p)
        if readCount > 0 {
            fmt.Printf("%s", p[:readCount])
        }
        if readErr != nil {
            break
        }
    }
}
```

## Delete Object
```
var ak = "*** Provide your Access Key ***"
var sk = "*** Provide your Secret Key ***"
var endpoint = "https://your-endpoint"
 
wosClient, _ := wos.New(ak, sk, endpoint)
 
input := &wos.DeleteObjectInput{}
input.Bucket = bucketName
input.Key = objectKey
output, err := wosClient.DeleteObject(input)
if err == nil {
    fmt.Printf("StatusCode:%d, RequestId:%s\n", output.StatusCode, output.RequestId)
}
```

## Multipart Upload
### Simple Multipart Upload
Please reference to `example/simple_multipart_upload_sample.go`

### Multipart Upload
Please reference to `example/multipart_upload_sample.go`

## Basic Functions
The examples locate at `wos_go_sample.go`

| Name |
| -- |
|  ListBuckets
|  HeadBucket
|  SetBucketLifecycleConfiguration
|  GetBucketLifecycleConfiguration
|  deleteBucketLifecycleConfiguration
|  ListObjects
|  ListObjectV2
|  ListMultipartUploads
|  DeleteObject
|  DeleteObjects
|  RestoreObject
|  InitiateMultipartUpload
|  UploadPart
|  CopyPart
|  ListParts
|  AbortMultipartUpload
|  PutObject
|  PutFile
|  HeadObject
|  GetObjectMetadata
|  GetObject
|  GetAvinfo

## Samples
| Samples |
| -- |
|  concurrent_copy_part_sample.go
|  concurrent_download_object_sample.go
|  multipart_upload_sample.go
|  create_folder_sample.go
|  delete_objects_sample.go
|  download_sample.go
|  list_objects_sample.go
|  list_objects_in_folder_sample.go
|  object_meta_sample.go
|  object_operations_sample.go
|  simple_multipart_upload_sample.go
|  temporary_signature_sample.go
