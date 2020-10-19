
# Helm Chart for Check Point WAAP

## Introduction
Helm charts provide the ability to deploy a collection of kubernetes services and containers with a single command. This helm chart deploys an ingress controller integrated with the Check Point container images that include Check Point WAAP nano agent. If you want to integrate the Check Point WAAP nano agent with an ingress controller other than nginx, follow the instructions in the WAAP installation guide. Another option would be to download the helm chart and modify the parameters to match your Kubernetes/Application environment.
## Prerequisites
*   Properly configured access to a K8s cluster (helm and kubectl working)
*   Helm 3.x installed
*   Access to a repository that contains the Check Point nginx controller and cp-alpine images
*   An account in portal.checkpoint.com with access to Infinity Next

## Description of Helm Chart components
*   _Chart.yaml_ \- the basic definition of the helm chart being created. Includes helm chart type, version number, and application version number 
*   _values.yaml_ \- the application values (variables) to be applied when installing the helm chart. In this case, the CP WAAP nano agent token ID, the image repository locations, the type of ingress service being used and the ports, and specific application specifications can be defined in this file. These values can be manually overridden when launching the helm chart from the command line as shown in the example below.
*   _templates/ingress-configmap.yaml_ \- configuration information for nginx.
*   _templates/ingress-crd.yaml_ \- CustomResourceDefinitions for the ingress controller.
*   _templates/ingress-cr.yaml_ \- specifications of the ClusterRole and ClusterRoleBinding role-based access control (rbac) components for the ingress controller.
*   _ingress-deploy-nano.yaml_ \- container specifications that pull the nginx image that contains the references to the CP Nano Agent and the CP Alpine image that includes the Nano Agent itself.
*   _templates/ingress.yaml_ \- specification for the ingress settings for the application.
*   _templates/ingress-service.yaml_ \- specifications for the ingress controller, e.g. LoadBalancer listening on port 80, forwarding to nodePort 30080 of the application 
## Installing the Chart 

Define your application in the “Infinity Next” application of the Check Point Infinity Portal according to the CloudGuard WAAP Deployment Guide section on WAAP Management .

Once the application has been configured in the Infinity Next Portal, retrieve the value for the nanoToken.

Next, install the chart with the chosen release name (e.g. `my-release`), run:

```bash
$ helm repo add checkpoint https://raw.githubusercontent.com/CheckPointSW/charts/master/repository/
$ helm install my-release checkpoint/cpWaapJuice --set nanoToken="{your Infinity Next token string here}" 
```
These are additional optional flags:
```bash
--set image.nginxCtlCpRepo="{repo location of the nginx image}"  
--set image.cpRepo="{repo location of the Check Point Alpine image}" 
--set namespace="{your namespace}" 
--set appURL="{DNS hostname of your application}"
```
## Uninstalling the Chart
To uninstall/delete the `my-release` deployment:
```bash
$ helm delete my-release
```
This command removes all the Kubernetes components associated with the chart and deletes the release.

## Configuration

Refer to [values.yaml](values.yaml) for the full run-down on defaults. These are the Kubernetes directives that map to environment variables for the deployment.

Specify each parameter using the `--set key=value[,key=value]` argument to `helm install`. For example,

```bash
$ helm install my-release --namespace=myapp checkpoint/cpWaapJuice --set nanoToken="3574..." --set appURL="juice-shop.checkpoint.com"

```

Alternatively, a YAML file that specifies the values for the parameters can be provided while installing the chart. For example,

```bash
$ helm install my-release -f values.yaml checkpoint/cpWaapJuice
```
> **Tip**: You can use the default [values.yaml](values.yaml)

The following table lists the configurable parameters of this chart and their default values.

| Parameter                                                  | Description                                                     | Default                                          |
| ---------------------------------------------------------- | --------------------------------------------------------------- | ------------------------------------------------ |
| `image.nginxCtlCpRepo`                                             | Dockerhub location of the nginx image integrated with Check Point Waap                     | `mnicholssync/nginx-ingress-ctl-cp:demo`                                              |
| `image.cpRepo`                                              | Dockerhub location of the Check Point Alpine image integrated with the NanoAgent              | `mnicholssync/nginx-ingress-ctl-cp:cp-agent-alpine`                                           |
| `appURL`                                           | URL of the application (must resolve to Cluster IP address after deployment              | `juice-shop.checkpoint.com`                                          |
| `nameoverride`                                           | Your application name               | `juice-shop`                         |

### Generating WaaP results and Testing the Application

Use kubectl to get the IP address of the ClusterIP
```
kubectl get all
```

Ensure your DNS or host file maps the application URL to the IP address of the ClusterIP. 

Open a Firefox browser tab (Chrome redirects to https, which will not work for the juice-shop example) and go to _**http://{yourapplicationURL}**_

From the application, Click on “Account” and select “Login”. In the username field, enter 
```
select * from ‘1’=’1’ --
```
Enter any password and click “Log In”.  For the normal juice-shop app without Waap, the results show a compromised application. In this case, you will see the login blocked by the Check Point Waap.

Review the log files in the Infinity Portal to see the WaaP Results

## Additional Option: Adding CP WAAP on top of an existing k8s application (replacing the existing ingress controller)
Download the juice-chart helm file from the repository (an example is )
```
tar zxvf juice-chart-0.1.4.tgz
mv juice-chart {your-chart-name}
```

1.  Copy the files from the sample application into your directory structure
2.  Edit the Chart.yaml file to update the version number
3.  (Optional) Edit the values.yaml file to set your default values and the repository paths
4.  Remove the juice-shop.yaml file.
5.  Depending on your environment, edit the yaml files in the template directory to adjust the port numbers, application name, etc to match your environment.

For example, if you already have an nginx ingress deployment defined,

   *   in _**template/ingress-deploy-nano.yaml**_ file, modify the **metadata name**, and **selector:matchLabels** to match your application
    *   in _**template/ingress-service.yaml**_ file, modify the nodePort specification to match your environment (and possibly the selector:app)
    *   you may possibly reuse your _**template/ingress.yaml**_ file, but confirm the **host, serviceName**, and **servicePort** to match your existing application (replace referenced to "juice")

Once you have confirmed the configuration of  your yaml files, rebuild the helm chart. In the parent directory of the juice-chart directory:
```
helm package {your-chart-name}
```
* * *
