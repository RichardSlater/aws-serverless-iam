With the rise of AWS Lambda and [Serverless Architectures][serverless], which are often best
developed in AWS itself rather than trying to emulate locally, it is best to
give developers the ability to freely deploy to a development account where they
can be most productive.

It is important to understand that by giving too wider permissions we are
exposing a sizable attack surface that could significantly impact your bottom
line.

The [principle of least privilege][polp] when applied to services states that:

> A service must be given no more privilege than necessary to perform it's job.

This principle must apply equally to [user accounts][sqa-lua], service accounts
and [Amazon IAM Policies][iam-lua]. Serverless Framework Deployment profiles is
no exception.

It may be tempting to get up and running quickly by granting
**Full Administrator** privileges to your IAM Role as the Serverless Framework
documentation suggests however this would result in highly privileged
credentials being left on your development laptop.

## Getting Setup with Serverless Framework

If you have used Serverless Framework you will have worked through setting up a
new project, first you create your new project with:

    serverless create --template aws-nodejs --path my-service

You will also have configured your AWS Credentials with:

    serverless config credentials --provider aws --key YOUR_KEY --secret YOUR_SECRET

## Applying the Principle of Least Privalage

### Overview

![Architecture][architecture]

#### Design Principles

 - **Least Privilege** - the minimum working set of permissions will be assigned
   to any given user or service.
 - **Separation of Duties** - checks and balances ensure that multiple users are
   required to be able to view or mutate production.
 - **Apply Policy to Groups not Users** - never applying permissions or Policy
   directly to users, only ever assign policy to groups.
 - **No Shared Credentials** - by creating a One-to-one Mapping between
   developers and IAM Users we have complete traceability of users rights and
   actions on AWS.

### Instructions

If you have followed the [official instructions][serverless-creds] then you will
have granted the above credentials AdministratorAccess, however you don't need
to follow this advice:

 1. Install [Yeoman][yeoman] with

    `npm install -g yo`

 2. Install the [Serverless Policy Generator][policy-generator] with

    `npm install --global generator-serverless-policy`

 3. Then simply generate the policy with the following command:

    `yo serverless-policy`

 4. Enter the name of your service, make sure it is named exactly the same as
 your `service` field in `serverless.yml`, in the above example `my-service`.

 5. I recommend using a different policy for each SDLC Phase / Stage, for the
 next question answer `dev`.

 6. If it is possible to restrict your deployments to a specific region this
 will further reduce your surface area, I choose `eu-west-2` (London).

 7. If your service relies on DynamoDB and S3 then the policy generator can
 provision the appropriate permissions within the policy to configure it.

 8. Assuming you selected all of the above options then you will end up with a
file named `my-service-dev-eu-west-2-policy.json`.  The contents of this file
can be loaded into an AWS IAM Policy:

  1. Open [Identity and Access Management][iam] in the AWS Management Console.

  2. Click **Policies**.

  3. Click **Create policy**.

  4. Enter `ServerlessFrameworkDeployment-MyService-Dev` as the name of your
  policy, add an appropriate description and paste in the contents of your
  `my-service-dev-eu-west-2-policy.json`:

     ![Screenshot of IAM Policy][iam-screenshot]

  5. Save the policy and [attach it to an IAM Group][iam-attachgroup].

  6. If you have a pre-existing user with excessive permissions for Serverless
  Framework Deployments you should strip of it's existing policies and groups
  and then add the user to the group you created above.  Alternatively you may
  create a new user and cut-over to the new credentials on developer
  workstations.

With the above created you should be in a position where your workstations are
capable of deploying to the **Dev** stage of **my-service** and nothing else.

By repeating the instructions above for each SDLC phase and each service you
have the capability of creating a structured and least privilege credential
management system for your company.  Where by users can be assigned the
permissions they need to deploy their services to development, it would then
usually be the responsibility of Quality Assurance to deploy services to a Test
environment.

From a sustainability and accountability perspective it would be wise to use
automation engines such as Jenkins or TeamCity to orchestrate the Development,
QA, SIT and Production deployment processes so while the above diagram describes
a QA user, that may actually be a service account.

If you want to know more about Serverless Architectures, DevOps and Cloud First
Adoption patterns, then get in touch with me on
[richard.slater@amido.com][email] or [LinkedIn][linkedin].

  [serverless]: https://aws.amazon.com/lambda/serverless-architectures-learn-more/
  [polp]: https://en.wikipedia.org/wiki/Principle_of_least_privilege
  [sqa-lua]: https://www.sqa.org.uk/e-learning/NetInf103CD/page_03.htm
  [iam-lua]: http://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html#grant-least-privilege
  [serverless-creds]: https://serverless.com/framework/docs/providers/aws/guide/credentials/
  [yeoman]: http://yeoman.io/
  [policy-generator]: https://github.com/dancrumb/generator-serverless-policy
  [iam]: https://console.aws.amazon.com/iam/home
  [iam-screenshot]: https://rawgit.com/RichardSlater/aws-serverless-iam/master/images/iam-screenshot.png
  [architecture]: https://rawgit.com/RichardSlater/aws-serverless-iam/master/images/leastprivalage.png
  [iam-attachgroup]: http://docs.aws.amazon.com/IAM/latest/UserGuide/id_groups_manage_attach-policy.html
  [email]: mailto:richard.slater@amido.com
  [linkedin]: https://www.linkedin.com/in/devopsrichard
