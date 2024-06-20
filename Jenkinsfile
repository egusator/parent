pipeline {
    agent any
    triggers { pollSCM('* * * * *') }

    environment {
        JAVA_HOME = "/usr/lib/jvm/java-17-openjdk-amd64"
    }

    stages {

        stage('Checkout') {
            steps {
                sh 'git submodule update --init --recursive'
            }
        }

        stage('Prepare') {
            steps {
                dir ('/out') {
                    sh 'rm -rf snapshot && mkdir -p snapshot'
                }
                dir('base/sounds') {
                    sh './make'
		    sh 'rm -f ../../luwrain/src/main/resources/org/luwrain/core/sound/* && cp *.wav ../../luwrain/src/main/resources/org/luwrain/core/sound/'
            }
        }
	}

        stage('Build') {
            steps {
                sh 'mvn install'
                dir('base/scripts') {
                    sh './lwr-ant-gen-all'
                    sh './lwr-build'
                }
            }
        }

    stage ('Snapshot') {
        steps {
            dir ('base/scripts') {
                sh './lwr-snapshot /out/snapshot'
            }
            dir ('out') {
                sh 'date > /out/snapshot/timestamp.txt'
            }
        }
    }
	    
    stage('Generate win dist') {
            steps {
                dir('base/scripts') {
                    sh './lwr-dist-win /out/win_generated_files'
                }
		dir('out/win_generated_files/installer-x64') {
		    sh 'docker run --rm -i -v "$(pwd):/work" amake/innosetup Luwrain.iss'
		}
            }
        }
    }
}
