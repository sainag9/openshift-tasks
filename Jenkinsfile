// Run this node on a Maven Slave edit
	// Maven Slaves have JDK and Maven already installed
	node {
	  notifySuccessful()
		//emailext body: 'A Test EMail', recipientProviders: [[$class: 'DevelopersRecipientProvider'], [$class: 'RequesterRecipientProvider']], subject: 'Test', to: ''`
	}
	
def notifySuccessful() {

 
   emailext (
       subject: "SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
       body: """<p>SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
         <p>Check console output at "<a href="${env.BUILD_URL}">${env.JOB_NAME} [${env.BUILD_NUMBER}]</a>"</p>""",
       recipientProviders: [[$class: 'RequesterRecipientProvider']],to:'k.sainagarjuna11@gmail.com'
     )
 }
