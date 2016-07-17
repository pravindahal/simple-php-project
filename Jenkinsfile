#!groovy

node {

  // We are pushing to a private secure Docker registry in this demo.
  // 'dockerhub-pravindahal' is the username/password credentials ID as defined in Jenkins Credentials.
  // This is used to authenticate the Docker client to the registry.
  docker.withRegistry('https://index.docker.io/v1/', 'dockerhub-pravindahal') {

    stage 'Checkout'

    // Let's use default scm configuration to checkout
    checkout scm

    // Could not find native way to find commit id, so using this hack
    sh 'git log --format="%H" -n 1 > COMMIT_ID'
    commitId = readFile('COMMIT_ID').replaceAll("^\\s+","").replaceAll("\\s+\$","")
    sh 'rm COMMIT_ID'

    // Abbreviated commit hash
    sh 'git log --format="%h" -n 1 > COMMIT_ID_SHORT'
    commitIdShort = readFile('COMMIT_ID_SHORT').replaceAll("^\\s+","").replaceAll("\\s+\$","")
    sh 'rm COMMIT_ID_SHORT'

    // Get commit message
    sh 'git log --format="%s" -n 1 > COMMIT_MESSAGE'
    commitMessage = readFile('COMMIT_MESSAGE').replaceAll("^\\s+","").replaceAll("\\s+\$","")
    sh 'rm COMMIT_MESSAGE'

    // Get commit author
    sh 'git log --format="%an" -n 1 > COMMIT_AUTHOR'
    commitAuthor = readFile('COMMIT_AUTHOR').replaceAll("^\\s+","").replaceAll("\\s+\$","")
    sh 'rm COMMIT_AUTHOR'

    stage 'Bake Docker image'

    sh 'cp docker/nginx/Dockerfile .'
    def nginxImg = docker.build("pravindahal/simple-php-project.nginx:${commitId}")
    sh 'rm Dockerfile'

    sh 'cp docker/web/Dockerfile .'
    def webImg = docker.build("pravindahal/simple-php-project.web:${commitId}")
    sh 'rm Dockerfile'


    stage 'Push this version tagged by commit id'
    // Let us tag and push the newly built image. Will tag using the image name provided
    // in the 'docker.build' call above (which included the build number on the tag).
    nginxImg.push()
    webImg.push()


    stage name: 'Promote Image', concurrency: 1
    // All the tests passed. We can now retag and push the 'latest' image.
    nginxImg.push(env.BRANCH_NAME)
    webImg.push(env.BRANCH_NAME)
    //pcImg.push() //TODO version by calculating version number based on github release
    nginxImg.push('latest')
    webImg.push('latest')

    stage 'Slack Notify'

    githubUser = 'pravindahal'
    slackChannel = '#jenkins-build'
    repoName = 'simple-php-project'
    slackMessage = "<https://github.com/$githubUser/$repoName/tree/${env.BRANCH_NAME}|[$repoName:${env.BRANCH_NAME}]> Image built for commit by $commitAuthor\n `<https://github.com/$githubUser/$repoName/commit/$commitId|$commitIdShort>` $commitMessage"

    slackSend channel: slackChannel, color: 'good', message: slackMessage
  }
}
