pipeline {
    agent any
    tools{
        oc 'oc'
        maven 'maven-3.6.3'
        jdk 'jdk11'
    }
    environment {
        // This can be nexus3 or nexus2
        NEXUS_VERSION = "nexus3"
        // This can be http or https
        NEXUS_PROTOCOL = "http"
        // Where your Nexus is running
        NEXUS_URL = "localhost:8081"
        // Repository where we will upload the artifact
        NEXUS_REPOSITORY = "nexus-repo"
        // Jenkins credential id to authenticate to Nexus OSS
        NEXUS_CREDENTIAL_ID = "nexus-credentials"
    }
    stages {
        stage('Fetching Git Repository') {
            steps {
                git url: GIT_URL, branch: BRANCH
            }
        }
        stage ('Maven Build') {
            steps {
                  sh "mvn install"
                    sh "mvn package -DskipTests=true"
            }
        }
     
        /*
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
        */
        
        stage("publish to nexus") {
            steps {
                script {
                    // Read POM xml file using 'readMavenPom' step , this step 'readMavenPom' is included in: https://plugins.jenkins.io/pipeline-utility-steps
                    pom = readMavenPom file: "pom.xml";
                    sh "cd target"
                        

                    
                        
                        nexusArtifactUploader(
                            nexusVersion: NEXUS_VERSION,
                            protocol: NEXUS_PROTOCOL,
                            nexusUrl: NEXUS_URL,
                            groupId: pom.groupId,
                            version: pom.version,
                            repository: NEXUS_REPOSITORY,
                            credentialsId: NEXUS_CREDENTIAL_ID,
                            artifacts: [
                                // Artifact generated such as .jar, .ear and .war files.
                                [file: 'target/IterationDemo-1.0.0-SNAPSHOT.jar',
                                type: 'kjar'],

                                // Lets upload the pom.xml file for additional information for Transitive dependencies
                                [artifactId: pom.artifactId,
                                classifier: '',
                                file: "pom.xml",
                                type: "pom"]
                            ]
                        );

                
                }
        }
        }
        stage ('Building and Pushing Image to Quay') {
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
 
