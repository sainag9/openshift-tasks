// Run this node on a Maven Slave edit
	// Maven Slaves have JDK and Maven already installed
	node('maven') {
	  // Make sure your nexus_openshift_settings.xml
	  // Is pointing to your nexus instance
	  def mvnCmd = "mvn"
	
	  stage('Checkout Source') {
	    // Get Source Code from SCM (Git) as configured in the Jenkins Project
	    // Next line for inline script, "checkout scm" for Jenkinsfile from Gogs
	    //git 'http://gogs.xyz-gogs.svc.cluster.local:3000/CICDLabs/openshift-tasks.git'
	    checkout scm
	  }
			
  
	  // The following variables need to be defined at the top level and not inside
	  // the scope of a stage - otherwise they would not be accessible from other stages.
	  // Extract version and other properties from the pom.xml
	  def groupId    = getGroupIdFromPom("pom.xml")
	  def artifactId = getArtifactIdFromPom("pom.xml")
	  def version    = getVersionFromPom("pom.xml")
	
	  stage('Build war') {
	    echo "Building version ${version}"
	
	    sh "${mvnCmd} clean package -DskipTests"
	  }
stage('Unit Tests') {
	    echo "Unit Tests"
	    sh "${mvnCmd} test"
	  }
node {
  stage('JIRA') {
    // Look at IssueInput class for more information.
  jiraComment body: 'test case executed successfully', issueKey: '10000'
 
  }
}
stage('Code Analysis') {
            echo "Code Analysis"

          // Replace xyz-sonarqube with the name of your project
           sh "${mvnCmd} org.sonarsource.scanner.maven:sonar-maven-plugin:3.4.0.905:sonar -Dsonar.host.url=http://sonarqube-xyz-jenkins.apps.rhosp.com/ -Dsonar.projectName=${JOB_BASE_NAME}"
		   }
stage('Build OpenShift Image') {
	    def newTag = "TestingCandidate-${version}"
	    echo "New Tag: ${newTag}"
	
	    // Copy the war file we just built and rename to ROOT.war
	    sh "cp ./target/openshift-tasks.war ./ROOT.war"
	
	    // Start Binary Build in OpenShift using the file we just published
	    // Replace xyz-tasks-dev1 with the name of your dev project
	    sh "oc project xyz-tasks-dev1"
	    sh "oc start-build tasks --follow --from-file=./ROOT.war -n xyz-tasks-dev1"
	
	    openshiftTag alias: 'false', destStream: 'tasks', destTag: newTag, destinationNamespace: 'xyz-tasks-dev1', namespace: 'xyz-tasks-dev1', srcStream: 'tasks', srcTag: 'latest', verbose: 'false'
	  }
stage('Deploy to Dev') {
	    // Patch the DeploymentConfig so that it points to the latest TestingCandidate-${version} Image.
	    // Replace xyz-tasks-dev1 with the name of your dev project
	    sh "oc project xyz-tasks-dev1"
	    sh "oc patch dc tasks --patch '{\"spec\": { \"triggers\": [ { \"type\": \"ImageChange\", \"imageChangeParams\": { \"containerNames\": [ \"tasks\" ], \"from\": { \"kind\": \"ImageStreamTag\", \"namespace\": \"xyz-tasks-dev1\", \"name\": \"tasks:TestingCandidate-$version\"}}}]}}' -n xyz-tasks-dev1"
	
	    openshiftDeploy depCfg: 'tasks', namespace: 'xyz-tasks-dev1', verbose: 'false', waitTime: '', waitUnit: 'sec'
	    openshiftVerifyDeployment depCfg: 'tasks', namespace: 'xyz-tasks-dev1', replicaCount: '1', verbose: 'false', verifyReplicaCount: 'false', waitTime: '', waitUnit: 'sec'
	    //openshiftVerifyService namespace: 'xyz-tasks-dev1', svcName: 'tasks', verbose: 'false'
	  }
stage('Integration Test') {
	    // TBD: Proper test
	    // Could use the OpenShift-Tasks REST APIs to make sure it is working as expected.
	
	    def newTag = "ProdReady-${version}"
	    echo "New Tag: ${newTag}"
	
	    // Replace xyz-tasks-dev1 with the name of your dev project
	    openshiftTag alias: 'false', destStream: 'tasks', destTag: newTag, destinationNamespace: 'xyz-tasks-dev1', namespace: 'xyz-tasks-dev1', srcStream: 'tasks', srcTag: 'latest', verbose: 'false'
	  }
// Blue/Green Deployment into Production
	  // -------------------------------------
	  def dest   = "tasks-green"
	  def active = ""
	
	  stage('Prep Production Deployment') {
	    // Replace xyz-tasks-dev1 and xyz-tasks-prod1 with
	    // your project names
	    sh "oc project xyz-tasks-prod1"
	    sh "oc get route tasks -n xyz-tasks-prod1 -o jsonpath='{ .spec.to.name }' > activesvc.txt"
	    active = readFile('activesvc.txt').trim()
	    if (active == "tasks-green") {
	      dest = "tasks-blue"
	    }
	    echo "Active svc: " + active
	    echo "Dest svc:   " + dest
	  }
	  stage('Deploy new Version') {
	    echo "Deploying to ${dest}"
	
	    // Patch the DeploymentConfig so that it points to
	    // the latest ProdReady-${version} Image.
	    // Replace xyz-tasks-dev1 and xyz-tasks-prod1 with
	    // your project names.
	    sh "oc patch dc ${dest} --patch '{\"spec\": { \"triggers\": [ { \"type\": \"ImageChange\", \"imageChangeParams\": { \"containerNames\": [ \"$dest\" ], \"from\": { \"kind\": \"ImageStreamTag\", \"namespace\": \"xyz-tasks-dev1\", \"name\": \"tasks:ProdReady-$version\"}}}]}}' -n xyz-tasks-prod1"
	
	    openshiftDeploy depCfg: dest, namespace: 'xyz-tasks-prod1', verbose: 'false', waitTime: '', waitUnit: 'sec'
	    openshiftVerifyDeployment depCfg: dest, namespace: 'xyz-tasks-prod1', replicaCount: '1', verbose: 'false', verifyReplicaCount: 'true', waitTime: '', waitUnit: 'sec'
	    //openshiftVerifyService namespace: 'xyz-tasks-prod1', svcName: dest, verbose: 'false'
	  }
	  stage('Switch over to new Version') {
	    input "Switch Production?"
	
	    // Replace xyz-tasks-prod1 with the name of your
	    // production project
	    sh 'oc patch route tasks -n xyz-tasks-prod1 -p \'{"spec":{"to":{"name":"' + dest + '"}}}\''
	    sh 'oc get route tasks -n xyz-tasks-prod1 > oc_out.txt'
	    oc_out = readFile('oc_out.txt')
	    echo "Current route configuration: " + oc_out
	  }}
	
	// Convenience Functions to read variables from the pom.xml
	def getVersionFromPom(pom) {
	  def matcher = readFile(pom) =~ '<version>(.+)</version>'
	  matcher ? matcher[0][1] : null
	}
	def getGroupIdFromPom(pom) {
	  def matcher = readFile(pom) =~ '<groupId>(.+)</groupId>'
	  matcher ? matcher[0][1] : null
	}
	def getArtifactIdFromPom(pom) {
	  def matcher = readFile(pom) =~ '<artifactId>(.+)</artifactId>'
	  matcher ? matcher[0][1] : null
}
def notifySuccessful() {
  emailext (
      subject: "SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
      body: """<p>SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
        <p>Check console output at "<a href="${env.BUILD_URL}">${env.JOB_NAME} [${env.BUILD_NUMBER}]</a>"</p>""",
      recipientProviders: [[$class: 'RequesterRecipientProvider']],to:'k.sainagarjuna11@gmail.com'
    )
}
