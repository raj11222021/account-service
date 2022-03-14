podTemplate(cloud: 'kubernetes',label: 'builder',
            containers: [
                    containerTemplate(name: 'jnlp', image: 'ninech/jnlp-slave-with-docker', privileged: true, args: '${computer.jnlpmac} ${computer.name}')
                    
            ],
            volumes: [
                    hostPathVolume(hostPath: '/var/run/docker.sock', mountPath: '/var/run/docker.sock')
					
            ]) 
{
node{
 def MAVEN_HOME = tool "mymaven"
 env.PATH = "${env.PATH}:${MAVEN_HOME}/bin"
  
  stage('checkout'){  
       checkout([$class: 'GitSCM', branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/raj11222021/account-service.git']]])
  }
  
  stage('compile'){
      sh 'mvn clean compile'
  }
  
  
  stage('unit testing'){
      sh 'mvn test'
  }
  
  
  
    stage('Code quality analysis') 
	{
		withSonarQubeEnv('mysonar') 
		{
                 sh 'mvn sonar:sonar -Dsonar.organization=myorg11222021'
		
    		}
	 }

  	stage("Quality Gate"){
          timeout(time: 1, unit: 'HOURS') {
              def qg = waitForQualityGate()
              if (qg.status != 'OK') {
                  error "Pipeline aborted due to quality gate failure: ${qg.status}"
              }
          }
      }
}

stage('build')
	{
	node('builder'){
				
		container('jnlp') {
			unstash 'dfile'
			unstash 'efile'
			sh 'docker build -t snehalj/assetvalidator:${TIME} .'			
			withCredentials([usernamePassword(credentialsId: 'snehaldockerhub', passwordVariable: 'PWDD', usernameVariable: 'USER')]) {
           			 sh 'docker login -u=$USER -p=$PWDD'
			}
			sh 'docker push snehalj/assetvalidator:${TIME}'
		 }
	}
}
}
