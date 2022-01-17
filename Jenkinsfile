def staticStageResult = false
def response, staticCheck, qaCheck, unitCheck
def lengthOfArray=0
def holidayCheck = true

pipeline{
    
    agent any
    
    
    environment {
        
        currentDate = sh ( script: "date +'%Y-%m-%d'", returnStdout: true ).trim()
        currentYear = sh ( script: "date +'%Y'", returnStdout: true ).trim()
    }
    
    parameters{
        checkboxParameter( name: 'checkbox', format: 'YAML', pipelineSubmitContent: "CheckboxParameter: \n  - key: Static_Check\n    value: Status_Check\n  - key: QA\n    value: QA\n  - key: Unit_Test\n    value: Unit_Test\n")
        string( name: 'Success_Email', trim: true)
        string( name: 'Failure_Email', trim: true)
    }
    
    stages{
        
        stage('Git Pull'){
            steps{
                checkout(
                   [$class: 'GitSCM', 
                    branches: [[name: '*/master']], 
                    userRemoteConfigs: [[credentialsId: 'Git_Credentials', url: 'https://github.com/sunkaravineeth/game-of-life']]
                ])
            }
        }
        
        stage(' Is the run required?'){
            steps{
                script{
                    sh 'rm -rf *'
                    staticCheck = params.checkbox.indexOf('Status_Check')
                    qaCheck = params.checkbox.indexOf('QA')
                    unitCheck = params.checkbox.indexOf('Unit_Test')
                    
                    response = httpRequest "https://calendarific.com/api/v2/holidays?api_key=90e19de04681bc27e9492b478df436f43aed8c4c&country=IN&year=${currentYear}"
                   
                    def parser = new groovy.json.JsonSlurper()
                    def json = parser.parseText(response.content)
                    def json1 = json.response.holidays
                    lengthOfArray = json1.results.size()
                    for (int i=0; i< lengthOfArray; ++i) {
                         if (json1[i].date.iso == '$currentDate}'){
                             holidayCheck = false
                         }
                    }
                    
                }
            }
            
        }
        
        stage('Build'){
            when{
                expression{
                    return holidayCheck
                }
            }
            
            steps{
                script{
                    def json2 = parseString(response.content)
                    writeJSON file: 'data.json', json: json2, pretty:2
                    
                    dir ('builds') {
                        
                        for(int i=0; i< lengthOfArray; ++i){
                            writeFile file: json2.response.holidays[i].name , text: json2.response.holidays[i].description
                        }
                    }
                      
                    sh "zip -r builds.zip builds"

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
                        stage(' Static check'){
                            when{
                                expression{
                                    return staticCheck != -1 
                                }
                            }
                            steps{
                                script{
                                    statusStageResult = true
                                }
                            }
                        }
                        stage('QA'){
                            when{
                                expression{
                                    return qaCheck != -1 && staticStageResult
                                }
                            }
                            steps{
                                script{
                                }
                            }
                        }
                    }
                }
                
                stage('Unit Test'){
                    when{
                        expression{
                            return unitCheck != -1 
                        }
                    }
                    steps{
                        script{
                        }
                    }
                }
                
            }
        }
        
        stage('Summary'){
            steps{
                script{
                    println "Stages executed based on check boxes --> "+params.checkbox
                    
                }
            }
        }
    }
    
    post{
        
        success{
            script{
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



def parseString(def json){
    new groovy.json.JsonSlurperClassic().parseText(json)
}


