def VersionVar=''
pipeline {
    agent any
    options { buildDiscarder( logRotator( numToKeepStr:'15'))}

    tools {
        maven "maven3.8.4"
        jdk "jdk1.8"
    }

    stages {

        stage('Build SpringBoot project') {          
            steps{
                echo '========^^^^^^ Start Building the SpringBoot project ^^^^^^========'
                dir('spring-boot-server'){
                    script {
                        if (currentBuild.result == null || currentBuild.result == 'SUCCESS') {
                            def buildversion = getReleaseVersion()
                            VersionVar = buildversion
                            bat "mvn  versions:set  -DgenerateBackupPoms=false -DnewVersion=${buildversion} clean package"   
                        
                        } else {
                            error "Build is not successful"
                        }
                    }
                }

            }    
        }

        stage('Build Angular Project') {
            steps {
                echo '========^^^^^^ Start Building the Angular project ^^^^^^========'
                dir('angular-11-client'){
                    bat 'npm install'
                    bat 'npm run ng -- build'
                    bat "tar.exe cf dist-${VersionVar}.zip dist" 
                }
            }
        }

    }

    post { always { archiveArtifacts artifacts: "**/target/*.jar, **/angular-11-client/dist-${VersionVar}.zip", onlyIfSuccessful: true } } 

}    

def getReleaseVersion() {
        def pom = readMavenPom file: 'pom.xml'
        def gitCommit = bat(returnStdout: true, script: 'git rev-parse HEAD').trim()
        def versionNumber;
        versionNumber = env.BUILD_NUMBER;
        
        def version = pom.version.replace("-SNAPSHOT", ".${versionNumber}-SNAPSHOT")
        version_array = version.split('\\.')
        //version_array.each() {
        //    echo it
        //}
        version_array[2] = "${version_array[2]}".toInteger() + 1
        version = version_array.join('.')
        pom.version = version
        writeMavenPom model: pom
        return version
}
        