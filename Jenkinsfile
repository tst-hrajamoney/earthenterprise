pipeline {
  agent { label  "centos7base" }

  options {
        disableConcurrentBuilds()
    }

  stages {
    stage('Prep') {
      steps {
        setEnv()
      }
    }
    stage('Fetch Source') {
      steps {
        sh """
          sudo yum install -y epel-release
          sudo yum install -y https://centos7.iuscommunity.org/ius-release.rpm || true
          sudo yum remove -y git || true
          sudo yum install -y git2u python-pip git-lfs
          sudo pip install gitpython
        """
        fetchGeoPlatform()
        fetchGEE()
        tagBuild()
      }
    }
    // stage('Portable Tag') {
    //   steps {
    //     sh './gradlew config'
    //     sh './gradlew createBuildTag'
    //     sh 'git describe > described.txt'
    //   }
    // }
    stage('Build Docs') {
      agent { label "opengee-build-el7" }
      steps {
        fetchGeoPlatform()
        buildDocs()
        buildDocsPortable()
      }
    }

    stage('Spawn Builds') {
      parallel {
        stage('CentOS 7') {
          agent { label "opengee-build-el7" }
          stages {
            stage('Fetch GeoPlatform') {
              steps {
                fetchGeoPlatform()
              }
            }

            stage('MrSID') {
              steps {
                downloadMrSID()
              }
            }

            stage('Fetch GEE') {
              steps {
                fetchGEE()
              }
            }

            stage('Build') {
              steps {
                cleanBuild()
              }
            }

            stage('Unit Test') {
              when {
                  expression { env.DO_TEST }
              }
              steps {
                sh """
                  ./gradlew testGee -Poslabel=centos7
                  echo "DEBUG: Find test results"
                  find . -name Output.xml
                """
              }
              post {
                always {
                  junit "earthenterprise/earth_enterprise/src/**/bin/tests/Output.xml"
                }
              }
            }

            stage('Package') {
              steps {
                doPackage("centos7_stash")
              }
            }

            stage('Prep Portable') {
              steps {
                prepPortable()
              }
            }

            stage('Build Portable') {
              steps {
                buildPortable()
              }
            }

            stage('Archive') {
              steps {
                tryArchive()
              }
            }

            stage('Publish') {
              steps {
                tryPublish('true')
              }
            }

            stage('Functional Test') {
              when {
                  expression { env.DO_TEST }
              }
              agent { label 'centos7-gauge'}
              steps {
                fetchGeoPlatform()
                acquireInstance("${AMI_IMAGE_7}")
                script {
                  env.EL7_GEE_ID = "${env.GEE_ID}"
                  env.EL7_GEE_IP = "${env.GEE_IP}"
                }
                prepInstance("${EL7_GEE_IP}","centos7_stash")
                runFuncTests("${EL7_GEE_IP}")
              }
              post {
                cleanup {
                  sh "ls -l"
                  echo "terminating VM..."
                  sh "aws ec2 terminate-instances --instance-ids ${EL7_GEE_ID}"
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

        stage('CentOS 6') {
          agent { label "opengee-build-el6" }
          stages {
            stage('Fetch GeoPlatform') {
              steps {
                fetchGeoPlatform()
              }
            }

            stage('MrSID') {
              steps {
                downloadMrSID()
              }
            }

            stage('Fetch GEE') {
              steps {
                fetchGEE()
              }
            }

            stage('Build') {
              steps {
                cleanBuild()
              }
            }

            stage('Unit Test') {
              when {
                expression { env.DO_TEST }
              }
              steps {
                sh """
                  ./gradlew testGee -Poslabel=centos6
                  echo "DEBUG: Find test results"
                  find . -name Output.xml
                """
              }
              post {
                always {
                  junit "earthenterprise/earth_enterprise/src/**/bin/tests/Output.xml"
                }
              }
            }

            stage('Package') {
              steps {
                doPackage("centos6_stash")
              }
            }

            stage('Prep Portable') {
              steps {
                prepPortable()
              }
            }

            stage('Build Portable') {
              steps {
                buildPortable()
              }
            }

            stage('Archive') {
              steps {
                tryArchive()
              }
            }

            stage('Publish') {
              steps {
                tryPublish('true')
              }
            }

            stage('Functional Test') {
              when {
                expression { env.DO_TEST }
              }
              agent { label 'centos7-gauge'}
              steps {
                fetchGeoPlatform()
                acquireInstance("${AMI_IMAGE_6}")
                script {
                  env.EL6_GEE_ID = "${env.GEE_ID}"
                  env.EL6_GEE_IP = "${env.GEE_IP}"
                }
                prepInstance("${EL6_GEE_IP}","centos6_stash")
                runFuncTests("${EL6_GEE_IP}")
              }
              post {
                cleanup {
                  sh "ls -l"
                  echo "terminating VM..."
                  sh "aws ec2 terminate-instances --instance-ids ${EL6_GEE_ID}"
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

        // No MrSID parallel
        stage('CentOS 7 No MrSID') {
          when {
            expression { env.NO_MRSID == 'true' }
          }
          agent { label "opengee-build-el7" }
          stages {
            stage('Fetch GeoPlatform') {
              steps {
                fetchGeoPlatform()
              }
            }

            stage('Remove MrSID') {
              steps {
                removeMrSID()
              }
            }

            stage('Fetch GEE') {
              steps {
                fetchGEE()
              }
            }

            stage('Build') {
              steps {
                cleanBuild()
              }
            }

            stage('Unit Test') {
              when {
                expression { env.DO_TEST }
              }
              steps {
                sh """
                  ./gradlew testGee -Poslabel=centos7
                  echo "DEBUG: Find test results"
                  find . -name Output.xml
                """
              }
              post {
                always {
                  junit "earthenterprise/earth_enterprise/src/**/bin/tests/Output.xml"
                }
              }
            }

            stage('Package') {
              steps {
                doPackage("centos7NoMrSID_stash")
              }
            }

            stage('Prep Portable') {
              steps {
                prepPortable()
              }
            }

            stage('Build Portable') {
              steps {
                buildPortable()
              }
            }

            stage('Archive') {
              steps {
                tryArchive()
              }
            }

            stage('Publish') {
              steps {
                tryPublish('false')
              }
            }

            stage('Functional Test') {
              when {
                expression { env.DO_TEST }
              }
              agent { label 'centos7-gauge'}
              steps {
                fetchGeoPlatform()
                acquireInstance("${AMI_IMAGE_7}")
                script {
                  env.EL7_GEE_ID_NO_MRSID = "${env.GEE_ID}"
                  env.EL7_GEE_IP_NO_MRSID = "${env.GEE_IP}"
                }
                prepInstance("${EL7_GEE_IP_NO_MRSID}","centos7NoMrSID_stash")
                runFuncTests("${EL7_GEE_IP_NO_MRSID}")
              }
              post {
                cleanup {
                  sh "ls -l"
                  echo "terminating VM..."
                  sh "aws ec2 terminate-instances --instance-ids ${EL7_GEE_ID_NO_MRSID}"
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

        stage('CentOS 6 No MrSID') {
          when {
            expression { env.NO_MRSID == 'true' }
          }
          agent { label "opengee-build-el6" }
          stages {
            stage('Fetch GeoPlatform') {
              steps {
                fetchGeoPlatform()
              }
            }

            stage('Remove MrSID') {
              steps {
                removeMrSID()
              }
            }

            stage('Fetch GEE') {
              steps {
                fetchGEE()
              }
            }

            stage('Build') {
              steps {
                cleanBuild()
              }
            }

            stage('Unit Test') {
              when {
                expression { env.DO_TEST }
              }
              steps {
                sh """
                  ./gradlew testGee -Poslabel=centos6
                  echo "DEBUG: Find test results"
                  find . -name Output.xml
                """
              }
              post {
                always {
                  junit "earthenterprise/earth_enterprise/src/**/bin/tests/Output.xml"
                }
              }
            }

            stage('Package') {
              steps {
                doPackage("centos6NoMrSID_stash")
              }
            }

            stage('Prep Portable') {
              steps {
                prepPortable()
              }
            }

            stage('Build Portable') {
              steps {
                buildPortable()
              }
            }

            stage('Archive') {
              steps {
                tryArchive()
              }
            }

            stage('Publish') {
              steps {
                tryPublish('false')
              }
            }

            stage('Functional Test') {
              when {
                expression { env.DO_TEST }
              }
              agent { label 'centos7-gauge'}
              steps {
                fetchGeoPlatform()
                acquireInstance("${AMI_IMAGE_6}")
                script {
                  env.EL6_GEE_ID_NO_MRSID = "${env.GEE_ID}"
                  env.EL6_GEE_IP_NO_MRSID = "${env.GEE_IP}"
                }
                prepInstance("${EL6_GEE_IP_NO_MRSID}","centos6NoMrSID_stash")
                runFuncTests("${EL6_GEE_IP_NO_MRSID}")
              }
              post {
                cleanup {
                  sh "ls -l"
                  echo "terminating VM..."
                  sh "aws ec2 terminate-instances --instance-ids ${EL6_GEE_ID_NO_MRSID}"
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
        // End No MrSID

        stage("windows") {
          agent { label 'windows' }
          stages {
            stage('Fetch GeoPlatform') {
              steps {
                git branch: 'dev',
                  credentialsId: '2f281863-af0d-492d-a758-81acc26f5aa9',
                  url: 'https://bitbucket.org/t-sciences/geoplatform.git'
                bat """
                  dir
                  MOVE geoplatform\\* .
                  dir
                """
              }
            }
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

def moveRoot(def dir) {
  makeDir("$dir")
  sh """
    for i in `ls | grep -v earthenterprise`; do
      mv \$i earthenterprise/
    done || true
    mv .git* earthenterprise/ || true
  """
}

def makeDir(def dir) {
  dirCheck = sh (script: "test -d ${dir} && echo '1' || echo '0' ", returnStdout: true).trim()
  if (dirCheck=='1') {
    echo "Cleaning contents of following directory: $dir"
    sh "rm -rf $dir"
  }
  sh "mkdir -p $dir"
}

def setEnv() {
  sh "env"
  // Set defaults
  env.ANSIBLE_INVENTORY_VARS = 'ansible_user=centos ansible_become=yes geep_firewall=true'
  env.BUILD_TYPE = "$BRANCH_NAME"
  env.MRSIDTGZ = "MrSID_DSDK-9.5.4.4703.tar.gz"
  env.DO_BUILD = 'true'
  env.OPENGEE_REPO = 'tst-hrajamoney/earthenterprise'
  env.OPENGEE_REF = "${GIT_BRANCH}"
  env.OPENGEE_REF_CHECKOUT = "${GIT_BRANCH}"
  env.OPENGEE_LABEL = 'anon'
  env.PR = 'false'
  env.DO_TEST = 'true'
  env.DO_ARCHIVE = 'false'
  env.DO_PUBLISH = 'true'
  env.BLAST_EMAIL = 'false'
  env.WITH_MRSID = 'true'
  env.NO_MRSID = 'false'
  env.SCONS_CPUS = '8'
  env.ANSIBLE_HOST_KEY_CHECKING = 'False'
  env.AMI_IMAGE_6 = "ami-0c8bf051b55247400"
  env.AMI_IMAGE_7 = "ami-058f5ca0bea34f985"
  env.FUNCTIONAL_TEST_AGENT = 'centos7-gauge'
  env.GAUGE_SUBNET = 'subnet-75664c01'
  env.GAUGE_TAGS = '{Key=pipeline,Value=gauge},{Key=Project,Value=DevTesting}'
  // Override defaults based on google/earthenterprise branch/tag/PR
  if ( BUILD_TYPE.startsWith("PR-") ) {
    env.DO_ARCHIVE = 'false'
    env.PR = 'true'
    env.OPENGEE_REF_CHECKOUT = "master"
    env.PR_NUM = sh (script: "echo ${BRANCH_NAME} | cut -d '-' -f 2", returnStdout: true).trim()
  }
  if ( BUILD_TYPE.startsWith("release_") ) {
    env.DO_ARCHIVE = 'false'
    env.NO_MRSID = 'true'
  }
  if ( BUILD_TYPE.equals("master") ) {
    env.DO_ARCHIVE = 'false'
    env.NO_MRSID = 'true'
    env.BLAST_EMAIL = 'true'
  }
  if ( BUILD_TYPE.equals("dev") ) {
    env.BLAST_EMAIL = 'true'
  }
  if ( buildingTag() ) {
    env.DO_ARCHIVE = 'true'
    env.NO_MRSID = 'true'
  }
  if ( JENKINS_URL.equals("https://ci-gee-product.t-sciences.com/") ) {
    env.GAUGE_SUBNET = 'subnet-01b2077a7097cddd4'
    env.GAUGE_TAGS = '{Key=pipeline,Value=gauge},{Key=Activity,Value=GEEDev},{Key=Environment,Value=Dev},{Key=Owner,Value=jenkins},{Key=Team,Value=DevOps}'
  }
  sh "env"
}

def fetchGeoPlatform() {
  withCredentials([file(credentialsId: 'tst-jenkins2', variable: 'sshKey')]) {
    sh """
      mkdir ~/.ssh/ || true
      ssh-keyscan localhost >> ~/.ssh/known_hosts
      ssh-keyscan github.com >> ~/.ssh/known_hosts
      ssh-keyscan bitbucket.com >> ~/.ssh/known_hosts
      ssh-keyscan bitbucket.org >> ~/.ssh/known_hosts
      cp -pf $sshKey ~/.ssh/id_rsa || true
      chmod 400 ~/.ssh/id_rsa
    """
  }
  moveRoot('earthenterprise')
  gitCheckout('geoplatform','dev')
  sh "mv -f geoplatform/* . && mv geoplatform/.git* . && rmdir geoplatform"
}

def gitCheckout(def repo, def branch) {
  sh """
    rm -rf $repo
    git clone "git@bitbucket.org:t-sciences/${repo}.git"
    cd $repo
    git checkout $branch
  """
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

def removeMrSID() {
  sh """
    # Remove existing MrSID library and refresh linker library config
    sudo rm -f /etc/ld.so.conf.d/MrSID.conf
    sudo ldconfig
    # Remove directory
    sudo rm -Rf /opt/MrSID_DSDK-*
  """
}

def downloadMrSID() {
  sh """
    sudo scripts/get-MrSID-environment.sh
  """
}

def fetchGEE() {
  // super clean
  dir = 'earthenterprise'
  dirCheck = sh (script: "test -d ${dir} && echo '1' || echo '0' ", returnStdout: true).trim()
  if (dirCheck=='1') {
    echo "Cleaning contents of following directory: $dir"
    sh "rm -rf $dir"
  }

  dir('earthenterprise') {
    echo 'Pulling open code'
    sh """
      git clone -b ${OPENGEE_REF_CHECKOUT} https://github.com/tst-hrajamoney/earthenterprise.git
      mv earthenterprise/* .
      mv earthenterprise/.git* .
      mv earthenterprise/.travis* .
      rmdir earthenterprise/
    """

    if ( env.PR.toBoolean() ) {
      // Fetch and checkout PR branch merged with the target
      sh """
        git fetch origin pull/${env.PR_NUM}/merge:PR-${env.PR_NUM}-merge
        git checkout PR-${env.PR_NUM}-merge
      """
    }

    sh """
      git lfs pull
      git --version
      git status
      git describe
      git describe --first-parent
      // git describe --tags --match '[0-9]*\\.[0-9]*\\.[0-9]*\\-*' --first-parent
      ./earth_enterprise/src/scons/getversion.py -l
      ./earth_enterprise/src/scons/getversion.py -l > build_tag.txt
      git rev-parse --short HEAD > hash.txt
    """
  }
}

def tagBuild() {
  def build_tag = ''
  build_tag = readFile('earthenterprise/build_tag.txt').trim()
  currentBuild.displayName = "${build_tag} + " + readFile('earthenterprise/hash.txt').trim()
  sh "git tag -a ${build_tag} -m 'build tag for build ${build_tag}'"
}

def cleanBuild() {
  sh """
    ././gradlew clean
    ./gradlew cacheThirdParty -Pnum_jobs_allowed=${SCONS_CPUS}
    ./gradlew publishThirdPartyCache
    ./gradlew assembleGee -Pnum_jobs_allowed=${SCONS_CPUS}
  """
}

def doPackage(def stashName) {
  sh "./gradlew stageGee"
  dir('earthenterprise/earth_enterprise/rpms') {
    sh "./gradlew clean openGeeRpms"
  }
  stash name: "${stashName}", includes: '**/build/distributions/opengee*.rpm'
}

def prepPortable() {
  dir('portable') {
    ansiblePlaybook(
      playbook: 'ansible/pre-build.yml',
      inventoryContent: "localhost,",
      extras: "-c local"
    )
  }
}

def buildPortable() {
  dir('earthenterprise/earth_enterprise/src/portableserver') {
    sh './build.py --clean'
    sh './build.py'
    sh 'cp build/portableserver-*.tar.gz build/portableserver-linux.tgz'
  }
}

def tryArchive() {
  if ( env.DO_ARCHIVE.toBoolean() ) {
    archive "earthenterprise/earth_enterprise/rpms/build/distributions/opengee*.rpm"
  }
}

def tryPublish(def withMrSid) {
  if ( env.DO_PUBLISH.toBoolean() ) {
    env.MR_SID_BUILD = withMrSid
    sh 'tree -L 3'
    sh './gradlew -b publish-rpms.gradle publish -PopengeeRepo=${OPENGEE_REPO} -PopengeeRef=${OPENGEE_REF} -PopengeeLabel=${OPENGEE_LABEL} -PopengeeMrSid=${MR_SID_BUILD}'
  }
}

def acquireInstance(def amiImage) {
  echo 'creating VM...'
  script {
    sh "aws ec2 run-instances --image-id ${amiImage} --count 1 --instance-type t2.medium --subnet-id ${GAUGE_SUBNET} --tag-specifications \'ResourceType=instance,Tags=[${GAUGE_TAGS}]\' > aws_instance.json"
    def aws_instance_json = readFile(file: 'aws_instance.json')
    def aws_instance_object = readJSON text: aws_instance_json
    env.GEE_IP = aws_instance_object.Instances[0].NetworkInterfaces[0].PrivateIpAddress
    env.GEE_ID = aws_instance_object.Instances[0].InstanceId
    echo aws_instance_json
    echo "new instance IP address: ${GEE_IP}"
    echo "new instance ID: ${GEE_ID}"
  }
}

def prepInstance(def instanceIP, def stashName) {
  echo "Executing prep against: ${instanceIP}"
  unstash "${stashName}"

  sshagent (credentials: ['jenkins2-centos']) {
    script {
      echo "ANSIBLE_INVENTORY_VARS:"
      echo "${ANSIBLE_INVENTORY_VARS}"
      sh """
        printf "[geep-single]\n${instanceIP} ${ANSIBLE_INVENTORY_VARS}" > installer/ansible/inventory/geep.ini
        # wait for VM to be ssh-ready
        ansible all -i installer/ansible/inventory/geep.ini -m wait_for_connection
      """
    }
  }

  if ( env.DO_TEST.toBoolean() && env.WITH_MRSID.toBoolean() ) {
    echo 'Download MrSID'
    sh 'wget http://nexus.t-sciences.local:8081/nexus/service/local/repositories/thirdparty/content/com/lizardtech/MrSID_DSDK/9.5.4.4703/${MRSIDTGZ} -O /tmp/${MRSIDTGZ}'

    sshagent (credentials: ['jenkins2-centos']) {
      script {
        echo "Install MrSID"
        sh """
          scp -o StrictHostKeyChecking=no /tmp/${MRSIDTGZ} centos@${instanceIP}:/tmp

          ssh -o StrictHostKeyChecking=no centos@${instanceIP} "
            sudo tar -xzf /tmp/${MRSIDTGZ} -C /opt
            echo /opt/MrSID_DSDK-9.5.4.4703-rhel6.x86-64.gcc482/Raster_DSDK/lib | sudo tee --append /etc/ld.so.conf.d/MrSID.conf
            sudo /sbin/ldconfig
          "
        """
      }
    }
  }

  unstash "${stashName}"
  sshagent (credentials: ['jenkins2-centos']) {
    script {
      echo 'Download RPMs'
      sh "scp -o StrictHostKeyChecking=no ./earthenterprise/earth_enterprise/rpms/build/distributions/opengee*.rpm centos@${instanceIP}: "
      echo "Install RPMs"
      sh "ssh -o StrictHostKeyChecking=no  centos@${instanceIP} sudo yum install -y --nogpgcheck opengee*.rpm"
    }
  }
}

def runFuncTests(def instanceIP) {
  sshagent (credentials: ['jenkins2-centos']) {
    dir('test') {
      script {
        // create hosts.ini
        sh """
          printf "[geep-single]\n${instanceIP} ${ANSIBLE_INVENTORY_VARS} ansible_user=centos ansible_ssh_pass=Sp@rt@ns12  geep_server_url=http://${instanceIP}" > hosts.ini
          echo "Run Functional Tests"
          ./run-all.sh
        """
      }
    }
  }
}

def emailAlert() {
  if ( env.BLAST_EMAIL.toBoolean() ) {
    mail to:"geedev@t-sciences.com", subject:"${currentBuild.fullDisplayName} - Failed!", body: "Failure at ${env.BUILD_URL}!"
  }
}
