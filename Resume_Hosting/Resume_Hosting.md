# RESUME HOSTING

In this project I will try to host my resume on AWS S3 and then try to host it on a custom domain.

Final resume : https://harsh-resu.me

(Since I have used my resume in pdf format it may ask you to download it if you have an android device)

Error Page : https://harsh-resu.me/error.html

You can also rech the error page if you try to access a random route.

![architecture](./ASSETS/23.png)

## STEPS

### AWS S3

#### 1. Create a bucket

![creating a bucket](./ASSETS/1.png)
This is the main bucket that will contain my resume as well as the error page.

![creating redirect bucket](./ASSETS/2.png)

This is the bucket that will redirect all the traffic to the main bucket.

![allow public access](./ASSETS/3.png)

This is the bucket policy that will allow public access to the bucket.

#### 2. Upload files

![uploading files](./ASSETS/4.png)

Here i upload my resume in the s3 bucket. I also upload the error page in the same bucket. We can leave the redirect bucket empty as it will only be used to redirect traffic to the main bucket.

### 3. Bucket policy

This is the bucket policy that will allow public access to the bucket.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowPublicReadAccess",
      "Effect": "Allow",
      "Principal": "*",
      "Action": ["s3:GetObject", "s3:GetObjectVersion"],
      "Resource": "arn:aws:s3:::harsh-resu.me/*"
    }
  ]
}
```

we can also generate policy ourselves using the policy generator provided by AWS.

[POLICY GENERATOR](https://awspolicygen.s3.amazonaws.com/policygen.html)

The page will look something like this.

![policy generator](./ASSETS/8.png)

After we fill all the fields we are ready to generate our desired policy.
![generated policy](./ASSETS/9.png)

We can now copy this policy and paste it in the bucket policy section.

![editing s3 policy](./ASSETS/10.png)

This policy allows any user to perform the getobject action on the bucket. Hence making the bucket public.

#### 4. Enable static website hosting

This was the most important step. Here we enable static website hosting and provide the error page as the error document. We also provide the index document as the resume.

![before static website hosting](./ASSETS/5.png)

![configuring our bucket](./ASSETS/6.png)

![after static website hosting](./ASSETS/7.png)

Now we have hosted a static website on AWS S3. We can access it using the endpoint provided by AWS. But this is an http endpoint and we want to use a custom domain. So we will use AWS CloudFront to do that.

### AWS CloudFront

AWS cloudfront is the worlds largest CDN. It is used to deliver content to users with low latency and high transfer speeds. It also provides us with a custom domain.

Since my bucket was in the us-east-1 region and I am in India this would mean any time I access my resume it would take a lot of time to load. So I used cloudfront to reduce the latency. Since cloudfront is a CDN it will cache my resume in the edge locations and hence reduce the latency. Hence the page will load faster in any location in the world.

#### 1. Create a distribution

We create a distribution and select the bucket as the origin. We also select the http endpoint as the origin domain name.

![creating a distribution](./ASSETS/11.png)

![cache settings](./ASSETS/13.png)

We can leave the cache settings as default or we can optimize it for our workload.

AWS has now provided us with an endpoint but this is still now a domain name we want however now it is https. We can now use this endpoint to access our resume.

<!-- <> (discuss this at the end and remove it from here) -->
<!-- ![cloudfront settings](./ASSETS/12.png) -->

<!-- <> (thi also) -->

### Route 53

Route 53 is a DNS service provided by AWS. It is used to route traffic to different services. We will use it to route traffic to our cloudfront distribution.

#### 1. Create a hosted zone

I used [https://www.namecheap.com/](Namecheap) to purchase my .me domain name you can also use any other domain name provider. If you use AWS to do this then you can skip the step of adding a hosted zone.

![create hosted zone](./ASSETS/14.png)

![edit nameservers on namecheap](./ASSETS/15.png)

AWS provides us with name servers in our hosted zone we can copy these name servers and paste them in the name servers section of our domain name provider.

> **_NOTE:_** In AWS dashboard the nameservers have **-> . <-** (a dot that means root nameserver) at the end of them. We need to remove them before pasting them in our domain name provider.

> **_NOTE:_** It may take some time for the nameservers to propagate. So if you are not able to access your resume using your domain name then wait for some time and try again.

#### 2. Create a record set

We choose simple routing then we create a record pointing to our s3 bucket

![create record set](./ASSETS/16.png)

### 3. Adding redirection

We now have to revisit the redirect bucket and edit its static website hosting rules

![redirect bucket](./ASSETS/17.png)

![redirect record](./ASSETS/18.png)

now we add a redirect record to our domain name.

Now we have added the cutom domain name hence we can access our resume using our domain name. But this is still not secure as we are using http and not https. So we will use AWS certificate manager to get a certificate for our domain name. For now see a caution symbol on the left of our domain name. This means that our website is not secure.

### AWS Certificate Manager

Now we need to get a certificate for our website this will allow us to use https instead of http. We can get a certificate for free using AWS certificate manager. A certificate indicates that the website is secure and the data is encrypted as well as the domain name is verified.

#### 1. Request a certificate

We request a certificate for our domain name.

![request certificate](./ASSETS/19.png)

![certificate form](./ASSETS/20.png)

#### 2. create CNAME record

CNAME records are used to map a domain name to another domain name. We create a CNAME record in our hosted zone and point it to our cloudfront distribution.

![create CNAME record](./ASSETS/21.png)

Now everthing is done just a bit more work with cloud front and we will be done with our resume.

We edit the settings of our cloudfront distribution and add the certificate to it. We also change the http to https in the origin settings. We also change the viewer protocol policy to redirect http to https. If not done before hand. We also change the alternate domain names to our domain name.

![cloudfront settings](./ASSETS/22.png)

Similar to how we created a distribution for our main bucket we also will now create a redirect distribution and we are now all done.

VISIT AT : [harsh-resu.me](https://harsh-resu.me)
