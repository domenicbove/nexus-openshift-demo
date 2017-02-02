1. Create Nexus

`oc new-project nexus`

** Run from within workspace/openshift/cicdi

`oc process -f nexus-template.yaml | oc create -f -`

** Click link, login with admin:admin123, click repositories

** Add spring-release repo to nexus

Repository ID: spring-milestone
Repository Name: spring-milestone
Repository Type: proxy
Repository Policy: Release
Repository Format: maven2
Contained in groups:

Remote URL: https://repo.spring.io/libs-release/

** Take note of public repo in nexus

2. Create App Environment

`oc new-project fis-springboot`

3. Create App

`oc new-app fabric8/s2i-java~https://github.com/codecentric/springboot-sample-app.git`

4. Add MAVEN_MIRROR_URL env to BuildConfig

strategy:
    sourceStrategy:
      env:
      - name: MAVEN_MIRROR_URL
        value: http://nexus-cicd.rhel-cdk.10.1.2.2.xip.io/content/groups/public/

** Above url needs to match the public repo in nexus

// If using a private repo
//oc new-app fabric8/s2i-java~https://gitlab.consulting.redhat.com/dbove/springboot-sample-app.git

5. Add basic secret into build config (If using a private git repo)
** Will be using applicationId/password
** https://docs.openshift.com/enterprise/3.2/dev_guide/builds.html#using-private-repositories-for-builds

`oc secrets new-basicauth basicsecret --username=USERNAME --password=PASSWORD`

`oc secrets link builder basicsecret`

`oc set build-secret --source bc/sample-build basicsecret`

`oc edit bc/springboot-sample-app`

*add in
source:
    git:
      uri: "https://github.com/user/app.git"
    sourceSecret:
      name: "basicsecret"
    type: "Git"

`oc start-build springboot-sample-app`

6. Add to 8080 port to Service
    -
      name: 8080-tcp
      protocol: TCP
      port: 8080
      targetPort: 8080

7. Create Route

BOOM!
