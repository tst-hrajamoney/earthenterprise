pipeline {
  agent { label "opengee-build-el7" }

  options {
        disableConcurrentBuilds()
    }

  stages {
    stage('Prep') {
      steps {
        setup()
        setEnv()
      }
    }
    stage('Tag Build') {
      steps {
        tagBuild()
      }
    }

    stage('Unit Tests') {
      parallel {
        stage('CentOS 6') {
          agent { label "opengee-build-el6" }
          stages {
            stage('Unit Test') {
              steps {
                sh """
                  ./gradlew createBuildTag
                  ./gradlew clean unitTest -PgeepPlatform=el6
                """
              }
            }
          }
          post {
            cleanup {
              cleanWs()
            }
          }
        }

        stage('CentOS 7') {
          agent { label "opengee-build-el7" }
          stages {
            stage('Unit Test') {
              steps {
                sh """
                  ./gradlew createBuildTag
                  ./gradlew clean unitTest -PgeepPlatform=el7
                """
              }
            }
          }
          post {
            cleanup {
              cleanWs()
            }
          }
        }

      }
    }

    stage('Build Docs') {
      steps {
        buildDocs()
        buildDocsPortable()
      }
    }

    stage('Spawn Builds') {
      parallel {
        stage('CentOS 6') {
          agent { label "opengee-build-el6" }
          stages {
            stage('Build GEEP installer') {
              steps {
                sh './gradlew createBuildTag'
                // get the sphinx docs
                unstash 'docs'
                // build the rest of geep
                sh """
                  ./gradlew clean distTar
                  tree -L 3 build
                """
              }
            }

            stage('Archive') {
              steps {
                tryArchive()
              }
            }

            stage('Publish') {
              steps {
                tryPublish("el6")
              }
            }

            stage('Functional Test') {
              when {
                expression { env.DO_TEST_DEPLOY == 'true' }
              }
              agent { label 'centos7-gauge'}
              steps {
                acquireInstance("${AMI_IMAGE_6}")
                script {
                  env.EL6_GEEP_ID = "${env.GEEP_ID}"
                  env.EL6_GEEP_IP = "${env.GEEP_IP}"
                }
                prepInstance("${EL6_GEEP_IP}","el6")
                runFuncTests("${EL6_GEEP_IP}","el6")
              }
              post {
                failure {
                  makeDir("logs")
                  sshagent (credentials: ['jenkins2-centos']) {
                    sh """
                      ssh -o StrictHostKeyChecking=no centos@${EL6_GEEP_IP} "
                        mkdir -p /tmp/test-logs/gehttpd || true
                        mkdir -p /tmp/test-logs/pgsql || true
                        sudo chmod o+rx /opt/google/gehttpd/logs/ || true
                        sudo cp /opt/google/gehttpd/logs/*.out /tmp/test-logs/gehttpd || true
                        sudo cp /opt/google/gehttpd/logs/*_log /tmp/test-logs/gehttpd || true
                        cp /var/opt/google/log/* /tmp/test-logs || true
                        sudo cp /var/opt/google/pgsql/logs/pg.log /tmp/test-logs/pgsql || true
                        sudo cp /var/log/secure /tmp/test-logs || true
                        sudo cp /var/log/messages /tmp/test-logs || true
                        sudo cp /var/log/dmesg /tmp/test-logs || true
                        sudo cp /var/log/faillog /tmp/test-logs || true
                        sudo chmod -R 0777 /tmp/test-logs/* || true
                      "
                      scp -o StrictHostKeyChecking=no -r centos@${EL6_GEEP_IP}:/tmp/test-logs/* logs/ || true
                    """
                  }
                  sh 'zip -r logs_el6.zip logs'
                  echo 'archiving logs...'
                  archiveArtifacts artifacts: 'logs_el6.zip', allowEmptyArchive: true
                }
                cleanup {
                  echo "terminating VM..."
                  sh "aws ec2 terminate-instances --instance-ids ${EL6_GEEP_ID}"
                  cleanWs()
                }
              }
            }
          }
          post {
            cleanup {
              cleanWs()
            }
          }
        }

        stage('CentOS 7') {
          stages {
            stage('Build GEEP installer') {
              steps {
                sh """
                  ./gradlew clean distTar
                  tree -L 3 build
                """
              }
            }

            stage('Archive') {
              steps {
                tryArchive()
              }
            }

            stage('Publish') {
              steps {
                tryPublish("el7")
              }
            }

            stage('Functional Test') {
              when {
                expression { env.DO_TEST_DEPLOY == 'true' }
              }
              agent { label 'centos7-gauge'}
              steps {
                acquireInstance("${AMI_IMAGE_7}")
                script {
                  env.EL7_GEEP_ID = "${env.GEEP_ID}"
                  env.EL7_GEEP_IP = "${env.GEEP_IP}"
                }
                prepInstance("${EL7_GEEP_IP}","el7")
                runFuncTests("${EL7_GEEP_IP}","el7")
              }
              post {
                failure {
                  makeDir("logs")
                  sshagent (credentials: ['jenkins2-centos']) {
                    sh """
                      ssh -o StrictHostKeyChecking=no centos@${EL7_GEEP_IP} "
                        mkdir -p /tmp/test-logs/gehttpd || true
                        mkdir -p /tmp/test-logs/pgsql || true
                        sudo chmod o+rx /opt/google/gehttpd/logs/ || true
                        sudo cp /opt/google/gehttpd/logs/*.out /tmp/test-logs/gehttpd || true
                        sudo cp /opt/google/gehttpd/logs/*_log /tmp/test-logs/gehttpd || true
                        cp /var/opt/google/log/* /tmp/test-logs || true
                        sudo cp /var/opt/google/pgsql/logs/pg.log /tmp/test-logs/pgsql || true
                        sudo cp /var/log/secure /tmp/test-logs || true
                        sudo cp /var/log/messages /tmp/test-logs || true
                        sudo cp /var/log/dmesg /tmp/test-logs || true
                        sudo cp /var/log/faillog /tmp/test-logs || true
                        sudo chmod -R 0777 /tmp/test-logs/* || true
                      "
                      scp -o StrictHostKeyChecking=no -r centos@${EL7_GEEP_IP}:/tmp/test-logs/* logs/ || true
                    """
                  }
                  sh 'zip -r logs_el7.zip logs'
                  echo 'archiving logs...'
                  archiveArtifacts artifacts: 'logs_el7.zip', allowEmptyArchive: true
                }
                cleanup {
                  echo "terminating VM..."
                  sh "aws ec2 terminate-instances --instance-ids ${EL7_GEEP_ID}"
                  cleanWs()
                }
              }
            }
          }
          post {
            cleanup {
              cleanWs()
            }
          }
        }

        stage("Windows") {
            agent { label 'windows' }
            stages {
              stage('Build win64') {
                steps {
                  dir('portable') {
                    unstash 'docsPortable'

                    bat 'gradlew.bat createBuildTag'
                    bat "gradlew.bat -PwinArch=win64 clean build"
                  }
                }
              }
              stage('Build win32') {
                steps {
                  dir('portable') {
                    unstash 'docsPortable'

                    bat 'gradlew.bat createBuildTag'
                    bat "gradlew.bat -PwinArch=win32 clean build"
                  }
                }
              }
              stage('Publish win') {
                steps {
                  dir('portable') {
                    // get the sphinx docs
                    bat "gradlew.bat publish"
                  }
                }
              }
          }
          post {
            cleanup {
              cleanWs()
            }
          }
        }
        stage("Portable el7") {
          agent { label 'opengee-build-el7' }
          stages {
            stage('Setup el7 build env') {
              steps {
                dir('portable') {
                  //TODO: remove this plugin dependency
                  ansiblePlaybook(
                    playbook: 'ansible/pre-build.yml',
                    inventoryContent: "localhost,",
                    extras: "-c local"
                  )
                }
              }
            }
            stage('Build el7 portable') {
              steps {
                sh './gradlew createBuildTag'
                dir('portable') {
                  sh './gradlew clean build publish'
                }
              }
            }
          }
          post {
            cleanup {
              cleanWs()
            }
          }
        }
        stage("Portable el6") {
          agent { label 'opengee-build-el6' }
          stages {
            stage('Setup el6 build env') {
              steps {
                dir('portable') {
                  //TODO: remove this plugin dependency
                  ansiblePlaybook(
                    playbook: 'ansible/pre-build.yml',
                    inventoryContent: "localhost,",
                    extras: "-c local"
                  )
                }
              }
            }
            stage('Build el6 portable') {
              environment {
                PATH = "/opt/rh/devtoolset-2/root/usr/bin:$PATH"
              }
              steps {
                sh './gradlew createBuildTag'
                dir('portable') {
                  sh './gradlew clean build publish'
                }
              }
            }
          }
          post {
            cleanup {
              cleanWs()
            }
          }
        }

      }
    }
  }

  post {
    success {
      withCredentials([string(credentialsId: 'SLACK_TOKEN', variable: 'SLACK_TOKEN')]) {
        slackSend (color: '#00FF00', channel: "gee-bots", token: "$SLACK_TOKEN", message: "SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
      }
    }
    failure {
      withCredentials([string(credentialsId: 'SLACK_TOKEN', variable: 'SLACK_TOKEN')]) {
        slackSend (color: '#FF0000', channel: "gee-bots", token: "$SLACK_TOKEN", message: "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
      }
      emailAlert()
    }
    aborted {
      withCredentials([string(credentialsId: 'SLACK_TOKEN', variable: 'SLACK_TOKEN')]) {
        slackSend (color: '#FFA600', channel: "gee-bots", token: "$SLACK_TOKEN", message: "ABORTED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
      }
      // mail to:"${COMMIT_AUTHOR_EMAIL}", subject:"${currentBuild.fullDisplayName} - Aborted!", body: "Aborted at ${env.BUILD_URL}!"
    }
    unstable {
      withCredentials([string(credentialsId: 'SLACK_TOKEN', variable: 'SLACK_TOKEN')]) {
        slackSend (color: '#FFA600', channel: "gee-bots", token: "$SLACK_TOKEN", message: "UNSTABLE: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
      }
      // mail to:"${COMMIT_AUTHOR_EMAIL}", subject:"${currentBuild.fullDisplayName} - Unstable!", body: "Unstable at ${env.BUILD_URL}!"
    }
    cleanup {
      cleanWs()
    }
  }
}

def makeDir(def dir) {
  dirCheck = sh (script: "test -d ${dir} && echo '1' || echo '0' ", returnStdout: true).trim()
  if (dirCheck=='1') {
    echo "Cleaning contents of following directory: $dir"
    sh "rm -rf $dir"
  }
  sh "mkdir -p $dir"
}

def gitCheckout(def repo, def branch) {
  sh """
    rm -rf $repo
    git clone "git@bitbucket.org:t-sciences/${repo}.git"
    cd $repo
    git checkout $branch
  """
}

def setup() {
  withCredentials([file(credentialsId: 'id_rsa_nstires', variable: 'GIT_KEY')]) {
    sh """
      mkdir -p ~/.ssh/
      chmod 750 ~/.ssh/
      cat $GIT_KEY > ~/.ssh/id_rsa || true
      chmod 600 ~/.ssh/id_rsa
      ssh-keygen -y -f ~/.ssh/id_rsa | sudo tee -a /root/.ssh/authorized_keys

      ssh-keyscan github.com >> ~/.ssh/known_hosts
      ssh-keyscan bitbucket.org >> ~/.ssh/known_hosts
      ssh-keyscan localhost >> ~/.ssh/known_hosts
      ssh-keyscan 127.0.0.1 >> ~/.ssh/known_hosts
      ssh-keyscan `hostname` >> ~/.ssh/known_hosts
    """
  }
}

def setEnv() {
  // Set defaults
  env.ANSIBLE_INVENTORY_VARS = 'ansible_user=centos ansible_become=yes geep_firewall=true'
  env.GEEP_BUILD_ARGS = ''
  env.BUILD_TYPE = "$BRANCH_NAME"
  env.PR = 'false'
  env.DO_TEST_DEPLOY = 'true'
  env.DO_ARCHIVE = 'true'
  env.DO_PUBLISH = 'true'
  env.BLAST_EMAIL = 'false'
  env.ANSIBLE_HOST_KEY_CHECKING = 'False'
  env.AMI_IMAGE_6 = "ami-0c8bf051b55247400"
  env.AMI_IMAGE_7 = "ami-058f5ca0bea34f985"
  env.FUNCTIONAL_TEST_AGENT = 'centos7-gauge'
  env.GAUGE_SUBNET = 'subnet-75664c01'
  env.GAUGE_TAGS = '{Key=pipeline,Value=gauge},{Key=Project,Value=DevTesting}'
  // Override defaults based on google/earthenterprise branch/tag/PR
  if ( BUILD_TYPE.startsWith("PR-") ) {
    env.DO_PUBLISH = 'true'
    env.PR = 'true'
    env.PR_NUM = sh (script: "echo ${BRANCH_NAME} | cut -d '-' -f 2", returnStdout: true).trim()
  }
  if ( BUILD_TYPE.equals("master") ) {
    env.DO_PUBLISH = 'true'
    env.BLAST_EMAIL = 'true'
  }
  if ( BUILD_TYPE.equals("dev") ) {
    env.BLAST_EMAIL = 'true'
  }
  if ( JENKINS_URL.equals("https://ci-gee-product.t-sciences.com/") ) {
    env.GAUGE_SUBNET = 'subnet-01b2077a7097cddd4'
    env.GAUGE_TAGS = '{Key=pipeline,Value=gauge},{Key=Activity,Value=GEEDev},{Key=Environment,Value=Dev},{Key=Owner,Value=jenkins},{Key=Team,Value=DevOps}'
  }
  sh "env"
}

def tagBuild() {
  sh """
    ./gradlew createBuildTag
    git describe > described.txt
  """
  script {
    currentBuild.displayName = readFile('described.txt').trim()
  }
}

def buildDocs() {
  sh "./gradlew createBuildTag"
  sh "./gradlew -b docs/build.gradle clean build"
  stash name: 'docs', includes: 'docs/build/sphinx-output/**/*'
}

def buildDocsPortable() {
  dir('portable') {
    sh './gradlew buildSphinxDocs'
    stash name: 'docsPortable', includes: 'installer/docs/**/*'
  }
}

def buildInstaller() {
  unstash 'docs'
  sh """
    ./gradlew ${GEEP_BUILD_ARGS} clean distTar
    tree -L 3 build
  """
  stash name: 'docs', includes: 'docs/build/sphinx-output/**/*'
}

def tryArchive() {
  if ( env.DO_ARCHIVE.toBoolean() ) {
    archive "build/distributions/geep*.tgz"
  }
}

def tryPublish(def platform) {
  if ( env.DO_PUBLISH.toBoolean() ) {
    unstash 'docs'
    sh """
      ./gradlew ${GEEP_BUILD_ARGS} clean distTar publish -PROOT_ONLY=true
      cp -p build/distributions/get_geep.sh build/distributions/get_geep_${platform}.sh
    """
    archiveArtifacts "build/distributions/get_geep_${platform}.sh"
    stash name: "get_geep_${platform}", includes: "build/distributions/get_geep_${platform}.sh"
  }
}

def acquireInstance(def amiImage) {
  echo 'creating VM...'
  script {
    sh """
      ./gradlew createBuildTag
      echo "\$(git describe)"
      aws ec2 run-instances --image-id ${amiImage} --count 1 --instance-type t2.medium --subnet-id ${GAUGE_SUBNET} --tag-specifications 'ResourceType=instance,Tags=[${GAUGE_TAGS}]' > aws_instance.json
      find . -name aws_instance.json | xargs cat
    """
    def aws_instance_json = readFile( 'aws_instance.json' )
    def aws_instance_object = readJSON text: aws_instance_json
    env.GEEP_IP = aws_instance_object.Instances[0].NetworkInterfaces[0].PrivateIpAddress
    env.GEEP_ID = aws_instance_object.Instances[0].InstanceId
    echo aws_instance_json
    echo "new instance IP address: ${GEEP_IP}"
    echo "new instance ID: ${GEEP_ID}"
  }
}

def prepInstance(def instanceIP,def platform) {
  echo "Executing prep against: ${instanceIP}"
  sh './gradlew -b infrastructure/build.gradle fetchRoles'

  sshagent (credentials: ['jenkins2-centos']) {
    sh """
      ansible all -i ansible/inventory/geep.ini -m wait_for_connection
      echo 'Setting hostname on new instance...'
      taws hostname ${instanceIP} -u centos -n tstserver.vm
    """

    dir('infrastructure/playbooks') {
      sh """
        ansible-playbook -e "geep_version=\$(git describe).${platform}-SNAPSHOT" -e "{geep_firewall_port_80: true}" -i ${instanceIP}, geep-install-locally-on-remote.yml
      """
    }
  }
}

def runFuncTests(def instanceIP, def platform) {
  echo 'Running functional test...'
  sh """
    gauge version
    mkdir -p automated_fusion/test_functional/env/test
    printf "automated_fusion_url = http://${instanceIP}/automatedfusion/\n" > automated_fusion/test_functional/env/test/user.properties
    sed "s/localhost ansible_connection=local/${instanceIP} ansible_user=centos/g" -i automated_fusion/test_functional/ansible/inventory/hosts.ini
    cat automated_fusion/test_functional/ansible/inventory/hosts.ini
  """

  sshagent (credentials: ['jenkins2-centos']) {
    sh './gradlew functionalTestAutomatedFusion -Penv_test_folder=test'
  }

  publishHTML (target: [
    allowMissing: false,
    alwaysLinkToLastBuild: false,
    keepAll: true,
    reportDir: 'automated_fusion/test_functional/reports/html-report/',
    reportFiles: 'index.html',
    reportName: "Gauge Report ${platform}"
  ])

  sh """
    mkdir -p panoramic_fusion/test_functional/env/test
    printf "panoramic_fusion_url = http://${instanceIP}/panoramicfusion/\n" > panoramic_fusion/test_functional/env/test/user.properties
    sed "s/^localhost ansible_connection=local/${instanceIP} ansible_user=centos/g" -i panoramic_fusion/test_functional/ansible/inventory/hosts.ini
  """

  sshagent (credentials: ['jenkins2-centos']) {
   sh "./gradlew functionalTestPanoramicFusion -Penv_test_folder=test -Ptest_platform=${platform}"
  }

  publishHTML (target: [
   allowMissing: false,
   alwaysLinkToLastBuild: false,
   keepAll: true,
   reportDir: 'panoramic_fusion/test_functional/reports/html-report/',
   reportFiles: 'index.html',
   reportName: "Gauge Report ${platform}"
  ])
}

def emailAlert() {
  if ( env.BLAST_EMAIL.toBoolean() ) {
    mail to:"geedev@t-sciences.com", subject:"${currentBuild.fullDisplayName} - Failed!", body: "Failure at ${env.BUILD_URL}!"
  }
}
