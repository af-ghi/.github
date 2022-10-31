# Technical Problem and Solution
GitHub Technical Interview


Problem:

"Our security team is asking for help ensuring **proper reviews** are being done to code being **added** into our repositories.
We have hundreds of repositories in our organization. What is the best way we can achieve at **scale**? 
We are new to some of the out-of-the-box settings and the GitHub API. Can you please help us create a solution that will accomplish this for our security team?"

The technical solution to accomplish this is to **listen for organization events** to know when a **repository has been created**.

When the repository is created, please automate the protection of the default branch. Notify yourself with an @mention in an issue
within the repository that outlines the protections that were added.

*The first thing that comes to mind is that if this customer or organization is asking these kinds of questions, they probably haven't changed too many settings.*
*I would need to check if they are using all default rules for repositories in their organization, and if they have customized some settings to see what those are.*

*It sounds like at the very least they will need to set some basic status checks, set some basic permissions,* 
*and then set up some branch protection rules and then the apis to create the issues.*

*Since they are concerned about scale, i.e. they report they have hundreds of repos, I wonder if a rule which requires a too many code reviewers is*
*likely to work for them since that would require too much manual intervention (and hard to test).*

___

### I think the following would be a good start:

- set a default branch name for any repository
- make sure a README.md is created whenever a repo is created so all repos have similar starting structure (ex. all repos have "main" branch) **template**
- set all "main" branches to be protected with branch protection rules and pull reviews 
- webhook to listen for certain events (repo creation) and then apply protections on it
- create an issue when a repo is created, notify myself with an @mention and return the API result from a GET on the repo
- will use apis to verify rules and check to make sure all permissions are present
- test by making new repos several different ways

___

### Here's what I need to communicate to the customer to do:

Decide as an organization on the following measures
01. Choose a default branch name and set that at the organizational level
02. Create a default README and if applicable, a default folder structure which will become the organization's template for repositories
03. Review the available branch protection settings and see which ones would be required and which ones would be unnecessary 
04. set up a new service (or utilize an existing one) to listen for webhooks and perform business logic
05. set up a webhook at the organizational level to fire on all repository creations (as per the scope of the project)
06. when a webhook is received, do validation, and depending on the business rules, apply the correct permissions with a corresponding API call
07. after the default branch of the new repository has protections placed on it, create an issue in that same repository outlining the new protections and assign it appropriately 
08. communicate this solution could be also used for webhooks which fire on other events too.

___

### Example Response to the Customer

Hi there, thanks for reaching out to GitHub with this issue.
We can absolutely help with building out some protections for your Organization's repositories in GitHub!

Many customers have found that setting different levels of protections on "branches" of their repositories can help them maintain more granular control of what is being added to their code.
You can kick off workflows or enforce requirements before an individual collaborator can make changes to a branch in your repository, including merging a pull request into the protected branch, by using branch protection rules.
See the following page in the GitHub Docs for more information. 
<https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/defining-the-mergeability-of-pull-requests/about-protected-branches#about-branch-protection-rules>

Whatever rules are agreed upon as an organization can be implemented with these branch protection rules.

According to your email, however, it seems like your organization is quite large! So we would need to come up with a more custom implementation of this, utilizing API calls and webhooks to set these protections programmatically and in an automated fashion.

Using Webhooks and APIs, we can have these rules be applied whenever a repository is created so any further pushes to these repositories will have to go through the workflows or adhere to the requirements set by these protections.

If this sounds like a good plan, let me know and we can begin applying some preliminary default settings which will allow us to configure the rest of the solution.
Hopefully we will be able to get you to a place where your Security Team feels more confident about the kind of protections placed on these repositories!

Let us know what you decide, and we'll go ahead and get started with securing these repos!

Thanks,
Andrew Ferguson

___

### Here is what I did:

01. created an organization 'af-ghi'
02. invited a member to the organization (so have and admin and a regular member)
03. created a public repo for testing, used private default repo for initial testing
04. tested creating branch protections on private repo with API call
05. create an aws ec2 instance / get public ip address and open fw over port 8080
06. create simple node.js app to listen for webhooks on port 8080 - make it a service, test and verify it works
07. create webhook and set payload url to server ip:8080 / json / pass secret whenever there is a repo created
08. check what is being passed (json/content) with the webhook, need repo-id so we can pass that to the API call to see if branch of repo is protected
09. stringified the json from the webhook and then parsed it out to extract some values from it (repo name)
10. pass this to a curl statement, and using a personal access token set the branch protection, then create an issue and mention myself
11. make sure webhook only fires on "repository" events (creation)
12. start using variables and functions to clean up very messy code (curls are too long and could be easily parameterized)
13. add actual logging to nodejs app, not just writing to a file or console.log
14. what happens when a repository is created with different default branch names? need to do more error handling
___

### How to run the webhook.js

01. I used an Ubuntu 22.04.01 LTS server
02. Made sure nodejs was installed
03. Created a directory and cloned the webhook.js to this directory
04. Install npm package(s) needed `npm install express --save`
05. Create and enable this file as a service in systemd so it is always running and restarts on reboot
06. Open port 8080/tcp on Firewall on the server 
07. Get public IP from this server and use it for the payload URL in Webhook setup

___

#### Some misc curl statements I was using...

*Before running these in a shell, first set token variable to the api token* 

`token=<api-token-here>`

##### THESE API CALLS WORK NOW #####

*CURL-1  working but need to tweak this to make it more useful; this is the example from docs.github.com*

`curl -X POST -H "Accept: application/vnd.github+json" -H "Authorization: Bearer $token" https://api.github.com/orgs/af-ghi/hooks -d '{"name":"web","active":true,"events":["push","pull_request"],"config":{"url":"http://example.com/webhook","content_type":"json"}}'`

*CURL-2  working - this will create an issue*

`curl -X POST -H "Accept: application/vnd.github+json" -H "Authorization: Bearer $token" https://api.github.com/repos/af-ghi/demo-repository/issues -d '{"title":"Found a bug","body":"I'\''m having a problem with this.","assignees":["aeferguson"],"labels":["bug"]}'`

*CURL-3  working - will return protection status and details from a branch*

`curl -H "Accept: application/vnd.github+json" -H "Authorization: Bearer $token" https://api.github.com/repos/af-ghi/demo-01/branches/main/protection`

___

##### THESE API CALLS ARE JUST FOR GETTING INFORMATION #####

*list webhooks for an org*

`curl   -H "Accept: application/vnd.github+json"   -H "Authorization: Bearer $token"   https://api.github.com/orgs/af-ghi/hooks`

*lists public org info*

`curl -X GET --url "https://api.github.com/orgs/af-ghi"`

*lists all public repos in an org*

`curl -X GET --url "https://api.github.com/orgs/af-ghi/repos"`

*get all issues for an organization*

`curl -H "Accept: application/vnd.github+json" -H "Authorization: Bearer $token" https://api.github.com/orgs/af-ghi/issues`

*get all repos for an organization* 

`curl -H "Accept: application/vnd.github+json" -H "Authorization: Bearer $token" https://api.github.com/orgs/af-ghi/repos`
