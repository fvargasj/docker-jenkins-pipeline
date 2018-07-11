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
  	  	  couchbase = docker.image('arungupta/oreilly-couchbase:latest').run('--name db -p 8091-8093:8091-8093 -p 11210:11210')
  	  }
	  stage ('Run Application') {
	      // Run application using Docker image
	      def dbAddress = sh (script:"DB=`docker inspect --format='{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' db` | echo $DB",
    		returnStdout: true,
  		  )
  		  echo dbAddress
	      sh "DB=`docker inspect --format='{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' db`"
	      //customImage.run('-e DB_URI=172.17.0.2')
	      sh "docker run -e DB_URI=$DB arungupta/docker-jenkins-pipeline:${env.BUILD_NUMBER}"
	  }
	  stage ('Run Tests') {
	  	  dir('webapp') {
		  	sh "mvn test"
			customImage.push()
		  }
	  }

  } catch (error) {
  	echo 'Err: Incremental Build failed with Error: ' + error.toString()
  } finally {
    // Stop and remove database container here
    couchbase.stop()
    junit '**/target/surefire-reports/*.xml'
  }
}
