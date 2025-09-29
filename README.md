                                                                    ***GitHub Hosted Runner***

A GitHub hosted runner is a virtual machine (VM) provided and maintained by GitHub to run your workflows in GitHub Actions. You don’t have to set up or manage the machine yourself—GitHub automatically provisions and tears it down for each workflow run.
When a workflow is triggered by an event such as a push, pull request, or schedule, GitHub automatically assigns an available hosted runner to execute the job. The runner, which comes pre-installed with commonly used tools like Node.js, Python, and Docker, checks out the repository, runs the CI/CD pipeline steps defined in. github/workflows/, and upon completion, provides logs and artifacts before being released
________________________________________
Today I set up a CI/CD pipeline using GitHub-hosted runners to deploy a static website (blogcard) automatically to an AWS EC2 instance whenever changes are pushed to the main branch.
Project Directory Structure:
blogcard/
├── index.html         # Main HTML page
├── style.css          # CSS file for styling
├── images/            # Folder containing image assets
└── .github/
    └── workflows/
        └── deploy.yml  # GitHub Actions workflow file
Workflow File: .github/workflows/deploy.yml
name: Deploy Static Website

on:
  push:
    branches:
      - main

env:
  REMOTE_HOST: ${{ secrets.REMOTE_HOST }}
  REMOTE_USER: ${{ secrets.REMOTE_USER }}
  REMOTE_PATH: /var/www/blogcard

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Copy files to EC2
        uses: appleboy/scp-action@v0.1.7
        with:
          host: ${{ env.REMOTE_HOST }}
          username: ${{ env.REMOTE_USER }}
          key: ${{ secrets.AWS_SSH_KEY }}
          source: "./*"
          target: ${{ env.REMOTE_PATH }}

      - name: Reload Nginx
        uses: appleboy/ssh-action@v1.0.0
        with:
          host: ${{ env.REMOTE_HOST }}
          username: ${{ env.REMOTE_USER }}
          key: ${{ secrets.AWS_SSH_KEY }}
          script: |
            sudo systemctl reload nginx
________________________________________
Steps Taken:
1.	Prepared AWS EC2 Instance:
o	Installed Nginx.
o	Created deployment folder /var/www/blogcard.
o	Configured Nginx to serve from /var/www/blogcard.
2.	Checked/Generated SSH Key:
o	Verified id_ed25519 exists locally.
o	Copied public key to EC2 ~/.ssh/authorized_keys.
o	Added private key content to GitHub Secrets as AWS_SSH_KEY.
3.	Configured GitHub Actions:
o	Created workflow file deploy.yml under .github/workflows.
o	Set environment variables: REMOTE_HOST, REMOTE_USER, REMOTE_PATH.
o	Configured steps to:
	Checkout repository.
	Copy files to EC2 using SCP.
	Reload Nginx via SSH.
4.	Tested Deployment:
o	Pushed code to main branch.
o	Verified GitHub Actions ran successfully.
o	Accessed the EC2 public IP to confirm the blogcard website was displayed.
________________________________________
GitHub Secrets Used:
•	AWS_SSH_KEY → Private key content
•	REMOTE_HOST → EC2 public IP
•	REMOTE_USER → EC2 username (usually ubuntu)
________________________________________
Outcome:
•	CI/CD pipeline successfully deploys the static site automatically.
•	Changes pushed to main branch reflect instantly on the EC2 server.
•	Nginx serves the blogcard site from /var/www/blogcard instead of the default welcome page.
Benefits of GitHub Hosted Runners
They save setup time since they come pre-installed with common tools and dependencies, automatically scale to handle workloads without requiring server management, are maintained and updated by GitHub to ensure security and reliability, and allow teams to focus on development instead of maintaining infrastructure.
