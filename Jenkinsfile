node {
    def mvnHome
    def repoAddress = 'https://github.com/orglace/pipeline-demo.git'
    stage('Preparation') { // for display purposes
        // Get some code from a GitHub repository
        stageCheckout(repoAddress, "*/${env.BRANCH_NAME}")
        // Get the Maven tool.
        // ** NOTE: This 'M3' Maven tool must be configured
        // **       in the global configuration.
        mvnHome = tool 'M3'
    }

    // Tag Creation Stage
    stageTagCreation(env.BRANCH_NAME)

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

def stageCheckout(repo, branch) {
  checkout([
    $class                           : 'GitSCM',
    branches                         : [[name: branch]],
    doGenerateSubmoduleConfigurations: false,
    extensions                       : [[$class: 'CleanCheckout']],
    submoduleCfg                     : [],
    userRemoteConfigs                : [[url: repo]]
  ])
}

def stageTagCreation(String currentBranch) {
    
    if(currentBranch.equalsIgnoreCase('master')) {
        stage('Tag Creation') {
            
            newTag = sh(script: 'git log --merges -n1 --format="%s%n%b" | grep -m 1 -o "from [a-z]*\\/*release\\/[0-9]\\+\\.[0-9]\\+\\.[0-9]\\+" | sed "s/from [a-z]*\\/*release\\//v/"', returnStdout: true).trim()
            echo "The new tag ${newTag} for ${currentBranch}"
            
            tagExist = sh(script: "git tag -l ${newTag}", returnStdout: true).trim()
            echo "Tag existence result: [${tagExist}]"
            if(!tagExist) {
                echo "Tag ${newTag} must be created on ${currentBranch}"
                createTag(newTag)
            } else {
                echo "Tag ${newTag} already exist on ${currentBranch}"
            }
        }
    }
}

def createTag(def tag) {

    echo "Creating/pushing Git tag: ${tag}"
    sh(script: """
        git config user.email leeroyjenkins@rccl.com
        git config user.name leeroy_jenkins
        git tag -a ${tag} -m release/${tag.substring(1)}
        git push --tags
    """)
}