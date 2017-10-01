#!groovy​
def gitUrl = 'https://github.com/ricardozanini/soccer-stats.git'
def nexusUrl = 'http://nexus.local:8081/repository/ansible-meetup'

stage('Build') {
    node {
        git gitUrl
        withEnv(["PATH+MAVEN=${tool 'm3'}/bin"]) {
            def pom = readMavenPom file: 'pom.xml'
            sh "mvn -B versions:set -DnewVersion=${pom.version}-${BUILD_NUMBER}"
            sh "mvn -B -Dmaven.test.skip=true clean package"
            stash name: "artifact", includes: "target/soccer-stats-*.jar"
            stash name: "source", includes: "**", excludes: "target/"
        }
    }
}

/*
stage('Unit Tests') {
    node {
        unstash 'source'
        withEnv(["PATH+MAVEN=${tool 'm3'}/bin"]) {
            sh "mvn -B clean test"
            stash name: "unit_tests", includes: "target/surefire-reports/**"
        }
    }
}

stage('Integration Tests') {
    node {
        unstash 'source'
        withEnv(["PATH+MAVEN=${tool 'm3'}/bin"]) {
            sh "mvn -B clean verify -Dsurefire.skip=true"
            stash name: 'it_tests', includes: 'target/failsafe-reports/**'
        }
    }
}

stage('Static Analysis') {
    node {
        unstash 'source'
        withEnv(["PATH+MAVEN=${tool 'm3'}/bin"]) {
            withSonarQubeEnv('sonar'){
                unstash 'it_tests'
                unstash 'unit_tests'
                sh 'mvn sonar:sonar -DskipTests'
            }
        }
    }
}
*/

stage('Artifact Upload') {
    node {
        unstash 'source'
        unstash 'artifact'

        def pom = readMavenPom file: 'pom.xml'
        def file = "${pom.artifactId}-${pom.version}"
        def jar = "target/${file}.jar"
        def path = "${pom.groupId}".replace(".", "/") + "/${pom.artifactId}/${pom.version}"

        echo "The path is ${path}"

        withCredentials([usernamePassword(credentialsId: 'nexus', passwordVariable: 'pass', usernameVariable: 'user')]) {
            sh "curl -v -u ${user}:${pass} --upload-file pom.xml ${nexusUrl}/${path}/${file}.pom"
            sh "curl -H 'Content-Type: application/java-archive' -v -u ${user}:${pass} --upload-file ${jar} ${nexusUrl}/${path}/${file}.jar"    
        }
        
        /*
        withEnv(["PATH+MAVEN=${tool 'm3'}/bin"]) {
            withCredentials([usernamePassword(credentialsId: 'nexus', passwordVariable: 'pass', usernameVariable: 'user')]) {
                sh "mvn deploy:deploy-file -DgroupId=${pom.groupId} " +
                "-DartifactId=${pom.artifactId} -Dversion=${pom.version} -DgeneratePom=false " +
                "-Dpackaging=jar -DrepositoryId=nexus -Durl=${nexusUrl} " +
                "-DpomFile=pom.xml -Dfile=${file}"
            }
        }
        */

        /*
        nexusArtifactUploader artifacts: [
                [artifactId: "${pom.artifactId}", classifier: '', file: "${file}.jar", type: 'jar']
            ], 
            credentialsId: 'nexus', 
            groupId: "${pom.groupId}", 
            nexusUrl: "${nexusUrl}", 
            nexusVersion: 'nexus3', 
            protocol: 'http', 
            repository: 'ansible-meetup', 
            version: "${pom.version}",
            type: "jar"
        */
    }
}

/*
stage('Approval') {
    timeout(time:3, unit:'DAYS') {
        input 'Do I have your approval for deployment?'
    }
}
*/

stage('Deploy') {
    node {
        ansiblePlaybook colorized: true, 
        credentialsId: 'nexus', 
        installation: 'ansible', 
        inventory: 'provision/inventori.ini', 
        playbook: 'provision/playbook.yml', 
        sudo: true,
        sudoUser: 'jenkins'
    }
}