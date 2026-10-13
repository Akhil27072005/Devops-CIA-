Course Registration Service — DevOps Assignment

Question Set: F — Successful-Registration Audit Log
Technologies: Java, Maven, JUnit 5, Git, Jenkins, Docker, Kubernetes, Minikube

1. Project Overview

This project improves the supplied Student-Starter Course Registration application without replacing the original project.

The objectives are to:

Refactor the application into a multi-module Maven project.
Fix the course-capacity boundary defect.
Implement an in-memory audit log for successful registrations.
Add JUnit 5 tests for registration and audit-log behaviour.
Configure a Jenkins Declarative Pipeline for automated build and testing.
Deploy Nginx on Minikube and demonstrate scaling and self-healing.
Maintain a meaningful Git history and document implementation evidence.


2. Project Structure

course-registration/
├── pom.xml
├── README.md
├── Jenkinsfile
├── deployment.yaml
├── service.yaml
├── screenshots/
└── course-registration-app/
    ├── pom.xml
    └── src/
        ├── main/
        │   └── java/
        └── test/
            └── java/


The root POM is the parent and aggregator project. The course-registration-app directory contains the existing application source code and tests.

The original application logic and existing tests are preserved, with targeted changes for the capacity defect and audit-log requirements.

3. Git Workflow
3.1 Create a feature branch

Start from the supplied baseline repository and create a feature branch.

git status
git switch -c feature/registration-audit-log


If the branch already exists, switch to it instead of creating it again.

3.2 Commit changes separately

git add course-registration-app/src/main/java
git commit -m "fix: correct course registration capacity boundary"

git add course-registration-app/src/test
git commit -m "test: cover course capacity boundary cases"

git add course-registration-app/src/main/java course-registration-app/src/test
git commit -m "feat: add successful registration audit log"


Additional commits should cover the Maven refactoring, Jenkins pipeline, Kubernetes manifests, and documentation.

Stage only the files relevant to each commit.

3.3 Verify the branch and history
git branch
git log --oneline --graph --decorate --all
git status

Screenshot: Git branch and commit history

<img width="794" height="138" alt="image" src="https://github.com/user-attachments/assets/bcbc7c07-6ea3-48d5-8c41-c7f0972350f9" />


Show the feature branch and relevant commits.

Screenshot: Clean working tree
<img width="940" height="160" alt="image" src="https://github.com/user-attachments/assets/def326ff-3bc7-4ad9-a882-c6168784628d" />


Show the actual git status output.




4. Maven Multi-Module Configuration
4.1 Parent POM

The root pom.xml defines the parent Maven project.

It includes:

pom packaging.
Shared properties such as the Java and JUnit versions.
The course-registration-app module declaration.
JUnit 5 dependency management.
Shared Maven plugin configuration where appropriate.

The parent POM allows Maven to build the child module through the Maven reactor.

4.2 Child POM

The course-registration-app/pom.xml inherits from the root POM using the correct parent coordinates and relative path.

The child module contains the Java source files and test files moved from the supplied baseline.

JUnit 5 is declared in the child POM without repeating the version managed by the parent. The Maven Compiler Plugin and Maven Surefire Plugin are configured for the selected Java version and JUnit 5 execution.

4.3 Build and test

Run these commands from the repository root:

mvn clean test
mvn package


Additional verification commands:

mvn validate
mvn clean package


The packaged JAR is generated under:

course-registration-app/target/

4.4 Actual build results


<img width="940" height="351" alt="image" src="https://github.com/user-attachments/assets/8171917a-7fb6-4e8f-a87b-fc00a6037654" />


Show the actual test results and Maven reactor summary.

Screenshot: Maven package
<img width="940" height="178" alt="image" src="https://github.com/user-attachments/assets/de998984-66a4-4f3d-b6df-8c03aad20256" />


5. Course-Capacity Boundary Defect
5.1 Problem

The registration logic must reject new registrations when a course is full while allowing a registration when exactly one seat remains.

An incorrect boundary comparison can either reject the final available seat or permit registration beyond the course's capacity.

5.2 Correction

The capacity validation is corrected without removing the existing validation rules.

The corrected behaviour ensures that:

Registration succeeds when capacity remains available.
The final available seat can be filled.
Registration is rejected when the course is full.
A rejected registration does not change the student's registration or credit state.
A rejected registration does not increase the course enrollment count.
5.3 Tests

JUnit 5 tests cover:

A course with exactly one seat remaining.
A course that is already full.
The student's state after a rejected attempt.
The course enrollment count after a rejected attempt.


7. Jenkins Declarative Pipeline
7.1 Purpose

The root-level Jenkinsfile automates the Maven build, test, packaging, report publication, and artifact archiving workflow.

Target environment: [Windows agent using bat / Linux agent using sh]

All Maven commands execute from the repository root.

7.2 Pipeline stages

The pipeline should include:

Checkout the source code.
Run mvn clean test.
Publish JUnit test results.
Run mvn package only when the tests succeed.
Archive the packaged JAR only when packaging succeeds.

