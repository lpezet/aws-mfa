# awf-mfa

Most AWS users are privileged users and for that reason, security must be taken into consideration for those users.
Key pairs (Access and Secret keys) are a good first level protection when it comes to authentication against AWS resources.
However, keys are static and even if rotated, may get compromised (e.g. https://securosis.com/blog/my-500-cloud-security-screwup).
The best next step for security is to use another factor for authentication, preferrably something temporary, like one-time passwords. AWS supports MFA and AWS users can leverage their MFA Device to elevate authentication security.
It's however a tad bit tedious to deal with MFA with AWS CLI. To smooth it out and encourage adoption, we developed a script called aws-mfa to take care of 1) creating temporary credentials using one's key pairs and MFA Device id and 2) storing those temporary credentials as a new profile to use it with aws cli.

In a nutshell, here's how to use aws-mfa once all the setup has been done:
```bash
aws-mfa myprofile _token-from-mfa-device_
# simply add -mfa to profile specified aove, like so:
aws --profile myprofile-mfa ec2 do-something-that-would-be-denied-otherwise-without-mfa
```

## Installation


```
git clone https://github.com/lpezet/aws-mfa.git
cd aws-mfa
sudo ln -s aws-mfa /usr/local/bin/aws-mfa
```

Before using `aws-mfa` you'll need to install a few dependencies.
The first being, of course, `awscli`. The second is `jq`.
Here's how to go about it using `homebrew`:

```
brew install awscli
brew install jq
```


## Setup

### aws-cli setup

Once the `aws-mfa` script has been installed, you must configure your local aws-cli. For this consult aws-cli documentation.
It basically boils down to:

```bash
aws configure --profile someprofile
```

You'll be prompted for information:

```
AWS Access Key ID [None]: _enter here your access key from AWS Console_
AWS Secret Access Key [None]: _enter here your secret key from AWS Console_
Default region name [None]: us-east-1
Default output format [None]: json
```

### Create MFA Device using AWS Console

Log in to AWS Console. Look for "IAM" Service, then click on "Users" in the left navigation, then find your user in the list.
Click on your username, then click on the "Security Credentials" tab.
If it's not already setup, setup your MFA Device now.
Once setup, copy the "arn" of your device.
It should be in the form of `arn:aws:iam::377201731123:mfa/_username_`.

### Store MFA Device ARN in aws profile

We will now create a new property in your aws cli profile to hold the arn of your MFA Device.
Simply run the following command:

```bash
aws --profile someprofile configure set mfa_serial _your_mfa_device_arn_
```

### Test

Now is the time to test your setup.

```bash
aws-mfa someprofile _token-from-your-mfa-device_
```

This will create/reset the profile `someprofile-mfa``. To test it worked properly:

```bash
aws --profile someprofile-mfa iam get-user
```

The response (error or not) should display your current aws username (if your AWS user has been granted permission to query get-user from IAM service).



