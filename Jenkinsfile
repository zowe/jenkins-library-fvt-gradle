#!groovy

/**
 * This program and the accompanying materials are made available under the terms of the
 * Eclipse Public License v2.0 which accompanies this distribution, and is available at
 * https://www.eclipse.org/legal/epl-v20.html
 *
 * SPDX-License-Identifier: EPL-2.0
 *
 * Copyright IBM Corporation 2018, 2019
 */


/**
 * These 2 build parameters are only required for running integration test
 */
def opts = []
// define custom build parameters
def customParameters = []
customParameters.push(booleanParam(
  name: 'FETCH_PARAMETER_ONLY',
  description: 'By default, the pipeline will exit just for fetching parameters.',
  defaultValue: true
))
customParameters.push(string(
  name: 'LIBRARY_BRANCH',
  description: 'Jenkins library branch to test',
  defaultValue: '',
  trim: true
))
opts.push(parameters(customParameters))

// set build properties
properties(opts)

/**
 * This check is only required for running integration test
 */
if (params.FETCH_PARAMETER_ONLY) {
    currentBuild.result = 'NOT_BUILT'
    error "Prematurely exit after fetching parameters."
}

node('ibm-jenkins-slave-nvm') {
  /**
   * This section is only required for running integration test.
   *
   * In real consumption of library, we should use default library branch. For
   * example:
   *
   * def lib = library("jenkins-library").org.zowe.jenkins_shared_library
   */
  def branch = 'master'

  if (params.LIBRARY_BRANCH) {
      branch = params.LIBRARY_BRANCH
  } else if (env.CHANGE_BRANCH) {
      branch = env.CHANGE_BRANCH
  } else if (env.BRANCH_NAME) {
      branch = env.BRANCH_NAME
  }

  echo "Jenkins library branch $branch will be used to build."
  def lib = library("jenkins-library@$branch").org.zowe.jenkins_shared_library

  def pipeline = lib.pipelines.gradle.GradlePipeline.new(this)

  /**
   * These 2 build parameters are only required for running integration test
   */
  pipeline.addBuildParameters(customParameters)

  pipeline.admins.add("jackjia")

  pipeline.setup(
    packageName: 'org.zowe.jenkins-library-test.gradle',
    github: [
      email                      : 'zowe.robot@gmail.com',
      usernamePasswordCredential : 'zowe-robot-github',
    ],
    artifactory: [
      url                        : 'https://gizaartifactory.jfrog.io/gizaartifactory',
      usernamePasswordCredential : 'GizaArtifactory',
    ]
  )

  // we have build stage
  pipeline.build()

  pipeline.test(
    name          : 'Unit',
    junit         : [
      allowEmptyResults : true,
      testResults       : '**/test-results/**/*.xml'
    ],
    htmlReports   : [
      [dir: "build/reports/tests/test", files: "index.html", name: "Report: Unit Test"],
    ],
  )

  // we don't have sonarscan defined for this test project
  // pipeline.sonarScan()

  // how we packaging jars/zips
  pipeline.packaging(
      name: 'jenkins-library-fvt-gradle',
      operation: {
          sh './gradlew jar'
      }
  )

  // define we need publish stage
  pipeline.publish(
    artifacts: [
      'build/libs/jenkins-library-fvt-gradle-*.jar'
    ]
  )

  // define we need release stage
  pipeline.release()

  pipeline.end()
}
