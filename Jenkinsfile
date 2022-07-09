podTemplate(inheritFrom: "mem2_large", containers: [
        containerTemplate(name: 'build', image: 'registry.aws-digital.rccl.com/commerce/hybris-pipeline-template:1.0.1', command: 'sleep', args: '99d', resourceLimitCpu: '2000m', resourceRequestCpu: '400m', resourceRequestMemory: '3584Mi', resourceLimitMemory: '6144Mi')
        
]) {
    node(POD_LABEL) {
        container("build") {
            
            env.GROOVY_HOME = "${tool 'groovy-2.4.9'}"
            env.PATH = "${env.GROOVY_HOME}/bin:${env.PATH}"  
            env.JAVA_HOME = "/usr/lib/jvm/sapmachine-11"
            env.MAVEN_HOME = "${tool 'Maven 3.5'}"
            env.PATH = "${env.MAVEN_HOME}/bin:${env.PATH}"
            env.HYBRIS_OPT_CONFIG_DIR = "$WORKSPACE/hybris/config/envs/ci"
            
            stage('Maven') {
                sh "mvn -v"
                sh "cat ${env.MAVEN_HOME}/conf/settings.xml"
            }
        }
    }
    
}
