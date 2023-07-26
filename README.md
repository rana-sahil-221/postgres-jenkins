# postgres-jenkins
Readme of Main
JENKINS PIPELINE 


==> Initialises Postgres databases inside the statefulset pod.

==> List the Branches available on github during runtime and ask for user choice to build on.

==> Deploy the pods, service and configmap to the kubernetes cluster.

==> Fetches the list of available databases and ask user for input to select one of the database.

==> Stores the database as an artifact in jenkins workspace.

==> Integrate SonarQube with Jenkins pipeline for Code Analysis.

==> Paramterized builds to dynamically select branches to build. 

==> Supports Slack Notifications such as Sending Build details with color code for success and failure.
