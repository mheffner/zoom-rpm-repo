# Zoom RPM Repo Builder

Build your own custom RPM repo for tracking [Zoom](https://zoom.us)
client updates for Linux so that you can stay up to date using your
normal `dnf` software upgrade workflow. This Github repo's workflow
will download the Zoom RPM and sync it to a RPM repo structure in your
own AWS S3 bucket.

This Zoom RPM should work on Fedora and Redhat/CentOS 7.0+ distros.

This repo will also watch for Zoom client updates and create a PR when
a new version is released, allowing you to update your repo on your
own schedule.

![Update PR](contrib/pr.png?raw=true "PR")

# How to use

## Fork / clone this repo to your own Github

This will allow you to run the Github actions using your own custom S3
endpoints.

## Create S3 bucket on AWS and create IAM user

Create an S3 bucket and ensure that it can serve public assets
(private buckets are a TODO).

Set up an IAM user that can access the bucket with the following
credentials, replacing `${S3_BUCKET}` for your bucket name. Record the
access key and secret key for the IAM user, these will be used to sync
the repo changes to the bucket.

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:PutObject",
                "s3:GetObject",
                "s3:DeleteObject",
                "s3:PutObjectAcl"
            ],
            "Resource": "arn:aws:s3:::${S3_BUCKET}/*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "s3:ListBucket"
            ],
            "Resource": "arn:aws:s3:::${S3_BUCKET}"
        }
    ]
}
```

## Set the Github secret variables

This repo requires the following Github
[secrets](https://help.github.com/en/actions/configuring-and-managing-workflows/creating-and-storing-encrypted-secrets)
set in your repository config:

* `S3_BUCKET`: name of the S3 bucket, eg. "zoom-repo-123456"
* `S3_REGION`: region of the S3 bucket, eg. "us-east-1"
* `AWS_ACCESS_KEY_ID`: access key for IAM user
* `AWS_SECRET_ACCESS_KEY`: secret key for IAM user

## Create Yum repo config

Create the following file in `/etc/yum.repos.d/zoom.repo`:

```
[zoom]
name=zoom
baseurl=https://${S3_BUCKET}.s3.amazonaws.com/$basearch
enabled=1
gpgkey=https://zoom.us/linux/download/pubkey?version=6-3-10
gpgcheck=1
metadata_expire=6h
```

## Install

Now you'll be able to install the latest version of Zoom:

```
sudo dnf install zoom
```

## Track updates

The default Github action workflow will check for updates every six
hours and will open a PR anytime a new version is released. You can
merge this PR whenever you're comfortable and when merged to master it
will update the repo in S3. The next time you run a `dnf upgrade` it
will update to the latest zoom version.

**NOTE**: The PR that is opened will not automatically run the through
  the Github workflow steps that verify the version of Zoom can be
  downloaded. This is because PRs opened through automation do not
  trigger the workflows. For now you can close and then reopen the PR
  in order to trigger the workflow run, or just merge directly to
  master to release.

# TODO

* Support for i386
* Non-public S3 buckets using Yum repo credentials.
* Better workflow triggering for automated PRs.
