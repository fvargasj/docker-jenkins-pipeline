node {
  checkout scm
  env.PATH = "${tool 'Maven3'}/bin:${env.PATH}"
  stage('Package') {
    dir('webapp') {
      sh 'mvn clean package -DskipTests'
    }
  }

  def customImage
  def couchbase

  stage('Create Docker Image') {
    dir('webapp') {
      customImage = docker.build("arungupta/docker-jenkins-pipeline:${env.BUILD_NUMBER}")
    }
  }


  try {
  	  stage ('Start Couchbase Application') {
  	  	  // Start database container here
  	  	  couchbase = docker.run('--name db -p 8091-8093:8091-8093 -p 11210:11210 arungupta/oreilly-couchbase:latest')
  	  }
	  stage ('Run Application') {
	      // Run application using Docker image
	      sh "DB=`docker inspect --format='{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' db`"
	      sh "docker run -e DB_URI=$DB arungupta/docker-jenkins-pipeline:${env.BUILD_NUMBER}"
	  }
	  stage ('Run Tests') {
	  	  dir('webapp') {
		  	sh "mvn test"
			customImage.push()
		  }
	  }

  } catch (error) {
  } finally {
    // Stop and remove database container here
    couchbase.stop()
    junit '**/target/surefire-reports/*.xml'
  }
}
