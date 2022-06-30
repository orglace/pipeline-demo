node {
    def mvnHome
    def repoAddress = 'git@github.com:orglace/pipeline-demo.git'
    stage('Preparation') { // for display purposes
        // Get some code from a GitHub repository
        stageCheckout(repoAddress, "*/${env.BRANCH_NAME}", 'leeroy-jenkins-ssh')
        // Get the Maven tool.
        // ** NOTE: This 'M3' Maven tool must be configured
        // **       in the global configuration.
        mvnHome = tool 'M3'
    }

    // Tag Creation Stage
    stageTagCreation(repoAddress, env.BRANCH_NAME, 'leeroy-jenkins-ssh')

    stage('Build') {
        // Run the maven build
        withEnv(["MVN_HOME=$mvnHome"]) {
            if (isUnix()) {
                sh '"$MVN_HOME/bin/mvn" -Dmaven.test.failure.ignore clean package'
            } else {
                bat(/"%MVN_HOME%\bin\mvn" -Dmaven.test.failure.ignore clean package/)
            }
        }
    }
    stage('Results') {
        junit '**/target/surefire-reports/TEST-*.xml'
        archiveArtifacts 'target/*.jar'
    }
}

def stageCheckout(repo, branch, credentials) {
  checkout([
    $class                           : 'GitSCM',
    branches                         : [[name: branch]],
    doGenerateSubmoduleConfigurations: false,
    extensions                       : [[$class: 'CleanCheckout']],
    submoduleCfg                     : [],
    userRemoteConfigs                : [[credentialsId: credentials, url: repo]]
  ])
}

def stageTagCreation(def repo, String currentBranch, credentials) {
    if(currentBranch.equalsIgnoreCase('master')) {

        stage('Tag Creation') {
            sshagent(credentials: [credentials]) {
                newTag = sh(script: 'git log --merges -n1 --format="%s%n%b" | grep -m 1 -o "from [a-z]*\\/*release\\/[0-9]\\+\\.[0-9]\\+\\.[0-9]\\+" | sed "s/from [a-z]*\\/*release\\//v/"', returnStdout: true).trim()
                echo "The new tag ${newTag} for ${currentBranch}"
                
                tagExist = sh(script: "git tag -l ${newTag}", returnStdout: true).trim()
                echo "Tag existence result: ${tagExist?: 'none'}"
                if(!tagExist) {
                    echo "Tag ${newTag} must be created on ${currentBranch}"
                    createTag(newTag)
                } else {
                    echo "Tag ${newTag} already exist on ${currentBranch}"
                }
            }
        }
    }
}

def createTag(def tag) {

    echo "Creating/Pushing Git tag: ${tag}"
    sh(script: """
        git remote -vv
        git config user.email leeroyjenkins@rccl.com
        git config user.name leeroy_jenkins
        git tag -a ${tag} -m release/${tag.substring(1)}
        git push --tags
    """)
}