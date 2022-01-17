
// Cloning the github branch using setup in UI with clean before checkout

pipeline{
    agent any
    environment {
        currentDate = sh ( script: "date +'%Y-%m-%d'", returnStdout: true ).trim()
        currentYear = sh ( script: "date +'%Y'", returnStdout: true ).trim()
        staticStageResult = false
        holidayCheck = true
    }
    
    parameters{
        checkboxParameter( name: 'checkbox', format: 'YAML', pipelineSubmitContent: "CheckboxParameter: \n  - key: Static_Check\n    value: Static_Check\n  - key: QA\n    value: QA\n  - key: Unit_Test\n    value: Unit_Test\n")
        string( name: 'Success_Email', trim: true)
        string( name: 'Failure_Email', trim: true)
    }
    
    stages{
        // This stage runs every time
        stage(' Is the run required?'){
            steps{
                script{
                    response = httpRequest "https://calendarific.com/api/v2/holidays?api_key=90e19de04681bc27e9492b478df436f43aed8c4c&country=IN&year=${currentYear}"
                    def jsonObj = new groovy.json.JsonSlurperClassic().parseText(response.content)
                    def json = jsonObj.response.holidays
                    def lengthOfArray = json.results.size()
                    for (int i=0; i< lengthOfArray; ++i) {
                         if (json[i].date.iso == '${currentDate}'){
                             holidayCheck = false
                         }
                    }
                    
                }
            }
            
        }
        
        // This stage runs only if current day is not holiday
        stage('Build'){
            when{
                expression{
                    return holidayCheck
                }
            }
            
            steps{
                script{
                    def jsonObj = readJSON file: 'build.json'
                    dir ('builds'){
                        jsonObj.each { key, value ->
                            writeFile (file: "${value.Name}.txt" , text: "${value.Content}")
                        }
                    }
                    zip zipFile: 'builds.zip' , dir: 'builds'
                }
            }
        }
        
        stage('Quality...'){
            
            when{
                expression{
                    return holidayCheck
                }
            }
            parallel{
                
                stage('Checks'){
                    stages{
                        stage('Static_Check'){
                            // This stage runs only if current day is not holiday and Static_Check is enabled
                            when{
                                expression{
                                    return params.checkbox.indexOf('Static_Check') != -1 
                                }
                            }
                            steps{
                                script{
                                    
                                    folderCreation("${env.STAGE_NAME}")
                                    staticStageResult = true
                                }
                            }
                        }
                        // This stage runs only if current day is not holiday , QA is enabled and Static_check stage passed
                        stage('QA'){
                            when{
                                expression{
                                    return params.checkbox.indexOf('QA') != -1 && staticStageResult == true
                                }
                            }
                            steps{
                                script{
                                    folderCreation("${env.STAGE_NAME}")
                                }
                            }
                        }
                    }
                }
                // This stage runs only if current day is not holiday and Unit_Test is enabled
                stage('Unit_Test'){
                    when{
                        expression{
                            return params.checkbox.indexOf('Unit_Test') != -1 
                        }
                    }
                    steps{
                        script{
                            folderCreation("${env.STAGE_NAME}")
                        }
                    }
                }
                
            }
        }
        
        stage('Summary'){
            steps{
                script{
                    String [] str = params.checkbox.split(',')
                    for (String values : str)
                        println(values +" stage is executed and the "+values+".txt file is copied")
                }
            }
        }
    }
    
    post{
        success{
            script{
                echo params.Success_Email
                print params.Failure_Email
                emailext(
                    to: params.Success_Email,
                    subject: "${env.JOB_NAME} - ${env.BUILD_NUMBER} is ${env.BUILD_STATUS}",
                    body: "Please check ${env.BUILD_URL} for the details",
                    attachLog: true
                )
            }
        }
        failure{
            script{
                emailext(
                    to: params.Failure_Email,
                    subject: "${env.JOB_NAME} - ${env.BUILD_NUMBER} is ${env.BUILD_STATUS}",
                    body: "Please check ${env.BUILD_URL} for the details",
                    attachLog: true
                )
            }
        }
    }
}

// function to create folder and copy respective files to that folder
def folderCreation(String str){
    fileOperations([folderCreateOperation(
        folderPath: "${str}"
    )])
    // fileCopyOperation can also be used 
    sh "cp -r builds/$str.* $str"
}
