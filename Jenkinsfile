node('maven') {
  // define commands
  def mvnCmd = "mvn"
  // injection of environment variables is not done so set them here...
  def sourceRef = "master"
  def sourceUrl = "https://github.com/sainag9/openshift-tasks"
  def devProject = "ocp-tasks-4"
  def applicationName = "jkf-tasks"

  stage 'build'
    git branch: sourceRef, url: sourceUrl
    sh "${mvnCmd} clean install -DskipTests=true"
  stage 'test'
    sh "${mvnCmd} test"
  stage 'deployInDev'
    sh "rm -rf oc-build && mkdir -p oc-build/deployments"
    sh "cp target/openshift-tasks.war oc-build/deployments/ROOT.war"
    // clean up. keep the image stream
    sh "oc project ${devProject}"
    // build image
    sh "oc start-build ${applicationName} --from-dir=oc-build --wait=true -n ${devProject}"
    // deploy image
    sh "oc new-app ${applicationName}:latest -n ${devProject}"
    sh "oc expose svc/${applicationName} -n ${devProject}"
}
