node {
    environment {
        docker_creds = credentials("dockerhub")
        username = "${env.dockerhub_USR}"
        password = "${env.dockerhub_PSW}"
    }
    def app

    stage('Clone repository') {
        /* Let's make sure we have the repository cloned to our workspace */
        checkout scm
    }

    stage('Test') {
        /* Ideally, we would run a test framework against our image.
         * For this example, we're using a Volkswagen-type approach ;-) */
        docker.image('redis:5.0.3-alpine').withRun('--name redis') { container ->
            docker.image('dsanderscan/jenkins-py3-0.1').inside('--link redis:redis') {
                withEnv(["HOME=${env.WORKSPACE}"]) {
                    checkout scm
                    sh """
                        pwd
                        ls -als
                        python3 -m venv env
                        source ./env/bin/activate 
                        export PYTHONPATH="\$(pwd)/:\$(pwd)/tests"
                        export PERSISTER='{"engine_name": "redis", "parameters": {"host": "redis", "port": 6379, "db": 0}}'
                        echo "*** PYTHONPATH=\${PYTHONPATH}"
                        python3 -m pip install -r requirements.txt --no-cache --user
                        python3 -m unittest tests
                    """
                }
            }
        }
    }

    stage('Build image') {
        sh """
            docker build -t dsanders/cowbull:jenkins-test -f vendor/docker/Dockerfile .
        """
//        def pkg = docker.build("dsanderscan/cowbull", "-f vendor/docker/Dockerfile .")
    }

    stage('Push image') {
        withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'dockerhub',
usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD']]) {
            sh """
            docker login -u "${USERNAME}" -p "${PASSWORD}"
            docker tag dsanderscan/cowbull dsanderscan/cowbull:jenkins-test-"${env.BUILD_NUMBER}"
            docker push dsanderscan/cowbull:jenkins-test-"${env.BUILD_NUMBER}"
            """
        }
        /* Finally, we'll push the image with two tags:
         * First, the incremental build number from Jenkins
         * Second, the 'latest' tag.
         * Pushing multiple tags is cheap, as all the layers are reused. */
//        docker.withRegistry('https://registry.hub.docker.com', 'dockerhub') {
//            pkg.push("jenkins-test-${env.BUILD_NUMBER}")
//            // app.push("latest")
//        }
    }
}