#!groovy

node {

  // We are pushing to a private secure Docker registry in this demo.
  // 'docker-registry-login' is the username/password credentials ID as defined in Jenkins Credentials.
  // This is used to authenticate the Docker client to the registry.
  docker.withRegistry('https://index.docker.io/v1/', 'dockerhub-pravindahal') {

    stage 'Checkout'

    checkout scm

    echo "My branch is:"
    sh 'git log --format="%H" -n 1 > COMMIT_ID'
    commitId = readFile('COMMIT_ID').replaceAll("\\s+","")

    stage 'Bake Docker image'

    sh 'cp docker/nginx/Dockerfile .'
    def pcImg = docker.build("pravindahal/simple-php-project.nginx:${commitId}")
    sh 'rm Dockerfile'

    stage 'Push various versions'
    // Let us tag and push the newly built image. Will tag using the image name provided
    // in the 'docker.build' call above (which included the build number on the tag).
    pcImg.push()

    stage name: 'Promote Image', concurrency: 1
    // All the tests passed. We can now retag and push the 'latest' image.
    pcImg.push(env.BRANCH_NAME)
    //pcImg.push() //TODO version by calculating version number based on github release
    pcImg.push('latest')
  }
}
