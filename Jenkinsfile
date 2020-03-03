pipeline {
  agent any
  environment {
    PACKAGE_NAME = 'interop'
    ROS_WORKSPACE = "${WORKSPACE}_ws"
  }
  stages {
    stage('Setup') {
      steps {
        sh 'printenv'
        sh """
          mkdir -p ${ROS_WORKSPACE}/src
          cp -R . ${ROS_WORKSPACE}/src/${PACKAGE_NAME}
        """
      }
    }
    stage('Build') {
      steps {
        dir(path: "${ROS_WORKSPACE}") {
          sh '''
            . /opt/ros/melodic/setup.sh
            catkin build --no-status --verbose
          '''
        }
        
      }
    }
    stage('Test') {
      steps {
        dir(path: "${ROS_WORKSPACE}") {
          sh '''
            . /opt/ros/melodic/setup.sh
            . devel/setup.sh
            catkin run_tests
            catkin_test_results build --verbose
          '''
        }
      }
    }
    stage('Sanity Check') {
      parallel {
        stage('Lint') {
          steps {
            sh '''
              . /opt/ros/melodic/setup.sh
              catkin lint --explain -W2 --strict .
            '''
          }
        }
        stage('Format') {
          steps {
            sh '[ -z "$(yapf --recursive --parallel --diff .)" ]'
          }
        }
      }
    }
  }
  post {
    always {
      dir(path: "${ROS_WORKSPACE}") {
        archiveArtifacts(artifacts: "logs/**/*.log", fingerprint: true)
        junit "build/**/test_results/**/*.xml"
      }
      sh "rm -rf ${ROS_WORKSPACE}"
    }
  }
}
