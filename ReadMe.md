Nexus on OpenShift
==================

Nexus Setup
-----------

In this example, we use a Docker build to take SonaType's Nexus image and add a few Repositories to it, like Maven, Red Hat, and Springboot. To see how these repositories are configured look at *nexus/conf/nexus.xml*

To build the image, log into OpenShift and create a new project:

```
oc login <OCP_URL>
oc new-project nexus
```
Next create the necessary OpenShift objects, they are all in list form in the file */nexus/nexus-list.yaml*

```
oc create -f nexus/nexus-list.yaml
```

This should start the build and spin up the pod automatically. You can access the nexus instance at the link above in the overview. Click link and login with *admin:admin123*

Springboot App Hooked-Up
------------------------

In this section we will deploy a sample application to OpenShift and have it pull its dependency from our nexus during builds on OpenShift.

First create a new OpenShift project:

`oc new-project springboot-sample`

Now you can use one of my [sample springboot applications](https://github.com/domenicbove/springboot-sample-app) sample springboot applications, which I forked from elsewhere on the web. And using fabric8's java source to image builder create create the app within OCP:

`oc new-app fabric8/s2i-java~https://github.com/domenicbove/springboot-sample-app`

To connect the build up with the newly configured nexus, jump into the OpenShift console, click on Builds -> springboot-sample-app -> Actions -> Edit YAML. Now add MAVEN_MIRROR_URL env to the BuildConfig as below:

```
strategy:
    sourceStrategy:
      env:
      - name: MAVEN_MIRROR_URL
        value: http://nexus-cicd.rhel-cdk.10.1.2.2.xip.io/content/groups/public/
```

Above url needs to match the public repo in nexus

This s2i builder image does a good job in building the image and creating the deployment. It does not set up the 8080 port and route for your application. To do this click Services -> springboot-sample-app -> Edit YAML and add in the 8080 port like below:

```
- name: 8080-tcp
  protocol: TCP
  port: 8080
  targetPort: 8080
```
Now click the link *Create Route*

For extra credit here is how to add a Git Secret to your build config. This would be useful if your source code was hosted on a private repository. This process is laid out [here](https://docs.openshift.com/enterprise/3.2/dev_guide/builds.html#using-private-repositories-for-builds).

First create the secret in the same project as your BuildConfig:

`oc secrets new-basicauth basicsecret --username=USERNAME --password=PASSWORD`

Next link the secret to the builder service account:

`oc secrets link builder basicsecret`

Finally add that secret to the build config with:

`oc set build-secret --source bc/springboot-sample-app basicsecret`
