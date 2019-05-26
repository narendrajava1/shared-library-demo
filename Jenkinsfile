@Library('shared-library') _
node(label: 'master'){
    //Variables
    def gitURL = "https://github.com/anoop600/MVC.git"
    def repoBranch = "master"
    def applicationName = "mvc"
    def sonarqubeServer = "sonar"
    def sonarqubeGoal = "clean verify sonar:sonar"
    def mvnHome = "MAVEN_HOME"
    def pom = "pom.xml"
    def goal = "clean install"
    def artifactoryServer = "Artifactory"
    def releaseRepo = "generic-local"
    def snapshotRepo = "generic-snapshot"
    def dockerRegistry = "https://registry.hub.docker.com"
    def dockerImageRemove = "registry.hub.docker.com"
    def dockerRegistryUserName = "anoop600"
    def dockerCredentialID = "dockerID" 
    def dockerImageName = "${dockerRegistryUserName}/${applicationName}"
    def vmPort = 9999
    def containerPort = 8080
    def lastSuccessfulBuildID = 0
    
    //Git Stage
    stage('Git-Checkout'){
        gitClone "${gitURL}","${repoBranch}"    
    }
    
    //Sonarqube Analysis
    stage('Sonarqube-scan'){
        sonarqubeScan "${mvnHome}","${sonarqubeGoal}","${pom}", "${sonarqubeServer}"
    }
    
    //Quality-gate
    stage('Quality-Gate'){
        qualityGate "${sonarqubeServer}"
    }
    
    //MVN Build
    stage('Maven Build and Push to Artifactory'){
        mavenBuild "${artifactoryServer}","${mvnHome}","${pom}", "${goal}", "${releaseRepo}", "${snapshotRepo}"
    }
    
    //docker-image-build
    stage('Build Docker image'){
        dockerBuildAndPush "${dockerRegistry}","${dockerCredentialID}","${dockerImageName}"
    }
    
    //Remove extra image
    stage('Remove image'){
        removeDockerImage "${dockerImageRemove}","${dockerImageName}"
    }
    
    stage('Get Last Successful Build Number'){
        //def lastSuccessfulBuildID = 0
        def build = currentBuild.previousBuild
        while (build != null) {
            if (build.result == "SUCCESS")
            {
                lastSuccessfulBuildID = build.id as Integer
                break
            }
            build = build.previousBuild
        }
        echo "${lastSuccessfulBuildID}"
        
 
    }
    
    //Delete Old running Container and run new built
    stage('Run Docker Image'){
        echo "Last Successful Build = ${lastSuccessfulBuildID}"
        runDockerImage "${vmPort}","${containerPort}", "${dockerImageName}", "${BUILD_NUMBER}", "${lastSuccessfulBuildID}"
    }
    
}
