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
    stage('prepare') {
      steps {
        deleteDir()
        writeYaml(file: 'platforms.yaml', data: readYaml(text: """
platform: "linux"
stages:
  windows:
    mage:
      - "mage build unitTest"
    platforms:
      - "windows-1"
      - "windows-a"
      - "windows-b"
      - "windows-c"
      - "windows-d"
      - "windows-e"
      - "windows-f"
      - "windows-g"
      - "windows-h"
  linux:
    mage:
      - "mage build unitTest"
    platforms:
      - "windows-1"
      - "windows-a"
      - "windows-b"
      - "windows-c"
      - "windows-d"
      - "windows-e"
      - "windows-f"
      - "windows-g"
      - "windows-h"
  arm:
    mage:
      - "mage build unitTest"
    platforms:
      - "windows-1"
      - "windows-a"
      - "windows-b"
      - "windows-c"
      - "windows-d"
      - "windows-e"
      - "windows-f"
      - "windows-g"
      - "windows-h"
  mac:
    mage:
      - "mage build unitTest"
    platforms:
      - "windows-1"
      - "windows-a"
      - "windows-b"
      - "windows-c"
      - "windows-d"
      - "windows-e"
      - "windows-f"
      - "windows-g"
      - "windows-h"
"""))
      }
    }
    stage('platforms') {
      steps {
        script {
          def ret = beatsStages(project: 'test', content: readYaml(file: 'platforms.yaml'), function: new RunCommand(steps: this))
          parallel(ret)
        }
      }
    }
    stage('foo') {
      steps {
        echo 'foo'
      }
    }
  }
}

class RunCommand extends co.elastic.beats.BeatsFunction {
  public RunCommand(Map args = [:]){
    super(args)
  }
  public run(Map args = [:]){
    if (args?.content?.mage) {
      steps.node('linux') {
        steps.withGithubNotify(context: "${args.context}") {
          steps.dir(args.project) {
            steps.echo "mage ${args.label}"
          }
        }
      }
    }
    if (args?.content?.make) {
      steps.node('linux') { 
        steps.withGithubNotify(context: "${args.context}") {
          steps.echo "make ${args.label}"
        }
      }
    }
  }
}
