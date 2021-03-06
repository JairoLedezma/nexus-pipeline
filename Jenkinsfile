pipeline {
    agent any
    tools{
        oc 'oc'
        maven 'maven-3.6.3'
        jdk 'jdk8'
    }
    stages {
        stage('Fetching Git Repository') {
            steps {
                git url: GIT_URL, branch: BRANCH
            }
        }
        stage ('Deploy Kieserver') {
            steps {
                script {
                    openshift.withCluster( CLUSTER_NAME ) {
                        openshift.withProject( PROJECT_NAME ){
                            def processedTemplate
                            
                           
                           if( NEW_PROJECT ){
                                 try {
                                    processedTemplate = openshift.process( "-f", "./templates/template-create.yaml", "--param-file=./templates/template-create.env")
                                    def createResources = openshift.create( processedTemplate )
                                    createResources.logs('-f')
                                    
                                 } catch (err) {
                                     echo err.getMessage()
                                     
                                }
                            } 
                         
                        }
                    }
                }
           }
        }
        
        stage ('Building and Pushing Image to Nexus') {
            steps {
                script {
                    openshift.withCluster( CLUSTER_NAME ) {
                        openshift.withProject( PROJECT_NAME ){
                            def buildConfig = openshift.selector( 'buildconfig/' + BUILD_CONFIG )
                            buildConfig.startBuild()
                            buildConfig.logs('-f')
                        }
                    }
                }    
            }
        }
        stage ('Deploying') {
            steps {
                script {
                    openshift.withCluster( CLUSTER_NAME ) {
                        openshift.withProject( PROJECT_NAME ){
                            def devConfig = openshift.selector( 'deploymentconfig/' + DEPLOYMENT_CONFIG )
                            devConfig.rollout().latest()
                        }
                    }
                }    
            }
        }
    }

}
 
