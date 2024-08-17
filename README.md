# Situation

You already have an Node project that is configured with Bitbucket CI/CD pipielines but want to introduce security scanning to stop your builds if it detects a security vulnerability.

# Target

Use Snyk, a security vulnerability scanning tool.

# Bitbucket Setup Instructions

## Create Snyk account and with Bitbucket repo

1. Create an account with [Snyk.io](http://Snyk.io)
2. Follow the the integration on the Snyk website to connect your Bitbucket repository as a project

## Setup Bitbucket Repo Variables

### Obtain Snyk User Auth Token

1. Log into your Snyk account
2. On the bottom left click your user name
3. Click **Account Settings**
4. Click **General**
5. Under **Auth Token** you should see an auth token item
6. Click **click to show** to reveal and **copy the auth token**

### Create Bitbucket SNYK_TOKEN Variable

1. Log into your Bitbucket account
2. Click on your repository
3. On the bottom left, click **Repository Settings**
4. On the left panel look for the section called **PIPELINES**
5. Under PIPELINES section, click **Repository variables**
6. Locate the empty text fields labeled **Name** and **Value**
7. In the Name text field, enter **SNYK_TOKEN**
8. In the Value text field, **paste in your user Auth Token** you copied from your Snyk account
9. Ensure the **Secured** checkbox is **checked**
10. Click **Add**

### Obtain your Snyk Organization ID

1. Log into your Snyk account
2. On the left panel under ORGANIZATION, click **Settings**
3. Under Organization ID section, **copy** your organization ID

### Create Bitbucket SNYK_ORG_ID Variable

1. Log into your Bitbucket account
2. Click on your repository
3. On the bottom left, click **Repository Settings**
4. On the left panel look for the section called **PIPELINES**
5. Under PIPELINES section, click **Repository variables**
6. Locate the empty text fields labeled **Name** and **Value**
7. In the Name text field, enter **SNYK_ORG_ID**
8. In the Value text field, **paste in your user Organization ID** you copied from your Snyk account
9. Ensure the **Secured** checkbox is **checked**
10. Click **Add**

### Create Bitbucket SNYK_PROJECT_NAME Variable

1. Log into your Bitbucket account
2. Click on your repository
3. On the bottom left, click **Repository Settings**
4. On the left panel look for the section called **PIPELINES**
5. Under PIPELINES section, click **Repository variables**
6. Locate the empty text fields labeled **Name** and **Value**
7. In the Name text field, enter **SNYK_PROJECT_NAME**
8. In the Value text field, type your desired **project name** (I used the same name as my repo)
9. Ensure the **Secured** checkbox is **checked**
10. Click **Add**

## Implement Snyk Pipeline Step

### Create Snyk Scan Step Definition

1. Open your repositories Bitbucket Pipeline YML file
2. Under `definitions:` and `steps:` add the following code
    
    ```yaml
      - step: &snyk_security_scan
          name: Snyk Security Scan # Vulnerability Scanning
          caches:
            - node
          script:
            - npm install -g snyk
            - snyk auth $SNYK_TOKEN
            - snyk monitor --org=$SNYK_ORG_ID --project-name=$SNYK_PROJECT_NAME --severity-threshold=low # Sends report to Snyk WebUI
            - snyk test --org=$SNYK_ORG_ID --project-name=$SNYK_PROJECT_NAME --severity-threshold=low # Stops build if fails
    ```
    
    After adding this, this step is defined and you can now call it anywhere you’d like within your Pipeline section.
    

The `snyk mointor` will scan and send the report to your Snyk account for you to inspect in your the results but won’t stop your pipeline if theres vulnerabilities.

The `snyk test` command WILL stop your build if it detects vulnerabilities.

### Call the Snyk Scan Step in Pipelines

Add the following step anywhere under your `pipelines:` section that you would like to run snyk security scans 

```yaml
- step: *snyk_security_scan
```

# Conclusion

By implementing the above Snyk and Bitbucket configurations, you will be able to have your CI/CD pipeline run vulnerability checks against your code.  

Depending on how you configure Bitbucket and Synk notifications, you can receive a failed build alert from Bitbucket and an alert from Synk if it detected a vulnerability in your code.
