# Jenkins Pipeline and Shared Library Notes

This document covers our Jenkins pipeline setup, shared libraries, build and test steps, deployment workflows, and more.

---

## Table of Contents

- [1. Pipeline Declaration and Libraries](#1-pipeline-declaration-and-libraries)
- [2. Pipeline Structure](#2-pipeline-structure)
  - [Parameters](#parameters)
  - [Agent and Environment](#agent-and-environment)
  - [Triggers](#triggers)
- [3. Stages and Steps](#3-stages-and-steps)
  - [Deploy Stage Example](#deploy-stage-example)
  - [Node.js Build and Test Steps](#nodejs-build-and-test-steps)
- [4. Shared Library: jenkins-shared-library](#4-shared-library-jenkins-shared-library)
- [5. Workflow Overview](#5-workflow-overview)
- [6. Build Tools and Testing](#6-build-tools-and-testing)
- [7. Pipeline Parameters and UI Elements](#7-pipeline-parameters-and-ui-elements)
- [8. Artifact Management and Deployment](#8-artifact-management-and-deployment)
- [9. Additional Notes and Best Practices](#9-additional-notes-and-best-practices)
- [10. Conclusion](#10-conclusion)

---

## 1. Pipeline Declaration and Libraries

We use the following top-of-file declaration to import shared libraries for agents and shared functionality:

```groovy
@Library(["jenkins-agent@stable", "jenkins-shared-library"]) _
```

*Note: This line loads the specified libraries from your shared repositories so that common steps and configurations can be reused.*

---

## 2. Pipeline Structure

### Parameters

Our pipelines define various parameters to control behavior. For example, a boolean parameter to optionally build a Docker image:

```groovy
pipeline {
  parameters {
    booleanParam(
      name: 'buildAlamySolrDocker', 
      defaultValue: false, 
      description: 'Do you want to build alamy-solr-docker image?'
    )
    // Additional parameters...
  }
  ...
}
```

### Agent and Environment

We run our pipeline on a Kubernetes agent (from the jenkins-agent library):

```groovy
agent {
  kubernetes(
    k8sagent(/* configuration details */) // Provided by jenkins-agent@stable
  )
}
```

We also define environment variables for the build:

```groovy
environment {
  AWS_REGION = 'eu-west-1'
  // Additional environment variables...
}
```

### Triggers

We schedule the pipeline to run periodically using cron triggers:

```groovy
triggers {
  cron('H * * * *')
}
```

---

## 3. Stages and Steps

### Deploy Stage Example

In our pipeline, the deploy stage conditionally builds and pushes a Docker image using Kaniko if the parameter is set:

```groovy
stages {
  stage('deploy') {
    steps {
      container('kubectl/kaniko') {
        script {
          def currDir = pwd()
          if (params.buildAlamySolrDocker) {
            buildDockerImage(currDir, env.AWS_REGION, env.AWS_CRD_ID)
          }
        }
      }
    }
  }
}
```

*Note: The use of a container block ensures that the correct tools (like Kaniko) are available for the deployment step.*

### Node.js Build and Test Steps

For our Node.js application, we define a stage that performs linting, building, and testing:

```groovy
steps {
  container('nodejs-test') {
    script {
      githubNotify status: "PENDING", description: 'Test App stage started'
      usePackageCache()
      sh '''
        npm run lint
        npm run build
      '''
      if (env.BRANCH_NAME == 'main') {
        stash allowEmpty: true, includes: '.next/routes-manifest.json', name: 'routes'
      }
      if (params.runTestsCoverage || env.BRANCH_NAME == 'main') {
        sh 'npm run test:coverage'
      } else {
        sh "npm run test"
      }
      tested = true
      stash allowEmpty: true, includes: '**/*.xml', name: 'unitTestStash'
    }
  }
}
```

*Note: This stage uses conditional logic to run different test commands based on branch or parameter values and stashes test reports for later use.*

---

## 4. Shared Library: jenkins-shared-library

Our shared library provides reusable functions. For example, the `buildDockerImage` function:

```groovy
def buildDockerImage(currDir, awsRegion, awsCredsId) {
  dir(currDir) {
    sh "rm -rf ./*"
    echo "Building and pushing ${repoName}"
    withAWS(credentials: awsCredsId, region: awsRegion) {
      echo "Building and pushing"
      sh "/kaniko/executor -f Dockerfile -c . --cache=true --destination=${imageTag}"
      echo "=============================="
    }
  }
}
```

*Note: This function encapsulates Docker image build and push steps using Kaniko and AWS credentials.*

---

## 5. Workflow Overview

Our organization follows a two-week sprint process with the following workflow:

1. **Development Phase:**  
   - Developers create feature branches from the main branch.
   - Code is pushed to the feature branch and the dev pipeline is triggered.
   
2. **Pull Request and Merge:**  
   - After successful testing, a PR is raised and, once approved, merged into the main branch.
   
3. **Staging Pipeline:**  
   - The staging pipeline is automatically or manually triggered.
   - QA tests the application on staging (e.g., via CRM or staging web pages).
   
4. **Production Deployment:**  
   - After QA sign-off, a change request is raised, and the prod pipeline is triggered.

5. **Issue Resolution:**  
   - If defects are detected in production, development starts on fixes from a dedicated branch, followed by a new dev pipeline run and eventual redeployment.

*Note: This workflow ensures thorough testing before code is promoted to production.*

---

## 6. Build Tools and Testing

Our pipelines use Node.js and npm as the primary build tools. The following commands are part of our build and test process:

| Task                            | Command Used           | Possible Tool                          |
|---------------------------------|------------------------|----------------------------------------|
| Linting                         | `npm run lint`         | ESLint                                 |
| Building the App                | `npm run build`        | Webpack, Next.js, or Parcel            |
| Running Unit Tests              | `npm run test`         | Jest, Mocha, or Jasmine                |
| Running Tests with Coverage     | `npm run test:coverage`| Jest, Istanbul (nyc)                   |

*Additional Tools:*  
- **Next.js:** Used for React application bundling.
- **Tailwind CSS:** For styling (using `npx tailwindcss`).
- **Webpack:** For JavaScript optimization (via Next.js).
- **Cross-env:** For setting environment variables.

*Security:*  
- Grype is used to scan Docker images for Node.js vulnerabilities.
- Use `npm audit` or Snyk for dependency security.

---

## 7. Pipeline Parameters and UI Elements

Our Jenkins UI supports various parameter types to customize pipeline runs. For example:

| Parameter Type               | UI Appearance                       | Typical Use Case                                  |
|------------------------------|-------------------------------------|---------------------------------------------------|
| Boolean                      | Checkbox                            | Toggle features on/off                            |
| Choice                       | Dropdown                            | Select environment, branch, etc.                  |
| String                       | Text box                            | Provide version, name, or other strings           |
| Password                     | Masked input                        | Enter sensitive credentials                       |
| File                         | File upload button                  | Upload configuration files                        |
| Active Choices               | Dynamic dropdown/checkbox           | Generate dynamic options based on context         |

*Real-Time Use Cases:*  
- Boolean parameters for feature flags.
- Git parameters to select branches.
- Run parameters to reuse artifacts from previous builds.

---

## 8. Artifact Management and Deployment

### Docker Image Artifacts

- **Storage:**  
  Docker images are stored in AWS ECR.
- **Build Tool:**  
  Kaniko builds Docker images which are then pushed to ECR.
- **Artifact Type:**  
  Our artifacts are container images rather than traditional JARs or ZIPs.

### Test Reports and Kubernetes Manifests

- **Test Reports:**  
  Unit test results (XML files) are stashed and processed using Jenkins' JUnit plugin.
  ```groovy
  unstash 'unitTestStash'
  junit '**/*.xml'
  ```
- **Kubernetes Manifests:**  
  Temporarily stashed in Jenkins for deployment.
  ```groovy
  stash includes: 'k8s/**', name: 'k8s'
  // Later, retrieved using:
  unstash 'k8s'
  ```

*Note: Using stashes allows temporary storage of build artifacts and configuration files within the pipeline.*

---

## 9. Additional Notes and Best Practices

- **Disable Concurrent Builds:**  
  Use `disableConcurrentBuilds()` to ensure that only one build runs at a time.
- **ANSI Color:**  
  Use `ansiColor('xterm')` to enable colored console output.
- **User Input:**  
  Use the `input` step to prompt for manual intervention during pipeline execution.
  ```groovy
  input {
    message "Should we continue?"
    ok "Yes, we should."
  }
  ```
- **Zip Commands:**  
  Use `zip -q -r` for quiet zipping (suppressing unnecessary output).

- **Deployment Workflow:**  
  Ensure that feature branches are merged into main only after passing the dev pipeline and obtaining QA sign-off. Production deployments are then triggered via a change request.

- **Build and Test Tools:**  
  - Node.js is used for building and testing.
  - Linting is performed with ESLint.
  - Unit tests and coverage use frameworks like Jest or Mocha.
  - Next.js and Tailwind CSS are used in the frontend build.

*Tip: "Shift left" by completing as much testing in development as possible to catch issues early.*

---

## 10. Conclusion

This document outlines the key components of our Jenkins pipelines, including the use of shared libraries, conditional build steps, artifact management, and our overall development workflow. By following these guidelines, we ensure a consistent and robust CI/CD process that supports our agile, sprint-based development cycle.
