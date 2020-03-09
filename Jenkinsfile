pipeline {
    agent {
        dockerfile {
            args '--name paslushenka_jenkins -u 0:0 --network=paslushenka_itnet -v /var/run/docker.sock:/var/run/docker.sock'
            additionalBuildArgs '-t paslushenka_jenkins'
        }
    }
    stages {
        stage('project_build') {
            sh """
                export PATH="\$PATH:${WORKSPACE}/bin"
                sed -i \'s/1.DEVELOPMENT/1.${BUILD_NUMBER}/g\' ./rice-box.go
                make lint
                make test
                make clean
                make build
                md5sum artifacts/*/word-cloud-generator* >artifacts/word-cloud-generator.md5
                gzip artifacts/*/word-cloud-generator*
                chmod -R 777 /workspace
            """
            nexusArtifactUploader artifacts: [[artifactId: 'word-cloud-generator', classifier: '', file: 'artifacts/linux/word-cloud-generator.gz', type: 'gz']], credentialsId: 'nexus-creds', groupId: '1', nexusUrl: 'nexus:8081', nexusVersion: 'nexus3', protocol: 'http', repository: 'word-cloud-generator', version: '1.$BUILD_NUMBER'
        }
        stage('project_test') {
            environment {
                NEXUS_CREDS = credentials('nexus-creds')
            }
            steps {
                sh """
                    docker build -t wcgen_alpine --build-arg NEXUS_CREDS=${NEXUS_CREDS} --build-arg BUILD_NUMBER=${BUILD_NUMBER} --network=paslushenka_itnet -f ./wcgen/Dockerfile .
                """
                sh "docker run -d --name wcgen_alpine --network=paslushenka_itnet wcgen_alpine"
                sh """
                    res=`curl -s -H "Content-Type: application/json" -d '{"text":"ths is a really really really important thing this is"}' http://wcgen_alpine:8888/version | jq '. | length'`
                    if [ "1" != "\$res" ]; then
                      exit 99
                    fi

                    res=`curl -s -H "Content-Type: application/json" -d '{"text":"ths is a really really really important thing this is"}' http://wcgen_alpine:8888/api | jq '. | length'`
                    if [ "7" != "\$res" ]; then
                      exit 99
                    fi
                """
                sh "docker rm -f wcgen_alpine"
                sh "docker rmi wcgen_alpine"
            }
        }
    }
}
