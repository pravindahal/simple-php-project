#!groovy

node {

  // We are pushing to a private secure Docker registry in this demo.
  // 'dockerhub-pravin' is the username/password credentials ID as defined in Jenkins Credentials.
  // This is used to authenticate the Docker client to the registry.
  docker.withRegistry('https://index.docker.io/v1/', 'dockerhub-pravindahal') {

    stage 'Checkout'

    // Let's use default scm configuration to checkout
    checkout scm

    // Could not find native way to find commit id, so using this hack
    sh 'git log --format="%H" -n 1 > COMMIT_ID'
    commitId = readFile('COMMIT_ID').replaceAll("\\s+","")

    // Can't get branch name in normal pipeline job
    sh 'git rev-parse --abbrev-ref HEAD > BRANCH_NAME'
    branchName = readFile('BRANCH_NAME').replaceAll("\\s+","")


    stage 'Bake Docker image'

    sh 'cp docker/nginx/Dockerfile .'
    def pcImg = docker.build("pravindahal/simple-php-project.nginx:${commitId}")
    sh 'rm Dockerfile'


    stage 'Push this version tagged by commit id'
    // Let us tag and push the newly built image. Will tag using the image name provided
    // in the 'docker.build' call above (which included the build number on the tag).
    pcImg.push()


    stage name: 'Promote Image', concurrency: 1
    // All the tests passed. We can now retag and push the 'latest' image.
    pcImg.push(branchName)
    //pcImg.push() //TODO version by calculating version number based on github release
    pcImg.push('latest')
  }
}
