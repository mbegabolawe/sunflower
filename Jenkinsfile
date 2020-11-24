import com.sun.org.apache.xerces.internal.parsers.XMLParser

class Constants {
    static final String MASTER_BRANCH = 'master'
    static final String DEVELOP_BRANCH = 'main'
    static final String STAGING_BRANCH = 'release-'

    static final String FLAVOUR_DEVELOPMENT = 'Development'
    static final String FLAVOUR_PRODUCTION = 'Production'
    static final String FLAVOUR_STAGING = 'Staging'

    static final String DEBUG_BUILD_TYPE = 'Debug'
    static final String RELEASE_BUILD_TYPE = 'Release'
}

def initialiseBuildEnv() {
    env.BUILD_TYPE = Constants.RELEASE_BUILD_TYPE
    if (env.BRANCH_NAME == Constants.DEVELOP_BRANCH || env.CHANGE_TARGET == Constants.DEVELOP_BRANCH) { //main = develop
        env.BUILD_FLAVOUR = Constants.FLAVOUR_DEVELOPMENT
        env.BUILD_TYPE = Constants.DEBUG_BUILD_TYPE
    } else if (env.BRANCH_NAME == Constants.MASTER_BRANCH || env.CHANGE_TARGET == Constants.MASTER_BRANCH) {
        env.BUILD_FLAVOUR = Constants.FLAVOUR_PRODUCTION
    } else {
        //do staging type
        env.BUILD_FLAVOUR = Constants.FLAVOUR_STAGING
    }
}

def getApkFileName(version) {
    switch (env.BUILD_FLAVOUR) {
        case Constants.FLAVOUR_PRODUCTION:
            env.FILE_NAME = "${appName}-v$version.apk"
            break
        case Constants.FLAVOUR_STAGING:
            env.FILE_NAME = "${appName}-v$version-${BUILD_NUMBER}-uat.apk"
            break
        default:
            env.FILE_NAME = "${appName}-v$version-SNAPSHOT_${BUILD_NUMBER}.apk"
            break
    }
}

pipeline {
    agent { dockerfile true }
    environment {
        appName = 'Sunflower'
        FILE_NAME = "1.10"
    }
    stages {
        stage("Initialise") {
            steps {
                initialiseBuildEnv()
                echo "Branch to build is: ${env.BRANCH_NAME}"
                echo "Change branch is: ${env.CHANGE_BRANCH}"
                echo "Target branch is: ${env.CHANGE_TARGET}"
                echo "Build number: ${env.BUILD_NUMBER}"
                echo "Stage Initialise - ${currentBuild.result}"
            }
        }

        stage("Test") {
            steps {
                echo 'Testing'
                echo "Stage Testing - ${currentBuild.result}"
//                sh "./gradlew test${env.BUILD_FLAVOUR}${env.BUILD_TYPE}UnitTest"
//                sh "./gradlew test${env.BUILD_FLAVOUR}${env.BUILD_TYPE}UnitTestCoverage"
            }
        }

        stage("Build") {
            steps {
//                echo 'Building apk'
                echo "Successful build ${currentBuild.fullDisplayName}"
                echo "Url:  ${currentBuild.absoluteUrl}"
                echo "Workspace: ${env.WORKSPACE}"
                echo "DIR: ${currentBuild.fullProjectName}"
                echo "Stage Build - ${currentBuild.result}"

//                script {
//                    def d = [versionName: 'unversioned', versionCode: '1']
//                    echo d.toString()
//                    // Read properties from file (Right now we only keep versionName and VersionCode there)
//                    HashMap<String, Object> props = readProperties defaults: d, file: 'gradle.properties'
//
//                    echo "versionName: ${props.versionName}"
//                    echo "versionCode: ${props.versionCode}"
//
//                    try {
//                        props.versionName = '0.1.7'
//                        props.versionCode = '2'
//                        echo("Parameters changed")
//
//                        echo "versionName: ${props.versionName}"
//                        echo "versionCode: ${props.versionCode}"
//                        getApkFileName(props.versionName)
//                        env.APP_VERSION = props.versionName
//                        echo env.FILE_NAME
//
//                        env.COMMON_BUILD_ARGS = "-PversionName=${props.versionName} -PversionCode=${props.versionCode}"
//
//                        sh "./gradlew clean assemble${BUILD_FLAVOUR}${BUILD_TYPE} ${env.COMMON_BUILD_ARGS}"
//
//                    } catch (Exception e) {
//                        echo "User input timed out or cancelled, continue with default values"
//                    }
//                }
            }
        }

        stage("post-actions") {
            steps {
                echo 'Post-actions'
                echo 'new step'
                echo "Stage Post-actions - ${currentBuild.result}"

//                sh "./gradlew testDevelopmentDebugUnitTestCoverage"
                script {
//                    //Get TestCoverage summary for posting
                    def unitTestCoverageXML = readFile "${env.WORKSPACE}/app/build/reports/jacoco/test${env.BUILD_FLAVOUR}${env.BUILD_TYPE}UnitTestCoverage/test${env.BUILD_FLAVOUR}${env.BUILD_TYPE}UnitTestCoverage.xml"
                    def parser = new XmlParser()
                    parser.setFeature("http://apache.org/xml/features/disallow-doctype-decl", false)
                    parser.setFeature("http://apache.org/xml/features/nonvalidating/load-external-dtd", false)

                    def coverageReport = ""
                    def coveragePercentage = ""
                    def report = parser.parseText(unitTestCoverageXML)
                    report['counter'].each
                            {
                                coverageReport += "${it['@type']} Missed: ${it['@missed']} Covered: ${it['@covered']} \n"
                            }

                    double missedInt = report['counter'][0]['@missed']
                    double coveredInt = report['counter'][0]['@covered']
                    if (missedInt > 0 && coveredInt > 0) {
                        double percent = coveredInt / missedInt * 100
                        coveragePercentage = "Summary Test coverage percentage is: ${percent.round(1)}%"
                    } else {
                        coveragePercentage = "0%"
                    }

                    env.COVERAGE_REPORT = coverageReport
                    env.COVERAGE_PERCENTAGE = coveragePercentage
                }
            }

            //Consider merging branch when doing merge to master
        }
    }

    post {
        always {
            echo "TODO Send out comms to Slack"
            echo "Stage Finally - ${currentBuild.result}"

        }

        success {
            script {
                if (env.CHANGE_ID) {
                    pullRequest.comment("Test Coverage was [${env.COVERAGE_PERCENTAGE}] \n\n" +
                            "----- Coverage Report ---- \n " +
                            env.COVERAGE_REPORT)
                    pullRequest.removeLabel('CI is taking a peak')
                    pullRequest.addLabel('Passed')
                }
            }
        }

        failure {
            script {
                if (env.CHANGE_ID) {
                    pullRequest.comment('Build failure')
                    pullRequest.addLabel('Failed')
                }
            }
        }
    }
}
