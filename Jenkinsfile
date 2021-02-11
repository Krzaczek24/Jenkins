def gitPath = 'https://github.com/Krzaczek24/BlazorApplication.git'
def dotNetPath = '"C:\\Program Files\\dotnet\\dotnet.exe"'
def projectPaths = '"DynamicManager\\__project__\\DynamicManager.__project__.csproj"'
def projectNames = Projects.tokenize(',').sort()
def projectsData = []

for (projectName in projectNames) {
	def project = projectName.tokenize('_').last()
    projectsData.add([name: project, path: projectPaths.replace("__project__", project)])
}

node {
    checkout scm
    
    stage('Checkout') {
        git url: gitPath, branch: 'CICD'
    }
    
    stage('Restore packages') {
        if (Restore == true) {
            for (project in projectsData) {
                println "Restoring packages for '${project.name}' project ... "
                bat "${dotNetPath} restore ${project.path}"
            }
        }
    }
    
    stage('Clean') {
        if (Clean == true) {
	        for (project in projectsData) {
                println "Cleaning for '${project.name}' project ... "
                bat "${dotNetPath} clean ${project.path}"
            }   
	    }
    }
    
	for (project in projectsData) {
		stage("Build '${project.name}'") {
			println "Building '${project.name}' project ... "
			bat "${dotNetPath} build ${project.path} --configuration Release"
		}
	}
	
    stage('Unit Test') {
        println "No unit tests definied"
		//bat "${dotNetPath} test C:\\Windows\\System32\\config\\systemprofile\\AppData\\Local\\Jenkins\\.jenkins\\jobs\\DynamicManager\\workspace\\DynamicManager\\Test\\DynamicManager.Test.csproj"
    }
    
    stage('Integration Test') {
        println "No unit tests definied"
		//bat "${dotNetPath} test C:\\Windows\\System32\\config\\systemprofile\\AppData\\Local\\Jenkins\\.jenkins\\jobs\\DynamicManager\\workspace\\DynamicManager\\Test\\DynamicManager.Test.csproj"
    }
	
	for (project in projectsData) {
		stage("Publish '${project.name}'") {
			println "Publishing '${project.name}' project ... "
			bat "${dotNetPath} publish ${project.path}"
		}
	}
}
