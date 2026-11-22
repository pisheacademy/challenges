# Challenge 02 — Nobody gets more than they need

This is the second challenge. The rules are the same as the first one. The difference is that this time all three questions are about one service: IAM.

In the first challenge you told me what a user, a role and a policy are. Now I want to see you use them. Read a policy and know what it really allows. Keep a secret out of a config file. Know what to do on the bad day when a key leaks. A lot of the expensive mistakes on AWS are not about servers. They are about someone having more access than they needed, for longer than they needed it.

Three questions. Write the answers in Persian, on paper, in your own words. A short answer that is yours beats a long one you copied from a page. If I can see you transcribed a document instead of explaining it, I will treat it as not yours.

At the end of each answer, on the same page, write the references you actually used. A doc title and a link is enough. If you used nothing, write that. Do not leave the bottom of the answer empty.

## How you submit

Write it by hand. Take a photo of each answer. The photo has to be readable: light on the page, the whole page in the frame, your handwriting big enough that I can read the Persian and the references without zooming until it breaks.

Open a pull request with those photos.

In this folder, `competitions/iran/nsc/23rd/cloud-computing/02/`, create a new folder. The folder name is your GitHub username, the same spelling, the same caps. Put the photos in that folder. Name them by question so I do not have to guess:

- `01.jpg` for the first answer
- `02.jpg` for the second
- `03.jpg` for the third

If one answer needs a second page, name it `01-2.jpg`, and so on. The references stay at the end of that answer, not in a separate file.

If your username is `ali-cloud`, I should see something like:

`competitions/iran/nsc/23rd/cloud-computing/02/ali-cloud/01.jpg`

One person, one folder, one pull request. Do not edit this README. Do not put files in someone else's folder. Do not type the answer into GitHub. I want the paper.

## 1. Read the policy before you trust it

The IAM user `report-bot` has this policy attached:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:*",
      "Resource": [
        "arn:aws:s3:::nsc-reports",
        "arn:aws:s3:::nsc-reports/*"
      ]
    },
    {
      "Effect": "Deny",
      "Action": "s3:DeleteObject",
      "Resource": "arn:aws:s3:::nsc-reports/*"
    }
  ]
}
```

There is no bucket policy on any bucket, and no other policy in the account says anything about this user.

For each case, tell me yes or no, and the reason in one line:

- `report-bot` downloads a file from `nsc-reports`.
- `report-bot` deletes a file from `nsc-reports`.
- `report-bot` lists the files in another bucket, `nsc-backups`.
- Later, someone adds `report-bot` to a group whose policy allows `s3:DeleteObject` on every bucket. Can it delete a file from `nsc-reports` now?

The yes or no is the easy part. The reason is where the points are. When you are done, write the rule AWS follows to decide between allow and deny, in two or three sentences of your own.

## 2. No keys in the code

A teammate wrote a small app. It runs on an EC2 instance and uploads files to an S3 bucket. To make it work, they created an IAM user, made an access key for it, and pasted the key ID and the secret into a config file on the server. The app works.

I am still not happy. Tell me why. Give me at least two real problems with this setup, not "it is bad practice".

Then fix it with a role. A role has two policies that answer two different questions: who is allowed to become this role, and what can it do once it has. For this app, tell me what each one should say. Plain words are fine. You do not have to write JSON.

Last part. With the role, the app still needs credentials to call S3. Where do they come from, and how long do they live? Why is that better than the key in the config file?

## 3. The key is out

Go back to the old version of that app, the one with the key in the config file. Somebody pushed that file to a public GitHub repository. You notice one hour later.

Tell me what you do, in order, first step first. For each step, one line on why it sits in that place and not later. Also tell me why "I deleted the file and pushed again" does not close the problem.

Then the part I care about more. Imagine the key belonged to a user with `AdministratorAccess`. Now imagine it belonged to a user that could only put files in one bucket. Same leak, very different day. Give me at least three things you would set up in IAM before anything goes wrong, so the next leak hurts less. For each one, tell me which part of the damage it cuts.
