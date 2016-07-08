---
id: 448
title: Migrating Spring App to MicroServices App on AWS
date: 2015-12-07T01:00:45+00:00
author: Shannon Lal
layout: post
guid: http://www.javaadvent.com/?p=448
permalink: /2015/12/migrating-spring-app-to-microservices-app-on-aws.html
dsq_thread_id:
  - 4962579375
categories:
  - 2015
  - Amazon Web Services
  - java
  - Java Advent
  - Spring
---
<strong><u>Migrating Spring App to MicroServices App on AWS</u></strong>

&nbsp;

The company I am working for has recently gone through a migration of refactoring our code base from a monolithic application (Java Spring WAR) into a MicroServices Application hosted on the Amazon PAAS (specifically Beanstalk and CloudFront). As part of this blog post I have provided a small and simple Sales Demo application and will discuss the steps of what is required for refactoring the application so that it can be run within Beanstalk/S3/CloudFront environments.

For the purposes of this blog, I will be using a SalesTax demo application and the code can be found here (<a href="https://github.com/shannonlal/salesdemo">https://github.com/shannonlal/salesdemo</a>). This site will provide users a list of products and give them the ability to create an order and apply sales tax. I have created a more detailed guide, which includes steps for creating the different services in AWS. The guide can be found at this location (<a href="https://github.com/shannonlal/salesdemo">https://github.com/shannonlal/salesdemo</a>/AWS-MigrationGuide.pdf). The following is a diagram of the Spring Architecture:

<a href="http://www.javaadvent.com/content/uploads/2015/12/FirstPicture.png"><img class="alignnone size-medium wp-image-455" src="http://www.javaadvent.com/content/uploads/2015/12/FirstPicture-300x162.png" alt="FirstPicture" width="300" height="162" /></a>

&nbsp;

The above architecture is a pretty standard Spring architecture for most monolithic web applications. In our migration, we broke up our code and separated the backend services from the front end content JSPs(Now HTML), CSS and JS. The following is a diagram illustrating our model of how we controlled access:

<a href="http://www.javaadvent.com/content/uploads/2015/12/SecondPicture.png"><img class="alignnone size-medium wp-image-454" src="http://www.javaadvent.com/content/uploads/2015/12/SecondPicture-300x268.png" alt="SecondPicture" width="300" height="268" /></a>

<strong><u>Amazon Web Services </u></strong>

I am going to start by explaining at a high-level what these different components in AWS are and how we integrate them together.

&nbsp;

<strong><u>Route 53</u></strong>

Route 53 is a Domain Name Service(https://aws.<strong>amazon</strong>.com/<strong>route53</strong>/) which allows you to route traffic to different internal AWS services. In our model we used Route 53 to host our DNS servers (for example <a href="http://www.mycompany.com">www.mycompany.com</a>).

&nbsp;

<strong><u>S3</u></strong>

Amazon S3 (https://aws.amazon.com/s3/) is a simple storage service which allows you to store content (html, css, js files in buckets in the cloud). In this demo we will be using Amazon S3 to host the static content (html, css, and JS).

&nbsp;

<strong><u>Beanstalk</u></strong>

Beanstalk (https://aws.amazon.com/elasticbeanstalk/)is an application stack which will be used to host our individual services. Beanstalk has access to multiple stacks (Tomcat, PHP, Node, Ruby, Go, .Net). In this demo we will be using Beanstalk to host our different web services (as Spring WARS running on Tomcat).

&nbsp;

<strong><u>RDS</u></strong>

Amazon Relational Database Service (RDS <a href="https://aws.amazon.com/rds/">https://<strong>aws</strong>.<strong>amazon</strong>.com/<strong>rds</strong>/</a>) will be used to host our database. We will create an RDS database and our web services will be used to connect to the database.

&nbsp;

<strong><u>CloudFront</u></strong>

Amazon CloudFront is the glue that will tie all your different services together under one common URL. We will define an origin (which will correspond to our URL, defined in Route 53 <a href="http://www.mycompany.com">www.mycompany.com</a>). When the user hits this URL Route53 will route the traffic to CloudFront. CloudFront will host the content and push it to edge locations around the world. In CloudFront you are able to redirect traffic based on URL patterns. For example anyone coming to the default pattern (/*) can be redirected to a bucket in S3 which hosts your static content (i.e. html, css, images). If they come to say an API URL (/api/products) you can route them to a Beanstalk service in the backend.

<strong>Infrastructure Security</strong>

In our production systems we have all our web services hidden behind different VPCs and have implemented network rules to restrict access to our backend services. I do not think I will have time to address this in this blog, but will try to talk about this in my next.

&nbsp;

<strong>Application Security</strong>

One major component I have not included in the Sales Demo is Spring Security. In our application, we removed our Spring Security and replaced access control using an API Gateway. I will discuss this concept briefly at the end of this blog.

&nbsp;

NOTE: AWS is a very sophisticated and complex ecosystem that provides multiple ways to integrate these different services. The model I will be discussing is similar to the model which we implemented at our company.

&nbsp;

<strong>SalesTax Application Overview</strong>

&nbsp;

The SalesTax Demo application will look like a traditional Spring Application with one exception. The JSP pages do not follow the traditional Spring MVC model with data being passed from the controller and then the JSP pages rendering the view. Instead we are using Angular, which makes REST calls to the backend controllers and renders of the content in the browser. The reason that we are doing this is so that we can migrate our static content (html, css, js files) to S3 buckets and have our backend services run in beanstalk.

&nbsp;

I have created a guide, which provides step-by-step instructions with pictures on how to setup your environment in AWS. You can find a link to the document on github at this location. The rest of the document will provide a summary of the process with references to the guide. If you would like to try this on your own AWS setup I recommend you look at the detailed guide here ( <a href="https://github.com/shannonlal/salesdemo/AWS-MigrationGuide.pdf">https://github.com/shannonlal/salesdemo/AWS-MigrationGuide.pdf</a> ).

&nbsp;

<strong><u>Migration Process</u></strong>

&nbsp;

The following section will provide a high-level overview of the migration process. Again if you would like to try this out for yourself, I would recommend using the detailed guide.

&nbsp;

<strong>Deploy Application to Beanstalk</strong>

&nbsp;

The first step will be to build the application and deploy it into a beanstalk instance. To checkout the code please run the following command:

Git clone <a href="https://github.com/shannonlal/salesdemo">https://github.com/shannonlal/salesdemo</a> step0

&nbsp;

You can import the project into your IDE (Eclipse, NetBeans, STS, etc) or you can just build this from the command line. To build the project run the following commands:

&nbsp;

<em>mvn clean install</em>

&nbsp;

Once the WAR has been built, log into the AWS Adminstration console and deploy your WAR in a new Beanstalk Instance. For detailed instructions see the install guide

&nbsp;

<strong>Configure CloudFront to point to yourBeanstalk Instance</strong>

&nbsp;

Login into the Amazon Console and click on the CloudFront link. At this point you have two options:

-Use your own domain name( www.example.com)

-Use the default provided by Cloud Front(this will look something like <a href="https://xxxxxxxxxx.cloudfront.net">https://xxxxxxxxxx.cloudfront.net</a>).

If you already have your own domain name you can add it to Route 53. The following link provides detailed instructions on how to do this (<a href="http://docs.aws.amazon.com/gettingstarted/latest/swh/website-hosting-intro.html">http://docs.aws.amazon.com/gettingstarted/latest/swh/website-hosting-intro.html</a>). If you do not have your own you can just create a CloudFront Origin and it will give you a url.

&nbsp;

The goal of this step is to use CloudFront to map your url (either your own <a href="http://www.example.com">www.example.com</a> or generated <a href="https://xxxxxxxxxx.cloudfront.net">https://xxxxxxxxxx.cloudfront.net</a>) to your hosted application in BeanStalk. In CloudFront you will define a Web Distribution and then for that distribution you will define an Origin.   Origins in Cloud Front represent backend services (i.e. S3 buckets to host static content or Beanstalk Applications which host your Spring Apps). Finally, you will create a Behavior that will instruct CloudFront to map all requests of a certain url pattern to a specific Beanstalk Instance. For first step we will map all requests (/*) to the Beanstalk instance. In future steps will map all requests of the format (/api/*) to your Beanstalk instance and the rest (/*) will go to your S3 Bucket. Below is an image of what the screen for creating a Behavior would look like.

<a href="http://www.javaadvent.com/content/uploads/2015/12/ThirdPicture.png"><img class="alignnone size-medium wp-image-453" src="http://www.javaadvent.com/content/uploads/2015/12/ThirdPicture-280x300.png" alt="ThirdPicture" width="280" height="300" /></a>

<strong><u>Create RDS Postgres instance and connect to Beanstalk</u></strong>

&nbsp;

In this step we create a publicly accessible RDS instance and then connect to it from our pgAdmin tool to create the database. The sql script and updated code can be found by pulling down the step1 branch as follows:

&nbsp;

<em>Git clone </em><a href="https://github.com/shannonlal/salesdemo"><em>https://github.com/shannonlal/salesdemo</em></a><em> step1</em>

&nbsp;

The sql create script can be found in the following location

<em>src/resources/sql/ createSalesTax-DB-Postgres.sql</em>

&nbsp;

Once your database is created you can rebuild your project with maven using the following command:

<em>mvn clean install</em>

&nbsp;

Log back into your Amazon console and redeploy your latest war file. You will also need to append environment properties to your Beanstalk instance so it knows where to find your database. This can be done by clicking on Configuration, Software Configuration, and adding them to Environment Properties

<a href="http://www.javaadvent.com/content/uploads/2015/12/Third-A.png"><img class="alignnone size-medium wp-image-457" src="http://www.javaadvent.com/content/uploads/2015/12/Third-A-300x84.png" alt="Third-A" width="300" height="84" /></a>

If you reload your application you will see that it is now pulling the products from the database instance in AWS.

&nbsp;

<strong><u>Create an S3 Bucket and deploy Static Content to it</u></strong>

&nbsp;

In this step we are going to create an S3 bucket and will move our Static Content (html, css, images, etc) to it. To get the latest code for this we will need to pull down the latest changes from the git. Run the following command

&nbsp;

&nbsp;

<em>Git clone </em><a href="https://github.com/shannonlal/salesdemo"><em>https://github.com/shannonlal/salesdemo</em></a><em> step2</em>

<em> </em>

Log back into the Amazon Console and click on S3. Click on Create Bucket and create a new bucket.

&nbsp;

<a href="http://www.javaadvent.com/content/uploads/2015/12/ForthPicture.png"><img class="alignnone size-medium wp-image-452" src="http://www.javaadvent.com/content/uploads/2015/12/ForthPicture-300x150.png" alt="ForthPicture" width="300" height="150" /></a>

Once your bucket is created, click on Properties (upper right corner) and click on Static Website Hosting to enable hosting of content. Once your S3 bucket is ready you can transfer the static content of the project to S3. The code to transfer is in the following directory:

web/build/prod/

<strong><u>Update Cloud Front to reflect new origins</u></strong>

We will need to update CloudFront to redirect the requests to their appropriate origins. The first step will be to log into CloudFront and create an Origin for your newly created bucket. Once your Origin has been created you will need to modify the Behavior so that your default Behavior (*) now points to your static content in S3 and your API requests (/api/*) are redirected to your Elastic Beanstalk instance.  The following is a diagram of the proposed changes to CloudFront.

<a href="http://www.javaadvent.com/content/uploads/2015/12/FifthPicture.png"><img class="alignnone size-medium wp-image-451" src="http://www.javaadvent.com/content/uploads/2015/12/FifthPicture-300x236.png" alt="FifthPicture" width="300" height="236" /></a>

<strong><u>Redeploy Application</u></strong>

Once CloudFront has been updated and the status has changed to <strong>deployed</strong>, your static content, which is hosted in S3, will now be accessible by your CloudFront url. The only thing left to do is rebuild the sales demo application and redeploy it into Beanstalk. At this stage, all the front end code (html, js, css) has been moved to the <strong>web</strong> directory and the backend functionality is in the <strong>services</strong> directory. To rebuild your application run the maven command in <u>services directory</u>

&nbsp;

<em>mvn clean install</em>

&nbsp;

Log back into the Amazon Console and redeploy your Beanstalk application with the new WAR.

The above architecture is a good starting point for anyone who is looking at migrating their Spring application to a cloud based MicroServices. As part of your migration I would suggest looking at incorporating an API Gateway. There are a series of open source and commercially available API Gateways (Amazon released their API Gateway in July 2015, membrane-soa.org/, etc). The API Gateway will sit in between CloudFront and your backend services and will handle authentication and access control, and it will redirect your requests to the appropriate Beanstalk instance.   I have included a picture of the API Gateway below.

<a href="http://www.javaadvent.com/content/uploads/2015/12/SixthPicture.png"><img class="alignnone size-medium wp-image-456" src="http://www.javaadvent.com/content/uploads/2015/12/SixthPicture-300x187.png" alt="SixthPicture" width="300" height="187" /></a>

&nbsp;

&nbsp;

&nbsp;