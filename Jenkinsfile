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

    branchName = env.BRANCH_NAME
    tagifiedBranchName = branchName.replaceAll("/", "--")

    // Lets find out if this commit has git tags
    sh("git show-ref --tags | grep $commitId  | cut -d' ' -f2 > RELEASE")
    releaseVersion = readFile('RELEASE').replaceAll("^\\s+","").replaceAll("\\s+\$","").replaceAll("refs/tags/", "")

    stage 'Bake Docker images'

    sh 'cp docker/nginx/Dockerfile .'
    def nginxImg = docker.build("pravindahal/simple-php-project.nginx:$commitId")
    sh 'rm Dockerfile'

    sh 'cp docker/web/Dockerfile .'
    def webImg = docker.build("pravindahal/simple-php-project.web:$commitId")
    sh 'rm Dockerfile'


    stage 'Push this version tagged by commit id'

    dockerhubTagNames = ""
    dockerhubTagNames = dockerhubTagNames + " `$commitId`"

    // Let us tag and push the newly built image. Will tag using the image name provided
    // in the 'docker.build' call above (which included the build number on the tag).
    parallel(nginxImage: {
        nginxImg.push()
    }, webImage: {
        webImg.push()
    })


    stage name: 'Promote Image', concurrency: 1

    dockerhubTagNames = dockerhubTagNames + " `$tagifiedBranchName`"
    dockerhubTagNames = dockerhubTagNames + " `latest`"
    if (releaseVersion != "") {
        dockerhubTagNames = dockerhubTagNames + " `$releaseVersion`"
    }

    parallel(nginxImage: {
        nginxImg.push(tagifiedBranchName)
        nginxImg.push('latest')
        if (releaseVersion != "") {
            nginxImg.push(releaseVersion)
        }
    }, webImage: {
        webImg.push(tagifiedBranchName)
        webImg.push('latest')
        if (releaseVersion != "") {
            webImg.push(releaseVersion)
        }
    })

    stage 'Slack Notify'

    githubUser = 'pravindahal'
    slackChannel = '#jenkins-build'
    repoName = 'simple-php-project'
    githubBranchUrl = "https://github.com/$githubUser/$repoName/tree/$branchName"
    githubCommitUrl = "https://github.com/$githubUser/$repoName/commit/$commitId"
    dockerUsername = 'pravindahal'
    dockerRepo1Name = 'simple-php-project.web'
    dockerRepo1Url = "https://hub.docker.com/r/$dockerUsername/$dockerRepo1Name/tags/"
    dockerRepo2Name = 'simple-php-project.nginx'
    dockerRepo2Url = "https://hub.docker.com/r/$dockerUsername/$dockerRepo2Name/tags/"

    githubMessage = "*<$githubBranchUrl|[$repoName:$branchName]>* Images built for commit by $commitAuthor\n `<$githubCommitUrl|$commitIdShort>` $commitMessage"
    dockerhubMessage = "<$dockerRepo1Url|[$dockerRepo1Name]>$dockerhubTagNames\n\n<$dockerRepo2Url|[$dockerRepo2Name]>$dockerhubTagNames"

    slackMessage = "$githubMessage\n$dockerhubMessage"

    slackSend channel: slackChannel, color: 'good', message: slackMessage
  }
}
