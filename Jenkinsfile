podTemplate(yaml: '''
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: slave-worker
    image: docker:19.03.6
    command:
    - sleep
    args:
    - 99d
    env:
      - name: DOCKER_HOST
        value: tcp://localhost:2375
  - name: docker-daemon
    image: docker:19.03.6-dind
    securityContext:
      privileged: true
    env:
      - name: DOCKER_TLS_CERTDIR
        value: ""
''') {
    node(POD_LABEL) {
        git 'https://github.com/jenkinsci/docker-jnlp-slave.git'
        container('slave-worker') {
            sh 'docker version'
            def app

            stage('Clone repository') {
                /* Let's make sure we have the repository cloned to our workspace */

                checkout scm
            }

            stage('Build image') {
                /* This builds the actual image; synonymous to
                 * docker build on the command line */

                app = docker.build("jaesanglee/nginx")
            }

            stage('Test image') {
                app.inside {
                 sh 'echo "Tests passed"'
                }
            }

            stage('Push image') {
                /* Finally, we'll push the image with two tags:
                 * First, the incremental build number from Jenkins
                 * Second, the 'latest' tag.
                 * Pushing multiple tags is cheap, as all the layers are reused. */
                 docker.withRegistry('https://registry.hub.docker.com', 'docker-hub') {
                     app.push("${env.BUILD_NUMBER}")
                     app.push("latest")
                 }
            }
        }
    }
}
