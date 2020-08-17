# cpWaapJuice
This Helm Chart deploys the OWASP Juice-shop app with an nginx ingress controller that is enhanced with Check Point's WAAP technology to provide Web Application and API Protection.

Prerequisites:
Creation of a portal.checkpoint.com account with access to "Infinity Next"
The token for using the Check Point Nano agent
Access to the private docker hub repo for mnicholssync
Helm installed

Instructions for deploying after repackage...
Download the Helm Chart
Extract the tgz file 
Modify the values.yaml file to use your WAAP token (generated in the Check Point Infinity Next Portal) and repackage
Repackage the helm chart
  helm package
Deploy the Helm chart
  helm install {your app name} helm juice-0.1.1.tgz
