pipeline {
  agent any
  stages {
    stage('Build') {
      agent {
        docker {
          image 'python:2-alpine'
        }

      }
      steps {
        sh 'python -m py_compile sources/add2vals.py sources/calc.py'
        stash(name: 'compiled-results', includes: 'sources/*.py*')
      }
    }

    stage('Test') {
      agent {
        docker {
          image 'qnib/pytest'
        }

      }
      post {
        always {
          junit 'test-reports/results.xml'
        }

      }
      steps {
        sh 'py.test --verbose --junit-xml test-reports/results.xml sources/test_calc.py'
      }
    }

    stage('Deliver') {
      agent any
      environment {
        VOLUME = '$(pwd)/sources:/src'
        IMAGE = 'cdrx/pyinstaller-linux:python2'
      }
      steps {
        dir(path: env.BUILD_ID) {
          unstash 'compiled-results'
          sh "docker run --rm -v ${VOLUME} ${IMAGE} 'pyinstaller --onefile  --name MyApp -F add2vals.py'"
        }

      }
            post {
        success {
          archiveArtifacts "${env.BUILD_ID}/sources/dist/MyApp"
          sh "docker run --rm -v ${VOLUME} ${IMAGE} 'rm -rf build dist'"
        }

      }
    }

  }
}