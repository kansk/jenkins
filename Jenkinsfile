//TODO - Make SVN and GIT Checkout steps perfect with Jenkins way. Do not use Shell way.
node {
  echo "Parameters"
  echo "SCM Type: ${scmSourceRepo}"
  echo "SCM Path: ${scmPath}"
  echo "SCM User: ${scmUsername}"
  echo "SCM Pass: ${scmPassword}"
  echo "HTTP Proxy: ${httpProxy}"
  echo "HTTPS Proxy: ${httpsProxy}"
  
//  sh "export OS_PROJECT_DOMAIN_NAME=default"
//  sh "export OS_USER_DOMAIN_NAME=default"
  sh "export OS_PROJECT_NAME=admin"
  sh "export OS_USERNAME=admin"
//  sh "export OS_PASSWORD=password"
  sh "export OS_PASSWORD=abc123"
  sh "export OS_AUTH_URL=http://172.19.74.169:35357/v2"
//  sh "export OS_AUTH_URL=http://172.19.74.170/v3"
  sh "export OS_IDENTITY_API_VERSION=2"
  sh "export OS_IMAGE_API_VERSION=2"
  
  //--------------------------------------
  //To escape all Special Charecters in a given input string password
  def pwdstr = scmPassword
  scmPassword = pwdstr.replaceAll( /([^a-zA-Z0-9])/, '\\\\$1' )
  
  def usrstr = scmUsername
  scmUsername = usrstr.replaceAll( /([^a-zA-Z0-9])/, '\\\\$1' )
  
  def pwdstr2 = scmPassword
  def usrstr2 = scmUsername
  scmPassword = pwdstr2.replaceAll( /([@])/, '%40' )
  scmUsername = usrstr2.replaceAll( /([@])/, '%40' ) 
  //----------------------------------------
  

  stage('Code Pickup') {
    echo "Source Code Repository Type : ${scmSourceRepo}"
    echo "Source Code Repository Path : ${scmPath}"
    
    if("${scmSourceRepo}".toUpperCase()=='SVN'){
        //Not a perfect solution. Reimplement with SVN Step or checkout Step
        sh "svn co --username ${scmUsername} --password ${scmPassword} ${scmPath} ."
        
    } else if("${scmSourceRepo}".toUpperCase()=='GIT' || "${scmSourceRepo}".toUpperCase()=='GITHUB'){
        //Not a perfect Solution. Reimplement with Git Step or checkout Step
        if(scmPath.startsWith("ssh://")){
            scmPath = scmPath.substring(0, scmPath.indexOf("//")+2) + scmUsername + "@" +scmPath.substring(scmPath.indexOf("//")+2, scmPath.length());
        } else {
            scmPath = scmPath.substring(0, scmPath.indexOf("//")+2) + scmUsername + ":" + scmPassword + "@" +scmPath.substring(scmPath.indexOf("//")+2, scmPath.length());
        }  
      
        //echo "GIT PATH: ${scmPath}"
        try {
            //If we use git clone, it will not clone in the same path if we rebuild the pipeline
            sh 'ls -a | xargs rm -fr'
        } catch (error) {
        }
      //  sh " sshpass -p 'abc123' ssh ubuntu@172.19.74.170"
      
        if(scmPath.startsWith("ssh://")){
          if(httpsProxy != null && httpProxy!=null && httpsProxy.length()>0 && httpProxy.length()>0){
            echo "Looks like this Jenkins behind Proxy"
            sh "export https_proxy=${httpsProxy} && export http_proxy=${httpProxy} && sshpass -p ${scmPassword}   git clone ${scmPath} ."
          } else {
            echo "Looks like this Jenkins is not behind Proxy"
            sh "sshpass -p ${scmPassword}   git clone ${scmPath} ."
          }            
        } else {
            if(httpsProxy != null && httpProxy!=null && httpsProxy.length()>0 && httpProxy.length()>0) {
              echo "Looks like this Jenkins behind Proxy"
              sh "export https_proxy=${httpsProxy} && export http_proxy=${httpProxy} && git clone ${scmPath} ."
            } else {
              echo "Looks like this Jenkins is not behind Proxy"
              sh "git clone ${scmPath} ."
            }            
        } 
        
        //The below solutions not working with username and password
        //git "${scmPath}" //Alternate option
        //checkout scm: [$class:'GitSCM', userRemoteConfigs: [[url: scmPath ]]]
    } else {
        error 'Unknown Source code repository. Only GIT and SVN are supported'
    }
  } 
  //---------------------------------------
  
 //BUILD & PACKAGE
def appModuleSeperated = fileExists 'app'
def testModuleSeperated = fileExists 'test'
def appPath = ''
def testPath = ''
if (appModuleSeperated) {
    echo 'App Module is found , assumed that application is present in /app directory'
    appPath='app/'
} else {
    echo 'There is no defined Application path , hence it is assumed that application is in current directory'
    appPath = ''
}

if (testModuleSeperated) {
    echo 'Test Module is found , assumed that Test Cases are Present for the concerned Modules and has to be performed'
    testPath = 'test/'
} else {
    echo 'No Test Modules found , hence it is assumed that no test environment and / or test cases to be performed'
    testPath = ''
}
  if (appPath + fileExists("${FileName}")) {
    echo 'This application contains a single packer file , it is assumed to be developed in compiler in-dependant / interpreter based programming language.'
    distPackerFile = appPath + "${FileName}"
} else {
    echo 'Packerfile not found under ' + appPath
  }
// COPYING APP Directory to Current Working Directory
  def appWorkingDir = (appPath=='') ? '.' : appPath.substring(0, appPath.length()-1)  
// NEXUS file for Time Stamp comparison. This file is used for comparing time stamps and differentiating input files from generated output files.        
    sh 'echo Nexus>Nexus.txt'
//END OF INITIALIZING.
//_______________________________________________________________________________________________________________________________________________________________________  
//BUILD & PACKING
  //---------------------------------------
  if("${stage}".toUpperCase() == 'VALIDATE') {
    echo "Running packer validate on : ${distPackerFile}"
//    sh "packer -v "
    sh "packer validate ${distPackerFile}"
  
  }
  if("${stage}".toUpperCase() == 'BUILD') {
    echo 'It is inferred that the package is a Build application , hence it has to be validated , built and moved to a temporary repository'
      stage('BUILD') {
        echo "Running packer validate on : ${distPackerFile}"
        sh "packer validate ${distPackerFile}"
        sh "packer build ${distPackerFile}"
    }   
  }  else if ("${stage}".toUpperCase() == 'TEST'){
    echo 'It is inferred that the package is a test application , hence it has to be moved to a provisioned with a runtime sandbox environment , validate , build and tested before pushing into repo'
        stage('TEST') {
        echo "Running packer validate on : ${distPackerFile}"
        sh "packer validate ${distPackerFile}"
        sh "packer build ${distPackerFile}"
      }   
  }

//END OF IMAGE PUSHING INTO REPOSITORY
// NEXUS UPDATE
  stage('Publish Jenkins Output to Nexus'){
        echo 'Publishing the artifacts...';
        sh 'rm Nexus.txt'
//NEXUS FLOW ENDS HERE
  }
}
