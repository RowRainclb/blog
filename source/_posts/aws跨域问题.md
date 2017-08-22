---
title: aws跨域问题
date: 2016-10-24 12:48:55
tags:
    - aws
---
1 跨域问题
  如果步骤都正确还一直报跨域问题
原因可能由于未加如下配置导致:
``` javascript
<ExposeHeader>x-amz-server-side-encryption</ExposeHeader>
		<ExposeHeader>x-amz-request-id</ExposeHeader>
		<ExposeHeader>x-amz-id-2</ExposeHeader>
		<ExposeHeader>ETag</ExposeHeader>
```
   则尝试如下设置:

AWS S3 CROS设置如下:
``` javascript
<?xml version="1.0" encoding="UTF-8"?>
<CORSConfiguration xmlns="http://s3.amazonaws.com/doc/2006-03-01/">
	<CORSRule>
		<AllowedOrigin>*</AllowedOrigin>
		<AllowedMethod>GET</AllowedMethod>
		<AllowedMethod>PUT</AllowedMethod>
		<AllowedMethod>POST</AllowedMethod>
		<AllowedMethod>DELETE</AllowedMethod>
		<MaxAgeSeconds>3000</MaxAgeSeconds>
		<ExposeHeader>x-amz-server-side-encryption</ExposeHeader>
		<ExposeHeader>x-amz-request-id</ExposeHeader>
		<ExposeHeader>x-amz-id-2</ExposeHeader>
			<ExposeHeader>ETag</ExposeHeader>
		<AllowedHeader>*</AllowedHeader>
	</CORSRule>
</CORSConfiguration>
```
参考跨源资源共享
			http://docs.aws.amazon.com/zh_cn/AmazonS3/latest/dev/cors.html
			http://docs.aws.amazon.com/zh_cn/sdk-for-javascript/v2/developer-guide/s3-example-photo-album.html

