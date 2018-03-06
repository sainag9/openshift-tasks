node {
    stage('rollback') {
	sh 'oc project xyz-tasks-dev2'
        sh 'oc rollback tasks --to-version=13'
    }
}
