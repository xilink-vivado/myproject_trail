pipeline {
  agent any

  stages {
    stage('Create project') {
      steps {
        deleteDir() // clean up workspace
        checkout([$class: 'GitSCM', branches: [[name: '*/main']],
          doGenerateSubmoduleConfigurations: false,
          extensions: [[$class: 'SubmoduleOption',
            disableSubmodules: false,
            parentCredentials: false,
            recursiveSubmodules: true,
            reference: '',
            trackingSubmodules: true]],
          submoduleCfg: [],
          userRemoteConfigs: [[
            url: 'https://github.com/xilink-vivado/myproject_trail']]])
      sh 'cd scripts && vivado -mode batch -source recreate_prj.tcl'
      }
    }
    stage('Run simulation') {
      steps {
        sh 'cd vivado && vivado -mode batch -source run_simulation.tcl'
      }
    }
    stage('Run synthesis') {
      steps {
        sh 'cd vivado && vivado -mode batch -source run_synthesis.tcl'
      }
    }
    stage('Run implementation') {
      steps {
        sh 'cd vivado && vivado -mode batch -source run_implementation.tcl'
      }
    }
    stage('Generate bitstream') {
      steps {
        sh 'cd vivado && vivado -mode batch -source generate_bitstream.tcl'
      }
    }
    stage('Release bitfile') {
      steps {
        sh '''
        PROJ_NAME=example_blog1
        RELEASE_DIR=/usr/share/nginx/html/releases/
        BASE_NAME=$PROJ_NAME-`date +"%Y-%m-%d-%H-%H:%M"`
        BITFILE=$BASE_NAME.bit
        INFOFILE=$BASE_NAME.txt
        git log -n 1 --pretty=format:"%H" >> $INFOFILE
        echo -n " $PROJ_NAME " >> $INFOFILE
        git describe --all >> $INFOFILE
        echo "" >> $INFOFILE
        echo "Submodules:" >> $INFOFILE
        git submodule status >> $INFOFILE
        cp $INFOFILE $RELEASE_DIR
        cp vivado/example_blog1.runs/impl_1/example_blog1_wrapper.bit $RELEASE_DIR/$BITFILE
        '''
      }
    }
  }
  post {
    failure {
      emailext attachLog: true,
      body: '''Project name: $PROJECT_NAME
Build number: $BUILD_NUMBER
Build Status: $BUILD_STATUS
Build URL: $BUILD_URL''',
      recipientProviders: [culprits()],
      subject: 'Project \'$PROJECT_NAME\' is broken'
    }
  }
}
