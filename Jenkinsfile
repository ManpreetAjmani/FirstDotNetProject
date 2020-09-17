#!groovy
pipeline {
    agent {
        docker {
            image 'nosinovacao/dotnet-sonar:20.07.0'
        }
    }
    environment {
        HOME = '/tmp'
    } 
    stages {
        stage('Preparation') {
            steps {
                checkout scm
            }
        }
        
   
        stage('Build') {
            steps {
                sh """
                #!/bin/bash
                dotnet build FirstSolution.sln
                """
            }
        }
        stage('Test') {
            steps {
                sh """
                #!/bin/bash
                dotnet test /p:CollectCoverage=true /p:CoverletOutputFormat=opencover
                """
            }
        }
        
        stage('Sonar') {
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh """
                    #!/bin/bash
                    ls FirstCoreProject
                    dotnet build-server shutdown
                    dotnet /sonar-scanner/SonarScanner.MSBuild.dll begin /k:"FirstCoreProject" /d:sonar.host.url=http://172.18.0.2:9000 /d:sonar.login="89766ff7ff2ae765e271a71d9b16626b71335e97" /d:sonar.cs.opencover.reportsPaths="FirstCoreProject/coverage.opencover.xml" 
                    dotnet build FirstSolution.sln
                    dotnet /sonar-scanner/SonarScanner.MSBuild.dll  /d:sonar.login="89766ff7ff2ae765e271a71d9b16626b71335e97" end
                    """
                }    
            }
        }
        
        stage("Quality Gate") {
            steps {
              timeout(time: 5, unit: 'MINUTES') {
                waitForQualityGate abortPipeline: true
              }
            }
        }

        stage('Run') {
            steps {
                sh """
                #!/bin/bash
                cd FirstCoreProject
                dotnet run
                """
            }
        }
    }
}
