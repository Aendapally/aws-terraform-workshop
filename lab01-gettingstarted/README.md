## Getting Started with Terraform Enterprise

Duration: 45 minutes

This lab demonstrates how to connect Terraform Enterprise to a source code management system (GitHub) and create a Workspace that can apply the Terraform configuration when changes are committed.

**Notes:**
- This branch is for use with a student's own AWS credentials and is intended to run on Terraform Enterprise
  - Note: This lab can also be run locally, see [local.md](local.md) for steps.
- In this branch a RSA public and private key pair will be generated by Terraform using the Terraform [tls_private_key](https://www.terraform.io/docs/providers/tls/r/private_key.html) provider. This will be used for SSH access.

**Tasks:**
- Task 1: Connect GitHub to TFE and Fork a GitHub Repo
- Task 2: Configure Variables
- Task 3: Queue a Plan
- Task 4: Edit Code on GitHub to Use Variables instead of file
- Task 5: Confirm and Apply the Plan

### Terraform Enterprise

Pre-requisites:
- An AWS account with IAM user credentials: `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY`
- Access to a Terraform Enterprise Server. This Lab will use the hosted SaaS version on [app.terraform.io](https://app.terraform.io), and you can [sign up](https://app.terraform.io/account/new) for a free trial.
- A [GitHub](https://github.com/) account.


### Task 1: Connect GitHub to TFE and Fork a GitHub Repo

Using a GitHub repository will allow us to use source control best practices on our infrastructure configs.

Populating variables to Terraform Enterprise will give Terraform Enterprise our AWS credentials so it can run Terraform on our behalf.

Connecting Terraform Enterprise to GitHub will give us a continuous integration style of workflow for managing infrastructure.

#### Step 1.1: Fork the repo

Visit this GitHub repository and fork it so you have a copy in your own GitHub account:

    https://github.com/hashicorp/demo-terraform-101

Optionally, clone the repository to your local machine (if you prefer to edit code locally instead of in the browser).

```bash
$ git clone https://github.com/$USER/demo-terraform-101.git
```

We will work with the `after-tfe` branch. If you choose to work locally, check out this branch:

```bash
$ git checkout -t origin/after-tfe
```

#### Step 1.2: Connect GitHub to TFE

Now go to https://app.terraform.io

We are going to connect our GitHub repository to Terraform Enterprise. If you don’t already have an organization in Terraform Enterprise, create one. If you were invited to an existing TFE organization, you can access that as well.

You’ll see an empty page where your workplaces will be. Click the "New Workspace" button in the top right.

We can’t create a workspace yet because there is no source repository to connect to. Let’s setup GitHub by creating and connecting an OAuth client.

Go to [GitHub](https://github.com/settings/profile) and find your Settings page, accessed from the menu on your avatar.

At the bottom of the settings page is "Developer Settings." Click that.

Now you’ll see a list of OAuth Apps. Click the "New OAuth App" button.

This form is straightforward with one exception. None of these fields are critical except the one that we’ll have to leave blank initially..."Authorization Callback URL".

![TFE](images/tfe-basics/01.png "TFE")

Type in any name for the "Application Name." Use "https://app.terraform.io" as the URL.

Leave the final "Authorization Callback URL" field blank for now. Click "Register Application."

You’ll need the Client ID and Client Secret from the resulting page. Leave the page open and copy the "Client ID" to your clipboard.

![TFE](images/tfe-basics/02.png "TFE")

Now go back to Terraform Enterprise.

Go to your OAuth Configuration by clicking the downward facing arrow by your organization’s name and choosing "Organization Settings." You’ll See "OAuth Configuration."

Paste in the copied Client ID and Client Secret from GitHub. Scroll down and click the button to "Create OAuth Client."

Now we finally have a "GitHub Callback URL" to use! Copy it from Terraform Enterprise and we’ll take it back to GitHub.

![TFE](images/tfe-basics/03.png "TFE")

Back at GitHub, scroll down and paste the URL into "Authorization Callback URL." Save the form.

![TFE](images/tfe-basics/04.png "TFE")

We’re almost done. Back at Terraform Enterprise, click the purple "Connect Organization" button. You'll see an authorization screen at GitHub. Click to approve.

![TFE](images/tfe-basics/05.png "TFE")

Back at Terraform Enterprise, you’ll see that it’s connected.

**NOTE:** For full capabilities, you can add your private SSH key which will be used to clone repositories and submodules. This is especially important if you use submodules and those submodules are in private repositories. That isn’t the case for us so I’ll leave that up to you.

### Step 1.3: Create a workspace in TFE

Finally, we’re ready to fully create a Terraform Enterprise workspace. Go to https://app.terraform.io and click the "New Workspace" button at the top right.

Give it a name such as "training-demo".

GitHub is our only VCS connection. Click the "Repository" field and you’ll see a list of available repositories in an auto-complete menu. Find the `demo-terraform-101` repo. If yours isn’t here, refresh the page.

![TFE](images/tfe-basics/06.png "TFE")

Terraform Enterprise can deploy from any branch. We'll use the `after-tfe` branch which has been minimally modified to work with Terraform Enterprise.

![TFE](images/tfe-basics/07.png "TFE")

Click the "More Options" link and scroll down to "VCS Branch." Type `after-tfe` to use that branch.

You’ll see a screen showing that a Terraform Enterprise workspace is connected to your GitHub repository. But we still need to provide Terraform with our secret key, access key, and other variables defined in the Terraform code as variables.

## Task 2: Configure variables

Go to the "Variables" tab.  On the variables page, you'll see there are two kinds of variables:

- Terraform variables: these are fed into Terraform, similar to `terraform.tfvars`
- Environment variables: these are populated in the runtime environment where Terraform executes

In the top "Terraform Variables" section, click "Edit" and add keys and values for all the variables in the project's `variables.tf` file. The only one you'll need initially is `identity` which is your unique animal name.

### Step 2.1: Enter AWS Credentials

There is also a section for environment variables. We'll use these to store AWS credentials.

Click "Edit" and add variables for your AWS credentials.

```bash
AWS_ACCESS_KEY_ID="AAAA"
AWS_SECRET_ACCESS_KEY="AAAA"
AWS_DEFAULT_REGION="us-west-2"
```

Click the "Save" button.

## Task 3: Queue a Plan

For this task, you'll queue a `terraform plan`.

### Step 3.1: Queue a plan and read the output

Click the "Queue Plan" button at the top right.

Go to the "Runs" tab, or "Latest Run". Find the most recent one (there will probably be only one).

Scroll down to where it shows the plan. Click the button to "View Plan." You’ll see the same kind of output that you are used to seeing on the command line.

We'll make another change from GitHub before running this plan, so click "Discard Plan."

## Task 4: Edit Code on GitHub to Upgrade the AWS Provider Version

Edit code on GitHub to upgrade the AWS provider version to `>= 1.20.0`.

You'll make a pull request with these changes and observe the status of the pull request on GitHub.

### Step 4.1

On GitHub, find the "Branch" pulldown and switch to the `after-tfe` branch.

Navigate to `main.tf`. Find the pencil icon. Click to edit this file directly in the browser.

![TFE](images/tfe-basics/09.png "TFE")

Edit the code to match the lines below.

```bash
provider "aws" {
  # MODIFY this line to look for 1.20.0 or greater
  version = ">= 1.20.0"
}
```

Scroll to the bottom and select the option to "Create a new branch and start a pull request."

![TFE](images/tfe-basics/10.png "TFE")

You’ll be taken to a screen to create a pull request. Click the green "Propose file change" button. The page will be pre-populated with your commit message. Click "Create pull request."

After a few seconds, you'll see that Terraform Enterprise checked the plan and that it passed.

![TFE](images/tfe-basics/12.png "TFE")

To see what was checked, click "Show all checks" and click "Details" next to the line that says "Terraform plan has changes."

Merge the pull request to the `after-tfe` branch with the big green "Merge pull request" button. Click the "Confirm merge" button.

## Task 5: Confirm and Apply the Plan

### Step 5.1: Confirm and `apply`

Back at Terraform Enterprise, find the "Current Run" tab. Click it and you’ll see the merge commit has triggered a plan and it needs confirmation.

![TFE](images/tfe-basics/13.png "TFE")

Scroll to the bottom of the run and confirm the `plan`. At the bottom of the page you’ll see a place to comment (optional) and click "Confirm & Apply."

![TFE](images/tfe-basics/14.png "TFE")

This will queue a `terraform apply`.

Examine the output of `apply` and find the IP address of the new instance. The output looks like what you’ve previously seen in the terminal. Copy the `public_ip` address and paste it into your browser. You'll see the running web application.

![TFE](images/tfe-basics/15.png "TFE")

## Task 6: Destroy

To clean up, destroy the infrastructure you've just created.

### Step 6.1: Configure CONFIRM_DESTROY variable

Go to the "Settings" tab in Terraform Enterprise and scroll to the bottom. Note the instructions under "Workspace Delete." We want to destroy the infrastructure but not necessarily the workspace.

You'll need to create an environment variable (not a Terraform variable) named `CONFIRM_DESTROY` and set it to `1`.

Go to the "Variables" tab and do that.

![Confirm Destroy](images/confirm-destroy.png "Confirm Destroy variable")

Click "Add" and "Save".

### Step 6.2: Queue destroy plan

It's sometimes necessary to queue a normal plan and then queue the destroy plan.

At the top of the page, click the "Queue Plan" button. The plan will run and detect that no changes need to be provisioned.

Now go back to the "Settings" tab. Scroll to the bottom and click "Queue Destroy Plan." Note the messages under "Plan" that indicate that it will destroy several resources.

Click "Confirm and Apply." After a few seconds, your infrastructure will be destroyed as requested.