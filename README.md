# [AWS Serverless Image Handler](https://github.com/aws-solutions/serverless-image-handler)

AWS Serverless Image Handler를 Fork 해서 제가 사용하면서 필요한 기능을 추가한 Repository 입니다.

## To-do list

- [x] Multi Bucket 사용 가능하도록 기능 추가
- [x] Key path에 따른 default image 사용 가능 하도록 기능 추가

> ## [Solution Overview](https://github.com/aws-solutions/serverless-image-handler#solution-overview)
>
> The Serverless Image Handler solution helps to embed images on websites and mobile applications to drive user engagement. It uses Sharp to provide high-speed image processing without sacrificing image quality. To minimize costs of image optimization, manipulation, and processing, this solution automates version control and provides flexible storage and compute options for file reprocessing.
>
> This solution automatically deploys and configures a serverless architecture optimized for dynamic image manipulation. Images can be rendered and returned spontaneously. For example, an image can be resized based on different screen sizes by adding code on a website that leverages this solution to resize the image before being sent to the screen using the image. It uses Amazon CloudFront for global content delivery and Amazon Simple Storage Service (Amazon S3) for reliable and durable cloud storage.
>
> For more information and a detailed deployment guide, visit the Serverless Image Handler solution page.

## Runtime Settings

**Runtime** : Node.js 14.x

**Handler** : image-handler/index.handler

## Environment variables

| Variable                        | Meaning                                       |
| :------------------------------ | :-------------------------------------------- |
| `AUTO_WEBP`                     | AUTO_WEBP 설정 여부 (Yes / No)                |
| `SOURCE_BUCKETS`                | Serverless Image Handler로 접근할 버킷 리스트 |
| `ENABLE_DEFAULT_FALLBACK_IMAGE` | 기본 이미지 사용 활성화 여부 (Yes / No)       |
| `DEFAULT_FALLBACK_IMAGE_BUCKET` | 기본 이미지 파일이 있는 버킷 이름             |
| `DEFAULT_FALLBACK_IMAGE_KEY`    | 기본 이미지 파일 이름                         |
| `CORS_ENABLED`                  | CORS 활성화 여부                              |
| `CORS_ORIGIN`                   | CORS Origin 설정                              |

## 추가 기능 1. checkImageBucket()

### image-handler/image-request.js

    환경 변수로 설정한 sourceBuckets에 targetBucket이 있는지 확인하는 메서드

기존 sourceBuckets에서는 가장 앞 Bucket만 반환하도록 되어있었지만,
필요에 따라 multi Bucket을 사용할 수 있도록 targetBucket의 존재 유무를 확인하도록 추가함.

```javascript
/**
 * Parses the name of the appropriate Amazon S3 bucket to source the original image from.
 * @param sourceBuckets Lambda source buckets list.
 * @param targetBucket Image handler request buekct name alias.
 * @returns The name of the appropriate Amazon S3 bucket.
 */
checkImageBucket(sourceBuckets, targetBucket) {
    const { DEFAULT_FALLBACK_IMAGE_BUCKET } = process.env;
    let find_bucket_name = DEFAULT_FALLBACK_IMAGE_BUCKET;

    sourceBuckets.forEach((currentElement) => {
        if (currentElement.includes(targetBucket)) {
            find_bucket_name = currentElement;
        }
    });

    return find_bucket_name;
}
```

## 추가 기능 2. 각 폴더에 대한 default 이미지 사용 가능하도록 기능 추가

### image-handler/image-request.js

    s3Client.getObject() 메서드를 이용해서 현재 Key 가 유효한지 확인

Key가 잘못 되었다면, Key의 path에 default.jpg 이미지가 있는지 확인하여 있다면 해당 path의 default.jpg 이미지를 반환하고, 없다면 환경변수로 지정해 준 Bucket의 default.jpg 이미지를 반환 함.

```javascript
   /**
     * Initializer function for creating a new image request, used by the image handler to perform image modifications.
     * @param event Lambda request body.
     * @returns Initialized image request information.
     */
    setup(event) {
        return __awaiter(this, void 0, void 0, function* () {
            try {
                ...
                try {
                    const imageLocation = { Bucket: imageRequestInfo.bucket, Key: imageRequestInfo.key };
                    yield this.s3Client.getObject(imageLocation).promise();
                }
                catch {
                    const originKey = imageRequestInfo.key;
                    imageRequestInfo.key = originKey.substring(0, originKey.lastIndexOf('/') + 1) + "default.jpg";
                }
                ...
            }
            catch {...}
        });
    }

}
```

## Deploy

- image-handler
- node_modules
- solution-utils

3개의 폴더를 압축(zip)하여 업로드 하면 됩니다.