Expected paths:

JUnit reports:
course-registration-app/target/surefire-reports/TEST-*.xml

Packaged artifact:
course-registration-app/target/*.jar


The junit pipeline step publishes test results, while archiveArtifacts archives the packaged JAR.

7.3 Failure handling

The pipeline must prevent packaging and artifact archiving after a failed test stage.

JUnit report publication should run even when tests fail, where reports are available, so that failure details remain visible in Jenkins.

7.4 Execution instructions
Configure a Jenkins Pipeline job using the repository.
Select the appropriate branch and agent.
Ensure Git, a compatible JDK, Maven, and the required Jenkins plugins are available.
Execute the pipeline.
Review stage results, published test reports, and archived artifacts.

<img width="940" height="328" alt="image" src="https://github.com/user-attachments/assets/a7261bd0-f749-4565-8ee8-955eb6c5dc5d" />

8. Deploy and Manage Nginx Using Minikube
8.1 Objective

Deploy nginx:1.27 using a Kubernetes Deployment with two replicas and expose it through a NodePort Service.

The Kubernetes deployment is separate from the Java application's Maven build.

8.2 Deployment manifest

The root-level deployment.yaml defines:

Image: nginx:1.27
Initial replicas: 2
Matching Deployment selectors and Pod labels
A container port for HTTP traffic

Apply and verify the Deployment:

minikube status
kubectl apply -f deployment.yaml
kubectl get deployments
kubectl get pods -o wide
kubectl rollout status deployment/nginx-deployment

8.3 NodePort Service

The root-level service.yaml exposes the Deployment through a NodePort Service.

Apply and inspect the Service:

kubectl apply -f service.yaml
kubectl get services
kubectl describe service nginx-service
kubectl get endpoints nginx-service


Retrieve the Minikube Service URL:

minikube service nginx-service --url


Use the actual resource names specified in the YAML files if they differ from the examples.

Verify that the Service selects the intended Pods and that Nginx returns a successful HTTP response.

8.4 Scale from two to three replicas

Run:

kubectl scale deployment/nginx-deployment --replicas=3
kubectl rollout status deployment/nginx-deployment
kubectl get deployments
kubectl get pods -o wide


Verify that the Deployment reaches three available replicas.

8.5 Demonstrate self-healing

List the Pods:

kubectl get pods


Delete one Pod belonging to the Deployment:

kubectl delete pod <pod-name>


Replace <pod-name> with the actual name of a Pod returned by kubectl get pods.

Observe the replacement:

kubectl get pods -w
kubectl get deployments
kubectl get pods


8.7 Kubernetes execution results
<img width="940" height="129" alt="image" src="https://github.com/user-attachments/assets/e343819d-30a2-4c9f-96cf-517baa0cccef" />

<img width="940" height="149" alt="image" src="https://github.com/user-attachments/assets/66552443-7096-4b20-ae98-453aba9b9eb5" />

<img width="940" height="177" alt="image" src="https://github.com/user-attachments/assets/81b3d3de-12be-4e1c-95ae-ece19dc708d3" />

<img width="940" height="171" alt="image" src="https://github.com/user-attachments/assets/a0637d2b-544d-4c28-90ff-7ebd2e652487" />

![Uploading image.png…]()

9. Setup and Execution Commands
Prerequisites

Install and configure:

JDK compatible with the project
Apache Maven
Git
Jenkins, if running the pipeline
Docker or another supported container runtime
Minikube
kubectl
Build and test

Execute from the repository root:

mvn clean test
mvn package

Git verification
git branch
git log --oneline --graph --decorate --all
git status

Deploy to Minikube
minikube start
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
kubectl get deployments,pods,services
minikube service nginx-service --url

Scale and test self-healing
kubectl scale deployment/nginx-deployment --replicas=3
kubectl get pods
kubectl delete pod <pod-name>
kubectl get pods -w


10. Change Summary

The work covers the following areas:

Git: Feature branch, separate commits, merge into main, and working-tree verification.
Maven: Parent POM, child module, dependency management, compiler and Surefire configuration.
Defect fix: Correct capacity boundary validation.
JUnit 5: Tests for capacity, rejected registrations, and audit-log behaviour.
Audit log: In-memory entries for successful registrations only.
Jenkins: Automated build, test-report publication, packaging, and conditional artifact archiving.
Kubernetes: Nginx Deployment, NodePort Service, scaling, and Pod self-healing.
Documentation: Setup instructions, execution evidence, and technical explanations.
Modified files

Update this list to match the actual repository changes.

File	Purpose
pom.xml	Parent and aggregator POM
course-registration-app/pom.xml	Child module configuration
course-registration-app/src/main/java/	Application and implementation changes
course-registration-app/src/test/java/	Existing and new JUnit tests
Jenkinsfile	CI pipeline
deployment.yaml	Nginx Deployment
service.yaml	NodePort Service
README.md	Project documentation and evidence placeholders
11. Limitations and Final Verification
