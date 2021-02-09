// Licensed to Elasticsearch B.V. under one or more contributor
// license agreements. See the NOTICE file distributed with
// this work for additional information regarding copyright
// ownership. Elasticsearch B.V. licenses this file to you under
// the Apache License, Version 2.0 (the "License"); you may
// not use this file except in compliance with the License.
// You may obtain a copy of the License at
//
//     http://www.apache.org/licenses/LICENSE-2.0
//
// Unless required by applicable law or agreed to in writing,
// software distributed under the License is distributed on an
// "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
// KIND, either express or implied.  See the License for the
// specific language governing permissions and limitations
// under the License.

@Library('apm@git_base_commit') _

pipeline {
  agent { label 'linux && immutable' }
  environment {
    REPO = 'its-gitbase'
    OWNER = 'v1v'
    BASE_DIR = "src/github.com/${OWNER}/${env.REPO}"
    NOTIFY_TO = credentials('notify-to')
    JOB_GCS_BUCKET = credentials('gcs-bucket')
    JOB_GIT_CREDENTIALS = "f6c7695a-671e-4f4f-a331-acdce44ff9ba"
    PIPELINE_LOG_LEVEL='DEBUG'
  }
  options {
    timeout(time: 1, unit: 'HOURS')
    buildDiscarder(logRotator(numToKeepStr: '20', artifactNumToKeepStr: '20'))
    timestamps()
    ansiColor('xterm')
    disableResume()
    durabilityHint('PERFORMANCE_OPTIMIZED')
    rateLimitBuilds(throttle: [count: 60, durationName: 'hour', userBoost: true])
    quietPeriod(10)
  }
  stages {
    stage('Checkout scm') {
      options { skipDefaultCheckout() }
      steps {
        script {
          def testList = ["a", "b", "c", "d"]
          def branches = [:]
          for (int i = 0; i < 20 ; i++) {
            int index=i, branch = i+1
            stage ("branch_${branch}"){
              branches["branch_${branch}"] = {
                withGithubNotify(context: "branch_${branch}") {
                  sleep randomNumber(min: 5, max: 30)
                  node ('linux'){
                    sh "echo 'node: ${NODE_NAME},  index: ${index}, i: ${i}, testListVal: " + testList[index] + "'"
                  }
                }
              }
            }
          }
          parallel branches
        }
      }
    }
  }
}
